# The Definitive ANTLR 4 Reference by Terence Parr
https://www.youtube.com/watch?v=q8p1voEiu8Q
"Why program by hand in 5 days what you can spend 5 years automating."
- Visitor pattern to walk syntax trees
## Ch. 1
### Installing
https://www.stringtemplate.org/about.html
ANTLR converts grammars into programs that recognize sentences in the language described by the grammar. For example, given a grammar for JSON, the ANTLR tool generates a program that recognizes JSON input using some support classes from the ANTLR runtime library.

    export CLASSPATH=".:/usr/local/lib/antlr-4.0-complete.jar:$CLASSPATH"

then

    java -jar /usr/local/lib/antlr-4.0-complete.jar

or

    java org.antlr.v4.Tool

or

    alias antlr4='java -jar /usr/local/lib/antlr-4.0-complete.jar'

or as a bash script

    #!/bin/sh
    java -cp "/usr/local/lib/antlr4-complete.jar:$CLASSPATH" org.antlr.v4.Tool $*

### Testing
Hello.g4 file:

    grammar Hello;             // Define a grammar called Hello
    r : 'hello' ID ;           // match keyword hello followed by an identifier
    ID : [a-z]+ ;              // match lower-case identifiers
    WS : [ \t\r\n]+ -> skip ;  // skip spaces, tabs, newlines, \r (Windows)

then generate parser and lexer using antlr4 alias from before

    antlr4 Hello.g4

then compile 

    javac *.java

Running the ANTLR tool on Hello.g4 generates an executable recognizer embodied by HelloParser.java and HelloLexer.java.
ANTLR provides a flexible testing tool in the runtime library called TestRig. It can display lots of information about how a recognizer matches input from a file or standard input. TestRig uses Java reflection to invoke compiled recognizers.

    alias grun='java org.antlr.v4.runtime.misc.TestRig'

then start the TestRig on grammar Hello at rule r

    grun Hello r -tokens

then print the parse tree in LISP-style text form (root children)

    grun Hello r -tree

or

    grun Hello r -gui

or generate a visual representation of the parse tree in PostScript and store it in file.ps

    grun Hello r -ps file.ps

## Ch. 2
### Implementation
Recursive-descent parsers are really just a collection of recursive methods, one per rule. The descent term refers to the fact that parsing begins at the root of a parse tree and proceeds toward the leaves (tokens). The rule we invoke first, the start symbol, becomes the root of the parse tree.

    // assign : ID '=' expr ';' ;
    void assign() { // method generated from rule assign
        match(ID);  // compare ID to current input symbol then consume
        match('=');
        expr();     // match an expression by calling expr()
        match(';');
    }

    void stat() {
        switch ( «current input token» ) {
            CASE ID : assign(); break;
            CASE IF : ifstat(); break; // IF is token type for keyword 'if'
            CASE WHILE : whilestat(); break;
            ...
            default : «raise no viable alternative exception»
        }
    }

### Ambiguous statements
Below are included a few ambiguous grammars to make the notion of ambiguity more concrete.

    stat: ID '=' expr ';' // match an assignment; can match "f();"
        | ID '=' expr ';' // oops! an exact duplicate of previous alternative
        ;
    expr: INT ;

or

    stat: expr ';' // expression statement
        | ID '(' ')' ';' // function call statement
        ;
    expr: ID '(' ')'
        | INT
        ;
    
ANTLR resolves the ambiguity by choosing the first alternative involved in the decision. Ambiguities can occur in the lexer as well as the parser, but ANTLR resolves them so the rules behave naturally. ANTLR resolves lexical ambiguities by matching the input string to the rule specified first in the grammar. To see how this works, let’s look at an ambiguity that’s common to most programming languages: the ambiguity between keywords and identifier rules.

    BEGIN : 'begin' ; // match b-e-g-i-n sequence; ambiguity resolves to BEGIN
    ID : [a-z]+ ; // match one or more of any lowercase letter

### Parse Tree Visitor
There are situations, however, where we want to control the walk itself, explicitly calling methods to visit children. Option -visitor asks ANTLR to generate a visitor interface from a grammar with a visit method per rule.

To initiate a walk of the tree, our application-specific code would create a visitor implementation and call visit().

    ParseTree tree = ... ; // tree is result of parsing
    MyVisitor v = new MyVisitor();
    v.visit(tree);

ANTLR’s visitor support code would then call visitStat() upon seeing the root node. From there, the visitStat() implementation would call visit() with the children as arguments to continue the walk. Or, visitMethod() could explicitly call visitAssign(), and so on.
## Ch. 3
### Quick start
There are two key ANTLR components: the ANTLR tool itself and the ANTLR runtime (parse-time) API. When we say “run ANTLR on a grammar,” we’re talking about running the ANTLR tool, class org.antlr.v4.Tool. Running ANTLR generates code (a parser and a lexer) that recognizes sentences in the language described by the grammar. A lexer breaks up an input stream of characters into tokens and passes them to a parser that checks the syntax. The runtime is a library of classes and methods needed by that generated code such as Parser, Lexer, and Token. First we run ANTLR on a grammar and then compile the generated code against the runtime classes in the jar. Ultimately, the compiled application runs in conjunction with the runtime classes.

The first step to building a language application is to create a grammar that describes a language’s syntactic rules (the set of valid sentences). Grammars always start with a grammar header. This grammar is called ArrayInit and must match the filename: ArrayInit.g4:

    grammar ArrayInit;

A rule called init that matches comma-separated values between {...}:

    init : '{' value (',' value)* '}' ;

A value can be either a nested array/struct or a simple integer (INT):

    value   : init
            | INT
            ;

Parser rules start with lowercase letters, lexer rules with uppercase:

    INT : [0-9]+ ;
    WS : [ \t\r\n]+ -> skip ; // Define whitespace rule, toss it out

Then, we can run ANTLR (the tool) on the grammar file:

    antlr4 ArrayInit.g4 # Generate parser and lexer using antlr4 alias

### Testing generated parser
Once we’ve run ANTLR on our grammar, we need to compile the generated Java source code:

    javac *.java

(If you get a ClassNotFoundException error from the compiler, that means you probably haven’t set the Java CLASSPATH correctly.)

    grun ArrayInit init -tokens
    {99, 3, 451}
    CTRL + D

produces

    [@0,0:0='{',<1>,1:0]
    [@1,1:2='99',<4>,1:1]
    [@2,3:3=',',<2>,1:3]
    [@3,5:5='3',<4>,1:5]
    [@4,6:6=',',<2>,1:6]
    [@5,8:10='451',<4>,1:8]
    [@6,11:11='}',<3>,1:11]
    [@7,13:12='<EOF>',<-1>,2:0]

Each line of the output represents a single token and shows everything we know about the token. For example, [@5,8:10='451',<4>,1:8] indicates that it’s the token at index 5 (indexed from 0), goes from character position 8 to 10 (inclusive starting from 0), has text 451, has token type 4 (INT), is on line 1 (from 1), and is at character position 8 (starting from zero and counting tabs as a single character).

We can ask for the parse tree with the -tree option (grun ArrayInit init -tree).

    (init { (value 99) , (value 3) , (value 451) })

### Integrating generated parser into Java
Let's look at a simple Java main() that invokes our initializer parser and prints out the parse tree like TestRig’s -tree option:

    import org.antlr.v4.runtime.*;
    import org.antlr.v4.runtime.tree.*;

    public class Test {
        public static void main(String[] args) throws Exception {
            // create a CharStream that reads from standard input
            ANTLRInputStream input = new ANTLRInputStream(System.in);
            // create a lexer that feeds off of input CharStream
            ArrayInitLexer lexer = new ArrayInitLexer(input);
            // create a buffer of tokens pulled from the lexer
            CommonTokenStream tokens = new CommonTokenStream(lexer);
            // create a parser that feeds off the tokens buffer
            ArrayInitParser parser = new ArrayInitParser(tokens);
            ParseTree tree = parser.init(); // begin parsing at init rule
            System.out.println(tree.toStringTree(parser)); // print LISP-style tree
        }
    }

Here’s how to compile everything and run Test:

    javac ArrayInit*.java Test.java
    java Test
    {1,{2,3},4}
    CTRL + D

produces

    (init { (value 1) , (value (init { (value 2) , (value 3) })) , (value 4) })

### Building a translator that converts short array initializers to String objects
The task is to translate Java short arrays like {99 , 3 , 451 } to "\u0063\u0003\u01c3" where 63 is the hexadecimal representation of the 99 decimal.

To move beyond recognition, an application has to extract data from the parse tree. The easiest way to do that is to have ANTLR’s built-in parse-tree walker trigger a bunch of callbacks as it performs a depth-first walk. To write a program that reacts to the input, all we have to do is implement a few methods in a subclass of ArrayInitBaseListener. The basic strategy is to have each listener method print out a translated piece of the input when called to do so by the tree walker.

Listener gets notified at the beginning and end of phrases associated with rules in the grammar. 

Starting a translation project means figuring out how to convert each input token or phrase to an output string. To code the translator, we need to write methods that print out the converted strings upon seeing the appropriate input token or phrase. The built-in tree walker triggers callbacks in a listener upon seeing the beginning and end of the various phrases.

Convert short array inits like {1,2,3} to "\u0001\u0002\u0003":

    public class ShortToUnicodeString extends ArrayInitBaseListener {
        /** Translate { to " */
        @Override
        public void enterInit(ArrayInitParser.InitContext ctx) {
            System.out.print('"');
        }
        /** Translate } to " */
        @Override
        public void exitInit(ArrayInitParser.InitContext ctx) {
            System.out.print('"');
        }

        /** Translate integers to 4-digit hexadecimal strings prefixed with \\u */
        @Override
        public void enterValue(ArrayInitParser.ValueContext ctx) {
        // Assumes no nested array initializers
            int value = Integer.valueOf(ctx.INT().getText());
            System.out.printf("\\u%04x", value);
        }
    }

We don’t need to override every enter/exit method; we do just the ones we care about. The only unfamiliar expression is ctx.INT(), which asks the context object for the integer INT token matched by that invocation of rule value. Context objects record everything that happens during the recognition of a rule.

    import org.antlr.v4.runtime.*;
    import org.antlr.v4.runtime.tree.*;

    public class Translate {
        public static void main(String[] args) throws Exception {
            // create a CharStream that reads from standard input
            ANTLRInputStream input = new ANTLRInputStream(System.in);
            // create a lexer that feeds off of input CharStream
            ArrayInitLexer lexer = new ArrayInitLexer(input);
            // create a buffer of tokens pulled from the lexer
            CommonTokenStream tokens = new CommonTokenStream(lexer);
            // create a parser that feeds off the tokens buffer
            ArrayInitParser parser = new ArrayInitParser(tokens);
            ParseTree tree = parser.init(); // begin parsing at init rule
    ➤      // Create a generic parse tree walker that can trigger callbacks
    ➤      ParseTreeWalker walker = new ParseTreeWalker();
    ➤      // Walk the tree created during the parse, trigger callbacks
    ➤      walker.walk(new ShortToUnicodeString(), tree);
    ➤      System.out.println(); // print a \n after translation
        }
    }

then

    javac ArrayInit*.java Translate.java
    java Translate
    {99, 3, 451}
    CTRL + D

produces 

    "\u0063\u0003\u01c3"

## Ch. 4
### Tour of ANTLR
For our first grammar, we’re going to build a simple calculator. Doing something with expressions makes sense because they’re so common. To keep things simple, we’ll allow only the basic arithmetic operators (add, subtract, multiply, and divide), parenthesized expressions, integer numbers, and variables. We’ll also restrict ourselves to integers instead of allowing floating-point numbers.

A program in our expression language is a sequence of statements terminated by newlines. A statement is either an expression, an assignment, or a blank line.

    grammar Expr;   

    prog: stat+ ;
    stat: expr NEWLINE
        | ID '=' expr NEWLINE
        | NEWLINE
        ;

    expr: expr ('*'|'/') expr
        | expr ('+'|'-') expr
        | INT
        | ID
        | '(' expr ')'
        ;
    
    ID : [a-zA-Z]+ ;
    INT : [0-9]+ ; 
    NEWLINE : '\r'? '\n' ; // return newlines to parser (is end-statement signal)
    WS : [ \t]+ -> skip ;  // toss out whitespace

- Grammars consist of a set of rules that describe language syntax. There are rules for syntactic structure like stat and expr as well as rules for vocabulary symbols (tokens) such as identifiers and integers.
- Rules starting with a lowercase letter comprise the parser rules.
- Rules starting with an uppercase letter comprise the lexical (token) rules.
- We separate the alternatives of a rule with the | operator, and we can group symbols with parentheses into subrules. For example, subrule ('*'|'/') matches either a multiplication symbol or a division symbol.

One of ANTLR v4’s most significant new features is its ability to handle (most kinds of) left-recursive rules. A left-recursive rule is one that invokes itself at the start of an alternative. For example, in this grammar, rule expr has alternatives on lines 11 and 12 that recursively invoke expr on the left edge. Specifying arithmetic expression notation this way is dramatically easier than what we’d need for the typical top-down parser strategy. In that strategy, we’d need multiple rules, one for each operator precedence level.

The easiest way to test grammars is with the built-in TestRig, which we can access using alias grun.

    antlr4 Expr.g4
    javac Expr*.java
    grun Expr prog -gui t.expr

It’s OK to develop and test grammars using the test rig, but ultimately we’ll need to integrate our ANTLR-generated parser into an application.

    import org.antlr.v4.runtime.*;
    import org.antlr.v4.runtime.tree.*;
    import java.io.FileInputStream;
    import java.io.InputStream;

    public class ExprJoyRide {
        public static void main(String[] args) throws Exception {
            String inputFile = null;
            if ( args.length>0 ) inputFile = args[0];
            InputStream is = System.in;
            if ( inputFile!=null ) is = new FileInputStream(inputFile);
            ANTLRInputStream input = new ANTLRInputStream(is);
            ExprLexer lexer = new ExprLexer(input);
            CommonTokenStream tokens = new CommonTokenStream(lexer);
            ExprParser parser = new ExprParser(tokens);
            ParseTree tree = parser.prog(); // parse; start at prog
            System.out.println(tree.toStringTree(parser)); // print tree as text
        }
    }

then

    javac ExprJoyRide.java Expr*.java
    java ExprJoyRide t.expr

produces

    (prog
        (stat (expr 193) \n)
        (stat a = (expr 5) \n)
        (stat b = (expr 6) \n)
        (stat (expr (expr a) + (expr (expr b) * (expr 2))) \n)
        (stat (expr (expr ( (expr (expr 1) + (expr 2)) )) * (expr 3)) \n)
    )

### Importing Grammars
It’s a good idea to break up very large grammars into logical chunks, just like we do with software. One way to do that is to split a grammar into parser and lexer grammars. Factoring out lexical rules into a “module” means we can use it for different parser grammars. Here’s a lexer grammar containing all of the lexical rules:

    lexer grammar CommonLexerRules; // note "lexer grammar"
    ID : [a-zA-Z]+ ;                // match identifiers
    INT : [0-9]+ ;                  // match integers
    NEWLINE: '\r'? '\n' ;           // return newlines to parser (end-statement signal)
    WS : [ \t]+ -> skip ;           // toss out whitespace

then

    grammar LibExpr; // Rename to distinguish from original
    import CommonLexerRules; // includes all rules from CommonLexerRules.g4

    prog: stat+ ;

    stat: expr NEWLINE
        | ID '=' expr NEWLINE
        | NEWLINE
        ;

    expr: expr ('*'|'/') expr
        | expr ('+'|'-') expr
        | INT
        | ID
        | '(' expr ')'
        ;

### Error handling
When using the -gui option on grun, the parse-tree dialog automatically highlights error nodes in red. Notice that ANTLR successively recovered from the error in the first expression again to properly match the second.

### Building a calculator using Visitor
ANTLR v4 encourages us to keep grammars clean and use parse-tree visitors and other walkers to implement language applications. In this section, we’ll use the well-known visitor pattern to implement our little calculator. To make things easier for us, ANTLR automatically generates a visitor interface and blank visitor implementation object.

First, we need to label the alternatives of the rules. (The labels can be any identifier that doesn’t collide with a rule name.) Without labels on the alternatives, ANTLR generates only one visitor method per rule.

    stat: expr NEWLINE              # printExpr
        | ID '=' expr NEWLINE       # assign
        | NEWLINE                   # blank
        ;

    expr: expr op=('*'|'/') expr    # MulDiv
        | expr op=('+'|'-') expr    # AddSub
        | INT                       # int
        | ID                        # id
        | '(' expr ')'              # parens
        ;

Next, let’s define some token names for the operator literals so that, later, we can reference token names as Java constants in the visitor.

    MUL : '*' ; // assigns token name to '*' used above in grammar
    DIV : '/' ;
    ADD : '+' ;
    SUB : '-' ;

Next, we create lexer and parser objects derived from grammar LabeledExpr, not Expr.

    LabeledExprLexer lexer = new LabeledExprLexer(input);
    CommonTokenStream tokens = new CommonTokenStream(lexer);
    LabeledExprParser parser = new LabeledExprParser(tokens);
    ParseTree tree = parser.prog(); // parse

We create an instance of our visitor class, EvalVisitor, which we’ll get to in just a second. To start walking the parse tree returned from method prog(), we call visit().

    EvalVisitor eval = new EvalVisitor();
    eval.visit(tree);

then

    antlr4 -no-listener -visitor LabeledExpr.g4

ANTLR generates a visitor interface with a method for each labeled alternative name.

    public interface LabeledExprVisitor<T> {
        T visitId(LabeledExprParser.IdContext ctx);         # from label id
        T visitAssign(LabeledExprParser.AssignContext ctx); # from label assign
        T visitMulDiv(LabeledExprParser.MulDivContext ctx); # from label MulDiv
        ...
    }

Next, ANTLR generates a default visitor implementation called LabeledExprBaseVisitor that we can subclass. In this case, our expression results are integers and so our EvalVisitor should extend LabeledExprBaseVisitor<Integer>. To implement the calculator, we override the methods associated with statement and expression alternatives.

import java.util.HashMap;
import java.util.Map;

    public class EvalVisitor extends LabeledExprBaseVisitor<Integer> {
        /** "memory" for our calculator; variable/value pairs go here */
        Map<String, Integer> memory = new HashMap<String, Integer>();
        /** ID '=' expr NEWLINE */
        @Override
        public Integer visitAssign(LabeledExprParser.AssignContext ctx) {
            String id = ctx.ID().getText(); // id is left-hand side of '='
            int value = visit(ctx.expr()); // compute value of expression on right
            memory.put(id, value); // store it in our memory
            return value;
        }
        /** expr NEWLINE */
        @Override
        public Integer visitPrintExpr(LabeledExprParser.PrintExprContext ctx) {
            Integer value = visit(ctx.expr()); // evaluate the expr child
            System.out.println(value); // print the result
            return 0; // return dummy value
        }
        /** INT */
        @Override
        public Integer visitInt(LabeledExprParser.IntContext ctx) {
            return Integer.valueOf(ctx.INT().getText());
        }
        /** ID */
        @Override
        public Integer visitId(LabeledExprParser.IdContext ctx) {
            String id = ctx.ID().getText();
            if ( memory.containsKey(id) ) return memory.get(id);
            return 0;
        }
        /** expr op=('*'|'/') expr */
        @Override
        public Integer visitMulDiv(LabeledExprParser.MulDivContext ctx) {
            int left = visit(ctx.expr(0)); // get value of left subexpression
            int right = visit(ctx.expr(1)); // get value of right subexpression
            if ( ctx.op.getType() == LabeledExprParser.MUL ) return left * right;
            return left / right; // must be DIV
        }
        /** expr op=('+'|'-') expr */
        @Override
        public Integer visitAddSub(LabeledExprParser.AddSubContext ctx) {
            int left = visit(ctx.expr(0)); // get value of left subexpression
            int right = visit(ctx.expr(1)); // get value of right subexpression
            if ( ctx.op.getType() == LabeledExprParser.ADD ) return left + right;
            return left - right; // must be SUB
        }
        /** '(' expr ')' */
        @Override
        public Integer visitParens(LabeledExprParser.ParensContext ctx) {
            return visit(ctx.expr()); // return child expr's value
        }
    }

then, with obligatory -visitor flag:

    antlr4 -no-listener -visitor LabeledExpr.g4
    javac Calc.java LabeledExpr*.java
    cat t.expr
    java Calc t.expr

Let’s switch gears now and think about translation rather than evaluating or interpreting input. In the next section, we’re going to use a variation of the visitor called a listener to build a translator for Java source code.

### Building Translator using Listener
For example, we’d like to read in Java code like this:

    import java.util.List;
    import java.util.Map;

    public class Demo {
        void f(int x, String y) { }
        int[ ] g(/*no args*/) { return null; }
        List<Map<String, Integer>>[] h() { return null; }
    }

and generate an interface with the method signatures, preserving the whitespace and comments.

    interface IDemo {
        void f(int x, String y);
        int[ ] g(/*no args*/);
        List<Map<String, Integer>>[] h();
    }

This can be solved by listening to “events” fired from a Java parse-tree walker. The key “interface” between the grammar and our listener object is called JavaListener, and ANTLR automatically generates it for us. It defines all of the methods that class ParseTreeWalker from ANTLR’s runtime can trigger as it traverses the parse tree. In our case, we need to respond to three events by overriding three methods: when the walker enters and exits a class definition and when it encounters a method definition.

    public interface JavaListener extends ParseTreeListener {
        void enterClassDeclaration(JavaParser.ClassDeclarationContext ctx);
        void exitClassDeclaration(JavaParser.ClassDeclarationContext ctx);
        void enterMethodDeclaration(JavaParser.MethodDeclarationContext ctx);
    }

The biggest difference between the listener and visitor mechanisms is that listener methods are called by the ANTLR-provided walker object, whereas visitor methods must walk their children with explicit visit calls. Forgetting to invoke visit() on a node’s children means those subtrees don’t get visited. To build our listener implementation, we need to know what rules classDeclarationand methodDeclaration look like because listener methods have to grab phrase elements matched by the rules.

    classDeclaration
        :   'class' Identifier typeParameters? ('extends' type)?
            ('implements' typeList)?
            classBody
        ;

    methodDeclaration
        : type Identifier formalParameters ('[' ']')* methodDeclarationRest
        | 'void' Identifier formalParameters methodDeclarationRest
        ;

Our basic strategy will be to print out the interface header when we see the start of a class definition. Then, we’ll print a terminating } at the end of the class definition. Upon each method definition, we’ll spit out its signature.

    import org.antlr.v4.runtime.TokenStream;
    import org.antlr.v4.runtime.misc.Interval;

    public class ExtractInterfaceListener extends JavaBaseListener {
        JavaParser parser;
        public ExtractInterfaceListener(JavaParser parser) {this.parser = parser;}
        @Override
        public void enterClassDeclaration(JavaParser.ClassDeclarationContext ctx){
            System.out.println("interface I"+ctx.Identifier()+" {");
        }
        @Override
        public void exitClassDeclaration(JavaParser.ClassDeclarationContext ctx) {
            System.out.println("}");
        }
        @Override
        public void enterMethodDeclaration(
            JavaParser.MethodDeclarationContext ctx
        )
        {
            // need parser to get tokens
            TokenStream tokens = parser.getTokenStream();
            String type = "void";
            if ( ctx.type()!=null ) {
                type = tokens.getText(ctx.type());
            }
            String args = tokens.getText(ctx.formalParameters());
            System.out.println("\t"+type+" "+ctx.Identifier()+args+";");
        }
    }

To fire this up, we need a main program:

    JavaLexer lexer = new JavaLexer(input);
    CommonTokenStream tokens = new CommonTokenStream(lexer);
    JavaParser parser = new JavaParser(tokens);
    ParseTree tree = parser.compilationUnit(); // parse
    ParseTreeWalker walker = new ParseTreeWalker(); // create standard walker
    ExtractInterfaceListener extractor = new ExtractInterfaceListener(parser);
    walker.walk(extractor, tree); // initiate walk of tree with listener

The visitor and listener mechanisms work very well and promote the separation of concerns between parsing and parser application. Sometimes, though, we need extra control and flexibility.

### Changes during the parsing stage
For the ultimate flexibility and control, however, we can directly embed code snippets (actions) within grammars. These actions are copied into the recursive-descent parser code ANTLR generates. We can compute values or print things out on-the-fly during parsing if we don’t want the overhead of building a parse tree. On the other hand, it means embedding arbitrary code within the expression grammar, which is harder; we have to understand the effect of the actions on the parser and where to position those actions.

    parrt Terence Parr 101
    tombu Tom Burns 020
    bke Kevin Edgar 008

The columns are tab-delimited, and each row ends with a newline character. Matching this kind of input is pretty simple grammatically.

    file : (row NL)+ ; // NL is newline token: '\r'? '\n'
    row : STUFF+ ;

We need to create a constructor so that we can pass in the column number we want (counting from 1), and we need an action inside the (...)+ loop in rule row.

    grammar Rows;

    @parser::members { 
        int col;
        public RowsParser(TokenStream input, int col) { // custom constructor
            this(input);
            this.col = col;
        }
    }

    file: (row NL)+ ;

    row
    locals [int i=0]
        :   ( STUFF
                {
                    $i++;
                    if ( $i == col ) System.out.println($STUFF.text);
                }
            )+
        ;

    TAB : '\t' -> skip ;    // match but don't pass to the parser
    NL : '\r'? '\n' ;       // match and pass to the parser
    STUFF: ~[\t\r\n]+ ;     // match any chars except tab, newline

The STUFF lexical rule matches anything that’s not a tab or newline, which means we can have space characters in a column.

    RowsLexer lexer = new RowsLexer(input);
    CommonTokenStream tokens = new CommonTokenStream(lexer);
    int col = Integer.valueOf(args[0]);
    RowsParser parser = new RowsParser(tokens, col); // pass column number!
    parser.setBuildParseTree(false); // don't waste time bulding a tree
    parser.file(); // parse

### Semantic predicates
Let’s look at a grammar that reads in sequences of integers. The trick is that part of the input specifies how many integers to group together. We don’t know until runtime how many integers to match.

p. 48
