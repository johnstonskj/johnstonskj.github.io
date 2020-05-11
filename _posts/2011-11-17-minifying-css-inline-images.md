---
title: Minifying CSS - inline images
layout: postx
category: code
tags: [archive, css, dojo, javascript]
---

Having worked on a pretty complex web application, a lot of JavaScript with [Dojo](http://dojotoolkit.org/), we had to tackle the common problem of browser load performance. Dojo provides some pretty good build tools to minify the JavaScript used by your application, and does compress all the CSS in your application directories as well. The issue is that our CSS references a lot of images, either background, buttons, or other images used in widgets.

So, one possibility is to actually inline the images themselves into your CSS, and with compression and concatenation of the CSS you get to one nice big resource rather than a whole bunch of related and smaller resources.

The trick is to identify all the CSS rules that have values of the form `url("...")` or `url('...')`, which are relative file locations. Now, load that resource, convert it into a base-64 encoded value and write this back out as a value of the form:

```css
url('data:image/png;base64,...encoded-value...')
```

We use the simple Python script below to process a single CSS file, then a shell script that finds and applies this to all our CSS files before we run the standard Dojo build process.

```python
#!/usr/bin/env python

import base64, logging, os, os.path, re, sys

log = None

RE_URL = re.compile("url\([\"'](?P[^\"']+)[\"']")

def init():
    global log
    logging.basicConfig(level=logging.DEBUG)
    log = logging.getLogger('cssmin')

def parse_command_line():
    " returns (options, args) "
    global log
    if len(sys.argv) == 0:
        print "Must specify a CSS file"
        sys.exit(1)
    return ({}, sys.argv[1:])

def encode(file):
    input = open(file)
    encoded = base64.b64encode(input.read())
    input.close()
    return encoded

def process_css(css_file):
    original_file = "%s.original" % css_file
    os.rename(css_file, original_file )
    log.info("Reading file %s." % original_file)
    input = open(original_file, "rt")
    log.info("Writing file %s." % css_file)
    output = open(css_file, "wt")
    for raw_line in input:
        match = RE_URL.search(raw_line)
        if match:
            url = match.group("url")
            src = os.path.join(os.path.split(original_file)[0], url)
            type = os.path.splitext(src)[1]
            if type[1:].lower() in ["gif", "png", "jpeg", "jpg"]:
                data = encode(src)
                output.write(raw_line[:match.start("url")])
                output.write(
                    "data:image/%s;base64,%s" %
                    (type,data)
                    )
                output.write(raw_line[match.end("url"):])
                log.info("inlined resource %s." % src)
            else:
                log.info("ignoring url for type %s." % type)
                output.write(raw_line)
        else:
            output.write(raw_line)

if __name__ == '__main__':
    init()
    (options, args) = parse_command_line()
    process_css(args[0])
```