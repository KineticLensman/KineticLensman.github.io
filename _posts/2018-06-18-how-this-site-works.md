---
layout: post
title: "How this site works"
date: 2018-06-18
---

## Development goals

I had several goals when creating this site:
* Give myself a simple workflow that lets me focus on creativity, not the tools
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

If you want to try this yourself, there are some [really great tutorials](http://jmcglone.com/guides/github-pages/) that explain the specific steps involved.
## Assessment
Now that I have the site up and running, the tools aren't getting in my way (unlike some - I'm looking at you, [Blender](https://www.blender.org/)) and I can completely focus on the creative process. The individual tools are reasonably well documented and so far I haven't hit any fundamental show-stoppers. 

Just as my site went live, GitHub announced that they were going to be acquired by Microsoft. It's not yet clear whether Microsoft will keep GitHub ad-free and free-to-use, or take actions that degrade the GitHub service. Luckily, at the start of this whole process, I'd paid to register the non-kinetic-effects domain name so that I didn't have to have 'GitHub' in my site's address. As a result, and because I'm using open source tools, I can move the entire site elsewhere if necessary.
