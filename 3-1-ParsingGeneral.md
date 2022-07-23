# Parsing General
Parsing a document means translating it to a structure the code can use. 

The Result of Parsing  
Is usually a tree of nodes that represent the structure of the document. This is called a **parse tree** or a **syntax tree**.

## Grammars
Every format you can parse must have deterministic grammar consisting of **vocabulary** and **syntax rules**. It is called a context free grammar. 

### A Context Free Grammar 
We said that a language can be parsed by **regular parsers** if its grammar is a context free grammar, which consisting of: 
- Vocabulary
- Syntax / Syntax Rules

#### Formal Definitions
- Vocabulary - is usually expressed by **regular expressions**.
- Syntax - is usually defined in a format called **BNF**.
        
#### Parsing Example
Let's parse the mathematical expression 2 + 3 - 1.
- Vocabulary (regular expressions)
    1. Include integers, plus signs and minus signs.  
    ```
    INTEGER: 0|[1-9][0-9]*
    PLUS: +
    MINUS: -
    ```
- Syntax (BNF)
    1. The language syntax building blocks are expressions, terms and operations.  
    2. Can include any number of expressions.  
    3. An expression is defined as a "term" followed by an "operation" followed by another "term".  
    4. An operation is a plus token or a minus token.  
    5. A term is an integer token or an expression.  
    ```
    expression :=  term  operation  term
    operation :=  PLUS | MINUS
    term := INTEGER | expression
    ```

- Processes  
    1. The first match is an integer 2: according to rule #5 it is a term.   
    2. The second match is an expression 2 + 3(rule #3).  
    3. The next match will only be hit at the end of the input. 2 + 3 - 1 is an expression because we already know that 2 + 3 is a term(rule #5), so we have a term followed by an operation followed by another term(rule #3).   

### Conventional and Unconventional Parser
We can separate the parses to two types: 
- Conventional Parser (Regular Parser)  
    The language can be defined by a context free grammar that parsers need.
    - css
    - javascript
- Unconventional Parser  
    The language cannot be parsed easily by conventional parsers, since its grammar is not context free.
    - HTML

## Parsing - Parser and Lexer Combination
### Two Sub Processes and Two Components
Parsing can be separated into two sub processes:  
1. Lexical Analysis  
    - It is the process of breaking the input into tokens. 
    - Tokens are the language **vocabulary**: the collection of valid building blocks.
2. Syntax Analysis  
    - Applying of the language **syntax rules**. 

And there are the components of parser for **lexical analysis** & **syntax analysis**
1. The Lexer (Tokenizer)  
    - Create tokens.
    - Strip irrelevant characters like white spaces and line breaks.
2. The Parser 
    - Constructing the parse tree by analyzing the document structure according to the **syntax rules**. 

![Figure : from source document to parse trees](/images/5.png) 

### The Parsing Process
The parsing process is iterative. 
1. The parser will usually ask the lexer for a new token and try to match the token with one of the syntax rules. 
2. If a rule is matched, a node corresponding to the token will be added to the parse tree and the parser will ask for another token. 
3. If no rule matches, the parser will store the token internally, and keep asking for tokens until a rule matching all the internally stored tokens is found. 

### Syntax Errors  
If no rule is found then the parser will raise an exception. This means the document was not valid and contained syntax errors.


## Translation
In many cases the parse tree is not the final product. Parsing is often used in translation: transforming the input document to another format. 

An example is compilation. The compiler that compiles source code into machine code first parses it into a parse tree and then translates the tree into a machine code document.

![Figure : compilation flow](/images/6.png) 

## Generating Parsers Automatically
There are tools that can generate a parser. You feed them the grammar of your language
(vocabulary and syntax rules) and they generate a working parser. 

Creating a parser requires a deep understanding of parsing and it's not easy to create an optimized parser by hand, so parser generators can be very useful.

Ready available parsers:
- Flex
- Lex
- Yacc
- Bison

WebKit uses two parser generators: 
1. Flex 
    - For creating a lexer.
    - Flex's input is a file containing regular expression definitions of the tokens.
2. Bison 
    - For creating a parser.
    - Bison's input is the language syntax rules in BNF format.