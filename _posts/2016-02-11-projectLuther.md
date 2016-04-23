---
layout: post
title: Scraping boxofficemojo
---

This is going to be part 1 of a 3 part series of the different components of my analysis of movie data to predict revenue. This is going to focus on the scraping and preprocessing component of the anaylsis.

So a quick overview of my methodology when it comes to scraping. 

Here I am using Beautiful soup, however usually I prefer Scrapy, which is another python scraping framework. I use a tool called selector gadget to find CSS selectors for me, and if I don't get what I want that way, I use selector gadget to point me in the right direction, then just inspect the element and use CSS logic highlighted here: http://www.w3schools.com/cssref/css_selectors.asp 

_(Note: Not all the operations on this page can be used by most scraping libraries, specifically the nth-* commands other than nth-of-type, at least for beautiful soup)_

So first things first we need to goto boxofficemojo and figure out the logic in links so that we can scrape all the movie info on the site. 

After looking around for a short while we can quickly figure out that a good place to start is the alphabetical directory of all the movies on the site, so we start by writing a method to scrape this.

But first we import all our necessary resources for all our methods that we are going to write.

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

http://www.boxofficemojo.com/movies/alphabetical.htm?letter=A&p=.htm

Now if you want to see what I am seeing I suggest you go over an get the selector gadget chrome add-on.

https://chrome.google.com/webstore/detail/selectorgadget/mhjhnkcfbdhnjickkkdbjoemdmbfginb?hl=en

Click on the selector gadget icon and then click on the "Ad-Ad" and then deselect something else other than the letter ranges, what you will see down in the bottom right corner is this selector: **.alpha-nav-holder a**

So we then are going to use that selector in the method to get the links. Here is what that method looks like:

```python
def get_movie_alpha_index():
    """
    Scrapes the links of the alphabetical indices from the alphabetical
    directory
    """
    print 'Creating alpha index links'
    alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ' # Each letter for the URL 
    links = []
    f = open("alpha_index.txt", "w")
    f.write('/movies/alphabetical.htm?letter=NUM&p=.htm\n') # Scrape the titles that start with numbers
    base_url = 'http://www.boxofficemojo.com'
    for x in alphabet:
        end_url = '/movies/alphabetical.htm?letter='+x+'&p=.htm' 
        response = requests.get(base_url+end_url)
        soup = BeautifulSoup(response.content,'lxml')]
```
Now lets look at the link below because a lot is going on.

_*tmplink = list(set(soup.select('.alpha-nav-holder a')))*_

list(set()) because the selector selects the links twice, once at the bottom, once at the top. 
Notice the soup.select() instead of the more common soup.find(), select() allows you to select by  
CSS selector instead of selecting by parent or child, just select it directly.

Lets finsih up the method.
```python
        tmplink = list(set(soup.select('.alpha-nav-holder a'))) 
        link = [elem['href'] for elem in tmplink]
        link.append(end_url)
        link.sort()
        links.extend(link)
    [f.write(line+'\n') for line in links]
    f.close()
```

