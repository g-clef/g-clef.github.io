# Connecting to SMB shares from kubernetes

By default, kubernetes does not support pure SMB shares. I think this is an mistake, but apparently none 
of the major cloud providers want to offer that, they have their own custom file sharing protocols (because, of 
course they do). This makes it hard to run a k8s-based lab at home, since out of the box the only options are 
NFS (ick), or manually mounting file shares on every node in the cluster (gods, no). 

## CSI

Kubernetes does, however, support writing your own storage systems. They call them "Container Storage Interfaces" or
CSI. In the case of SMB shares, there is a kubernetes-supported project that supports SMB, which makes it possible
to use SMB protocols. You just have to install it in your cluster. It's called, unsurprisingly, csi-smb, and it's here:
https://github.com/kubernetes-csi/csi-driver-smb . 

Installing it is fairly straightforward, as long as you're running a recent kubernetes cluster version. When I 
first tried to install it, I was on 1.17, and it never worked right. Upgrading to 1.21 made the installation
effortless. In my case, I used helm, and just followed the instructions at https://github.com/kubernetes-csi/csi-driver-smb/tree/master/charts
and used the default helm chart. 

The result of that installation is a few pods running in my `kube-system` namespace, and an ability to 
write persistent volumes and persistent volume claims connecting to smb shares.

## The catch

Of course, it's not quite that simple. Firstly, because it's not built in to kubernetes itself, using the 
custom storage interface itself consumes some resources inside the cluster. So you will see pods running as
"controller"s, which offer the service to the cluster overall, and control things like provisioning. You 
will also see "node" pods for CSI, which are special pods that are deployed one-per-node to coordinate 
making the actual mounts on the nodes, and offering those mounts up to the pods running on those nodes.

Also, because the pods are reaching down to the OS for various mount options, the CSI install will need to 
be deployed on "privileged" containers. Allowing this isn't always the default.

## The result

Once you have it up & running, writing PersistentVolumes and PersistentVolumeClaims becomes fairly straightforward.
I've been using them all over teh place, but one example is here: https://github.com/g-clef/stoq_transformer/blob/main/kubernetes/persistent_volumes.yaml

A few important things to keep in mind about that example: 
 1) I'm using a pre-defined share created on my NAS. That means that the `capacity` line in the yaml is going to be ignored. That's fine.
 2) To actually connect to the share, you will need to also create a k8s secret to hold teh credentials. An example of one of those secrets is here: https://github.com/g-clef/stoq_transformer/blob/main/kubernetes/persistent-volume-secrets.yaml

