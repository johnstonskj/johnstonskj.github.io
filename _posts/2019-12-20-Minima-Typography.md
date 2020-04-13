---
title: Minima & Typography
layout: postx
category: code
tags: [jekyll, blog, typography]
---

I have been using Jekyll's [Minima](https://github.com/jekyll/minima) theme exclusively because I like the general 
layout and clean lines. However, the default font choice just didn't feel satisfying and so I embarked on an attempt
to alter the theme in as few ways as possible, without forking, to get a feel I felt I liked more. Now, I know there 
are rules and definitely many opinions on web typography but I decided to start from something I knew I had seen and
liked. [Jordan Santell](https://jsantell.com/) has a great site with some interesting articles and while the color 
elements don't do much for me the typography is fantastic. So, using the Chrome dev tools I inspected the properties
of the text and found that the majority of all text uses the [Fira Sans](https://en.wikipedia.org/wiki/Fira_Sans) 
typeface originally developed at Mozilla. So, that seemed like a good start.

## Font Selection

A quick trip to Google fonts, I found the Fira family, and while there is a Fira Mono I chose Fira Code as my only
real use of a monospaced font will be for code examples. These are the final weights I chose to use.

* [Fira Sans](https://fonts.google.com/specimen/Fira+Sans) 100 (thin), 200 (extra-light), and 400 (regular) 
* [Fira Code](https://fonts.google.com/specimen/Fira+Code) 300 (light), and 600 (semi-bold).

Looking at the theme structure I copied over `_includes/head.html` and added the following line to pull the fonts
from the Google CDN.

```html
<link href="https://fonts.googleapis.com/css?family=Fira+Code:300,600|Fira+Sans:100,100i,200,200i,400,400i&display=swap" 
      rel="stylesheet">
```

## CSS Modifications

To modify the actual CSS you have to copy the initial `_sass/minima.scss` file and make modifications. Wherever 
possible adjusting the variables at the beginning is best, but you can always override styles after the `@import`
that pulls in the main bulk of the declarations.

To change the main text I updated the following block to select the font, weight, and line height I had chosen.
```scss
$base-font-family: "Fira Sans", sans-serif !default;
$base-font-size:   16px !default;
$base-font-weight: 200 !default;
$small-font-size:  $base-font-size * 0.8 !default;
$base-line-height: 1.5 !default;
```

I also adjusted some of the text colors, just to tone things down a little.

```scss
$text-color:       #3c484e !default;
$background-color: #fdfdfd !default;

$grey-color:       #828282 !default;
```

Finally, I added variables for the code blocks, this would be great to use in the highlighter in the future, but
at least I can use it below.

```scss
$code-font-family: "Fira Code", monospace !default;
$code-font-weight: 300;
$code-font-size:   90%;
$code-line-height: 1.4;
```

After all of the theme CSS has been defined I override as little as possible at the end.

The following change some details of text blocks, justifying the main body text and setting sixes, weights
and colors for secondary elements. Note the use of a 2× multiplier for weight so select the semi bold form.

```scss
main p {
  text-align: justify;
}

strong {
  font-weight: 200%;
}

figure {
  font-size: $small-font-size;
}

a, a:visited {
  color: #0037B3;
}

.meta {
  color: $grey-color-light;
  font-size: $small-font-size;
}
```

The following makes some minor adjustments to headings. Note, that while headings are in larger sizes the weight is 
reduced which keeps a nicer sense of proportion between body and heading.

```scss
h3 > .post-link {
  margin: 0px;
}

main h1, main h2, main h3, main h4 {
  font-weight: 50%;
}
```

Finally, here are the changes I made for code blocks, again using a 2× multiplier for bold elements.

```scss
pre, code {
  font-family: $code-font-family;
  font-size: $code-font-size;
  font-weight: $code-font-weight;
  line-height: $code-line-height;
}

.highlight .o, .highlight .k {
  font-weight: 200%;
}
```

So, if you are reading this page, hopefully it was at least a typographical pleasure if nothing else.