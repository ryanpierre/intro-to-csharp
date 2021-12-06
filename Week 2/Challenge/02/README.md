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

Our goal today is simple: To containerise our web app we made in the first excercise so that we can run it locally from a docker container. 

We wouldn't necessarily want to do this in development because Visual Studio provides us so many tools to do that effectively, but in terms of testing how our app will work in production this is the perfect way to do it.

We'll have a realistic 1:1 environment of exactly how the app will behave in production (because we'll be deploying with docker as well !). Often times, we test an app in development and then see completely different results in production. This will give us an intermediate step as a means of quality control.

The first step is to create a `Dockerfile` in our project. Here's an example of a Dockerfile:

```Dockerfile
# Select the template from docker hub we're using as our build environment
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
WORKDIR /app

# Copy csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# Copy everything else and build
COPY ../engine/examples ./
RUN dotnet publish -c Release -o out

# Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "App.dll"]
```







