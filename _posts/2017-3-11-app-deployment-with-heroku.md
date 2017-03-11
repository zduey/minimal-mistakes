---
title: Deploying an Application with Heroku

categories:
    - snippets
tags:
    - python
    - sys-admin
    - web
---

tl;dr Deploying an app is easy with heroku: [IEX Streaming App](http://iex-streaming.herokuapp.com/) ([Code](https://github.com/zduey/iex))



This is a follow-up to an [earlier blog post](https://zduey.github.io/snippets/streaming-stock-data-with-bokeh/) on building a single-page web app in bokeh. In that post, I went through some of the background on how to build a chart in bokeh that streams data. 

This post is about deploying an application using as PaaS provider. It is not a thorough comparison of the various PaaS providers out there, because quite honestly, my objective function was this simple:

- Do I know about it (binary)
- Can I deploy for free (binary)
- Ease of use (hand-wavy continuous variable)

The providers I was choosing between included:

1. Amazon AWS
2. Google Cloud Platform
3. Digital Ocean
4. Heroku

In terms of cost, all of the providers (except Digital Ocean) have some sort of free tier. AWS and Google Cloud Platform are both feature-rich, which is both a blessing and a curse. On the plus side, if my intent was to build something scalable and production-ready, those would likely be the clear front-runners. On the negative side, because of all the features, it takes a bit of effort to navigate all of the microservices and to determine what to include as part of the stack. As a hobbyist (at least in this use case), I needed a very small compute instance with a simple deployment mechanism. In the end, Heroku was the top choice because of its free tier and relatively straightforward (and automated) deployment mechanism. The downside to Heroku's free tier is that after 30 minutes of inactivity, the application will sleep, which causes the page to load a bit slowly the next time it is accessed. Heroku has very readable and helpful documentation, so if I do a poor job with this overview, go directly to the source where they have plenty of information to get you up and running. 

Once you create your Heroku account, I recommend reading [How Heroku Works](https://devcenter.heroku.com/articles/how-heroku-works). It covers all of the concepts you need to get a basic app up and running but without any messy or overly-technical architectural details. After that, work your way through [Getting Started on Heroku with Python](https://devcenter.heroku.com/articles/getting-started-with-python#introduction). As part of the tutorial, you will need to install the Heroku CLI. On my Ubuntu workstation, it was as simple as adding their apt repository. Heroku provides instructions for other operating systems as well.

```bash
sudo add-apt-repository "deb https://cli-assets.heroku.com/branches/stable/apt ./" 
curl -L https://cli-assets.heroku.com/apt/release.key | sudo apt-key add - 
sudo apt-get update
sudo apt-get install heroku 
```

While you are logged in, go to your dashboard and create a new app. All you need to do is give it a name. The name is not strictly necessary as heroku will randomly choose one if you leave it blank. I named my app ```iex-streaming```. Once you create the app, you will be brought to the deployment screen where you can choose between Heroku Git, GitHub, and Dropbox. I chose to deploy from a GitHub repo. If you go this route, create a repo from within GitHub so that you can connect it to Heroku. Alternatively, you can use Heroku as a remote and push to Heroku directly using the CLI tool. You can also use Dropbox, but I will not talk about that here since I do not see why you would choose that option unless you wanted to avoid having to learn Git. If you are in that boat, then my suggestion would be to rethink that decision and take the time to at least learn the basics of Git. A great way to get started is to go through the [Try Git](https://try.github.io/levels/1/challenges/1) tutorial. This tutorial gives you enough to follow along the rest of the way.

With the preliminaries out of the way, there are only a few things that need to be done to get our single page bokeh app deployed. First, turn the folder where your code is stored in a Git repository. You will also need to set up your local repository to track the remote repository. If you are using Heroku Git, the command is a bit different, but you can find the instructions in the deployment page I mentioned earlier.

```bash
git init
git remote add origin https://github.com/zduey/iex.git
```

If you have not already done so, create an environment for the project (I am using conda) and install pandas and bokeh. Once you are done with that, make sure you activate the environment and check that the app still runs locally in case there are any missing dependencies.

```bash
conda create -n iex python=3.6
source activate iex
pip install bokeh
pip install pandas
```

Now, use pip to export the information about your environment to a requirements.txt file. Heroku will use this file to set up your remote machine when you deploy.

```bash
pip freeze > requirements.txt
```

Next, create a runtime.txt file. In this file, you will identify the version of python to use for the application. At the time this was written Heroku only supports two python runtimes: python-2.7.13 and python-3.6.0.

```bash
touch runtime.txt
```

```text
# runtime.txt
python-3.6.0
```

If you would like, you can add a README file.

```
touch README.md
```

Finally, you need to create a Procfile. This file contains the instructions for launching your application from Heroku. In the simplest case, it can be a single line long.

```bash
touch Procfile
```

```
# Procfile
web: bokeh serve --port $PORT --host iex-streaming.herokuapp.com --address=0.0.0.0 --use-xheaders iex.py
```

This step was the only part of the deployment where I hit a snag. I ended up relying heavily on this [stackoverflow question](http://stackoverflow.com/questions/38417200/) to work out the last issues I was having.

Starting on the left, ```web:``` is a directive to Heroku indicating the type of process that the command is controlling. In this case, the app requires a single web process. Everything afterwards is the shell command that gets executed to start the app. If you read the earlier post,  ```bokeh serve``` should be familiar to you. Bokeh comes with a CLI and ```serve``` is the command that fires up the bokeh server. Everything else is an argument to this command. ```--port $PORT --host iex-streaming.herokuapp.com``` instructs bokeh to listen for requests sent to iex-streaming.herokuapp.com on the port defined by the environment variable ```$PORT```, which is controlled by Heroku. ```--address=0.0.0.0``` tells bokeh to listen on all network interfaces for requests. ```--use-xheaders``` is a flag indicating that bokeh should use X-headers for IP/protocol information (according to the man page). Networking is not a strong suit of mine (yet... I'll get there one day), so unfortunately, that last part is still a black box for me.


With all of those files in place, we are just about finished. Your repo should contain the following files:

1. iex.py
2. Procfile
3. requirements.txt
4. runtime.txt
5. README.md

Go ahead and add/commit/push. If you are using Heroku Git, your push will be slightly different.

```
git add .
git commit -m "Starter files for app"
git push -u origin master
```

As soon as you push your changes, Heroku will re-deploy your application with the code changes. You can see the build results in your Heroku dashboard and after a minute or two, your changes will be live. Go to your app url and check it out! My sample version is available at:

[http://iex-streaming.herokuapp.com/](http://iex-streaming.herokuapp.com/)