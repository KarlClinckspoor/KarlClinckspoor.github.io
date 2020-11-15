---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults
# layout: home
title: My CV
author: Karl Jan Clinckspoor
layout: default
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js">
</script>

<script type="text/javascript">
window.MathJax = {
  tex: {
    packages: ['base', 'ams', 'mhchem']
  },
  loader: {
    load: ['ui/menu', '[tex]/ams', '[tex]/mhchem']
  }
};
</script>


Hi, I'm Karl, a PhD in Physical Chemistry with emphasis in Colloidal Chemistry.
I finished my PhD in 2019 and am currently working as a research chemist at
LABORE, a group dedicated to studying oil reservoirs, since June/2019.

# Current work

My current role is project leader, where I am developing solutions without
solids to combat lost circulation in fractured oil reservoirs, financed by
PETROBRAS. My tasks are varied, from literature review, planning and performing
experiments, mentoring students, and also more administrative tasks, such as
procuring reagents and equipment.

I am also collaborating on a project focused on using biopolymers for enhanced
oil recovery, financed by Shell. My main responsibility is consulting on the
chemical aspects of the system, and implementing NMR relaxometry.

# Education

I got my bachelor in Chemistry at Unicamp in 2012. Then, I joined Edvaldo
Sabadini's research group, where I did my Master's and my PhD. In both instances
I studied wormlike micelles.

## Master's

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
[here](assets/Clinckspoor_KJ_mestrado.pdf) (in portuguese).

## PhD

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
throughout it.

A copy of my thesis can be found [here](assets/Clinckspoor_KJ_doutorado.pdf). I
decided to write it in Portuguese, sacrificing international appeal, because I
wanted my work to be more approachable to Brazilian students, who often struggle
with texts in English.

# Publications

A list of my publications can be found [here](publications.html)


<!-- List of blog posts:
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul> -->