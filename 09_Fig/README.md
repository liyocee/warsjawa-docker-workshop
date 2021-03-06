## FIG

First of all, we need to install FIG, here you can find some instructions http://www.fig.sh/install.html.

**Note**: For OSX users, to take the advantages of FIG, we highly recommend running docker via https://github.com/noplay/docker-osx. Some features, which can be found in FIG, such as volume mapping, won't work in boot2docker.

FIG makes linking many different containers very easy. All we have to do is to describe our environment in a .yml file. No big deal at all. 

We will be running a web app built on Flask and Redis. Remeber linking containers inprevious excercise? Can you imagine what would happen, if number of Redis instances grows up to 4? That's why we need a tool like FIG, which will do most of the crappy, repeatable, human-keyboard-like work for us :smile:

## Let's do this

Our goal would be to build web application separated into two containers - one of them will cover the python-flask-based webapp responding to some HTTP requests, and the second container will be our Redis instance - key-value store in backend.

Please go to `example_1` directory.
Source code of the app can be found in `our_web_app.py`.
We still have a Dockerfile, which will be a base to build webapp container. Take a closer look on its code.
As a Redis container, we will use the default image from Docker Hub.

Now let's take a look on how everything of our webapp ecosystem is defined in `fig.yml` file. We can run it, and check out how it works:
```
fig up
```

We strongly encourage encourage you to play around with fig. First of all, meet its most useful options and switches:
* `fig up -d` will let you to run your container stack in detached mode. However, all the logs can be still easily browsed later on with `fig logs`. 
* `fig ps` will show you all running containers
* SCALE DAT! Run `fig scale redis=3` to bring up more or less instances of certain service. 
* Typing just `fig` with no options will display some help and list of available commands. Just in case you forget some obvious `rm` `start` `stop` `build` actions.
For more, look go to http://www.fig.sh/cli.html

Somebody might ask, how it's possible, that the containers 'link' to each other. How do they now, for instance, whats' the other containers PostgreSQL password and username? Well, it's nothing more than classic docker link we've learned so far, in previosu excercise :smile: This command `fig run redis env` and `fig run web env` might give you the answers. Also check out the `/etc/hosts` file. That's the trick.  It's just like linking containers with docker underneath - tons of environment variables, shared in a namespace let two services talk to each other :smile: Having a closer look to this tutorial http://www.fig.sh/wordpress.html is really all you need to know as for the practical example. More on WHY, rather than HOW: http://www.fig.sh/env.html , http://docs.docker.com/userguide/dockerlinks/

## Let's try some experiments 

First of all, ***modify the source code of the webapp*** and send another GET request. Because of using current directory as a volume, you don't need to rebuild the container - all the changes apply immediately!

Now please change the direction of the relationship. Let the redis container link to the web right now. Some of you may be surprised with 
```
ConnectionError: Error -2 connecting to redis_1:6379. Name or service not known.
```
after running the application. This is one of the most important thing to keep in mind while designing services in several containers - the linking relation is NOT bi-directional.

Service `web` needs to **be aware** of redis instance, so that it needs to link to redis instance. On the other hand, redis instance is completely unaware of what is happening around it. It's a bit tricky, and as you guess dynamically spawned containers needs some service discovery and maintaining such an environment with fig only is an overkill.

Keep in mind previous lesson about linking - linking two containers is nothing more that passing environment variables from one container to another, so that one of them 'is equipped with contact info' of the second one and can establish a connection. ***When designing a Dockerfile for your custom app make sure that it contains enough pieces of information (env. variables) about its services (like login credentials) so that after passing them to another container it will be really able to contact it***


