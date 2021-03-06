Ogel
====

Templating system for the rapid prototyping of flat html builds.

This is meant to help facilitate the creation of flat builds. It pieces together fragments of a site build and produces
the final flat html output that you would get sign off on etc.

## Installation

`npm install xp-ogel`

## Usage

```js
var ogel = require('xp-ogel');

var options = {
  src: 'src',
  dest: 'build',
  templateDir: 'src/templates'
};

// preview mode - will trigger the function callback for each file found and parsed
// returns an object with the filename and compiled html
ogel.preview(options, function(args){
   console.log('new file parsed: ' + args.file);
   console.log(args.html);
});

//build mode - returns when completed and generates the new flat build for you based on the 'dest' directory

ogel.build(options);

```

## TODO

In no particular order

1. ~~Fix \r\n vs. \n differences between systems in regex. \s\S produced odd results~~
2. The system currently works inwards. Some efficiencies could be gained by working outwards
3. Tests (started)
4. General regex cleanup. I'm terrible with regex and I'm sure there could be some fixes applied
5. It would be nice to randomize the lorem ipsum that is generated
6. ~~Turn this into a npm module + grunt task (two separate projects where the task references the npm module)~~

## Features

1. [Templates (inline and block)](#templates)
2. [Nested Templates](#nested)
3. [Optional placeholders (yields)](#yields)
4. [Basic model support (think of having a layout where you need to set values based on interior page)](#model)
5. [Lorem ipsum generation](#ipsum)
6. [Placeholder.it shortcut](#placeholder)
7. [Repeatable blocks](#repeat)

<a name="templates"/>
## Templates

Templates are found via `{{name}}`, where the name would match a filename in your templates directory. If you place your template in a sub folder, such as `templates/menus/main-navigation.html`, you would reference it via `{{menus/main-navigation}}`

There are two types of templates. 

#### Inline Templates

This is a straight replacement. 


An example:
```html
<body>
  ...
  {{footer}}
</body>
```
In this scenario, the `{{footer}}` would fetch the contents of the footer.html template and swap the content.


#### Block Templates

This allows for you to nest content within the data of a template. In order to do so, you must include a `{{yield}}` in your template. You must also include an opening and closing tag for the template. 

An example (say index.html referencing the layout template):
```html
{{layout}}
    <article>
      main article content goes here
    </article>
{{/layout}}
```

<a name="nested"/>
## Nested Templates

You can nest templates. So a template can reference another template. It's a recursive system. 

An exmaple:
```html
{{layout}}
  {{aside}}
  <article>
    main article content goes here
  </article>
{{/layout}}
```

You can see above that we've used both the `layout` template as well as the `aside` template. In fact, the layout template could also reference multiple other templates etc.

<a name="yields"/>
## Optional Placeholders (optional yield blocks)

You can include optional placeholders (or yield blocks) in your templates. For instance, maybe your layout has the ability to be either 1 or 2 columns. Your aside (column 1) is optional, so you can declare it like:

```html
<!-- layout.html (a template) -->
<body>
  <div class="row">
    {{yield:aside}} <!-- define an optional yield block -->
    <div class="col-md-10">
      {{yield}} <!-- main content of callee goes here -->
    </div>
  </div>
</body>
```

And in your page that references the layout template
```html
<!-- index.html (a page) -->
{{layout}}
  {{layout:aside}}
    <div class="col-md-2">
      side nav could go here
    </div>
  {{/layout:aside}}
  main content goes here and ends up in the {{yield}} block
{{/layout}}
```

<a name="#model"/>
## Basic Model Support

In the previous example you saw that we had a `col-md-10` in our layout template. The `aside` yield block is also optional, so ideally, you would want the `col-md-10` to change based on whether you used the aside block or not.

One way that you could achieve this is through exposing model properties. 

Some things to keep in mind.

1. Models can support properties
2. Models can support methods
3. Models compound as templates are used. Meaning that they stack properties. If you had 4 templates, all adding different properties to the model, all of those properties would be made available. 

In the situation described above, we could handle it via:

```html
<!-- layout.html (a template)-->
{{model}}
  {
    width: 12
  }
{{/model}}
<body>
  <div class="row">
    {{yield:aside}} <!-- define an optional yield block -->
    <div class="col-md-{{model.width}}"> <!-- here we use the model property instead -->
      {{yield}} <!-- main content of callee goes here -->
    </div>
  </div>
</body>

<!-- index.html (a page using the layout) -->
{{model}}
  {
    width: 10 <!-- here we override the default width and set it to 10, as we're using 2 columns for our aside -->
  }
{{/model}}
{{layout}}
  {{layout:aside}}
    <div class="col-md-2">
      side nav could go here
    </div>
  {{/layout:aside}}
  main content goes here and ends up in the {{yield}} block
{{/layout}}
```

Models can also utilize functions. For instance, in your copyright notice you might want to use the current year. 

```html
{{model}}
  {
    year: function(){ return new Date().getFullYear(); }
  }
{{/model}}

<!-- footer.html (a template) -->
<p class="copyright">&copy; {{model.year}} company inc.</p>

```

Notice that you still reference it using the same syntax. 

Model methods do not currently support arguments. 

<a name="ipsum"/>
## Lorem ipsum generation

This shortcut allows you to quickly inject lorem ipsum placeholder data. It uses <a href="https://github.com/shyiko/lorem" target="_blank" title="shyiko lorem">lorem by shyiko</a> and allows for any of the classnames that he supports. 

A few exmaples

`{{ipsum:w10}}` would generate 10 words of lorem ipsum

`{{ipsum:p4}}` would generate 4 paragraphs of lorem ipsum

`{{ipsum:s8}}` would generate 8 sentences of lorem ipsum


<a name="placeholder"/>
## Placeholder.it shortcut

This shortcut allows you to quickly specify placeholder.it images. There are to styles that you can leverage. 

`{{img:200x200}}` where the 200x200 specifies the dimensions being passed to placeholder.it. Because of this, any of their syntax is valid here as well. 

or

`{{img:200x200:class="pull-left" alt="no image"}}` here you can see that we're also including additional attributes. 

<a name="repeat"/>
## Repeatable blocks

This shortcut allows you to quickly repeat content with a repeatable block. An example could be a list or table of data. This lets you quickly set how many items or rows etc should show up. 

```html
<ul>
{{repeat:10}} <!-- 10 could be any number -->
  <li>some data here</li>
{{/repeat}}
</ul>
```



