# Docker for JavaScript Developers

## Intro

### About Shane

👋 Hi! I'm Shane. I'm a Senior Software Engineer & Instructor at The Home Depot. As a part of the OrangeMethod team, I build applications and supporting curriculum on a wide range of topics. Currently, I am helping run our 15 week fullstack bootcamp, and preparing a series of workshops on React, Go, and Node. 



### Purpose

The subject of this talk is Docker for JavaScript developers. 



If you're here, my best guess is that you fall into one of the following categories. 

- You think Docker is neat, and want to start working with it
- You know you need to learn more about Docker, and find it tough to get started
- You know you _should_ care, but don't, and have nothing better to do



The goal for this talk is to really dig into the development side of Docker and showcase some of the benefits. 



#### What this talk _is not_ about

Before moving into the good stuff, I first want to discuss what this talk is _not_ about. 

We will not be discussing: 

- Using Docker for Deployment/Production
- Kubernetes, Swarm

In addition, this will not be another tutorial using Redis. (Redis is great btw!)



#### What this talk _is_ about

We are going to look at a real world example of a Node.js API with a Postgres database. 

The goal is to demonstrate

- Why you should consider using Docker for development
- How to use Docker effectively in Development



## Containers?

So it seems like containers are all the rage these days. I mean, it makes sense, store your application and dependencies in a container and ship it :shipit: anywhere you want. One of the key benefits, is that your application will run without the classic fear of your application performing differently in another environment. 



![works-on-my-machine](./images/works-on-my-machine.jpeg)



Of course, there are many other benefits, such as load-balancing, resiliency, etc. However, that typically doesn't come into play when you're involved in day-to-day coding. 



### My experience

I had a moment of enligtenment when I saw an image of shipping containers as analogy for Docker.

![expectation](./images/expectation.jpg)



Unfortunately, my experience working on teams using Docker was much more like this:



![experience](./images/disaster.jpg)

As a consultant (prior to The Home Depot), I would find myself on a new project every few months. I had several **painful** experiences using Docker for development purposes. 

It seemed to be that daily I would run into problems with things like:

- Error messages on startup, and no explanation of how critical they might be
- Inability to run a debugger, or even get a simple `console.log` to work
- Data vanishing without a trace



Sure, I didn't have to spend an entire day setting up my environment, and yes down the road it *should* make our lives easier. For me however, those weren't big enough selling points. 

![throw-computer](./images/throw-computer.gif)



I ended up spending a stretch of time absolutely hating Docker. Mainly because I never took the time to understand the benefits in development. It always seemed to make my daily work miserable, only to solve some future problem that could be solved in other ways. 



#### What Changed?

So, I'm sure you're asking yourself something like: "okay, Shane, what changed your mind?"

And while the answer is actually pretty simple, It is not the one I want to give. What changed my mind was going to the docs and learning about Docker from the ground up (along with the help from a couple of excellent tutorials). 

Yes, it took me a while, but it wasn't until I gave Docker an open-minded chance, that I started to see the benefits. 



![enlightenment](./images/enlightenment.gif)



## Getting into the Code

As stated in the intro, we are going to use a small API built with Node.js & Express, that interacts with a Postgres Database. 

The repo can be found [here](https://github.com/shanebarringer/docker-music). We'll start with a fairly standard setup, which can be found on master:



### Scripts and Databases

Looking at the `package.json` file, you'll see that we have a couple of additional scripts for database setup. 

```json
"scripts": {
	"start": "NODE_ENV=production node app.js",
	"dev": "NODE_ENV=development nodemon app.js",
	"db:migrate": "./node_modules/knex/bin/cli.js migrate:latest --env development",
    "db:seed": "./node_modules/knex/bin/cli.js seed:run --env development"
},
```

I'm of the opinion that npm scripts are a great way to automate repetitive tasks for your project. Here, you will see two scripts related to database setup, `db:migrate` and `db:seed`. Both of these scripts make use of the [knex.js](https://knexjs.org/) library. Knex is a SQL query builder for Node.js. _note: it is not a full-featured ORM_. 



You may notice that the db commands are looking into the `node_modules` directory.  `"db:migrate": "./node_modules/knex/bin/cli.js migrate:latest --env development"`



This is to avoid installing yet another global package. _Yes, we could use `npx`, which is great, but not ideal for these types of tasks_. So we are running into our first problem. Any new developer on the team, will need to globally install `knex`, or know how to run the project-specific version. 



While not the biggest deal, we are now relying on a device-specific dependency, which could end up being problematic in the future. In addition, anyone new to the project will need to setup Postgres on their local machine. Again, while not a big deal, but it can be a time-consuming affair. _note: I'm not recommending that you avoid a local Postgres install_ 



As you have all experienced, project setup can definitely eat away at your initial productivity. Especially when considering that no two projects are exactly the same.



#### A note on the Knexfile

`knexfile.js` is where the db connection info is stored. In this particular case, we have 3 configurations. This demo will only make use of the `development` config.

```javascript
  development: {
    client: "postgresql",
    debug: false,
    connection: {
      host: "127.0.0.1",
      database: `${db}_dev`,
      charset: "utf8"
    },
    migrations: {
      directory: path.join("db", "migrations")
    },
    seeds: {
      directory: path.join("db", "seeds")
    }
  },
```



The items of note are inside the `connection` object. 

- `host` - The host URL

-  `database` - Database name, in this case `docker-music_dev`



We have not provided any type of security at this point. Obviously, that's a good idea, and we'll take care of it when setting up Docker. 



### Running the App

#### DB Setup

Let's assume that you've performed the necessary setup tasks. The next steps would be:

- Create a local database
- Perform migrations
- Seed the data




```shell 
$ createdb docker-music_dev
$ npm run db:migrate
$ npm run db:seed
```



#### Inspecting Data

Upon success, we can log in to our local Postgres db and inspect the data. 

```shell
$ psql docker-music_dev
```



Once in the db, we'll check our Artists table to ensure the data has been seeded.

```sql
docker-music_dev=# SELECT id, name FROM artists;

 id |         name
----+-----------------------
  1 | Sufjan Stevens
  2 | Explosions in The Sky
  3 | Fleet Foxes
  4 | Hammock
  5 | Run the Jewels
  6 | Lord Huron
  7 | Charles Bradley
  8 | Local Natives
  9 | A Tribe Called Quest
 10 | Ryan Adams

docker-music_dev-# \q
```



#### Starting the Server


We can start the server with `npm run dev`. We'll use Postman (or the browser) to query data. 

The Postman collection can be found here: [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/4937978ee04fcc43af64)


Running our first query should result in the following output

```json
[
    {
        "id": 1,
        "name": "Sufjan Stevens",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 2,
        "name": "Explosions in The Sky",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 3,
        "name": "Fleet Foxes",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 4,
        "name": "Hammock",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 5,
        "name": "Run the Jewels",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 6,
        "name": "Lord Huron",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 7,
        "name": "Charles Bradley",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 8,
        "name": "Local Natives",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 9,
        "name": "A Tribe Called Quest",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    },
    {
        "id": 10,
        "name": "Ryan Adams",
        "genre": null,
        "created_at": "2018-10-14T17:09:55.807Z",
        "updated_at": "2018-10-14T17:09:55.807Z"
    }
]
```

Great! So, it looks like our application is working. Now, let's take a look at how we could make this whole setup process easier with Docker. 



## Docker Setup

Next, we're going to checkout the `docker-setup` branch. This branch has the entire configuration in place. We are first going to run the app, to ensure everything works as expected. Then we'll step through the additional files and break them down line-by-line.



### Startup

First, we'll perform some cleanup, from our previous example. 

```shell
$ dropdb docker-music_dev
$ npm uninstall knex -g
$ rm -rf node_modules
```

Here, we are deleting the database, uninstalling knex (if applicable), and removing the `node_modules` directory. This will allow us to imagine we are approaching the project for the first time. 



At this point, we just need to ensure that Docker is running on our local machine. Refer to [Docker for Mac](https://docs.docker.com/docker-for-mac/install/) or [Docker for Windows](https://docs.docker.com/docker-for-windows/install/) for installation instructions. Once installed, we just need to start docker and wait for the little green light.



We'll also need to create a `.env` file in the root directory, which holds our relevant environment variables. 

```shell
$ touch .env
```



In the `.env` file we'll add the following:

```
DB_PASSWORD=changeme
DB_USER=postgres
DB_NAME=docker_music_dev
```



From here, we can run: 

```shell
$ docker-compose up --build
```

This command will read our `docker-compose.yml` file, follow the instructions, and setup our containers. The `build` flag will build/rebuild our containers from the Docker images. 



The first build will take slightly longer as it needs to pull in the appropriate images and install our dependencies in the container. After a few minutes, we should be able to run our collection in Postman and get the expected output. 



![celebrate](./images/celebrate.gif)



**Now, imagine that you are assigned to a new project, on day 1, you clone down the project, run one command for setup, and start coding.** This, to me, is one of the really nice features of development with Docker. Anyone can get up and running quickly, without spending hours setting up a local environment. 



With this particular setup, we are creating a container for our Node.js/Express API, as well as a separate container for our Postgres database. They are each running in isolated environments and communicating with each other on an as-needed basis.



### Exploring the Dockerfile

To shut down our containers, we can just press `control` + `c` . This command will attempt to shut down the containers gracefully. If the action takes more than 10 seconds, a kill command will be sent. 



Now, we can step through the `Dockerfile` to understand what's happening. 

```dockerfile
FROM node:latest
WORKDIR /usr/app

COPY package*.json ./

RUN npm install
RUN npm install knex -g

COPY . .

EXPOSE 3001

ENTRYPOINT ["sh", "./docker-entrypoint.sh"]
```

As you can see, this is a relatively small file, yet powerful file. Obviously, these are the instructions that Docker uses to build images and create containers. However, an often overlooked benefit is that this also serves as instructions to your entire development team. Any team member, can open up the `Dockerfile` and understand every action that is being taken. This implies, that the `Dockerfile` can also serve as a source of truth for your containers.



#### FROM

`FROM`  installs the base image for our container. You can consider this like installing a base Operating System. In addition, it will pull in a basic (and relevant) enviornment configuration. 

In our case, we are pulling in the latest Node image. You'll see in the [Official Node Docker Repo](https://hub.docker.com/_/node/) that there are various flavors and versions of Node that we can install. You'll also notice that we can also lock in a specific Node version, if needed. 



Related the version of Node that we have selected. The Documentation states that:

>  This is the defacto image. If you are unsure about what your needs are, you probably want to use this one. It is designed to be used both as a throw away container (mount your source code and start the container to start your app), as well as the base to build other images off of.



Sounds good enough for our use case. As the project grows, it is very easy to swap the Node base image if the need arises. _This is also a great way to see if newer versions will cause regression bugs and/or breaking changes, as you can spin up a new container and easily throw it away_.



#### WORKDIR

`WORKDIR` sets the directory where our the rest of our work should be performed. A typical convention is to use `/usr/app` as it will hold our application code. Unless a new _working directory_ is specified, the remainder of our _actionable commands_ will be performed in `/usr/app`. This can be roughly equated to  running the `cd` command.



#### COPY

`COPY` will, as expected, copy files from our host machine to the container. 

The syntax is: `COPY <from-source-file-on-local-machine> <destination-in-container>` 



In this first step, we are only copying our `package.json` and `package-lock.json` files. Why? you might ask? You'll see that our next step is to run `npm install`, by copying our `package.json` related files, we are allowing the `npm install` command to look at our cached dependencies. The benefit is that our containers will restart more quickly as they won't need to perform an entire install every time. 

Later, we will also run `COPY . .` - this will copy all of our source code to the container. 



#### RUN

`RUN` will execute specific commands on our image. In our case, we want to run an `npm install` to ensure that all of our dependencies are running in the container. 



We also install `knex` globally, as it makes our development lives much easier. 



#### EXPOSE

`EXPOSE` tells the Docker container what ports to listen to at runtime. In our case, we are exposing port `3001` 



It's also worth noting that the documentation on `EXPOSE` states: 

> The `EXPOSE` instruction does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published. To actually publish the port when running the container, use the `-p` flag...and map one or more ports...



This essentially means that we'll need to ensure that the appropriate ports are exposed when starting the container. This can be accomplished with the `-p` flag, or in the `docker-compose.yml` file.



#### ENTRYPOINT

`ENTRYPOINT` allows you to run an executable command or file. In our case, we are specifiying that we want to run the `docker-entrypoint` shell script. 



An alternative approach to entrypoint is `CMD` which executes a command when the container starts. An example would be `CMD npm run dev`. 

We are choosing to use `ENTRYPOINT` based on convention. 



## docker-entrypoint

The `docker-entrypoint.sh` file is executed when our container starts. Let's take a look at the code.

```bash
#!/bin/bash

echo "waiting for db connection"
sleep 5

echo "resetting the database"
knex migrate:rollback
knex migrate:latest --env development
knex seed:run --env development

echo "starting the server"
npm run dev
```

This is a fairly simple script, however, let's step through it. 

- `#!/bin/bash` signifies that this should be run as a bash script. 
- We'll then `echo` that we are waiting for database connections. 
- `sleep 5` is a simple way to ensure that our database will be up and accepting connections. <br/> _note: there are definitely more elegant ways to accomplish this task_
- Next we will echo that the database is being reset
- `knex migrate:rollback` will rollback our migration
- `knex migrate:latest` will run our migrations
- `knex seed:run` will seed the database
- Finally, we will `echo` that we are spinning up the server and call the familiar `npm run dev` command



This script will run automatically every time we spin up our container. As it stands right now, we are going to _destroy_ and _create_ the same data each time. In the next iteration, we'll create some logic around this, so that our data is not destroyed each time. 



## docker-compose

The next file to consider is `docker-compose.yml`. 



First, it's worth noting that `docker-compose` is not entirely necessary, in fact, a `Dockerfile` is also not needed for Docker. We could actually create a single command that would spin up our container. ie: `docker run --rm -v $(pwd):/usr/app -w /usr/app -p 3001:3001 node:latest npm run dev` However, this can become quickly un-manageble. 



We can also run our container with: `docker container start docker-music_api_1` <br/>This will start up our API container. We can then perform a query with Postman to `http:localhost:3001/` which should provide a welcome message. 

Neat! our API is running!

Unfortunately, upon querying any routes that require data will result in an error message stating that our Postgres database is not available. 

_note: we can get the name of our container by running `docker container ls -a`_



This is where `docker-compose` comes in. According to the [Docker Compose Overview](https://docs.docker.com/compose/overview/):

> Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.



This convenience is definitely worth exploring. Let's again, step through the code line-by-line. 

```yaml
version: "3.1"
services:
  api:
    build: .
    volumes:
      - .:/usr/app
      - /usr/app/node_modules
    ports:
      - "3001:3001"
    networks:
      - dm_network
    depends_on:
      - postgres
    environment:
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_USER=${DB_USER}
      - DB_NAME=${DB_NAME}

  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    expose:
      - "5432"
    networks:
      - dm_network

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
    networks:
      - dm_network

volumes:
  ? postgres-data

networks:
  dm_network:
    driver: bridge

```

Okay, so there is a **lot** going on here. From the 10,000 foot view, we are setting up 3 separate containers, and linking them together. 

- `api` - Our API
- `postgres` - Our instance of Postgres
- `adminer` - A tool for interacting with the database

We also have `volumes` and `networks` which we'll explore later. 



### Breaking down Docker Compose

The first two lines define the version of `docker-compose` that we want to use, and what `services` we intend to work with.



#### api

As you can guess, `api` defines all relevant data from our application. This allows us to define all the relevant info that would normally be passed as command line arguments. 

- `build`  - specifies the configuration to use at build time, we are saying to use the files in our current directory.
- `volumes` - allows us to share files, data, etc located on the host. <br/>With `.:/usr/app` we are sharing data from our _current directory_ with `usr/app` in the container.<br/>We are also sharing node_modules<br/>_note: volumes, in-and-of themselves could be an entire talk_
- `ports` - what we use to specifiy (to the container) what ports should be exposed. The syntax is `<host-port>:<container-port>` <br/>_While these arguments can differ, I'd advise starting with the same port._
- `networks` - defines what network we want our container to run on. We are stating that our container should run on a network named `dm-music` <br/>_As with Volumes, Networks deserve an entire talk._
- `depends_on` - notes that our container relies on the`postgres` container succesfully starting. <br/>Meaning, our postgres container will start first
- `environment` - pulls in environment variables set in the `.env` file



### postgres

You probably noticed, we haven't really had to do anything with postgres at this point. This is because we're pulling in the [Official Postgres Docker Image](https://hub.docker.com/_/postgres/). We'll need to provide some basic configuration, and then we will be good to go. Let's step through the configuration. 

- `image` - pulls in the latest postgres image
- `environment` - supplies relevant environment variables for our database, including:
  - user
  - password
  - name
- `volumes` - creates a shared volume between the host and container. Allowing us to easily persist data across restarts.<br/>
- `ports` - expose the standard postgres port of `5432` on the host and container
- `network` - puts our container on the shared network. Allowing communication with the `api` container



#### Updating the knexfile

Before moving on, it's worth taking a look at the updated knexfile configuration. In the `development` object, have made a few changes. 

```javascript
development: {
  // ...
    connection: {
      host: "postgres",
      database: process.env.DB_NAME,
      password: process.env.DB_PASSWORD,
      user: process.env.DB_USER,
      charset: "utf8"
    },
    pool: { min: 0, max: 10 },
   // ...
  },
```



- `host` - the host is now our `postgres` container defined in `docker-compose.yml`<br/>_this only works, because we've created a shared network_
- `database` - use the `DB_NAME` property found in the `.env` file
- `password`- use the `DB_PASSWORD` property found in the `.env` file
- `user`- use the `DB_USER` property found in the `.env` file
- `pool` - define a connection pool <br/>_by default this is set to `min: 2` and `max: 10`_



We've also added a `require` statement in `app.js` that loads the [dotenv js package](https://www.npmjs.com/package/dotenv)

```javascript
const express = require("express");
const logger = require("morgan");
const bodyParser = require("body-parser");
require("dotenv").config();
// ...
```



### adminer

`adminer` is a nice addition to Postgres, as it gives us a UI to interact with. The `adminer` configuration is fairly straight forward. In fact, we are using the exact code specified in the Docker Postgres repo. 

- `image` - pull in the `adminer` image
- `restart` - always restart the container on build
- `ports` - expose `8080` on the host and container
- `networks` - place container on the `dm-network` 



We can access the interface by navigating to [localhost:8080](http://localhost:8080) and entering the following credentials: 

- System - PostgreSQL
- Server - postgres (this connecting to the `postgres` container on `dm-network`)
- Username - postgres
- Password - changeme
- Database - docker_music_dev



Here, we can perform all the standard CRUD actions, add new tables, and more. 



### volumes

We have moved out of `service` object at this point, and have a top-level `volume` object. This is due to the fact that we want to share our `postgres-data` volume across multiple services. 

The syntax may look slightly strange. This is due to the fact that our volume is already established. An alternative syntax is: 

```yaml
volumes:
  postgres-data:
```



### networks

Also, at the root (top) level, we need to specify our shared network. Here we will explicitly specify that the `dm-network` will use Docker's default `bridge` driver. 

While there are several network options provided by Docker, `bridge` is by far the most common.  



If we run `docker network ls` we can see a list of all networks

```shell
NETWORK ID          NAME                      DRIVER              SCOPE
1f1b75b0291c        bridge                    bridge              local
64167645bdb4        docker-music_dm_network   bridge              local
c6540f56f965        host                      host                local
526520305df5        none                      null                local
```



Here, you'll see that the `docker-music_dm_network` is sharing the `bridge` driver. 

To learn more about Docker's network drivers, check out [The Docker Networking Guides](https://docs.docker.com/network/#network-drivers)



## .dockerignore

Last but not least, we've added several items to our `.dockerignore` file. 

```
.git
*Dockerfile*
*docker-compose*
node_modules
```

These files do not need to be copied over to the container, by adding them to `.dockerignore` we can improve build times and ensure that our container is slim. 



## Docker-Updates

This branch is actually, slightly misleading. We're not going to actually make changes to our `Dockerfile` or `docker-compose.yml`. Instead, we are going to create a couple of helpful scripts for setting up and tearing down the database. 

The code can be found on the `docker-updates` branch. 



### Changes

Here, we are going to create two scripts in the `db` directory 

```shell
$ touch db/setup.sh
$ touch db/reset.sh
```



Next, we'll remove most of the code from our `docker-entrypoint.sh` file. 

```bash
#!/bin/bash

echo "starting the server"
npm run dev
```



In `db/setup.sh` we'll add the setup scripts

```bash
#!/bin/bash

echo "setting up the database"
knex migrate:latest --env development
knex seed:run --env development
```



In `db/reset.sh` we'll add a couple of extra lines

```bash
#!/bin/bash

echo "resetting the database"
knex migrate:rollback
knex migrate:latest --env development
knex seed:run --env development
```



Finally, let's add a couple of simple scripts in our `package.json` file

```json
"scripts": {
	"start": "NODE_ENV=production node app.js",
	"dev": "NODE_ENV=development nodemon app.js",
	"db:setup" : "sh ./db/setup.sh",
	"db:reset" : "sh ./db/reset.sh"
},
```

Now, anytime we want to setup, or reset the database, we can make use of the new scripts. 



### Detached Mode

Before we move on, let's see if our application is still performing as expected. 

```shell
$ docker-compose up --build
```

_note: you may notice the container takes longer to build. This is because our `package.json` file has changed, and this layer must be re-built_



Running a few calls on our endpoints, will reveal the expected behavior. 



Until now, we have started up our containers and let it kind-of over-take the terminal. Obviously, we can open a second terminal window to perform various tasks. However, Docker does offer us the `-d` flag which means we can run our containers in `detached` mode. 



What does that look like? 

```shell
$ docker-compose up --build -d
```

Upon success, you'll be dropped back into your normal shell. This could cause us to think that something went wrong. Fortunately, that's not the case. Instead, our containers are now running detached from the terminal. We can verify the containers are running with `docker container ls`. 



#### exec

Now, we can execute commands against our containers. Let's try resetting the database

```shell
$ docker-compose exec api npm run db:reset
```

You'll see output indicating that the command was successful. What is happening here, is that we are telling docker-compose to **execute** a command. This command can be broken down into 3 separate parts

- `exec` - execute a command
- `api` - the container to operate on
- `npm run db:reset` - the command we wish to execute



Alternatively, we can use the `docker` command. The major difference is that we'll have to provide the full name of the container. 

```shell
$ docker exec docker-music_api_1 npm run db:reset
```



The `exec` command will also allow us to log in to the container on an as-needed basis. 

```shell
$ docker-compose exec api bash
```

This command will execute the `bash` command, dropping us into a `bash` session inside of the container. Here we can interact with the application as we would normally expect. Once we're ready to exit, we can simply type `exit` to be brought back to the shell session on our host machine.



It's worth nothing that an alternative (and fairly common) syntax for this same command is: 

```shell
$ docker exec -it docker-music_api_1 bash
```

Here, we are passing an additional flag of `-it` to `exec`. This flag ensures that our container will accept input and enables `TTY`. 



There are many additional commands that can be executed against your containers, it will definitely take some time to explore them. 



## Summary

We've covered some basic development use cases for working with Docker. My hope is that you'll find this information compelling enough to explore Docker on your own. 



If you want to reach me, you can find me on twitter @shanebarringer. I'm not very good at tweeting, but I'll try to be better. Also, we are hiring Software Engineers of all skill levels at The Home Depot. Feel free to reach out if you'd be interested in learning more about THD Technology. 