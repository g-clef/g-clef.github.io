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
the whole cluster automatically. I admit, this is not possible with juju, so I'm setting a higher bar for ansible
than I did with juju...I feel like ansible should be capable of passing this bar, however.

The first way to connect to LXD in ansible is via the LXD https interface. That has the advantage only needing a URL 
and some authentication settings. It has the disadvantage of not actually working. No matter what I tried, I couldn't get 
this option within ansible to work, it repeatedly complained about the local LXC not being set up for remote repos. 
This despite the fact that running LXC locally on the command line connected to the remote LXC servers without issue, 
and the remote repo was set up and named exactly as the ansible errors were telling me it should be. I concluded 
eventually that this was making an assumption that the LXC server was local somehow, and gave up.

The second way to connect is to have ansible ssh to the LXD host like it would for any other host it's managing, 
and run the LXD commands locally on that server. That has the advantage of meeting the assumptions about local host
that the ansible LXD plugin seems to make, but does require you to set up ssh to the lxd VMs. This is probably fine,
since remotely managing those boxes is probably something I should be doing anyway. 

Of course, there's always a catch.

The catch is that ansible's LXD plugin does not seem to be able to handle connecting to any other hosts in a cluster.
(See previous comment about assuming everything is running on localhost.) This is a problem if you have two or more 
boxes in a cluster. If you just blindly create a VM on a server that's part of a cluster, there's no guarantee that 
it will actually be run on that machine...it might get assigned to a different system in the cluster. So, if I sent 
ansible to log into `albatross1` to provision everything, LXD might put the actually-created VM on `albatross2`. The 
problem is that ansible's LXD plugin once again assumes everything is on localhost. So, when it asks the local LXC for 
the details of the just-provisioned VM, and LXC on albatross1 (accurately) says "I have no host of that name" since 
the VM is on albatross2. This causes the provisioning to fail, leaving a created-but-stopped VM on albatross2.

The way around that is to not use LXD's automatic load-balancing when provisioning VMs, which is a shame, but fine 
for the moment. So when I define the VMs for the cluster, I manually assign them to each server by hand. 

In the ansible inventory file, that looks like: 
```all:
  children:
    lxd_hosts:
        albatross1.g-clef.net:
          ansible_user: g-clef
        albatross2.g-clef.net:
          ansible_user: g-clef
    masters:
      hosts:
        lab-cluster-master-1:
          ansible_connection: lxd
          ansible_host: albatross2.g-clef.net:lab-cluster-master-1
```
The special thing to note here is that I've defined `lxd_hosts` as a group of its own, and given ansible a way to
connect to those hosts themselves (ssh via username & key). Then, in each host, I define the `ansible_host` to be the
server I want to run the VM on and then the VM name, like `<servername>:<vmname>`.

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

