[![view on npm](http://img.shields.io/npm/v/dmd.svg)](https://www.npmjs.org/package/dmd)
[![npm module downloads per month](http://img.shields.io/npm/dm/dmd.svg)](https://www.npmjs.org/package/dmd)
[![Build Status](https://travis-ci.org/75lb/dmd.svg?branch=next)](https://travis-ci.org/75lb/dmd)
[![Dependency Status](https://david-dm.org/jsdoc2md/dmd.svg)](https://david-dm.org/75lb/dmd)

# dmd
dmd (document with markdown) is a module containing [handlebars](http://handlebarsjs.com) partials and helpers intended to transform [jsdoc-parse](https://github.com/jsdoc2md/jsdoc-parse) output into markdown API documentation. It exposes <code>[dmd](#module_dmd)</code>, a function which requires data and a template. See [jsdoc-to-markdown](https://github.com/jsdoc2md/jsdoc-to-markdown) for example output.

## Synopsis
With this input file containing [jsdoc-parse](http://handlebarsjs.com) output:
```json
[
    {
        "id": "fatUse",
        "name": "fatUse",
        "kind": "member",
        "description": "I am a global variable",
        "scope": "global"
    }
]
```

this command:
```
$ cat examples/input/doclet.json | dmd
```

produces this markdown output: 
```
<a name="fatUse"></a>
## fatUse
I am a global variable

**Kind**: global variable
```

## Install and use
### As a library
Install:
```sh
$ npm install dmd --save
```

Example:
```js
var dmd = require("dmd");

var options = {
   template: "my-template.hbs"
};
process.stdin.pipe(dmd(options)).pipe(process.stdout);
```

### At the command line
Install the `dmd` tool globally: 
```sh
$ npm install -g dmd
```
Example:
```sh
$ cat examples/doclet.json | dmd
$ dmd --help
```

## Templates
The default template contains a single call to the  [main](https://github.com/jsdoc2md/dmd/blob/master/partials/main.hbs) partial:
```hbs
{{>main}}
```

This partial outputs all documentation and an index (if there are enough items). You can customise the output by supplying your own template. For example, you could write a template like this:
```hbs
# A Module
This is the readme for a module. 

## Install
Install it using the power of thought. While body-popping.

# API Documentation
{{>main}}
```

and employ it like this: 
```
$ cat your-docs.json | dmd --template readme-template.hbs
```

## Customising
You can customise the generated documentation to taste by overriding or adding partials and/or helpers.

For example, let's say you wanted this datestamp at the bottom of your generated docs:

```
**documentation generated on Sun, 01 Mar 2015 09:30:17 GMT**
```

You need to do two things:

1. Write a helper method to return the date in your preferred format
2. Override the appropriate partial, inserting a mustache tag (e.g. ``) where you would like it to appear. We'll override the [main](https://github.com/jsdoc2md/dmd/blob/master/partials/main.hbs) partial.

### Write a new helper
A helper file is just a plain commonJS module. Each method exposed on the module will be available as a helper in your templates. So, our new helper module:

```js
exports.generatedDate = function(){
    return new Date().toUTCString();
}
```

[Read more about helpers in the handlebars documentation](http://handlebarsjs.com).

### Write a new [main](https://github.com/jsdoc2md/dmd/blob/master/partials/main.hbs) partial
Create a duplicate of the [main](https://github.com/jsdoc2md/dmd/blob/master/partials/main.hbs) partial (typically in the project you are documenting) containing your new footer:

```hbs
{{>main-index~}}
{{>all-docs~}}

**documentation generated on {{generatedDate}}**
```

*the file basename of a partial is significant - if you wish to override `main` (invoked by `{{>main}}`) then the filename of your partial must be `main.hbs`.*

### Employ
To use the overrides, pass their file names as options to dmd (or [jsdoc-to-markdown](https://github.com/jsdoc2md/jsdoc-to-markdown) if you're using that):
```
$ cat your-parsed-docs.json | dmd --partial custom/main.hbs --helper custom/generatedDate.js
```

If you have multiple overrides, the syntax is 
```
$ cat your-parsed-docs.json | dmd --partial override1.hbs override2.hbs
```

Globbing also works:
```
$ cat your-parsed-docs.json | dmd --partial overrides/*.hbs
```

### Create a plugin
If you wish to version-control and/or share your customisations you can create a plugin for distribution via npm. See [dmd-plugin-example](https://github.com/jsdoc2md/dmd-plugin-example) as an example and boilerplate to get you started.

Once you have your plugin, install it where required as a dev-dependency. Then supply the plugin package name(s) to the `--plugin` option, for example: 
```
$ cd my-project
$ npm install dmd-plugin-example --save-dev
$ jsdoc2md lib/my-module.js --plugin dmd-plugin-example
```
    
# API Reference
<a name="exp_module_dmd--dmd"></a>
### dmd([options]) ⇒ <code>[Transform](http://nodejs.org/api/stream.html#stream_class_stream_transform)</code> ⏏
Transforms doclet data into markdown documentation. Returns a transform stream - pipe doclet data in to receive rendered markdown out.

**Kind**: Exported function  
**Params**

- [options] <code>module:dmd~dmdOptions</code> - The render options  

<a name="module_dmd--dmd..DmdOptions"></a>
#### dmd~DmdOptions
All dmd options including defaults

**Kind**: inner class of <code>[dmd](#exp_module_dmd--dmd)</code>  

* [~DmdOptions](#module_dmd--dmd..DmdOptions)
  * [.template](#module_dmd--dmd..DmdOptions#template) : <code>string</code>
  * [.heading-depth](#module_dmd--dmd..DmdOptions#heading-depth) : <code>number</code>

<a name="module_dmd--dmd..DmdOptions#template"></a>
##### dmdOptions.template : <code>string</code>
The template the supplied documentation will be rendered into. Use the default or supply your own template for full control over the output.

**Kind**: instance property of <code>[DmdOptions](#module_dmd--dmd..DmdOptions)</code>  
**Default**: <code>&quot;{{&gt;main}}&quot;</code>  
**Example**  
```js
var fs = require("fs");
var dmd = require("../");

var template = "The description from my class: {{#class name='MyClass'}}{{description}}{{/class}}";

fs.createReadStream(__dirname + "/my-class.json")
    .pipe(dmd({ template: template }))
    .pipe(process.stdout);
```
outputs: 
```
The description from my class: MyClass is full of wonder
```
the equivation operation using the command-line tool: 
```
$ dmd --template template.hbs --src template.js
```
<a name="module_dmd--dmd..DmdOptions#heading-depth"></a>
##### dmdOptions.heading-depth : <code>number</code>
The initial heading depth.

**Kind**: instance property of <code>[DmdOptions](#module_dmd--dmd..DmdOptions)</code>  
**Default**: <code>2</code>  

* * *

&copy; 2015 Lloyd Brookes \<75pound@gmail.com\>. Documented by [jsdoc-to-markdown](https://github.com/jsdoc2md/jsdoc-to-markdown).
