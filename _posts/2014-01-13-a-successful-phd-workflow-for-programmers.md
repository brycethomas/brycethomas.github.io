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
text.  Not only is this critical for automating the production (and
re-production) of your results, but mastering and working in a plain
text editor turns out to be a fairly enjoyable experience.  If I were
to start my PhD over, the first thing I would do is this:

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
PhD.  What's going to happen over time is, you're going to start
writing your thesis in `main.tex`.  A big portion of it will just be
your amazing prose.  But, you're also going to have figures, tables,
variables, *data*.  Before we get onto how to handle this properly,
let's talk about how not to.

The greedy algorithm for completing your PhD is to take your
interesting results (this is what they expect you to generate in
exchange for your scholarship money), and then manually
enter/manipulate or copy/paste them into your thesis.  The problem
with this strategy is that you're too optimistic.  The folly starts
with thinking you're only ever going to have to produce and transfer
your results into your thesis document once.  In reality, you'll
produce your results, slap them into your thesis, then a few
re-readings later realise they're slightly wrong and/or crap in some
way and need to be reproduced/further prettified.  On a timescale of
minutes to days, you should expect that either the presentation of the
results or the results themselves will need to updated 10's of times.
At first this doesn't seem like much of a problem if you enjoy
copy/paste.  The more serious problem though is that by violating the
DRY principle you've now got your results in two places -- wherever
you output them when you produced them, and in your thesis document.
You're now relying on yourself to remember this and update your thesis
accordingly when your results inevitably change.  Things only get
worse when you start submitting your work for review.  As it turns
out, academia is slow as shit with giving feedback.  It typically
takes on the order of months to get a reviewer's feedback on a paper.
And guess what -- they're probably going to be critical of at least
one of your results or how they're presented.  By now you have no
fucking clue how you generated your results in the first place -- it's
been 6 months.  If you thinking revising the prose of a paper is
painful, think about revising your results.  See the thing about the
copy/paste approach to getting results into your thesis is that it
allows you to get lazy on the *production* side as well.  6 months on
you won't remember what scripts you ran in what order or what magic
sequence of buttons you pressed in Excel to generate your results.
And it's easy to produce your results in this way when you convince
yourself you'll only have to transfer them over to a document once.
Now onto the better way.

The optimal solution is to build out your `build_phd.sh` script to
compute all of your results from the raw input data and then inject
those results into your thesis.  An example of this is have code in
`build_phd.sh` to generate an important .pdf figure from the raw input
data as well as save that figure into your thesis directory.

At this point you might be wondering why you just went to all that
effort to write a script to compile a latex document, you normally
just do that by pressing the pretty play button in LyX, right?  Wrong.
The perennial issue you're going to face if you rely on GUIs to
produce results is lack of automation and painstaking repetition.  At
the moment your thesis document is super simple -- it doesn't include
any figures or calculated results.  As you analyze data and include
results though, you're going to want an automated way to build those
results into a final deliverable product e.g. the thesis or a paper.
To be fair, having to press the "compile" button in a LaTeX document
will be the least of your worries.  Where things get really painful is
when you rely on GUIs to *produce the results* going into your LaTeX
documents.  Here's the thing about using GUIs to produce your results
-- whether you think so or not, you *will* end up needing to produce
those results again and you *will* forget what steps you took to
produce them.  Also you *won't* make a written record of those steps,
as it too is incredibly painful.  The best way around this is to write
scripts that perform the *equivalent* (not identical) GUI steps and
then call those scripts from `build_phd.sh`.  The first paper I ever
wrote I failed to heed this advice and paid for it.  Even though I
wrote code to describe the simulation itself, I never recorded the
correct order of steps to actually run it.  As it turns out, academia
is slow as hell at giving feedback.  You'll submit a paper and be
relatively care-free for 6 months.  Then, the review will come back --
*this work is good rar rar rar but could be improved if you instead
did x*, where *x* is some small variant on your existing work.  Cue
inertia.  The good news is, you'll find in research there are nearly
always equivalent command line tools to GUIs.

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
except for all the rest.  Which makes LaTeX better than Word.  Word is
a great tool for basic documents written by your Mom, but not for your
thesis.  The biggest reason not to use Word is it breaks the cardinal
rule: *everything in plain text*.  Try automating in Word the
insertion of figures derived from your calculated results and let me
know how you go.  The second reason not to use Word is that like most
GUI tools it makes for a lowsy text editor.  You'll write a lot
throughout a PhD so it's worth taking the time to learn a decent text
editor (e.g. Emacs, Vim, etc.).  One of your objectives, for both your
PhD and becoming a more efficient computer user in general should be
to move away from relying on the mouse.  The GUI itself was a great
idea for making apps easy to learn and discoverable.  The computer
mouse, though lauded as a great invention, is not a particularly
efficient way to navigate a computer.  It's not even a particularly
efficient way to navigate a GUI for that matter.  If you've ever used
a tool like Vimium in Google Chrome you'll see what I mean.  With a
single key combination you can bring up these little label thingies
over all the clickable links that prompt you to enter another short
letter combination to follow that link, rather than futzing around
with the mouse.  GUI and keyboards don't need to be mutually
exclusive, but I digress.  There other reasons not to use Word as
well.  Firstly, as crap as PDF (the output of LaTeX) is as an editing
format, at least you reliably know other people will be able of
reading it.  There's nothing worse than getting an email *Oh hai I've
attached my paper blurblurblur.doc pls read and give feedback tnx*.
And then you don't have Word, so you open it in a purportedly Word
compatible program and it's mangled, usually not quite enough for you
to actually know that it's mangled.  You usually just assume the
sender messed something up.  Seriously though, even relatively
unsophisticated computer users have multiple devices now running
different operating systems.  Expecting other people to have Word
installed, especially in a technical field, is an insult.  There's
other lesser problems with Word too, like the default font choices
among other things looking like crap.  LaTeX was built for producing
publication quality works, and for all its shortcomings, it at least
does that.  Also, because LaTeX is plain text, it plays nice with
version control, which I'll get to shortly.

### How Not to Lose All Your Shit and About Version Control 

It doesn't happen as much anymore, but every now and then I hear a
story about how so and so lost all their hard work when their hard
drive failed.  For Christ's sake, it's 2014, at least get a Dropbox
account.  There is a better option though and that's version control.
If you've been following along so far you've already made a concerted
effort to keep all your various PhD artifacts in plain text which just
so happens is version control's forte.  Being a student you can get a
free Micro account on GitHub, so go get one.  This will give you a way
to keep a versioned copy of all your work stored privately on the
cloud.  Dropbox kind of sort of has version control if you look hard
enough for it, but it's rudimentary.  Dropbox also doesn't have any
notion of branching, 

### Collaborating With Your Supervisor(s) on Papers

I've had a moan already about receiving Microsoft Word documents from
people who expect me to be able to do something with it.  Then I
praised LaTeX as a superior option because it produces PDF output.
What I conveniently failed to mention is that when it comes to editing
documents, there's even less chance the people you're sending stuff to
have LaTeX installed than even Word.  Plus, even if they do, chances
are you'll decide you want to use some obscure LaTeX package that your
supervisor doesn't have installed, and so the document fails to build
on their machine.

<!--  Once you start writing your thesis (and papers) you're
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
using Microsoft Word, had to insert a figure and -->