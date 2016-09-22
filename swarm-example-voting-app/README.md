Swarm Mode Demo with Instavote
=========


Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

Getting started in AWS
---------------

Setup a Docker Swarm cluster in AWS with your own template or use the Cloud Formation Swarm Template from this repository:
https://github.com/alexvranceanu/docker/tree/master/swarm-aws


----------


**Login to one of your Swarm managers via SSH.**


----------


Swarm manual deployment
-----
Create the Swarm **Network** configuration:

	docker network create --driver overlay --subnet 10.0.100.0/24 frontend

	docker network create --driver overlay --subnet 10.0.200.0/24 backend


----------


Create the Swarm **Services**:
		docker service create --name app-db --network backend postgres:9.4

	docker service create --name app_redis --network frontend,backend redis:alpine

	docker service create --name app_vote --network frontend --publish 5000:80 vralex/vote

	docker service create --name app_worker --network backend vralex/worker

	docker service create --name app_result --network frontend,backend --publish 5001:80 --publish 5858:5858 vralex/result

The app will be running at [http://YOUR_EC2_IP:5000](http://YOUR_EC2_IP:5000), and the results will be at [http://YOUR_EC2_IP:5001](http://YOUR_EC2_IP:5001).

You can use any of the public IPs from your Swarm cluster.


Docker experimental Distributed Application Bundle (DAB) deployment
-----
The docker-compose.yml file from this directory is set up so a bundle can be easily created from it.

Clone this repository on a Swarm manager node:

> curl -O https://github.com/alexvranceanu/docker/archive/master.zip

Run in this directory (`cd swarm-example-voting-app`):


	docker-compose pull

    docker-compose bundle

    docker deploy --file swarmexamplevotingapp.dab app


Expose the application ports:

    docker service update --publish-add 5000:80 app_vote

    docker service update --publish-add 5001:80 app_result

The app will be running at [http://YOUR_EC2_IP:5000](http://YOUR_EC2_IP:5000), and the results will be at [http://YOUR_EC2_IP:5001](http://YOUR_EC2_IP:5001).

You can use any of the public IPs from your Swarm cluster.


Getting started on your laptop
---------------

Download [Docker for Mac or Windows](https://www.docker.com).

Setup your Swarm cluster with docker-machine.

You might need to manually install the experimental Docker binary to use Distributed Application Bundles.

The steps from above also apply in this scenario.

The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).




