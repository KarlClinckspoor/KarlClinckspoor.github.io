---
layout: post
title: U4 pdf -> print
description: How to print the Ultima IV GOG scanned manuals into booklets
author: Karl Jan Clinckspoor
tags: 
    - ultima
    - games
    - bookbinding
    - latex
---

## Inspiration

To make booklets out of the pdfs, so I can use the booklets in physical form for my adventures.

## Requirements

* Some knowledge of the command line
* A LaTeX distribution (MikTeX, TeXLive)
* Python with PyPDF2 installed

## Using LaTeX to create the booklets

Spellbook and reference card are simple.

```latex
\documentclass[a4paper]{article}
\usepackage{pdfpages}
\begin{document}
	\includepdf[pages={2-62},booklet=true,landscape,fitpaper=false]{original_spellbook.pdf}
\end{document}
```

```latex
\documentclass[a4paper]{article}
\usepackage{pdfpages}
\begin{document}
	\includepdf[pages={1-6},booklet=true,landscape, delta=0 2mm]{original_refcard.pdf}
\end{document}
```

## Cropping pages in half

Requires also converting temporarily to eps, because that was the easiest way of removing the
content outside the mediaBox.

```python
import PyPDF2
from pathlib import Path

path = Path(".") / "original_manual.pdf"
out_path = Path(".")
out_path.mkdir(exist_ok=True)
out_file_path = out_path / "original_manual_split.pdf"

pdf_writer = PyPDF2.PdfFileWriter()
pdf_reader_left = PyPDF2.PdfFileReader(str(path))
pdf_reader_right = PyPDF2.PdfFileReader(str(path))

print("Starting to split the pages in two... ", end="")
for i in range(pdf_reader_left.getNumPages()):
    if (i == 0) or (i == len(pdf_reader_left.pages) - 1):
        pdf_writer.addPage(pdf_reader_left.getPage(i))
        continue

    left = pdf_reader_left.getPage(i)
    right = pdf_reader_right.getPage(i)

    *_, width, height = left.mediaBox
    right_shift = 5

    # I think mediaBox is the only one necessary, but I'm not sure
    left.mediaBox.upperRight = (width / 2 - right_shift, height)
    right.mediaBox.lowerLeft = (width / 2 + right_shift, 0)
    left.trimBox.upperRight = (width / 2 - right_shift, height)
    right.trimBox.lowerLeft = (width / 2 + right_shift, 0)
    left.bleedBox.upperRight = (width / 2 - right_shift, height)
    right.bleedBox.lowerLeft = (width / 2 + right_shift, 0)
    left.artBox.upperRight = (width / 2 - right_shift, height)
    right.artBox.lowerLeft = (width / 2 + right_shift, 0)

    left.compressContentStreams()
    right.compressContentStreams()

    pdf_writer.addPage(left)
    pdf_writer.addPage(right)

with out_file_path.open(mode="wb") as fhand:
    pdf_writer.write(fhand)
print("Finished")

from subprocess import run

print("Creating eps, might take a while... ", end="")
run(["pdf2ps", "--eps", str(out_file_path), str(out_file_path.stem) + ".eps"])
print("Finished.")
print("Converting back to pdf... ", end="")
run(["ps2pdf", str(out_file_path.stem) + ".eps", str(out_file_path)])
print("Finished")
print("Removing temporary eps")
Path(out_file_path.stem + ".eps").unlink()
print("All done")
```

Then, just need to compile this simple latex file.

```latex
\documentclass[a4paper]{article}
\usepackage{pdfpages}
\begin{document}
	\includepdf[pages={3-38},booklet=true,landscape]{original_manual_split.pdf}
\end{document}
```

## Cluebook

Same thing for the cluebook, but had to modify some stuff slightly.

```python
import PyPDF2
from pathlib import Path

path = Path(".") / "original_cluebook.pdf"
out_path = Path(".")
out_path.mkdir(exist_ok=True)
out_file_path = out_path / "original_cluebook_split.pdf"

pdf_writer = PyPDF2.PdfFileWriter()
pdf_reader_left = PyPDF2.PdfFileReader(str(path))
pdf_reader_right = PyPDF2.PdfFileReader(str(path))

print("Starting to split the pages in two... ", end="", flush=True)
for i in range(pdf_reader_left.getNumPages()):
    left = pdf_reader_left.getPage(i)
    right = pdf_reader_right.getPage(i)

    *_, width, height = left.mediaBox
    right_shift = 0

    # I think mediaBox is the only one necessary, but I'm not sure
    left.mediaBox.upperRight = (width / 2 - right_shift, height)
    right.mediaBox.lowerLeft = (width / 2 + right_shift, 0)
    left.trimBox.upperRight = (width / 2 - right_shift, height)
    right.trimBox.lowerLeft = (width / 2 + right_shift, 0)
    left.bleedBox.upperRight = (width / 2 - right_shift, height)
    right.bleedBox.lowerLeft = (width / 2 + right_shift, 0)
    left.artBox.upperRight = (width / 2 - right_shift, height)
    right.artBox.lowerLeft = (width / 2 + right_shift, 0)

    left.compressContentStreams()
    right.compressContentStreams()

    pdf_writer.addPage(left)
    pdf_writer.addPage(right)

with out_file_path.open(mode="wb") as fhand:
    pdf_writer.write(fhand)
print("Finished", flush=True)

from subprocess import run

print("Creating eps, might take a while... ", end="", flush=True)
run(["pdf2ps", "--eps", str(out_file_path), str(out_file_path.stem) + ".eps"])
print("Finished.", flush=True)
print("Converting back to pdf... ", end="", flush=True)
run(["ps2pdf", str(out_file_path.stem) + ".eps", str(out_file_path)])
print("Finished", flush=True)
print("Removing temporary eps", flush=True)
Path(out_file_path.stem + ".eps").unlink()
print("All done", flush=True)
```

Then, the tex file is quite simple.

```latex
\documentclass[a4paper]{article}
\usepackage{pdfpages}
\begin{document}
	\includepdf[pages={4-43},booklet=true,landscape, delta=0 0]{original_cluebook_split.pdf}
\end{document}
```