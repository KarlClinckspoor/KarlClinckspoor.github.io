---
title: Evolution of a Thesis
author: Karl Jan Clinckspoor
layout: default
---

[link](#fast-version-concerto-linverno-vivaldi)

## Short story

I had a hunch it would be cool to watch my thesis evolving, so I used git to
version control the LaTeX source files, then used Python with mainly
`matplotlib`, `numpy` and `wordcloud`. The source code can be found
[here](https://github.com/KarlClinckspoor/EvolutionThesis).

It first calculates the stats for each commit (stage), then compiles the pdf,
arranges the pages into a grid, generates a workcloud for that commit and
finally joins everything into a single figure (frame). Each frame was imported
into Da Vinci Resolve 17 (free) and merged with a soundtrack of my choosing. 

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
years. It was rough, much rougher than Python, but I persevered and managed to
create my first few decent documents.

By this time, I had finished my experiments and had to start writing my thesis.
Since I had migrated totally to Python, I had to re-analyze and re-plot
everything I had done before. I welcomed that, as looking again at old data is
often quite informative, and makes me double-check anything more carefully. I
could do that as I wrote the text, so instead of splitting everything into
separate folders, I put everything into one, and version controlled it with git.
This way, I had a third layer of backup, and could get some experience with git.

I think the idea to make a "movie" out of the evolution of my thesis came while
I was writing it, but I kept that in the backburner for a very long time. More
than a year after getting my degree, I decided to revisit this theme, and hence
this project was born.

The main gist of how the script works is that it first iterates through all the
commits, then merges all the `.tex` files, do some counting of stuff like figure
environments, commands, etc. Then, the troublesome LaTeX stuff is incrementally
stripped, so that word counting can be done without much hassle. All of this is
stored into a `Stats` object, which is then `pickled` for later use.

Then I go again through each commit and compile it, making a few modifications,
such as removing the `\includeonly` commands. I copy the pdfs to another folder,
then use `pdftoppm` to separate each pdf into pngs. These pngs are arranged in a
grid, generating an absolutely massive image that was quite cumbersome to manage
with a ~10 year old overheating notebook with only 4 GB of RAM. 
## Fast version (Concerto L'inverno, Vivaldi)

<iframe width="560" height="315" src="https://www.youtube.com/embed/jKUw8FVsJxE" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Slow version (Gymnopedie No.1 Erik Satie)

<iframe width="560" height="315" src="https://www.youtube.com/embed/o8FFMAszz5s" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>