# A regular expression implementation for Motoko

Throughout the last few weeks, we wondered what an implementation
of regular expressions in Motoko might look like.

We want to give Motoko programmers and dapp users convenient
and consise means for expressing patterns in text, useful for defining
linear input formats for canisters or simply powering search bars in apps.

We quickly realized that some regexp engines sacrifice the small,
comprehensive size of regexps in exchange for greater language recognition
properties, and at that, ones which go beyond the theoretical basis of regular expressions.

Perl, for example, bases its entire syntax on such 'regexes', often
due to the lack of equivalent native constructs.

The developers of Perl have reconsidered, and for the 6th
edition of Perl (Raku) redesigned the language to instead
rely on a theoretically sound basis of contextful grammars in the style of PEG. [^1]

  [^1]: http://en.wikipedia.org/wiki/Raku_(programming_language)#Regular_expressions

We too want to turn the unfortunate history of regexp misuse around.

Even worse, naive implementations of regular expressions
can suffer tragic performance characteristics resulting in what's called
the [Regular Expression Denial of Service](en.wikipedia.org/wiki/ReDoS) (ReDos) attack,
which is strictly unacceptable on IC.

In contrast to classic versions of Perl, Motoko is a richly expressive language,
and already provides constructs for analysis of data structures, including text strings,
through its [pattern matching constructs](internetcomputer.org/docs/current/motoko/main/motoko)
<!-- put a more specific link here -->
While pattern matching is widely useful for discriminating against the values and types of
elements of composite data structures, it cannot tackle the relationships between them.
For example, a for-switch loop, often used in hand-written lexers, which might look like

```
for {
  switch(input[i]) {
    case ('"') { backup(); parseString() }
    case ('0' or '1' or '2' or ...) { backup(); parseNumber() }
  }
  i++
}
```

discards the intermediate lexing results -- we know what is to be done in the cases
but must nonetheless rewind so that a subroutine implementing the regular expression
sees its entire input, an unavoidable result of the subroutine structure of 'structured' programs.
Other solutions, such as 'prefix consumed' conventions or returning a function from every parsing function,
exist, but are much more complex and heavyweight and better suited to full-on language analysis
than the simple one-off cases we're after.

Regular expression compilers, such as lex, are able to mitigate such shortcomings
and produce readable, although not structured (using `goto`s), host language (Motoko, in our case)
output from plain regular expressions.
Lex is commonly used for parsing and language analysis, and most commonly deployed
with a full parser generator. This is not what our focus is at this time,
and for our purposes classic lex, as originally authored by Mike Lesk, alone does not suffice.

Ocamllex (used for Motoko's compiler), on the other hand, is a rich variant which extends the traditional
pattern-action syntax of lex with _named patterns_, in the style of SNOBOL.
This allows patterns to be composed cleanly from independent subparts, mitigating
the obscurity that large regular expressions tend to dwelve into.

What's more, some named patterns can be defined natively in Motoko and
shared as independent libraries, lifting the burden of implementing such trivia
in the core of our solution. That is, our lex wouldn't need to define such
constructs as POSIX's `[:alpha:]`, `[:ascii:]`, (or even worse, `\p{Greek}`!) and so on directly,
but would instead allow any module to define their own or make use of existing
packages. Classic PCRE-style regexps, on the other hand, do not allow any such extensions.
Thus, for example, `let alpha = func(x) { return x >= 'a' && x <= 'z' }` and
`let greek = func(x) { return unicode.isGreek(x) }` would be efficient replacements for the above.

Such a system could be integrated into the build system of Motoko programs
and produce independent modules to be imported by them.
Some consideration should also be put into keeping the regexps
close to their use -- the use of a precompiler such as lex would necessitate
putting regexps in a seperate file. Some languages (Go) have a convention of keeping
compiler directives in comments, which would allow regexps to be written in special
comments near their use, an example of such use being Go's "C" pseudopackage.
Such behavior would need to be part of the compiler, or implemented with lexical preprocessing
(as C's cpp, but cleaner).

Any implementation of regexps cannot directly apply to the syntax and semantics of Motoko itself
-- it was an early goal of the language to be kept simple and supplemented with a rich standard
library of more elaborate types and operations.

Javascript, per the ECMAScript standard, for example, defines regular expressions
as part of the language, both syntactically and semantically.
This was aided by the nature of Javascript's environment -- the plain text source
code for scripts is fetched from a web server and parsed in the browser.
Such a solution allows the regexp literals to be indentified along with host language constructs
and analyzed at the compilation stage, which wouldn't be the case has a simple function call been used.
Also, since javascript is weakly typed and dynamic, compiling regular expressions
seperately comes at a significant gain of performance over any native implementation.

Motoko, on the other hand, was from the beginning designed as a _backend_ language,
compiled into low-level bytecode in the canister before being deployed to IC.
That gives us a large field for code analysis and preprocessing, without the
payoffs of client-side scripting languages.

Such compile-time solutions are interesting future work considering
the nature of IC -- cycles are paid _per instruction_,
rather than _per month_ as with centralized cloud services.
Any such no-cost extensions would bring a great cost-expressiveness gain for Motoko programmers,
supporting the lightweight and expressive script-like style of the language.

---

Returning briefly to a dynamic, runtime implementation of regexps,
we have found that a native, Motoko-based implementation might suffice -
widely useful extended regular expressions can be parsed and compiled
to automata representations in as little as 2k lines of code (as with Golang's "regexp")
and the overhead is tolerable for the most common usage, when regexps are used in dapps for, e.g.,
search for posts in forums, looking through a filesystem, and any kind of search bar functionality.

In summary: a Motoko-native library for dynamically executing regexps is feasable and
would pose as a great showcase of Motoko's dynamics.
A lex-like preprocessor for supplementing the syntax and static semantics of canisters is
interesting considering the structure of the of IC, and should be undertaken next.

Any further developments on this case will be housed in this repository.


## See also

A seperate bounty concerned the adaptation of a Rust 'regexp' crate to
a stand-alone canister for use by Motoko programs.
It is currently fully functional and available at github.com/holykol/ic-regex
