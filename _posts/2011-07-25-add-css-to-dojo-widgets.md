---
title: Add CSS to Dojo Widgets
layout: postx
category: code
tags: [archive, css, dojo]
---

Many [Dojo](https://dojotoolkit.org/) widgets have custom CSS, and you have to remember to include them either in your application HTML pages or in 
your application CSS file. The problem is that when we add such a widget into our application code we have to figure out 
whether it has it's own CSS we have to weave in. Usually we find this when we load the page and it looks a mess. So, a 
widget declares it's templates, it declares the i18n bundles it uses, why doesn't it also include the CSS as well?

What I want to be able to do is something like the following, add a require call to declare my stylesheet and have it 
dynamically added to the page.

```javascript
dojo.provide('foo.bar.MyWidget');

dojo.require('dijit._Widget');
dojo.require('dijit._Contained');
dojo.require('dojox.dtl._Templated');

dojo.requireLocalization('foo.bar', 'MyWidget');

dojo_requireStylesheet('foo.bar', 'MyWidget', 'myWidgetClass');

dojo.declare('foo.bar.MyWidget', [dijit._Widget, dijit._Contained, dojox.dtl._Templated], {

    ...
```

So, I created the `dojo.requireStylesheet` method shown below. Right now it's pretty simple, it takes a module name and 
the base name for the CSS stylesheet so that the result looks much like `requireLocalization`. The additional parameter 
is a class name from the target CSS that we know exists, this allows the method to avoid attempting to load the 
stylesheet if it has already been loaded (either in code or in the HTML).

```javascript
dojo.loadedStylesheets = {};
dojo.requireStylesheet = function(/*String*/moduleName, /*String*/styleSheetName, /*String*/expectedClass) {
 // summary:
 //  This call will loaded a CSS stylesheet dynamically, by adding a new
 //  link DOM node to the page head. The caller specifies the name of 
 //  the module owning the CSS, and the call assumes that all CSS files
 //  exist within a "resources" sub directory. The stylesheet name is
 //  again the base name only and the call will append ".css". The 
 //  purpose of this is to allow a requireStylesheet call to look and
 //  feel like existing calls such as dojo.requireLocalization. Note
 //  that best effort will be made to ensure that the same stylesheet
 //  is not loaded more than once and the caller can provide a class
 //  name from the target stylesheet that is expected to be loaded, if
 //  this class is found then it is assumed that the stylesheet has
 //  been loaded elsewhere.
 // moduleName:
 //  the base module name owning the stylesheet. All stylesheets live in
 //  a subdirectory named "resources" of the owning module.
 // styleSheetName:
 //  the base name, without the ".css" suffix, of the stylesheet.
 // expectedClass:
 //  a class in the target stylesheet to be used to test whether it 
 //  has already been loaded. Note that if this is undefined the call
 //  will always try and and load the stylesheet.
 //
 function onClassFound(classItem) {
  if (classItem === null || classItem === undefined) {
   var url = dojo.moduleUrl(moduleName + '.resources', styleSheetName + '.css');  
   var link = dojo.create('link', {
     type: 'text/css',
     rel:  'stylesheet',
     href: url},
        dojo.doc.getElementsByTagName('head')[0]);
   dojo_loadedStylesheets[styleSheetName] = true;
  }
 }
 if (moduleName !== undefined && styleSheetName !== undefined) {
  if (dojo_loadedStylesheets[styleSheetName] === undefined) {
   if (expectedClass !== undefined) {
    var classStore = new dojox.data.CssClassStore();
    classStore.fetchItemByIdentity({identifier: expectedClass, onItem: onClassFound});
   } else {
    onClassFound(undefined); /* force loading */
   }
  }
 }
}
```

Now your widgets have the following structure, and while a lot of Dojo widgets and user widgets use the `resources` 
directory the code above makes that a requirement in the same way that `dojo.i18n` requires the `nls` directory.

```
/ foo.bar
  +-- nls
      +-- Mywidget.js
  +-- resources
      +-- Mywidget.css
  +-- templates
      +-- Mywidget.html
  +-- Mywidget.js
```

TODO: one issue with this is that it is not "theme" aware, the issue today is that the notion of theme is encoded in 
the CSS (and image, and other) resource URLs. A common approach to the identification of the current theme and encoding 
the theme in the path would be really valuable - if anyone from Dojo is listening :-)

