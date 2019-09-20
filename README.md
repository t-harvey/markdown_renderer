<!DOCTYPE html>
<html>
    <head>
        <title>Scripting : Using JerryGen, the WebIDL Compiler</title>
        <link rel="stylesheet" href="styles/site.css" type="text/css" />
        <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>

    <body class="theme-default aui-theme-default">
        <div id="page">
            <div id="main" class="aui-page-panel">
                <div id="main-header">
                    <div id="breadcrumb-section">
                        <ol id="breadcrumbs">
                            <li class="first">
                                <span><a href="index.html">Scripting</a></span>
                            </li>
                                                    <li>
                                <span><a href="Scripting-Home_76288708.html">Scripting Home</a></span>
                            </li>
                                                </ol>
                    </div>
                    <h1 id="title-heading" class="pagetitle">
                                                <span id="title-text">
                            Scripting : Using JerryGen, the WebIDL Compiler
                        </span>
                    </h1>
                </div>

                <div id="content" class="view">
                    <div class="page-metadata">
                        
        
    
        
    
        
        
            Created by <span class='author'> Timothy Harvey</span>, last modified on Sep 20, 2019
                        </div>
                    <div id="main-content" class="wiki-content group">
                    <p> The JerryGen compiler takes WebIDL as input and produces C code.  The compiler has two goals: to write &quot;glue code&quot; that utilizes the Jerryscript C API and to provide &quot;stubs&quot; of C code that can be inserted into the glue code and so provide Javascript programmers using the Jerryscript interpreter with functionality from existing C code.</p><p>This document will explain some of the fundamentals of using the compiler, including building C code from a WebIDL file, augmenting the output of the compiler with human-generated C code, and building a version of the Jerryscript interpreter that includes the C code.</p><h2 id="UsingJerryGen,theWebIDLCompiler-WebIDL">WebIDL</h2><p>WebIDL is a specification language – WebIDL files describe APIs, but they do not implement code.</p><p>A WebIDL file is a simple text file with any number of WebIDL &quot;constructs&quot; – i.e., types – that define an API.</p><p>Both the syntax and type system of WebIDL look like C, although the built-in types largely have different names.  Table 1 shows the mapping of WebIDL types to C types.</p><div class="table-wrap"><table class="wrapped confluenceTable"><colgroup><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/><col/></colgroup><tbody><tr><td style="text-align: center;" class="confluenceTd"><p>WebIDL Type</p></td><td style="text-align: center;" class="confluenceTd">byte</td><td class="confluenceTd">octet</td><td class="confluenceTd">short</td><td class="confluenceTd">unsigned short</td><td class="confluenceTd">long</td><td class="confluenceTd">unsigned long</td><td class="confluenceTd">long long</td><td class="confluenceTd">unsigned long long</td><td class="confluenceTd">float</td><td class="confluenceTd">double</td><td class="confluenceTd">boolean</td><td class="confluenceTd">string</td></tr><tr><td style="text-align: center;" class="confluenceTd">C Type</td><td class="confluenceTd">int8_t</td><td class="confluenceTd">uint8_t</td><td class="confluenceTd">int16_t</td><td class="confluenceTd">uint16_t</td><td class="confluenceTd">int32_t</td><td class="confluenceTd">uint32_t</td><td class="confluenceTd">int64_t</td><td class="confluenceTd"><span>uint64_t</span></td><td class="confluenceTd">float</td><td class="confluenceTd">double</td><td class="confluenceTd">bool</td><td class="confluenceTd">char *</td></tr></tbody></table></div><div class="table-wrap"><table class="wrapped confluenceTable"><colgroup><col/><col/></colgroup><tbody><tr><th class="confluenceTh">WebIDL type</th><th class="confluenceTh">C Type</th></tr><tr><td class="confluenceTd">byte</td><td class="confluenceTd">int8_t</td></tr><tr><td class="confluenceTd">octet</td><td class="confluenceTd">uint8_t</td></tr><tr><td class="confluenceTd">short</td><td class="confluenceTd">int16_t</td></tr><tr><td class="confluenceTd">unsigned short</td><td class="confluenceTd"><p>uint16_t</p></td></tr><tr><td class="confluenceTd">long</td><td class="confluenceTd">int32_t</td></tr><tr><td class="confluenceTd">unsigned long</td><td class="confluenceTd">uint32_t</td></tr><tr><td class="confluenceTd">long long</td><td class="confluenceTd">int64_t</td></tr><tr><td class="confluenceTd">unsigned long long</td><td class="confluenceTd">uint64_t</td></tr><tr><td class="confluenceTd">float</td><td class="confluenceTd">float</td></tr><tr><td class="confluenceTd">double</td><td class="confluenceTd">double</td></tr><tr><td class="confluenceTd">bool</td><td class="confluenceTd">boolean</td></tr><tr><td class="confluenceTd">string</td><td class="confluenceTd">char *</td></tr></tbody></table></div><p><br/></p><p>The compiler supports only a subset of WebIDL – the four constructs that we support are: enumeration types, callbacks, dictionaries, and interfaces.</p><h4 id="UsingJerryGen,theWebIDLCompiler-Enumerationtypes">Enumeration types</h4><p>Enumeration types in WebIDL are just sets of strings.  The syntax for enumeration types is as follows:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum identifier { &quot;enum&quot;, &quot;values&quot; /* , ... */ };</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-Callbacks">Callbacks</h4><p>Callbacks can be thought of as function-type specifiers; the syntax for callbacks is as follows:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">callback identifier = return_type (/* arguments... */);</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-Dictionaries">Dictionaries</h4><p>Dictionaries are equivalent to C structs.  The syntax for dictionaries is:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">dictionary identifier {
  /* dictionary_members... */
};</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-Interfaces">Interfaces</h4><p>Interfaces are equivalent to structs in C++; they contain &quot;attributes&quot;, which are equivalent to structure fields, and &quot;operations&quot;, which are equivalent to methods.  The syntax for interfaces is as follows:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">interface identifier {
    attribute long x;
    double foo(string y, boolean z);
  /* ...etc. */
};</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-Example">Example</h4><p>As a running example to show how all of the pieces fit together, we will build a WebIDL file for a calculator.  The calculator will support four functions, described by an enumeration type:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };</pre>
</div></div><p>The calculator will be defined by an interface with a an attribute to control the precision of the output and a single operation:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">interface Calculator {
	attribute long digits_of_precision;
    float calculate( Calculator_Function function, float arg1, float arg2 );
};</pre>
</div></div><p>...to use this WebIDL file, we'll have to build the compiler and interpreter...</p><h2 id="UsingJerryGen,theWebIDLCompiler-Buildingthecompiler">Building the compiler</h2><p><span class="inline-comment-marker" data-ref="2add9a68-a084-4d6e-a703-eec4bf49699b">First</span>, create a directory in which to work and go there.  For the rest of the document, we'll assume that the directory is called <code>~/JerryGen</code>.</p><p> Now, clone the repo :</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">git clone ssh://git@bitbucket.itg.ti.com/jp/jerrygen.git</pre>
</div></div><p>or, from the public GitHub:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">git clone https://github.com/t-harvey/JerryGen.git</pre>
</div></div><p><br/></p><p>...next, install the necessary Javascript modules:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">cd generator
npm install
cd ..</pre>
</div></div><p>...now, fork the Jerryscript repo ({{jerryscript-project/jerryscript}}) from GitHub:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">git clone https://github.com/jerryscript-project/jerryscript.git</pre>
</div></div><p>...build the pieces of the interpreter:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">cd jerryscript
tools/build.py --debug --clean --verbose --lto=OFF
cd ..
</pre>
</div></div><p>...now, create a space to work and go there:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">mkdir working
cd working</pre>
</div></div><p>Next, cut/paste the WebIDL constructs for our simple calculator into a single file called &quot;simple_calculator.idl&quot;.  (We've shown the pieces from the earlier example in this code block to make it easier)</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };


interface Calculator {
	attribute long digits_of_precision;
    float calculate( Calculator_Function function, float arg1, float arg2 );
};</pre>
</div></div><h2 id="UsingJerryGen,theWebIDLCompiler-InvokingtheCompiler">Invoking the Compiler</h2><p>The compiler is a conglomeration of Javascript and Mustache code initiated by the generate.js script (historically, the compiler is referred to as the &quot;generator&quot;).  Invoking the script (from ~/JerryGen/working) with no parameters (or incorrect parameters) will show the usage instructions:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">-&gt; ../generator/generate.js 
ERROR: you must supply a package name and .idl file(s).
          
Usage: generate.js &lt;options&gt; --package=&lt;name&gt; &lt;.idl file(s)&gt;
Options:
fix_type_errors       possible values: true, false
stubs                 possible values: on, off, overwrite
debug_printing        possible values: off, on
output_utility_files  possible values: false, true
quiet                 possible values: false, true
help                  
Example:
generate.js --stubs=on --package=foobar foo.idl bar.idl</pre>
</div></div><p>The generator produces many files from a single WebIDL file, as explained in the next section.</p><p>The &quot;package&quot; parameter is the name of the directory that will be created to put these files into; in addition to the actual WebIDL files, this is the only required  parameter (and it must immediately precede the list of WebIDL files (which must be last on the command line)).</p><p>The following table explains the remaining parameters.</p><div class="table-wrap"><table class="relative-table wrapped confluenceTable" style="width: 100.0%;"><colgroup><col style="width: 10.351869%;"/><col style="width: 9.679431%;"/><col style="width: 79.969925%;"/></colgroup><tbody><tr><th class="confluenceTh">Parameter</th><th class="confluenceTh"><p>Values</p><p>(default in red)</p></th><th class="confluenceTh">Explanation</th></tr><tr><td class="confluenceTd">fix_type_errors</td><td class="confluenceTd"><span style="color: rgb(255,0,0);">true</span>, false</td><td class="confluenceTd">Simple type errors are fixed; in particular, we found that many programmers left off the return type of operations, so we default to a type of &quot;void&quot; for those operations and print error messages telling the user what the compiler did.</td></tr><tr><td class="confluenceTd">stubs</td><td class="confluenceTd"><span style="color: rgb(255,0,0);">on</span>, off, overwrite</td><td class="confluenceTd">Controls the creation of the *_stubs files, explained in the next section. We default to creating stubs files each time the compiler is invoked, but because the stubs contain human-generated code, we don't overwrite existing stubs files. This can be changed with the &quot;overwrite&quot; value, which creates new stubs files on top of any existing stubs files.</td></tr><tr><td class="confluenceTd">debug_printing</td><td class="confluenceTd"><span style="color: rgb(255,0,0);">off</span>, on</td><td class="confluenceTd">When set to &quot;on&quot;, the compiler will produce debug_print_* functions for every type, and each operation will print out the passed-in values of its parameters.</td></tr><tr><td class="confluenceTd">output_utility_files</td><td class="confluenceTd"><span style="color: rgb(255,0,0);">false</span>, true</td><td class="confluenceTd">All of the code produced by the compiler relies on a library of utility functions. Instead of storing this file in some arbitrary directory, the user can have the compiler write out the utilities whenever/whereever he needs.</td></tr><tr><td class="confluenceTd">quiet</td><td class="confluenceTd"><span style="color: rgb(255,0,0);">false</span>, true</td><td class="confluenceTd">Normally, the compiler shows the list of files created; setting this value to &quot;true&quot; will inhibit all but error messages.</td></tr><tr><td class="confluenceTd">help</td><td class="confluenceTd"><br/></td><td class="confluenceTd">Print out the usage message (above).</td></tr></tbody></table></div><p>For our example, invoke the compiler as follows:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">../generator/generate.js --output_utility_files --package=simple_calculator simple_calculator.idl</pre>
</div></div><p>...the output should be a listing of the files the compiler <span class="inline-comment-marker" data-ref="28098fa2-6f98-43da-988a-c3e44b03b8ed">created</span>:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">Creating directory... (/Users/a0179262/work/examples/simple_calculator)
Creating C File... (/Users/a0179262/work/examples/simple_calculator/Calculator.h)
Creating C File... (/Users/a0179262/work/examples/simple_calculator/Calculator_private.h)
Creating C File... (/Users/a0179262/work/examples/simple_calculator/Calculator.c)
Creating C Stubs File: &gt;/Users/a0179262/work/examples/simple_calculator/Calculator_stubs.h&lt;
Creating C Stubs File: &gt;/Users/a0179262/work/examples/simple_calculator/Calculator_stubs_private.h&lt;
Creating C Stubs File: &gt;/Users/a0179262/work/examples/simple_calculator/Calculator_stubs.c&lt;
Creating C File... (/Users/a0179262/work/examples/simple_calculator/Calculator_Function.h)
Creating C File... (/Users/a0179262/work/examples/simple_calculator/Calculator_Function_private.h)
Creating C File... (/Users/a0179262/work/examples/simple_calculator/Calculator_Function.c)
Creating C Utilities File: &gt;/Users/a0179262/work/examples/simple_calculator/webidl_compiler_utilities.h&lt;
Creating C Utilities File: &gt;/Users/a0179262/work/examples/simple_calculator/webidl_compiler_utilities_private.h&lt;
Creating C Utilities File: &gt;/Users/a0179262/work/examples/simple_calculator/webidl_compiler_utilities.c&lt;</pre>
</div></div><p>...the next section explains these files.</p><h2 id="UsingJerryGen,theWebIDLCompiler-Filesproducedbythegeneratorforoursimplecalculator">Files produced by the generator for our simple calculator</h2><p>The compiler produces three C files for each WebIDL construct, a public header file (aka .h file), a private header file (indicated with the suffix &quot;_private.h&quot;), and a supporting code file (aka .c file).  The public header files describe the C data structures and calls associated with the construct.  The private header files and code files implement the glue code for the interpreter and need not be examined.  The file names are derived from the name of the construct.</p><p>Every type, both intrinsic types like float and long, as well as user-defined types from each WebIDL file, has a <code>&lt;type_name&gt;_uid</code> value – this should be considered a read-only value, although the user will need to use it for composite types (see the section on composite types for more).</p><p>Below, we'll look at what's important in the .h files created for our calculator example.</p><h4 id="UsingJerryGen,theWebIDLCompiler-EnumerationTypes">Enumeration Types</h4><p>Each .h file for an enumeration type contains the enumeration type itself.  Everything else in the file should be ignored.</p><h4 id="UsingJerryGen,theWebIDLCompiler-Interfaces.1">Interfaces</h4><p>Interfaces are objects, made up of attributes and operations – the attributes correspond to object fields, and the operations are methods.  The .h file created for an interface will contain a struct of fields and function pointers, much like callbacks.  In our example, the Calculator.h file contains the Calculator struct.</p><p>Operations are the functionality that needs to be built by a human.  To facilitate this, each interface produces not only a .c/.h pair, but also a pair of .c/.h files that contain the &quot;stubs&quot;, where a &quot;stub&quot; is, essentially, an empty function that looks like the operation defined in the WebIDL file.</p><h4 id="UsingJerryGen,theWebIDLCompiler-Fillingthestubs">Filling the stubs</h4><p>NOTE: ignore the &quot;<code>Native_Object</code>&quot; code for now; that functionality will be explained, below.</p><p>There is only one operation for the <code>Calculator</code> interface.  The name of the operation's stub is derived from the interface name and operation name, so the function that we'll edit is named <code>Calculator_calculate</code>.  Notice that the call signature for the stub functions match exactly the types specified by the WebIDL file; the underlying mechanics of hooking into the interpreter (e.g., marshaling/unmarsharling parameters) is handled by all of the underlying C code (that we encourage the user to ignore).</p><p>So again: inside of Calculator_calculator, look for the string &quot;USER CODE GOES HERE&quot; – all code produced by the compiler will have this string wherever the user needs to specialize the code – and add the following.  (Notice that each value in an enumeration type has as its prefix the name of the enumeration type; this is b/c C doesn't distinguish between two enumeration-type values in separate enumeration types, leading to annoying name clashes.  The way around this was to tack on the enumeration type's name onto each of its values – if this is too burdensome, the generator accepts the &quot;–leave_enums_alone=true&quot; flag.)</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">switch (function)
{
    case Calculator_Function_add:
        return arg1 + arg2;
    case Calculator_Function_subtract:
        return arg1 - arg2;
    case Calculator_Function_multiply:
        return arg1 * arg2;
    case Calculator_Function_divide:
        return arg1 / arg2;
}</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-BuildingJerryscript">Building Jerryscript</h4><p>To build the interpreter with the <code>Calculator</code> example, go into the <code>simple_calculator</code> directory.  </p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">cd simple_calculator</pre>
</div></div><p>In this directory are the files created from the WebIDL input.  We will need to compile these with a main routine and then link all of these with the interpreter libraries.  The command is:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">gcc -g --std=c99 -Djerry_value_has_error_flag=jerry_value_is_error -I../../jerryscript/jerry-port/default/include -I../../jerryscript/jerry-core/include -I../../jerryscript/jerry-ext/include -I../../jerryscript/jerry-ext/include/jerryscript-ext -I. ../../generator/unit_tests/template/main_jerrygen.c *.c ../../jerryscript/build/lib/libjerry-core.a ../../jerryscript/build/lib/libjerry-ext.a ../../jerryscript/build/lib/libjerry-port-default.a -lm &amp;&amp; ./a.out</pre>
</div></div><p>...this will build and run (b/c the command ends with &quot;&amp;&amp; ./a.out&quot;) the command line interpreter.  Because all of the paths are relative (i.e., &quot;../..&quot;), this command will work for building all of the examples in the rest of the document – the usual method is to build a package with the generator, cd into that directory, modify the *_stubs.c file(s), and use the above command to compile and get an interpreter.</p><p>Try the <span class="inline-comment-marker" data-ref="6f78a50f-29b6-4f43-819f-762cb6a6c710">following commands</span> (use &quot;control-d&quot; on a blank line to end the repl):</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var calc = new Calculator;
calc.calculate(&quot;add&quot;, 1.55, 3.77);
calc.calculate(&quot;divide&quot;, 1.55, 3.77);
</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-AccessingAttributes">Accessing Attributes</h4><p>Notice that we haven't provided all of the functionality that we promised; we designed the Calculator to only compute values to a specified degree of accuracy, but we didn't utilize the <code>digits_of_precision</code> attribute.</p><p>Attributes live on the (Javascript) object that is the interface, so we use a macro/function to call into the Javascript environment and get an attribute's value.  The INTERFACE_EXTRACT macro is defined in the <span style="font-family: monospace;">webidl_compiler_utilities</span> file.  The macro is used to access any attribute by giving the attribute name as one of the parameters to the macro.  The macro is just shorthand for calls to each attribute's <code>extract</code> function: the naming convention for attribute access functions is: get<code>_&lt;Interface Name&gt;_&lt;attribute_name&gt;(), if you want to use those directly</code>, and the parameter is the function argument <span style="font-family: monospace;">self</span>, which is a handle back through the interpreter to the Javascript object.  For these examples, we'll use the macro; but it may be a better interface to allow the user to call the <code>extract_*</code> functions directly, which is why we've included them.</p><p>With that function, we can rewrite the <code>Calculator_calculate</code> function to use the <code>digits_of_precision</code> attribute as follows:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">float answer;
switch (function)
{
    case Calculator_Function_add:
        answer = arg1 + arg2;
        break;
    case Calculator_Function_subtract:
        answer = arg1 - arg2;
        break;
    case Calculator_Function_multiply:
        answer = arg1 * arg2;
        break;
    case Calculator_Function_divide:
        answer = arg1 / arg2;
        break;
}

/* move the decimal over by the amount to shift, type-convert this new
   value into an integer to truncate the remaning decimal, and shift it back */
int shift_amount = 1;
for(int i = 0; i &lt; INTERFACE_EXTRACT(this, Calculator, digits_of_precision); i++)
    shift_amount *= 10;
answer = (float)((float)((int)(answer*shift_amount))/(float)(shift_amount));

 return answer;

</pre>
</div></div><p>...and now try the following commands.  Notice that we have built a new <code>Calculator</code> object and supplied a value for the attribute digits_of_precision – constructors assign their parameters in order of the WebIDL declarations, so if an interface, I, has <span class="inline-comment-marker" data-ref="6e5006c5-5761-4fd9-8611-12126e61cf74">attributes named a, b, and c,</span> the command &quot;<code>new_thing = new I(x, y, z)&quot;</code>, will assign <code>x</code> to <code>a</code>, <code>y</code> to <code>b</code>, and <span style="font-family: monospace;">z</span> to <span style="font-family: monospace;">c</span>.  Our WebIDL compiler includes default values for all types, so if not all of the attributes are filled in by the constructor, those fields get initialized with a default value. </p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var calc = new Calculator(2);
calc.calculate(&quot;add&quot;, 1.55, 3.77);
calc.calculate(&quot;divide&quot;, 1.55, 3.77);
</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-UsingCallbacks">Using Callbacks</h4><p>... the answers in our example are close to what we'd like to see, but not exactly.  Although not the main point, the example illustrates the problem of handing values between differing environments – Javascript's only numeric type is a double, so converting between ints/longs/floats/doubles/etc. is likely to give slightly different results.</p><p>An alternative approach will work better.  Since we suspect that the problem is the handoff between different environments, we can perform the rounding in the Javascript environment by using a callback that we will store as part of the interface.  Let's create a new file, called <code>simple_calculator2.idl</code>, that will have the updated interface:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };

callback rounding_function = float (float unrounded_value, long precision);

interface Calculator {
	attribute long digits_of_precision;
	attribute rounding_function round_it;
    float calculate( Calculator_Function function, float arg1, float arg2 );
};</pre>
</div></div><p>...build the files with the generator (notice that we're using the &quot;<code>simple_calculator2.idl</code>&quot; file and creating the files in a new directory, &quot;simple_calculator2&quot;) :</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">../generator/generate.js --output_utility_files --package=simple_calculator2 simple_calculator2.idl</pre>
</div></div><p>...and we'll augment the Calculator_calculate as follows.  Note that invoking a Javascript function from the C code requires an additional parameter, the &quot;this&quot; pointer that all operation's bodies have as their last parameter.  As a result, the C type associated with a callback is actually the native type of the interpreter, what we call &quot;<code>Interpreter_Type</code>&quot; – in practice, this is just an integer handle to an interpreter-internal value, and because we don't have direct access to it, we will always invoke callbacks through a wrapper whose name follows the pattern: &quot;run_&lt;callback_name&gt;_function&quot;, and its parameters will be the <code>Interpreter_Type</code> value representing this kind of callback, the &quot;<code>self</code>&quot; pointer, and then the actual parameters of the callback.</p><p>In the code below, </p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">float answer;
switch (function)
{
    case add:
        answer = arg1 + arg2;
        break;
    case subtract:
        answer = arg1 - arg2;
        break;
    case multiply:
        answer = arg1 * arg2;
        break;
    case divide:
        answer = arg1 / arg2;
        break;
}

/* use the callback to round the answer before returning */
rounding_function rounding_func = INTERFACE_EXTRACT(self, Calculator, round_it);
int shift_amount = INTERFACE_EXTRACT(self, Calculator, digits_of_precision);

answer = (float)run_rounding_function_function(rounding_func, self,
                                               answer, shift_amount);

return answer;</pre>
</div></div><p>Now, build an interpreter with the files in the simple_calculator2 directory and try these Javascript commands:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);};
var calc = new Calculator(2, round);
calc.calculate(&quot;multiply&quot;, 1.55, 3.77)</pre>
</div></div><p>...you'll notice that we still get some rounding errors – as an aside, you can get the answer we're looking for inside of Jerryscript by typing the following directly at the Jerryscript prompt:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">calc.round_it(calc.calculate(&quot;multiply&quot;, 1.55, 3.77), 2);
</pre>
</div></div><p><strong>Using dictionaries</strong></p><p>Now imagine that we want to send in the arguments and calculator operation directly in to the &quot;calculate&quot; method.  We can use a dictionary for this:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };

callback rounding_function = float (float unrounded_value, long precision);

dictionary Calculator_Arguments {
     Calculator_Function function;
     float arg1;
     float arg2;
};

interface Calculator {
    attribute long digits_of_precision;
    attribute rounding_function round_it;
    float calculate( Calculator_Arguments args );
};</pre>
</div></div><p>Cut/paste this into a file called <code>simple_calculator3.idl</code> and invoke the generator.  The following code should be pasted into Calculator_calculate:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">float answer;

switch (args.function)
{
    case Calculator_Function_add:
        answer = args.arg1 + args.arg2;
        break;
    case Calculator_Function_subtract:
        answer = args.arg1 - args.arg2;
        break;
    case Calculator_Function_multiply:
        answer = args.arg1 * args.arg2;
        break;
    case Calculator_Function_divide:
        answer = args.arg1 / args.arg2;
        break;
}

/* use the callback to round the answer before returning */
rounding_function rounding_func = INTERFACE_EXTRACT(this, Calculator, round_it);
int shift_amount = INTERFACE_EXTRACT(this, Calculator, digits_of_precision);

answer = (float)run_rounding_function_function(rounding_func, this,
                                               answer, shift_amount);

return answer;</pre>
</div></div><p>...and now with our new interface, some slightly modified Javascript:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);};
var calc = new Calculator(2, round);
var args = new Calculator_Arguments(&quot;multiply&quot;, 1.55, 3.77);
calc.calculate(args)

/* or -- all in one cut/paste: */
var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);}; var calc = new Calculator(2, round); var args = new Calculator_Arguments(&quot;multiply&quot;, 1.55, 3.77); calc.calculate(args)</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-Arrays">Arrays</h4><p>Now let's build a calculator with an added feature: our new calculator should be able to calculate on an arbitrary number of arguments.  For this, we'll use WebIDL's &quot;sequence&quot; declaration, which is analogous to C arrays.  Because we need to know the length of an array, all arrays translated from WebIDL will be structs containing the array of values plus a field containing the length of the array of values.</p><p>The WebIDL for <code>simple_calculator4.idl</code> looks like:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };

callback rounding_function = float (float unrounded_value, long precision);

dictionary Calculator_Arguments {
     Calculator_Function function;
     sequence&lt;float&gt; args;
};

interface Calculator {
    attribute long digits_of_precision;
    attribute rounding_function round_it;
    float calculate( Calculator_Arguments args );
};</pre>
</div></div><p>Notice that invoking the generator on this file produces more files than the last example – arrays, because they are structs in their own right, will produce their own .c/.h files.</p><p>The code for <code>Calculator_calculate</code> is:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">float answer = 0.0;
if (args.function == Calculator_Function_multiply || args.function == Calculator_Function_divide)
	answer = 1.0;

for (int i = 0; i &lt; args.args.length; i++)
    switch (args.function)
    {
        case Calculator_Function_add:
            answer += args.args.items[i];
            break;
        case Calculator_Function_subtract:
            answer -= args.args.items[i];
            break;
        case Calculator_Function_multiply:
            answer *= args.args.items[i];
            break;
        case Calculator_Function_divide:
            answer /= args.args.items[i];
            break;
    }

/* use the callback to round the answer before returning */
rounding_function rounding_func = INTERFACE_EXTRACT(this, Calculator, round_it);
int shift_amount = INTERFACE_EXTRACT(this, Calculator, digits_of_precision);

answer = (float)run_rounding_function_function(rounding_func, this,
                                        answer, shift_amount);

return answer;</pre>
</div></div><p>...and the Javascript commands now look like:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);};
var calc = new Calculator(2, round);
var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77]);
calc.calculate(args)

/* or -- all in one cut/paste: */
var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);}; var calc = new Calculator(2, round); var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77]); calc.calculate(args)</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-CompositeTypes">Composite Types</h4><p>Now, let's suppose that we want our calculator to handle lists of floats or integers.  We could write a whole new <code>calculate</code> function that takes longs instead of floats, but WebIDL has a way to specify &quot;composite&quot; types, types made up of more than one other type.  The syntax is the composite types are surrounded by parentheses and joined by the word &quot;or&quot;.  For example, imagine that we want a dictionary to hold one of two enumeration types – the WebIDL would look something like:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum ordinal_number { &quot;first&quot;, &quot;second&quot;, &quot;third&quot; };
enum nominal_number { &quot;one&quot;, &quot;two&quot;, &quot;three&quot; };


dictionary number {
	(ordinal_number or nominal_number) value;
};</pre>
</div></div><p>...in C, composite types are represented as unions.  The canonical method of implementing unions in C is with a struct with two fields: the first field describes which of the members is stored in the union, while the second is the union itself.  In the above example, the generator would produce the following structure for use in the <code>number</code> dictionary:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">typedef struct {
    int union_type;
    union {
        nominal_number nominal_number_field;
        ordinal_number ordinal_number_field;
    } value;
} nominal_number_or_ordinal_number;
</pre>
</div></div><p>...notice that the name of the union is simply a concatenation of the types in the WebIDL file – we flatten the composite (i.e.: &quot;(a or (b or (c or d)))&quot; is treated as &quot;(a or b or c or d)&quot;...) and alphabetize the types to canonicalize the name of each composite type.  Each member of the union is the type name with &quot;<code>_field</code>&quot; tacked on to the end.</p><p>This brings us back around to a detail mentioned at the beginning of this document: what are the <code>&lt;type_name&gt;_uid</code> values used for?  As we said, every type, both the intrinsic types and types built in the WebIDL files have their own uid, and it is used in the union's structure to describe the union's member type.</p><p>So now, let's modify our latest calculator to accept an array of either long or float values (we're now on simple_calculator5):</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };

callback rounding_function = float (float unrounded_value, long precision);

dictionary Calculator_Arguments {
     Calculator_Function function;
     sequence&lt;(float or long)&gt; args;
};

interface Calculator {
    attribute long digits_of_precision;
    attribute rounding_function round_it;
    float calculate( Calculator_Arguments args );
};
</pre>
</div></div><p>...and the C code for <code>Calculator_calculate</code>:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">float answer = 0.0;
if (args.function == Calculator_Function_multiply || args.function == Calculator_Function_divide)
	answer = 1.0;

for (int i = 0; i &lt; args.args.length; i++)
{
    float_or_long next_arg = args.args.items[i];
    float arg;
    switch(next_arg.union_type)
    {
        case float_uid:
            arg = next_arg.value.float_field;
            break;
        case long_uid:
            arg = (float) next_arg.value.long_field;
            break;
    }
    switch (args.function)
    {
        case Calculator_Function_add:
            answer += arg;
            break;
        case Calculator_Function_subtract:
            answer -= arg;
            break;
        case Calculator_Function_multiply:
            answer *= arg;
            break;
        case Calculator_Function_divide:
            answer /= arg;
            break;
    }
}
/* use the callback to round the answer before returning */
rounding_function rounding_func = INTERFACE_EXTRACT(this, Calculator, round_it);
int shift_amount = INTERFACE_EXTRACT(this, Calculator, digits_of_precision);

answer = (float)run_rounding_function_function(rounding_func, this,
                                        answer, shift_amount);
return answer;</pre>
</div></div><p>...and try the following Javascript:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);};
var calc = new Calculator(2, round);
var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77, 2]);
calc.calculate(args)


/* or -- all in one cut/paste: */
var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);}; var calc = new Calculator(2, round); var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77, 2]); calc.calculate(args)</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-NativeObjects">Native Objects</h4><p>At the beginning of this tutorial, we recommended ignoring a large block of code at the beginning of the <code>Calculator_stubs.c</code> file.  This code relates to a &quot;hidden&quot; piece of memory that gets associated with each interface.  The idea is that because the interface is being computed on the C side of the interpreter and implements libraries that remain opaque to the Javascript user, it may be necessary to save &quot;state&quot; of the interface that is invisible/inaccessible to the user.</p><p>The Jerryscript interpreter calls these &quot;<code>Native Objects</code>&quot;, so we just borrowed the name – a <code>Native Object</code> is a piece of memory that Jerryscript associates with a Javascript object and can be accessed by the C code (specifically, each of the operations/methods in an interface).</p><p>As we've seen from our running example, interfaces are fully functional without utilizing the <code>Native Object</code> data structure.</p><p>In our running example, let's add a feature to the calculator that every tenth time we call calculate, we'll print out an extra message congratulating the user.</p><p>To keep track of the call count, go into <code>Calculator_stubs.h</code> and edit the data structure named <code>Native_Object_Calculator</code>:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">typedef struct {
    /* USER CODE GOES HERE */
    int call_count;
} Native_Object_Calculator;</pre>
</div></div><p>...now, we need to tell the code how to initialize the <code>Native Object</code> for our <code>Calculator</code> interface.  This can be found in <code>Calculator_stubs.c</code>, and is the function <code>Native_Object_create</code>; the code defaults to malloc'ing the<code> Native_Object_Calculator</code> data structure, and we can add our own code to perform initialization of our <code>Native Object</code>.  This function gets called each time a new interface gets created.</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">Native_Object_Calculator *Native_Object_Calculator_create(void)
{
    Native_Object_Calculator *new_object = (Native_Object_Calculator *)malloc(sizeof(Native_Object_Calculator));

        /* USER CODE GOES HERE */
    new_object-&gt;call_count = 0; 
    return new_object;
} /* Native_Object_Calculator_create */</pre>
</div></div><p>...Jerryscript is garbage collected, so when it needs to garbage-collect an interface, we need to tell it how to deallocate the <code>Native Object</code> associated with the interface.  The deallocating function is co-located with the create function – in our example, it is called <code>Native_Object_Calculator_deallocator</code>, and we would want to put a call to free here, so that when an <code>Calculator</code> interface gets deallocated, we also free the memory associated with our call counter:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">void Native_Object_Calculator_deallocator(void *native_object)
{
        /* USER CODE GOES HERE */
    free(native_object);
} /* Native_Object_Calculator_deallocator */
</pre>
</div></div><p>...and this explains the code at the top of the <code>Calculator_calculate</code> function: each method defaults to retrieving the pointer to the <code>Native Object</code> associated with the method's interface.  Adding code to increment the count at each call is straightforward.</p><p>As a last point: all of the stubs contain a call to Native_Object_get to set up a local pointer to the interface's Native_Object.  If a particular stub does not use that pointer, this call should, for efficiency, be removed – also, if the pointer changes to point to a new object, the programmer has to remember to include a call to Native_Object_set to ensure that the interface gets the new value.</p><h4 id="UsingJerryGen,theWebIDLCompiler-SeparateCompilation–theExternalInterfaceattribute">Separate Compilation – the ExternalInterface attribute</h4><p>Although WebIDL does not have a model for separate compilation, we know that C programmers especially will not want to compile entire, monolithic programs.  In this section, we'll look at how to create separately compilable glue/stub code from different .idl files that can be compiled into a single interpreter.</p><p>First, WebIDL allows the writer to attach arbitrary attributes to interfaces – each attribute may not be part of the language (like a reserved word), the list of attributes can be any length, and interpreters are free to ignore them.  These attributes precede the <code>interface</code> declaration and are declared as a comma-separated list surrounded by square brackets.</p><p>Supporting separate compilation requires a mechanism to list, in one file, interfaces that might be defined in a separate file.  For this, we use an attribute called <code>ExternalInterface</code> that takes a single parameter, the name of the external interface.</p><p>As an example, imagine that we have an interface with a single operation that takes two parameters, both of which are external.  The .idl file would look like this:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">[ ExternalInterface=(Extern_Int1), ExternalInterface=(Extern_Int2) ]
interface Example_Interface {
    void foo(Extern_Int1 x, Extern_Int2 y);
};</pre>
</div></div><p> ...here, the user will generate code for <code>Extern_Int1</code> and <code>Extern_Int2 at a different time</code>.  This declaration allows the WebIDL compiler to put out #include directives in the C code.</p><p>To make this concrete, let's imagine that our calculator is used by teachers to monitor their students' activity – the students call the calculator with a special data structure the teacher controls.  For this, create a file called teacher.idl with the following .idl code:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">interface Teacher_Feedback {
    void record_activity(string student_name);
};</pre>
</div></div><p>...and compile that with the usual command:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">../generator/generate.js --package=teacher teacher.idl</pre>
</div></div><p>Notice that we didn't generate utility files; this only needs to be done once per compilation, and we'll do it when we compile our new calculator, below.</p><p>For now, we'll make the body of &quot;record_activity&quot; a simple printf just to show it's working:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">printf(&quot;Inside record_activity, with student: %s\n&quot;, student_name);</pre>
</div></div><p><br/></p><p>Now, copy simple_calculator5.idl into simple_calculator6.idl, and add the <code>Teacher_Feedback</code> interface (compile it with the usual call to the generator) :</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };

callback rounding_function = float (float unrounded_value, long precision);

dictionary Calculator_Arguments {
     Calculator_Function function;
     sequence&lt;(float or long)&gt; args;
};

[ExternalInterface=(Teacher_Feedback)]
interface Calculator {
	attribute string student_name;
    attribute long digits_of_precision;
    attribute rounding_function round_it;
    float calculate( Calculator_Arguments args, Teacher_Feedback my_teacher );
};
</pre>
</div></div><p>It may also be instructive to show how this code would be used inside of <code>calculate</code>.  Because the <code>my_teacher</code> parameter is an interface and interfaces live in memory on the Javascript side, we need to use a special macro designed to call functions that are stored as interface attributes.  Add the following code to `Calculator_calculate` in `Calculate_stubs.c`.</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">/* call the teacher&#39;s interface */
string student_name = INTERFACE_EXTRACT(this, Calculator, student_name);
Teacher_Feedback_record_activity(my_teacher, student_name, error);

float answer = 0.0;
if (args.function == Calculator_Function_multiply || args.function == Calculator_Function_divide)
	answer = 1.0;

for (int i = 0; i &lt; args.args.length; i++)
{
    float_or_long next_arg = args.args.items[i];
    float arg;
    switch(next_arg.union_type)
    {
        case float_uid:
            arg = next_arg.value.float_field;
            break;
        case long_uid:
            arg = (float) next_arg.value.long_field;
            break;
    }
    switch (args.function)
    {
        case Calculator_Function_add:
            answer += arg;
            break;
        case Calculator_Function_subtract:
            answer -= arg;
            break;
        case Calculator_Function_multiply:
            answer *= arg;
            break;
        case Calculator_Function_divide:
            answer /= arg;
            break;
    }
}
/* use the callback to round the answer before returning */
rounding_function rounding_func = INTERFACE_EXTRACT(this, Calculator, round_it);
int shift_amount = INTERFACE_EXTRACT(this, Calculator, digits_of_precision);

answer = (float)run_rounding_function_function(rounding_func, this,
                                        answer, shift_amount);
return answer;</pre>
</div></div><p>Now generate the new calculator with the usual command.  It's instructive to look at the command used to compile a new interpreter:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">gcc -g --std=c99 -Djerry_value_has_error_flag=jerry_value_is_error -I../../jerryscript/jerry-port/default/include -I../../jerryscript/jerry-core/include -I../../jerryscript/jerry-ext/include -I../../forked_jerryscript/jerry-ext/include/jerryscript-ext -I. -I../teacher ../../generator/unit_tests/template/main_jerrygen.c *.c ../teacher/*.c ../../jerryscript/build/lib/libjerry-core.a ../../jerryscript/build/lib/libjerry-ext.a ../../jerryscript/build/lib/libjerry-port-default.a -lm &amp;&amp; ./a.out</pre>
</div></div><p>Note that we added both the C code (&quot;<code>../teacher/*.c</code>&quot;) and the include path to the Teacher_Feedback code (&quot;<code>-I../teacher</code>&quot;).  Also remember to set up the Native_Object itself as we showed in the previous section.</p><p>...and the Javascript for this example:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);};
var calc = new Calculator(&quot;Michael&quot;, 2, round);
var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77, 2]);
var teacher = new Teacher_Feedback;
calc.calculate(args, teacher);


/* or -- all in one cut/paste: */
var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);}; var calc = new Calculator(&quot;Michael&quot;, 2, round); var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77, 2]); var teacher = new Teacher_Feedback; calc.calculate(args, teacher);</pre>
</div></div><h4 id="UsingJerryGen,theWebIDLCompiler-UsingtheNative_ObjectstructureandcallingJerryGen-createdfunctionsdirectly">Using the Native_Object structure and calling JerryGen-created functions directly</h4><p>The example with the Teacher_Feedback is unrealistic (<code>calculate</code> does not need an extra, largely unrelated parameter); in the next example, we build on the Teacher_Feedback idea, but this time, the <code>Calculator</code> object will have a hidden structure holding the <code>Teacher_Feedback</code> interface.  The idea is that the teacher will want to monitor his student's use of the calculator, but that monitoring should be transparent to the student.  To do this, we'll set up the <code>Teacher_Feedback</code> interface in the <code>Native_Object_Calculator_create</code> function, which it gets when initializing a <code>Calculator</code> interface.</p><p>We'll start with a (slightly) simplified <code>.idl</code> file, which we'll call <code>simple_calculator7.idl</code>:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">enum Calculator_Function { &quot;add&quot;, &quot;subtract&quot;, &quot;multiply&quot;, &quot;divide&quot; };

callback rounding_function = float (float unrounded_value, long precision);

dictionary Calculator_Arguments {
     Calculator_Function function;
     sequence&lt;(float or long)&gt; args;
};

interface Calculator {
	attribute string student_name;
    attribute long digits_of_precision;
    attribute rounding_function round_it;
    float calculate( Calculator_Arguments args );
};</pre>
</div></div><p>Notice that we have removed the ExternalInterface attribute on the Calculator interface – the user of this API will no longer see the Teacher_Feedback interface (because we have removed it from the call to calculate), even though we will still use it in require_Calculator function.</p><p>We need to define the <code>Native_Object</code> to contain the <code>Teacher_Feedback</code> object, so add the following code to <code>Native_Object_Calculator</code> struct definition in <code>Calculator_stubs.h</code>:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">    Teacher_Feedback feedback_interface;</pre>
</div></div><p>...of course, since we have introduced a type that wasn't derived from the WebIDL file by JerryGen, we need to add the include file at the top of <code>Calculator_stubs.h</code> manually:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">#include &quot;Teacher_Feedback.h&quot;</pre>
</div></div><p>...and then set up the interface inside of <code>Native_Object_Calculator_create()</code> – this will create a new <code>Teacher_Feedback</code> each time the <code>Calculator</code> constructor is called.</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">    new_object-&gt;feedback_interface = Teacher_Feedback_constructor();</pre>
</div></div><p>For the body of <code>calculate</code>, notice that we can derive the C functions that get created for each construct in a WebIDL file.  (Some day, we'll have a guide showing all of the names that the compiler creates...)  Thus, the constructor for a <code>Teacher_Feedback</code> is called <code>Teacher_Feedback_constructor()</code>.</p><p><span style="letter-spacing: 0.0px;">...all of these examples exemplify a very important rule for using JerryGen: </span><em style="letter-spacing: 0.0px;"><strong>ONLY ADD CODE TO THE *_stubs FILES</strong></em><span style="letter-spacing: 0.0px;">.  Remember that the <code>*_stubs</code> files are retained when we recompile a <code>.idl</code> file, while all of the rest of the code is overwritten.</span></p><p>Now, the body of <code>calculate</code> uses the <code>Teacher_Feedback</code> interface that is stored in the <code>native_object</code> variable.  Its single operation has a name that can be derived from its WebIDL file (remember that we have to add the <code>self</code> pointer – here, the object stored in the <code>Native_Object</code>) :</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">/* call the teacher&#39;s interface */
string student_name = INTERFACE_EXTRACT(self, Calculator, student_name);
Teacher_Feedback_record_activity(student_name, native_object-&gt;feedback_interface);

float answer = 0.0;
if (args.function == multiply || args.function == divide)
	answer = 1.0;

for (int i = 0; i &lt; args.args.length; i++)
{
    float_or_long next_arg = args.args.items[i];
    float arg;
    switch(next_arg.union_type)
    {
        case float_uid:
            arg = next_arg.value.float_field;
            break;
        case long_uid:
            arg = (float) next_arg.value.long_field;
            break;
    }
    switch (args.function)
    {
        case add:
            answer += arg;
            break;
        case subtract:
            answer -= arg;
            break;
        case multiply:
            answer *= arg;
            break;
        case divide:
            answer /= arg;
            break;
    }
}
/* use the callback to round the answer before returning */
rounding_function rounding_func = INTERFACE_EXTRACT(self, Calculator, round_it);
int shift_amount = INTERFACE_EXTRACT(self, Calculator, digits_of_precision);

answer = (float)run_rounding_function_function(rounding_func, self,
                                        answer, shift_amount);
return answer;</pre>
</div></div><p>Now generate the new calculator with the usual command.  (We didn't need to recompile teacher.idl; we assume it is still there from the previous example...)</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">gcc -g --std=c99 -Djerry_value_has_error_flag=jerry_value_is_error -I../../jerryscript/jerry-port/default/include -I../../jerryscript/jerry-core/include -I../../jerryscript/jerry-ext/include -I../../forked_jerryscript/jerry-ext/include/jerryscript-ext -I. -I../teacher ../../generator/unit_tests/template/main_jerrygen.c *.c ../teacher/*.c ../../jerryscript/build/lib/libjerry-core.a ../../jerryscript/build/lib/libjerry-ext.a ../../jerryscript/build/lib/libjerry-port-default.a -lm &amp;&amp; ./a.out</pre>
</div></div><p>Note that we added both the C code (&quot;<code>../teacher/*.c</code>&quot;) and the include path to the Teacher_Feedback code (&quot;<code>-I../teacher</code>&quot;).</p><p>...and the Javascript for this example:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);};
var calc = new Calculator(&quot;Cindy-Lou&quot;, 2, round);
var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77, 2]);
calc.calculate(args);


/* or -- all in one cut/paste: */
var round = function(value, amount) { return Number(Math.round(value+&#39;e&#39;+amount)+&#39;e-&#39;+amount);}; var calc = new Calculator(&quot;Cindy-Lou&quot;, 2, round); var args = new Calculator_Arguments(&quot;multiply&quot;, [1.55, 3.77, 2]); calc.calculate(args);</pre>
</div></div><p><span style="font-weight: bold;">A FInal Challenge:</span></p><p>The calculator starting with simple_calculator5.idl is a little odd, in that we pass in an array of either longs or floats, whereas usually you'd pass in an array of all floats or all longs.  Create a new <code>Calculator</code> that expects either an array of floats or an array of longs.</p><h4 id="UsingJerryGen,theWebIDLCompiler-Next:SeePart2">Next: See <a href="https://confluence.itg.ti.com/pages/viewpage.action?pageId=155070831" rel="nofollow">Part 2</a></h4>
                    </div>

                    
                                                        <div class="pageSection group">
                        <div class="pageSectionHeader">
                            <h2 id="comments" class="pageSectionTitle">Comments:</h2>
                        </div>

                        <table border="0" width="100%">
                                                        <tr>
                                <td >
                                    <a name="comment-118139860"></a>
                                    <font class="smallfont"><p>mkdir -p</p></font>
                                    <div align="left" class="smallfont" style="color: #666666; width: 98%; margin-bottom: 10px;">
                                        <img src="images/icons/contenttypes/comment_16.png" height="16" width="16" border="0" align="absmiddle"/>
                                        Posted by a0866938 at May 24, 2018 15:20
                                    </div>
                                </td>
                            </tr>
                                                    </table>
                    </div>
                                      
                </div>             </div> 
            <div id="footer" role="contentinfo">
                <section class="footer-body">
                    <p>Document generated by Confluence on Sep 20, 2019 14:03</p>
                    <div id="footer-logo"><a href="http://www.atlassian.com/">Atlassian</a></div>
                </section>
            </div>
        </div>     </body>
</html>
