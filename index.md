## The obligatory blog

Like a lot of techies, I've got a home lab. It's not the biggest, fanciest lab, but it's mine. I recently rebuilt it.
While building it *this* time, I took notes on what I was doing. These notes were taken mostly for the sake of my own 
sanity, but I thought it might help people not make the same mistakes I did if they saw what I did.

TLDR: my lab is 2 recycled Google search boxes running an lxd container cluster, with containers running docker (yo, 
dawg, I heard you like containers), postgres, ubuntu juju, and kubernetes, with a Drobo NAS holding all the persistent 
data.

* [but why?](/why.md) - Why I rebuilt it, and what it used to be
* [hardware & OS](/lab/hardware.md) - What hardware I'm using, how I've got it set up, and any special OS configurations.
* [containers](/lab/lxd.md) - The container system I'm using (LXC/LXD), how I got it set up, and how I'm using it.
* [kubernetes](/lab/kubernetes.md) - How I've got kubernetes installed
* [projects](/projects/index.md) - The fun part: the actual projects I'm working on in the lab.

#### whoami

I'm just some guy, really. I've been doing Infosec since the first .com boom, which mostly just means I've watched my 
401k get nuked 3 times (as of this writing...could be more by the time you read this).