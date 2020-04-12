# The Order of Processing Scripts and Style Sheets
## Scripts
### Synchronous
- The model of the web is synchronous.
- To be parsed and executed immediately when the parser reaches a &lt;script&gt; tag. 
- The parsing of the document halts until the script has been executed. 
- If the script is external then the resource must first be fetched from the network–this is also done synchronously, and parsing halts until the resource is fetched. 

### Asynchronous
- Add the "defer" attribute to a script, in which case it will not halt document parsing and will execute after the document is parsed. 
- HTML5 adds an option to mark the script as asynchronous so it will be parsed and executed by a different thread.

## Speculative Parsing
- While executing scripts, another thread parses the rest of the document and finds out what other resources need to be loaded from the network and loads them. 
- Resources can be loaded on parallel connections and overall speed is improved. 
- Only parses references to external resources like external scripts, style sheets and images: it doesn't modify the DOM tree–that is left to the main parser.
- Both WebKit and Firefox do this optimization.

## Style Sheets
- It seems that since style sheets don't change the DOM tree, there is no reason to wait for them and stop the document parsing. 
- There is an issue, when scripts asking for style information that is not loaded and parsed yet during the document parsing stage, the script will get wrong answers and apparently this caused lots of problems.
- Firefox blocks all scripts when there is a style sheet that is still being loaded and parsed. 
- WebKit blocks scripts only when they try to access certain style properties that may be affected by unloaded style sheets.