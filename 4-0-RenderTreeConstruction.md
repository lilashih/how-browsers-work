# Render Tree Construction
While the DOM tree is being constructed, the browser constructs another tree, the render tree. 

## Render Tree
- Render tree generated while DOM tree is being constructed.
- The visual representation of the document.
- The purpose is to enable painting the contents in their correct order.

### The Elements In The Render Tree
- Elements in the render tree are called renderer or render objects.
- Visual elements in the order which they are going to displayed.
- Render object is a rectangle.

#### Terms
- In Firefox
    - Firefox calls the elements in the render tree **frames**. 
- In WebKit
    - WebKit uses the term **renderer**r or **render object**. 

#### WebKit's Renderers
WebKit's RenderObject class, the base class of the renderers, has the following definition:
```
class RenderObject{
    virtual void layout();
    virtual void paint(PaintInfo);
    virtual void rect repaintRect();
    Node* node;  //the DOM node
    RenderStyle* style;  // the computed style
    RenderLayer* containgLayer; //the containing z-index layer
}
```
- A renderer knows how to **layout** and **paint** itself and its children.
- The **rectangular area** of renderer usually corresponding to a node's CSS box, as described by the CSS2 spec. It includes geometric information like width, height and position.

### Type of Renderer
What type of renderer should be created for a DOM node is according to the **display** value of the style attribute.

Here is WebKit code:
```
RenderObject* RenderObject::createObject(Node* node, RenderStyle* style)
{
    Document* doc = node->document();
    RenderArena* arena = doc->renderArena();
    ...
    RenderObject* o = 0;

    switch (style->display()) {
        case NONE:
            break;
        case INLINE:
            o = new (arena) RenderInline(node);
            break;
        case BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case INLINE_BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case LIST_ITEM:
            o = new (arena) RenderListItem(node);
            break;
       ...
    }

    return o;
}
```
Switch case if DOM elements needs to be displayed and how: 
- Render none
- Render inline
- Render inline-block
- Render list-item