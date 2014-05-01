---
layout: post_page
title: A PhD Workflow for Programmers
---

I've been working towards my PhD for almost three years now and have
tried a lot of different tools for improving my workflow during that
time.  For the sake of others about to start a PhD or currently
working towards one, I've decided to document what I've found works.
I can summarise it in two words -- *build automation*.  Ideally you
want to be able to build all of your intellectual outputs with a
single call on the command line.  Do this and you'll have an
unambiguous record of the steps you took to produce your results while
solving all nature of other problems along the way, which is what this
post is largely about.

I'd suggest starting your PhD like this:

```bash
cd ~
mkdir -p phd
cd phd
echo "echo \"TODO: complete PhD\"" > build_phd.sh
chmod a+x ./build_phd.sh
build_phd.sh
```

This sets you up with a `phd` directory and an executable main build
script named `build_phd.sh`.

The objective from this point forward is to maintain the ability to
reproduce all PhD outputs with a single call to `~/phd/build_phd.sh`.
Allow me to get you off to a good start on building out
`build_phd.sh`:

```bash
cd ~/phd
mkdir -p thesis
cd thesis
touch main.tex
```

Then open `main.tex` and add the following:

```latex
\documentclass{article}
\begin{document}
TODO: write my thesis.
\end{document}
``` 

Now open `~/phd/build_phd.sh` and edit it to read:

```
echo -e "compiling my PhD..."
cd thesis
pdflatex main.tex
bibtex main
# Run twice to get references right.
for run in {1..2}; do pdflatex main.tex; done
cd ../
```

At this point you've got a main LaTeX document setup for your thesis
that gets recompiled every time you run `build_phd.sh`.

The idea is that over time you'll start fleshing out your thesis in
`main.tex`.  A big portion of it will just be your prose.  But, it
will also contain figures, tables, variables -- *data*.  Before we get
onto how to handle this properly, let's talk about how not to.

The greedy algorithm for completing a PhD is to take your interesting
results (they expect you to generate these in return for your
scholarship stipend) and manually enter/manipulate/copy/paste them
into your thesis.  The problem with this strategy is your optimism.
The folly starts with thinking you're only ever going to have to
produce and transfer your results into your thesis once.  In reality,
you'll produce the results, slap them into your thesis then later
realise they're slightly wrong and/or crap in some way and need to be
redone.  On a timescale of minutes to days, you should expect the
presentation of your results (or the results themselves) will change
10's of times.  At first this doesn't seem like much of a problem if
you enjoy copy/paste.  Labouriousness isn't the only problem though.
The bigger issue is that by violating the DRY principle you've now got
your results in two places -- wherever you output them when you
produced them, and in your thesis document.  You're now relying on
yourself to remember this and update your thesis accordingly when your
results inevitably change.  

Things only get worse when you start submitting your work for review.
As it turns out, academia is slow as shit when it comes to giving
feedback.  It typically takes on the order of months to get a
reviewer's feedback on a paper.  And guess what -- they're probably
going to be critical of at least one of your results or at least how
they're presented.  By the time you get the review six months has
passed and you have NFI how you generated your results in the first
place.  Revising prose is a walk in the park compared to revising
results generated six months ago.  The thing about the copy/paste
approach to getting results into your thesis is that it allows you to
get lazy on results *production* as well.  I'm telling you this from
firsthand experience.  I once wrote a set of Java simulations for a
paper I submitted.  Several months after I submitted the paper I got
feedback indicating there were improvements that needed to be made.  I
couldn't remember a damn thing about how I generated the results.  I
couldn't even remember the name of the simulator I used.  I still
can't.  Did I run it in Eclipse?  Did it use any special configuration
file?  What tool did I use to plot the results?  What format was the
intermediate data in between running the simulation and plotting the
results?  Did I even write a script to plot the results, or did I do
it all interactively on the command line?  This folks, is one very
good reason to automate your PhD outputs.

With the greedy solution out of the way, onto the optimal solution.
The optimal solution is to build out your `build_phd.sh` script to
compute all of your results from the raw input data and then inject
those results into your thesis.  An example of this would be to add
code in `build_phd.sh` that runs a simulation over your raw input
data, generates an important figure `my_awesome_figure.pdf` and saves
it into your thesis directory.  So you end up with a directory
structure like this:

```
phd
├── build_phd.sh
└── thesis
    ├── main.tex
    └── my_awesome_figure.pdf
```

where you've got:

``` 
\begin{figure} 
\includegraphics{my_awesome_figure} 
\end{figure}
``` 

somewhere in `main.tex`.  Now you can generate `my_awesome_figure.pdf`
with a single call and you've got a written record in `build_phd.sh`
of precisely how you did it.  Another good candidate for automation
are simple variables derived from your computation.  Say you performed
your analysis over a trace with 283 widgets, or at least you think you
did, until you found out there were actually 284 widgets.  If you'd
been following the greedy strategy, you would have hard-coded the
value 283 everywhere in your thesis and need to go back and update it.
To treat it as data instead of prose, you could have a step in your
`build_phd.sh` script that emitted a file `widgets.tex` that simply
read `283`.  In the thesis you then replace the literal `283` with
`\input{widgets}`.

So now your directory looks like:

```
phd
├── build_phd.sh
└── thesis
    ├── main.tex
    ├── my_awesome_figure.pdf
    └── widgets.tex
```

Now when you update your data processing and find there are now 284
widgets, `284` automatically finds its way into your thesis.  You can
obviously make your directory structure as simple or as complicated as
you like.  The key point is to automate the whole sequence of events
starting at input data and ending at the presentation of results in
your thesis.

Aside from the aforementioned benefits of build automation, there's
another nice by-product you get for free and the research community
will thank you for it.  By automating your work, you make your results
trivally reproducible.  Other fields don't have this luxury.  They
have to settle for describing their methods in prose.  You on the
other hand get a non-ambiguous description of your method that someone
else can execute, analyze and modify.  Even if you need to add
anonymizing features to your data before distributing your code,
you've still got a very solid foundation to build from.

So automation has largely been covered.  There's still the question of
how this all works when you have to collaborate with others, primarily
your supervisors.  The good and/or bad news is, your supervisors will
probably never read nor execute a single line of your code.  They will
read and review your written outputs though.  I've found the best way
to collaborate on academic works (papers, technical reports, thesis)
is through Scribtex (TODO: link).  Scribtex lets you write and compile
LaTeX documents in a web browser and share them with collaborators.
The single best part about this is, your collaborators can view and
edit the document in a web browser.  This is good, because now you're
not relying on your collaborator's having LaTeX setup on their machine
in order to modify the document.  This is an importance difference
between Scribtex and Dropbox.  If you were using Dropbox, you could
sync the documents with your supervisors just fine, *but*, you're now
relying on your supervisors having LaTeX configured correctly.

The other nice thing about Scribtex is that it includes a version
history feature so when your supervisor asks what changed since the
last time they saw it, you've got a URL to point them too.  At this
point I know what you're thinking -- how am I meant to automate my PhD
build when my thesis document is hosted on the web?  Well, the good
news is, Scribtex supports Git Distributed Version Control.  This
means you can sync the document to your own computer into a
subdirectory of your phd directory, run your build script and push the
changes back to Scribtex when you're ready.  This is actually good for
other reasons too.  Firstly, it's now backed up locally on your
machine decorrelating your failures.  You see it's 2014 and losing
critical data because you only ever kept a single copy or multiple
copies in one geographic location makes you an embarrassment to the
discipline.  The second reason is, *you* shouldn't be writing in a web
browser -- leave that to your supervisors who will be making minor
edits.  You want to be using a proper text editor.  Seriously, you're
going to write a lot throughout your PhD, and you'll start realising
productivity gains after a few days of using a decent text editor.  So
bite the bullet, pick one, and learn it.  If you can't decide which,
use Emacs.

A little caveat re. "you should definitely use Scribtex" -- depending
on your supervisors, Dropbox may suffice.  You think you know where
this is going, don't you?  "If *my* supervisors have LaTeX setup on
all their machines, then Scribtex provides no extra value to us".
Actually, that's not the reason.  The reason is, there's the distinct
possibility your supervisor will never make *any* changes to your
`.tex` documents.  Yup.  Instead, they'll print out the PDF, write all
over it in red pen and scan and email the whole thing back to you
instead.  I shit you not.  If this baffles you at first, you're
probably not alone.  It's actually not as inefficient or irrational as
you might think though.  See as an undergrad you developed the view
that printing and scanning is a time and cost expensive operation,
which it was on your $29 bubblejet.  In any given week you were
practically trading off feeding yourself vs. feeding your printer more
ink catridges. Your professors on the other hand have been living
above the poverty line and have become accustom to creature comforts
like printing allowances.  On business-grade printers, printing is
usually pretty pain-free and cost effective.  And despite advances in
tablet readers, printed documents are still the most tactile format
and the easiest to review.  Especially in the Journal of Archaic
Layout's double-column-3-point-no-margin page format.  So
unfortunately here your incentives aren't aligned.  You'd rather your
supervisors just edit the `.tex` file, but they'd rather just scribble
on a hard copy.  The other reason your supervisors might prefer
printing documents is that reading your paper only reaches top
priority when they board a plane and the best available alternative is
in-flight magazine.  You think I'm joking, I'm not.  If you want your
writing reviewed, get a printed copy to your supervisor the day before
they have a flight scheduled.

So anyway, just a quick word on Git.  It's a Distributed Version
Control System that you can read all about on the Interwebs.  I've
already mentioned that you can and should use it with Scribtex.  You
should also use it for your main `phd` directory too though.  Go
request a free micro plan on Github and get setup.  Then, turn your
top level `phd` directory into a Git repository and push it to Github.
The idea with Scribtex integration is to add each scribtex document
(e.g. *paper1*, *paper2*) as a Git submodule to your main repository.
This way you can do all your phd build automation, writing results
into the Scribtex directories and pushing those changes back to
Scribtex whenever your ready.  Your supervisors won't know (and
doesn't care) that you're using Git to manage your Scribtex documents.
It's for the benefit of *your* workflow.  Having your documents hosted
on Scribtex just gives you the highest probability of having your
supervisors actually edit the `.tex`, rather than the
print/scribble/scan routine described above.  By the way, reassure
your supervisors they can't break anything by editing the Scribtex
document.  There's a version history on Scribtex as well as a Git
version history now that you're using Scribtex with Git.

So that's pretty much it.  Get the tooling in place to automate the
build of your PhD and to ease collaboration with your supervisors.  I
should probably leave it at that, but there's one other sensitive
topic that needs to be addressed -- Microsoft Word.  Like a programmer
that refuses to learn how to touch type, occasionally you'll come
across an academic that refuses to learn LaTeX.  To be clear, LaTeX is
a massive pain in the arse, but at least it's a plain text pain in the
arse.  With Word, it's very hard (if not impossible?) to automate your
PhD build as described above because you have no way to inject
variables or figures into your document.  You're also missing out on
working in a good text editor.  Yes, both LaTeX and good text editors
have a steeper learning curve than Word, but you'll reach the
intersection point pretty quickly during the course of completing a
PhD.

And now for some disclaimers.  My PhD is very much data-driven -- take
input trace data, perform computations over it and generate results.
If you're doing a PhD in pure mathematics then maybe the workflow
above doesn't make sense for you.  But you're a PhD candidate now and
smart enough to take what's relevant and leave the rest.  Another
thing I can see some people having a sook about - the command line.  I
was awarded a degree in Information Technology specializing in
software development, without ever having to touch the command line.
If someone managed to construct your degree in such a way that you
miraculously missed out on this too, all dem commands may look
daunting.  But if you're starting a PhD you'll be around a few years,
and it pays to learn this stuff.  Besides, it turns out experience
with Linux, build automation and distributed version control are
is a useful thing for software developers to have. 