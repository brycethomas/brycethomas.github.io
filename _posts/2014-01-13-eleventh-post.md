---
layout: post_page
title: A Successful PhD Workflow for Programmers
---

I've been working towards my PhD for almost three years now and have
tried a lot of different tools for improving my workflow during that
time.  For the sake of others about to start a PhD or currently
studying towards one, I've decided to document what I've found works.
I can summarise it in two words -- *plain text*.  You want all of the
artifacts you create throughout your PhD to be reproducible from plain
text.  The first thing you ought to do when you start your PhD is
this:

```bash
cd ~
mkdir -p phd
cd phd
echo "echo \"TODO: complete PhD\"" > build_phd.sh
chmod a+x ./build_phd.sh
build_phd.sh
```

The holy grail now is to maintain the ability to compile the entirety
of your PhD artifacts with a single call to `~/phd/build_phd.sh`.  
Allow me to get you off to a good start on building out
`build_phd.sh`:

```bash
cd ~/phd
mkdir -p thesis
cd thesis
touch main.tex
```

Then open `main.tex` and add the following:

```
\documentclass{article}
\begin{document}
TODO: write my thesis.
\end{document}
``` 

Now open `~/phd/build_phd.sh` and edit it so it reads:

```
echo -e "compiling my PhD..."
cd thesis
pdflatex main.tex
bibtex main
# Run twice to get references right.
for run in {1..2}; do pdflatex main.tex; done
cd ../
```

You've now taken the first step towards automating the build of your
PhD.  At this point you might be wondering why you just went to all
that effort to write a script to compile a latex document, you
normally just do that by pressing the pretty play button in LyX,
right?  Wrong.  The perennial issue you're going to face if you rely
on GUIs to produce results is lack of automation and painstaking
repetition.  At the moment your thesis document is super simple -- it
doesn't include any figures or calculated results.  As you analyze
data and include results though, you're going to want an automated way
to build those results into a final deliverable product e.g. the
thesis or a paper.  To be fair, having to press the "compile" button
in a LaTeX document will be the least of your worries.  Where things
get really painful is when you rely on GUIs to *produce the results*
going into your LaTeX documents.  Here's the thing about using GUIs to
produce your results -- whether you think so or not, you *will* end up
needing to produce those results again and you *will* forget what
steps you took to produce them.  Also you *won't* make a written
record of those steps, as it too is incredibly painful.  The best way
around this is to write scripts that perform the *equivalent* (not
identical) GUI steps and then call those scripts from `build_phd.sh`.
The first paper I ever wrote I failed to heed this advice and paid for
it.  Even though I wrote code to describe the simulation itself, I never
recorded the correct order of steps to actually run it.  As it turns
out, academia is slow as hell at giving feedback.  You'll submit a
paper and be relatively care-free for 6 months.  Then, the review will
come back -- *this work is good rar rar rar but could be improved if
you instead did x*, where *x* is some small variant on your existing
work.  Cue inertia.  The good news is, you'll find in research there
are nearly always equivalent command line tools to GUIs.

To give a simple example, lets say you have some processing you need
to perform over some data that requires either 20 clicks between
various items in a GUI or can be automated in code.  Let's assume the
output is the figure *my_cool_result.pdf* and it needs to make its way
into your thesis.  Recall our directory structure already looks like
this:

TODO: show structure.

We can add a line to `build_phd.sh` like:

`python my_cool_result_generator.py > ./thesis/my_cool_result.pdf`.

`my_cool_result_generator.py` is now your written record of the steps
required to produce that result.  In programming, people always tell
you to write clear enough code that you could come back to it in 6
months and know what you were doing.  Even bad code is a record.  If
you rely on GUIs you're coming back to your work in 6 months with no
record of anything.

### Microsoft Word in Academia
Let's get this cleared up.  LaTeX is the worst typesetting system
except for all the rest.

  Once you start writing your thesis (and papers) you're
going to f
ind they need figures.  Those figures are going to be output
as the result of processing data.  Chances are you're going to mess up
that processing (and in turn your figures) many times over before you
finally get it right.  Have you ever had to work on a document in
Microsoft Word that required you insert a figure you created using
another program?  And you found the figure didn't look quite right
about 10,000 times over causing you to constantly click back and forth
between programs, reproducing the figure and then reinserting the
latest version into the document?  That's what we're trying to avoid
here.  LaTeX at least has the notion of referencing figures as file
names, so you can at least just update the file, manually recompile
and get the result you were after.  At a minimum you ought to  

 where you needed to insert before Have you ever been
using Microsoft Word, had to insert a figure and