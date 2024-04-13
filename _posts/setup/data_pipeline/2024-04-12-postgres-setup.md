---
title: Setting up a postgres docker container
date: 2024-04-12 13:00:00 -0500
categories: [setup, data_pipeline]
tags: [setup, docker, dbt, postgres]
img_path: /assets/img/setup/data_pipeline/2024-04-12/
---

### Prereqs:
- [Docker installed](https://www.docker.com/products/docker-desktop/)
- [Docker Hub account created](https://www.docker.com/products/docker-hub/)


Below is the initial structure that's been created to house the data pipeline.

![initial project structure](initial-structure.png){: width: 20% height: 10% }

That is a data_pipelines/ folder with an nws/ and pgdata/ folder inside.
The nws folder is intended to house Python code that will collect data
from the [National Weather Service
api](https://www.weather.gov/documentation/services-web-api) later down the
road. The pgdata folder will be used to store data from the postgres container.
This is needed because containers do not persist data after they are stopped,
so we'll lose all the data we put into our database unless we make a volume for
it.

Note:
In a nutshell, "volume" is what Docker calls it when you link a folder
between your local computer and a container. So anything you do to that folder
will be reflected in both the container and your local machine.

First, we'll create a network that this data pipeline will use. This way when
we pull in more containers, they can all communicate with each other over this
network.

```bash
docker network create networkdp
```

The above command creates a network called networkdp. You can confirm it has
been created via

```bash
docker network ls
```

Navigate to the pgdata folder and run the following command:
```bash
docker run -d \
--name postgres \
-e POSTGRES_PASSWORD=postgres \
-e POSTGRES_DB=raw \
--network networkdp \
-v $(pwd):/var/lib/postgres/data \
postgres
```

docker run
The docker run command is used to create a new container.

-d
Starts the container in detached mode. This way we keep our terminal rather than
jumping into the container.

--name postgres
This is how we name the container. Here we've named it postgres.

-e POSTGRES_PASSWORD=postgres
The -e option sets up environment variables in the container. Postgres requires
that there at least be a password variable set up, so we do that here. The
"postgres" password is fine for now, but there's a better way to handle
sensitive data altogether that will be covered later in these posts.

-e POSTGRES_DB=raw
This creates a database called raw. We're going to use this to dump our data
into once the pipeline gets flowing.

--network networkdp
This lets docker know that this container should run on the network we created

-v /path/to/data_pipelines/pgdata:/var/lib/postgres/data
This establishes a volume mount between a local directory and directory in the
container i.e. -v "local directory":"container directory". Since we navigated
to the pgdata folder before running the command the $(pwd) just punched in the
absolute path to the pgdata folder. It's not necessary to do that however, you
could also just write the path in yourself from whatever folder you're in.

The /var/lib/postgres/data/ folder is the folder inside the container where
postgres puts data. You can read up on that on the [docker postgres docs](https://github.com/docker-library/docs/blob/master/postgres/README.md#pgdata)

postgres
Finally, this is the name of the [image](https://hub.docker.com/_/postgres).
This pulls the latest version of postgres though you could use one of the tags
listed on the prior link to get a specific version. For example, you could do
something like postgres:14 to get version 14.


If you run
```bash
docker ps
```
You should see the postgres container running. And that's that. We now have a
container running postgresql that we can store data in later.


Extra
If you want to connect to the container and explore the databases there, you
can do so via psql. Run the following

```bash
docker exec -it postgres psql -U postgres
```

This will drop you in the container as the user postgres (that's what the -U is
for. If you omit it, you'll get an error because it'll default to root).

Use \\? to read through the list of commands.

Alternatively, you can also hook up a gui program to it (DBeaver, DBVis,
PGAdmin4, etc) I believe through localhost:5432.











