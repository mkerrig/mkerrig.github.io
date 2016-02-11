---
layout: post
title: Project Luther Post
---


Hello world,

I thought it be necessary to start this blog post off with the programmer's classic first words as it parallels my first real experience 
as a data scientist. In this project we Metis students were assigned the what at first seemed to be trivial assignment of scraping 
boxofficemojo.com box office data and then  perform linear regression on the data. It was very open ended as to what  we could try and 
predict with regression, so many of my more wise peers did some variation of predicting total domestic gross with budget, genre, opening weekend, 
etc. Not to say that they all did the same thing by any means, actually I think everyone managed to do something  significantly different 
from everyone else, not an easy task to do with such a basic data set. A real quick comment on my classmates actually, this group is f#$%ing 
incredible, so many diverse backgrounds, all creative in their own respect and experts of some domain from their respective backgrounds.
It's an honor to work amongst them and to call them my "peers" even is a strange concept to me me being 19 and all. I'm used to a much different
group of people, not saying the people I used to be around weren't awesome or anything just...this isn't college, and for anyone reading this
who has been through any kind of college experience, they know exactly what I mean. So anyways, my original goal was to predict total domesitc
gross from a list of actors and directors for each repspective movie.  Now, this doesn't sound too complex right? I mean movies with Tom Hanks 
should always make money right? Well lets find out, so we have to go about figuring out how much each actor is worth. So I thought to myself, 
"I know how I'll do this, I'll just set every actor as a column in a matrix, with each vector being a linear equation to solve for total 
domestic gross, set a 1 if the actor was in the movie, and a 0 if they weren't. This should solve for each coefficient of each actor and 
that's how much they are worth." So I did exactly that, and then the matrix never solved. It took me some time and some public humiliation to 
realize what I was doing was just a step or two short of just basic linear regression, and that there are some wonderful methods out there 
that are optimized to handle these calculations. 

So after banging my head against the wall for missing the obvious answer instead of my more abstract answer, I went ahead and performed my first 
regression. I went through the results and tried to find entries with low p-values because hopefully that would show people that I expected
to be in the upper 5% of the actor world. I was pleased to find that indeed Tom Hanks had a p-value of near 0, meaning that it worked right?
Well yeah that's an easy one, what about Samuel L. Jackson, you know the highest grossing actor of all time? He had a p-value more around
0.07 or something. Not bad, but, shouldn't he be higher? Let's just print the top 10 results. Very quickly I noticed some rather unexpected people
at the top of the list. I'm going through the names, and I didn't recognize a single one with the exception of Peter Mayhew whom I knew to
be chewbacca only because of one very bizarre friend I had in middle school who sort of idolized him and would constantly make the chewbacca noise.
So...where are the actors that I know and love? Let's start with Jackson, sure he averages some 60 million dollars a movie (total domestic
gross) but he's had enough flops to seriously screw up the calculation of his respective coefficient, especially in the context of lesser 
known actors. What I mean by this is if you have only been in one movie, than when I'm running linear regression, if that movie you were in 
did paticularly well, and the other actors in that movie are in a lot of other movies like Jackson and Hanks, than you may very well get 
one of the highest coefficients in the distribution. The reason is because the other actors have to hover around lower values to be able to 
add up anywhere near the respective values across the board. So if Jackson grossed $10M and $1M in two seprate movies, and John Smith co-starred 
with Jackson in the $10M movie, Jackson's value will probably be a lot closer to $1M, because otherwise the estimation
wouldn't be as accurate, but John Smith can then be manipulated to fill the hole that Jackon couldn't, so John ends up being worth around $9M.
This make's everything a lot more difficult, and even if you do regression with actors that have been in at least X movies, you will still notice
that the actors that have been in less movies are often worth more, just because there is more room for the manipulation of their coeffcients
within the regression. You will also notice that the actors for C-P30 and Chewbacca in the Starwars Series have been pretty much the same 
people for all the movies, they have been typecasted, and probably wouldn't be as successful outside the Starwars franchise. But if you 
run the regression, why is the guy for CP30 is the most successful actor of all time?! Because they have only been in successful movies, and once
again, other actors such as Harrison Ford or whoever are limited by their other films, making the actors that are in a group of of films
worth a lot more in the context of regression because regression says "OOOOH OOOH I KNOW HOW TO MAKE THE PUZZLE FIT! I'll just cut the square
block in half, and then I'll put half the square in the triangle space, and after that I'll mold the other half of the triangle and the circle
somehow to fit in the square block, I solved it right??!" While you bang your head against the wall an go "NO Linear Regression, I did not say you
could cut stuff, and what about the circle block? What are you going to put in the circle block?" Linear Regression promptly responds "The cookie 
I get for solving two thirds of your stupid puzzle, I mean I got 60% correlation, that's pretty good isn't it?", at which point you realize you are trying
to give an infant way too complex of a problem to just solve on it's own. I mean the poor guy doesn't even have object permanence yet, how can you
expect the kid to get this puzzle, despite how trivial it seems at first. Okay end of infant metaphor. Point is Regression needs some more direction.
So the answer is simple than right? just parse out people who have been in Starwars! But, Liam Neeson was in Starwars...and so was Harrison Ford, and Samuel
L. Jackson. Okay well than I know I'll just parse out anyone who's in more Starwars movies than they are other movies! But...then Liam Neeson and Samuel L. 
Jackson get ALL the credit, and is that really fair? But you can't not include Starwars I mean it's STARWARS, like one the most societal impactful 
movie series of all time (if not THE most impactful). So what do you do? You will notice a similar pattern with a lot of movie's with many sequels
actually, that they seriously make your life hell in determining who is important and who isn't important in the movie industry. So my problem was a 
very hard brick wall, and the red bricks only got redder, and slowly oxidated to brown while I tried to brute force the brick wall into something I could bring
to people and say "Look at this cool bloody thing I made". Unfortuntely the only thing that changed shape was my skull, and I kept getting stuck between this brick 
wall, which was this universe of terrible choices of actors, and somewhat satisfactory correlation, and a hard place, which was pretty good choices of actors, but like 
0.01% correlation because I was putting way to many filters on who and who I wasn't including in the regression. I was finding that I was being very biased in my 
choices of actors and actresses as well, Adam Sandler shouldn't be a high grossing actor, but...he is, in fact he is one of the most expensive actors in Hollywood.
This kind of drain bamaging brick wall brute regression continued well into the weekend. All weekend I theorized of ideas that could help, What if I made 
movie vectors and compared similarity between succsessful movies and movies that haven't happened yet? No that just sounds like "Let's make another Starwars". 
One of the better ideas I had was some sort of degree of separation metric between actors and directors to see if I could predict total gross based on the idea
that a director might work well with actors that have worked together successively before, implying they are like-minded or work well with similar people. At the time
I blew if off as it being too much detail, I need to get my base idea working first. I tried doing similar stuff with directors...this turned out just as useless
as the actors, with any director who had done any sequel ever getting the most total domestic gross. Furthermore I quickly noticed that some of the great directors
that society usually accepts as "great", weren't doing as well as one might have though. Kubrick wasn't even in the top 10%. I quckly threw away the idea and said
"Damnit I will figure this actor thing out, if its the last thing I do!"...

At approximately 2:00 AM in the morning, 12 hours before I had to present my project, I said to myself "I need a new plan". At this point I had told pretty much
everyone I was doing something with actors or directors, so I felt very committed to doing something related to it, maybe just not the way I had originally planned. 
So I instead decided to reduce the complexity of the problem, mostly because my brain was having a hard time  staying on topic and I needed somthing easier to think about. 
I remembered my theory about past director and actor relationships so I sought out to develop a metric for that. I came to the conclusion that the average gross between 
actor to actor and director to actor would probably be the best, because that's literally historical precedent for how these pairs have done together in the past. I also
realized that big names and general stardom probably have a pretty big impact on ticket sales, so I decided that an actor's all time gross is probably a pretty good measure 
on all time exposure. Sure Peter Mayhew and Anthony Daniels (C-P30) might be near the top of the list, but realistically they would rate really high with Spielberg as a director, 
and in popularity, which would correlate with the Starwars movies, and since those are the only movies that they are in, and will likely ever be in. Really there aren't many more 
examples of actors like them that exclusively only in such high grossing movies, and those that are in other movies outside of their respective franchises, the lack past relationships 
with other actors and directors will help bring them down. I decided a slightly different measure for directors, in that instead of being all time total gross, I chose average 
gross because I found that if you directed a Harry Potter movie, or Lord of the Rings, that you would be measured way higher than potentially more popular directors. Sure 
everyone knows Micheal Bay (cue explosion), and while he grosses really high, Kubrick and Taratino hardly even manage 1/4th of the total domestic gross that Micheal Bay has,
yet I think most people would consider them much better directors, and more popular amongst film critics. Now while Bay will still average higher, it still levels the playing field
in terms of making the distribution a little closer to truth in my opinion, also just in general less variance in the distribution probably making the whole model a little more accurate.
So an entire night of work, tears, and molding my head back to the shape it was before the weekend to avoid some strange looks from my peers at Metis, I had enough to present some interesting
findings on the subject. I found that my hypothesis of past relationships may very well be correct. While the whole model only got a R^2 of some 46%, both actor to actor, and director and actor
past relationships both correlated with about 30%, and not only that, but you can clearly see there are a lot of zero values where we could potentially apply some sort of similarity metric between
actors to further optimize predicting success based on past relationships. The other two metrics both hovered around 20% correlation. All in all, while I could have done a lot better on this project
I learned quite a bit, and I am in a good place to pickup this up again and actually potentially make a pretty accurate algorithm to predict total domestic gross or profits of movies based 
on the cast. We shall see if the great shape fitter in the sky ever tries to stuff me into this puzzle again, but until then I am off to bed to prepare for another awesome day at Metis. Hope 
if you are reading this ( and if you can read this message, I'm probably going to revise this whole thing a bit), that you at least enjoyed reading about someone else's struggle with the project, or
even better you actually understand what was going on as I explained it. If there is anything to learn from this story if you are a non-data scientist, it's that brick walls hurt, stop hitting your
head on walls earlier and try a different method sooner, because I promise the wall is much more stubborn than you are.
