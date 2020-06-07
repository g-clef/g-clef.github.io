# Why NFS?

NFS is a uniquely awful protocol from a security point of view, so why am I using it? Its security is basically
to trust that you are who you say you are, which is...problematic at best. I'd rather not be using it, but I'd 
also *really* rather not spend the time to set up its replacement, either...and the incentive to set up a 
better way to share files was too low for a home lab. I'd rather be doing actual research than dorking with 
infrastructure. 

## Secure by default...except when the default doesn't exist

So how did I get here? The default share from a Drobo is a Samba/CIFS share. At the time I'm writing this, Kubernetes 
doesn't have built-in support for mounting a CIFS volume in a pod, so using a built-in driver was not an option. 
Kubernetes does have an NFS driver, and a bunch of drivers for cloud-like things (AzureDisk, Google Cloud persistent 
disks, AWS EBS stores). So if I wanted to use a built-in volume type to mount a volume in a pod, NFS seemed to be 
the only way to do it.

Kubernetes does have some other volume types that could have worked with CIFS: local mount, or a custom driver. The
problem is that both of them required doing some manual configuration on each node that would run the pod. In the 
local case, I'd have to log into each  kubernetes node, and configure it to mount the volume when it starts. In the 
custom driver case, I'd have to log into each node and manually install the driver. 

I decided against using those for two reasons: 1) I wanted to have the least "custom" (aka the most vanilla) setup 
possible since I wanted to spend more time using the cluster instead of maintaining it; and 2) the kubernetes folks 
regularly use the phrase that you should treat your nodes (and clusters, and frankly, the vast majority of your 
kubernetes infrastructure) "like cattle, not pets." In other words, you should be prepared to kill your entire 
kubernetes setup periodically, and you should set up your kubernetes jobs and deployments so that's an okay thing 
to do on a regular basis. Manually configuring every node goes against that, and signs me up for a bunch of futzing
with nodes every time I do a rebuild, which is what I'm specifically trying to avoid. Since the kubernetes folks release 
new versions of kubernetes every 6 months or so, and stop supporting previous versions after 18 months, blowing 
up your entire kubernetes infrastructure is a fairly regular thing to do, so I need my setup to be as vanilla 
as possible so that I don't have to manually do a bunch of extra work every time I update kubernetes. That means no 
local mounts or custom drivers.

If/when there's a CIFS mount option in kubernetes, I will quite happily remove the NFS driver from my Drobo. I don't 
like using it, but I don't feel like I have a much better option at the moment. It's a shame that the "default" way to 
mount external volumes in a home kubernetes network is to use the most insecure option. I suspect that's because 
they're focusing on supporting the cloud providers rather than labs like mine, but it's still a shame.