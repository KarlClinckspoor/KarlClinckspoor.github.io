---
# layout: page
title: "Publications"
author: Karl Jan Clinckspoor
permalink: /publications
---

## Links

<!-- # ORCID ID -->
* <div itemscope itemtype="https://schema.org/Person"><a itemprop="sameAs" content="https://orcid.org/0000-0002-0916-2773" href="https://orcid.org/0000-0002-0916-2773" target="orcid.widget" rel="me noopener noreferrer" style="vertical-align:top;"><img src="https://orcid.org/sites/default/files/images/orcid_16x16.png" style="width:1em;margin-right:.5em;" alt="ORCID iD icon">https://orcid.org/0000-0002-0916-2773</a></div>

* [Lattes CV](http://lattes.cnpq.br/6890909681073402)
* [Google Scholar](https://scholar.google.com.br/citations?user=fjQ_aXUAAAAJ&hl=en)
* [Scopus](https://www.scopus.com/authid/detail.uri?authorId=55220196200)

## Overview

In this page, I will place the few articles I have written and give some
general, and personal, comments about what went into each article.

* [Articles in Journals](#articles-in-journals)
* [Book chapters](#book-chapters)

## Articles in journals

### (2021) Optimization of an *in-situ* polymerized and crosslinked hydrogel formulation for lost circulation control

[Link](https://doi.org/10.1016/j.petrol.2021.109687), [Data](https://data.mendeley.com/datasets/mrrr7rk5pr/1)

This article is I was the most involved in, from its conception, to the publication. It has 
quite a story attached to it.

We were working on this project, and we had to develop a gel to stop flow through fractures in a 
well (lost circulation). It was a rough start, since I had to learn quite a lot in a very short 
time. When we were progressing well, then pandemic struck, and we could not do any work 
whatsoever. We had an initial formulation we thought would work, but we had no experimental 
evidence. We had to plan something that would give an answer, and relatively quickly. The PI 
insisted we look for experimental design (DOE), and so we did.

In my studies of chemometrics, I was focused on techniques such as PCA, HCA, etc, and not in 
experimental design. However, I was a bit fascinated by the area, and thought it would greatly 
improve my technical repertoire. My wife studied the subject a few months earlier, during her 
Master's, so I bounced off a few ideas and questions to her. Unfortunately, I didn't learn too 
much from her, I just got very very confused with the bunch of plusses and minuses and how to
interpret that. I figured I had to learn it by myself, so I got the book she used (loaned by her
sister) and proceeded to study. I had a lot of "free" time, so why not. I did *every* exercise, 
took copious notes, and even book bound my notes for posterity. I think it took me 1 month, but 
I left feeling like I was an expert in the area. That's never a healthy mindset, especially 
in statistics, with its wildly variable nomenclature and expectations, but this hasn't 
caused us much trouble, yet.

Then, we (me and Fuat) sat down (remotely) and established the variables we were going to study. 
It's funny, he was adamant we should include pH, and I was against, but mostly due to laziness - 
adjusting and measuring pH is something I truly loathe. In the end, he was *very* right in 
including it. When the pandemic slowed down, we came back to the lab briefly, and he started his 
experiments. We faced *so* many problems you wouldn't believe. We had to alter the starting 
formulation several times, and we discovered many aspects about our system. But in the end, we 
got to an acceptable starting composition, and we had the experiments and outputs very well 
established. And so Fuat did all the experiments in a record time (two formulations per day, 
starting at 8 and going well past 6). 

One of the techniques consisted on monitoring the gelation using NMR. It was something entirely 
new in our group, and not very much used for this application in the literature. Because of that,
the equipment wasn't entirely built to support it. For example, we had to monitor the gel for 
several hours, analysing it from time to time. That had to be done manually, since the 
equipment software didn't support automation. Then, we had to analyse that in the software, and 
it also didn't support automated analysis. The total was ~107 measurements, and each had to be 
inverted, so ~214 data files, *for each composition*. That is about ~4000 individual files in 
total, *only for NMR*. We would have gone absolutely crazy if we had to analyse them one by one. 
Thankfully, the wonderful Python package [PyAutoGUI](https://pypi.org/project/PyAutoGUI/) came 
to our rescue. We just had to insert the sample and push start on my script, and it would 
analyse the data at specific intervals. After that, we ran another script that passed the data 
through the CONTIN inverter. Much simpler.

We now had to analyse the results. I taught Fuat how to do some of the fits in Origin, and wrote 
some scripts to extract data from NMR, be it the peak value or the monoexponential fits of the 
curves. Fuat merged everything in a homeric effort into a single Excel spreadsheet. We sat down 
(in person) and I taught him how to perform the matrix operations required to obtain the model 
parameters. Then, how to analyse the results, and how to reduce the model to have more "free" 
degrees of freedom. Fuat then defended and had to write the article.

Some time later, he hadn't advanced much, a lot due to lack of motivation and time. Having to 
find a job during the pandemic consumes everyone, and I had other responsibilities. The time 
came to write it finally, and so I started. For precaution, I re-analysed everything. I found 
some discrepancies. The fit parameters were all 100% matching, but not the variable names. After
some looking around, it was just some confusion in the variable names. Since I wrote the analysis in
a Jupyter notebook, I thought we could publish the data too, so I worked to make it more readable.

Then I started to write the article itself. That came quite easily, since I was so involved in 
the project, and had spent so much time reading about the theme itself, focusing on more 
fundamental aspects, rather than practical. In the end, I was able to explain everything we 
observed, and gave some general directions for future work. It was done in less than a month I 
think, from re-analysis to final article version.

We submitted the article to the Journal of Petroleum Science and Engineering. A *lot* of time 
passed, and we got an answer - minor corrections! I never, *ever* got an article so well 
accepted at first. You can read the other tales I have here, and how I suffered to publish some 
articles. Correction was very easy, just a few adjustments here and there. A *lot* of time later,
we were accepted! Such a good feeling. Our hard work paid off. This is one of the articles I'm 
most proud of.

### (2021) Urea induces (unexpected) formation of lamellar gel-phase in low concentration of cationic surfactants

[Link](https://doi.org/10.1016/j.jcis.2021.09.018)

I received news of this one and the next article literally 1 week apart! Academia is truly an 
unexpected place. Anyway, on to the story of this one.

This is the last remaining part of the work I conducted during my PhD. I was certain I didn't 
have enough data, or enough answers, to warrant publication as a full article. Some suggested I 
publish this as a letter to a journal. Letters are typically very short, only include the most 
relevant information. So that's what we did.

First, we tried with ACS Omega, in the format of a letter. It fell on the lap of some dude called
who has beef with my advisor. He was part of a friend's thesis committee and snubbed my supervisor,
in front of everyone. My supervisor was, and is, a very fine and controlled person, but he looked
definitely down after that. Since then, I've had this guy in my personal "black book" of names, a
position reserved for very few. Why do so many academics *need* to be so distasteful?

When we got the answers to our paper, I could feel this dude was happy at our failure, despite 
not having any evidence of the fact. Reviewer 1 made good observations, but his comments were 
focused as if we were trying to basically create a phase diagram of this system, which was *not* 
the case. We were simply showing that the formation of this lamellar gel-phase is unusual, and 
we reported it. And he said the packing parameter is useful in water only, which is false. I had 
to remove some data and text from the original draft because otherwise it wouldn't fit in the 
letter format, which is much more limited than a normal paper, so his additional information 
would probably not even fit in the format. 

Reviewer 2 receives the medal, though. Here's how their review starts:

    These kind of papers are annoying for both the reviewer(s) and the editor. The submitted
    manuscript is a lab report rather than a decent scientific paper and certainly deserves not
    publishing. Although it should not be my task to read and comment these kind of
    contributions I will briefly outline my main points of criticism:

And they go on to state that the results are obvious - which the last reviewers noted *wasn't*
the case. If it was obvious, why is it so hard to find anything on the subject in the literature?
Then, they give a long list of papers we should have cited - the majority of which are, being
generous, tangentially related to the subject, and not helpful to the case the paper was trying to
make, at all. For example, *none* of them mentioned urea, which was the main point we were trying to
make! The cherry on top was that the *first* paper they ask us to cite has as corresponding author
one of the reviewers I recommended, and this name repeated itself twice more in the reference list.
Now, there's no proof that this isn't a coincidence, but if it isn't, that really unprofessional of
you, C. S.

Then, we tried in this new journal called "Experimental Results". The whole stick of this 
journal is that they are focused on publishing results, not necessarily answers. They advertised 
the paper focusing on this aspect. I thought this was a fine opportunity, so we submitted to 
this journal. Man, I got one of the *worst* reviews I ever had the displeasure of reading. Here 
it is, in **full**:

    Conductivity could be monitored during the transitions to check for urea degradation!.

That's it. Nothing else. The worst part is that the suggestion is basically *nonsense*. The other 
reviewer was better, asked for some adjustments here and there, the normal stuff. I was quite 
furious with something so unprofessional, so I contacted our liaison with the journal, Dr. A. B. 
I expressed my disgust (not the words I used) with the review I got, and told them I understood 
if they kept their decision, but I wanted another reviewer to judge my paper. Literally 1 month
later, to the day, I got an answer, saying they understand my disappointment but there was nothing
they could do, and that I should take the reviewers comments into consideration and resubmit it. And
you know what? I didn't. I will never try publishing in this journal, and will speak ill of it 
to everyone I meet - not that this will affect them in any way, it's for the *principle*.

A long time passes, but my supervisor hasn't given up on this paper yet. He enlists the help of 
a new master's student, the second author of the paper, Fernando Okasaki. He was once a student 
of mine during experimental physical-chemistry lab - I just hope I wasn't an ass with him or 
anyone else in general (I overthink my social interactions sometimes). He made some additional 
measurements and found a few good references to the paper and, together with my supervisor, they 
shape the paper up to something much more presentable. We submit to a journal, much better than 
the two previous ones, and we get some very good reviewer comments, and finally get it accepted. 
Hooray! What a journey. I can't thank him enough for his part in helping this paper get 
published, and also I can't thank my supervisor enough for not giving up on me.

### (2021) Bulk rheology characterization of biopolymer solutions and discussions of their potential for enhanced oil recovery applications

[Link](https://doi.org/10.29047/01225383.367)

This is my first journal article published while working in LABORE. This one was a doozy. It was 
started by my predecessor, was revised several times, then overhauled to include more data, then 
politics dictated some of this data had to be removed, and then this was reviewed several mores 
times, rejected at least twice before we decided to submit to CT&F, a journal of the Colombian 
firm Ecopetrol. The review process was unusual - we submitted a docx document and received 4 
docx documents, each with comments. Funnily, some were not anonymized. In any case, the article 
turned out very satisfactory, and the aesthetics of this journal made it look even better.

### (2020) Safety, Tolerability and Effects of Sodium Bicarbonate Inhalation in Cystic Fibrosis

[Link](https://doi.org/10.1007/s40261-019-00861-x)

This was an interesting article. I was contacted by one of the coauthors, Prof.
Pessine, to do some rheological analyses of mucus. A bit grossed out, I
accepted. After a ton of work and more than 500 experiments (each with 3
separate data sets), we finished a *part* of the work necessary. And it's funny
how sometimes stuff just aligns. Had I not decided to learn Python a few months
prior, I would not have been able to analyze the absolutely huge amounts of data
in any reasonable time. [I made some time ago a short demo video of one of the
scripts I developed at work](https://www.youtube.com/watch?v=rG_MGW1XuRM). Dr. Carla, the leader, conducted the rest of the
enormous work necessary, and contacted a statistician to provide the rest of the
calculations necessary. 
 
I was very satisfied with the work, a very noble endeavor. In short, we noticed
that inhaling sodium bicarbonate led to patients with the genetic disease
*cystic fibrosis* experienced an improvement in their quality of life. A far cry
from my more fundamental studies so far, and very inspiring.

### (2019) A new interpretation of the mechanism of wormlike micelle formation involving a cationic surfactant and salicylate

[Link](https://doi.org/10.1016/j.jcis.2019.05.025)

I had minimal contributions to this work itself. My main contribution was
obtaining the initial data and performing preliminary experiments. As I was
focused on something else, my group colleagues noticed how interesting some of
the results I had were and developed this work.

This article is very niche, basically deals with something only our group
studied at that time, which is the formation of wormlike micelles based on
isothermal titration calorimetry. We had a previous interpretation ([here](#the-thermal-signature-of-wormlike-micelles)) of the
results after much thinking, and this article updates that interpretation using
some additional spectroscopic techniques.

### (2018) Rheological and calorimetric study of alkyltrimethylammonium bromide-sodium salicylate wormlike micelles in aqueous binary systems

[Link](https://doi.org/10.1016/j.jcis.2018.01.024)

This article explains the core part of my thesis. Normally, studies focus on the
effects a small amount of additives that place themselves on the micellar
palisade and change their properties, e. g., sodium salicylate (NaSal) leads to
growth of cetyltrimethylammonium bromide (CTAB). Few studies consider what the
solvent is doing. In this work, I kept the wormlike micelle system (CTAB+NaSal)
and changed the solvent by adding relatively large amounts of water-soluble
compounds.

The inspiration came from the work of Prof. Hoffmann, who we befriended in a
congress in Cyprus in 2014. He explained how adding glycerol led to a change in
the properties due to changes in the attractive forces between the micelles,
which can be indirectly viewed by the diffractive index difference between
solvent and micelle. We tested this hypothesis and added more parameters that
would explain our results, such as the Gordon parameter.

Of all the additives, two were most surprising. First, sucrose had almost no
effect on the system, despite it having several similar properties to Glycerol.
The other aspect is that urea led to crystallization at higher concentrations
(>35%). We still don't fully understand this fact and have tried to publish our
findings, but so far our work wasn't accepted anywhere.

The idea of the camel and dromedary was mine and I'm still proud of it. I think
having light-hearted parts to science is very beneficial.

### (2016) The thermal signature of wormlike micelles

[Link](https://doi.org/10.1016/j.jct.2015.10.021)

This article came after several students of my supervisor used isothermal
titration calorimetry to study wormlike micelles, and we had sufficient data to
propose some better hypotheses to what was happening.


### (2015) A Rehage-Hoffmann rheological approach to worm-like micelles of C16TAB and ortho-hydroxycinnamate

[Link](https://doi.org/10.1007/s00396-015-3672-y)

This is the main article of my master's dissertation. I studied this weird
additive, ortho-hydroxycinnamate, and how pH affected the properties of the
micelles. The most pleasant part was working with the additive, since it smelled
good. Adjusting the pH still gives me nightmares. Don't trust any person
reporting pH measurements to within 0.01 (or even 0.1) of any complex system,
especially one that has a cationic surfactant.

### (2012) Triboelectricity: Macroscopic Charge Patterns Formed by Self-Arraying Ions on Polymer Surfaces

[Link](https://doi.org/10.1021/la301228j)

This article came out when I was still an undergrad. I did a lot of the work and
plots contained in it, but the interpretation and writing was left to the other
people. I'm still a bit upset that another student that helped me a lot wasn't
added to the author list because, in the words of my supervisor, "she didn't
truly understand what was happening because she's an engineer". 

## Book chapters

### New Insights into the Formation of Wormlike Micelles: Kinetics and Thermodynamics

My advisor wrote most of the writing, I was tasked with literature review, text
revision, making the figures prettier and formatting stuff. It was a great honor
to participate in this, and I have Dr. Dreiss to thank for that. I hope I get to
meet her in the future, even though I'm not in the area anymore.