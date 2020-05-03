# Why?

Why am I wasting time rebuilding a perfectly working lab? 

Because it wasn't actually a "perfectly working lab." I forget whether I was rebuilding a server
for the n'th time, or just troubleshooting and restarting a crashed service for the n'th time, 
but I realized that my lab was taking a lot of my time with tasks that were pure maintenance.

Life has reached me, so I don't have a lot of spare anymore. I wanted to actually
do new things with the lab, rather than spend the little spare time I have doing maintenance. That 
means I was looking for a way to make the lab take as little of my time as possible on infrastructure, 
maximizing the time I could spend on having the lab actually *doing* stuff.

## The way it used to be

My first iteration of the lab looked like a yard-sale. I had
6 half-height desktop machines that I'd bought from Woot for about $40 each
running Linux for a task, and was using 2 Google Search boxes (more on them later) as 
the beefy ElasticSearch boxes.

This worked in the short term, but hard drive failures in the small boxes convinced me that I
couldn't count on the small boxes for storage. 


## Phase 2: more storage, more problems

To get over the local limitations, I added a Drobo RAID to store any data collected by my scripts.

Adding the NAS was the right thing to do, but exposed a race condition. My area was 
subject to repeated short power outages, because the local power company is, frankly, managed be 
jerks. When we'd lose power, my whole lab would, obviously, shut down. When power would come back up, 
systems would come back up in their own order. Usually the Drobo would be one of the last things
to come back. The problem here is that the servers were configured to mount the Drobo's shares at their
boot, and since teh Drobo wasn't running by then, that'd fail, and Linux wouldn't re-try the mount, 
it would just boot into a busted state.

This meant that every time we lost power (sometimes quite often), I would have to go downstairs, re-mount
the RAID, then restart the services that wrote to the RAID, and migrate any data that got saved in the 
wrong place onto the RAID.

Add to that the occasional hardware failure on the small boxes, ubuntu dist-upgrades failing every
single time I tried, and I found I was spending a lot of time just running in place.

That sucked.