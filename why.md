# But Why?

Why am I wasting precious time rebuilding a perfectly working lab? 

Because it wasn't actually a "perfectly working lab." I forget whether I was rebuilding a server
for the n-th time, or just troubleshooting and restarting a crashed service for the n-th time, 
but I realized that my lab was taking a lot of my time with tasks that were pure maintenance.

I have somehow managed to acquire a life, so I don't have a lot of spare time anymore. I 
still want to have the lab, and I still want to do the stuff it enables me to do. But, given the
existence of this new "life," I wanted to actually do new things with the lab rather than spend the
 little time I had doing maintenance on it. That meant I was looking for a way to make the lab 
 need as little maintenance as possible, maximizing the time I could spend *doing* stuff with it.

## The way it used to be

My first iteration of the lab looked like it came from a yard sale (which it really could have). I had
6 half-height desktop machines that I'd bought from Woot for about $40 each, and 2 Google Search boxes (more on them 
later) as beefy ElasticSearch boxes.

This worked in the short term, but hard drives kept filling, and the occasional catastrophic drive failure in 
the desktop boxes convinced me that I couldn't count on the small boxes for storage. 


## More storage, more problems

To get over the local limitations, I added a Drobo NAS to store all the data my little experiments collected.

Adding the NAS was the right thing to do, but it exposed a race condition. My house has regular short power 
outages, because the local power company is run by jerks (seriously, fuck Pepco). When I'd lose power, 
my whole lab would, obviously, shut down. When power would come back up, the lab systems would 
come back up in their own order, but the Drobo would be one of the last things to come back. The 
problem is that the servers were configured to mount the Drobo's shares at boot, and since the Drobo 
wasn't running when the servers booted, that mount would fail. Linux wouldn't re-try that mount, it would just boot 
into a busted state.

This meant that every time I lost power (sometimes quite often), I would have to go downstairs, re-mount
the RAID on each server, then restart the services that wrote to the RAID, and migrate any data that got 
saved in the wrong place onto the RAID.

Add to that the occasional hardware failure on the small boxes, ubuntu dist-upgrades failing every
single time I tried, and I found I was spending a lot of time just running in place.

That sucked. So...rebuild time.