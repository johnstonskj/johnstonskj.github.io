---
title: Google Code + Git = Fail
layout: postx
category: code
tags: [archive, git]
---

Well, I wanted to start a new project on Google Code and typically have used SVN before but as I'm using Git more and 
more on my machine for personal work it seemed a great chance to use their new Git support. So, I created a new 
project, set Git as my version control and went straight to the command line and cloned my new project.

```bash
% git clone https://myusername@code.google.com/p/myproject/ 
Cloning into myproject...
warning: You appear to have cloned an empty repository.
```

Well, that all seemed good, so I added a file, committed locally and tried a push ...

```bash
% vi LICENSE
% git add LICENSE 
% git commit
[master (root-commit) 4551a7d] Added file
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 LICENSE
% git push
fatal: https://code.google.com/p/myproject/info/refs not found: did you run git update-server-info on the server?
```

Oops, so what did that mean? I went to the web page for my project, to the Source tab and browsed the source and 
clicked on the "git" node; the Filename pane now displays "Error retrieving directory contents." which seems odd, after 
all there's nothing there but it should at least be able to display an empty project. So a little search suggested that 
there had been an issue with Google Code and projects that end with a trailing "/" when you cloned them, so I started 
all over but cloned `https://myusername@code.google.com/p/myproject` instead. The result? Absolutely the same.

So, I wondered, Google now recommends that you use the .netrc to store your login details and use a URL that doesn't 
include your email address. So, I added my details into .netrc and tried the whole process again. And this time? 
Absolutely the same result. So as far as I can tell, from my Mac, using Git 1.7.5.4, I cannot clone and push changes 
back to Google code :-(

I went to file a ticket and discovered someone else already had so I added some comments and now I guess I wait. You 
can follow along with the support ticket if you wish. [update - not so much any more Google Code is RIP]
