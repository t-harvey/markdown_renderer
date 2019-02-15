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

...then set NODE_PATH to /usr/local/lib/node_modules (the "-g" on the npm-install command puts them here; you can alternatively install them locally, and then do the obvious...
</details>

### Building Jerryscript
<details>
</details>
