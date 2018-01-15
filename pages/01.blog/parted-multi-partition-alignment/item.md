---
title: 'parted multi partition alignment'
published: true
date: '2018-01-12 11:53'
taxonomy:
    category:
        - blog
    tag:
        - USB
        - parted
        - pxe
visible: true
---

This post describes how I realised that I wasn't aligning our partitions and how I did to properly align multiple partitions using parted.

A few weeks back I was replacing a SATA-DOM with a (internal) USB stick on a node in our Hyper-Converged cluster. (Basically a OpenNebula + KVM + Open vSwitch + Ceph cluster.)
So out of 12 Nodes, we have 1 odd HP machine.
Being a typical HP node it need to have quirky quirks, otherwise it wouldn't be a HP. So using our our bare-metal image installer I boot a image over the network and partition and write the Linux image to the USB. Then the installer reboots the node and it is supposed to boot into the OS, err guess again! It doesn't!

So I'm at a loss here. I've done this procedure on our other Dell nodes in the cluster and it works. So my first instinct is, okay there's gotta be something wrong in the BIOS settings, I start testing shitloads of different setting combinations. That didn't work!
So my next instinct is to try the different pxe boot options.

```
KERNEL <%= foreman_server_url %>/files/helpers/chain.c32
APPEND hd0
```

```
LOCALBOOT 0
```

```
LOCALBOOT -1
```

I even upgrade the NIC firmware and try the above stuff again.

Ehrm I'm feeling my hair is getting greyer and greyer, so what do I do next. I start debugging the installer, it must have failed with grub or something. I spent more time that I would like to admit. But during all this debugging I get stuck cause I see a message in the output/journalctl log. The partitions are not optimally aligned. And I was devastated, cause previously I was under the illusion that I did properly align my partitions, which apparently I wasn't. So I spend a lot of time trying to fix that, for now I gotta keep you in suspense about this HP not booting a USB stick, cause it's more important to fix the damn alignment.

I did a google and I get to this blog [https://rainbow.chard.org/2013/01/30/how-to-align-partitions-for-best-performance-using-parted/](https://rainbow.chard.org/2013/01/30/how-to-align-partitions-for-best-performance-using-parted/) about how to calculate alignment. I learn from there that I can add `/sys/block/sdb/queue/optimal_io_size` with `/sys/block/sdb/alignment_offset` and divide the result with `/sys/block/sdb/queue/physical_block_size`. Sure, but I was under the illusion that parted could do the alignment for you. So I start doing a lot of trial-and-error and I learn a couple of things.

1. This Rainbow Chard blog only shows you how to align one partition.
2. The rest of the Internet is drowning in examples on how to align one partition with parted.
3. parted does handle alignment, but only under some circumstances which are not fitting for a granular multi partition of a disk.
4. Doing multiple partitions with parted and wanting to align them all is a real pain in the...

So basically you can use parted to align multiple partitions IF you use percentage. I tried

```bash
parted -a optimal -s -- /dev/sdX mkpart primary 0% 32MiB
parted -a optimal -s -- /dev/sdX mkpart primary 0% 32MB
```

Both of these fail caused next partition to not be aligned. I was under the misconception that you only need to align the start/first partition of the drive and the rest will be aligned. WRONG!
So what is wrong with above statement? First I have a small static size, apparently my USB ~32MB in alignment size. So it actually needs to start somewhere around where I end the partition in above examples.

Then I test something like

```bash
parted -a optimal -s -- /dev/sdX mkpart primary 0% 64MB
```

Ok, so now the partitions starts at a correct alignment position, but it doesn't end at one, or perhaps it does, but I'm not sure, but next partition is still not aligned.

So how should I go about this? I decided to use parted to calculate the optimal alignment size for me and use that size setup the different partitions.
I create a partition to get the sizes.

```bash
# create a temporary partition to find out optimal alignment sizes.
parted -a optimal -s -- /dev/sdX mkpart primary 0% 1%
# optimal align byte size
byte=$(parted -s -- /dev/sdX unit B print | grep " 1 " |awk '{print $2}')
byte=${byte::-1}
# optimal align sector size
sector=$(parted -s -- /dev/sdX unit S print | grep " 1 " |awk '{print $2}')
sector=${sector::-1}
# remove the temporary partition now that we know the sizes.
parted -s -- /dev/sdX rm 1
```

So now I the variables `byte` and `sector` with the optimal alignment size in bytes and sectors.
With those two variables we can create the partitions we want using basic math.

Let's create a small grub partition. I decided to size it as a minimal alignment size (this is not necessary, I'll show on the next partition how to size it to your choice). 

```bash
end=$((${sector}+${sector}))
parted -a optimal -s -- /dev/sdX mkpart primary ${sector}S ${end}S
parted -s -- /dev/sdX name 1 grub
parted -s -- /dev/sdX set 1 bios_grub on
```

Now we need to set a starting point for next partition.

```bash
start=$(parted -s -- /dev/sdX unit S print | grep " 1 " |awk '{print $3}')
start=${start::-1}
start=$((${start}+${sector}))
```

Let's make a 8GB swap partition.

```bash
end=$((8*1024*1024*1024)) # 8GB in bytes.
round=$((${end}/${byte})) # round it using optimal align byte size.
end=$((${byte}*${round})) # so alignment size in bytes + rounded size.
parted -a optimal -s -- /dev/sdX mkpart primary linux-swap ${start}S ${end}B
parted -s -- /dev/sdX name 2 swap
```

Ok, now just repeat to calculate start size for next partition (greping the second partition).

```bash
start=$(parted -s -- /dev/sdX unit S print | grep " 2 " |awk '{print $3}')
start=${start::-1}
start=$((${start}+${sector}))
```

Now that we have start position, let's create the last partition and let it end at end of disk.

```bash
parted -a optimal -s -- /dev/sdX mkpart primary ext4 ${start}S -1
parted -s -- /dev/sdX name 3 rootfs
parted -s -- /dev/sdX set 3 boot on
```

Alright, so I've showed how to align multiple partitions with with parted using division, multiplication and addition.
And now I'll finish the side story on what was wrong in the one odd HP server.

So after searching the entire net. I found a [Server Fault question](https://serverfault.com/a/750083/154660) about someone that had troubles booting from a USB stick on a HP Microserver. And the answer explained that it needs to be a MBR partition table and not a GPT when booting from USB on that device. So I figured that it must be the same thing on HP ProLiant Rack Servers.

So without this HP node not booting from the USB stick I wouldn't have grey hair and I wouldn't know that I didn't align my partitions properly. Still I'm wondering why HP doesn't treat a USB stick as a regular HDD?