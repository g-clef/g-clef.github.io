# Setting up kubernetes

## Why

When I first set this up, I used juju and charmed-kubernetes. That was fine, at first, but it turned out to be very
unstable. For reasons I could never quite clear up, it would periodically re-run juju tasks on master nodes, which 
meant that the master nodes were always re-negotiating failover between them, and eventually it just fell apart, 
denying me access to the cluster (yes, I'm aware of cert renewal and the like, this wasn't that, but I have no idea
what it actually *was*). Charmed k8s also had a bug that I was never able to chase down where a pod on a node couldn't
reach a service if the pod answering the service was on teh same node (i.e. traffic hairpinned back to the same node).
That really limited the internal traffic ability of the cluster. I worked around that for Prefect by putting Prefect 
oustide the cluster, but it still kinda sucked.

So, I replaced `juju + charmed-k8s` with `kubespray`. My thinking there was that kubespray is more officially supported 
by the kubernetes project itself (or at least, closer to the core maintainers), so it was less likely (not impossible, 
mind you) to have the fiddly bugs I was running in to. After all, I'm trying to be a *user* of k8s, not an admin of it. 

I left LXD/LXC in place, though, as the reasons for using it remained true.

## How

### Provisioning

So, the first problem is provisioning, i.e. setting up the VMs that we'll hand to kubespray to run kubernetes on. 
Since kubespray is based on ansible, it seemed reasonble to make the provisioning use that, too. This was, of course, 
the first problem. Ansible does have an LXD plugin, and that plugin does have the ability to define hosts and LXC 
policies. It even has two ways to connect to an LXD server (which seems overkill). They both are, however, 
ever so slightly broken. 

Before I launch into the problems, it's worth noting that because this is ansible, I was trying to do all of this 
remotely to the VM servers. I.e. I wanted my development OSX box to be able to run the ansible script and provision 
the whole cluster automatically. I admit, this is not something I was dong with juju, so I'm setting a higher bar for 
ansible than I did with juju...I feel like ansible should be capable of passing this bar, however - managing remote
servers is the whole *point* of ansible.

The first way to connect to LXD in ansible is via the LXD https interface. That has the advantage only needing a URL 
and some authentication settings. I eventually gave up on this approach as it repeatedly complains about missing 
remotes. I strongly suspect in hindsight that it needed the `ansible_lxd_remote` value to be set (more on this below)
in order for this to work, but by the time I'd figured that out I'd given up on the https interface entirely. (I should
note that the documentation about this doesn't mention this setting at *all*, which is not okay.) 

The second way to connect is to have ansible use the lxc shell commands. That has the advantage of meeting the 
assumptions about local host that the ansible LXD plugin seems to make, but does require you to set up ssh to the lxd 
VMs. This is probably fine, since remotely managing those boxes is probably something I should be doing anyway. It also
requires you to set up lxc "remotes" for each lxc server you want to provision things on to. Be careful naming these,
as you're going to need to use them later. 

For the record, an LXD "remote" is a connection to a remote lxd/lxc server. For this all to work you will have to
have LXC installed locally on your system, and have connected it to your lxc/lxd cluster by running 
`lxc remote add <name> <url> --accept-certificate --password <lxd cluster password>`.

Of course, there's always a catch or two.

The first catch is that you have to tell ansible explicitly which LXC "remote" to connect to when running commands.
The vast majority of LXD's documentation doesn't mention the "remote" setting, because most of their examples are set 
up assuming that you're running this on the same host with LXD, so you will have the default "local" lxc entry. If you 
don't set the `ansible_lxd_remote` variable for each host, then ansible will try to connect to the lxc "remote" named 
`local`, and you'll get weird errors that say things like "This client hasn't been configured to use an LXD remote yet." 
(An error which is super-unhelpful when you can trivially type `lxc remote list` and see them plain as day.) You will 
need to be careful that the names of the remotes listed in `lxc remote list` match the remote names you set in the 
ansible inventory.

The second catch is that host provisioning, for some reason, *only* works when calling lxc locally. So you will need
to delegate that task to be run directly on the VM servers themselves. That, of course, leads to a third catch: if 
you provision a host in an LXD cluster, LXD may not put it on the host that you run the commands on...if there's more
overhead available on another host in the cluster, it'll put the host there, instead. The problem here is that 
ansible's provisioning commands ask the *local* LXC for the status of the host after it's created...and the local
LXD won't have that host if it's created somewhere else. The way around that is to not use LXD's automatic 
load-balancing when provisioning VMs, which is a shame, but fine for the moment. So when I define the VMs for the 
cluster, I manually assign them to each server by hand. 

In the ansible inventory file, this all looks like: 
```all:
  children:
    lxd_hosts:
      hosts:
        albatross1.g-clef.net:
          ansible_user: g-clef
        albatross2.g-clef.net:
          ansible_user: g-clef
    masters:
      hosts:
        lab-cluster-master-1:
          ansible_connection: lxd
          ansible_lxd_remote: albatross2
          delegated_to: albatross2.g-clef.net
```
The special thing to note here is that I've defined `lxd_hosts` as a group of its own, and given ansible a way to
connect to those hosts themselves (ssh via username & key). Then, in each host, I define the `ansible_lxd_remote` 
to be the server I want to run the VM on. **NOTE** this name is the name that you see for this host in `lxc remote list`.
That may not be the same as its hostname or full network name. You'll also see a `delegated_to` entry. That's for

Then, in the actual role definition, I also had to make the worker/master tasks look like: 
```
- name: Create & start worker container
  delegate_to: "{{ ansible_host.split(':').0 }}"
  community.general.lxd_container:
    name: "{{ inventory_hostname }}"
    state: started
    source:
      type: image
      mode: pull
      server: https://images.linuxcontainers.org
      protocol: lxd
      alias: ubuntu/focal/amd64
    profiles: ["default", "{{ hostvars[inventory_hostname]['cluster_name']}}-worker-profile"]
    wait_for_ipv4_addresses: true
    timeout: 600
    target: "{{ ansible_host.split(':').0.split('.').0 }}"
```
Note the `delegate_to` and `target` entries. `delegate_to` tells ansible to ssh to that host first before trying to 
run any of the LXD provisioning commands, and `target` tells the LXD provisioning to put the VM on this particular
server.

You can see the full set up of this in the [malwarETL-k8s](https://github.com/g-clef/malwarETL-k8s) repository, which
has all my ansible build scripts for this effort (including the LXC policies, which I've skipped talking about here).

### Kubespray

The next thing to do was to follow the [kubespray integration instructions](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/integration.md)
to pull the instructions to deploy the cluster into the provisioning settings. Because I'm running this on a bare 
metal cluster, I added the metalLB pod.

Kubespray's recommended way of including the kubespray commands with an existing ansible playbook didn't work for me.
The problem seemed to be that they tell you simply `include` the kubespray `cluster.yml` file, which is dynamically 
adding it to your playbook...but that cluster.yml file has lines in it that say `import_playbook`, which is statically 
linking them, and ansible did not like it when I tried to mix and match like that. 

In the end, I just left the `cluster.yml` where it was, and called it directly with my own inventory file. Instead, I
copied the `all` and `k8s_cluster` group_vars folders over to my inventory, renamed the "all.yml" file in the `all`
directory to `kubespray.yml`, and made that a group which I assigned all the nodes to. In the k8s_cluster group_vars 
directory, I only brought in the `addons`, `k8s-cluster` and `k8s-net-calico` yaml files. I modified addons to 
enable metallb, and helm, and modified k8s-cluster to have it match the cluster-name.

NOTE: You may be tempted to install ansible independently of kubespray. Don't. Kubespray at the moment requires a very
small window of versions of ansible. If you leave that out, it will complain and refuse to run. I've seen tickets that 
claim you can override that without issue, but again, I don't want to maintain ansible/kubespray, I just want to use it.

NOTE 2: It may be tempting to run a different version of ubuntu underneath the images (18.04 vs 20.04). This is a mistake.
The kubespray install process will try to install kernel modules in the VMs, which won't have the same kernel version 
installed as what they're actually running, so the kernel module installation will fail. This also means you have to 
make sure the right kernel modules are available on the core VM servers with the LXC policy

NOTE 3: I was tempted to change the container runtime to containerd or cri-o, after all the noise about kubernetes
eventually deprecating docker. However [this ticket](https://github.com/kubernetes-sigs/kubespray/issues/6979) implies
that if you do that, etcd has to be installed locally (instead of inside the container framework), which I'd frankly
rather avoid if I can. 
