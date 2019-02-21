# Coding Standards for JerryGen

The design of JerryGen is to support multiple interpreters and
multiple scripting languages.  Because no two projects share the same
set of coding standards, it would be impossible to conform to one set
of coding standards without violating another set.

Thus, we will specify only a minimal set of standards that make sense
to us.  They apply to the code that makes up this project and the code
that this project produces.

The first rule is that the code must be readable:
<ol>
<li>
Code should be self-documenting<br>
    variable names should reflect purpose
</li>
<li>
<p>Clear, Consise Comments</p>
    a. self-documenting code cannot capture the richness of detail that the programmer has in his head<br>
    b. a maintainer should not have to de-code an entire routine to understand an isolated section of that code<br>
    c. code should be largely understood from the comments, with actual code simply filling in the gaps<br>
</li>
</ol>
