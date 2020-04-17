# The Rendering Engine's Threads

## Single Thread
Almost everything, except network operations, happens in a single thread. Network operations can be performed by several parallel threads. The number of parallel connections is limited (usually 2–6 connections).

### Firefox and Safari
This is the main thread of the browser. 

### Chrome
It's the tab process main thread. 


## Event loop
- The browser main thread is an event loop. 
- An infinite loop that keeps the process alive. 
- Waiting for events (like layout and paint events) and processes them. 