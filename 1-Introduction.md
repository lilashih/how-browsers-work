# Introduction
## The Main Functionality
The main function of a browser is to present the web resource you choose, by requesting it from the server and displaying it in the browser window.

- The resource  
An HTML document, PDF, image, or some type of content, etc.

- URI (Uniform Resource Identifier)  
The location of the resource is specified by the user using a URI (Uniform Resource Identifier).

- Interpret and Display  
These specifications of interprets and displays HTML files are maintained by the W3C (World Wide Web Consortium) organization, which is the standards organization for the web.  For years browsers conformed to only a part of the specifications and developed their own extensions. That caused serious compatibility issues for web authors.

## User Interfaces
The common browser user interface elements are:
1. Address bar for inserting a URI
2. Back and forward buttons
3. Bookmarking options
4. Refresh and stop buttons for refreshing or stopping the loading of current documents
5. Home button that takes you to your home page

Strangely enough, the browser's user interface is not specified in any formal specification, it just comes from good practices shaped over years of experience and by browsers imitating each other.

## The Main Components
The browser's main components are:

![Figure : Browser components](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/layers.png)

1. The user interface  
    - Every part of the browser display except the window where you see the requested page.
2. The browser engine  
    - Marshals actions between the UI and the rendering engine.
3. The rendering engine  
    - Responsible for displaying requested content. 
    - If the requested content is HTML, the rendering engine parses the requested content, and displays it on the screen.
    - For example, WebKit, Gecko, Blink, etc.
4. Networking  
    - For network calls such as HTTP requests, using different implementations for different platform behind a platform-independent interface.
5. UI backend  
    - Used for drawing basic widgets like combo boxes and windows. 
    - This backend exposes a generic interface that is not platform specific. 
    - Underneath it uses operating system user interface methods.
6. JavaScript interpreter
    - Used to parse and execute JavaScript code.
7. Data storage
    - This is a persistence layer. The browser may need to save all sorts of data locally, such as cookies. 
    - Browsers also support storage mechanisms such as localStorage, IndexedDB, WebSQL and FileSystem.
    
It is important to note that browsers such as Chrome run multiple instances of the rendering engine: one for each tab. Each tab runs in a separate process.