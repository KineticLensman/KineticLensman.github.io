---
layout: page
title: Art
---

Art content here

First, a fixed size image embedded via Markdown

![Image](/image_test_JPEGs/Image01.1000.jpg)

Now, a responsive image embedded by HTML tags. Different browsers respond differently.  

<img srcset="/image_test_JPEGs/Image01.200.jpg 200w,
             /image_test_JPEGs/Image01.400.jpg 400w,
             /image_test_JPEGs/Image01.800.jpg 800w,
	     /image_test_JPEGs/Image01.1000.jpg 1000w"
     sizes="(max-width: 220px) 200px,
            (max-width: 440px) 400px,
						(max-width: 840px) 800px,
            1000px"
     src="/image_test_JPEGs/Image01.1000.jpg" alt="Image test 3">
