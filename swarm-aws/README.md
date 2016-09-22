About
--------
This Cloud Formation template will deploy a configurable and scalable Docker Swarm cluster n AWS with the latest Docker code (tested on 1.12.1).

Instances are based on Centos-7 AMI.

It creates a VPC with one subnet (10.0.0.0/24), proper security group for Swarm, SSH and NFS, Network ACLs, Routing Table and Internet Gateway.

Public DNS is not configured, only public IPs will be assigned.

The token mechanism is pretty basic, tokens are shared via NFS from the "Lord" instance (which is the main Swarm manager) :)

How to launch
--------
1. Login to your AWS account

2. Navigate to CloudFormation

3. Click Design Template

4. On the bottom of the page, click the Template button

5. Paste the full code from "swarm-cloudformation.tmpl"

6. On the top of the page, click "Validate Template" icon, then "Create Stack" icon.

7. Fill in the necessary parameters (stack name, additional managers/workers, ssh key-pair) and create the stack.

8. Navigate to EC2, you will soon see the instances created by the template.

----------

Todo list:

- add multiple subnets
- add support for Availability Zone separation

Thanks:
----
- @stefan.szasz for helping eliminate hardcoded IPs