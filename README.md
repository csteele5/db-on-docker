# MySQL & PostgreSQL on Docker Tutorial

## Overview

Source video: [MySQL & PostgreSQL with Docker in development - Episode #8](https://www.youtube.com/watch?v=q5J3rtAGGNU)

Chandra Shettigar has a series of videos that I like. This is mainly a MySQL tutorial.

Good docker-compose exercise

## Technical details

This is initially edited using Visual Studio Code

## Process

Start with a blank folder
Create this file
Build the docker-compose.yml file
Run `docker-compose up`

Once container is up, open a new terminal to view status
`docker-compose ps`

Now test access using MySQL client

- create a table called posts (id, title)
- add a couple of records

Shut down docker container

**Access DB through second service**

Update docker-compose to add client section
Note that client container will exit after spin up - this is ok.
Verify the database is up and running via MySQL Workbench

Now go to terminal window and connect to database via new service we created:

- `docker-compose run --rm client`
- the --rm option ensure the container will be deleted on exiting the mysql prompt

This opens a mysql prompt.
run `show tables;` and select \* from posts;

**Access DB directly in primary database container**

`docker-compose exec mysql-dev mysql -uroot -ppassword blogapp`

### Adding multiple containers with different db versions

Why? This prevents version conflicts and installation challenges. Start doing this type of thing instead of installing the dev environment directly on the host OS!!!

**Update docker-compose**

- The client container is no longer necessary, but I'm not commenting it out
  Add a new service for 'legacy' version. This spins up on a different port
  Connect via workbench or command line
- `docker-compose exec mysql-legacy mysql -uroot -ppassword legacyapp`
- no tables yet, so just run show databases;

### How to modify MySQL configuration

Normally, this would be modified in the /etc/mysql/conf.d folder
We could either build a new mysql image with a new config file, or mount a volume from the host and use that to start up the docker image.

**Create a custom cnf file**
Create my.cnf - paste in some custom config stuff
Now add volumes section to yml file

Next, delete the containers and start all over again

- `docker-compose rm` (unless docker-compose down was run)

Run the containers again, and this time the custom cnf should be used.
NOTE: the data created previously has all been deleted!

### How to make data persistent on the host

This will keep us from losing the data during development

Create data directly in project folder
Delete the containers once again
Now we need to mount the data directly in the host to the mysql directly

- the documentation shows the default data directory in the image is /var/lib/mysql
- mount a volume in the yml, with read/write permissions
  - ./data:/var/lib/mysql:rw

Now spin up the container. We can target a specific service in the yml without commenting things out, so we don't unnecessarily run containers. Just identify the service in the up statement.

- `docker-compose up mysql-dev`
- recreate the posts table and entries
- spin down, delete container, spin back up and verify data persists

**If (when) you forget to indicate the database when opening mysql**
At mysql prompt, type `use <dbname>;`

VICTORY - Even if the container is deleted, the data will persist in the development environment.

## Handy web tool to access containers

From docker store, look up adminer (this is the image name)
Grab the service definition from the documentation
Note: I got an error on spin up, from mapping 8080:8080. I changed this port mapping to 3500:8080 and it worked.

Now, open the browser to localhost:3500 and open the database

- Set the server to the service name, not local host:port!!

**This is very cool and could be used for different databases.**

## PostgreSQL database setup

Next, add a new container running PostgreSQL and connect.

- Lookup postgres in the docker store to get the yml commands for setup [Reference](https://hub.docker.com/_/postgres)
- Add the environment variables for user, password, db

Run `docker-compose up`

Verify all images are running. Postgres does not have a port mapping, so cannot be accessed by a host client. It can be accessed by the other services in the same virtual network.

Open the admin web app and access postgres using the service name.
Create a table through the web interface.

Now, connect to the database through the terminal:
`docker-compose exec postgres-dev psql -U root -W blogapp`

`\d` shows items
