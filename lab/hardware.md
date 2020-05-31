## Lab hardware

### TLDR

For those of you who don't want to read all the discussion:
 * Servers: 2x Dell r710's (actually google search boxes, but they're just rebranded R710's)
   * 1 RAID-1 for OS, 2x RAID-1's for container storage.
 * NAS: Drobo B810N
   * Drives: Some same as in the Servers, others I scrounged from systems I had leftover from previous iterations
 * Network: Totally flat - everything's on a $50 switch
 * OS: Vanilla Ubuntu 18.04 LTS.

#### Servers

History lesson time: Google used to sell systems that ran Google's indexing and search software as an 
appliance - companies would buy them to get the equivalent of Google's search on their internal pages. Google 
got out of that business a few years ago but didn't take back the servers, so the companies that had them
sold them or donated them when the service stopped being supported.  Google released multiple variations 
on these guys, but they were usually re-braded commercially available servers.  The ones I got were 
re-branded Dell R710's. You can find something like these (or just straight R710's) on craigslist in the 
$200-$300 range. The ones I have came with 48GB of RAM, 2x 8-core CPUs, and no hard drives. So 
they were perfect for virtualization for a moderate-to-low CPU task set. They're also bright 
yellow. Since they're in my basement, their striking plumage is wasted.

The first problem with the Google Search boxes was that they were set up to run only the Linux distro that Google put 
on the box, and that just wasn't going to be okay...I needed to be able to patch the OS, at the very least, etc. 
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
and the Drobo another 8. The cost of 1-TB or greater drives adds up fast when you're buying 20 of them. I 
scrounged what I could from other systems I'd had lying around over the years (especially the previous iterations
of the lab), but eventually I just had to suck it up and buy some hard drives.

For the servers, since they have 6 bays, you could theoretically have 6 volumes on each server, each as a standalone 
RAID 0, which would maximize the storage space available on the servers. I didn't do that, for a couple reasons. 

First, local storage on the servers isn't actually that useful. I'm trying to design so that all the storage I actually
*care* about is on the NAS. So having tons of storage on the server doesn't really help that cause. 

Second, I'm optimizing this setup to minimize my time spent managing it. Maximizing the storage with RAID 0 also 
maximizes the effect of a single drive failure. One frustrating thing I've found is that SAS drives fail...often. 
To a certain extent when you're talking about 20 (or more if you're a datacenter manager) this is just a law of 
averages: if a drive is going to fail on average after 5 years, you should expect 5 of them to have one failure across 
them each year. With 20 that turns into one failure every few months. That's a bit of an exaggeration (if drives were 
really failing that often it would quickly become cheaper to do this in the cloud), but I did have a lot of trouble 
with drives dying either at or quite soon after delivery.

So I configured the server drives as 3 sets of RAID-1's. One RAID was for the OS (and I bought smaller drives for 
this set, since the OS won't need that much storage), the other two were for the container storage. I realize that 
RAID-1 is slower than RAID-0, but I'm willing to trade that speed for redundancy...I only go down to the basement 
to look at these servers once a month or so, and I want them to be able to absorb a failure or two in that time 
without taking the whole system down.

#### Network

I'm running all of these systems in a flat network, on a single switch. The switch is a gigabit-capable switch
(I forget the model), but nothing flashy...I think it cost $50 at Newegg. So they're all in the same broadcast domain,
no fancy routing between them.

#### OS

I'm running vanilla Ubuntu 18.04LTS (server...never run a UI on Linux if you can avoid it. That's a subject for another 
time) on both systems.