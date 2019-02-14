# JerryGen
A WebIDL to Jerryscript-glue-code compiler

### Overview
This compiler takes as input a subset of WebIDL and produces C-code that will "glue" existing C libraries into Jerryscript using that API.  The overarching purpose is to allow existing C code libraries to be used in other languages without having to rewrite those libraries, and Javascript, implemented by Jerryscript, is the first language that we attack.  Details of the high-level design can be found in our [whitepaper](../docs/TLC_scripting_submission_2017.pdf) and [poster](../docs/Scripting_Poster.pdf) presented at the TI Technical Conference in 2017.

### Building the generator
<details>
<summary>Click to show installation instructions for node packages</summary><br>
First, if you clone the repo and cd into that directory, you should be able to run a single command:<br>
<code>npm install</code><br>
...the individual steps are as follows:

<td style="padding-top: 2px;">
#### the WebIDL parser:
<code>sudo npm install -g webidl2</code>
</td>

#### file i/o:<br>
<code>sudo npm install -g q-io
npm install file-exists</code><br>
#### ast compiler:
<code>sudo npm install -g hogan.js</code><br>
(NOTE: "hogan.js", not "hogan"!)<br>
#### boost-y type functions:
<code>sudo npm install lodash</code><br>
#### continuation passing/async calls through promises:
<code>sudo npm install q</code><br>

<code>npm install minimist</code>

...then set NODE_PATH to /usr/local/lib/node_modules (the "-g" on the npm-install command puts them here; you can alternatively install them locally, and then do the obvious... (in (t)csh, the .cshrc command is:
</details>

###
