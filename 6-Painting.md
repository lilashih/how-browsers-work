# Painting
- The render tree is traversed and the renderer's **paint()** method is called to display content on the screen. 
- Painting uses the UI infrastructure component.


## Global and Incremental
Like layout, painting can also be global or incremental.

### Global Layout
- The entire tree is painted

### Incremental Layout
- Some of the renderers change in a way that does not affect the entire tree.
- The changed renderer invalidates its rectangle on the screen. This causes the OS to see it as a **dirty region** and generate a **paint** event.


## The Painting Order
1. background color
2. background image
3. border
4. children
5. outline