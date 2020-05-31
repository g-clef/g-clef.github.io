## Setting up a clustered container system with LXD
  
### WTF is lxd?

#### What is this, and why do I care? 

LXD is a virtualization system built on top of LXC. (Turns out there is an LX***E***, which is related to all of this,
but I'm not going to go into it...if we get to LXF I'll get worried.) 

LX***C*** is a container system built on top of some built-in Linux virtualization controls. LX***D*** uses LXC to run 
VMs as if they were LX***C*** containers. LXC is *kinda* like docker (Docker used to use LXC under the hood, but 
moved away from it a while ago), and LXD built on top of that to make full OS virtualization an option within LXC 
containers. Because they're still containers, it doesn't take as much overhead to run a Virtual Machine through LXD 
as it would with a regular Virtualization system like KVM. But, because they're VMs, each LXD VM gets their own 
networking stack, making it easier to get the applications you deploy in an LXD node to talk to other things (or have 
other things talk to it). In fact, LXD builds in an "Overlay" network where it assigns IP addresses to it's nodes and 
handles routing traffic between them automatically.

### Now, we install.

As mentioned in the hardware section, I have 3 drives on my lab machines: the OS, and 2 RAID-1 volumes (called, 
creatively, "data1" and "data2"). What I'm going to do is make "data1" and "data2" the storage locations for the
lxd daemon. I'm also going to configure LXD in a cluster, so that the two systems will share resources between them
when deciding where to launch a new container or VM.

So what did I do:

First note: I had to install LXD via Ubuntu's *snap* system. For reasons that I understand, but find 
regrettable, Ubuntu is trying to move everything away from the debian-based *apt* system and over to *snap*. This means
that projects like LXD will really only appear in the *snap*. My 18.04 systems didn't already have snap installed, so 
I had to do:

    sudo apt-get install snap
   
(Note If snap already exists, that won't break anything. If it doesn't, you'll get snap. Hooray.)

Next, I needed to install LXD (which will also install lxc and a few other things). In Ubuntu 18.04LTS they include an
older version of LXD in apt, which only caused confusion and delay. I had to uninstall that, then install lxd via snap:

    sudo apt-get remove lxd lxd-client lxcfs liblxc-common liblxc1 lxc-utils lxc-common  --purge
    snap install lxd

Next I set up the first LXD daemon. 

    lxd init
    
(Note, do not choose ***--auto*** for this step.) This launched an interactive set up questions to configure LXD.
For the first question (do you want to run this in a cluster), I said "yes". This triggered a bunch of followup
questions to configure the cluster. Some of those answers were things that I needed to have noted down (the cluster
password for example)