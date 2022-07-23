# HTML Parser
The job of the HTML parser is to parse the HTML markup into a parse tree.

## Grammar Definition  
HTML definition is in a DTD format, the specifications of vocabulary and syntax in DTD are defined by the W3C organization, but it is not a context free grammar.

### Not A Context Free Grammar
- Soft Syntax  
The HTML is more **forgiving**: it lets you omit certain tags, or sometimes omit start or end tags, and so on.

This is the main reason why HTML is so popular: it forgives your mistakes and makes life easy for the web author. On the other hand, it makes it difficult to write a formal grammar. 

So to summarize, HTML cannot be parsed easily by conventional parsers, since its grammar is not context free.

## HTML DTD (Document Type Definition)
There is a formal format for defining HTML – DTD, the format contains definitions for all allowed elements, their attributes and hierarchy.

There are a few variations of the DTD. The strict mode conforms solely to the specifications but other modes contain support for markup used by browsers in the past. The purpose is backwards compatibility with older content.

## DOM (Document Object Model)
The output tree (the "parse tree") is a tree of DOM element and attribute nodes. DOM is the object presentation of the HTML document and the interface of HTML elements to the outside world like JavaScript. Like HTML, DOM is specified by the W3C organization.

The root of the tree is the "Document" object. The DOM has an almost one-to-one relation to the markup. 

For example:
```HTML
<html>
    <body>
        <p>
            Hello World
        </p>
        <div> 
            <img src="example.png"/>
        </div>
    </body>
</html>
```

This markup would be translated to the following DOM tree:

![Figure : DOM tree of the example markup](/images/7.png)

The tree contains DOM nodes means the tree is constructed of elements that implement one of the DOM interfaces. Browsers use concrete implementations that have other attributes used by the browser internally.
 
 
## [The Parsing Algorithm](https://html.spec.whatwg.org/multipage/parsing.html)
As we saw in the previous sections, HTML cannot be parsed using the regular parsers.
 
The reasons are:
1. The forgiving nature of the language.
2. The browsers have traditional error tolerance to support well known cases of invalid HTML.
3. The parsing process is reentrant. For other languages, the source doesn't change during parsing, but in HTML, dynamic code can add extra tokens, so the parsing process actually modifies the input.

Unable to use the regular parsing techniques, browsers create custom parsers for parsing HTML.
 
The algorithm consists of two stages: 
1. Tokenization 
2. Tree Construction
 
### Tokenization 
- **Tokenization** is the lexical analysis. 
- HTML **tokens** are start tags, end tags, attribute names and attribute values.
- The **tokenizer** recognizes the token, gives it to the tree constructor, and consumes the next character for recognizing the next token, and so on until the end of the input.

![Figure : HTML parsing flow (taken from HTML5 spec)](/images/8.png)

#### The Tokenization Algorithm
The algorithm's output is an HTML token. The algorithm is expressed as a state machine. Each state consumes one or more characters of the input stream and updates the next state according to those characters. 

The decision is influenced by the current tokenization state and by the tree construction state. This means the same consumed character will yield different results for the correct next state, depending on the current state. 

Let's see a simple example:
```HTML
<html>
    <body>
        Hello world
    </body>
</html>
```

1. &lt;html&gt;
    1. Data State
        - The initial state.
    2. Tag Open State
        - < character is encountered.
    3. Tag Name State
        - Each character is appended to the new token name.
        - In this case the created token is an html token.
    4. Emit new a token and back to Data State
        - The > tag is reached.
        - The html tags were emitted.
2. &lt;body&gt;
    1. Data State
    2. Tag Open State
        - < character is encountered.
    3. Tag Name State
        - The created token is a body token.    
    4. Emit new a token and back to Data State
        - The body tags were emitted.
3. Hello world
    1. Data State
    2. Emit new character tokens
        - Consuming the H character of Hello world will cause creation and emitting of a character token.
        - Emitting a character token for each character of Hello world.
        - This goes on until the < of </body> is reached.
4. &lt;/body&gt;
    1. Tag Open State
        - < character is encountered.
    2. Close Tag Open State
        - / character is encountered.
    3. Tag Name State
        - The created token is a body token.
    4. Emit new a token and back to Data State
        - The > tag is reached.
        - The new tag token will be emitted.
5. &lt;/html&gt;
    1. Data State
    2. Tag Open State
    3. Close Tag Open State
    4. Tag Name State
    5. Emit new a token and back to Data State


![Figure : Tokenizing the example input](/images/9.png)
 
### Tree Construction
#### Tree Construction Algorithm
When the parser is created the Document object is created. During the tree construction stage the root of DOM tree will be modified and elements will be added to it. Each node emitted by the tokenizer will be processed by the tree constructor. 

For each token the specification defines which DOM element is relevant to it and will be created for this token. The element is added to the DOM tree, and also the stack of open elements. This stack is used to correct nesting mismatches and unclosed tags. The algorithm is also described as a state machine. The states are called "insertion modes".

Let's see the tree construction process for the example input:
```HTML
<html>
    <body>
        Hello world
    </body>
</html>
```

The input to the tree construction stage is a sequence of tokens from the tokenization stage.
1. initial mode
    - The first mode.
2. before html mode
    - Receiving the html token.
    - Reprocessing of the token.
    - Creating the HTMLHtmlElement element.
    - Appending the HTMLHtmlElement to the root Document object.
3. before head
    - The "body" token is then received.
    - An HTMLHeadElement will be created although we don't have a "head" token.
    - HTMLHeadElement will be added to the tree.
4. in head mode
5. after head mode
    - The body token is reprocessed.
    - An HTMLBodyElement is created and inserted.
6. in body mode
    - Receiving the character tokens of the "Hello world" string.
    - The first one will cause creation and insertion of a "Text" node.
    - The other characters will be appended to that node.
7. after body mode
    - Receiving the body end token.
8. after after body mode
    - Receiving the html end tag.

Receiving the end of file token will end the parsing.

![Figure : tree construction of example html](/images/10.gif)

## Actions When The Parsing Is Finished
At this stage the browser will mark the document as interactive and start parsing scripts that are in "deferred" mode: those that should be executed after the document is parsed. The document state will be then set to "complete" and a "load" event will be fired.

## Browsers' Error Tolerance
You never get an "Invalid Syntax" error on an HTML page. Browsers fix any invalid content and go on.
   
Take this HTML for example:
```HTML
<html>
    <mytag>
    </mytag>
    <div>
    <p>
    </div>
    Really lousy HTML
    </p>
</html>
```
I must have violated about a million rules ("mytag" is not a standard tag, wrong nesting of the "p" and "div" elements and more) but the browser still shows it correctly and doesn't complain.

Let's see some WebKit error tolerance examples:
1. &lt;/br&gt; instead of &lt;br&gt;
    - &lt;/br&gt; will be treated like &lt;br&gt;.
2. A stray table
    - A stray table is a table inside another table, but not inside a table cell.
    - The tables will be siblings.
    - For example:
        ```HTML
        <table>
            <table>
                <tr><td>inner table</td></tr>
            </table>
            <tr><td>outer table</td></tr>
        </table>
        
        <!-- WebKit will change the hierarchy to two sibling tables: -->
        <table>
           <tr><td>outer table</td></tr>
        </table>
        <table>
           <tr><td>inner table</td></tr>
        </table>
        ```
3. Nested form elements
    - In case the user puts a form inside another form, the second form is ignored.
4. A too deep tag hierarchy
    - We will only allow at most 20 nested tags of the same type before just ignoring them all together.
5. Misplaced html or body end tags
    - We never close the body tag, since some stupid web pages close it before the actual end of the doc. 
    - Let's rely on the end() call to close things.