# Setting Up My Home Lab

I've been rebuilding my home lab recently, and have settled on a setup that 
I'm fairly happy with. It took me some trial-and-error to get here
and to figure out all the problems with this approach, so in the interest
of sparing someone else that work, I'm going to write down what I 
did here. 

The short version of all of this is that my lab is: 2 recycled Google search boxes running a an lxd cluster, with containers
running docker, and kubernetes, with a Drobo RAID holding all the persistent data.

My lab is mostly focused on the work I tend to find interesting, so it's mostly
focused on data collection and analysis, rather than malware reverse engineering or 
simulating desktop networks, or any one of a number of other (perfectly reasonable)
projects you could do in your home lab. 

Each section can get a bit into the weeds, so I'm going to split things up into three
sections: 

* [hardware](/lab/hardware.md) - What hardware I'm using and how I've got it set up.
* [containers](/lab/lxd.md) - The container system I'm using, how I got it set up, and how I'm using it.
* [kubernetes](/lab/kubernetes.md) - How I've got kubernetes installed & what I'm using it for

