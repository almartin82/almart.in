Title: Pelican and R markdown: an update
Date: 2015-04-10
Category: meta
Slug: pelican-with-rmd
Author: Andrew Martin

I leaned heavily on a great post from [Rebecca Weiss](https://rjweiss.github.io/) titled [*"How easy is it to use R markdown and knitr with pelican?"*](https://rjweiss.github.io/articles/2014_08_25/testing-rmarkdown-integration/) to get Pelican/knitr integration up and running.  Rebecca wrote her post in 2014 in a Mac/Unix environment, so I thought that I would contribute an update, and some errata for Windows users.

If you're landing here from Google, a quick overview & definition of some terms:

* [Pelican](http://docs.getpelican.com/en/3.5.0/) is a static site generator written in Python.  If you've ever seen the [Github pages](http://jekyllrb.com/docs/github-pages/) tutorials that use Jekyll, Pelican is a great alternative if you want to stay in the Python universe.

* [knitr](http://yihui.name/knitr/demo/minimal/) is a report generation package for R (especially [R Studio](http://www.rstudio.com/)) that makes it easy to mix words, simple formatting and R code.

It would be great to be able to write up an .Rmd, run `pelican content`, and have everything just show up on the web, with syntax highlighting, inline images, etc.  That's what Pelican + knitr lets us do!<!-- PELICAN_END_SUMMARY -->

# behind the scenes

A few quick words about what is happening - the `rmd_reader` plugin depends on the python [`rpy2`](http://rpy.sourceforge.net/rpy2/doc-2.1/html/) package, which exposes an interface from Python into R.  `rpy2` turns the .Rmd into an [.aux file](http://tex.stackexchange.com/questions/47943/how-to-use-the-aux-file) _(things get a little hand-wavy for me here - I am team html/markdown/mathjax)_ and then calls `knit()` in R to generate a markdown file.

# general thoughts

1) Pelican does not like the way RStudio sets up the document [skeleton](http://rmarkdown.rstudio.com/developer_document_templates.html).  Specifically, the `---` characters used to gate the yaml-like metadata from the content appear to be a no-no.  As far as I can tell, this isn't in the docs for rmd_reader - you just kind of figure it out by trial and error.  You'll know that this is your issue if you are seeing errors like `ERROR: Skipping .\sample-rmd.Rmd: could not find information about 'title'`.  Look at my example in #3 above, or [Wilson's](https://raw.githubusercontent.com/wilsonfreitas/blog.aboutwilson/master/content/bmf-curve-interpolator.Rmd) example here. 

2) It took me a second to figure out [`pelican.publish=TRUE`](https://github.com/getpelican/pelican-plugins/tree/master/rmd_reader#plotting).  If you're writing in RStudio and want to see local output, set that to FALSE.  But if you are ready to run pelican content and publish to the web, that should be TRUE.

3) Rebecca has some great discussion about figure paths, and how to cajole knitr into putting image assets into the right place.  I may go down that road eventually - folder separation between categories might be nice.  For now, though, I am going with the [One Big Folder](https://github.com/almartin82/almart.in-source/tree/master/content) strategy, and am throwing a numeric prefix onto the text files so that posts will sort sequentially.  That didn't require any tinkering with `fig.path` -- so if you are trying to get up and running, my advice would be to start by writing an .Rmd in `content/` with no special `ARTICLE_URL` or `ARTICLE_SAVE_AS` paths, get your publishing workflow nailed down, and then start to re-organize your content - otherwise it could be tricky to identify what's a problem with rpy2/knitr dispatch, and what's simple an issue with filenames/`fig.path`.

4) Not strictly .Rmd related, but a note on feed generation.  It's not always immediately clear which `pelicanconf.py` parameters are True/False, and which ones need a text value.  If you set `FEED_ALL_ATOM=True`, Pelican will throw a `CRITICAL: 'bool' object has no attribute 'lstrip'` error.

# this was kind of a pain on Windows

Yeah.  You'll never believe it.  Some things that came up:

1) Make sure you have a [`R_HOME`](http://stackoverflow.com/questions/17573988/r-home-error-with-rpy2) environment variable set up, or else your `rpy2` install might be pointing at the oldest version of R on your system.  Symptoms that this is happening: you'll get knitr errors when you try to process the .Rmd, because knitr might not be installed.

2) There was a slight problem with the way `rmd_reader` was reading pathnames - `c:\\Users...` was getting interpreted as a Unicode character.  I pushed a [patch](https://github.com/getpelican/pelican-plugins/pull/464) the plugin library that should address this, if the pull request is accepted.

3) Graphics drivers were a pain point.  Drivers that were working in RStudio (png, svg) were not working when called via the rpy2 interface.  Not totally sure what the story is here, but I ended up throwing `opts_chunk$set(dev='Cairo_svg')` into my rmd_reader block -- take a look at the [example](https://raw.githubusercontent.com/almartin82/almart.in-source/849d26d53631523b10c18bb77c08135ed0d7d6b1/content/sample-rmd.Rmd) if you want to see it in action.

Overall, though, I have to say that I'm thrilled about having a straightforward .Rmd publishing tool.  [rpubs](https://rpubs.com/) is great, but having the ability to publish .Rmds to my own site is exciting.

# enhancement ideas

One thing that would be helpful would be a pelican_rmd template, following the [tutorials]((http://rmarkdown.rstudio.com/developer_document_templates.html)) for creating custom knitr templates on RStudio.  That would fix the metadata problem, and it could have a nice block with all the `hook_content` stuff all ready to go.  If I find some time while this is still fresh, I'd like to add some of this back to Wilson's readme for rmd_reader.  **Shiny** support is also something to look into.











