# JerryGen
A WebIDL to Jerryscript-glue-code compiler

### Overview
This compiler takes as input a subset of WebIDL and produces C-code that will "glue" existing C libraries into Jerryscript using that API.  The overarching purpose is to allow existing C code libraries to be used in other languages without having to rewrite those libraries, and Javascript, implemented by Jerryscript, is the first language that we attack.  Details of the high-level design can be found in our [whitepaper](../docs/TLC_scripting_submission_2017.pdf) and [poster](../docs/Scripting_Poster.pdf) presented at the TI Technical Conference in 2017.

We currently support only a subset of the large [WebIDL standard](https://github.com/w3c/webidl2.js).
Our design goal was to support the functionality that makes sense for
programming embedded architectures, and we used the [zephyr.js project](https://github.com/intel/zephyr.js)
as our guide.

<details>
<summary>Click here to see the list of WebIDL constructs
supported.</summary>
<dl style="list-style-type:none;">
<dt> Enumeration types </dt>
<dd> - these are strings in WebIDL and Javascript, but we treat them as
proper enum types in C.
<dt> Callbacks </dt>
<dd> - these are function pointers in all three languages.
<dt> Dictionaries </dt>
<dd> - these are data structures in all three languages.
<dt> Interfaces </dt>
<dd> - these are objects, containing both methods and attributes, and
as such, are stored in the Javascript environment and only accessed by
getters/setters on the C side.
</dl>
</details>

### Building the generator

JerryGen is controlled by the generate.js script, found in the
generator repo.

<details>
<summary>Click to show installation instructions for the
generator.</summary>
Clone the generator repo:
<code>git clone https://github.com/t-harvey/JerryGen.git</code>

The generator is built on top of Javascript, so no compilation of the
tool is necessary.
</details>

<details>
<summary>Click to show installation instructions for node packages.</summary><br>
First, if you clone the repo and cd into that directory, you should be
able to run a single command:<p>

<code>npm install</code><p>

...the individual steps are as follows:

#### the WebIDL parser:
<code>sudo npm install -g webidl2</code>

#### file i/o:<br>
<code>sudo npm install -g q-io<br>
npm install file-exists</code><br>
#### ast compiler:
<code>sudo npm install -g hogan.js</code><br><br>
(NOTE: "hogan.js", not "hogan"!)<br>
#### boost-y type functions:
<code>sudo npm install lodash</code><br>
#### continuation passing/async calls through promises:
<code>sudo npm install q</code><br>

<code>npm install minimist</code>

...then set NODE_PATH to /usr/local/lib/node_modules (the "-g" on the
npm-install command puts them here; you can alternatively install them
locally, and then do the obvious...
</details>


### Building Jerryscript
<details>
The instructions for building Jerryscript are [here](https://github.com/pando-project/jerryscript/blob/master/docs/01.GETTING-STARTED.md) -- note that building Jerryscript without
ES2015 features can give results that are difficult to pin down.
For example, if the config.h file in the jerry-core directory does not
have the variable CONFIG_DISABLE_ES2015_TYPEDARRAY_BUILTIN commented
out, then any attempt to use the ArrayBuffer in a script will result
in a "script error" message from the interpreter, even though the
script containing the ArrayBuffer declaration may be otherwise error
free.  Of course, if a user's scripts don't use ArrayBuffer, then it might
behoove him to compile without that feature and thus minimize the size
of the interpreter.

Using tools/build.py will produce libraries in the build/lib directory.
</details>
