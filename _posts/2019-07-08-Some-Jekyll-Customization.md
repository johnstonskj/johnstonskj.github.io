---
title: Some Jekyll Customization
layout: post
categories: [code]
tags: [jekyll, blog]
---

As you can probably tell, this site was created using [GitHub Pages](https://pages.github.com/)
along with the [Jekyll](https://jekyllrb.com/) static site generation tool and the 
[minima]https://github.com/jekyll/minima) theme. This suits me perfectly, a nice git-based
workflow, easy to sync and author across machines (and even iPad), and a nice clean 
template with few visual distractions.

However, after a while, perhaps I might want some, _just some_ visual distractions? For
example, why add a `tags` field to the front matter if you can't see them in the page?
So, I started adding a custom layout, and then another, and, well just two for now.

## Adding tags to posts

I just want a list at the bottom of each post to show the tags I added in the front matter.
This turned out to be remarkably easy, specifically adding a new layout `postx` that
in turn is based on the standard `post`. After displaying the content, I just iterate
through the tags and output each in it's own span. 

```Liquid
---
layout: post
---

<div>
  {{ content }}
</div>
<div class="post-tags">
  {% for tag in page.tags %}
    <span class="tag">{{ tag }}</span>
  {% endfor %}
</div>
```

For my preference I added some CSS, to reduce the font size to `0.75em`, the text color
to a dark gray, and to add the hashtag character as `::before`  content.

## Adding Related Posts

This turned out a little more complex, basically I wanted to the top-level files in
the site to be anchors for categories, so at least _work_, _fun (code)_, _audio_, 
and _diving_ categories. Then, I want to list at the end of each of these any related
posts with the same category. 

First, I created a new layout `catpage` for category page, and for each top-level 
page not only add this as the layout but also add a `category` front matter variable
to name the category it represents.

Next the layout adds a new `div` after the content, only if there are related posts,
with a heading and list of post titles. By using the `page.category` variable I was
able to only show the block if there are related posts, which turned out pretty 
easy in the end.

```Liquid
---
layout: default
---

<div>
  {{ content }}
</div>
{% for category in site.categories %}
  {% if category[0] == page.category %}
    <div class="related-posts">
      <h2>Related Posts</h2>
      <ul>
        {% for post in category[1] %}
          <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
      </ul>
    </div>
  {% endif %}
{% endfor %}
```
