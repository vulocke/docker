# Docker 101

🎯 The purpose of this exercise is to:

* guide you through common Docker commands, using the `docker` CLI,
* while giving some details about them and the associated Docker concepts

---

### Run a container

The **`docker run`** command is used to run a container from an image.  
<br>
👉 For instance, run: ```docker run postgres```

It will run an instance of the postgres database (that you know from yesterday: we used it for our Twitter API). How 🤔 ?

- Docker will use the image from the host (your machine) if it exists
- If the image is not present on the host, it will go to the Docker Hub and pull the image for you 👌!
  
You can see [here](https://hub.docker.com/_/postgres) that the postgres image is available on the Hub !  
Note that for the subsequent executions, the same image will be re-used. Your machine won't have to pull the image over and over.

But notice that when you ran `docker run postgres`, it actually did not work ! Why ?


<details>
	<summary markdown='span'>View solution</summary>

```bash
Error: Database is uninitialized and superuser password is not specified.
   You must specify POSTGRES_PASSWORD to a non-empty value for the
   superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".
```
Well, it looks like the postgres image is expecting something on `docker run`!

</details>

👉 Try to fix it !

<details>
	<summary markdown='span'>View solution</summary>

Well let's just do what `docker run` asks for:

```bash
docker run -e POSTGRES_PASSWORD=password postgres
```	

</details>

🍾 That's it! No need for a complex installer, no conflicts on your laptop, no long hours scrolling tech forums to fix your problem : you have a running database in a single command.


The `-e` flag in `docker run -e ...` stands for environment variable. Here, we just created a password for the database superuser, and passed it the `docker run` command. Ideally a password should be more complex, but here we don't really care, we will get rid of this container in a few minutes anyway !


Now, you should see an output in your terminal: it looks like your database is initialized and ready to accept connections !

<details>
	<summary markdown='span'>View solution</summary>

Your terminal output should look like this:

```bash
...
PostgreSQL init process complete; ready for start up.

2020-11-19 19:00:28.075 UTC [1] LOG:  starting PostgreSQL 13.1 (Debian 13.1-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
2020-11-19 19:00:28.076 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2020-11-19 19:00:28.076 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2020-11-19 19:00:28.079 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2020-11-19 19:00:28.084 UTC [56] LOG:  database system was shut down at 2020-11-19 19:00:27 UTC
2020-11-19 19:00:28.091 UTC [1] LOG:  database system is ready to accept connections
```	

</details>


👉 Stop the container by hitting CTRL-C.

---

### Run a container in the background

One other thing you can do, is run a container in the background (using the "detached" mode). This way, your terminal prompt will be available.

👉 Try run a new postgres container in detached mode, and give it a name: `pg`.

<details>
	<summary markdown='span'>Hint</summary>
	
If you are not sure about the syntax and available options: ask for them !
```bash
docker run --help
```

</details>

<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker run -d -e POSTGRES_PASSWORD=password --name=pg postgres
```

</details>


What happens if we do not give it a name ?   
👉 Run another postgres container without giving it a name: what name has it been assigned ?  

<details>
	<summary markdown='span'>Hint</summary>
	
Hint know our to run a container already... Do you remember how to list all running containers ?

</details>
<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker run -d -e POSTGRES_PASSWORD=password postgres
docker ps
```

Well Docker generates some names for us if we do not pass any.

</details>

---

### Access the Postgres database

Now that your container is running, you might want to run a SQL query.  
Let's first get a bash shell in the container:

👉 Run ```docker run -it pg /bin/bash```

What have we done here 🤔 ? We have asked Docker to run a command (`/bin/bash`: to get a bash shell) in the container, passing the flag `-i` and `-t` together: 

* the `-i` flag stands for "interactive" mode: it gets us a standard input **stdin** (by default, a container runs in a non-interactive mode: it does not listen for input from your side). To provide an input, you need to pass this `-i` flag.
* the `-t` flag stands for "tty", and is a Unix-like operating system command: with this flag, you will get a "prompt"

So the combination of these two flags give us access to a "terminal", in the container 🎉 !

We can now access our DB using the `psql` CLI:

👉 Run ```psql -U postgres```

It gives you access to the Postgresql command line, where you could write SQL.

<p><img src="https://github.com/lewagon/fullstack-images/blob/master/reboot-python/psql-docker.png?raw=true" width="500"></p>


That is typically what developers do locally when they work on a webapp project (like you did for the Twitter API): they dockerize the webserver application, and the database. This way their setup is standardized, and they can easily share it !

---

### Clean up 

#### stop & remove containers

To stop a container, use the `docker stop` command. You will need to pass the container ID, or the container name.

👉 Stop your `pg` container, and check the list of running containers, and the list of non-running containers.


<details>
	<summary markdown='span'>View solution</summary>
	
Stop the container:
```bash
docker stop pg
```

And list running and exited containers:
```bash
docker ps
docker ps --filter "status=exited"
```
</details>

You should see that your container changed state, from `running` to `exited`, but it is still here ! To remove it from your host, run the `docker rm` command.

👉 Remove your postgres container

<details>
	<summary markdown='span'>Hint</summary>
	
```bash
docker rm --help
```

</details>
<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker rm pg
```

</details>

#### remove an image

So you have stopped and removed your postgres container, but how about the postgres image that your host initially pulled from the Docker Hub.

👉 Do you remember how to list docker images ?

<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker images
```

</details>

👉 Try to remove the `postgres` image using `docker rmi` and double check it worked

<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker rmi postgres
docker images
```

</details>

---

### Dockerfile, docker build, docker push

Let's recap what we just did:

* we ran a `pg` container from the `postgres` image
* we stopped it
* we removed it from the host
* we also removed its image

That was convenient because we wanted to play with the Postgres image - already available on the Hub.

🤔 But what if we wanted to create our own image ? And put it on the Hub to make it available for other projects, other developers ?

Well, we would have to:

* ➡️ write a `Dockerfile`, 
* ➡️ build our Docker container,
* ➡️ push it onto the Docker Hub !

💪 **Let's do this !**


We will play with the `docker/whalesay` image, and create a custom image out of it.

👉 Pull the `docker/whalesay` image from the Hub

<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker pull docker/whalesay
```

</details>

👉 Run a container from this image (please check [how to use this image](https://hub.docker.com/r/docker/whalesay/))

<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker run docker/whalesay cowsay "This whale is speaking ..."
```

</details>


From there, we would like to modify this image, such that the whale automatically "says" something (randomly generated).
 
We would like to run something like: ```docker run <YOUR DOCKER ID>/custom-whale``` that generates a famous quote.

So we need different things here:
* build a new image from the `docker/whalesay` image: you already know we will have to write a [Dockerfile](https://docs.docker.com/engine/reference/builder/)
* be able to generate random quotes: we will be using [Unix Fortune](https://en.wikipedia.org/wiki/Fortune_(Unix))
* push this built image to your personal Docker Hub repository


👉 Create a file named Dockerfile in this challenge folder

```bash
cd ~/code/<user.github_nickname>/reboot-python
cd 05-Docker/01-Docker-101
touch Dockerfile
```

👉 Copy and paste the following text in it

```dockerfile
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
```

What does it do ?

---

```dockerfile
FROM docker/whalesay:latest
```

> The **FROM** instruction initializes a new build stage and sets the Base Image for subsequent instructions. Here, we make use of the `docker/whalesay` image (its `latest` version) and build our custom image out of it.

```dockerfile
RUN apt-get -y update && apt-get install -y fortunes
```

> The **RUN** instruction will execute any commands in a new layer on top of the current image and commit the results. Each line of Dockerfile represents a "layer" of the full image.
Here, the command updates the list of available packages and their versions, and install the `fortune` program for random quote generation.

```dockerfile
CMD /usr/games/fortune -a | cowsay
```

> There can only be one **CMD** instruction in a Dockerfile. The main purpose of a **CMD** is to provide defaults for an executing container.
Here, the trick is to make use of the "pipe" (`|`) that will first generate a random quote (using `/usr/games/fortune -a`) and pass it to the [`cowsay`](https://en.wikipedia.org/wiki/Cowsay) command.

---

👉 It's now time to build your image: ```docker build -t $DOCKER_ID/custom-whale .```  - which will read instructions from the Dockerfile. Do not forget to replace `$DOCKER_ID` with your actual docker ID ⚠️ !
  
You should see that it has been built layer by layer.

👉 Run it to see what it does

<details>
	<summary markdown='span'>View solution</summary>
	
```bash
docker run $DOCKER_ID/custom-whale
```

</details>

👉 You can now push your image to your own repository on the Docker Hub. Make sure you are logged in:
```docker login``` and run the following:

```bash
docker push $DOCKER_ID/custom-whale
```

This way, since it is public, anyone would be able to pull it and run it as a container.

👉 Same as what we saw for the Postgres image, you can see it listed [here](https://hub.docker.com/) 

👉 Some people went even further ... Try this one: ```docker run -it --rm danielkraic/asciiquarium``` !

---

## I'm done! 🎉

That's it for this challenge ! Before you jump to the next challenge (Twitter-API), let's mark your progress with the following:

```bash
cd ~/code/<user.github_nickname>/reboot-python
cd 05-Docker/01-Docker-101
touch DONE.md
git add DONE.md && git commit -m "05-Docker/01-Docker-101"
git push origin master
```
