---
layout: post
title: Scraping boxofficemojo
---

This is going to be part 1 of a 3 part series of the different components of my analysis of movie data to predict revenue. This is going to focus on the scraping and preprocessing component of the anaylsis.

So a quick overview of my methodology when it comes to scraping. 

Here I am using Beautiful soup, however usually I prefer Scrapy, which is another python scraping framework. I use a tool called selector gadget to find CSS selectors for me, and if I don't get what I want that way, I use selector gadget to point me in the right direction, then just inspect the element and use CSS logic highlighted here: [http://www.w3schools.com/cssref/css_selectors.asp] 

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

[http://www.boxofficemojo.com/movies/alphabetical.htm?letter=A&p=.htm]

Now if you want to see what I am seeing I suggest you go over an get the selector gadget chrome add-on.

[https://chrome.google.com/webstore/detail/selectorgadget/mhjhnkcfbdhnjickkkdbjoemdmbfginb?hl=en]

Click on the selector gadget icon and then click on the "Ad-Ad" and then deselect something else other than the letter ranges, what you will see down in the bottom right corner is this selector: **.alpha-nav-holder a**

So we then are going to use that selector in the method to get the links. Here is what that method looks like:

```python
def get_movie_alpha_index():
    """
    Scrapes the links of the alphabetical indices from the alphabetical
    directory
    """
    print 'Creating alpha index links'
    
     # Each letter for the URL 
    alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    links = []
    f = open("alpha_index.txt", "w")
    
    # Scrape the titles that start with numbers
    f.write('/movies/alphabetical.htm?letter=NUM&p=.htm\n') 
    base_url = 'http://www.boxofficemojo.com'
    for x in alphabet:
        end_url = '/movies/alphabetical.htm?letter='+x+'&p=.htm' 
        response = requests.get(base_url+end_url)
        soup = BeautifulSoup(response.content,'lxml')]
```
Now lets look at the line below because a lot is going on.

_*tmplink = list(set(soup.select('.alpha-nav-holder a')))*_

list(set()) because the selector selects the links twice, once at the bottom, once at the top. 
Notice the soup.select() instead of the more common soup.find(), select() allows you to select by  CSS selector instead of selecting by parent or child.

Lets finish up the method.

```python
        tmplink = list(set(soup.select('.alpha-nav-holder a'))) 
        link = [elem['href'] for elem in tmplink]
        link.append(end_url)
        link.sort()
        links.extend(link)
    [f.write(line+'\n') for line in links]
    f.close()
```

So now we have all the links for each alphabetical index, so now we get to scrape... more links. 
This time it's going to be the link to each individual movie page. Click on the selector gadget again, 
except this time, from any of the alphabetical index pages, click on one of the movies. Then deselect something
other than a movie. The selector you will see is: **a b** 

However if you try and use that selector you won't get the best results. So after inspecting the element and adding some logic of my own from the cheat sheet I found a better selector: **#body > div table table tr a[href^="/movies/?"]** 

Now time to get the movie page links, here is the method to do so:

```python 
def get_movie_links(links_list):
    """
    Scrape the links of each movie's respective in depth description page
    """
    print 'Creating movie links'
    links =[]
    f = open("movie_links.txt", "w") # Use the links we scraped earlier
    base_url = 'http://www.boxofficemojo.com'
    for end_url in links_list:
        print 'Scraping: '+end_url+'\n'
        response = requests.get(base_url+end_url)
        soup = BeautifulSoup(response.content,'lxml')
        tmp = soup.select('#body > div table table tr a[href^="/movies/?"]')
        link = [elem['href'] for elem in tmp]
        link.sort()
        links.extend(link)
    for line in links:
        f.write(line.encode('utf-8'))
        f.write('\n')
    f.close()
```
Pretty much same thing as earlier except with a few minor tweaks.

Now that we have the links of all the movies all we need to do is scrape all the pages.

However before we do that there is quite a bit of information we need to parse on each page,
and I'd rather just get that information straight from the page instead of parsing it later.
So quite a few helper methods need to be written.

So lets take a look at two extreme cases for the movie pages. 

The first being one with nearly complete information: [http://www.boxofficemojo.com/movies/?id=starwars7.htm]

There is a lot of information to be had on this page. Actors, Directors, Total Domestic gross, foreign gross, etc.
We probably don't have to worry about "Related Stories" as they wouldn't really be pertinent to most movies success 
at the box office.

So lets take at the second source with almost no information: [http://www.boxofficemojo.com/movies/?id=bestiaire.htm]

So this doesn't have nearly as much information as Starwars, but it does still have the same summary box as Starwars. 
However I wouldn't like to assume that they all have summary boxes. It appears there is a lot of variance in the kind of 
information that exists on any given page, so lets just handle for both case the information doesn't exist and if it  
does exist, and then we can take data out later if it doesn't have enough information.

So here are all the selectors we will need:

- Title: **br + font b**
- Domestic Total Gross: **center font > b**
- Summary Table: **center td**
- Links to records for different time periods and locations: **.nav_tabs a** 
- Opening weekend and theater stats: **td td td td .mp_box + .mp_box td**
- Cast: **td + td .mp_box_content table:nth-of-type(1) tr td font**
- Rankings: **.mp_box**

We need to write helper methods to parse all of them. I would go through the details, but realistically it's
just tedious dirty string parsing, nothing really special about it. So I'm just going to post the methods:

```python
def cast_format(cast_list):
    """
    Parse the list of the cast into a list

    Note: From what I recall this wasn't parsed in the best possible format
    and I had to parse it more later in another method
    """
    try:
        print 'Cast...\n'
        cast = {}
        last_key = ''
        for entry_tmp in cast_list:
            newEntry = []
            entry = entry_tmp.get_text().replace('*','')
            #if ends in colon, make a key by that name with a list
            if entry[-1] == ':':
                last_key = entry[0:-1]
                cast[last_key] = []
            else:
                for letter in entry:
                    #if empty add first letter to avoid index out of bounds
                    if len(newEntry) == 0:
                        newEntry.append(letter)
                    else:
                        #if lowercase before upper insert space
                        # also handle for names like "McCarthy"
                        if newEntry[len(newEntry)-2] == 'M' and newEntry[-1]==\
                        'c' and letter.isupper():
                            newEntry.append(letter)

                        #if lowercase before upper insert space
                        elif  newEntry[-1] == ')' and letter.isupper():
                            cast[last_key].append(''.join(newEntry))
                            newEntry=[]
                            newEntry.append(letter)
                        #if lowercase before upper insert space
                        elif  newEntry[-1].islower() and letter.isupper():
                            cast[last_key].append(''.join(newEntry))
                            newEntry=[]
                            newEntry.append(letter)

                        else:
                            newEntry.append(letter)
                cast[last_key].append(''.join(newEntry))
    except KeyError:
        return {}
    return cast
def isInt(s):
    try:
        int(s)
        return True
    except ValueError:
        return False

def table_parser(lst):
    """
    Parse all the different charts and ranking tables into a meaningful
    format
    """
    print 'Ranks...\n'
    i = 1
    ret_list = []
    while i < len(lst):

        if lst[i-1].get_text()=='Chart' and lst[i].get_text() == 'Rank':
            i+=2
        elif lst[i-1].get_text() == 'Charts (Premier Pass Users Only)' \
        and lst[i].get_text() == 'Rank':

            i+=2
        elif lst[i-1].get_text() == 'Rank':
            i+=2
        elif isInt(lst[i-1].get_text()):
            i+=1
            ret_list.append((lst[i-1].get_text(),lst[i].get_text()))
        else:
            ret_list.append((lst[i-1].get_text(),lst[i].get_text()))
            i+=2
    return ret_list

def mp_box_helper(lsts):
    '''
    Takes all the tables on a movie page except Total Lifetime Grosses,
    Domestic Summary, The Players, only tables with ranks, creates a dict
    with title of each table as the key, and then creates a list of tuples
    in the following format '[(RANK_LIST1, RANK1), (RANK_LIST2, RANK2), ... ]'
    '''
    print 'mp_box...\n'
    table_dict = {}
    table_title = ''
    links = []
    tagSoup = BeautifulSoup('<b class="boldest">I Am a Tag</b>')
    tag = tagSoup.b
    dont_parse = ['Related Products','Domestic Summary','Total\
     Lifetime Grosses','The Players','Related Stories']

    for lst in lsts:
        tag_cnt = 0
        for content in lst:
            if type(content) == type(tag) and tag_cnt == 0 and \
            content.get_text() in dont_parse:

                break

            if type(content) == type(tag) and tag_cnt == 0 and \
            content.get_text() not in dont_parse:

                tag_cnt+=1
                table_title = content.get_text()
                table_dict[table_title] = []
                continue
            if type(content) == type(tag) and tag_cnt == 1:
                try:
                    links_unparsed = [x for x in content if type(x) == \
                    type(tag) and type(x.a) == type(tag)]

                    links = list(set([x.a['href'] for x in links_unparsed[0].\
                    descendants if type(x) == type(tag) and type(x.a) ==\
                    type(tag)]))

                    if len(links) > 1: #if not an awards table
                        #Get the raw table html and all descendants,
                        #contains annoying messy duplicates
                        raw_ranks = [x.b for x in content.descendants if\
                        type(x) == type(tag) and type(x.b) == type(tag) ]
                        #Use groupby to remove to duplicates
                        table_ranks_unparsed =[x[0] for x in groupby(raw_ranks)]
                        table_ranks = table_parser(table_ranks_unparsed)
                        table_dict[table_title].extend(table_ranks)
                    else:
                        table_dict[table_title].extend(links)
                except IndexError:
                    table_dict[table_title] = 'unable to parse'
    return table_dict






def parse_budget(num):
    """
    Parse the budget on each movie page
    """
    if num.strip() != 'N/A':
        if len(num.split()) > 1:
            ret_int = num.split()[0]+'000000'

            return int(ret_int.replace('$',''))
        else:
            return int(num.lstrip('$'))
    else:
        return num


def parse_domestic(lst):
    """
    Parse the opening weekend total domestic gross and the widest
    theater release

    If the info isn't there or is N/A it equals 'Unable to parse (n/a?)'
    """
    unparsed = [x.get_text() for x in lst]
    previous = ''
    weekend = ''
    theater = ''
    print'Domestic...\n'
    for elem in unparsed:
        try:
            if previous.split() == [u'Opening', u'Weekend:']:
                weekend = int(elem.replace('$','')[1:len(elem)].replace(',',''))
        except ValueError:
            weekend = 'Unable to parse (n/a?)'
        try:
            if previous.split() == [u'Widest', u'Release:']:
                theater = int(elem.split()[0].replace(',',''))
        except ValueError:
            weekend = 'Unable to parse (n/a?)'
        previous = elem
    return weekend,theater

def parse_info_list(info_list):
    """
    Parse the summary table for the following information:
    distributor,date,genre,movie_time,rating,budget

    if the info isn't available then it equals 'Unable to parse'
    """
    i = 0
    distributor= 'N/A'
    date= 'N/A'
    genre= 'N/A'
    movie_time= 'N/A'
    rating= 'N/A'
    budget= 'N/A'
    print 'Info_list...\n'
    while i < len(info_list):
        try:
            if info_list[i].get_text()[:info_list[i].get_text().index(':')+1]\
            .split()[0:2] == [u'Distributor:']:

                distributor = info_list[i].get_text()[info_list[i].get_text()\
                .index(':')+1:].strip()

        except ValueError:
            distributor = 'Unable to parse'
        try:
            if info_list[i].get_text()[:info_list[i].get_text().index(':')+1]\
            .split()[0:2] == [u'Release', u'Date:']:

                date = time.strptime(info_list[i].get_text()[info_list[i]\
                .get_text().index(':')+1:].strip(),'%B %d, %Y')

        except ValueError:
            date = 'Unable to parse'
        try:
            if info_list[i].get_text()[:info_list[i].get_text().index(':')+1]\
            .split()[0:2] == [u'Genre:']:

                genre = info_list[i].get_text()[info_list[i].get_text()\
                .index(':')+1:].strip()

        except ValueError:
            genre = 'Unable to parse'
        try:
            if info_list[i].get_text()[:info_list[i].get_text().index(':')+1]\
            .split()[0:2] == [u'Runtime:']:

                time_split = info_list[i].get_text()[info_list[i].get_text()\
                .index(':')+1:].lstrip().split()

                movie_time = dt.time(int(time_split[0]),int(time_split[2]),0,0)
        except ValueError:
            movie_time = 'Unable to parse'
        try:
            if info_list[i].get_text()[:info_list[i].get_text().index(':')+1]\
            .split()[0:2] == [u'MPAA', u'Rating:']:

                rating = info_list[i].get_text()[info_list[i].get_text()\
                .index(':')+1:].strip()
        except ValueError:
            rating = 'Unable to parse'
        try:
            if info_list[i].get_text()[:info_list[i].get_text().index(':')+1]\
            .split()[0:2] == [u'Production', u'Budget:']:

                budget = parse_budget(info_list[i].get_text()[info_list[i]\
                .get_text().index(':')+1:])

        except ValueError:
            budget = 'Unable to parse'
        i+=1
    return distributor,date,genre,movie_time,rating,budget
```

So then all that is left is to actually scrape all the movie pages:

```python
def get_movie_pages(links):
    """
    Get the movie page info and write it to a file, makes calls to all the
    other respective parsers for each part of the movie page.
    
    This takes a little while to run, go off and get a coffee or luncb and 
    come back to it
    """
    print 'THE BIG ONE!!! Scraping all the movie info from each movie\n'
    base_url = 'http://www.boxofficemojo.com'
    for end_url in links:
        url = base_url+end_url[:-1]
        response = requests.get(url)
        soup = BeautifulSoup(response.content,'lxml')
        print 'Scraping: '+base_url+end_url+'\n'
        if len(soup.select('p')) < 1 or soup.select('p')[0].get_text() ==\
         u"Sorry, we're not able to process your request.":

            print "Skipping "+ end_url + '\n'
            continue
        # GET TITLES 
        title = soup.select('br + font b')[0].get_text() 
        
        # GET DOMESTICAL TOTAL GROSS
        domestic_t = soup.select('center font > b') 
        domestic_total = 'N/A'
        if len(domestic_t) > 0:
            domestic_total = int(domestic_t[0].get_text().replace(',','').\
            lstrip('$').split()[0])
            
        # GET SUMMARY TABLE (or here it is called "info_list")
        info_list = soup.select('center td')
        distributor,date,genre,movie_time,rating,budget =\
         parse_info_list(info_list)
         
        # GET LINKS
        nav_tabs_lst = soup.select('.nav_tabs a')
        nav_tabs = [link['href'] for link in nav_tabs_lst]

        # GET OPENING WEEKEND AND THEATERS
        unparsed_domestic = soup.select('td td td td .mp_box + .mp_box td')
        opening_weekend,theaters = parse_domestic(unparsed_domestic)

        # GET CAST
        cast_temp = soup.select('td + td .mp_box_content \
        table:nth-of-type(1) tr td font')

        cast = cast_format(cast_temp)
        mp_box = soup.select('.mp_box')
        ranks = mp_box_helper(mp_box)

        # CREATE PYTHON DICT TO WRITE TO CSV FILE, ROW BY ROW
        insert_movie = {'title':title,'domestic_total':domestic_total,
        'distributor':distributor, 'date':date,'genre':genre,
        'movie_time':movie_time,'rating':rating,'budget':budget,
        'nav_tabs':nav_tabs,'opening_weekend':opening_weekend,
        'theaters':theaters,'cast':cast,'ranks':ranks}

        # WRITE TO FILE
        with open('movies.csv', 'a') as csvfile:
            fieldnames = ['title','domestic_total','distributor','date',\
            'genre','movie_time','rating','budget','nav_tabs','opening_weekend'\
            ,'theaters','cast','ranks']

            writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
            writer.writerow(insert_movie)
            csvfile.close()
```

Then we're pretty much done! The only other thing I would do is add a main method to regulate the scraping
of all the other links. This way you don't scrape the same links twice.

```python
def main_spider():
    """
    main method to run script
    """
    if os.path.isfile("alpha_index.txt"):
        pass
    else:
        get_movie_alpha_index()

    if os.path.isfile("movie_links.txt"):
        pass
    else:
        f = open("alpha_index.txt","r")
        text = f.readlines()
        f.close()
        get_movie_links(text)
    f = open("movie_links.txt")
    links = f.readlines()
    f.close()
    get_movie_pages(links)
  
#Run the script  
main_spider()
```

NOTE: All the scraping was on boxofficemojo.com, and their robots.txt allows you to pretty much scrape anywhere, however other sites would require a much different approach in that one would have to contruct a network of proxy servers to get past anti-scraping (really anti-DDOS) defense mechanisms. I plan on writing on article on how to go about doing this in the future as I have some experience in doing exactly this 
