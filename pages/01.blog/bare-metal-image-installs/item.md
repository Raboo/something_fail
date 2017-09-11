---
title: 'Linux image installs for Bare Metal'
published: true
taxonomy:
    tag:
        - 'bare metal'
        - foreman
        - image
        - cloud-init
        - pxe
visible: true
---

<img src="https://media.giphy.com/media/xsATxBQfeKHCg/giphy.gif" alt="Typical deployment" style="width: 190px;"/>
Are you like me that use preseed/kickstart netboot installs for physical nodes and perhaps use images prepped with cloud-init to deploy virtual nodes?  
Having two or more different ways of deploying bare metal, virtual servers, multiple ways of bootstrapping a node.  
Have you been thinking or wanting to deploy physical nodes the same way you would virtual ones? Are you tired of the whole setup that needs to be different on different scenarios?  
If so, read on.

So for a while I was looking for a decent solution to deploy physical/bare metal nodes in the same way we do with cloud or virtual nodes. I've saw some small mentions on one or two blog posts about installing Linux images on bare metal. But no Open Source solution for it.  
So I was thinking, how hard can this be, why haven't anyone done this? So I wanted to to do this.

My next thought was, do I have to components to be abile to achieve my goal? I went on looking at the components that we were currently using.  
We use [Foreman](https://theforeman.org) for lifecycle management of our Linux nodes. Foreman manages our PXE, DHCP and DNS and later even auto discovery.  
And with the help of Foreman we were deploying Ubuntu Linux nodes using preseed netboot installs. So installation of our bare metal node were already automated.  
But the downsides where that it took a long time to install. We had to utilize preseed which IMO sucks. It needs a lot of guesswork to get working and you end up needing to compromise. And whenver a new Ubuntu version comes you have to verify that you preseed scripts work for that version. I think the worst of all is the partman part of preseed. Don't get me started on partman, I get upset just writing about it. Then if you are going to deploy CentOS, RHEL, SuSE etc. They all have different ways of doing netinstalls (it's the linux way, no two vendors can cooperate and make something universal).

So I went on. How am I going to use this toolset I have available to write an Linux image to a hard drive.  
The idea is simple.
1. Network boot a machine.
2. Write a Linux image to disk, reboot.
3. Configure/bootstrap the OS.

I mean how hard can that be, it's just 3 steps? Well there were some challanges, but I won't go in to that cause it would lead to a very long blog post.

First of all we had Foreman in place and PXE+DHCP setup. So we are able to netboot machines.  
But then I had find a way to have a lightweight image (that has support for different types of hardware and RAID controllers) to boot and download a image and write it to disk.  
Really I wanted a Linux image or ISO that can PXE boot and directly after execute a script, so I Googled a bit and didn't find anything suiting. And I really didn't want to build and maintain my own image.

So I asked in #theforeman IRC channel on Freenode. And then I was told that [Foreman Discovery Image](https://github.com/theforeman/foreman-discovery-image) was desigend to be able to be extended. Foreman Dicovery Image is the image that boots via PXE and does a inventory of the machine and then waits for a reboot order from Foreman, in order to run a regular(preseed/kickstart) netboot install after discovery.

I then took the discovery image and used it for something that it wasn't intended originally. The extention part was mostly designed to add discovery facts or drivers.  
I used that function to shut down the discovery function and execute a set of scripts that would partition up a disk and write a Linux image to it and then setup grub plus some other stuff and reboot after.  
After the reboot the machine bootstraps using cloud-init. Same thing you can use in the cloud or some virtualization systems like OpenNebula, OpenStack, CloudStack.

For almost a year now we have been installation our bare metal in the same way we do with our virutal nodes. So if you want to try out image based installs for Linux you can find the Foreman plugin ehrm I mean a foreman discovery image plugin or whatever you want to call it on GitHub under the name [Foreman discovery image installer](https://github.com/deltaprojects/foreman_discovery_image_installer). Nope, i didn't do like this [xkcd](https://xkcd.com/910/).  
But really it can probably be used with tools other than foreman as well.