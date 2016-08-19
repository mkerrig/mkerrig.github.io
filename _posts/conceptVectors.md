---
layout: post
title: Concept Vectors
---

I'm writing this post to flesh out my thoughts and current work with an idea that I have been playing around with for quite awhile:  
*Concept Vectors*  
The idea is inspired by the idea that we can actually hold concepts and ideas in a vector space model, as demonstrated with the idea of Thought Vectors, yet another beautiful creation from Geoff Hinton.  
A quick summary of Thought vectors and their creator in case one is not familiar. First and foremost, Geoff Hinton, is a boss ass dude, he's one of the key people who helped design the back propagating
neural network back in the 70's. Now he's head of one of Google's AI labs, where he helped desgin the Google translate algorithm, which consequently seems to be the birth of what we know as "Thought Vectors".
Thought vectors, as described by Geoff Hinton, are more general semantic representations beyond just individual words, ideally thought vectors contain human logic and reasoning.
Now realistically Thought Vectors don't really exist in their ideal state, but there are versions that do exist as of now. The OG Thought Vector, is the vector space produced from the LTSM
Google Translate model, which is basically two recurrent neural networks sitting on both sides of a languagless word/phrase vector space. 
This vector space can be translated into from any language, and once you have language represented in this "languagless wordvector space" you can translate to any other language. 
This is the first example of the idea of thought vectors, because these vectors transcend language, and can represent ideas, for example there are words in Russian that don't directly translate to
english, you can describe the *idea* behind this russian word, however it's not a word2word translation, this *idea* is represented in this space, not the specific word or language. 
Now this is really f***ing cool! We have a relational space that now stores very simple human thought, beyond just simple words! Now my question, and current obsession, is how do we improve upon this simple idea?
How do we even realistically store thought? My goal is to try and do just that.



```python
from bs4 import BeautifulSoup
import requests 
from pprint import pprint
import os.path
from itertools import groupby
import datetime as dt
import time
import csv
```
Now back to the alphabetical index.

This is what it looks like:

<http://www.boxofficemojo.com/movies/alphabetical.htm?letter=A&p=.htm>



