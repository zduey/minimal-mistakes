---
title: Understanding LSTM Model Parameters
categories:
    - machine-learning
tags:
    - personal
    - learning
    - ML
---

# Understanding LSTM Parameterization

Working through the final exerise of Udacity Deep learning has been quite a challenge. I tried to approach the assignment by developing a high level understanding of LSTMs so that I would have some intuition about them. One problem I faced was in understanding how the different model parameters would affect model accuracy. This notebook is an exercise in exploring how tweaking the model parameters independently (and in one case jointly) affects the accuracy of the model. The code below is directly out of the 6_lstm.ipynb notebook (with some modification) available on the course [github page](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/udacity). I apolgoize to any coders out there for the ugly way I turned the model setup and computation into a single function.

Resource Links

1. [Udacity Deep Learning Course](https://www.udacity.com/course/deep-learning--ud730)
2. [Udacity Deep Learning Notebooks](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/udacity)
3. [Chris Olah's Blog](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)

### Hypotheses:
1. Increasing the number of unrollings will reduce the validation set perplexity ceteris paribus.
2. Increasing num_nodes/batch_size will not reduce the validation set perplexity ceteris paribus.
3. Increasing the number of epochs will reduce the perplexity.
4. The number of nodes must be equal to the batch size.


### Questions:
1. How is the validation set perplexity affected by jointly changing num_nodes and num_unrollings?
    - Best guess: Ski-slope surface. I realize this is inconsistent with the hypotheses above.
2. Can you have more unrollings than nodes? What happens?
    - Best guess: Yes, but I'm not sure how this affects perplexity.
3. Can you have more unrollings than epochs? What happens?
    - Best Guess: It's the same as if the model had no unrollings.

## Setup


```python
from __future__ import print_function
import os
import numpy as np
import random
import string
import tensorflow as tf
import matplotlib.pyplot as plt
import zipfile
import itertools
from six.moves import range
from six.moves.urllib.request import urlretrieve
from mpl_toolkits.mplot3d import Axes3D
from matplotlib import cm

%matplotlib inline

url = 'http://mattmahoney.net/dc/'

def maybe_download(filename, expected_bytes):
    """Download a file if not present, and make sure it's the right size."""
    if not os.path.exists(filename):
        filename, _ = urlretrieve(url + filename, filename)
    statinfo = os.stat(filename)
    if statinfo.st_size == expected_bytes:
        print('Found and verified %s' % filename)
    else:
        print(statinfo.st_size)
        raise Exception(
            'Failed to verify ' + filename + '. Can you get to it with a browser?')
    return filename

filename = maybe_download('text8.zip', 31344016)

def read_data(filename):
    f = zipfile.ZipFile(filename)
    for name in f.namelist():
        return tf.compat.as_str(f.read(name))
    f.close()
    
text = read_data(filename)

valid_size = 1000
valid_text = text[:valid_size]
train_text = text[valid_size:]
train_size = len(train_text)

vocabulary_size = len(string.ascii_lowercase) + 1 # [a-z] + ' '
first_letter = ord(string.ascii_lowercase[0])

def char2id(char):
    if char in string.ascii_lowercase:
        return ord(char) - first_letter + 1
    elif char == ' ':
        return 0
    else:
        print('Unexpected character: %s' % char)
        return 0
    
def id2char(dictid):
    if dictid > 0:
        return chr(dictid + first_letter - 1)
    else:
        return ' '

def logprob(predictions, labels):
    """Log-probability of the true labels in a predicted batch."""
    predictions[predictions < 1e-10] = 1e-10
    return np.sum(np.multiply(labels, -np.log(predictions))) / labels.shape[0]

def sample_distribution(distribution):
    """Sample one element from a distribution assumed to be an array of normalized
    probabilities.
    """
    r = random.uniform(0, 1)
    s = 0
    for i in range(len(distribution)):
        s += distribution[i]
        if s >= r:
            return i
    return len(distribution) - 1

def sample(prediction):
    """Turn a (column) prediction into 1-hot encoded samples."""
    p = np.zeros(shape=[1, vocabulary_size], dtype=np.float)
    p[0, sample_distribution(prediction[0])] = 1.0
    return p

def random_distribution():
    """Generate a random column of probabilities."""
    b = np.random.uniform(0.0, 1.0, size=[1, vocabulary_size])
    return b/np.sum(b, 1)[:,None]

def characters(probabilities):
    """Turn a 1-hot encoding or a probability distribution over the possible
    characters back into its (most likely) character representation."""
    return [id2char(c) for c in np.argmax(probabilities, 1)]

def batches2string(batches):
    """Convert a sequence of batches back into their (most likely) string
    representation."""
    s = [''] * batches[0].shape[0]
    for b in batches:
        s = [''.join(x) for x in zip(s, characters(b))]
    return s

def lstm_model_test(batch_size, num_nodes, num_unrollings, num_steps, summary_frequency):
    class BatchGenerator(object):
        def __init__(self, text, batch_size, num_unrollings):
            self._text = text
            self._text_size = len(text)
            self._batch_size = batch_size
            self._num_unrollings = num_unrollings
            segment = self._text_size // batch_size # length of the word
            self._cursor = [ offset * segment for offset in range(batch_size)]
            self._last_batch = self._next_batch()

        def _next_batch(self):
            """Generate a single batch from the current cursor position in the data."""
            batch = np.zeros(shape=(self._batch_size, vocabulary_size), dtype=np.float)
            for b in range(self._batch_size):
                batch[b, char2id(self._text[self._cursor[b]])] = 1.0
                self._cursor[b] = (self._cursor[b] + 1) % self._text_size
            return batch

        def next(self):
            """Generate the next array of batches from the data. The array consists of
            the last batch of the previous array, followed by num_unrollings new ones.
            """
            batches = [self._last_batch]
            for step in range(self._num_unrollings):
                batches.append(self._next_batch())
            self._last_batch = batches[-1]
            return batches

    train_batches = BatchGenerator(train_text, batch_size, num_unrollings)
    valid_batches = BatchGenerator(valid_text, 1, 1)
    graph = tf.Graph()
    with graph.as_default():

            # Parameters:
            # Input gate: input, previous output, and bias.
            ix = tf.Variable(tf.truncated_normal([vocabulary_size, num_nodes], -0.1, 0.1))
            im = tf.Variable(tf.truncated_normal([num_nodes, num_nodes], -0.1, 0.1))
            ib = tf.Variable(tf.zeros([1, num_nodes]))
            # Forget gate: input, previous output, and bias.
            fx = tf.Variable(tf.truncated_normal([vocabulary_size, num_nodes], -0.1, 0.1))
            fm = tf.Variable(tf.truncated_normal([num_nodes, num_nodes], -0.1, 0.1))
            fb = tf.Variable(tf.zeros([1, num_nodes]))
            # Memory cell: input, state and bias.                                                         
            cx = tf.Variable(tf.truncated_normal([vocabulary_size, num_nodes], -0.1, 0.1))
            cm = tf.Variable(tf.truncated_normal([num_nodes, num_nodes], -0.1, 0.1))
            cb = tf.Variable(tf.zeros([1, num_nodes]))
            # Output gate: input, previous output, and bias.
            ox = tf.Variable(tf.truncated_normal([vocabulary_size, num_nodes], -0.1, 0.1))
            om = tf.Variable(tf.truncated_normal([num_nodes, num_nodes], -0.1, 0.1))
            ob = tf.Variable(tf.zeros([1, num_nodes]))
            # Variables saving state across unrollings.
            saved_output = tf.Variable(tf.zeros([batch_size, num_nodes]), trainable=False)
            saved_state = tf.Variable(tf.zeros([batch_size, num_nodes]), trainable=False)
            # Classifier weights and biases.
            w = tf.Variable(tf.truncated_normal([num_nodes, vocabulary_size], -0.1, 0.1))
            b = tf.Variable(tf.zeros([vocabulary_size]))

            # Definition of the cell computation.
            def lstm_cell(i, o, state):
                """Create a LSTM cell. See e.g.: http://arxiv.org/pdf/1402.1128v1.pdf
                Note that in this formulation, we omit the various connections between the
                previous state and the gates."""
                input_gate = tf.sigmoid(tf.matmul(i, ix) + tf.matmul(o, im) + ib)
                forget_gate = tf.sigmoid(tf.matmul(i, fx) + tf.matmul(o, fm) + fb)
                update = tf.matmul(i, cx) + tf.matmul(o, cm) + cb
                state = forget_gate * state + input_gate * tf.tanh(update)
                output_gate = tf.sigmoid(tf.matmul(i, ox) + tf.matmul(o, om) + ob)
                return output_gate * tf.tanh(state), state

            # Input data.
            train_data = list()
            for _ in range(num_unrollings + 1):
                    train_data.append(
                        tf.placeholder(tf.float32, shape=[batch_size,vocabulary_size]))
                    train_inputs = train_data[:num_unrollings]
                    train_labels = train_data[1:]    # labels are inputs shifted by one time step.

            # Unrolled LSTM loop.
            outputs = list()
            output = saved_output
            state = saved_state
            for i in train_inputs:
                    output, state = lstm_cell(i, output, state)
                    outputs.append(output)

            # State saving across unrollings.
            with tf.control_dependencies([saved_output.assign(output),saved_state.assign(state)]):
                # Classifier.
                logits = tf.nn.xw_plus_b(tf.concat(0, outputs), w, b)
                loss = tf.reduce_mean(
                    tf.nn.softmax_cross_entropy_with_logits(
                        logits, tf.concat(0, train_labels)))

            # Optimizer.
            global_step = tf.Variable(0)
            learning_rate = tf.train.exponential_decay(
            10.0, global_step, 5000, 0.1, staircase=True)
            optimizer = tf.train.GradientDescentOptimizer(learning_rate)
            gradients, v = zip(*optimizer.compute_gradients(loss))
            gradients, _ = tf.clip_by_global_norm(gradients, 1.25)
            optimizer = optimizer.apply_gradients(
            zip(gradients, v), global_step=global_step)

            # Predictions.
            train_prediction = tf.nn.softmax(logits)

            # Sampling and validation eval: batch 1, no unrolling.
            sample_input = tf.placeholder(tf.float32, shape=[1, vocabulary_size])
            saved_sample_output = tf.Variable(tf.zeros([1, num_nodes]))
            saved_sample_state = tf.Variable(tf.zeros([1, num_nodes]))
            reset_sample_state = tf.group(
            saved_sample_output.assign(tf.zeros([1, num_nodes])),
            saved_sample_state.assign(tf.zeros([1, num_nodes])))
            sample_output, sample_state = lstm_cell(sample_input, saved_sample_output, saved_sample_state)
            with tf.control_dependencies([saved_sample_output.assign(sample_output),
                                         saved_sample_state.assign(sample_state)]):
                    sample_prediction = tf.nn.softmax(tf.nn.xw_plus_b(sample_output, w, b))

    # Training the model
    with tf.Session(graph=graph) as session:
        tf.initialize_all_variables().run()
        mean_loss = 0
        for step in range(num_steps):
            batches = train_batches.next()
            feed_dict = dict()
            for i in range(num_unrollings + 1):
                feed_dict[train_data[i]] = batches[i]

            _, l, predictions, lr = session.run([optimizer, loss, train_prediction, learning_rate], feed_dict=feed_dict)
            mean_loss += l
        reset_sample_state.run()
        valid_logprob = 0
        for _ in range(valid_size):
            b = valid_batches.next()
            predictions = sample_prediction.eval({sample_input: b[0]})
            valid_logprob = valid_logprob + logprob(predictions, b[1])

        perplexity = float(np.exp(valid_logprob / valid_size))
    return perplexity
```


```python
lstm_model_test(64, 64, 10, 3001, 1000)
```

	4.675105314113517


## Testing

Hypothesis: The number nodes must be equal to the batch size.

Result: Incorrect --The number of nodes and the batch size can be different.


```python
lstm_model_test(32, 64, 10, 3001, 1000)
```




    4.913481800643745




```python
lstm_model_test(64, 32, 10, 3001, 1000)
```




    5.748203599403953



Question: Can you have more unrollings than number of epochs?

Result: Yes, however, it is not completely clear if it is different (aside from some randomness) than  if you had just 1 unrolling. Note: You cannot have 0 unrollings.


```python
lstm_model_test(64, 64, 100, 65, 1000)
```




    11.477824989981638




```python
lstm_model_test(64, 64, 1, 65, 1000)
```




    13.85651774565771



Question:  Can you have more unrollings than nodes?

Result:  Yes, and it seems like the number of nodes places a significant role in reducing validation set perplexity.


```python
lstm_model_test(64, 12, 15, 3001, 1000)
```




    7.328228938264665



Hypothesis: Increasing the num_nodes/batch_size will not reduce the perplexity. Given the above, this rests on the hidden assumption that the number of nodes and the bath size must be equal. Let's test them independently since I've already demonstrated that hidden assumption was incorrect.
Result: Increasing the number of nodes or the batch size (holding everything else constant) results in lower perplexity. In fact, the validation set perplexity is a monotonically decreassing function of these parameters.


```python
batch_sizes = [2, 4, 8, 16, 32, 64, 128, 256]
batch_size_results = []
for batch_size in batch_sizes:
    print("Testing batch size ", batch_size)
    result = lstm_model_test(batch_size, 64, 10, 3001, 10000)
    batch_size_results.append(result)
```

    Testing batch size  2
    Testing batch size  4
    Testing batch size  8
    Testing batch size  16
    Testing batch size  32
    Testing batch size  64
    Testing batch size  128
    Testing batch size  256



```python
plt.plot(batch_sizes, batch_size_results, label='', linewidth=2)
plt.xlabel('Batch Size')
plt.ylabel('Perplexity')
plt.title("Batch Size and Perplexity")
```


{% raw %}![Batch Size]({{ site.url }}/images/batch_size.png){% endraw %}



```python
num_nodes = [2, 4, 8, 16, 32, 64, 128, 256]
num_nodes_results = []
for nodes in num_nodes:
    print("Testing number of nodes: ", nodes)
    result = lstm_model_test(64, nodes, 10, 3001, 10000)
    num_nodes_results.append(result)
```

    Testing number of nodes:  2
    Testing number of nodes:  4
    Testing number of nodes:  8
    Testing number of nodes:  16
    Testing number of nodes:  32
    Testing number of nodes:  64
    Testing number of nodes:  128
    Testing number of nodes:  256



```python
plt.plot(num_nodes, num_nodes_results, linewidth=2)
plt.xlabel('Number of Nodes')
plt.ylabel('Perplexity')
plt.title("Number of Nodes and Perplexity")
```


{% raw %}![Number of Nodes]({{ site.url }}/images/nodes.png){% endraw %}


Hypothesis: Increasing the number of unrollings will reduce the validation set perplexity.
Result:  Correct --Like the number of nodes and batch size, increasing the number of unrollings reduces the validation set perplexity. However, it seems like after a certain point, this effect disapears, and may actually result in an increasing validation set perplexity.


```python
num_unrollings = [1,2,3,4,5,10,15,20,25,30,50]
num_unrollings_results = []
for unrollings in num_unrollings:
    print("Testing number of unrollings", unrollings)
    result = lstm_model_test(64, 64, unrollings, 3001, 10000)
    num_unrollings_results.append(result)
```

    Testing number of unrollings 1
    Testing number of unrollings 2
    Testing number of unrollings 3
    Testing number of unrollings 4
    Testing number of unrollings 5
    Testing number of unrollings 10
    Testing number of unrollings 15
    Testing number of unrollings 20
    Testing number of unrollings 25
    Testing number of unrollings 30
    Testing number of unrollings 50



```python
plt.plot(num_unrollings, num_unrollings_results, linewidth=2)
plt.xlabel('Number of Unrollings')
plt.ylabel('Perplexity')
plt.title("Number of Unrollings and Perplexity")
```

{% raw %}![Number of Unrollings]({{ site.url }}/images/unrollings.png){% endraw %}


Hypothesis: Increasing the number of epochs will reduce the perplexity

Result: Increasing the number of epochs does result in a generally decreasing perplexity, however it is not a monotonically decreasing function.


```python
epochs = [1001, 2001, 3001, 4001, 5001, 6001, 7001, 8001, 9001, 10001]
epoch_results = []
for epoch in epochs:
    print("Testing number of epochs", epoch)
    result = lstm_model_test(64, 64, 10, epoch, 10000)
    epoch_results.append(result)
```

    Testing number of epochs 1001
    Testing number of epochs 2001
    Testing number of epochs 3001
    Testing number of epochs 4001
    Testing number of epochs 5001
    Testing number of epochs 6001
    Testing number of epochs 7001
    Testing number of epochs 8001
    Testing number of epochs 9001
    Testing number of epochs 10001



```python
plt.plot(epochs, epoch_results, linewidth=2)
plt.xlabel('Number of Epochs')
plt.ylabel('Perplexity')
plt.title("Number of Epochs and Perplexity")
```

{% raw %}![Number of Epochs]({{ site.url }}/images/epochs.png){% endraw %}


Question: How is the model accuracy (validation set perplexity) affected by the joint changing of num_nodes and num_unrollings?
Result: The resulting surfacie is a "ski slope." As you increase both the number of nodes and the number of unrollings, the perplexity decreases.


```python
num_unrollings = [1,2,3,4,5,10,15,20,25,30,50]
num_nodes = [2, 4, 8, 16, 32, 64, 128, 256]
num_epochs = [3001]
batch_sizes = [64]
summary = [10000]

params = list(itertools.product(batch_sizes, num_nodes, num_unrollings, num_epochs, summary))
params_results = []
for params in params:
    print("Testing paramters: ", params)
    result = lstm_model_test(*params)
    params_results.append(result)
```

    Testing paramters:  (64, 2, 1, 3001, 10000)
    Testing paramters:  (64, 2, 2, 3001, 10000)
    Testing paramters:  (64, 2, 3, 3001, 10000)
    Testing paramters:  (64, 2, 4, 3001, 10000)
    Testing paramters:  (64, 2, 5, 3001, 10000)
    Testing paramters:  (64, 2, 10, 3001, 10000)
    Testing paramters:  (64, 2, 15, 3001, 10000)
    Testing paramters:  (64, 2, 20, 3001, 10000)
    Testing paramters:  (64, 2, 25, 3001, 10000)
    Testing paramters:  (64, 2, 30, 3001, 10000)
    Testing paramters:  (64, 2, 50, 3001, 10000)
    Testing paramters:  (64, 4, 1, 3001, 10000)
    Testing paramters:  (64, 4, 2, 3001, 10000)
    Testing paramters:  (64, 4, 3, 3001, 10000)
    Testing paramters:  (64, 4, 4, 3001, 10000)
    Testing paramters:  (64, 4, 5, 3001, 10000)
    Testing paramters:  (64, 4, 10, 3001, 10000)
    Testing paramters:  (64, 4, 15, 3001, 10000)
    Testing paramters:  (64, 4, 20, 3001, 10000)
    Testing paramters:  (64, 4, 25, 3001, 10000)
    Testing paramters:  (64, 4, 30, 3001, 10000)
    Testing paramters:  (64, 4, 50, 3001, 10000)
    Testing paramters:  (64, 8, 1, 3001, 10000)
    Testing paramters:  (64, 8, 2, 3001, 10000)
    Testing paramters:  (64, 8, 3, 3001, 10000)
    Testing paramters:  (64, 8, 4, 3001, 10000)
    Testing paramters:  (64, 8, 5, 3001, 10000)
    Testing paramters:  (64, 8, 10, 3001, 10000)
    Testing paramters:  (64, 8, 15, 3001, 10000)
    Testing paramters:  (64, 8, 20, 3001, 10000)
    Testing paramters:  (64, 8, 25, 3001, 10000)
    Testing paramters:  (64, 8, 30, 3001, 10000)
    Testing paramters:  (64, 8, 50, 3001, 10000)
    Testing paramters:  (64, 16, 1, 3001, 10000)
    Testing paramters:  (64, 16, 2, 3001, 10000)
    Testing paramters:  (64, 16, 3, 3001, 10000)
    Testing paramters:  (64, 16, 4, 3001, 10000)
    Testing paramters:  (64, 16, 5, 3001, 10000)
    Testing paramters:  (64, 16, 10, 3001, 10000)
    Testing paramters:  (64, 16, 15, 3001, 10000)
    Testing paramters:  (64, 16, 20, 3001, 10000)
    Testing paramters:  (64, 16, 25, 3001, 10000)
    Testing paramters:  (64, 16, 30, 3001, 10000)
    Testing paramters:  (64, 16, 50, 3001, 10000)
    Testing paramters:  (64, 32, 1, 3001, 10000)
    Testing paramters:  (64, 32, 2, 3001, 10000)
    Testing paramters:  (64, 32, 3, 3001, 10000)
    Testing paramters:  (64, 32, 4, 3001, 10000)
    Testing paramters:  (64, 32, 5, 3001, 10000)
    Testing paramters:  (64, 32, 10, 3001, 10000)
    Testing paramters:  (64, 32, 15, 3001, 10000)
    Testing paramters:  (64, 32, 20, 3001, 10000)
    Testing paramters:  (64, 32, 25, 3001, 10000)
    Testing paramters:  (64, 32, 30, 3001, 10000)
    Testing paramters:  (64, 32, 50, 3001, 10000)
    Testing paramters:  (64, 64, 1, 3001, 10000)
    Testing paramters:  (64, 64, 2, 3001, 10000)
    Testing paramters:  (64, 64, 3, 3001, 10000)
    Testing paramters:  (64, 64, 4, 3001, 10000)
    Testing paramters:  (64, 64, 5, 3001, 10000)
    Testing paramters:  (64, 64, 10, 3001, 10000)
    Testing paramters:  (64, 64, 15, 3001, 10000)
    Testing paramters:  (64, 64, 20, 3001, 10000)
    Testing paramters:  (64, 64, 25, 3001, 10000)
    Testing paramters:  (64, 64, 30, 3001, 10000)
    Testing paramters:  (64, 64, 50, 3001, 10000)
    Testing paramters:  (64, 128, 1, 3001, 10000)
    Testing paramters:  (64, 128, 2, 3001, 10000)
    Testing paramters:  (64, 128, 3, 3001, 10000)
    Testing paramters:  (64, 128, 4, 3001, 10000)
    Testing paramters:  (64, 128, 5, 3001, 10000)
    Testing paramters:  (64, 128, 10, 3001, 10000)
    Testing paramters:  (64, 128, 15, 3001, 10000)
    Testing paramters:  (64, 128, 20, 3001, 10000)
    Testing paramters:  (64, 128, 25, 3001, 10000)
    Testing paramters:  (64, 128, 30, 3001, 10000)
    Testing paramters:  (64, 128, 50, 3001, 10000)
    Testing paramters:  (64, 256, 1, 3001, 10000)
    Testing paramters:  (64, 256, 2, 3001, 10000)
    Testing paramters:  (64, 256, 3, 3001, 10000)
    Testing paramters:  (64, 256, 4, 3001, 10000)
    Testing paramters:  (64, 256, 5, 3001, 10000)
    Testing paramters:  (64, 256, 10, 3001, 10000)
    Testing paramters:  (64, 256, 15, 3001, 10000)
    Testing paramters:  (64, 256, 20, 3001, 10000)
    Testing paramters:  (64, 256, 25, 3001, 10000)
    Testing paramters:  (64, 256, 30, 3001, 10000)
    Testing paramters:  (64, 256, 50, 3001, 10000)



```python
num_unrollings = [1,2,3,4,5,10,15,20,25,30,50]
num_nodes = [2, 4, 8, 16, 32, 64, 128, 256]
params = list(itertools.product(num_nodes, num_unrollings))

x = [t[0] for t in params]
y = [t[1] for t in params]
z = params_results
```


```python
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
ax.set_xlabel('Nodes')
ax.set_ylabel('Unrollings')
ax.plot_trisurf(x, y, z, linewidth=0)
```

{% raw %}![Surface Plot]({{ site.url }}/images/surface.png){% endraw %}


## Summary

### Hypotheses:
1. Increasing the number of unrollings will reduce the validation set perplexity ceteris paribus.
    - Correct: However, after about 15 unrollings, the effect diminishes.
2. Increasing num_nodes/batch_size will not reduce the validation set perplexity ceteris paribus.
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
