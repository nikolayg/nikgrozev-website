---
layout: post
title: Octave GUI in Ubuntu 13.x - First Impressions
date: 2013-11-29 14:26:34.000000000
type: post
excerpt: 
    GNU Octave is a powerful open source alternative to Matlab. 
    The latest version 3.7 of Octave introduces a new GUI front-end with an integrated debugger. 
    Octave 3.7 has not been officially released, but you can download the code, build and test it locally ...
published: true
status: publish
categories:
- Octave
tags:
- Data Science
- Octave
- Octave GUI
- Ubuntu
---

[GNU Octave](http://www.gnu.org/software/octave/) is a powerful open source alternative to Matlab. 
A major obstacle to its adoption is the lack of simple and convenient graphical environment 
(like [RStudio](http://www.rstudio.com/) for R). The most famous Octave GUI front-end 
[QtOctave is no longer developed](https://sites.google.com/site/davidecittaro/apple-stuff/qtoctavenomoresupported) 
and the new environments like [Domain Math IDE](https://sites.google.com/site/domainmathide/) and 
[Octclipse](http://sourceforge.net/projects/octclipse/) still haven't gained maturity. 
Developing Octave code in an external editor is not too hard, as many editors (e.g. [Geany](http://www.geany.org/)) 
support Matlab/Octave syntax highlighting and even code completion. For me, the real problem is the lack of 
an integrated debugger, which makes diagnosing and fixing bugs pretty hard.

Fortunately, the latest version 3.7 of Octave introduces a new GUI front-end with an integrated debugger. 
Octave 3.7 has not been officially released, but you can download the code, build and test it locally. 
In Ubuntu 13.x the following script does the trick:

```bash
#!/bin/bash

sudo apt-get install mercurial
sudo apt-get install libtool
sudo apt-get install libqt4-*
sudo apt-get install libqscintilla-*
sudo apt-get install bison
sudo apt-get install llvm
sudo apt-get build-dep octave

hg clone http://www.octave.org/hg/octave
cd octave
hg update default

./bootstrap
./configure
make
```

The installation may take a while. Depending on your environment you may also like to install the following packages: 
*Autoconf*; *Automake*; *Flex*; *Gnulib*; *gperf*; *Gzip*; *Perl*; *Rsync*. For most Ubuntu installations, these should 
be already present.

Then you can start Octave GUI from the newly created folder "octave":

```bash
./run-octave
```

This should open the new Octave GUI. It seems like the major functionalities (syntax highlighting, code completion, debugger, help) 
are already there :-). Here are some screen-shots:

<!-------------------------------------------- Image Galery -------------------------------------------->
<figure class="third">
    <a class="image-popup-fit-width" 
        href="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/1-welcome-screen.png" 
        title="Welcome Screen.">
        <img src="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/1-welcome-screen.png">
    </a>
    <a class="image-popup-fit-width" 
        href="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/2-command-window.png" 
        title='A standard terminal for invoking commands. Session variables are listed in the "Workspace" panel'>
        <img src="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/2-command-window.png">
    </a>
    <a class="image-popup-fit-width" 
        href="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/3-text-editor-and-automcomplete1.png" 
        title="The code editor offers code completion.">
        <img src="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/3-text-editor-and-automcomplete1.png">
    </a>
    <a class="image-popup-fit-width" 
        href="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/4-graphics.png" 
        title="Graphics.">
        <img src="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/4-graphics.png">
    </a>
    <a class="image-popup-fit-width" 
        href="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/5-debugging.png" 
        title="Debugging! The breakpoint is in red, the yellow arrow indicates curren line in the code.">
        <img src="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/5-debugging.png">
    </a>
    <a class="image-popup-fit-width" 
        href="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/6-documentation.png" 
        title="The documentation/help is organised hierarchically.">
        <img src="/images/blog/Octave GUI in Ubuntu 13.x - First Impressions/6-documentation.png">
    </a>
    <figcaption>Overview of Octave GUI.</figcaption>
</figure>
<!-------------------------------------------- Image Galery -------------------------------------------->

