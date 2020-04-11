# The Rendering Engine
Rendering, that is display of the requested contents on the browser screen.

## Rendering Engines
Different browsers use different rendering engines: 
- Internet Explorer uses Trident
- Firefox uses Gecko
- Safari uses WebKit
- Chrome and Opera (from version 15) use Blink, a fork of WebKit

### Basic Rending Engine Flow
First, the rendering engine will get the contents from the networking layer.  
After that, **this is the basic flow of the rendering engine**:
1. Parsing
2. Render tree
3. Layout
4. Paint

![Figure : Rendering engine basic flow](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/flow.png)

##### Parsing HTML to construct the DOM tree
The rendering engine will start parsing the HTML document and convert elements to **DOM nodes in a tree** called **the content tree**. 

##### Render tree construction
The engine will parse the style data, both in external CSS files and in style elements. Styling information together with visual instructions in the HTML will be used to create another tree: **the render tree**.

The render tree contains rectangles with visual attributes like color and dimensions. The rectangles are in the right order to be displayed on the screen.

##### Layout the render tree
After the construction of the render tree it goes through a "layout" process. This means giving each node the exact coordinates where it should appear on the screen. 

##### Painting the render tree
The next stage is painting, the render tree will be traversed and each node will be painted using the UI backend layer.

#### Gradual Process
For better user experience, the rendering engine will try to display contents on the screen as soon as possible. It will not wait until all HTML is parsed before starting to build and layout the render tree. 

Parts of the content will be parsed and displayed, while the process continues with the rest of the contents that keeps coming from the network.


## WebKit
WebKit is an open source rendering engine which started as an engine for the Linux platform and was modified by Apple to support Mac and Windows.

### The Main Flow of WebKit
![Figure : WebKit main flow](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/webkitflow.png)

#### The Terms
1. Attachment
    - For connecting DOM nodes and visual information to create the render tree.
2. Render Tree
    - Consists of **render objects** or **renderers**.
3. Layout
    - For the placing of elements.


## Gecko

### The Main Flow of Gecko
![Figure : Mozilla's Gecko rendering engine main flow](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image008.jpg)

#### The Terms
1. Content Sink
    - An extra layer between the HTML and the DOM tree.
    - A factory for making DOM elements.
2. Frame Tree
    - The tree of visually formatted elements.
3. Reflow
    - For the placing of elements.
    - Which is called layout in WebKit.