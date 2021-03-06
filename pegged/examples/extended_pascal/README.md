# Folder `pegged/examples/extended_pascal`

## An example parser for ISO 10206:1990 Extended Pascal.

This grammar features several cases of (indirect) left-recursion. It is also
ambiguous. Normally, this ambiguity would be resolved by using a symbol table
and possibly semantic analysis, which is omitted in this example. So the syntax
table generated by this parser is not necessarily identical to the one
constructed by an Extended Pascal compiler.

The command `dub build` initiates a two-step build process:

 1. The file `source/make.d` is compiled and executed. This produces the parser
    module `epparser.d` from the grammar file `epgrammar.d`.
 2. The `parse_ep` parser application is built from `app.d` and `epparser.d`.

The parser is able to produce the AST in two different formats:

> `parse_ep --text example.pas`

produces a text file `example.txt` (this is the default, `--text` is optional)
and

> `parse_ep --html example.pas`

produces an expandable tree using HTML5 in [`example.html`](https://cdn.rawgit.com/PhilippeSigaud/Pegged/ade2aa5d/pegged/examples/extended_pascal/example.html).
Please browse with a
browser that supports the
[&lt;details>](https://www.w3schools.com/tags/tag_details.asp) tag.
