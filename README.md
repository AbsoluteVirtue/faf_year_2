# DSL 2024
## TO READ
1. Robert Nystrom, **Crafting interpreters** -- https://craftinginterpreters.com/introduction.html
2. Ian D. Allen, **Recursive descent parsing** -- http://teaching.idallen.com/cst8152/98w/recursive_decent_parsing.html
3. Jack W. Crenshaw, **Let's build a compiler!** -- https://compilers.iecc.com/crenshaw/tutor1.txt
4. Alfred Aho et al., **Compilers, principles, techniques, and tools** (Aho, Sethi, Ullman) -- https://ia902208.us.archive.org/24/items/aho-compilers-principles-techniques-and-tools-2e_202203/Aho%20-%20Compilers%20-%20Principles%2C%20Techniques%2C%20and%20Tools%202e.pdf
5. Des Watson, **A practical approach to compiler construction** --
6. Franklyn Turbak, David Gifford, **Design concepts in programming languages** --
### Extra
1. Robert Nystrom, **Game programming patterns** -- http://gameprogrammingpatterns.com/contents.html
2. Evan Ovadia, **Vale programming language** -- https://verdagon.dev/home
3. John Backus, **Can programming be liberated from the von Neumann style?** -- http://worrydream.com/refs/Backus-CanProgrammingBeLiberated.pdf
### ANTLR4
1. Terrence Parr, **The Definitive ANTLR 4 Reference**,
https://dl.icdst.org/pdfs/files3/a91ace57a8c4c8cdd9f1663e1051bf93.pdf
2. ANTLR 4 Documentation,
https://github.com/antlr/antlr4/blob/master/doc/index.md
3. Federico Tomasetti, **The ANTLR Mega-Tutorial**,
https://tomassetti.me/antlr-mega-tutorial/ 
4. Federico Tomasetti, **How to create pragmatic lightweight languages** (incomplete),
https://pdfcoffee.com/create-languages-pdf-free.html
5. "How to create" code examples,
https://github.com/JohnMcGuinness/CreateLanguages
### Getting started
Step 1. Download Java Runtime Environment, https://www.java.com/en/download/manual.jsp

Step 2. Set the CLASSPATH environment variable, for example:
        (in Linux CLI) export CLASSPATH=".:/Downloads/antlr-4.13.1-complete.jar:$CLASSPATH"

Step 3. Launch the tool in Java, for example:
        (in Linux CLI) java org.antlr.v4.Tool
        or
        java -jar /Downloads/antlr-4.0-complete.jar
- this will list all available optional parameters

Step 4. Create a grammar definition in a .g4 file. Invoke ANTLR with this file to generate your lexer, parser and listener interfaces:
        (in Linux CLI) java org.antlr.v4.Tool Grammar.g4
- where "Grammar" is the name of your grammar and of your file

This step requires compiled *.class files, if you're generating Java with JDK.

For an alternative target platform use the -Dlanguage option during generation:
        (in Linux CLI) java org.antlr.v4.Tool Grammar.g4 -Dlanguage=Python3

To use visitor-classes instead of event listeners during AST traversal use the -visitor option:
        (in Linux CLI) java org.antlr.v4.Tool Grammar.g4 -visitor

Step 5. At this point you should have all of the files generated. For example, with a grammar called Grammar you should see at least:
- GrammarLexer(.py), GrammarListener(.py), GrammarParser(.py), GrammarListener(.py) and corresponding *.tokens and *.interp files

Optional step. If you're using Java, you can now visualize your tree etc.:
        (in Linux CLI) java org.antlr.v4.gui.TestRig Grammar r -gui
- where "Grammar" is your grammar's name and "r" is your primary rule

You can check for syntax errors in your source files using the tree diagram, alternatively:
        (in Linux CLI) java org.antlr.v4.gui.TestRig Grammar r -tokens
- this will produce the lexer's output for your source file.

For accessing the "test rig" in other target languages use that particular language's ANTLR runtime.

Step 6. Create a "driver" file that will instantiate the lexer, parser and tree-walker. For example:
        (Python 3) lexer = LabeledExprLexer(input_stream)
                   token_stream = CommonTokenStream(lexer)
                   parser = LabeledExprParser(token_stream)
                   tree = parser.r()
- here, "r" is the generated method for your primary rule. If the rule is named differently, use that name to call the method.

You can now initiate tree traversal using either the default walker:
                   listener = GrammarListener()
                   ParseTreeWalker().walk(listener, tree)
or using the visitor:
                   GrammarVisitor().visit(tree)

For more Python 3 examples see: https://github.com/jszheng/py3antlr4book/tree/master

Step 7. At this point you should fill out the listener/visitor methods with logic specific to your task.
