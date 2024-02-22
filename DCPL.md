## S-Expressions
Symbolic expressions are formed using a notation for representing trees by parenthesized linear text strings.

The leaves of the tree are symbolic tokens, where a symbolic token is any sequence of characters that does not contain a left parenthesis, a right parenthesis, or a whitespace character.

    ((this is ) an ((example) (s-expression tree)))

An s-expression grammar combines the domain notation with s-expressions to specify the syntactic structure of a language.
1. List of syntactic domains (one for each kind of phrase).
2. Set of production rules that define the structure of compound phrases.
A syntactic domain is a collection of program phrases. Primitive syntactic domains are collections of phrases with no substructure.

Compound syntactic domains are collections of phrases built out of other phrases. Because compound syntactic domains are defined by a grammar’s production rules, the list of syntactic domains does not explicitly indicate their structure. All syntactic domains are annotated with domain variables that range over their elements.

The production rules specify the structure of compound domains. There is one rule for each compound domain. A production rule has the form:

    domain-variable ::=   pattern [phrase-type]
                        | pattern [phrase-type]
                        ...
                        | pattern [phrase-type]

Each line of the rule is called a production; it specifies a collection of phrases that are considered to belong to the compound syntactic domain being defined.

S-expression grammars are specialized versions of context-free grammars, the standard way to define programming language syntax. Domain variables play the role of nonterminals in such grammars. Our grammars are context-free because each production specifies the expansion of a single nonterminal in a way that does not depend on the context in which that nonterminal appears.

