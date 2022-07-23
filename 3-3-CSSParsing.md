# [CSS Parsing](https://www.w3.org/TR/CSS2/grammar.html)
Unlike HTML, CSS is **a context free grammar** and can be parsed using the conventional parsers. 

## Grammar Definition
In fact [the CSS specification](https://www.w3.org/TR/CSS2/grammar.html) defines CSS lexical and syntax grammar.

### Vocabulary
The lexical grammar is defined by regular expressions for each token:
```
comment     \/\*[^*]*\*+([^/*][^*]*\*+)*\/
num         [0-9]+|[0-9]*"."[0-9]+
nonascii    [\200-\377]
nmstart     [_a-z]|{nonascii}|{escape}
nmchar      [_a-z0-9-]|{nonascii}|{escape}
name        {nmchar}+
ident       {nmstart}{nmchar}*
```

### Syntax
The syntax grammar is described in BNF.
```
ruleset
  : selector [ ',' S* selector ]*
    '{' S* declaration [ ';' S* declaration ]* '}' S*
  ;
selector
  : simple_selector [ combinator selector | S+ [ combinator? selector ]? ]?
  ;
simple_selector
  : element_name [ HASH | class | attrib | pseudo ]*
  | [ HASH | class | attrib | pseudo ]+
  ;
class
  : '.' IDENT
  ;
element_name
  : IDENT | '*'
  ;
attrib
  : '[' S* IDENT S* [ [ '=' | INCLUDES | DASHMATCH ] S*
    [ IDENT | STRING ] S* ] ']'
  ;
pseudo
  : ':' [ IDENT | FUNCTION S* [IDENT S*] ')' ]
  ;
```

### For Example
This is a ruleset is this structure:
```CSS
div.error, a.error {
  color:red;
  font-weight:bold;
}
```
- selectors   
    div.error and a.error are selectors
- ruleset  
    The part inside the curly braces contains the rules that are applied by this ruleset

This structure is defined formally in this definition:
```
ruleset
  : selector [ ',' S* selector ]*
    '{' S* declaration [ ';' S* declaration ]* '}' S*
  ;
```
This means a ruleset:
- It is a selector or optionally a number of "selectors" separated by a comma and spaces (S stands for white space). 
- Contains curly braces and inside them a "declaration" or optionally a number of declarations separated by a semicolon. 
- "declaration" and "selector" will be defined in the following BNF definitions.


## WebKit CSS Parser
WebKit uses Flex and Bison parser generators to create parsers automatically from the CSS grammar files. Bison creates a bottom up shift-reduce parser. Firefox uses a top down parser written manually. 

In both cases each CSS file is parsed into a StyleSheet object. Each object contains CSS rules. The CSS rule objects contain selector and declaration objects and other objects corresponding to CSS grammar.

![Figure : parsing CSS](/images/11.png)