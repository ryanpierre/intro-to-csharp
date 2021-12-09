# CI/CD Pipelines

## Intro

So we've successfully been able to package our source code from our local machine in to a docker image, push it to a container registry (docker hub), and have that pulled in by microsoft azure app service. Fantastic !

But what happens when that code is collaborative, and it's not just one person who deems a piece of code is ready to ship ? 

Furthermore, without anyone actually simulating the deployment or personally overlooking it, how can we ensure we're pushing quality code ?

We'll attempt to address those problems today with **CI/CD Pipelines**

### Github Actions

Github actions are a form of CI/CD pipeline that allow us to create sequences of events. Those sequences of events live on their own, they aren't explicity triggered by anything in their own code, but the merely exist as tools to be executed by other events.

Events like:
- When you push code to main branch
- When you create a pull request that is attempting to push to main branch
- When code has been reverted on the main branch

Actions, like other things we've encountered so far can be public as well. If they are public, they'll be available on [Github Marketplace](https://github.com/marketplace?type=actions). Much like the docker hub images we can use to create our docker images, we can use actions from github marketplace (or create our own) to accomplish commonly requested tasks. 

Github actions are also like small computer programs on there own in the sense that an action will be executed by a real, one time use computer somewhere in the world called a **Lamda function**. Azure functions are an example of this: https://azure.microsoft.com/en-gb/services/functions/#overview

So, much like our Dockerfiles, we will need to specify what type of machine will be needed to run our code in a github action. A github action might look something like this:

```
name: Azure CI/CD Pipeline

on:
  push:
    branches:
      - deploy
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - deploy

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
    - uses: actions/checkout@v2
    - name: Install dependencies
      run: dotnet restore
```

First, we're specifying that we'd like this action to trigger on 

Then, we are saying that we need a computer using the latest version of ubuntu to perform this task. Ubuntu is a slim version of linux that is widely used on servers globally because of how lightweight it is.

We're also saying, only run this job if the event name that triggered it is a push, or a pull request action that isn't closing the pull request. 

Finally, we're saying "use the actions/checkout@v2" action from github actions marketplace as one of our steps to complete our action and installing dependencies. Each `-` in our steps section specifies another step in our action. It's not uncommon for actions to have many steps associated with them. You can also pass arguments to steps using the `with` keyword. For example:

```
- name: Setup .NET 5.x
  uses: actions/setup-dotnet@v1
  with:
    dotnet-version: '5.x'
```

This action uses the `actions/setup-dotnet` action from the marketplace, and then passes the argument `dotnet-version: '5.x'` to say: any version of .NET version should be fine. You could also be more specific and specify the exact version number of .NET if you'd prefer.

You'll also likely need to login to a service or two while you're doing this. Since this code might (and likely will) be public, we don't want to be posting our username and password to github (ever !)

Instead, we'll use **secrets**. Secrets are referenced in our github actions like so:

```
-
  name: Login to DockerHub
  uses: docker/login-action@v1 
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

We would then add those secrets (DOCKERHUB_USERNAME and DOCKERHUB_TOKEN) to our github account so that it could automatically read those values and fill them in at run time. 

The final thing to remember with all this: in github actions, **indentation matters**. 

This

```
- name: Ryan
  uses: something
```

is not the same thing as

```
- name: Ryan
    uses: something
```

### Your task

Your task today is to connect your source code repository to Azure Cloud service using Github Actions and Docker Hub. 

Your github actions should:
- Run tests when a pull requests is made to merge to the main branch
- Create a docker image when code is pushed to the main branch, tag it, and send it to your azure cloud repo.

Then, try adding a new controller route to the HomeController and a matching view. Finally, create a pull request and test that your full pipeline works and you see your app on your Azure website.

### Resources 

- [Github Actions](https://resources.github.com/devops/tools/automation/actions)
- [Testing dotnet with Github Actions](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-net)
- [Workflow Events (such as triggering on push)](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows)
- [.NET action on github marketplace](https://github.com/marketplace/actions/setup-net-core-sdk)
- [Docker Actions on github marketplace](https://github.com/marketplace/actions/build-and-push-docker-images)
- [Github Secrets](https://github.com/Azure/actions-workflow-samples/blob/master/assets/create-secrets-for-GitHub-workflows.md)