---
title: 2017 Retrospective and 2018 Goals

categories:
    - miscellaneous
tags:
    - self-study
    - personal
---
2017 went by in a hurry. My wife and I made the move from Chicago to Philadelphia for her to start a PhD program, which also entailed me switching jobs. After some initial adjustment, work is going well and I have found myself continuing to enjoy writing code for a living. Since switching away from economics research as a career track in 2016, I have been trying to sort out exactly how far towards software engineering I want to veer. My current work is a mixture of model building/validation/testing and more traditional programming projects. Quite honestly, this mixture suits me well, although I often worry about being a jack of all trades and master of none. Although I have no regrets about moving away from the academic research career path, I do wonder whether I'll need to go back to grad school for people to take my quantitative skills seriously. In the meantime, the plan for 2018 is to continue filling in gaps in my technical background, while adding some more tools to my applied statistics belt.

### Topics

- Containers & Docker -- I worked through a few tutorials and set up my own container for running a small webserver that runs a service which builds 3D brain visualizations using a few somehwat challenging to install pieces of software. I wish I had taken a deeper dive to understand more about *how* containers work. 

- Bayesian Statistics -- I had the chance to do some applied work in this area through my current job by building some bayesian hierarchical models that we use in production, so I ended up on a pretty deep dive on this one. I'll probably put together shorter blog pointing out some of the helpful resources I found/used in the process.

- ~~Time Series Analysis~~ -- Although some of the analysis I do at work has a strong time series component, I did not end up pursuing any side projects like I had originally planned. I'll call this one a failed goal.

- Web programming -- Throughout the year, I worked on 4 web-based applications. The first one was a job search app that would take a job title and city, scrape indeed, and store results in a database for later review. I built it with Flask and Mongo DB and used it when I was job hunting in the early part of the year. I ended up adding on to this with by using airflow to automatically search for a set of job titles and cities every hour, which I would then review nightly to see if anything interesting popped out. Interestingly enough, the job I have now was actually one of the results returned during this automated web scraping phase. The second application is an application that I initially built in the first few weeks of my current job that we use for viewing 3D brain visualizations, running ad-hoc queries to our database, and for viewing PDF reports related to the experiments we run. The third project I started working on with my wife since we are both foodies and enjoy cooking together. The idea is to build a social-networking site for food lovers. The initial prototype is designed primarily to make it easier for people to digitize the recipes they commonly use. The last project came out of a civic tech hackathon hosted by the [Reentry Project](https://thereentryproject.org/). I worked with a group of people to prototype a web application that is designed to match mentors with re-entering citizens. One nice thing was that the use case was similar enough to the foodie website that I was able to re-use most of the code. I've continued to work with a subset of that initial group to move the prototype closer to production, which will hopefully be finished some time in 2018.

- Data Pipelines -- Largely because of my current work, I've worked with 4 different 'libraries' for building data-pipelines: airflow, luigi, joblib, and a homegrown solution. While they all solve similar problems, after working with them to solve real problems, I now have a clearer understanding of the best use cases for each. As I'm thinking about it, there is probably a useful blog post to be written going through this in more detail. I came across a few while I was researching my options initially, but nothing that was as detailed as I would have liked looking back.

### Books

- [Cython](https://www.amazon.com/Cython-Programmers-Kurt-W-Smith/dp/1491901551/ref=sr_1_1?ie=UTF8&qid=1484107172&sr=8-1&keywords=cython) -- I remebmer finding the book helpful at the time, although since I did not do any side projects or work requiring cythonizing python code, I'm sad to say I've forgotten most of what was in there at this point.
- ~~[Big Data: Principles and best practices of scalable realtime data systems](https://www.amazon.com/Big-Data-Principles-practices-scalable/dp/1617290343/ref=sr_1_1?ie=UTF8&qid=1484107234&sr=8-1&keywords=big+data%3A+principles+and+best+practices)~~ -- I started reading this, but at the time, I did not have much of a chance to apply it professionally, so I cut the reading short.
- [Flask Web Development](https://www.amazon.com/Flask-Web-Development-Developing-Applications/dp/1449372627/ref=sr_1_1?ie=UTF8&qid=1484107257&sr=8-1&keywords=flask+web+development) -- Working through this book was how I built the job search application I described earlier. Looking back, this is the approach I should take when reading any technical book. It really helps to find a toy or side project that you can use to apply what you learn while reading. 
- [Think Bayes](https://www.amazon.com/Think-Bayes-Bayesian-Statistics-Python/dp/1449370780/ref=sr_1_1?ie=UTF8&qid=1484107459&sr=8-1&keywords=think+bayes) -- This was one of the first books I read in 2017 and sadly I don't remember a ton about it other than that it was a helpful introduction to the topic of Bayesian statistics, but that I would have liked it to be a bit more technical. However, the book was intended to be for programmers looking to learn Bayesian statistics.
- [Data Analysis Using Regression and Multilevel/Hierarchical Models](https://www.amazon.com/Analysis-Regression-Multilevel-Hierarchical-Models/dp/052168689X/ref=sr_1_1?ie=UTF8&qid=1484107411&sr=8-1&keywords=data+analysis+using+regression+and+multilevel+hierarchical+models) -- This book was constantly by my side as I was building the hierarchical models I mentioned earlier. This is the book I would recommend to someone who has had an introductory statistics course and wants a crash course in applied statistics.
- ~~[Bayesian Data Analysis](https://www.amazon.com/Bayesian-Analysis-Chapman-Statistical-Science/dp/1439840954/ref=sr_1_1?ie=UTF8&qid=1484107432&sr=8-1&keywords=bayesian+data+analysis)~~ (Pushed to 2018)
- [Statistical Rethinking](https://www.amazon.com/Statistical-Rethinking-Bayesian-Examples-Chapman/dp/1482253445/ref=sr_1_1?ie=UTF8&qid=1514759892&sr=8-1&keywords=statistical+rethinking): By far my favorite technical book of 2017. Although I regret not working through the R-based examples that are sprinkled throughout the text, I found the explanations of concepts incredibly clear and consise. After reading it, I feel like I now have the intuition and level of understanding to really dive in to the topic of bayesian data analysis.
- [Two Scoops of Django](https://www.amazon.com/Two-Scoops-Django-1-11-Practices/dp/0692915729/ref=sr_1_1?s=books&ie=UTF8&qid=1514759910&sr=1-1&keywords=two+scoops+of+django): Extremely helpful in learning how to effectively and idiomatically build Django-based web applications.
- [Analyzing Neural Time Series Data](https://www.amazon.com/Analyzing-Neural-Time-Data-Neuropsychology/dp/0262019876/ref=sr_1_1?s=books&ie=UTF8&qid=1514759926&sr=1-1&keywords=analyzing+neural+time+series+data) -- I read this as part of getting up to speed for my current work. One of the highlights for me was an intuitive explanation of Fourier transforms.
- [The Mythical Man-Month](https://www.amazon.com/Mythical-Man-Month-Software-Engineering-Anniversary/dp/0201835959/ref=sr_1_1?s=books&ie=UTF8&qid=1514759942&sr=1-1&keywords=the+mythical+man+month) -- A classic for those who are interested in how software gets made. I was pleasantly surprised to find that much of the advice/observations apply as much to building software in the 21st century as they did when the book was originally published in 1975.

### Projects

- Set up home workstation as a server
- Contribute to an open source project
- Build a docker container image
- ~~Compete in Kaggle's Two Sigma code competition~~
- Build a simple analytics web app

### Coursework

- [Scaling Python for Big Data](https://www.safaribooksonline.com/library/view/learning-path-scaling/9781491977804/)
- [Deep Learning](https://www.udacity.com/course/deep-learning--ud730)

## 2018

### Topics

- Operating Systems
- Networking
- C++
- Web Programming
- Bayesian Statistics

### Books

- [Modern Operating Systems](https://www.amazon.com/Modern-Operating-Systems-Andrew-Tanenbaum/dp/013359162X/ref=sr_1_1?s=books&ie=UTF8&qid=1514758673&sr=1-1&keywords=Operating+Systems+Tanenbaum)
- [Accelerated C++](https://www.amazon.com/gp/product/020170353X/ref=ox_sc_act_title_1?smid=ATVPDKIKX0DER&psc=1)
- [Effective Modern C++](https://www.amazon.com/Effective-Modern-Specific-Ways-Improve/dp/1491903996/ref=sr_1_1?s=books&ie=UTF8&qid=1514758711&sr=1-1&keywords=effective+modern+c%2B%2B)
- [Bayesian Data Analysis](https://www.amazon.com/Bayesian-Analysis-Chapman-Statistical-Science/dp/1439840954/ref=sr_1_1?ie=UTF8&qid=1484107432&sr=8-1&keywords=bayesian+data+analysis)

### Projects

- [MentorPhilly Web App](https://mentorphilly.herokuapp.com/)
- [MyRecipeStash Web App](http://www.myrecipestash.com/)
- Homework assignments for Software Systems course
- Submit PR revising logging in luigi

### Coursework

- [Software Systems](http://www.cis.upenn.edu/~cis505/)

