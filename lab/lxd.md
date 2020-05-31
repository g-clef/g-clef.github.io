## Setting up a clustered container system with LXD
  
###WTF is lxd?

####Let's start with the basics: what is LXD, and why do I care? 

LXD is a virtualization system built on top of LXC. (turns out there is an LX**E**, which is related to all of this,
but I'm not going to go into it.) 

LX**C** is a container system built on top of some built-in Linux virtualization controls. LX**D** is  
running VMs as if they were LX**C** containers. LXC is *kinda* like docker (Docker used to use LXC under the hood, 
but moved away from it a while ago), and LXD built on top of that to make full OS virtualization an option within LXC 
containers. Because they're still containers, you're not burning as much overhead doing Virtual Machines through LXD 
as you would with something like KVM. But, because they're VMs, they get their own networking stack, making
it easier to get the applications you deploy in an LXD node to talk to other things (or have other things talk to it). 
In fact, LXD builds in an "Overlay" network where it assigns IP addresses to it's nodes and handles routing traffic
between them automatically.

####Why do I care? 

Simply: scripts and applications can be built on top of LXC (or LXD) containersallowing you to automatically deploy VMs 
(or clusters of VMs) quite easily. Also, as mentioned earlier, LXD's networking handles routing traffic between VMs (and 
from the outside), so you don't have to mess with "exposing" ports on the host OS...each LXD node has it's own IP and 
can have its own bound applications on ports just like a normal OS. Lastly, LXD has a concept of clustering built into 
it, so you can bridge machines together into a larger LXD cluster, allowing it to handle moving nodes between systems
automatically in the case of failure of one of your machines.

Most of this will be handled automatically, especially when we get to using *juju* and *conjure-up*, but for 
the moment, this is why I'm using it.

###Okay, I'm sold. Now what?

Now, we install.

As mentioned in the hardware section, I have 3 drives on my lab machines: the OS, and 2 RAID-1 volumes (called, 
creatively, "data1" and "data2"). What I'm going to do is make "data1" and "data2" the storage locations for the
lxd daemon, and then make a cluster of them to talk to each other.