+++
title = '''maybe it's time for a lab'''
date = 2025-04-29T09:10:58+10:00
draft = false
+++

I'm 3.5 years into a career as a Security Consultant - it's about time I started to skill up more generally. I've learnt a lot in this short space of time, but there are still so many gaps in my knowledge simply because I didn't study anything computer related at school or university.

A home lab seems like a good place to start.

First step - get a Lab PC. I picked up a nice, beefy Beelink Mini PC with a Ryzen CPU for about $800 AUD.

Now what did I intend on doing in this "home lab"? Well, I wanted to:
- Quickly be able to spin up tooling to learn how they work
- Learn docker and set up the tools in containers to keep the host clean
- Set-up a lab environment to test Active Directory infrastructure
	- see [GOAD](https://github.com/Orange-Cyberdefense/GOAD)

I always felt that installing tools on the base OS (think specific dependencies, outdated tools, or even just shiny, new tools) created such a minefield of directories and files and orphaned packages that just cluttered the whole experience. Not to mention updates breaking tools, which has happened a few times across a few Linux distros I've played with.

A colleague of mine suggested to try Proxmox instead. The convincing statement was along the lines of, "By setting everything up in Proxmox, you can create containers or VMs and install tools directly without needing to worry about docker. And if you break something, it's easy to spin up fresh instances."

Trying to keep everything clean by using docker felt like it was going to add a lot of unnecessary overhead; it would add a layer of complexity for things like networking or troubleshooting. My goal wasn't actually to master docker, but rather just to quickly spin up tools and test things.

So what did Proxmox allow? It meant I could make Virtual Machines or Containers en masse, when I wanted them. I could even set-up a VM and carry out my initial docker plan if I really wanted to. And if I screwed up, I wouldn't have to wipe the entire device and start from scratch.

Ok. I have a box and I have the perfect base OS for my needs.

Now the hard part of actually doing it.

I set this box up at my parent's house. At the time I was living in a share house and I didn't necessarily want to leave an always-on, lab environment sitting in the open near the router even if it was an unobtrusive, inconspicuous mini PC. Setting up Proxmox was as easy as following the GUI installer and most of the setting were left as default. But how would I access the lab if I wasn't physically at my parent's? "A VPN of course!" I hear you say. Though it would be nice to have easy configuration, minimal overhead, and the option for scalability.

My dad actually had a solution for this...

The magic of [Tailscale](https://tailscale.com/). (And it really feels like magic)

Tailscale is not necessarily a new tool, nor a new idea, but boy is it a well-polished product. It's simply a mesh overlay VPN network using Wireguard. Essentially, every device that you add to your tailnet can directly connect to any other machine in your network. It has access control lists, the ability to provide access to nodes to other users, and perks like Magic DNS (again... magic) making it a breeze to connect to other nodes in your tailnet. Honestly, it's just really easy to use.

It also has this feature where you can turn a node into a subnet router, meaning a single node can let you access a network subnet, which was the exact use case that I was after. So the first thing I set-up in Proxmox was a lightweight Ubuntu LXC to host my Tailscale subnet router (they even have a tutorial to configure an Apple TV as a subnet router[^ts-docs][^ts-youtube]). Now I can be anywhere in the world, and as long as my laptop is connected to the internet and that specific container in my homelab is online, I can connect to my Tailscale account and pivot into the lab environment.

All without the security headache of opening up or exposing any ports to the internet as well!

The next few weeks I spent spinning up the Game of Active Directory (GOAD). Now the maker of this vulnerable lab environment ([`@mayfly`](https://twitter.com/M4yFly)) wrote up a tutorial[^mayfly-blog] for how to set this up within Proxmox. Following along was fairly straight-forward but did require a few tweaks along the way. Most of the troubleshooting steps were actually covered by [`@_0xBEN_`](https://twitter.com/_0xBEN_), but I found his blog[^ben-blog] a little too late into the process.

The tweaks tl;dr:
- Needed to add additional PFSense rules to allow the Provisioning Box and GOAD Lab to have internet access.
	- The Provisioning Box requires access so you can download and set-up the tools needed to create and manage the lab.
	- The GOAD Lab needs this so the Ansible playbook can configure the machines appropriately.
- The Provisioning Box was easier to set-up (less breakages and errors) using Ubuntu 22.04 LTS as it had `Python3.10` instead of `Python3.12`.
- The Packer Templates needed to specify the VM disk formats as `raw` instead of `qcow2`.
	- This was due to the configuration of the storage formats within my Proxmox environment. YMMV.
- And finally, make sure to change the French keyboard option (if you're not using a French keyboard) from `/root/GOAD/globalsettings.ini` to save yourself a headache when trying to troubleshoot issues during the Ansible playbook stage. Set the value to your specific keyboard preference.
	- By default, the GOAD settings file has the French and US keyboards in that order. I missed this initially and spent a solid 10 minutes struggling to understand why the credentials `vagrant:vagrant` weren't working - where in reality I was inadvertently typing in `vqgrqnt` as the password. Whoops!

Eventually, I had a vulnerable AD Lab configured in Proxmox and threw in a Kali VM on the VLAN network alongside the GOAD boxes. All those troubleshooting steps along the way helped me learn that little bit more about networking and firewall configurations. Still got a long way to go, but one step at a time.

Now I've got to convince myself to actually use it!

[^ts-docs]: https://tailscale.com/kb/1280/appletv#advertise-apple-tv-as-a-subnet-router

[^ts-youtube]: https://www.youtube.com/watch?v=hYd5etBpsO0

[^mayfly-blog]: https://mayfly277.github.io/posts/GOAD-on-proxmox-part1-install/

[^ben-blog]: https://benheater.com/proxmox-lab-goad-installing-the-lab/
