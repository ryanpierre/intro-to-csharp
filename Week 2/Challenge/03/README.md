# Azure App Service 

## Intro

What is Microsoft Azure ? Well it's a lot of things. 

In short, it's a hosted cloud solution for application development. Azure is meant to replace what was once the work of physical machines in your office. 

When you run your web app on your computer, you can access it from localhost. Think of a server as a computer in the office whose only job it is to run your app, and provide a way people can connect to it from outside your office. It's still going to run your application on localhost:3000, or whatever you normally do in your dev environment, but this time it's got a special adapter to say`"When I receive traffic to my IP address, if it's ___, ___ or ___ type of traffic, treat them as if they were connecting to localhost:3000 and send the traffic to whatever is happening there (your app)`

This is a fairly boilerplate process, so Azure has solved this problem by offering a powerful infrastructure to host your apps and create traffic rules like the above so that people on the internet can connect to your app easily. 

But wait ! Theres more. Azure also provides a suite of other services to optimize processes that normal servers might have dificulty accomplishing. In our office scenario, it's more likely that you'd have many servers, and they are all orchestrated to handle incoming traffic. Azure can optimize it's resources (given, they have 1000s of servers) to best suit your app since they are managing other apps you didn't write too. Depending on what resources each app needs, and when, Azure can optimize resources to lower costs for everyone. 

This allows us to just focus on what we do best: build apps ! Azure hopes to solve everything else in terms of ensuring your app is always available to users in the way you want it to be.

## To the cloud !

The first thing we'll need to do before we can put our app onto Azure's cloud services is to [create a Docker Hub Account](https://hub.docker.com/signup). You'll have to create a unique docker id you'll use for your docker services. Choose the Free plan and finish signing up. 

Ideally, you'll make your own account to better match what you'd likely be doing in the real world. If you're not comfortable using your credit card for a check, ask your coach to make you an account instead. If you're okay with that, [make an account with Azure](https://azure.microsoft.com/en-us/free/). 

Cloud services like Azure expose themselves to security risks by hosting your app, but more importantly they allow a potential user to make massive changes to a company's products. With the higher level of risk, they need to know you're a real person and there is a real way to contact you.  

After the account creation process is complete, you'll be taken to your dashboard. 

## Task

Your task is to deploy your ASP.NET app to docker hub, then to consume it via Azure App Serverice. If this all goes successfully, you should be able to manage your asp net app in the azure app service interface, and view it online via the URL provided.

Docker Hub works something like github. You'll need to make sure the name you give your image when you build it matches the repository name in order for it to get picked up. For example, if my app was called `DockerDemo`, but my docker hub repo was `makerstech/docker-demo`, then I would input into my terminal:

`docker build -t makerstech/docker-demo .` when I'm in my DockerDemo/App directory that contains my Dockerfile. There is a lot of naming confusion here. 

You'll need to learn how to build an image, see a list of images, tag an image, and push it to docker hub first to achieve this. Then, you'll need to connect that to your Microsoft Azure App Service.

## Resources

- [A fantastic introduction to all things Azure](https://www.azurebarry.com/introduction-to-azure-app-service-part-1/)
- [A great article about pipelines, and github actions vs azure pipelines](https://docs.microsoft.com/en-us/dotnet/architecture/devops-for-aspnet-developers/actions-vs-pipelines)
- [Pushing an image to docker hub](https://docs.docker.com/docker-hub/repos/)
- [Learn about docker tags](https://docs.docker.com/engine/reference/commandline/tag/)
- [Docker getting started guide](https://docs.docker.com/get-started/)

