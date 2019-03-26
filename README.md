When we translate WebIDL into C code, we often need to modify the
names slightly to account for C rules.  This document will explain all
of the conventions.

File Names

Each instance of an enumeration or callback type causes the creation
of one .c file and one .h file.  The names of the files are the names
of the types.

Interfaces and dictionaries cause the creation of three files, two .h
files and one .c file.  The names of the files are the names of the
types, but one of the .h files' names will have "_private" as a
suffix.  The intent is that the plain .h file will contain the data
structures and hooks that the C programmer needs, while the _private.h
file will contain the calls that the glue code needs, that the C
programmer should never have to touch.

| WebIDL Type | Files | Example |
| --- | --- | --- | --- |
| Enumerations | .h/.c | "enum_name" | enum_name.h<br>enum_name.c |
| Callbacks | .h/.c | "callback_name"| callback_name.h<br>callback_name.c |
| Definitions | .h/.c | "definition_name" | definition_name.h<br>definition_name_private.h<br>definition_name.c |
| Interfaces | .h/.c | "interface_name" | interface_name.h<br>interface_name_private.h<br>interface_name.c<br>interface_name_stubs.h<br>interface_name_strubs.c |

Enumeration Types

In WebIDL and Javascript, enumeration values are strings.  In C, the
names are simple integers, but their namespaces are shared -- this
means that if you have two enumeration types, <code>foo1</code> and
<code>foo2</code>, with the same value, say, <code>bar</code>, the C
compiler will refuse to compile that code, because it has no way to
distinguish <code>bar</code> in <code>foo1</code> from
<code>bar</code> in <code>foo2</code>.

To solve this problem, we prefix each enumeration value with its type
name.  In our example, we would have the values <code>foo1_bar</code> and
<code>foo2_bar</code>, which would not conflict.

Callbacks

Dictionaries

Interfaces
