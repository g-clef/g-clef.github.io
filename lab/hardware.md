## Lab hardware and base OS setup

### TLDR

For those of you who don't want to read all the discussion:
 * Nodes: one [picocluster 20H raspberry pi](https://www.picocluster.com/products/pico-20-raspberry-pi4-8gb)
   * Added 500GB SSDs to each node.
 * NAS: Synology DS1821+
   * Added 32 GB RAM
   * 8x 6TB drives
 * Network: All in one subnet
 * OS: Vanilla Ubuntu 22.04 LTS.

#### Servers

When I originally started this all, I had 6 small ex-desktop servers, and 2 re-purposed Google search servers. The 
more I looked at usage, the small boxes were overloaded with tasks, and the search servers were bored. This seemed 
backwards. In fact, when looking at workloads (especially if I was going to use Kubernetes), it started to look like
I didn't need the small servers at all, and could just make containers out of the jobs I'd run on the desktop servers,
so I gave away the desktop servers, and made the Google boxes my only servers. 

This worked well for a while, but my montly power bill started to make me unhappy. So, I shifted over to a 
cluster of 20 Raspberry PI 4b's. Why 20? The original setup allowed me to set up a kubernetes cluster with a 
total of 10 nodes (3 masters, 7 workers). This was fine, but I did run into resource constraints at times,
and there wasn't an option for a 15-node picocluster. So, in the interest of giving the lab a bit of an 
upgrade, and lowering the power at the same time, I went with the 20-node version. 

Assembling the picocluster turned into a bit of a [challenge](/lab/picocluster_20_notes.html) , however.

One note on the picocluster beyond its assembly: because the pi's are using external storage, they
pull more power than a vanilla PI4 would. That is a bit more complicated than it should be...more below.

#### NAS Storage

Since I want to treat the servers as basically disposable, all the actual data needs to live somewhere else. 
That's where the NAS comes in. In this case, the NAS I'm using is a Synology DS1821+. I splurged and bought the
32GB RAM expansion and 8x 6TB drives for it, so I'm not concerned about running out of space. The NAS itself
is configured with 2 pools of drives, one for the actual lab storage, and one for image storage for Virtual 
Machines that will run directly on the NAS. Part of my reason for going with something like the Synology over the 
previous Drobo was the ability to run VMs on the NAS itself. (more on that when I talk about running Prefect)

When I originally started this I was using a Drobo B810N for storage. That worked, but had some limitations, and
it appears that Drobo is slowly losing support capability (they've dramatically pared back their product line, not 
released anything new for a while, and apparently their support is getting non-responsive). Also, after a power
failure, the Drobo wanted me to fsck the hard drives on it, which is a problem since that is almost 
impossible to do safely (there is no procedure supported by Drobo to fsck the drives on one of their NAS'. I 
feel like that's a major oversight). I wasn't feeling terribly good about the long term life of the Drobo, so I 
replaced it.


#### Node Storage

I wanted each node in the cluster to have a fair amount of local space available, because I want jobs running on the
nodes to be able to decompress fairly large files without having to do that over a network share. So, I gave each 
pi node a 500GB nvme drive, attached by USB. That drive is actually the boot drive as well, so none of the nodes are 
running from SD cards. This speeds up the node's IO, and also saves me from having to worry about processes with lots
of disk writes burning out the SD card. I did run into one problem, though: as of when I wrote this, the SSD drives
I was using all required me to set "usb quirks" mode on the OS for those drives. If I didn't do this, the drives were
not usable on the usb-3 adapters on the pi boards. [This post on the pi forums](https://forums.raspberrypi.com/viewtopic.php?t=245931)
outlines what to do, with one extra caveat: if you're using Ubuntu "cmdline.txt" is actually at `/boot/firmware/cmdline.txt`

It's also worth noting that the Pi boards don't have *quite* enough power to reliably run NVMe drives from USB. They
appear to at first - they would start, and would run okay initially, but nodes would gradually drop out of the cluster 
over time as they hit some power draw that was *just* over the limit of their power. What this looked like on the 
network was a node that would ping, but would reject any ssh login. Basically, the kernel and the daemons were all
still running, since the Pi was still up, but the moment it tried to read anything from disk, it failed. 
This doesn't seem to be a limitation of the power provided by the Picocluster power supply itself, as I tried 
unplugging the switches, and had the same problem. I ended up having to purchase 20 small powered USB hubs to avoid 
this. That was a pain, and made the outside of the cluster *really* messy, because now there's a tangle of USB and 
power lines around the pico cluster. But moving that USB power to outside the PI fixed the problem.


#### Network

I like packets, and networking, I really do, but there's really no reason to make this network fancy in any way. 
I'm running all of these systems in a flat network. The pico cluster comes with 4 switches, which you are encouraged
to bridge together (and are given cables to do so). I did not do this. I ended up connecting all of the pi nodes 
directly to my lab switch. This was initially done when I was troubleshooting the power draw of the usb drives (to see
if I could get enough power to the pi boards without driving the switches). That did not solve the problem, but 
once I had all the pis plugged into the lab switch, there didn't seem to be any value in removing them all again.

My home networking switch is a farily vanilla gigabit-capable switch. While I do want to keep things simple, I did 
make one concession to reality, though: the lab ports are all on a separate network from the regular house network.
Since the lab will have live malware on it, I wasn't a super-fan of having that be auto-discoverable from family 
devices.

#### OS

I'm running vanilla Ubuntu 22.04LTS  on the servers. I'm only running the server version. Life lesson: never 
run a GUI on Linux unless you like rebuilding your Window manager every other year. As mentioned before, I don't enjoy
that anymore. Looking back at the number of times I found myself with a broken window manager after a dist-upgrade, 
I don't think I enjoyed it then, either. Why I kept doing it remains a mystery to me.
