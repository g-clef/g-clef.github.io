# Setting up kubernetes

## Why

### The way it used to be

When I first set this up, I used juju and charmed-kubernetes. That was fine, at first, but it turned out to be very
unstable. For reasons I could never quite clear up, it would periodically re-run juju tasks on master nodes, which 
meant that the master nodes were always re-negotiating failover between them, and eventually it just fell apart, 
denying me access to the cluster (yes, I'm aware of cert renewal and the like, this wasn't that, but I have no idea
what it actually *was*). 

Charmed k8s also had a bug that I was never able to chase down where a pod on a node couldn't
reach a service if the pod answering the service was on teh same node (i.e. traffic hairpinned back to the same node).
That really limited the internal traffic ability of the cluster. I worked around that for Prefect by putting Prefect 
oustide the cluster, but it still kinda sucked.

### The way it is now

So, I replaced `juju + charmed-k8s` with `kubespray`. My thinking there was that kubespray is more officially supported 
by the kubernetes project itself (or at least, closer to the core maintainers), so it was less likely (not impossible, 
mind you) to have the fiddly bugs I was running in to. After all, I'm trying to be a *user* of k8s, not an admin of it. 

I also replaced LXC/LXD with Vagrant/Virtualbox for the google boxes, then running on metal for the picocluster.

Because this is ansible, I was also trying to do all of this remotely to the nodes. I.e. I wanted my 
development OSX box to be able to run the ansible script and provision the whole cluster automatically. I admit, 
this is not something I was dong with juju, so I'm setting a higher bar for ansible than I did with juju...I feel 
like ansible should be capable of passing this bar, however - managing remote servers is the whole *point* of ansible.

## How

### Kubespray

The next thing to do was to follow the [kubespray integration instructions](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/integration.md)
to pull the instructions to deploy the cluster into the provisioning settings. Because I'm running this on a bare 
metal cluster, I added the metalLB pod.

Kubespray's recommended way of including the kubespray commands with an existing ansible playbook didn't work for me.
The problem seemed to be that they tell you simply `include` the kubespray `cluster.yml` file, which is dynamically 
adding it to your playbook...but that cluster.yml file has lines in it that say `import_playbook`, which is statically 
linking them, and ansible did not like it when I tried to mix and match like that. 

In the end, I just left the `cluster.yml` where it was, and called it directly with my own inventory file. I
copied the `all` and `k8s_cluster` group_vars folders over to my inventory, renamed the "all.yml" file in the `all`
directory to `kubespray.yml`, and made that a group which I assigned all the nodes to. In the k8s_cluster group_vars 
directory, I only brought in the `addons`, `k8s-cluster` and `k8s-net-calico` yaml files. I modified addons to 
enable metallb, and helm, and modified k8s-cluster to have it match the cluster-name.

NOTE: You may be tempted to install ansible independently of kubespray. Don't. Kubespray at the moment requires a very
small window of versions of ansible. If you leave that out, it will complain and refuse to run. I've seen tickets that 
claim you can override that without issue, but again, I don't want to maintain ansible/kubespray, I just want to use it.
