# Build your Python image

## Prerequisites[🔗](https://docs.docker.com/language/python/build-images//#prerequisites)

-   You understand basic [Docker concepts](https://docs.docker.com/get-started/overview/).
-   You’re familiar with the [Dockerfile format](https://docs.docker.com/build/building/packaging/#dockerfile).
-   You have [Docker Desktop](https://docs.docker.com/desktop/)] installed on your machine.

## Overview[🔗](https://docs.docker.com/language/python/build-images//#overview)

Now that you have a good overview of containers and the Docker platform, let’s take a look at building your first image. An image includes everything needed to run an application - the code or binary, runtime, dependencies, and any other file system objects required.

To complete this tutorial, you need the following:

-   Python version 3.8 or later. [Download Python](https://www.python.org/downloads/)
-   Docker running locally. Follow the instructions to [download and install Docker](https://docs.docker.com/desktop/)
-   An IDE or a text editor to edit files. We recommend using [Visual Studio Code](https://code.visualstudio.com/Download).

## Sample application[🔗](https://docs.docker.com/language/python/build-images//#sample-application)

The sample application uses the popular [Flask](https://flask.palletsprojects.com/) framework.

Create a directory on your local machine named `python-docker` and follow the steps below to activate a Python virtual environment, install Flask as a dependency, and create a Python code file.

```
$ cd /path/to/python-docker
$ python3 -m venv .venv
$ source .venv/bin/activate
(.venv) $ python3 -m pip install Flask
(.venv) $ python3 -m pip freeze > requirements.txt
(.venv) $ touch app.py
```

Add code to handle simple web requests. Open the `python-docker` directory in your favorite IDE and enter the following code into the `app.py` file.

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Docker!'
```

## Test the application[🔗](https://docs.docker.com/language/python/build-images//#test-the-application)

Start the application and make sure it’s running. Open your terminal and navigate to the working directory you created.

```
$ cd /path/to/python-docker
$ source .venv/bin/activate
(.venv) $ python3 -m flask run
```

To test that the application is working, open a new browser and navigate to `http://localhost:5000`.

Switch back to the terminal where the server is running and you should see the following requests in the server logs. The data and timestamp will be different on your machine.

```
127.0.0.1 - - [22/Sep/2020 11:07:41] "GET / HTTP/1.1" 200 -
```

## Create a Dockerfile for Python[🔗](https://docs.docker.com/language/python/build-images//#create-a-dockerfile-for-python)

Now that the application is running, you can create a Dockerfile from it.

Inside the `python-docker` directory create a `Dockerfile` and add a line that tells Docker what base image to use for the application.

```
# syntax=docker/dockerfile:1

FROM python:3.8-slim-buster
```

Docker images can inherit from other images. Therefore, instead of creating your own base image, you can use the official Python image that has all the tools and packages needed to run a Python application.

> **Note**
> 
> To learn more about creating your own base images, see [Creating base images](https://docs.docker.com/build/building/base-images/).

To make things easier when running the remaining commands, create a working directory. This instructs Docker to use this path as the default location for all subsequent commands. This means you can use relative file paths based on the working directory instead of full file paths.

```
WORKDIR /app
```

Usually, the first thing you do with a project written in Python is to install `pip` packages to ensure the application has all its dependencies installed.

Before running `pip3 install`, you need the `requirements.txt` file into the image. Use the `COPY` command to do this.

The `COPY` command takes two parameters. The first parameter tells Docker what file(s) you would like to copy into the image. The second parameter tells Docker where to copy that file(s) to. For this example, copy the `requirements.txt` file into the working directory `/app`.

```
COPY requirements.txt requirements.txt
```

With the `requirements.txt` file inside the image, you can use the `RUN` command to run `pip3 install`. This works exactly the same as running `pip3 install` locally on a machine, but this time pip installs the modules into the image.

```
RUN pip3 install -r requirements.txt
```

At this point, you have an image based on Python version 3.8 and have installed the dependencies. The next step is to add the source code into the image. Use the `COPY` command as with the `requirements.txt` file. This `COPY` command takes all the files located in the current directory and copies them into the image.

```
COPY . .
```

Now, tell Docker what command to run when the image is executed inside a container using the `CMD` command. Note that you need to make the application externally visible (i.e. from outside the container) by specifying `--host=0.0.0.0`.

```
CMD ["python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```

Here’s the complete Dockerfile.

```
# syntax=docker/dockerfile:1

FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD ["python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```

### Directory structure[🔗](https://docs.docker.com/language/python/build-images//#directory-structure)

Just to recap, you created a directory on your local machine called `python-docker` and created a simple Python application using the Flask framework. You used the `requirements.txt` file to gather requirements, and created a Dockerfile containing the commands to build an image. The Python application directory structure should now look like the following:

```
python-docker
|____ app.py
|____ requirements.txt
|____ Dockerfile
```

## Build an image[🔗](https://docs.docker.com/language/python/build-images//#build-an-image)

Now that you’ve created the Dockerfile, let’s build the image. To do this, use the `docker build` command. The `docker build` command builds Docker images from a Dockerfile and a “context”. A build’s context is the set of files located in the specified PATH or URL. The Docker build process can access any of the files located in this context.

The build command optionally takes a `--tag` flag. The tag sets the name of the image and an optional tag in the format `name:tag`. Leave off the optional `tag` for now to help simplify things. If you don’t pass a tag, Docker uses “latest” as its default tag.

Build the Docker image.

```
$ docker build --tag python-docker .
[+] Building 2.7s (10/10) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 203B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/python:3.8-slim-buster
 => [1/6] FROM docker.io/library/python:3.8-slim-buster
 => [internal] load build context
 => => transferring context: 953B
 => CACHED [2/6] WORKDIR /app
 => [3/6] COPY requirements.txt requirements.txt
 => [4/6] RUN pip3 install -r requirements.txt
 => [5/6] COPY . .
 => [6/6] CMD ["python3", "-m", "flask", "run", "--host=0.0.0.0"]
 => exporting to image
 => => exporting layers
 => => writing image sha256:8cae92a8fbd6d091ce687b71b31252056944b09760438905b726625831564c4c
 => => naming to docker.io/library/python-docker
```

## View local images[🔗](https://docs.docker.com/language/python/build-images//#view-local-images)

To see a list of images you have on your local machine, you have two options. One is to use the Docker CLI and the other is to use [Docker Desktop](https://docs.docker.com/desktop/use-desktop/images/). As you are working in the terminal already, take a look at listing images using the CLI.

To list images, run the `docker images` command.

```
$ docker images
REPOSITORY      TAG               IMAGE ID       CREATED         SIZE
python-docker   latest            8cae92a8fbd6   3 minutes ago   123MB
```

You should see at least one image listed, including the image you just built `python-docker:latest`.

## Tag images[🔗](https://docs.docker.com/language/python/build-images//#tag-images)

As mentioned earlier, an image name is made up of slash-separated name components. Name components may contain lowercase letters, digits, and separators. A separator can include a period, one or two underscores, or one or more dashes. A name component may not start or end with a separator.

An image is made up of a manifest and a list of layers. Don’t worry too much about manifests and layers at this point other than a “tag” points to a combination of these artifacts. You can have multiple tags for an image. Let’s create a second tag for the image you built and take a look at its layers.

To create a new tag for the image you built, run the following command.

```
$ docker tag python-docker:latest python-docker:v1.0.0
```

The `docker tag` command creates a new tag for an image. It doesn’t create a new image. The tag points to the same image and is just another way to reference the image.

Now, run the `docker images` command to see a list of the local images.

```
$ docker images
REPOSITORY      TAG               IMAGE ID       CREATED         SIZE
python-docker   latest            8cae92a8fbd6   4 minutes ago   123MB
python-docker   v1.0.0            8cae92a8fbd6   4 minutes ago   123MB
python          3.8-slim-buster   be5d294735c6   9 days ago      113MB
```

You can see that two images start with `python-docker`. You know they’re the same image because if you take a look at the `IMAGE ID` column, you can see that the values are the same for the two images.

Remove the tag you just created. To do this, use the `rmi` command. The `rmi` command stands for remove image.

```
$ docker rmi python-docker:v1.0.0
Untagged: python-docker:v1.0.0
```

Note that the response from Docker tells you that Docker didn’t remove the image, but only “untagged” it. You can check this by running the `docker images` command.

```
$ docker images
REPOSITORY      TAG               IMAGE ID       CREATED         SIZE
python-docker   latest            8cae92a8fbd6   6 minutes ago   123MB
python          3.8-slim-buster   be5d294735c6   9 days ago      113MB
```

Docker removed the image tagged with `:v1.0.0`, but the `python-docker:latest` tag is available on your machine.

# Run your image as a container
## Overview[🔗](https://docs.docker.com/language/python/run-containers//#overview)

In the previous module, we created our sample application and then we created a Dockerfile that we used to produce an image. We created our image using the docker command `docker build`. Now that we have an image, we can run that image and see if our application is running correctly.

A container is a normal operating system process except that this process is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.

To run an image inside of a container, we use the `docker run` command. The `docker run` command requires one parameter which is the name of the image. Let’s start our image and make sure it is running correctly. Run the following command in your terminal.

```
$ docker run python-docker
```

After running this command, you’ll notice that you were not returned to the command prompt. This is because our application is a REST server and runs in a loop waiting for incoming requests without returning control back to the OS until we stop the container.

Let’s open a new terminal then make a `GET` request to the server using the `curl` command.

```
$ curl localhost:5000
curl: (7) Failed to connect to localhost port 5000: Connection refused
```

As you can see, our `curl` command failed because the connection to our server was refused. This means, we were not able to connect to the localhost on port 5000. This is expected because our container is running in isolation which includes networking. Let’s stop the container and restart with port 5000 published on our local network.

To stop the container, press ctrl-c. This will return you to the terminal prompt.

To publish a port for our container, we’ll use the `--publish` flag (`-p` for short) on the `docker run` command. The format of the `--publish` command is `[host port]:[container port]`. So, if we wanted to expose port 5000 inside the container to port 3000 outside the container, we would pass `3000:5000` to the `--publish` flag.

We did not specify a port when running the flask application in the container and the default is 5000. If we want our previous request going to port 5000 to work we can map the host’s port 8000 to the container’s port 5000:

```
$ docker run --publish 8000:5000 python-docker
```

Now, let’s rerun the curl command from above. Remember to open a new terminal.

```
$ curl localhost:8000
Hello, Docker!
```

Success! We were able to connect to the application running inside of our container on port 8000. Switch back to the terminal where your container is running and you should see the `GET` request logged to the console.

```
[31/Jan/2021 23:39:31] "GET / HTTP/1.1" 200 -
```

Press ctrl-c to stop the container.

## Run in detached mode[🔗](https://docs.docker.com/language/python/run-containers//#run-in-detached-mode)

This is great so far, but our sample application is a web server and we don’t have to be connected to the container. Docker can run your container in detached mode or in the background. To do this, we can use the `--detach` or `-d` for short. Docker starts your container the same as before but this time will “detach” from the container and return you to the terminal prompt.

```
$ docker run -d -p 8000:5000 python-docker
ce02b3179f0f10085db9edfccd731101868f58631bdf918ca490ff6fd223a93b
```

Docker started our container in the background and printed the Container ID on the terminal.

Again, let’s make sure that our container is running properly. Run the same curl command from above.

```
$ curl localhost:8000
Hello, Docker!
```

## List containers[🔗](https://docs.docker.com/language/python/run-containers//#list-containers)

Since we ran our container in the background, how do we know if our container is running or what other containers are running on our machine? Well, to see a list of containers running on our machine, run `docker ps`. This is similar to how the ps command is used to see a list of processes on a Linux machine.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ce02b3179f0f        python-docker         "python3 -m flask ru…"   6 minutes ago       Up 6 minutes        0.0.0.0:8000->5000/tcp   wonderful_kalam
```

The `docker ps` command provides a bunch of information about our running containers. We can see the container ID, the image running inside the container, the command that was used to start the container, when it was created, the status, ports that were exposed, and the name of the container.

You are probably wondering where the name of our container is coming from. Since we didn’t provide a name for the container when we started it, Docker generated a random name. We’ll fix this in a minute, but first we need to stop the container. To stop the container, run the `docker stop` command which does just that, stops the container. You need to pass the name of the container or you can use the container ID.

```
$ docker stop wonderful_kalam
wonderful_kalam
```

Now, rerun the `docker ps` command to see a list of running containers.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## Stop, start, and name containers[🔗](https://docs.docker.com/language/python/run-containers//#stop-start-and-name-containers)

You can start, stop, and restart Docker containers. When we stop a container, it is not removed, but the status is changed to stopped and the process inside the container is stopped. When we ran the `docker ps` command in the previous module, the default output only shows running containers. When we pass the `--all` or `-a` for short, we see all containers on our machine, irrespective of their start or stop status.

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
ce02b3179f0f        python-docker         "python3 -m flask ru…"   16 minutes ago      Exited (0) 5 minutes ago                        wonderful_kalam
ec45285c456d        python-docker         "python3 -m flask ru…"   28 minutes ago      Exited (0) 20 minutes ago                       agitated_moser
fb7a41809e5d        python-docker         "python3 -m flask ru…"   37 minutes ago      Exited (0) 36 minutes ago                       goofy_khayyam
```

You should now see several containers listed. These are containers that we started and stopped but have not been removed.

Let’s restart the container that we just stopped. Locate the name of the container we just stopped and replace the name of the container below in the restart command.

```
$ docker restart wonderful_kalam
```

Now list all the containers again using the `docker ps` command.

```
$ docker ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                    NAMES
ce02b3179f0f        python-docker         "python3 -m flask ru…"   19 minutes ago      Up 8 seconds                0.0.0.0:8000->5000/tcp   wonderful_kalam
ec45285c456d        python-docker         "python3 -m flask ru…"   31 minutes ago      Exited (0) 23 minutes ago                            agitated_moser
fb7a41809e5d        python-docker         "python3 -m flask ru…"   40 minutes ago      Exited (0) 39 minutes ago                            goofy_khayyam
```

Notice that the container we just restarted has been started in detached mode and has port 8000 exposed. Also, observe the status of the container is “Up X seconds”. When you restart a container, it starts with the same flags or commands that it was originally started with.

Now, let’s stop and remove all of our containers and take a look at fixing the random naming issue. Stop the container we just started. Find the name of your running container and replace the name in the command below with the name of the container on your system.

```
$ docker stop wonderful_kalam
wonderful_kalam
```

Now that all of our containers are stopped, let’s remove them. When you remove a container, it is no longer running, nor it is in the stopped status, but the process inside the container has been stopped and the metadata for the container has been removed.

```
$ docker ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
ce02b3179f0f        python-docker         "python3 -m flask ru…"   19 minutes ago      Exited (0) 5 seconds ago                        wonderful_kalam
ec45285c456d        python-docker         "python3 -m flask ru…"   31 minutes ago      Exited (0) 23 minutes ago                       agitated_moser
fb7a41809e5d        python-docker         "python3 -m flask ru…"   40 minutes ago      Exited (0) 39 minutes ago                       goofy_khayyam
```

To remove a container, run the `docker rm` command with the container name. You can pass multiple container names to the command using a single command. Again, replace the container names in the following command with the container names from your system.

```
$ docker rm wonderful_kalam agitated_moser goofy_khayyam
wonderful_kalam
agitated_moser
goofy_khayyam
```

Run the `docker ps --all` command again to see that all containers are removed.

Now, let’s address the random naming issue. Standard practice is to name your containers for the simple reason that it is easier to identify what is running in the container and what application or service it is associated with.

To name a container, we just need to pass the `--name` flag to the `docker run` command.

```
$ docker run -d -p 8000:5000 --name rest-server python-docker
1aa5d46418a68705c81782a58456a4ccdb56a309cb5e6bd399478d01eaa5cdda
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
1aa5d46418a6        python-docker         "python3 -m flask ru…"   3 seconds ago       Up 3 seconds        0.0.0.0:8000->5000/tcp   rest-server
```

That’s better! We can now easily identify our container based on the name.

# Use containers for development
## Prerequisites[🔗](https://docs.docker.com/language/python/develop//#prerequisites)

Work through the steps to build an image and run it as a containerized application in [Run your image as a container](https://docs.docker.com/language/python/run-containers/).

## Introduction[🔗](https://docs.docker.com/language/python/develop//#introduction)

In this module, we’ll walk through setting up a local development environment for the application we built in the previous modules. We’ll use Docker to build our images and Docker Compose to make everything a whole lot easier.

## Run a database in a container[🔗](https://docs.docker.com/language/python/develop//#run-a-database-in-a-container)

First, we’ll take a look at running a database in a container and how we use volumes and networking to persist our data and allow our application to talk with the database. Then we’ll pull everything together into a Compose file which allows us to setup and run a local development environment with one command.

Instead of downloading MySQL, installing, configuring, and then running the MySQL database as a service, we can use the Docker Official Image for MySQL and run it in a container.

Before we run MySQL in a container, we’ll create a couple of volumes that Docker can manage to store our persistent data and configuration. Let’s use the managed volumes feature that Docker provides instead of using bind mounts. You can read all about [Using volumes](https://docs.docker.com/storage/volumes/) in our documentation.

Let’s create our volumes now. We’ll create one for the data and one for configuration of MySQL.

```
$ docker volume create mysql
$ docker volume create mysql_config
```

Now we’ll create a network that our application and database will use to talk to each other. The network is called a user-defined bridge network and gives us a nice DNS lookup service which we can use when creating our connection string.

```
$ docker network create mysqlnet
```

Now we can run MySQL in a container and attach to the volumes and network we created above. Docker pulls the image from Hub and runs it for you locally. In the following command, option `-v` is for starting the container with volumes. For more information, see [Docker volumes](https://docs.docker.com/storage/volumes/).

```
$ docker run --rm -d -v mysql:/var/lib/mysql \
  -v mysql_config:/etc/mysql -p 3306:3306 \
  --network mysqlnet \
  --name mysqldb \
  -e MYSQL_ROOT_PASSWORD=p@ssw0rd1 \
  mysql
```

Now, let’s make sure that our MySQL database is running and that we can connect to it. Connect to the running MySQL database inside the container using the following command and enter “p@ssw0rd1” when prompted for the password:

```
$ docker exec -ti mysqldb mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.33 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### Connect the application to the database[🔗](https://docs.docker.com/language/python/develop//#connect-the-application-to-the-database)

In the above command, we logged in to the MySQL database by passing the ‘mysql’ command to the `mysqldb` container. Press CTRL-D to exit the MySQL interactive terminal.

Next, we’ll update the sample application we created in the [Build images](https://docs.docker.com/language/python/build-images/#sample-application) module. To see the directory structure of the Python app, see [Python application directory structure](https://docs.docker.com/language/python/build-images/#directory-structure).

Okay, now that we have a running MySQL, let’s update the `app.py` to use MySQL as a datastore. Let’s also add some routes to our server. One for fetching records and one for creating our database and table.

```
import json
import mysql.connector
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Docker!'

@app.route('/widgets')
def get_widgets():
    mydb = mysql.connector.connect(
        host="mysqldb",
        user="root",
        password="p@ssw0rd1",
        database="inventory"
    )
    cursor = mydb.cursor()


    cursor.execute("SELECT * FROM widgets")

    row_headers=[x[0] for x in cursor.description] #this will extract row headers

    results = cursor.fetchall()
    json_data=[]
    for result in results:
        json_data.append(dict(zip(row_headers,result)))

    cursor.close()

    return json.dumps(json_data)

@app.route('/initdb')
def db_init():
    mydb = mysql.connector.connect(
        host="mysqldb",
        user="root",
        password="p@ssw0rd1"
    )
    cursor = mydb.cursor()

    cursor.execute("DROP DATABASE IF EXISTS inventory")
    cursor.execute("CREATE DATABASE inventory")
    cursor.execute("USE inventory")

    cursor.execute("DROP TABLE IF EXISTS widgets")
    cursor.execute("CREATE TABLE widgets (name VARCHAR(255), description VARCHAR(255))")
    cursor.close()

    return 'init database'

if __name__ == "__main__":
    app.run(host ='0.0.0.0')
```

We’ve added the MySQL module and updated the code to connect to the database server, created a database and table. We also created a route to fetch widgets. We now need to rebuild our image so it contains our changes.

First, let’s add the `mysql-connector-python` module to our application using pip.

```
$ pip3 install mysql-connector-python
$ pip3 freeze | grep mysql-connector-python >> requirements.txt
```

Now we can build our image.

```
$ docker build --tag python-docker-dev .
```

If you have any containers running from the previous sections using the name `rest-server` or port 8000, [stop](https://docs.docker.com/language/python/run-containers/#stop-start-and-name-containers) them now.

Now, let’s add the container to the database network and then run our container. This allows us to access the database by its container name.

```
$ docker run \
  --rm -d \
  --network mysqlnet \
  --name rest-server \
  -p 8000:5000 \
  python-docker-dev
```

Let’s test that our application is connected to the database and is able to add a note.

```
$ curl http://localhost:8000/initdb
$ curl http://localhost:8000/widgets
```

You should receive the following JSON back from our service.

```
[]
```

## Use Compose to develop locally[🔗](https://docs.docker.com/language/python/develop//#use-compose-to-develop-locally)

In this section, we’ll create a [Compose file](https://docs.docker.com/compose/) to start our python-docker and the MySQL database using a single command.

Open the `python-docker` directory in your IDE or a text editor and create a new file named `docker-compose.dev.yml`. Copy and paste the following commands into the file.

```
version: '3.8'

services:
 web:
  build:
   context: .
  ports:
  - 8000:5000
  volumes:
  - ./:/app

 mysqldb:
  image: mysql
  ports:
  - 3306:3306
  environment:
  - MYSQL_ROOT_PASSWORD=p@ssw0rd1
  volumes:
  - mysql:/var/lib/mysql
  - mysql_config:/etc/mysql

volumes:
  mysql:
  mysql_config:
```

This Compose file is super convenient as we do not have to type all the parameters to pass to the `docker run` command. We can declaratively do that using a Compose file.

We expose port 8000 so that we can reach the dev web server inside the container. We also map our local source code into the running container to make changes in our text editor and have those changes picked up in the container.

Another really cool feature of using a Compose file is that we have service resolution set up to use the service names. Therefore, we are now able to use “mysqldb” in our connection string. The reason we use “mysqldb” is because that is what we’ve named our MySQL service as in the Compose file.

Note that we did not specify a network for those 2 services. When we use docker-compose it automatically creates a network and connect the services to it. For more information see [Networking in Compose](https://docs.docker.com/compose/networking/)

If you have any containers running from the previous sections, [stop](https://docs.docker.com/language/python/run-containers/#stop-start-and-name-containers) them now.

Now, to start our application and to confirm that it is running properly, run the following command:

```
$ docker compose -f docker-compose.dev.yml up --build
```

We pass the `--build` flag so Docker will compile our image and then start the containers.

Now let’s test our API endpoint. Open a new terminal then make a GET request to the server using the curl commands:

```
$ curl http://localhost:8000/initdb
$ curl http://localhost:8000/widgets
```

You should receive the following response:

```
[]
```

This is because our database is empty.