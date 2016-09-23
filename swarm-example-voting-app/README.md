Swarm Mode Demo with Instavote
=========

This is a simple walkthrough for setting up multiple components of an application in a Docker 1.12.1 Swarm cluster with multiple Swarm networks.

The entire tutorial should take approximately 1 hour if you already have an AWS account.

The images in this demo have been pushed to this Docker repo: https://hub.docker.com/u/vralex/

Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

More details about the components here:
https://github.com/docker/docker-birthday-3/blob/master/tutorial.md#dockercompetition

Getting started in AWS
---------------

(30 minutes): If you're using a fresh AWS account (you can [create a free account](https://aws.amazon.com)), you will need to first initialize your account by creating a free Centos-7 instance and creating a SSH Key Pair. The tutorial in this page requires your AWS account to be subscribed to the "Centos-7" AMI.

1. Login to your AWS account

2. Navigate to "**Services**" > "**Compute**" > "**EC2**"

- Select "**Ireland**" zone from top right
3. Navigate to "**Key Pairs**" (on the left of the page, under "NETWORK & SECURITY")

4. "**Create Key Pair**" with name: "**docker-meetup**"
- make sure you save the private key, you will not be able to download it again

5. Navigate to "**Instances**" (on the left of the page)

6. Launch your first Instance using the wizard (accept all defaults), **use the "Centos-7" AMI** (search in Marketplace, part of the instance launch wizard)

7. Wait for the instance to be created, then "**Terminate**" the instance

Your AWS account is now ready for the rest of the workshop.

----------

(1 hour): These steps apply to all meetup workshop participants:

1. Login to your AWS console

- Select "**Ireland**" zone from top right

2. Navigate to "**Services**" > "**Management Tools**" > "**CloudFormation**"

3. Click "**Create Stack**"

4. Download the Cloud Formation template to your computer from [here](https://raw.githubusercontent.com/alexvranceanu/docker-meetup-swarm/master/swarm-aws/swarm-cloudformation.tmpl)

5. Choose option "**Upload a template to Amazon S3**" and choose the Cloud Formation template file you downloaded previously

6. Click **Next**

7. Enter the following details:

- "**Stack name**": docker-swarm

- "**Number of additional managers**": 1

- "**Number of workers**": 1

- "**SSH Key Pair**": docker-meetup

8. Click **Next** until you reach the Review page, then click **Create**

9. Wait 5-10 minutes :) until the stack is created

10. Navigate to Services > Compute > EC2
- at this point you should see three instances created by the template (one initial manager called Lord, an additional Manager and one Worker)

11. The following steps will walk you through setting up the Swarm application, good luck (or have fun)! :)

----------

**Login to one of your Swarm managers via SSH and the private SSH Key you created initially (user: centos)**

----------


Swarm manual deployment
-----
Create the Swarm **Network** configuration:

	docker network create --driver overlay --subnet 10.0.100.0/24 frontend

	docker network create --driver overlay --subnet 10.0.200.0/24 backend


----------


Create the Swarm **Services**:

	docker service create --name app_db --network backend postgres:9.4

	docker service create --name app_redis --network backend redis:alpine

	docker service create --name app_vote --network frontend,backend --publish 5000:80 vralex/vote

	docker service create --name app_worker --network backend vralex/worker

	docker service create --name app_result --network frontend,backend --publish 5001:80 --publish 5858:5858 vralex/result

The app will be running at [http://YOUR_EC2_IP:5000](http://YOUR_EC2_IP:5000), and the results will be at [http://YOUR_EC2_IP:5001](http://YOUR_EC2_IP:5001).

You can use any of the public IPs from your Swarm cluster (the ones that start with 52.x.y.z).

----

**THE FOLLOWING STEPS ARE OPTIONAL AND YOU NEED TO DELETE THE SERVICES AND NETWORKS CREATED**
----

Docker experimental Distributed Application Bundle (DAB) deployment
-----

First, delete the services and networks created previously:

	docker service rm app_db app_redis app_vote app_worker app_result

	docker network rm frontend backend


The docker-compose.yml file from this directory is set up so a bundle can be easily created from it.

This was tested in Docker 1.12.1 and may not work in future version of Docker as it is an experimental feature.

Clone this repository on a Swarm manager node (download and unzip):

> curl -OL https://github.com/alexvranceanu/docker-meetup-swarm/archive/master.zip
> sudo yum -y install unzip
> unzip master.zip
> cd docker-meetup-swarm-master/swarm-example-voting-app

Run in this directory:

	docker-compose pull

    docker-compose bundle

    docker deploy --file swarmexamplevotingapp.dab app


Expose the application ports:

    docker service update --publish-add 5000:80 app_vote

    docker service update --publish-add 5001:80 app_result

The Voting application will be running at [http://YOUR_EC2_IP:5000](http://YOUR_EC2_IP:5000), and the results of the vote will be at [http://YOUR_EC2_IP:5001](http://YOUR_EC2_IP:5001).

You can use any of the public IPs from your Swarm cluster (the ones that start with 52.x.y.z).