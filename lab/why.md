# But Why?

Why am I wasting precious time rebuilding a perfectly working lab? 

Because it wasn't actually a "perfectly working lab." I forget whether I was rebuilding a server
for the n-th time, or just troubleshooting and restarting a crashed service for the n-th time, 
but I realized that my lab was taking a lot of my time with tasks that were pure maintenance.

I have somehow managed to acquire a life, so I don't have a lot of spare time anymore. I 
still want to have the lab, and I still want to do the stuff it enables me to do. But, given the
existence of this new "life" thing, I wanted to actually do new things with the lab rather than spend the
 little time I had doing maintenance on it. That meant I was looking for a way to make the lab 
 need as little maintenance as possible, maximizing the time I could spend *doing* stuff with it.

## The way it used to be

My first iteration of the lab looked like it came from a yard sale. It consistend of 6 half-height desktop machines 
that I'd bought from Woot for about $40 each, and 2 Google Search servers. The desktop machines ran the tasks, and 
the Google search boxes ran a minimally-redundant ElasticSearch cluster.

This worked in the short term, but hard drives kept filling, and the occasional catastrophic drive failure convinced me 
that I couldn't count on any of the servers (big or small) for storage of data. 

## More storage, more problems

To get over the local limitations, I added a Drobo NAS to store all the data my little experiments collected. (it's now
a Synology one, more on this in the [hardware](/lab/hardware.md)) section.)

Adding the NAS was the right thing to do, but it exposed a race condition. My house has regular short power 
outages, because the local power company is run by jerks (f&*k Pepco). When I'd lose power, 
my whole lab would, obviously, shut down. When power would come back up, the lab systems would 
come back up in their own order, but the NAS would be one of the last things to come back. This caused a problem 
because the Linux servers boxes were configured to mount the Drobo's shares at boot, and since the Drobo 
wasn't running when the servers booted, that mount would fail. Linux wouldn't re-try that mount, it would just boot 
into a busted state.

This meant that every time I lost power (sometimes quite often...seriously, f%^k Pepco), I would have to go downstairs, 
re-mount the RAID on each server, then restart the services that wrote to the RAID, and migrate any data that got 
saved in the wrong place onto the RAID.

Add to that the occasional hardware failure on the small boxes, Ubuntu dist-upgrades failing for a new reason every
 time I tried, and I found I was spending a lot of time just running in place.

That sucked. So...rebuild time.

## Why not just go to the cloud?

Since I want to spend more time with family but still have a lab, why not outsource the whole problem to 
aws/google/azure? I'm not doing that for a few reasons:
  1) Trust - Part of the point of this cluster is that I'm doing research on malware. While the various cloud providers 
all *say* that they won't reach inside customer datasets, at my scale I have no monetary or reputational weight to 
make sure that stays true. I've already gotten emails from my 'net provider telling me that they noticed my malware 
scraping traffic, and were concerned one of my systems had been hacked. The last thing I need is for an AWS support 
person to get one of those emails, look at my S3 bucket-o-malware, freak out, and delete multiple years of work at one 
swoop. 
  2) Cost - While buying servers and drives is expensive, it's a cost that gets amortized over a *long* time. I'm fairly
convinced that the long term cost of running my own lab is lower than it would be if I were running it in the cloud, 
but much spikier. I (probably) wouldn't get a $4k bill from AWS in one month, the way I did when I was buying a NAS and
a bunch of drives for it, but I wouldn't be at all surprised if my lab equivalent in AWS would cost that over the 
course of a year or so. I'm very lucky to have the income to be able to absorb the one-time cost spikes so that I can 
save more in the long run.

## Why do this all a *second* time?

The notes above cover why I did it the first time, why would I go to the trouble of rebuilding it again?

Frankly, while the Google Search boxes it ran on for the first version of the lab were cool (and very colorful),
they use a *lot* of power. The [picocluster](https://www.picocluster.com/products/pico-20-raspberry-pi4-8gb) 
will have just as much processing power (frankly, more) and probably more storage space per node once I add
USB SSDs. Overall, while the picocluster was expensive, I expect it'll pay for itself in power costs within a 
year or so.