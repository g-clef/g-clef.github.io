## Yo, Dawg, I heard you like abstraction layers



### WTF is lxd?

LXD is a virtualization system built on top of LXC. (Turns out there is an LX***E***, which is related to all of this,
but I'm not going to go into it...if we get to LXF I'll get worried.) 

LX***C*** is a container system built on top of Linux virtualization controls. LX***D*** uses LXC to run 
VMs as if they were LX***C*** containers. LXC is *kinda* like docker (Docker used to use LXC under the hood, but 
moved away from it a while ago), and LXD built on top of that to make full OS virtualization an option within LXC 
containers. Because they're still containers, it doesn't take as much overhead to run a Virtual Machine through LXD 
as it would with a regular Virtualization system like KVM. But, because they're VMs, each LXD VM gets their own 
networking stack, making it easier to get the applications you deploy in an LXD node to talk to other things (or have 
other things talk to it). In fact, LXD builds in an "Overlay" network where it assigns IP addresses to it's nodes and 
handles routing traffic between them automatically.

### Tonight, we install.

I have 3 RAID-1 volumes on each machine: the OS, and 2 storage volumes (called, creatively, "data1" and "data2"). 
What I'm going to do is make "data1" and "data2" the storage locations for the lxd daemon. I'm also going to configure 
LXD in a cluster, so that the two systems will share resources between them when deciding where to launch a new 
container or VM.

Aside: I had to install LXD via Ubuntu's *snap* system. For reasons that I understand, but find 
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
    
After some experimentation, I learned that it's important to *not* choose ***--auto*** for this step. Anyway, this 
command launched an interactive set-up prompts to configure LXD. Most of the answers to these questions were obvious, 
and the defaults reasonable, but I had to make sure to say that yes, I was making a cluster. This is the point where 
the serious note-taking started, since the answers here were important: on the first machine I was making a new
cluster, setting the password to join the cluster, creating the new bridge and fan overlay network, etc. Other than 
the password, the defaults here seemed reasonable to me (see previous comments about making things as vanilla as 
possible). 

My only concession to configuration was setting up storage. Since I'd configured the Google boxes with 3 RAID 1's, 
I wanted to use the non-OS-holding RAIDs for container storage. That meant I had to configure that in LXC. That's
fairly easy:
    	lxc storage create data1 dir source=/data1/lxd
		lxc storage create data2 dir source=/data2/lxd
		lxc storage create default dir source=/data1/lxd-default
		lxc profile device add default root disk path=/ pool=default
		
This made */data1* the place where the actual containers will live, and gave me two other pools to use to store 
other stuff (if using juju to make a new container, for example, I could add “storage=data2” to the create command
and it would be stored on the /data2 path).

I did all of this on the first machine, because when running the commands on the second machine to join the cluster,
it mostly just inherited the configuration from the first. The second *lxd init* command will as if you want to make 
a cluster (I said yes), then ask for the URL of the first server, and the password. It then asked me for the paths
to match the storage pools created on the first box, I entered those the same way they were on the first one, since the
two boxes are basically identical.

At that point, I was done with LXD/LXC setup. I had a working 2-node cluster. For more general safety, I should have 
three machines to make this a real cluster. When I get more money I'll get another Google box and some drives...this 
will do for now.