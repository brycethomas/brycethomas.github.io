---
layout: post_page
title: A PhD Workflow for Programmers
---

I've been working towards my PhD for a bit over three years now and
have tried a lot of different tools for improving my workflow during
that time.  For the sake of others about to start a PhD or currently
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

This sets you up with a `~/phd` directory with an executable main
build script `build_phd.sh`.

The invariant to try and maintain from this point forward is the
ability to reproduce all PhD outputs with a single call to
`build_phd.sh`.  I'll get you off to a good start building out
`build_phd.sh` in just a sec, but firstly:

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

This sets you up with a skeleton LaTeX document for your PhD.  The
`~/phd` directory now looks like:

```
phd
├── build_phd.sh
└── thesis
    └── main.tex
```

Now open `build_phd.sh` and edit it to read:

```
echo -e "compiling the PhD..."
cd thesis
pdflatex main.tex
bibtex main
# Run twice to get references right.
for run in {1..2}; do pdflatex main.tex; done
cd ../
```


Your thesis will now be compiled everytime you run `build_phd.sh`.

The idea is that over time you'll start fleshing out your computations
in `build_phd.sh` and your thesis in `main.tex`.  A big portion of
`main.tex` will just be your prose.  But, it will also contain
figures, tables, variables&mdash; *data*.  Before we get onto how to
handle this properly, let's talk about how not to.

The greedy algorithm for completing a PhD is to take your interesting
results and manually enter/manipulate/copy/paste them into your
thesis.  The problem with this strategy is thinking you'll only ever
produce and transfer the results into the thesis once.  In reality,
you'll produce the results, insert them into the thesis then later
realise they need to be redone for one reason or another.  On a
timescale of minutes to days, expect the presentation of your results
(or the results themselves) to change 10's of times.  At first this
doesn't seem like much of a problem if you enjoy copy/paste.
Labouriousness isn't the only problem though.  The bigger issue is
that by violating the
[DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) principle
you've now got your results in two places&mdash;wherever you output
them when you produced them and in your thesis document.  You're
essentially relying on yourself to remember this and update the thesis
accordingly when the results inevitably change.

Things only get worse when you start submitting your work for peer
review.  As it turns out, academia is slow as shit when it comes to
giving feedback.  It typically takes on the order of months to get a
reviewer's feedback on a paper.  And guess what&mdash;they're probably
going to be critical of at least one of your results or at a minimum
how they're presented.  By the time you get reviewer feedback six
months have passed and you have NFI how you generated your results in
the first place.  Revising prose will feel like a walk in the park
compared to revising results.

The thing about the copy/paste approach to getting results into your
thesis is that it allows you to get lazy on results *production* as
well.  I'm telling you this from firsthand experience.  I once wrote a
set of Java simulations for a paper I submitted.  Several months after
I submitted the paper I got feedback indicating there were
improvements that needed to be made.  I couldn't remember a damn thing
about how I generated the results.  I couldn't even remember the name
of the simulator I used.  I still can't.  Did I run it in Eclipse?
Did it use any special configuration file?  What tool did I use to
plot the results?  What format was the intermediate data in between
running the simulation and plotting the results?  Did I even write a
script to plot the results, or did I do it all interactively on the
command line?  This folks, is one very good reason to automate your
PhD outputs.

With the greedy solution out of the way, onto the optimal solution.
The optimal solution is to build out the `build_phd.sh` script to
compute all of your results from the raw input data and then inject
those results into the thesis.  An example of this would be to add
code in `build_phd.sh` that runs a simulation over your raw input
data, generates an important figure `my_awesome_figure.pdf` and saves
it into your thesis directory.  You end up with a directory structure
like:

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

somewhere in `main.tex`.  Now whenever you run `build_phd.sh`,
`my_awesome_figure.pdf` gets regenerated based on your code and
recompiled into your thesis.  `build_phd.sh` is your record of
precisely how you did it, which you can refer back to long after
you've forgotten the specifics.  

Another good candidate for automation are simple variables derived
from computation.  Say you performed your analysis over a trace with
283 widgets, or at least you think you did, until you found out there
were actually 284 widgets.  If you'd been following the greedy
strategy, you would have hard-coded the value 283 everywhere in your
thesis and need to go back and update it.  To treat it as data instead
of prose, you could have a step in `build_phd.sh` that emits a file
`widgets.tex` containing the number of widgets.  So now your directory
looks like:

```
phd
├── build_phd.sh
└── thesis
    ├── main.tex
    ├── my_awesome_figure.pdf
    └── widgets.tex
```

In the thesis you then replace the literal `283` with
`\input{widgets}`.  At first `widgets.tex` reads `283`, and so `283`
ends up in the thesis.  Later on when you fix up the computation,
`widgets.tex` reads `284` and this automatically makes its way into
the thesis.  There's no worrying about forgetting to replace the value
anywhere because you've explicitly tied it to the computation.

You can obviously make your directory structure as simple or as
complicated as you like.  The key point is to try and automate the
whole sequence of events starting at input data and ending at the
presentation of results in your thesis.

Aside from the aforementioned benefits of build automation, there's
another nice by-product you get for free and the research community
will thank you for it.  By automating your work, you make your results
trivally reproducible.  Other fields don't have this luxury.  They
have to settle for describing their methods in prose.  You on the
other hand get a non-ambiguous description of your method that someone
else can execute, analyze and modify.  Even if you need to add
anonymizing features to your data before distributing your code,
you've still got a very solid foundation to build on.

So automation has largely been covered.  There's still the question of
how this all works when you have to collaborate with others,
primarily, your supervisors.  The good and/or bad news is, your
supervisors will probably never read nor execute a single line of your
code.  They will read and review your written outputs though.  I've
found the best way to collaborate on academic works (papers, technical
reports, thesis) is through
[Scribtex](https://scribtex.sharelatex.com/).  Scribtex lets you write
and compile LaTeX documents in a web browser and share them with
collaborators.  The good thing about this is, your collaborators can
view and edit the document in a web browser.  This means you're not
relying on your collaborator's having LaTeX setup on their machine in
order to modify the document.  The other nice thing about Scribtex is
that it includes a version history feature so when your supervisor
asks what changed since the last time they saw it, you've got a URL to
point them to.

At this point I know what you're thinking&mdash;how am I meant to
automate my PhD build when my thesis document is hosted on the web?
Well, the good news is, Scribtex supports
[Git](http://git-scm.com/book/en/Getting-Started).  This means you can
sync the document to your own computer into a subdirectory of your phd
directory, run your build script and push the changes back to Scribtex
when you're ready.  This is actually good for other reasons too.
Firstly, it's now backed up locally on your machine decorrelating your
failures.  You see it's 2014 and losing critical data because you only
ever kept a single copy or multiple copies in one geographic location
is becoming embarrassing.  The second reason is, *you* shouldn't be
writing in a web browser&mdash;leave that to your supervisors who will be
making minor edits.  You want to be using a proper text editor.
Seriously, you're going to write a lot throughout your PhD, and you'll
start realising productivity gains after a few weeks of using a decent
text editor.  So bite the bullet, pick one, and learn it.  If you
can't decide which, use Emacs.

A little note re. "you should definitely use Scribtex"&mdash;you might be
able to get away with using Dropbox.  Dropbox has a kind-of versioning
system if you look hard enough.  Personally I prefer the Git workflow
(that your supervisor's never have to know about) that comes with
Scribtex and find it's better setup for the task at hand.  Seriously,
you nominate a main file for each project on Scribtex, and there's
literally a big "View as PDF" button that your supervisor's can then
press to see the document.  There's also a very prominent "History"
button, which as an added bonus will include commit comments alongside
each version if you've been using Git.


While we're on the topic of Git, one more thing&mdash;you should also
use it for your main `phd` directory.  I won't try and explain Git
itself here; there's heaps of good material you can lookup on Google.
The way I like to do this though is to have a main Git repository for
my phd, i.e. I turn the `phd` directory into a Git repository.  Then,
for each paper, I add the corresponding Scribtex Git repository as a
[submodule](http://git-scm.com/book/en/Git-Tools-Submodules) of the
`phd` repository.  Essentially the way submodules work is that you tie
a specific version (commit) of the submodule to a specific version of
the parent project when you make a commit in the parent project.  This
means versions of your code get tied to versions of your papers in
each of the main repository's commits.  The way things have played
out, GitHub has become synonymous with Git and seems to be the cloud
host of choice for syncing Git repositories.  At time of writing
[GitHub offer free micro accounts](https://education.github.com/) to
students meaning you can host your code privately online.

So that's pretty much it.  Once you get the basic tooling in place to
automate your PhD build and ease collaboration you just keep building
out the `build_phd.sh` script with your computations.  Of course this
doesn't mean shoving all your code in one file&mdash;you should be
calling subscripts, but you get the general idea.

I should probably leave it at that, but there's one other sensitive
topic that needs to be addressed&mdash;Microsoft Word.  Like a
programmer that refuses to learn how to touch type, occasionally
you'll come across an academic that refuses to learn LaTeX.  To be
clear, LaTeX is a massive pain in the arse, but at least it's a plain
text pain in the arse.  With Word, it's very hard (if not impossible?)
to automate your PhD build as described above because you have no way
to inject variables or figures into your document.  You're also
missing out on working in a good keyboard-oriented text editor.  Yes,
both LaTeX and good text editors have a steeper learning curve than
Word, but you'll reach the intersection point long before completing
your PhD.  So, do what you can to convince your supervisors to work
with you in LaTeX.  If you can't do that, see if they're happy to just
print things and make handwritten notes.  It's not optimal for you,
but often it is for them.  You'll probably find they do this
regardless of how they feel about the Word/LaTeX debate.

And now for some disclaimers.  My PhD is very much
data-driven&mdash;take input trace data, perform computations over it
and generate results.  If you're doing a PhD in pure mathematics then
maybe the workflow above doesn't make sense for you.  But I'm sure
you've figured that out.  Take what's relevant and leave the rest.
Another thing I can see some people having a sook about - the command
line.  I'll remind you that the title of this post is "A PhD Workflow
*for Programmers*".  Still, you might have been like me and
miraculously made it through an IT degree that technically never
required you to touch the command line, if you chose your electives
just right.  I can tell you from experience, picking up the command
line really isn't that scary, particularly not on the timescale of a
PhD.  You'll also come to find that [I/O
redirection](http://linuxcommand.org/lts0060.php) often trivially
solves routine computation steps with built in commands, meaning you
have to write less code.  Even if you're familiar with *none* of the
tools mentioned in this post (Git, Shell scripting, LaTeX, Emacs),
think of these as learning opportunities.  You'll walk away with an
appreciation of a widely used version control system, a generic
scripting tool, professional-grade typesetter and a text editor that
might even one day become your preferred IDE.

Final note: don't get *too* caught up in tooling.  You want to leave
some time for, y'know, research.