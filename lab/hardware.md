## Lab hardware and base OS setup

### TLDR

For those of you who don't want to read all the discussion:
 * Servers: 2x Dell r710's (actually google search boxes, but they're just rebranded R710's)
   * 1 RAID-1 for OS, 2x RAID-1's for container storage.
 * NAS: Drobo B810N
   * Drives: Some same as in the Servers, others I scrounged from systems I had leftover from previous iterations
 * Network: Totally flat - everything's on a $50 switch
 * OS: Vanilla Ubuntu 18.04 LTS.

#### Servers

When I originally started this all, I had 6 small ex-desktop servers, and 2 re-purposed Google search servers. The 
more I looked at usage, the small boxes were overloaded with tasks, and the search servers were bored. This seemed 
backwards. In fact, when looking at workloads (especially if I was going to use Kubernetes), it started to look like
I didn't need the small servers at all, and could just make containers out of the jobs I'd run on the desktop servers,
so I gave away the desktop servers, and made the Google boxes my only servers. 

Google released multiple variations on these servers, but they were usually re-braded commercially available systems.  
The ones I got were re-branded Dell R710's. You can find something like them (or just straight R710's) on 
Ccraigslist for a few hundred dollars each. The ones I have came with 48GB of RAM, 2x 8-core CPUs, and no hard drives. 
They're perfect for virtualization for a moderate-to-low CPU task set. They're also bright yellow. Since they're in my 
basement, their striking plumage is, sadly, wasted.

The first problem with the Google Search boxes was that they were set up to run only the Linux distro that Google put 
on the box, and that just wasn't going to be okay...I needed to be able to patch the OS, at the very least. 
Google posted instructions to "re-purpose" those systems at 
[https://support.google.com/gsa/answer/6055109?hl=en](https://support.google.com/gsa/answer/6055109?hl=en) , which was
a good start. I also had to re-flash the BIOS to unlock some of the other BIOS features (virtualization, for example). 
The instructions I followed to do that are at this link: 
[http://iblogit.net/index.php/2017/12/04/repurposing-a-google-gsa-google-search-appliance-a-step-by-step-how-to-guide/](http://iblogit.net/index.php/2017/12/04/repurposing-a-google-gsa-google-search-appliance-a-step-by-step-how-to-guide/) 

Once all that was done, I was left with two boxes that would boot whatever OS I wanted, but no hard drives. "Whatever I 
wanted" from an OS point of view was Ubuntu LTS 18.04. Why? Because I don't want to have to spend time messing with
stuff...Ubuntu isn't perfect, but it's easy for me to get going with it quickly.

So now I need somewhere for the various applications to store stuff (the local server storage isn't the right place),
and hard drives for the servers.

#### NAS Storage

For all long-term/persistent storage I'm using a Drobo B810N as a NAS. Its handling of different drive sizes makes it a 
good candidate for a patched-together lab like this...I grabbed the hard drives I had available, slapped them into it, 
and didn't spend very long thinking about what size they were.

The only unusual thing I'm doing with the Drobo is that I installed the NFS add-on to allow the Drobo to offer its 
shares as NFS shares as well as CIFS. With that one sentence I probably just lost the respect of a large number of the 
security people reading this, since NFS is uniquely awful from a security point of view. Allow me to try to reclaim 
some of my lost honor [here](NFS.md).


#### Hard drives

Hard drives were the most expensive part of this whole project. The Google boxes can hold 6 SAS drives each, 
and the Drobo another 8. The cost of 1-TB (or more) drives adds up fast when you're buying 20 of them. I 
scrounged what I could from other systems I'd had lying around over the years (especially the previous iterations
of the lab), but eventually I just had to suck it up and buy some hard drives.

For the servers, since they have 6 bays, you could theoretically have 6 volumes on each server, each as a standalone 
RAID 0, which would maximize the storage space available on the servers. I didn't do that, for a couple reasons. 

First, local storage on the servers isn't actually that useful. I leared the hard way with the previous iteration of 
my lab that local storage (even if in a RAID), while fast and easy, is a bad idea for data I want to keep and/or do
other analysis on. (If all your data is on server1, that means you have to run all the analysis jobs on that server,
which is a bit of a pain in the neck.) So I tried to design the lab so that all the data I actually
*care* about is on the NAS. So having tons of storage on the server doesn't really help that cause. 

Second, I'm optimizing this setup to minimize my time spent managing it. Maximizing the storage with RAID 0 also 
maximizes the effect of a single drive failure. One frustrating thing I've found is that SAS drives fail...often. 
To a certain extent when you're talking about 20 (or more if you're a datacenter manager) this is just a law of 
averages: if a drive is going to fail on average after 5 years, you should expect 5 of them to have one failure across 
them each year. With 20 that turns into one failure every few months. That's a bit of an exaggeration (if drives were 
really failing that often it would quickly become cheaper to do this in the cloud), but I did have a lot of trouble 
with drives dying either at or quite soon after delivery.

So I configured the server drives as 3 sets of RAID-1's. One RAID was for the OS (I used smaller drives for 
this set, since the OS won't need that much storage), the other two were for the container storage. I realize that 
RAID-1 is slower than RAID-0, but I'm willing to trade that speed for redundancy...I only go down to the basement 
to look at these servers once a month or so, and I want them to be able to absorb a failure or two in that time 
without taking the whole system down.

The RAID configuration was done in the Dell PERC setup program (Ctrl-R during boot). The main thing I took care to 
do was to pair the physicaldrives to be in RAID-1 groups physically near each other: the drive bays in the
 Google boxes are 2 high, and 3 across, so each vertical column was one RAID set. That way I can easily see which 
 RAID group is in trouble if a given drive indicator light turns orange. 

#### Network

I like packets, and networking...I really do, but there's really no reason to make this network fancy in any way. 
I'm running all of these systems in a flat network, on a single switch. The switch is a gigabit-capable switch (I 
forget the model), but nothing flashy...I think it cost $50 at Newegg. The servers are in the same broadcast domain,
no fancy routing between them.

#### OS

I'm running vanilla Ubuntu 18.04LTS  on the servers. I'm only running the server version. Life lesson: never 
run a GUI on Linux unless you like rebuilding your Window manager every other year. As mentioned before, I don't enjoy
that anymore...looking back at the number of times I found myself with a broken window manager after a dist-upgrade, 
I don't think I enjoyed it then, either. Why I kept doing it remains a mystery to me.
