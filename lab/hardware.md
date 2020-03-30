## Lab hardware

#### Servers


The servers I'm using are recycled Google search boxes. History lesson: Google used to sell systems 
that ran Google's indexing and search software in a self-contained box. Companies would buy them to get the equivalent
of Google's search on their internal pages (intranet sites, etc). They got out of that business a few years go, but 
the servers were perfectly reasonable systems. They went through a few revisions over the years, but the servers I got 
were actually re-branded Dell R710's. You can find something like these (or just straight R710's) on craigslist in 
the $200-$300 range. The ones I have came with 48GB of RAM, 2x 8-core CPU, and no hard drives. So they were perfect
for virtualization for a moderate-to-low CPU task set.

The first problem with the Google Search boxes was that they were set up to run only the Linux distro that Google put 
on the box. Google posted instructions to "re-purpose" those systems at 
[https://support.google.com/gsa/answer/6055109?hl=en](https://support.google.com/gsa/answer/6055109?hl=en) . I also had
to re-flash the BIOS to unlock some of the other BIOS features (virtualization, for example). The instructions
to do that are helpfully posted at 
[http://iblogit.net/index.php/2017/12/04/repurposing-a-google-gsa-google-search-appliance-a-step-by-step-how-to-guide/](http://iblogit.net/index.php/2017/12/04/repurposing-a-google-gsa-google-search-appliance-a-step-by-step-how-to-guide/) 
Once all that was done, I was left with two boxes that would boot whatever I wanted, but no hard drives.

#### Storage

For all long-term/persistent storage I'm using a Drobo B810N as a NAS. Frankly, once it's set up with drives, very little
special configuration was needed for the Drobo. I did make separate drive shares for the various projects run in the lab, 
but beyond that, it needed very little special configuration or consideration. Drobo systems are a bit more expensive
than taking another system and making a NAS out of it  (using something like FreeNAS). As mentioned in the general 
comments about the lab: I'm trading money in this case for ease of setup and reliability.

#### Hard drives

Frankly, hard drives were the most expensive part of this whole project. The Google boxes can hold 6 SAS drives each, 
and the Drobo another 8. Buying 1-TB or greater drives adds up fast when you're buying 20 of them.
 
For the servers, since they have 6 bays, you could theoretically have 6 volumes on each server, each as a standalone 
RAID 0. In my experience, though, SAS drives fail pretty often, and I didn't want to have to mess with hardware all the 
time or at all, if I can avoid it...this system is a means to an end, not an end in an of itself for me. So I 
configured the drives as 3 sets of RAID-1's. One set was for the OS (and I bought smaller drives for this set, since 
the OS won't need that much storage), the other two were for the cluster storage (more on this when we talk about the 
LXD configuration). I realize that RAID-1 is automatically slower, but I'm willing to trade that speed for 
redundancy...I only go down to the basement to look at these servers once a month or so, and I want them to be able to 
absorb a failure or two in that time without taking the whole system down.

On the Drobo, I bought some drives of the same size as the cluster storage RAIDs on the servers, but scavenged others 
from previous systems or servers I had lying around. This was my main reason for using a Drobo over other systems: 
Drobo is good at taking a diverse set of hard drives and making a usable NAS out of them. 

#### Network

I'm running all of this in a flat network, on a single switch. The switch is a gigabit-capable switch
(I forget the model), but nothing flashy...I think it cost $50 on NewEgg.

#### OS

I'm running vanilla Ubuntu 18.04LTS (server) on both systems.