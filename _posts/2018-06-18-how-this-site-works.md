---
layout: post
title: "How this site works"
date: 2018-06-18
---

## Development goals

I had several goals when creating this site:
* Give myself a simple workflow that lets me focus on creativity, not the tools
* Create clean, uncluttered pages that load quickly
* Be independent from the dominant web giants (Amazon, Apple, Facebook or Google) and blogging platforms
* Avoid services or technologies that track users or harvest their personal data
* Avoid services that rely on ads and other sponsored content
* Avoid lock-in to specific vendors by using free open source tools 

In contrast, hosting blog comments was not a key goal - partly because of my 'independent and non-tracking' approach, but also because I don't want to deal with spam or worry about content moderation. I'll provide contact mechanisms if necessary in due course.

These goals fundamentally drove my choice of hosting platform and development tools.
## Hosting
After some initial research, I selected [GitHub Pages](https://pages.github.com/) as my hosting environment. GitHub is a web-based content hosting service, and is itself built on the open-source [Git](https://git-scm.com/) version control system. GitHub Pages is a service offered by GitHub for hosting static websites - sites whose content is prepared in advance and which does not use information harvested from readers when they visit the site.
## Content creation

The content creation workflow I've adopted is a standard one for GitHub Pages: 

* I write the pages on my development machine using the [Atom](https://atom.io/) text editor. Compared with something like Microsoft Word, Atom is clean and minimalistic. It has one job to do (edit text) and it does it well
* I insert simple [Markdown](https://daringfireball.net/projects/markdown/) formatting instructions to get headings, bullet lists, etc. Atom gives a live preview of the Markdown so I can check that the formatting looks okay
* When I'm happy with the text, I upload it to my GitHub repository using the GitHub web interface
* GitHub then automatically processes the raw text files using [Jekyll](https://jekyllrb.com/) to create the static web pages you see in your browser

If you want to try this yourself, there are some [really great tutorials](http://jmcglone.com/guides/github-pages/) that explain the  steps needed to create a GitHub Pages site. The set-up process basically involves creating some folders and files that provide the framework Jekyll needs to automatically convert text files in a GitHub repository into web-pages. The tutorials typically provide full examples, although it helps to have beginner-level awareness of [html](https://www.w3schools.com/Html/) and [css](https://www.w3schools.com/css/default.asp) if you want to change fonts and colours away from the defaults. By default, a GitHub Pages website will have github in its web address. However, it's relatively easy to create your own custom domain (in this case, www.non-kinetic-effects.co.uk) and make it point at the GitHub site (although this will typically incur a small registration fee with a third party registration service).
    
## Assessment
Now that I have the site up and running, the tools aren't getting in my way (unlike some - I'm looking at you, [Blender](https://www.blender.org/)) and I can completely focus on the creative process. The individual tools are reasonably well documented and so far I haven't hit any fundamental show-stoppers. The only part that required a little trial and error was the process of pointing my custom domain at GitHub (by editing the CNAME record from my domain registration account), and this was a one-off thing.

Just as my site went live, GitHub announced that they were going to be acquired by Microsoft. It's not yet clear whether Microsoft will keep GitHub ad-free and free-to-use, or take actions that degrade the GitHub service. However, because I have a non-GitHub URL and am using open source tools, I can move the entire site elsewhere if necessary, and readers shouldn't notice any difference.
