# Setting up kubernetes

## Why

I set up kubernetes on top of Juju, which is an ubuntu-managed provisioning system, using `charmed kubernetes`. 
Juju can deploy to LXD/LXC, so once LXD was up, installing charmed kubernetes was fairly simple.

The advantage of doing it through juju and charmed kubernetes is that I'm not managing any of the 
installation myself. I'm very aggressively trying to be a user of kubernetes, not an admin of it. 

## How:

	1) snap install juju —classic

	2) juju add-cloud albatrosses
			choose lxd cloud type

			enter the URL of the first machine in the cluster (from the “lxc cluster list” command)
			
			I’m fine with default region, and the default cloud api url
		
			add cloud to this client? Y
		That command will end with a comment that you need to add a credential to this client.

		juju add-credential albatrosses 
			only add it to this client
			give it a name (albatrosses-remote-pw)
			any region
			interactive auth
			(paste the password from the lxd setup here)

	3) juju bootstrap albatrosses
		this will do a bunch of stuff, and download stuff

	4) juju status 
        this should show an empty system with a controller, and model “admin/default” is empty

	5) run “juju deploy hello-juju”. This will spin up a new virtual machine, and push a test application onto it, called ‘hello-juju”
		run “juju status” to watch it get deployed. You’re looking for the machine to be “started”, the “unit” “workload” to be “active” and the app to be “active”

		to test that it’s running take the IP from the machine and do “curl <IP>”. You should get “Hello Juju!” Back again.


	6) delete the app hello-juju: juju remove-application hello-juju
		note: at this point, you may have an empty machine left behind. Juju should automatically clean that up, but if it doesn’t, you can do “juju remove-machine <machine number>”

    7) juju deploy charmed-kubernetes


That's it. Since this is run on top of lxc, each "machine" created in juju is actually an LXC Virtual Machine, and
LXC handles assigning them to different servers automatically. 