---
layout: post
title: "Resizing a Debian VM in Qemu/KVM"
date: 2021-01-18 12:00:00 -0000
categories: devops thingsiforget
---
Yet Another Thing That I Always Forget (YATTIAF?), which is potentially tricky, and which I'm documenting here as much because I want to help myself remember as because it might be useful to anyone else, is how to fully resize a disk image for my VMs. When I say "fully", I mean not only the `qemu-img` part (which is really very easy) but what happens once you have a bigger disk image file, but your VM partitions don't yet use it. 

Here's what my VM partitions look like at the beginning, when I type `lsblk` into the shell:

![Screenshot from 2021-12-01 08-05-52](https://user-images.githubusercontent.com/5275/144239923-68b9fbe3-fa97-443e-9057-8e907b0c0301.png)

You can see that the `vda` disk has three partitions. Here's where my usual problem lies - with the default disk layout when I installed Debian 11, I get a primary partition on `vda1`, followed by an extended partition, which contains a logical partition that has my swapfile on it. 
This makes the instructions in articles such as [this](https://computingforgeeks.com/resize-ext-and-xfs-root-partition-without-lvm/), although excellent, not enough. 
After I've followed those instructions:
![Screenshot from 2021-12-01 08-24-32](https://user-images.githubusercontent.com/5275/144242352-199442fd-d4f4-404c-9759-6ac047325c35.png)

I log back into the VM to see that `vda` is bigger, but `vda1` doesn't know anything about it!

![Screenshot from 2021-12-01 08-29-29](https://user-images.githubusercontent.com/5275/144243065-dffe573d-a2df-4abb-8806-637bff1afb5b.png)

To tell `vda1` it's bigger, I need to use `fdisk`, or something else that will modify my disk's partition table. That's scary to me because if I get this wrong, I can wipe out my VM's data entirely! 

I have to delete all of my existing partitions and then recreate them, only bigger. 

But first, I have to turn off writes to my swapfile, since I'm going to delete that partition and the existing swap partition will effectively get overwritten by extending the size of `vda1`.

```sudo swapoff -a```

And then the magic of `fdisk`:

![Screenshot from 2021-12-01 08-38-08](https://user-images.githubusercontent.com/5275/144244394-fecfc663-1358-48a7-977a-edbdc81e2730.png)

As you see, the output of `fdisk` looks like the output of `lsblk`. You can see that `/dev/vda1` in particular doesn't yet know about the 32GB disk size it can now operate with. We're going to tell `fdisk` how to use that extra space, by deleting partitions.

__Make sure before you start, that you have either written down all of the attributes you got from the `p` command, or at least can continue to see the output of `p` as you make these changes. You will need to remember some of those attributes!__

And then, you just press `d` three times. D for Destroy, Drop or Delete.

![Screenshot from 2021-12-01 08-44-51](https://user-images.githubusercontent.com/5275/144245425-88c99c2a-586d-48a2-bf05-043b152d33f4.png)

Now start adding partitions back. 

![Screenshot from 2021-12-01 08-48-56](https://user-images.githubusercontent.com/5275/144246025-f02a2e4c-19f1-4a9b-9dc8-f5a80a464301.png)

You'll see that you can accept the defaults for this first partition, except when choosing the size. Make sure that the start sector matches what it was before you deleted the partitions (2048 in my case) - if you get that wrong, you'll lose data! Now that the partition is 10G bigger than it was, it will overwrite what was previously in the extended partition (the swapfile). 

I then create the extended partition `vda2`:

![Screenshot from 2021-12-01 08-54-04](https://user-images.githubusercontent.com/5275/144247154-ed10ef87-8989-437b-9532-05c215f43695.png)

Noting that this partition is also a little bigger than it was previously. 

Given that I now have two partitions, one 'primary', and the other 'extended', I should now ensure I mark one of them as bootable. If not, my VM will not boot up the OS.

![Screenshot from 2021-12-01 08-54-32](https://user-images.githubusercontent.com/5275/144247769-3afcff42-5836-4a4e-8f86-df0fa1dcd0aa.png)

Then I create the actual logical swap partition:

![Screenshot from 2021-12-01 09-24-41](https://user-images.githubusercontent.com/5275/144251734-e4355188-c54d-46b4-965d-7fbc922926fd.png)

You'll notice it is of type `Linux`. To be a swapfile, it needs to be of type `Linux swap`, which is accomplished using the `t` command:

![Screenshot from 2021-12-01 09-40-13](https://user-images.githubusercontent.com/5275/144254533-65e4ab02-bf06-4c0e-80d3-a733ad4fbedc.png)

Finally, all looks well. I've added the 10GB for real to my partitioned disk. So far, my changes haven't been written to disk. When I type 'w', however, `fdisk` will write them to disk. Hopefully it will go well!

![Screenshot from 2021-12-01 12-31-34](https://user-images.githubusercontent.com/5275/144284272-5bae7b3a-c319-44db-894a-981ec2f4cf88.png)

It's finally time to start using my newly-added gigabytes. To do that, I need to format the so-far unformmatted part of `vda1`. Of course, since no changes were effectively made to the part of the disk that already existed, all of my existing filesystem should be completely untouched. 

To resize my filesystem, I use `resize2fs` on `/dev/vda1`:

![Screenshot from 2021-12-01 12-51-16](https://user-images.githubusercontent.com/5275/144287048-153054ae-ca7c-4d26-96a7-9616b3da649d.png)

Finally, I need to clean up the swapfile I wiped out. you can see it has no UUID, so I can't add it to my filesystems in `/etc/fstab`:

![Screenshot from 2021-12-01 13-32-33](https://user-images.githubusercontent.com/5275/144295285-b59c806b-cd0d-4a2d-b470-5d81a14dd041.png)

So I make a new swapfile on `/dev/vda5` and update `/etc/fstab`.

![Screenshot from 2021-12-01 14-05-52](https://user-images.githubusercontent.com/5275/144297702-6540accc-8e9a-4d82-8aee-3eb4e91dfb1f.png)

And now we're finally ready to reboot into our new, bigger world...





