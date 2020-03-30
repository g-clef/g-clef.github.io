# Setting Up My Home Lab

I've been rebuilding my home lab recently, and have settled on a setup that 
I'm fairly happy with. It took me some trial-and-error to get here
and to figure out all the problems with this approach, though.

TLDR: my lab is 2 recycled Google search boxes running a an lxd cluster for containers, 
with containers running docker, postgres, ubuntu juju, and kubernetes, with a Drobo NAS holding all the persistent 
data.

My lab is focused on the work I tend to find interesting, so it's focused on data collection and analysis, 
rather than malware reverse engineering or simulating desktop networks, or any one of a number of other 
(perfectly reasonable) projects you could do in your home lab. 

Also, you'll notice in a few areas that I traded either speed or money for time. The decisions I made in building the 
lab were an exercise in balancing priorities. On the one hand, I don't want to spend a lot of money on the lab (and 
realistically I can't...I've got other priorities to spend money on). On the other hand, my time is also quite limited
these days, so there are points where it's worth it to me to spend money to save myself time. 

The end result of that push-and-pull is that I'm finding I prioritize long-term stability, reliability, and ease-of-use. 
Those things are is worth money to me, since my most limited resource at the moment is time. I'll gladly trade a 
few hundred for not having to fiddle with a solution regularly. Most of my decisions are made with that in mind: get me 
to doing the actually interesting *work* as quickly as possible, while minimizing my long-term maintenance tasks. That
shows up in the design of the lab in unusual ways: I use recycled hardware when it's available, but configure it in 
redundant pairs, and I'll spend money on a RAID enclosure rather than spend a couple days building and maintaining an 
FreeNAS system.

If you're interested in the details: 

* [hardware](/lab/hardware.md) - What hardware I'm using and how I've got it set up.
* [containers](/lab/lxd.md) - The container system I'm using, how I got it set up, and how I'm using it.
* [kubernetes](/lab/kubernetes.md) - How I've got kubernetes installed & what I'm using it for

