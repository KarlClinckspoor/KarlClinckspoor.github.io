---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults
# layout: home
title: My CV
author: Karl Jan Clinckspoor
layout: default
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js">
</script>

<script type="text/javascript">
window.MathJax = {
  tex: {
    packages: {'base', 'ams', 'mhchem']
  },
  loader: {
    load: ['ui/menu', '[tex]/ams', '[tex]/mhchem']
  }
};
</script>

<!-- <script type="text/javascript">
window.MathJax = {
  tex: {packages: {'[+]': ['mhchem']}}
};
</script> -->

Hi, I'm Karl, a PhD in Physical Chemistry with emphasis in Colloidal Chemistry.
I finished my PhD in 2019 and am currently working as a research chemist at
LABORE, a group dedicated to studying oil reservoirs, since June/2019.

- [Current work and interests](#current-work-and-interests)
- [Education](#education)
  - [Master's](#masters)
  - [PhD](#phd)
- [Other projects](#other-projects)

## Current work and interests

My current role is project leader, where I am developing solutions without
solids to combat lost circulation in fractured oil reservoirs, financed by
PETROBRAS. My tasks are varied, from literature review, planning and performing
experiments, mentoring students, and also more administrative tasks, such as
procuring reagents and equipment.

I am also collaborating on a project focused on using biopolymers for enhanced
oil recovery, financed by Shell. My main responsibility is consulting on the
chemical aspects of the system, and implementing NMR relaxometry.

I am interested in using my knowledge in colloidal chemistry, allied with
computational methods, to solve real-world problems and propose useful
applications.

## Education

I got my bachelor in Chemistry at Unicamp in 2012. Then, I joined Edvaldo
Sabadini's research group, where I did my Master's and my PhD. In both instances
I studied wormlike micelles. A list of my publications can be found
[here](publications.html)

### Master's

In my Master's, I studied a smart wormlike micelle system composed of a cationic
surfactant (cetyltrimethylammonium bromide, CTAB) with cinnamate derivatives.
These were chosen for their structural similarity to sodium salicylate (NaSal),
which is the go-to cossolute to form wormlike micelles with cationic
surfactants. The difference is a spacer group between the carboxylic acid and
the aromatic ring, and the ortho group. This spacer group could be
$$\ce{\phi-CH2 - CH2-COO-}$$ or $$\phi-\ce{CH=CH-COO-}$$ and the ortho group
could be $$\ce{H}$$, $$\ce{OH}$$ or $$\ce{OCH3}$$. These groups make the
cossolutes sensitive to pH, with the most interesting one being
ortho-hydroxy-cinnamate (OHCA). It was the strongest one, at around pH 9, but
became totally liquid at higher pH. Other cossolutes showed little interest in
pH variations (above pH 4), and the "most bare" cossolute, without the double
bond or ortho group, barely even formed micelles. To study these systems, I
employed a great deal of rheology, and also isothermal titration calorimetry.

A copy of my Master's dissertation can be found
[here](assets/Clinckspoor_KJ_mestrado.pdf) (in portuguese). One aspect of my
work has been developed in an article, check it out
[here](https://link.springer.com/article/10.1007/s00396-015-3672-y). We also
described the calorimetric profile of wormlike micelles in another article,
[here](https://www.sciencedirect.com/science/article/abs/pii/S0021961415003924).

### PhD

In my PhD, I decided to tackle a bigger problem. Instead of looking directly at
the components of the wormlike micelles, I looked at where they formed, i.e.,
the solvent. This was inspired greatly by the work of Prof. Heinz Hoffmann, who
became a friend after Prof. Sabadini introduced us during ECIS Cyprus in 2014.
Since then, we maintained contact. His work on adding glycerol to wormlike
micelles and bilayers was fascinating. Even after his visit to our group, we
spent quite some time debating and trying to understand exactly what was at
stake. I admit I felt very incompetent during this part.

When deciding my topic for the PhD, I thought about continuing Hoffmann's work,
using other solvents. Laila Lorenzetti, another Master's student, did some
initial studies with sucrose, and I picked up where she left off, and studied,
in total, glycerol, sucrose, dimethylsulfoxide, 1,3-butanediol and urea. These
were chosen for their relatively high polarity. The concentration ranges were
from 10% to 60% by mass. I decided to stick with the well known CTAB+NaSal
system, and did NaSal and solvent concentration sweeps.

What we found was quite interesting. Sucrose had little effect on the micelles,
even at concentrations as high as 50%. 1,3-butanediol was deleterious, and lead
to very low viscosity fluids. Urea showed a similar behavior to glycerol, but
the viscosity increased, instead of decreasing. In the end, we found that the
Gordon parameter, related to solvent cohesivity (and surface tension) was the
best descriptive parameter of the solvent to explain the changes observed, but
the refractive index and the dielectric constant (which control the
intermicellar attractions through the Hamaker constant) are also necessary to
better explain the system.

During my PhD I fomented my interest in chemometric/data analysis methods, such
as principal component analysis, clustering, etc. I also taught myself Python,
and learned Matlab, in order to apply these techniques to my studies, which is
something I was always interested in. At the moment, I do all my data analysis
using these tools, and highly recommend it, for the breadth, speed and
reproducibility you gain. My thesis reflects this, and I show many code snippets
throughout it. I also gave a short class on using Python for data analysis, and
the content can be found [on Github](https://github.com/KarlClinckspoor/CursoPython)

A copy of my thesis can be found [here](assets/Clinckspoor_KJ_doutorado.pdf). I
decided to write it in Portuguese, sacrificing international appeal, because I
wanted my work to be more approachable to Brazilian students, who often struggle
with texts in English. If you want a condensed version in English, check out my
published article
[here](https://www.sciencedirect.com/science/article/abs/pii/S0021979718300304).
If you're interested in the LaTeX source code, [check out the
repo](https://github.com/KarlClinckspoor/Tese). You need to request permission
before using anything from it, be it text, figures or data!

During my PhD, my supervisor and I were invited to write a book chapter, which
is partially available in [Google
Books](https://books.google.com.br/books?hl=en&lr=&id=L3UoDwAAQBAJ&oi=fnd&pg=PA298&dq=info:3qTJ9kG7eMUJ:scholar.google.com&ots=jns5DBE0Iu&sig=yBuIrxAcifUaIyfkctyJRutnh1A&redir_esc=y#v=onepage&q&f=false).

I also spent quite some time trying to measure the kinetics of formation of my
wormlike micelles, but didn't have much luck, unfortunately. I used
time-resolved fluorescence (long scales, in the order of milisseconds, not
picoseconds), with the help from Prof. René Nome from Unicamp, and time-resolved
small-angle X-ray scattering, with the help from Prof. Jan Skov Pedersen from
Aarhus, Denmark. Since I didn't get any positive result, no article was
published. Nevertheless, this work spawned another one, which I collaborated
some, and it can be found
[here](https://www.sciencedirect.com/science/article/abs/pii/S0021979719305636).
The model for wormlike micelles Prof. Pedersen developed in Fortran, and I
ported to Python, can be found [on
Github](https://github.com/KarlClinckspoor/SAXS_treatment).

In addition to all of this, I branched out to studying a biological system.
Prof. Pessine introduced me to Dr. Carla Gomes, who was conducting a beautiful
study on cystic fibrosis. We analysed hundreds of samples using the rheometer,
and the combined results of those experiments with their data has been published
[here](https://link.springer.com/article/10.1007/s40261-019-00861-x). My
interest in automating analyses was very useful for this study, and I could run
hundreds of fits in 1 minute, opposed to, at most, 5 or so, per minute, max. The
code I used for this can be found
[here](https://github.com/KarlClinckspoor/Rheology) and
[here](https://github.com/KarlClinckspoor/Tratamento_Muco)

## Other projects

I have a few concurrent interests, many of which are offline. I have several
coding projects of varying levels of completion, which are available on my
Github page, and which I'm describing here.

There's this implementation of the "haveibeenpwned.com" API. You supply it with
a password you use, and it returns, safely, if your password has been
compromised previously.
[Link](https://github.com/KarlClinckspoor/PasswordChecker).

I also decided to transform reddit's Askhistorians best questions and answers
into a book. I stopped after they said they didn't have any interest in it. It's
very crude and primitive, but I might pick it up at some later date.
[Link](https://github.com/KarlClinckspoor/AHBook)

I also used my own Android phone's accumulated location data and some Python
tools to visualize my daily commute and trips in recent years.
[Link](https://github.com/KarlClinckspoor/Plot-Location-Data)

There's this C# Project that aims to recreate Ultima Underworld 1 and 2 in
Unity. It's way above my head, but one of the tools they developed is an
extractor. The executable itself is now available at the [main
repo](https://github.com/hankmorgan/UnderworldExporter), but that wasn't the
case before. I spent some time figuring out how to build it, and had to actually
change some stuff in the C++ code. I am actually a bit proud of this, since I
consider C++ to be one of the hardest languages to deal with. The link to the
extractor is
[here](https://github.com/KarlClinckspoor/UnderworldExporter/releases/tag/ExtractorBuilt)

I did some crude Data analysis on the chemical industries from 2012 to 2014, in
Brazil. Unfortunately, I couldn't get any more detailed data, so the project
died early. [Link](https://github.com/KarlClinckspoor/IndustriaQuimica)

In addition, I'm developing some other repos, privately. When they are ready,
I'll list them here.

<!-- List of blog posts:
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul> -->
