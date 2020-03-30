## Lab hardware

#### Servers


The servers I'm using are recycled Google search boxes. For those that 
haven't seen those, Google used to sell systems that ran Google's indexing
and search software in a self-contained box. The servers I got were actually
re-branded Dell R710's. You can find something like these (or just straight R710's) on craigslist in the $400-$500 range. The ones I have came with 
48GB of RAM, 2x 8-core CPU, and no hard drives.

The first problem with the Google Search boxes was that they were set up to run only
the Linux distro that Google put on the box. Google posted instructions to "re-purpose" those systems
at https://support.google.com/gsa/answer/6055109?hl=en . In some cases, you may also want to re-flash 
the BIOS to unlock some of the other BIOS features (Virtualization, for example). the instructions
to do that are http://iblogit.net/index.php/2017/12/04/repurposing-a-google-gsa-google-search-appliance-a-step-by-step-how-to-guide/ 
Once all that was done, I was left with two boxes that would boot whatever I wanted, but no 
hard drives.

#### Storage

For all long-term/persistent storage I'm using a Drobo B810N as a NAS. Frankly, once it's set up with drives, very little
special configuration was needed for the Drobo. I did make separate drive shares for the various projects run in the lab, 
but beyond that, it needed very little special configuration or consideration.

#### Hard drives

 Frankly, hard drives were the most expensive part of this whole project. The Google boxes can hold 6 SAS drives each, 
 and the Drobo another 8. Buying 1-TB or greater drives adds up fast when you're buying 20 of them. 
 
For the servers, since they have 6 bays, in theory you could have 6 volumes on each server. 
In my experience, though, SAS drives fail pretty often, and I didn't want to
have to mess with hardware all the time...this system is a means to an end, not an end in an of itself.
So I configured the drives as 3 sets of RAID-1's. One set was for the OS (and I bought smaller
drives for this set, since the OS won't need that much storage), the other two were for the cluster
storage (more on this in later entries). I realize that RAID-1 is automatically slower, but I'm willing to trade that
speed for redundancy...I only go down to the basement to look at these servers once a month or so, and I want them
to be able to absorb a failure or two in that time without taking the whole system down.

On the Drobo, I bought some drives of the same size as the cluster storage RAIDs on the servers, but scavenged others 
from previous systems or servers I had lying around. This was my main reason for using a Drobo over other systems: 
Drobo is good at taking a diverse set of hard drives and making a usable NAS out of them. 

#### Network

I'm running all of this in a flat network at the moment, on a single switch. 