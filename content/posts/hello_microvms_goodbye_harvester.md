+++
date = '2026-05-29T21:07:49+02:00'
title = 'Hello MicroVMs Goodbye Harvester!'
author = "FKouhai"
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "nixoS", "kubernetes", "microvms", "k3s","microvms"]
keywords = ["nix", "kubernetes","harvester","k3s","cluster"]
description = "Migrating away from harvester into nixos microvms"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## Intro
Over the past couple of weeks I've been playing with the idea of ditching [harvester](https://harvesterhci.io/) in favor of using nixos microvms.

This is the story of how that went.

## What is Harvester
Harvester is a hypervisor which runs kube-virt and actually has an API (cough cough looking at you Proxmox).

Although the API was useful to deploy VMs through terraform and their official provider(cough cough unlike Proxmox), I dont really have enough hardware to justify running such a beast that eats around 6G of RAM on startup alone.

If you are wondering why, that is a really good question if you asked me and after doing some digging to see whats running under the hood:

Harvester itself as mentioned earlier is designed for hyper-convergent infrastructure, under the hood it's running a full Kubernetes control plane through RKE2, Longhorn for distributed storage, and Grafana + Prometheus + Loki for monitoring. That alone accounts for most of
the memory footprint, but then there's Rancher.

You guessed it another SUSE service, which is a management service for kubernetes clusters, that deploys more SUSE services, amongst which it includes fleet, another gitops control plane to deploy containers.

The rancher integration is really important to the story, as they also have an addon that can be enabled through the UI which gives you the extra context on all of the pods/deployments that are running in the cluster.

One of the biggest issues with it, is that the rancher add-on alone takes around 1.5 to 2G of RAM, and while I was blinded by the flashing lights of the fancy and confusing UI menus inside of menus inside of submenus.

I didnt notice at the time the big memory footprint it has and how much memory it was quietly eating away from my server.

My Hardware at the time:

* 3x raspberry pi 4b of 8GB of RAM (running a k3s cluster)
* 1x old workstation with a ryzen 7 2700x and 32GB of RAM (where harvester was deployed)
* 1x NAS (used for backups and storage)
* 1x cisco switch with 8p running at 1G speed

Taking into context the hardware where its running, I'm aware that Im basically using the bare minimum RAM that is required to run Harvester, it was good enough for my use case of running 2 VMs that were part of my k3s cluster.


## The fateful weekend 

### Why do I listen to people on the internet

I was bored on a weekend where the thought about migrating away from the kube-proxy and using cilium to its fullest seemed like a great idea, especially since it was only being used as the CNI and wasnt using a lot of the extra features Cilium has.

The kube-proxy replacement is well documented and usually goes smoothly, so I figured it was straightforward enough to have an LLM do it, and test if the internet hysteria was right about LLMs taking everyone's jobs.

And I went onto doing what every vibe coder on the internet does which is, start token maxxing on plan mode in my "agent" (yes a pretty confusing name), dump into the context the nix defined config, let it read the documentation and "told it" about ArgoCD managing whats deployed on the cluster and start feeling the vibes.

By the time it finished yapping, it was time to read some of the plan, as people on the internet where saying models one shot their tasks, so to prove those claims, the reasonable way was allowing it to be as autonomous as possible which on the other hand felt like a good test for some of the capabilities.

Some of you may think this is a [replit 2.0](https://medium.com/lets-code-future/the-replit-ai-deleted-my-entire-database-and-said-sorry-8f7923c5a7dc) incident and you wouldn't be wrong.

### The disaster piece or how to ruin a saturday night

The model started by updating the node configuration to disable kube-proxy. After redeploying through Colmena, for a split second it felt like the AI bros were right.

Right after came the update to the `values.yaml` for Cilium, committed straight to source control for ArgoCD to pick up.

And oh boy, let me tell you it brought the entire cluster to its knees as if it was a [raging kid telling its mom to get out of the room cause he's playing minecraft](https://www.youtube.com/watch?v=bMGwwVRvSu4)

Network connectivity all of a sudden became optional just like the side missions on the Assassins Creed Oddyssey, and the nodes had no way to talk to each other nor to anything, at this point I was starting to feel what my initial skepticism was telling me all along, AI bros are not real.

I told the model, hey "We dont have any network connectivity against the nodes, there might be something wrong with the cilium config", the bot went full robocop mode agreeing with my assesment and that this time pinky promise it will work, so it downloaded the chart on the version that I was using, took a look at the yaml, made some changes and re-commited.

Which is fine, but you may notice that, well without any form of network connectivity on the nodes, how is the argocd repo-server supposed to bring the changes.

Well, it just isnt and it won't.

After that fiasco, restarting the Pis and VMs seemed like the obvious move. What only became clear on the second reboot was that a race condition was lurking: the application controller needs time to come up, but if the CNI isn't ready the network stack goes down again, meaning neither the repo-server nor the application controller can do their job and apply the new config.

At that point the easiest path was clear: uninstall Cilium, redeploy with mostly default config, and re-enable kube-proxy.

Then I told Agent Smith about it had it reapply the chart values and just like in The Matrix it betrayed the human race and so the cluster sank deeper than the titanic.

But this time, instead of the network stack going dark, it was the control plane node hosted on one of the PIs that took the hit, and it was pretty much dunzo at this point. 

I didnt have a mini HDMI to HDMI that I could use to plug the pi to a monitor and just stop the k3s service, since for some reason it was memory hogging to the point where ssh wasnt even an option.

Then I ended up doing what any responsible adult would do: rebuild the entire cluster, but this time I'd have the control plane on Harvester, since I suspected the slow sd cards were partly causing those issues whilst benefiting from having the VM console available if something goes wrong.


### Rebuilding the cluster

Now rebuilding the entire cluster was pretty straightforward, as all the nodes were running NixOS and their configs are source controlled just like the services on the cluster were managed by ArgoCD.

Though I never accounted for backing up the etcd database so the secrets and anything else that was not managed by Argo directly would just be erased, but at that point I was already too deep in the rabbit hole just like Alice going all the way to Wonderland.

The fleet came back up with kube-proxy disabled, I reviewed the actual Cilium chart values, enabled WireGuard tunnels for pod-to-pod communication, recreated the secrets, configured etcd snapshots to ship to an S3 bucket, and all that jazz.

Little did I know that was the beginning of the end in my relationship with Harvester.


## Moving on to greener pastures


### The boiling point

After the cluster was rebuilt, I started to notice sporadic drops in connectivity against the k3s cluster, turns out Harvester was running into resource exhaustion and started comitting virtual harakiri.

So as an SRE the first thing I do is take a look at grafana, see if there were any spikes on CPU or memory usage, in fact I saw a spike on CPU usage that caused the 3 VMs and the harvester service to stop.

Then I decided to decrease the CPU those machines request down to 4CPUs each.

After a couple of days I decided to check on the cluster again and I noticed that I couldnt access it again, so I repeated the same drill, and noticed that the memory usage was a bit too high, and only scaled down the worker node down to 6GB of RAM.

At this point I was already annoyed by the situation, specially after doing a quick math, the 3 nodes only took 24G of RAM and now only 12 cores, having a total of 8G of RAM and 4 cores available for the rest of the system, which should be more than enough.

And then the final drop of water on the glass that is my patience, the cluster went down again the day after, I went onto grafana and took a look at the other services the cluster was running to see if there's something that could be done, there it was, the monitoring stack taking 4 gigs of ram for themselves, so the not so wise choice was to disable the add-on to reclaim some extra memory and cpu usage.

While that happened I've also noticed the VMs were not starting up, and to my surprise the volumes got stuck while getting recreated, then I went ahead and deleted the stuck volumes, after that then the VMs were able to start normally.

Since Monitoring was disabled to free up some resources which turned out to be foolish of me, I also wanted to keep some alerting around the cluster and found out that what I needed was to deploy [uptime kuma](https://uptimekuma.org/) on my NAS just to get an alert on when the Harvester decides to destroy my hopes and dreams of becoming the next hollywood star.

At this point I've already made up my mind about moving away from harvester, the memory and CPU footprint were too large for the little benefit I had of getting an API.

### Planning a trip to the promise land

I've previously heard of the term microvm but I was still a bit unclear on what it was, I've heard of technologies like [firecracker](https://firecracker-microvm.github.io/) that implement microvms, having some companies like [fly.io](https://fly.io/blog/sandboxing-and-workload-isolation/) powering their infrastructure with them.

A microvm sits somewhere between a container and a full VM, its more isolated than a container since it runs its own kernel rather than sharing the host's, but it's lighter than a traditional VM since it strips out a lot of the overhead that a hypervisor would normally carry, so kind of like a VM that actually hits the gym and its looks maxxing.


A long time ago I found on github the [microvm.nix](https://github.com/microvm-nix/microvm.nix) repo and I really liked the idea of having my VMs declaratively and never finding an answer to the question of what things nix is unable to do.

So the migration plans were already in the works, since I wanted to keep some availability on the cluster, the plan was simple:

1. Promote one of the PIs to be part of the control plane
2. Sync up ETCD
3. Remove the older node from the etcd cluster, to prevent issues with quorum as etcd implements [raft](https://raft.github.io/)
4. Scale down the Argocd and authentik deployments to 0 pods
5. Remove the VM nodes from the cluster (to prevent registration issues on the new startup)
6. Install nixos using nixos-anywhere on the harvester node and rollout the microvms
7. Be happy

### Did we get to the promise land?

So after executing steps 1-5 perfectly with some minor hiccups like having a stale k3s state on the node that I chose to promote, that got fixed by doing:
```bash
sudo systemctl stop k3s
sudo rm -rf /var/lib/rancher/k3s/agent
sudo rm -rf /var/lib/rancher/k3s/server/db/
sudo rm -rf /var/lib/rancher/k3s/server/tls/
sudo systemctl start k3s
```

Its stale state was due to the node being the previous control plane node of the cluster, and it was making it pretty difficult to join the etcd cluster.
I then checked the etcd status:
```bash
sudo etcdctl \
  --endpoints https://<node_1>:2379,https://<node_2>:2379 \
  --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt \
  --cert /var/lib/rancher/k3s/server/tls/etcd/server-client.crt \
  --key /var/lib/rancher/k3s/server/tls/etcd/server-client.key \
  endpoint status --write-out=table
```

Once etcd got synced I removed  the older node from the cluster firstly by getting its member ID and then removing it:
```bash
sudo etcdctl \
  --endpoints https://<node_ip>:2379 \
  --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt \
  --cert /var/lib/rancher/k3s/server/tls/etcd/server-client.crt \
  --key /var/lib/rancher/k3s/server/tls/etcd/server-client.key \
  member list


sudo etcdctl \
  --endpoints https://<node_ip>:2379 \
  --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt \
  --cert /var/lib/rancher/k3s/server/tls/etcd/server-client.crt \
  --key /var/lib/rancher/k3s/server/tls/etcd/server-client.key \
  member remove <node-member-id>
```

After all of that the plan went more or less smoothly, I did encounter a minor hiccup with nixos anywhere, where I was being completely unable to install nixos on top of harvester, this was due to when the NixOS kernel was getting loaded with kexec I couldnt get network connectivity against the machine to finish up the installation.

So I decided that I was gonna flash a usb with a NixOS and just run nixos-anywhere that way, especially since I already had the drive setup configured with disko.

```bash
nix run github:nix-community/nixos-anywhere -- \
     --flake .#copernico \
     --phases disko,install \
     --ssh-option StrictHostKeyChecking=no \
     root@<node_ip>
```
And once all was said and done, it worked like a charm.
```bash
$ microvm -l
epsylon: current(nixos-system-epsylon-26.05pre-git)
worker03: current(nixos-system-worker03-26.05pre-git)
worker04: current(nixos-system-worker04-26.05pre-git)
```

### What did I learn about getting to the promise land

The first takeaway is that what matters is not the destination but the friends you make along the way.

But on all seriousness I got to play with microvms which was super interesting, I learned that sometimes even having a nice API doesnt buy happiness, despite what you are told online AI is not particularly great at infrastructure although its pretty decent at some things I wouldnt trust an LLM touching a production system

Always backup your etcd database if you are not using a managed kubernetes service, homelab is not production so its always fun to experiment with and try new things

Overall I think that harvester crashing out on me was a blessing in disguise as now my homelab configuration is more unified through nixos, so far the cluster has been really stable and I got to do things I havent done in a long time.
