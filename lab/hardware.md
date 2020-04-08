## Lab hardware

#### Servers

The servers I'm using are recycled Google search boxes. 

History lesson: Google used to sell systems that ran Google's indexing and search software in a self-contained box. 
Companies would buy them to get the equivalent of Google's search on their internal pages (intranet sites, etc). Google 
got out of that business a few years ago, but you can still find the servers on ebay and craigslist occasionally. 
They're also bright yellow....I don't know why. Google apparently decided that they wanted their systems to be yellow 
(easier to find in a rack, maybe?). Since they're in my basement, their striking plumage is sadly rather wasted.

The Google boxes I got were actually re-branded Dell R710's. You can find something like these (or just straight R710's)
 on craigslist in the $200-$300 range. The ones I have came with 48GB of RAM, 2x 8-core CPU, and no hard drives. So 
 they were perfect for virtualization for a moderate-to-low CPU task set.

The first problem with the Google Search boxes was that they were set up to run only the Linux distro that Google put 
on the box, and that just wasn't going to do. Google posted instructions to "re-purpose" those systems at 
[https://support.google.com/gsa/answer/6055109?hl=en](https://support.google.com/gsa/answer/6055109?hl=en) . I also had
to re-flash the BIOS to unlock some of the other BIOS features (virtualization, for example). The instructions
to do that are helpfully posted at 
[http://iblogit.net/index.php/2017/12/04/repurposing-a-google-gsa-google-search-appliance-a-step-by-step-how-to-guide/](http://iblogit.net/index.php/2017/12/04/repurposing-a-google-gsa-google-search-appliance-a-step-by-step-how-to-guide/) 
Once all that was done, I was left with two boxes that would boot whatever I wanted, but no hard drives.

#### Storage

For all long-term/persistent storage I'm using a Drobo B810N as a NAS. Its handling of different drive sizes makes it a 
good candidate for a patched-together lab like this...I grabbed the hard drives I had available, slapped them into it, 
and didn't spend very long thinking about what size they were.

The only unusual thing I'm doing with the Drobo is that I installed the NFS add-on to allow the Drobo to offer its 
shares as NFS shares as well as CIFS. 

With that one sentence, I probably just lost the respect of a large number of the security people reading this, since 
NFS is uniquely awful from a security point of view. Allow me to try to reclaim my lost honor: 

At the time I'm writing this, Kubernetes doesn't have built-in support for mounting a CIFS volume in a pod, so 
using a built-in driver was not an option. Kubernetes does have an NFS driver, and a bunch of drivers for cloud-like 
things (AzureDisk, google cloud persistent disk, aws EBS stores). As mentioned previously, I'm looking to run as 
vanilla an installation as I could, so I went with the driver that's built in to kubernetes. 

There are other options that could have worked for this: local, or a custom driver. I decided against those because I
have semi-drunk the kubernetes kool-aid. The kubernetes folks regularly use the phrase that you should treat your nodes
 (and clusters, and frankly, the vast majority of your kubernetes infrastructure) as cattle, not pets. In other words, 
you should be prepared to kill your entire kubernetes setup periodically and you should set up your kubernetes 
jobs and deployments so that's an okay thing to do on a regular basis. With that in mind, having to run a manual 
command on every node seemed like the wrong way to go. It wasn't "kubernetes-esque." The local share or custom driver 
options would have required me to manually configure every node in my cluster with this share (or the driver), which 
also meant more manual work maintaining the cluster. 

If/when there's a CIFS mount option in kubernetes, I will quite happily remove the NFS driver from my Drobo...having 
it makes me nervous, but I don't feel like I have a much better option at the moment.

#### Hard drives

Frankly, hard drives were the most expensive part of this whole project. The Google boxes can hold 6 SAS drives each, 
and the Drobo another 8. Buying 1-TB or greater drives adds up fast when you're buying 20 of them. I scrounged what I
could from other systems I'd had lying around over the years, but mostly I just had to suck it up and buy some.

For the servers, since they have 6 bays, you could theoretically have 6 volumes on each server, each as a standalone 
RAID 0. As mentioned earlier, I'm optimizing some of this setup to minimize my time spent managing it, so I didn't do
that. I configured the drives as 3 sets of RAID-1's. One RAID was for the OS (and I bought smaller drives for this set, 
since the OS won't need that much storage), the other two were for the container storage (more on this when we talk about
 the LXD configuration). I realize that RAID-1 is slower than RAID-0, but I'm willing to trade that speed for 
redundancy...I only go down to the basement to look at these servers once a month or so, and I want them to be able to 
absorb a failure or two in that time without taking the whole system down.

One frustrating thing I've found is that SAS drives fail...often. To a certain extent when you're talking about 20 (or
more if you're a datacenter manager) this is just a law of averages: if a drive is going to fail on average after 5 
years, you should expect 5 of them to have one failure across them each year. With 20 that turns into one failure every
few months. That's a bit of an exaggeration (if drives were really failing that often it would quickly become cheaper
to do this in the cloud), but I did have a lot of trouble with drives dying either at or quite soon after delivery. 
So I ended up buying a 2 spares, just so I didn't have to deal with delivery waits to fix dead drives.


#### Network

I'm running all of these systems in a flat network, on a single switch. The switch is a gigabit-capable switch
(I forget the model), but nothing flashy...I think it cost $50 on NewEgg. So they're all in the same broadcast domain,
no fancy routing between them.

#### OS

I'm running vanilla Ubuntu 18.04LTS (server) on both systems.