---
title: Evolution of a Thesis
author: Karl Jan Clinckspoor
layout: default
---

## Summary

* Video link [fast](#fast-version-concerto-linverno-vivaldi)
* Video link [slow](#slow-version-gymnopedie-no1-erik-satie)
* [Short story](#short-story)
* [Long story](#long-story)


## Short story

I used git to version control the LaTeX source files, of my thesis, and had a
hunch it would be cool to see it evolve. Then used Python with mainly
`matplotlib`, `numpy` and `wordcloud` to create the frames. The source code can
be found [here](https://github.com/KarlClinckspoor/EvolutionThesis).

It first calculates the stats for each commit (stage), then compiles the `pdf`,
arranges the pages into a grid, generates a workcloud for that commit and
finally joins everything into a single figure (frame). Each frame was imported
into Da Vinci Resolve 17 (free) and merged with a soundtrack of my choosing.

### Fast version (Concerto L'inverno, Vivaldi)

<iframe width="560" height="315" src="https://www.youtube.com/embed/jKUw8FVsJxE"
 frameborder="0" allow="accelerometer; autoplay; clipboard-write; 
 encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Long story

Towards the end of my PhD, around August 2018, I had started learning Python,
having reawakened within me the desire to code. This completely revolutionized
the way I analyzed data and what I deemed to be possible to do in relatively
short times. I started small, then slowly got more comfortable with Jupyter
notebooks, `matplotlib`, `numpy`, `pandas` and before you know it, I had
switched 100% to using Python and abandoned OriginLab's Origin. I still think
it's a fine program and I use it occasionally, but if I need to do anything
that needs to be scalable, I prefer Python.

Immersed in this environment, I decided to confront LaTeX again, after many
years. It was rough, I remember struggling a lot in the beginning. Tons of
errors I couldn't understand. TeXStudio was a godsend for me, it helped me a lot
to troubleshoot my first documents. In the end, I persevered and got consistent
at it.

By this time, I had finished my experiments and had to start writing my thesis.
I often read that people version controlled their documents, so I decided to try
that on my thesis as well. My first commit after I had spent some time making a
general layout of what my thesis would looks like. Stuff like sections,
headings, fonts, appendices, etc. I wanted to write a thesis that would be easy
for anyone graduating in chemistry could understand, so I took time to explain a
lot of the more basic stuff. I wanted this text to be a reference for new
students in the group, and other places as well. That's the reason I wrote in
Portuguese. In Brazil, English literacy is still very poor, so good quality
texts in Portuguese are rare (not that I think my text if very high quality, as
any time I re-read it, I notice problems). That's the main reason the text
reached 300+ pages.

I started by writing the stuff that was freshest in my mind, and had to produce
the figures to go with the text. I used Python to do that, and placed the
notebooks and raw data together with the text. I felt inspired and wanted to
share with others my code, so in many instances, I also added some code listings
on how I created the more interesting stuff. The LaTeX package `minted` was
essential to do that.

When I started to write about previous experiments, I felt bad using the old
figures I had produced. They were too different in style, were scattered in a
million different places, and I felt I could do much better, and so I reanalyzed
everything. Revisiting old stuff is good, you always learn something new, or
correct a past mistake. Naturally, the raw data and notebooks were placed also
with the main text. Nowadays, I feel like I should always do this. When writing
a new article or whatever, there should be a folder with the data and analysis.
Makes finding stuff much easier, at the expense of wasting some disk space. The
only exception were some truly huge files (I think 30+ MB each, and there were
several dozen of them), and photos. I learned that saving the images as `pdf`s was
not only much better quality-wise, the compilation went much faster and the
output `pdf` was also much smaller.

I also dabbled with some animations at this time. When deciding about what I
would write in the Rheology section, I made a small animation of how the elastic
and viscous contributions to the total stress vary depending on the loss angle.
I will search for the code later, but the animation is
[here](https://www.youtube.com/watch?v=XRfU6Zdi9aM). I don't remember exactly,
but it was perhaps at this time I thought about doing an animation of the
evolution of my thesis.

Time went on, my supervisor was very supportive with my vision. When it came to
the defense, I had to print the text out, and it turned HUGE because I couldn't
do two-sided prints (it was too expensive). I printed the colored pages at my
supervisor's printer, and paid for the black and white pages. The result was two
large volumes of text. The committee made a lot of jokes about that, but it was
all in good fun. I was praised a lot, and we actually had some fun during my
defense.

My last obstacle was fitting the text to the university's standards. That was
quite troublesome, as they had modified the ABNT standard, which I had used
(courtesy of the `abntex2` package). For example, I made sure to fit the tables
to the format
[ABNT](https://tecnoblog.net/236041/guia-normas-abnt-trabalho-academico-tcc/)
requires
([IBGE](http://climacom.mudancasclimaticas.net.br/wp-content/uploads/2015/08/Normas-IBGE-simplificado.pdf)),
but they didn't care about that. They cared that I had blank pages (which are
necessary when you're doing two-sided prints), so you can notice that the page
number decreases towards the end of the video. They also didn't want the
references to be at the end, after the appendices (even if they had references
themselves), and an Index was a no-no. After several back and forths, it was
finally approved and I got my diploma.

Now, a few years later, I decided to cultivate that idea I had a long time ago,
but this time taking it to completion.

The main gist of how the script works is that it first iterates through all the
commits. At each commit, it merges all the `.tex` files, does some counting of
stuff like figure environments, commands, etc. Then, the troublesome LaTeX stuff
is stripped, so that word counting can be done without much hassle. All of this
is stored into a `Stats` object I created, which is then `pickled` for later use.

Then I go again through each commit and compile it, making a few modifications,
such as removing the `\includeonly` commands. I copy the `pdf`s to another
folder, then use `pdftoppm` to separate each `pdf` into `png`s. These `png`s are
arranged in a grid, generating an absolutely massive image that was quite
cumbersome to manage with a ~10 year old overheating notebook with only 4 GB of
RAM.

Finally, I loaded all the `Stats` objects, and iterated through them in
chronological order. I fed every stat up to the i-th commit I was processing in
order to build the evolving graph in the middle. The wordcloud was made by
creating a wordcloud of the last commit, then using that as a reference to the
current commit. I dabbled with making the text bigger or smaller, but settled
with a simple interpolation. The header was made mostly by fiddling with the
position of the text and making sure the description fit the width of the box.

Lastly, each image was created (takes some 40-ish minutes at least I think, with
my limited resources) in sequential order (`001.png`, so forth). Da Vinci
Resolve, a surprisingly powerful and free video editor, was used to join the
images together in a video. For the slow version, I decided to closely match
each "plin plon" with a frame change. I can now say I know that Gymnopedie no.1
has less than 95 "plin plons".

And that's what inspired me to do this, and roughly how I did it. [You can check
out the code here](https://github.com/KarlClinckspoor/EvolutionThesis).

## Slow version (Gymnopedie No.1 Erik Satie)

<iframe width="560" height="315" src="https://www.youtube.com/embed/o8FFMAszz5s"
 frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media
 ; gyroscope; picture-in-picture" allowfullscreen></iframe>