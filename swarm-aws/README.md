This Cloud Formation template will deploy a configurable and scalable Docker Swarm cluster with the latest Docker code (tested on 1.12.1).

It creates a VPC with one subnet (10.0.0.0/24), proper security group for Swarm and SSH, Network ACLs, Routing Table and Internet Gateway.
Public DNS is not configured.

The token mechanism is pretty basic, tokens are shared via NFS from the "Lord" instance (which is the main Swarm manager) :)
