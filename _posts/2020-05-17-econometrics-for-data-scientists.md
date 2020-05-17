---
title: Econometrics for Data Scientists 

categories:
    - resources
tags:
    - econometrics
    - presentations
---

One of my favorite parts about being in an R&D group is that we have a periodic “Journal Club” where we all read a paper that is relevant to some aspect of projects we are working on, and then discuss. Last year, we were reading some papers about factors that contribute to Parkinson’s Disease (PD) progression. In some cases, the authors were leveraging observational data to make claims (often using causal language) about the relationship between the factors they were studying and the rate of disease PD progression. Given my educational and early professional background in econometrics, I read those papers expecting there to be a focus on establishing and defending an “identification strategy” -- the combination of theory and methodology that help justify a causal claim. To my surprise, these papers neither made such an effort nor leveraged the methods that I expected: difference in differences, instrumental variables, regression discontinuity, etc. 

In talking about these papers with my co-workers, it was clear that I was in the minority with my surprise at the lack of an identification strategy. This sparked the idea to put together some materials that I tentatively titled “Econometrics for Data Scientists.” Around the same time, I was also finishing up a course in Statistical Learning at Penn ([STAT-974](https://apps.wharton.upenn.edu/syllabi/2020A/STAT974401/)). The course emphasized the algorithmic nature of inferential methods and largely de-emphasized causal inference as an enterprise. However, towards the end of the course, we covered a technique called “[double ML](https://economics.mit.edu/files/12538)” -- a method for leveraging machine learning techniques for doing causal inference. The timing seemed too perfect to pass up: over the next few weeks I put together a series of [Jupyter notebooks](https://github.com/zduey/metrics) and [slides](https://docs.google.com/presentation/d/15I1aXxlZCDbVoqGoqPJNl6654gGP_FZrBE6nQMSBCsQ/edit?usp=sharing) covering some core econometric methods all the way through to some more recent methods that bridge the machine learning/causal inference divide. For anyone looking to learn more, the slides contain numerous links (organized by topic) to other useful books, articles, and blog posts.
