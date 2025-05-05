[![Version](https://img.shields.io/badge/goversion-1.18.x-blue.svg)](https://golang.org)
[![License](http://img.shields.io/badge/license-mit-blue.svg?style=flat-square)](https://raw.githubusercontent.com/tsawler/goblender/master/LICENSE)

# Working with Microservices in Go

Link to the course https://www.udemy.com/course/working-with-microservices-in-go/

This is the source code for the Udemy course **Working with Microservices and Go**. This project
consists of a number of loosely coupled microservices, all written in Go:

- broker-service: an optional single entry point to connect to all services from one place (accepts JSON;
sends JSON, makes calls via gRPC, and pushes to RabbitMQ)
- authentication-service: authenticates users against a Postgres database (accepts JSON)
- logger-service: logs important events to a MongoDB database (accepts RPC, gRPC, and JSON)
- queue-listener-service: consumes messages from amqp (RabbitMQ) and initiates actions based on payload (sends via RPC)
- mail-service: sends email (accepts JSON)

All services (except the broker) register their access urls with etcd, and renew their leases automatically.
This allows us to implement a simple service discovery system, where all service URLs are accessible with
"service maps" in the Config type used to share application configuration in the broker service.

In addition to the microservices, the included `docker-compose.yml` at the root level of the project
starts the following services:

- Postgresql - used by the authentication service to store user accounts
- MongoDB - used by the logger service to save logs from all services
- etcd - used for service discovery
- mailhog - used as a fake mail server to work with the mail service

## Running the project
From the root level of the project, execute this command (this assumes that you have 
[GNU make](https://www.gnu.org/software/make/) and a recent version
of [Docker](https://www.docker.com/products/docker-desktop) installed on your machine):

~~~
make up_build 
~~~

If the code has not changed, subsequent runs can just be `make up`.

Then start the front end:

~~~
make start
~~~

Hit the front end with your web browser at `http://localhost:80`. You can also access a web 
front end to the logger service by going to `http://localhost:8082` (or whatever port you
specify in the `docker-compose.yml file`).

To stop everything:

~~~
make stop
make down
~~~

While working on code, you can rebuild just the service you are working on by
executing

`make auth`

Where `auth` is one of the services:

- auth
- broker
- logger
- listener
- mail


All make commands:

~~~
tcs@Grendel go-microservices % make help
 Choose a command:
  up               starts all containers in the background without forcing build
  down             stop docker compose
  build_auth       builds the authentication binary as a linux executable
  build_logger     builds the logger binary as a linux executable
  build_broker     builds the broker binary as a linux executable
  build_listener   builds the listener binary as a linux executable
  build_mail       builds the mail binary as a linux executable
  up_build         stops docker-compose (if running), builds all projects and starts docker compose
  auth             stops authentication-service, removes docker image, builds service, and starts it
  broker           stops broker-service, removes docker image, builds service, and starts it
  logger           stops logger-service, removes docker image, builds service, and starts it
  mail             stops mail-service, removes docker image, builds service, and starts it
  listener         stops listener-service, removes docker image, builds service, and starts it
  start            starts the front end
  stop             stop the front end
  test             runs all tests
  clean            runs go clean and deletes binaries
  help             displays help
~~~

## Databases
PostgreSQL: The `authentication-service/sql/users.sql` file contains the schema and initial data for the `users` table, including sequence creation and a default admin user.

Test connections PostgreSQL & Mongodb

~~~
jdbc:postgresql://localhost:5433/users
mongodb://admin:password@localhost:27017
~~~

# Project Launch

Build and Start Services with Docker Compose.

- From the `project` directory run `make up_build` — builds all services and starts Docker containers.
- Then run `make start` — builds and starts the frontend application locally.

Useful Make Commands
- `make up` — Start Docker containers without rebuilding.
- `make down` — Stop Docker containers.
- `make build_front` — Build only the frontend.
- `make build_broker` — Build only the broker service.
- `make build_auth` — Build only the authentication service.
- `make build_logger` — Build only the logger service.
- `make build_mail` — Build only the mailer service.
- `make build_listener` — Build only the listener service.
- `make start` — Start the frontend app manually.
- `make stop` — Stop the frontend app manually.

Deployment
1) Install Minikube
- Follow step 1 “Installation” at https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download
2) Install kubectl
- Download the kubectl binary from https://kubernetes.io/docs/tasks/tools/
- Make the kubectl binary executable, run `chmod +x ./kubectl`
- Move it to a directory in your PATH and assign proper ownership `sudo mv ./kubectl /usr/local/bin/kubectl`, and run `sudo chown root: /usr/local/bin/kubectl`
- Verify the installation `kubectl version --client`
3) Initialize a Local Kubernetes Cluster
- From the [project] directory (or any working directory), initialize a local Kubernetes cluster with 2 nodes by running: `minikube start --nodes=2` . This command starts a multi-node cluster locally using Minikube, which simulates a real Kubernetes environment on your machine.
- To verify that Minikube is running properly, check the status with: `minikube status`
4) Launch Kubernetes Dashboard
- To start the K8s dashboard UI, run `minikube dashboard`