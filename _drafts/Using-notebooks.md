---
layout: post
title: "Killer presentations with jupyter notebooks"
modified:
categories: blog
excerpt: "Creating presentations for c that can compile"
tags: [jupyter]
comments: true
share: true
image:
  feature: cow.jpg
date: 2017-03-10T22:33:01-04:00
---

Once in a while you will have to give a presentation about code.  My normal
flow used to be copy paste source code into
[notepad++](https://notepad-plus-plus.org/) and copy-paste it with RTF syntax
highlighting into powerpoint (_Notepad++ > Plugins > NppExport > Copy RTF to
clipboard_).  This used to be ok, but typically smalls edits done in the
presentation file would result in small typos/bugs creeping leading
to remarks during your presentation such as: _"But that won't compile"_. This
will distract yourself and your audience from your main message.

* Wouldn't it be wonderfull if all the snippets in our presentations are actually compiled?
* All in a sleek [reveal.js](http://lab.hakim.se/reveal-js/#/) presentation?
* Stored in a plain text format that you can store in version control?

Using Jupyter notebooks in combination with 2 powerfull extensions enables just that.

# Jupyter notebooks
If you haven't heard of [Jupyter notebooks](https://jupyter.org/), definitly
try it out (for python/R/Haskell/Julia) at
[try.jupyter.org/](https://try.jupyter.org/). Jupyter notebooks make it
possible to mix markdown and code allowing for executable documentation.
These can be exported to PDF, Presentations, Markdown, LaTeX, HTML etc.

## Installing jupyter
I prefer to keep my development away from my windows based corporate laptop, so
I have a virtual Ubuntu machine running. Jupyter is just a pip package that can
be installed, so lets first create a python virtualenv to keep our system
install clean.

```shell
mkdir my_jupyter_notebooks
cd my_jupyter_notebooks
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install jupyter
```

Now jupyter will be installed in your virtualenv and you can run the notebook
server locally (by using `--no-browser` flag, no browser will be opened automattically).

```shell
jupyter notebook --no-browser --ip=0.0.0.0 --port=8080
```

A link will be shown in the terminal with a secret token. If you copy paste
this link into your windows environment browser you should be able to show your
notebooks.  Make sure you have forwarded the 8080 port to 8080 on your host
machine.  You can create a new notebook (for now only for Python).

Once you are done playing, you can stop your jupyter notebook server by
pressing `ctrl`-`c` twice in the terminal it is running.

# Jupyter C kernel

Jupyter notebooks support a lot of languages (see the [complete list of
kernels](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels).

For this post I want to focus on `c` since that is the main language I use.
I've used the [jupyter-c-kernel](https://github.com/brendan-rius/jupyter-c-kernel) succesfully.

This kernel allows you to compile c using gcc directly from the Jupyter notebook.

## Setup
To install it follow the instructions there, make sure you have all the dependencies installed!
I followed the manual instructions from my `virtualenv`:

```shell
pip install jupyter-c-kernel
git clone https://github.com/brendan-rius/jupyter-c-kernel.git
cd jupyter-c-kernel
jupyter-kernelspec install c_spec/
```

After everything is installed you can once again start jupyter notebook and create a c-notebook:

```shell
jupyter notebook --no-browser --ip=0.0.0.0 --port=8080
```

> Insert example of C-notebook

Stop your jupyter notebook server again by pressing `ctrl`-`c` twice in the terminal it is running.

# RISE
But I promised you a sleak Reveal.js presentation. Another great plugin for jupyter notebook is RISE.

## Setup
This is again just a pip package to install in your venv:

```shell
pip install RISE
jupyter-nbextension install rise --py --sys-prefix
jupyter-nbextension enable rise --py --sys-prefix
```

Once installed, you can start the notebook server again:

```shell
jupyter notebook --no-browser --ip=0.0.0.0 --port=8080
```

This time a new button was added to the interface with which you can open your presentation.

## Using cell toolbar
A notebook consists of cells which can be code or markdown. To control how
they are shown in your presentation you can use the cell toolbar.
Open it from the view menu as shown below:

> Insert image

# Final slide show





<style>
.ipython-container {
    position: relative;
    padding-bottom: 56.25%;
    padding-top: 35px;
    height: 0;
    overflow: hidden;
}

.ipython iframe {
    position: absolute;
    top:0;
    left: 0;
    width: 100%;
    height: 100%;
}
</style>

<div class="ipython-container">
    <iframe src="https://nbviewer.ipython.org/gist/mvdwoord/5a5ea699a48439a4f26f" frameborder="0" height="500" width="100%"> </iframe>
</div>
