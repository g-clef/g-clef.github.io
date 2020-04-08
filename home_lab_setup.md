# Setting Up My Home Lab

TLDR: my lab is 2 recycled Google search boxes running an lxd container cluster, with containers running docker (yo, 
dawg, I heard you like containers), postgres, ubuntu juju, and kubernetes, with a Drobo NAS holding all the persistent 
data.

So, the thing about my lab is that I don't actually care much about the underlying technology. The point of the lab is 
what it allows me to *do*. That may not be the case for you, and that's fine. For some folks, the point of a lab is 
for them to install new software so they can figure out how it works, how to connect the parts together, etc. That's 
great...but that's not what I'm doing. I'm trying to build a lab that lets me do stuff with the technology in it. So in 
the course of these pages, you'll see in a few places where I took the blandest, vanilla-est route possible. There's a 
reason for that (aside from me being a bland, vanilla kinda guy, I guess). Frankly, one of the things I really, 
*really* don't want to do is spend a lot of time playing with the setup of the lab, I want to spend my time *using* the 
lab. 

Also, you'll notice in a few areas that I traded either speed or money for time. Here's the thing: I have a 3-year-old. 
He's great, but he also needs lots of my time, and keeping him fed/clothed/in daycare is kinda expensive. That means my 
time (and spare change) are somewhat limited. The decisions I made in building the lab were an exercise in balancing 
priorities. For example: it would have been significantly faster for me to just get an AWS or GCP account and move 
all of my tasks up to AWS or GCP. That would have saved me a bunch of time (and probably taught me some good AWS/GCP 
skills), but in the long run would have been more expensive than running it in my basement with hardware I happened to
have lying around. Also, since I'm working with malware in a few places, I wasn't super-confident that I could trust 
AWS or GCP to not look into my cloud store and freak out at the contents.

The end result of that push-and-pull is that I'm finding I prioritize long-term stability, reliability, and ease-of-use. 
Those things are is worth money to me, since my most limited resource at the moment is time. I'll gladly trade a 
few hundred for not having to fiddle with a solution regularly. Most of my decisions are made with that in mind: get me 
to doing the actually interesting *work* as quickly as possible, while minimizing my long-term maintenance tasks. That
shows up in the design of the lab in unusual ways: I use recycled hardware when it's available, but configure it in 
redundant pairs, and I'll spend money on a RAID enclosure rather than spend a couple days building and maintaining an 
FreeNAS system.

With all that said, there are even more details available for the various parts of the lab: 

* [hardware](/lab/hardware.md) - What hardware I'm using and how I've got it set up.
* [containers](/lab/lxd.md) - The container system I'm using, how I got it set up, and how I'm using it.
* [kubernetes](/lab/kubernetes.md) - How I've got kubernetes installed & what I'm using it for

