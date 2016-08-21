---
layout: post
title: Octave GUI in Ubuntu 13.x - First Impressions
date: 2013-11-29 14:26:34.000000000 +11:00
type: post
published: true
status: publish
categories:
- Miscellaneous
tags:
- GNU Octave
- Octave
- Octave GUI
- Ubuntu
meta:
  _wpas_done_all: '1'
  _wpcom_is_markdown: '1'
  _publicize_pending: '1'
  publicize_google_plus_url: https://plus.google.com/116873873608085229608/posts/Gg8HuNz9VWE
  _wpas_done_5385652: '1'
  _publicize_done_external: a:1:{s:11:"google_plus";a:1:{s:21:"116873873608085229608";b:1;}}
  publicize_twitter_user: nikolaygrozev
  publicize_twitter_url: http://t.co/VEaVmWf7jm
  _wpas_done_5385644: '1'
  publicize_linkedin_url: http://www.linkedin.com/updates?discuss=&scope=30071619&stype=M&topic=5812028592883781632&type=U&a=r4ZI
  _wpas_done_5385639: '1'
  _wpas_skip_5385652: '1'
  _wpas_skip_5385644: '1'
  _wpas_skip_5385639: '1'
  geo_public: '0'
  _edit_last: '1'
  _kad_blog_head: default
  _kad_post_summery: default
  _kad_post_sidebar: 'yes'
  _kad_sidebar_choice: sidebar-primary
  _kad_blog_author: default
  _kad_blog_carousel_similar: default
  _thumbnail_id: '30'
author:
  login: nikolaygrozev
  email: nikolay.grozev@gmail.com
  display_name: nikolaygrozev
  first_name: 'Nikolay'
  last_name: 'Grozev'
---

[GNU Octave](http://www.gnu.org/software/octave/) is a powerful open source alternative to Matlab. A major obstacle to its adoption is the lack of simple and convenient graphical environment (like [RStudio](http://www.rstudio.com/) for R). The most famous Octave GUI front-end [QtOctave is no longer developed](https://sites.google.com/site/davidecittaro/apple-stuff/qtoctavenomoresupported) and the new environments like [Domain Math IDE](https://sites.google.com/site/domainmathide/) and [Octclipse](http://sourceforge.net/projects/octclipse/) still haven't gained maturity. Developing Octave code in an external editor is not too hard, as many editors (e.g. [Geany](http://www.geany.org/)) support Matlab/Octave syntax highlighting and even code completion. For me, the real problem is the lack of an integrated debugger, which makes diagnosing and fixing bugs pretty hard.

Fortunately, the latest version 3.7 of Octave introduces a new GUI front-end with an integrated debugger. Octave 3.7 has not been officially released, but you can download the code, build and test it locally. In Ubuntu 13.x the following script does the trick:

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

The installation may take a while. Depending on your environment you may also like to install the following packages: Autoconf; Automake; Flex; Gnulib; gperf; Gzip; Perl; Rsync. For most Ubuntu installations, these should be already present.

Then you can start Octave GUI from the newly created folder "octave":

```bash
./run-octave
```

This should open the new Octave GUI. It seems like the major functionalities (syntax highlighting, code completion, debugger, help) are already there :). Here are some screen-shots:

<!-------------------------------------------- Image Galery -------------------------------------------->
<a class="image-popup-fit-width" href="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/1-welcome-screen.png" 
    title="Welcome Screen.">
	<img src="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/1-welcome-screen.png" width="15%">
</a>
<a class="image-popup-fit-width" href="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/2-command-window.png" 
    title='A standard terminal for invoking commands. Session variables are listed in the "Workspace" panel'>
	<img src="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/2-command-window.png" width="15%">
</a>
<a class="image-popup-fit-width" href="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/3-text-editor-and-automcomplete1.png" 
    title="The code editor offers code completion.">
	<img src="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/3-text-editor-and-automcomplete1.png" width="15%">
</a>
<a class="image-popup-fit-width" href="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/4-graphics.png" 
    title="Graphics.">
	<img src="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/4-graphics.png" width="15%">
</a>
<a class="image-popup-fit-width" href="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/5-debugging.png" 
    title="Debugging! The breakpoint is in red, the yellow arrow indicates curren line in the code.">
	<img src="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/5-debugging.png" width="15%">
</a>
<a class="image-popup-fit-width" href="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/6-documentation.png" 
    title="The documentation/help is organised hierarchically.">
	<img src="/assets/images/Octave GUI in Ubuntu 13.x - First Impressions/6-documentation.png" width="15%">
</a>
<!-------------------------------------------- Image Galery -------------------------------------------->

