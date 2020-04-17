# Dynamic Changes
- The minimal possible actions in response to a change. 
    - Changes to an element's color will cause only repaint of the element. 
    - Changes to the element position will cause layout and repaint of the element, its children and possibly siblings. 
- Adding a DOM node will cause layout and repaint of the node. 
- Major changes, like increasing font size of the **html** element, will cause invalidation of caches, relayout and repaint of the entire tree. 