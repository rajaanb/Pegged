PEGGED
======

**Pegged** is a parsing expression grammar (PEG) generator implemented in the D programming language. 

The idea is to give the generator a [PEG](en.wikipedia.org/wiki/Parsing_expression_grammar), with the syntax presented in [the reference article ](http://bford.info/pub/lang/peg). From this grammar definition a set of related parsers will be created, to be used at runtime or compile time.

Usage
-----

To use **Pegged**, just call the `grammar` function with a PEG and mix it in. For example:

```d
import pegged.grammar;

mixin(grammar(" 
    Expr     <   Factor AddExpr*
    AddExpr  <   ('+'/'-') Factor
    Factor   <   Primary MulExpr*
    MulExpr  <   ('*'/'/') Primary
    Primary  <   Parens / Number / Variable / '-' Primary

    Parens   <   '(' Expr ')'
    Number   <~ [0-9]+
    Variable <- Identifier
"));
```

This creates the `Expr`, `AddExpr`, `Factor` (and so on) parsers for basic arithmetic expressions with operator precedence ('*' and '/' bind stronger than '+' or '-'). `Identifier` is a pre-defined parser recognizing your basic C-style identifier (first a letter or underscore, then digits, letters or underscores). In the rest of this document, I'll call 'rule' a `Parser <- Parsing Expression` expression and I'll use 'grammar' to designate the entire group of rules given to `grammar`.

To use a parser, use the `.parse` method. It will return a parse tree containing the calls to the different rules:

```d
// Parsing at compile-time:
enum parseTree1 = Expr.parse("1 + 2 - (3*x-5)*6");

pragma(msg, parseTree1.capture);
assert(parseTree1.capture == ["1", "+", "2", "-", "(", "3", "*", "x", "-", "5", ")", "*", "6"]);
writeln(parseTree1);

// And at runtime too:
auto parseTree2 = Expr.parse(" 0 + 123 - 456 ");
assert(parseTree2.capture == ["0", "+", "123", "-", "456"]);
```

Even for such a simple grammar and such a simple expression, the resulting parse tree is a bit long to be shown here. See [the result here](https://github.com/PhilippeSigaud/Pegged/wiki/Parse-Result)

By default, the grammars do not silently consume spaces, as this is the standard behavior for PEGs. There is an opt-out though, with the simple `<` arrow instead of `<-` (you can see it in the previous example).

Tutorial and docs
-----------------

The **Pegged** wiki is [here](https://github.com/PhilippeSigaud/Pegged/wiki/). It contains a growing [tutorial](https://github.com/PhilippeSigaud/Pegged/wiki/Pegged-Tutorial). All the wiki pages are also present (as Markdown files) in the `docs` directory.


Features
--------

* The complete set of operators described [here](http://en.wikipedia.org/wiki/Parsing_expression_grammar) are implemented, with the 'traditional' PEG syntax. See [Peg Basics](https://github.com/PhilippeSigaud/Pegged/wiki/PEG-Basics).
* **Pegged** can parse its input at compile time and generate a complete parse tree at compile time. In a word: compile-time string (read: D code) transformation and generation. See [Generating Code](https://github.com/PhilippeSigaud/Pegged/wiki/Generating-Code) for example.
* You can parse at runtime also, you lucky you. ([Using the Parse Tree](https://github.com/PhilippeSigaud/Pegged/wiki/Using-the-Parse-Tree))
* Use a standard and readable PEG syntax as a DSL, not a bunch of templates that hide the parser in noise.
* But you can use expression templates if you want, as parsers are all available as such. **Pegged** is implemented as an expression template, and what's good for the library writer is sure OK for the user too. ([Behind the Curtain: How Pegged Works](https://github.com/PhilippeSigaud/Pegged/wiki/Behind-the-Curtain%3A-How-Pegged-Works)
* Some useful additional operators are there too: a way to discard matches (thus dumping them from the parse tree), to push captures on a stack, to accept matches that are equal to another match: see [PEG Additions](https://github.com/PhilippeSigaud/Pegged/wiki/Extended-PEG-Syntax).
* Adding new parsers is easy. See [User-Defined Parsers](https://github.com/PhilippeSigaud/Pegged/wiki/User-Defined-Parsers) to see how to do that.
* Grammars are composable: you can put different `mixin(grammar(rules));` in a module and then grammars and rules can refer to one another. That way, you can have utility grammars providing their functionalities to other grammars. [Grammar Composition](https://github.com/PhilippeSigaud/Pegged/wiki/Grammar-Composition)
* That's why **Pegged** comes with some pre-defined grammars (JSON, C, XML, CSV, the PEG grammar itself, etc). See [Grammar Examples](https://github.com/PhilippeSigaud/Pegged/wiki/Grammar-Examples).
* Grammars can be dumped in a file to create a module. Use the `asModule(string moduleName, string gram)` function in `pegged.grammar` to do that. See [Grammars as Modules](https://github.com/PhilippeSigaud/Pegged/wiki/Grammars-as-D-Modules).

More advanced features, outside the standard PEG perimeter are there to bring more power in the mix:

* **Parameterized rules**: `"List(E, Sep) <- E (Sep E)*"` is possible. The previous rule defines a parameterized parser taking two other parsers (namely, `E` and `Sep`) to match a `Sep`-separated list of `E`'s. See [Parametrized Rules](https://github.com/PhilippeSigaud/Pegged/wiki/Parametrized-Rules) to see what's possible.
* **Named captures**: any parser can be named with the `=` operator. The parse tree generated by the parser (so, also its matches) is delivered to the user in the output. Other parsers in the grammar see the named captures too. See [Named Captures](https://github.com/PhilippeSigaud/Pegged/wiki/Named-Captures) for more explanations on this.
* **Semantic actions** can be added to any rule in a grammar. Once a rule has matched, its associated action is called on the rule output and passed as final result to other parsers further up the grammar. Do what you want to the parse tree. If the passed actions are delegates, they can access external variables. See [Semantic Actions](https://github.com/PhilippeSigaud/Pegged/wiki/Semantic-Actions).


Limitations
-----------

* No effort was done to accelerate the parsing or reduce the memory consumption. The main goal was to 1) use the PEG syntax 2) parse at compile-time. Packrat parsing could be done, maybe. **Pegged** does some basic optimization, though: one-element sequences and choices are replaced by their only element and sequences of sequences are flattened (the same is done for ordered choices).
* No left-factorization for now. But you can do recursive grammars, of course.
* Only works on strings (no wstrings, dstrings or ranges). That's on my [todo](https://github.com/PhilippeSigaud/Pegged/wiki/TODO) list. I'm not sure ranges will work OK at compile-time and I think I'll need forward ranges at a minimum (or else store the produced elements anyway).
* Error reporting is the same as for any in-my-own-free-time / just-one-guy parsing project (read: perfectible).
* I couldn't find a way to use anonymous function templates as actions (`o => o`). Too bad.
* The entire grammar is defined at compile-time: no user input for actions and such. That was a design decision from the very beginning.

Future features (aka, my todo list) 
-----------------------------------

See [todo](https://github.com/PhilippeSigaud/Pegged/wiki/TODO)

Long-Term Goals (the Right to Dream)
------------------------------------

* As a long-term goal, parsing structures, as presented in [OMeta](http://www.vpri.org/pdf/tr2007003_ometa.pdf). Yeah, I know, but I find that wonderfully interesting. Rules could match not only strings by any D type inner structure (matching a struct if it contains an `int` member named `foo` and a `bar` method, etc).
* Hence, a pattern-matcher. if you used Haskell or ML, you know what I'm talking about.
* As a longer-term goal: implementing the complete D grammar and see if that flies.
* As an even longer-term goal: macros in D. Think Lisp and [talking to God](http://xkcd.com/224/).

References
----------

Articles:

* [The base PEG article from Bryan Ford](http://bford.info/pub/lang/peg).
* [Packrat parsing](http://pdos.csail.mit.edu/~baford/packrat/icfp02/).
* [OMeta](http://www.vpri.org/pdf/tr2007003_ometa.pdf).

D Code:

* Hisayuki Mima's [CTPG](https://github.com/youkei/ctpg), very similar, also done in D. Have a look!
* Nick Sabalausky's [Goldie](http://www.dsource.org/projects/goldie).
* Benjamin Shropshire's [dparser](http://dsource.org/projects/scrapple/browser/trunk/dparser).
* Martin Nowak put these gists on the D newsgroup:
    - https://gist.github.com/1255439 - lexer generator
    - https://gist.github.com/1262321 - complete and fast D lexer

Other languages:

* [pegtl](http://code.google.com/p/pegtl/), the PEG Template Library, in C++.
* [chilon::parser](http://chilon.net/library.html) in C++ also.
* [metaparse](http://abel.web.elte.hu/mpllibs/metaparse/index.html), in C++, is able to parse at compile-time.
* [Parslet](http://kschiess.github.com/parslet/) in Ruby and [Treetop](http://treetop.rubyforge.org/), in Ruby also.

License
-------

**Pegged** is released with the Boost license (like most D projects). See [here](http://www.boost.org/LICENSE_1_0.txt) for more details.


Author: Philippe Sigaud.
