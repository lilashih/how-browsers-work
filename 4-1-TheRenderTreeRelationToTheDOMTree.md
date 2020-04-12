# The Render Tree Relation to The DOM Tree
The renderers correspond to DOM elements, but the relation is not one to one. 

## Non-visual DOM Elements
Non-visual DOM elements will not be inserted in the render tree, for example:
- The "head" element. 
- Elements whose display value was assigned to "none".
    - Whereas elements with "hidden" visibility will appear in the tree.

## Multiple renderers
The DOM elements which correspond to several visual objects. These elements are usually with complex structure that **cannot be described by a single rectangle**. 

For example:
- The "select" element has three renderers: 
    - Display area.
    - The drop down list box.
    - The button.
- Text is broken into multiple lines.
    - The width is not sufficient for one line.
    - The new lines will be added as extra renderers.
- Broken HTML

## Not in the same place
Some render objects correspond to a DOM node but not in the same place in the DOM tree.

For example:
- Floats
- Absolutely positioned elements

These elements are out of flow, placed in a different part of the tree, and mapped to the real frame. A placeholder frame is where they should have been.

![Figure : The render tree and the corresponding DOM tree (3.1). The "Viewport" is the initial containing block. In WebKit it will be the "RenderView" object](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/image025.png)

The render tree and the corresponding DOM tree. The "Viewport" is the initial containing block. In WebKit it will be the "RenderView" object.