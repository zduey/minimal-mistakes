---
title: The Funnel Method

categories:
    - Tips
tags:
    - python
    - code style
    - advice
---

tl;dr Python script template for picky people

A while back, I saw the following post on Twitter about structuring Python scripts. 

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">A template for <a href="https://twitter.com/hashtag/Python?src=hash">#Python</a> scripting... <a href="https://twitter.com/swcarpentry">@swcarpentry</a> <a href="https://twitter.com/DataScienceHbt">@DataScienceHbt</a> <a href="https://t.co/D9DDoU6PrQ">pic.twitter.com/D9DDoU6PrQ</a></p>&mdash; Damien Irving (@DrClimate) <a href="https://twitter.com/DrClimate/status/779163490883731456">September 23, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I really liked the idea of a simple Python template, but I think there are two very easy (and important) improvements to be made. They may seem trivial, but stick with me through the explanation:

```python
# Import stuff

def what_main_func_does(args):
	""" Call a bunch of functions to do what the script does """

# define a bunch of functions

if __name__ == "__main__":
	# what to do if the script is being executed from the command line (instead of being imported)

	# use the argparse library to handle command line arguments (args)

	what_main_does(args)
```

The first change you may notice is that I renamed 'main' to somethig a bit more descriptive. This may seem nit-picky, but I think it is worth addressing in more detail because I see python files with main() all the time and I think it is a habit we all need to break. Ultimately, it comes down to one very simple fact: Python is not C, i.e. you are not required to have a function called 'main' as the entry point to a program. This is a very liberating aspect of the language as it means your top-level function can have a descriptive name keying readers in to what the script is doing.

The other change you will notice is that I put the main function at the top of the script with the function definitions below it. Again, this may seem nit-picky, however, there is a functional (pun intended) reason for doing this that I will get into in a moment. Putting function definitions ahead of your main function is another one of those habits that we all need to break once we have realized that Python is not C. In C, a function declaration must appear in a file before it is used. What this means is that if you wanted to put your 'main' function at the top of the script in C, you would have to put all of your function definitions ahead of it. This can start to look quite ugly if you have a lot of functions in the file, so I understand why most C code does not opt for this approach. As an example, consider the top of this C program implementing a hangman game:

```c
#include <stdbool.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <time.h>
#include "convert.h"

bool check_valid_input(char c);
bool pickword(char * buffer, size_t maxlength);
void showdiagram (unsigned int incorrect);
void showguesses (char *letter_positions, char *guesses, char *misses);
char readnext();
bool check_valid_input(char c);
bool is_valid_guess(char guess, char *guesses);


int main (int argc, char ** args)
{
...
}
```

On the one hand, it is not too hard to just glaze over those definitions, but I can definitely see why having too many more would get distracting and lead you to just put 'main' at the end of the file.

Aside from it being possible, there is also a best practices reason for putting  your top-level function first. Bear with me in this similie: writing code is like writing an essay. It is good practice to start with an introduction before getting into the heart of the argument. An introduction orients the reader and also gives lazy readers an easy point to tl;dr and move on to something else. The same is true for writing  code. Having your high level procedure at the top of the script gives readers of that code (provided you have descriptive function names) a good sense of the script's purpose. If the reader wants to get into the details, all they need to do is scroll down and continue reading. Keep in mind: this is not a new concept. The idea of having narrative-like code shows up in Kernighan and Plaugher's 1978 book [*The Elements of Programming Style*](https://en.wikipedia.org/wiki/The_Elements_of_Programming_Style). 

Robert Martin expands a bit upon this idea in his book [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1?ie=UTF8&qid=1477935335&sr=8-1&keywords=clean+code) where he describes *The Stepdown Rule*. At its core, *The Stepdown Rule* says that each function should descend one level of abstraction such that the code file can be read top to bottom. One way to visualize this principle was to imagine your code file as a funnel (hence the title of this post). The top contains the broadest function, equivalent to Damien's 'main' function in the template. As you get lower in the file, the functions become more and more detailed, all the while, doing exactly one thing. Ideally, what you end up with is code that is self-documenting and that takes the reader on a logical journey from high-level to the nitty-gritty.


```python
# Imports

def what_main_does(args):
    action_1()
    action_2()
    action_3()
    return

def action_1():
    # some actions
    _nitty_gritty_routine()
    return
.
.
.

def _nitty_gritty_routine():
    # nitty gritty details
    return

if __name__ == "__main__":
    # capture command line args
    args = sys.argv
    what_main_does(args)

```