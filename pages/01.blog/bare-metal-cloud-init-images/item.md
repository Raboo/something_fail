---
title: 'Bare Metal cloud-init images'
taxonomy:
    tag:
        - 'bare metal'
        - foreman
        - image
        - cloud-init
        - pxe
---

# Linux install using images
So for a while I was looking for a decent solution to deploy physical/bare metal nodes in the same way we do with cloud or virtual nodes.
I've seen some small mentions on a few blogs about using it. But no decent Open Source solution for it.
So I was thinking, how hard can this be? Do I have to components to be abile to achieve my goal?
I started looking at the components that we use.
We use [Foreman](https://theforeman.org) for deploing Linux nodes. Foreman manages our PXE rules/scipts, DHCP server, Bind DNS and later added auto discovery.
And with the help of foreman we were deploying Linux (Ubuntu) nodes using preseed netboot installs. So installation of our bare metal node were already automated.
But the downsides where that it took a long time to install. We had to utilize preseed which IMO sucks. It needs a lot of guesswork to get working and you end up needing to compromise.
And whenver a new Ubuntu version comes you have to verify that you preseeds work for that version. I think the worst of all is the partman part of preseed. Don't get me started on partman.
Then if you are going to deploy CentOS, RHEL, SuSE etc. They all have different ways of doing netinstalls (it's the linux way, no two vendors can cooperate and make a standard).

So I went on. how am I going to use this toolset I have available to write an Linux image to a hard drive.
The idea is simple.
1. Boot a machine.
2. Write a Linux image to disk, reboot.
3. Configure the OS.

I mean, how hard can that be? Well there were some challanges.
First of all we had Foreman in place and PXE+DHCP setup. So we are able to netboot machines.

But then I had find a way to have a lightweight image (that has support for different types of hardware and RAID controllers) to boot and download a image and write it to disk.
Really I wanted a Linux image or ISO that can boot and directly after run a script, so I Googled a bit and didn't find anything suiting.

So I asked in #theforeman IRC channel on Freenode. And then I was told that Foreman Discovery Image was desigend to be able to be extended. Foreman Dicovery Image is the image that boots via PXE and does a inventory of the machine and then waits for a reboot order from foreman, in order to run a regular netboot install after discovery.

I then took the discovery image and used it to execute my scripts.

Foreman plugin ehrm I mean a foreman discovery image plugin..
