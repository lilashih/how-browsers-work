# The Flow of Constructing The Tree

### Firefox
- The presentation is registered as a listener for DOM updates.
- The presentation delegates frame creation to the FrameConstructor.
- The FrameConstructor resolves style and creates a frame (Firefox calls the renderers as frames).

### WebKit
#### Attachment
- The process of resolving the style and creating a renderer is called **attachment**.
- Every DOM node has an attach method. 
- Attachment is synchronous.
- Node insertion to the DOM tree calls the new node attach method.

## Viewport
Viewport is the dimensions of root render object.

### Root Render Object
- Processing the html and body tags results in the construction of the render tree root. 
- It is the top most block that contains all other blocks.

### Viewport
- The dimensions or root render object are the Viewport.
- The browser window display area dimensions.
- Firefox calls it ViewPortFrame.
- WebKit calls it RenderView. 
- The rest of the tree is constructed as a DOM nodes insertion.

![Figure : The render tree and the corresponding DOM tree (3.1). The "Viewport" is the initial containing block. In WebKit it will be the "RenderView" object](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/image025.png)