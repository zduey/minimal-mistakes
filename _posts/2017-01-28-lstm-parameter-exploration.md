---
title: Understanding LSTM Model Parameters
categories:
    - snippets
tags:
    - self-study
    - machine learning
---

For the past few weeks, I have been working through the Udacity Deep Learning course assignments. The first 4 exercises are relatively straightfowarded, but exercises 5 and 6 are where things really start to get interesting.  I tried to approach the assignment by developing a high level understanding of LSTMs so that I would have some intuition about them. I'd recommend [Chris Olah's blog post on LSTMs](http://colah.github.io/posts/2015-08-Understanding-LSTMs/) as the best starting point. 

After doing some reading and looking through the starter code provided, one problem that remained was in understanding how the different model parameters would affect model accuracy. In beginning the assignment, I had a few guesses about how the different parameters would affect model accuracy as well as a few open questions. I've outline those below and included what I found out through this exploration at the bottom of the post. 

The code for the gist embeded below comes directly out of the 6_lstm.ipynb notebook (with some modification) available on the course [github page](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/udacity). I apolgoize to any coders out there for the ugly way I turned the model setup and computation into a single function.


### Hypotheses:
1. Increasing the number of unrollings will reduce the validation set perplexity
2. Increasing num_nodes/batch_size will not reduce the validation set perplexity
3. Increasing the number of epochs will reduce the perplexity
4. The number of nodes must be equal to the batch size


### Questions:
1. How is the validation set perplexity affected by jointly changing num_nodes and num_unrollings?
    - Best guess: Ski-slope surface. I realize this is inconsistent with the hypotheses above.
2. Can you have more unrollings than nodes? What happens?
    - Best guess: Yes, but I'm not sure how this affects perplexity.
3. Can you have more unrollings than epochs? What happens?
    - Best Guess: It's the same as if the model had no unrollings.



<script src="https://gist.github.com/zduey/d4e5eed6f099997b0a6af1ef0962b960.js"></script>
[Source Code](https://gist.github.com/zduey/d4e5eed6f099997b0a6af1ef0962b960)

## Summary

### Hypotheses:
1. Increasing the number of unrollings will reduce the validation set perplexity
    - Correct: However, after about 15 unrollings, the effect diminishes.
2. Increasing num_nodes/batch_size will not reduce the validation set perplexity
    - Incorrect: As you increase either one, the perplexity decreases, though the effect asymptotes out around 64 for the batch size and 128 for the number of nodes.
3. Increasing the number of epochs will reduce the perplexity.
    - Correct: It's not a monotonically decreasing function like the others, but it is generally getting smaller.
4. The number of nodes must be equal to the batch size.
    - Incorrect


### Questions:
1. How is the validation set perplexity affected by jointly changing num_nodes and num_unrollings?
    - Best guess: Ski-slope surface. I realize this is inconsistent with the hypotheses above.
    - Correct: It is definitely the ski slope I had in mind.
2. Can you have more unrollings than nodes? What happens?
    - Best guess: Yes, but I'm not sure how this affects perplexity.
    - Correct: The model runs and the higher number of unrollings reduces the final perplexity.
3. Can you have more unrollings than epochs? What happens?
    - Best Guess: It's the same as if the model had no unrollings.
    - Partialy-correct: It is possible to run the model, but it's not obvious if the unrollings have an effect on the validation set perplexity. More unrollings than epochs results in slightly higher perplexity than a model with the same paramters but just 1 unrolling.        

## Final Thoughts

To me, the parameters in the assignment were a total black box. The fact that num_nodes and batch_size were equal (which now just seems to be coincidence) led me to the incorrect assumption that they always had to be equal. Looking at the above results makes it clearer why they chose the default parameters that they did. It's almost as if they did this exercise in order to choose the parameeters.


#### Resources
1. [Udacity Deep Learning Course](https://www.udacity.com/course/deep-learning--ud730)
2. [Udacity Deep Learning Notebooks](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/udacity)
3. [Chris Olah's Blog](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)