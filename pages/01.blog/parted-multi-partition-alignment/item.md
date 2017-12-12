---
title: 'parted multi partition alignment'
---

A few weeks back I was replacing a SATA-DOM with a (internal) USB stick on a node in our Hyper-Converged cluster. (Basically a OpenNebula + KVM + Open vSwitch + Ceph cluster.)
So out of 12 Nodes, we have 1 odd HP machine.
Beeing a typical HP node it need to have quirky quirks, otherwisw it wouldn't be a HP. So using our our bare-metal image installer I boot a image over the network and partition and write the Linux image to the USB. Then the installer reboots the node and it is supposed to boot into the OS, err guess again! It doesn't! So I'm at a loss here. I've done this procedure on our other Dell nodes in the cluster and it works. So my first instinct is, okay there's gotta be something wrong in the BIOS settings, I start testing shitloads of different setting combinations. That didn't work!
Ehrm I'm feeling my hair is getting grayer and grayer, so what do I do next. I start debugging the installer, it must have failed with grub or something. I spent more time that I would like to admit. But during all this debuging I get stuck cause I see a message in the output/journalctl log. The partitions are not optimally aligned. And I was devestated, cause previously I was under the illusion that I did properly align my partitions, which apperanlty I wasn't. So I spend a lot of time trying to fix that, for now I gotta keep you in suspense about this HP not booting a USB stick, cause it's more important to fix the damn alignemnt.

I a google and I get to this blog https://rainbow.chard.org/2013/01/30/how-to-align-partitions-for-best-performance-using-parted/ about how to calculate alignment. I learn from there that I can add `/sys/block/sdb/queue/optimal_io_size` with `/sys/block/sdb/alignment_offset` and divide the result with `/sys/block/sdb/queue/physical_block_size`. Sure, but I was under the illusion that parted could do the alignment for you. So I start doing a lot of trial-and-error and I learn a couple of things.

1. This Rainbow Chard blog only shows you how to align one partition.
2. The rest of the Internet is drowning in examples on how to align one partition with parted.
3. parted does handle alignment, but only under some circumstances wich are not fitting for a granular multi partition of a disk.
4. Doing multiple partitions with parted and wanting to align them all is a real pain in the...

