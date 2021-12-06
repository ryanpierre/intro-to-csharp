# Working with Docker

## Introduction

Docker is one of the (if not *the*) most widely used tool for containerization. Almost every hosting service has streamlined services to accomodate docker, and others (like kubernetes) are hosting services *specifically* for docker.

Not surprisingly, our cloud service provider of choice for this course (Microsoft Azure) also plays very nicely with Docker.

In short - Docker gives us a tool where we can:
1. Describe the requirements of an application to run from a fresh machine
2. Build those requirements as needed on any Docker Host
3. Run the application on any Docker Host

This allows our app to be fully portable and scalable. Since we have a description of exactly how to replicate the app environment, and how to run the app, this becomes particularly useful in situations where our scale is in flux. That is, suppose your app suddenly sees an influx of 100,000 users. If you were using physical machines, you'd need to buy more machines and provision them to deal with the demand (slow). With docker, you can tell your hosting service to replicate more machines and within minutes you have more servers to handle your traffic.

We'll be using docker today to package our app we made in the first task in such a way that we can easily host it on the cloud, and scale/replicate it as needed

## Setup

The first thing we'll need to do is install the docker client for mac. Head over to https://docs.docker.com/get-docker/ and download/install the mac version. 

After you install and accept the agreements, it'll ask you to do a short tutorial. Go ahead and try it out, play with docker a little bit :) 

Make sure to skip the step about sharing your image on Docker Hub. We'll talk about that more later.

The tutorial should ask you to perform a few tasks with a demo project, and then at the end you'll be able to open it in your browser and have a look at the tutorial running on your docker container ! Cool !

In fact, the tutorial covers basically everything we would want to say about containers, so just spend some time fully reading the "Getting Started" section's material.

## Task

### Dockerfile

Our goal today is simple: To containerise our web app we made in the first excercise so that we can run it locally from a docker container. 

We wouldn't necessarily want to do this in development because Visual Studio provides us so many tools to do that effectively, but in terms of testing how our app will work in production this is the perfect way to do it.

We'll have a realistic 1:1 environment of exactly how the app will behave in production (because we'll be deploying with docker as well !). Often times, we test an app in development and then see completely different results in production. This will give us an intermediate step as a means of quality control.

The first step is to create a `Dockerfile` in our project. We'll create it in the App (or Core folder, or whatever you named it). This is because it's possible we might have different docker configurations for each project (UnitTests for example, but not likely). Here's where we'll start with our dockerfile:

```Dockerfile
# Select the template from docker hub we're using as our build environment
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
```

Here, we're telling docker to use the docker hub template `mcr.microsoft.com/dotnet/sdk:5.0`. This contains everything we need to get from an empty docker container to building a dotnet project and running it. 

Next, we'll run `docker build -t aspnet-example-image -f Dockerfile .` from the `App` or `Core` directory in our project order to build our docker **image**

In this command, we tell docker to build an image called `aspnet-example-image` using the Dockerfile located in the same directory we're in. 

In the dockerverse (and the containerverse for that matter) **image**s are the virtual machines we'll be mounting to the **container** (which we'll create later). 

Congrats ! You've made your first .net image. 

Next, we'll add some more to our Dockerfile:

```Dockerfile
# Select the template from docker hub we're using as our build environment
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env

COPY bin/Release/net5.0/publish/ DockerAppDemo/
WORKDIR /DockerAppDemo
ENTRYPOINT ["dotnet", "App.dll", "--urls", "http://0.0.0.0:5000"]
```

The new lines we've added tell the docker image to copy whatever is in our publish directory `bin/Release/net5.0/publish/` (when we publish a C# app for production, this is the folder it will be published to) into a folder on the image called DockerAppDemo

Then, we change the directory to /DockerAppDemo on the image, and set an entry point to our application. We're telling docker to run the command `dotnet` with the argument `App.dll`. App.dll is what's created when we compile and publish our app. It will always be the same name as the project (so UnitTests would be called UnitTests.dll). The final piece here is the host we set - we pass another two arguments: `--urls` and `http://0.0.0.0:5000`. This is a caveat of how docker work on mac that we just need to do. If we don't pass a url to 0.0.0.0, dotnet will use localhost by default instead, which is not the same thing. 

As a general rule, expect to add this urls flag to every docker entry point you create for asp net on mac. Otherwise, you won't be able to connect to your container (more reading on this at the bottom of the page)

In order to publish our app, we run `dotnet publish -c Release` from the project directory (App or Core whichever you chose). This will compile and build the production build. We can also use the argument `-c Debug` if we want a build for testing rather than production. Try running that now to ensure your web app builds correctly.

This is tantamount in other languages like ruby to `ruby App.rb` or `node index.js`. It's the entry point to our application that will run the main program. 

Let's try building again (`docker build -t aspnet-example-image -f Dockerfile .`)

Now, we'll create a container to mount our image to: enter `docker create --name aspnet-example-container aspnet-example-image` into the console.

Here, we're telling docker to create a container named aspnet-example-container and mount the aspnet-example-image to it. You'll be returned back a container id on success. You can now type `docker ps -a` to see all running containers on your computer. You should see your new one on this list ! Hooray. 

If you open the docker dashboard, you should also see your container there now too. You can either hit the play button in docker dashboard, or type `docker start <the id that was returned>` to start the container up. You can use the stop button in dashboard or `docker stop <id>` to stop a particular container.

If you haven't already, go ahead into your docker dashboard and hit the trash can icon to delete the tutorial container before we do the next part.

### Testing our image

We can create a container and run our image at the same time using the command:

`docker run -dp 5000:5000 --name app aspnet-example-image`

This is a *Very* important command. First of all, the first two flags: -d tells docker to run this in the background and not on the current terminal you're in. This is important because otherwise when you close the terminal your docker container will shut down too. The second, -p is the "Publish port" for our container. This is a tricky one. Essentially, *inside* of our docker container, which is it's own computer with it's own network, it runs the app on localhost:5000 (or 0.0.0.0:5000 in our case because of our --urls flag). 

Since the docker container is it's own computer, localhost:5000 on my macbook is not the same as localhost:5000 on the docker container, so i can't access it in my browser. The -p flag lets us say: Map the port 5000 on my network inside the container to port 5000 outside of the container. If we said 10:5000, it would map 0.0.0.0 port 10 inside the container to 0.0.0.0 5000 outside of the container. This is a lot of networking stuff, I would recommend reading the docker networking article linked at the bottom of the page.

The last little detail is we set the name to app. Otherwise, docker will generate something random. I personally enjoy using a name because if you try and spin up another container you'll get an error the name is in use. The random names will keep generating new containers and you'll have to delete them all !

When you visit localhost:5000 in your browser, you should now see your app, running inside your container. Cool !

### The task

Currently, we need to publish and build our app before we can run it in a container. Your task is to modify the Dockerfile so that it:

- Copies the source code either from git, or our local machine to the docker container
- Installs dependencies for our app *on the docker container*
- Builds/Compiles the code *on the docker container* rather than our local machine

### Resources

- [How networking on Docker works](https://pythonspeed.com/articles/docker-connection-refused/)
- [Building docker images with ASP.NET](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/building-net-docker-images?view=aspnetcore-5.0)
- [Containerize an ASP.NET application](https://docs.microsoft.com/en-us/dotnet/core/docker/build-container?tabs=windows)
- [Dockerfile explained (ALOT OF READING WARNING)](https://docs.docker.com/engine/reference/builder/)
- [The Docker Ecosystem explained](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components)







