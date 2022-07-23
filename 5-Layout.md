# Layout
When the renderer is created and added to the tree, it does not have a **position** and **size**. Calculating these values of **position** and **size** is called layout or reflow.

- Elements **later in the flow** do not affect the elements that are **earlier in the flow**, but there are exceptions: tables.
- Layout proceed left-to-right, top-to-bottom through the document. 
- Layout is a recursive process and begins at the root renderer(&lt;html&gt;). The position of the root renderer is 0,0 and its dimensions are the viewport. 
- All renderers have a "layout" or "reflow" method, each renderer invokes the layout method of its children that need layout.

## Dirty Bit System
Dirty Bit System is order not to do a full layout for every small change.

A renderer only mark as **dirty** need layout, there are two flags:
- Dirty
- Children are dirty  
    - The renderer itself may be OK, but its child that needs a layout.
    
## Global and Incremental Layout
### Global Layout
- Affect all renders, this can happen as a result of:
    - A global style change, like a font size change.
    - Screen resize.

### Incremental Layout
- It is triggered asynchronously. 
- When renderers are dirty. 
    - Extra content came from the network and was added to the DOM tree.

![Figure : Incremental layout–only dirty renderers and their children are laid out](/images/16.png)

## Asynchronous and Synchronous Layout
### Asynchronous Layout
- Incremental Layout  
    - Firefox queues "reflow commands" for incremental layouts.
    - WebKit also has a timer that executes an incremental layout.

### Synchronous Layout
- Global Layout  
    - Usually be triggered synchronously.
- Scripts asking for style information  
    - Like "offsetHeight" can trigger incremental layout synchronously.
- Layout is triggered as a callback after an initial layout because some attributes  
    - The scrolling position changed.


## Optimizations
- Cache  
    - When a layout is triggered by a **resize** or a change in the renderer position, the renders sizes are taken from a cache and not recalculated.  
- Not start from the root  
    - Only a sub tree is modified and layout does not start from the root. 
    - The change is local and does not affect its surroundings.
    - Like text inserted into text fields.
        
    
## The Layout Process
The layout usually has the following pattern:
1. Parent renderer determines its own width.
2. Parent goes over children and:
    1. Place the child renderer (sets its x and y).
    2. Calls child layout if needed (dirty or in a global layout, or for some other reason) which calculates the child's height.
3. Parent uses children's accumulative heights and the heights of margins and padding to set its own height–this will be used by the parent renderer's parent.
4. Sets its dirty bit to false.

## Width Calculation
The renderer's width is calculated using the container block's width, the renderer's style **width** property, the margins and borders. 

For example the width of the following div:
```html
<div style="width: 30%"/>
```
Would be calculated by WebKit as the following:
1. The container width is:
    - The maximum of the containers availableWidth and 0. 
    - The availableWidth in this case is the contentWidth which is calculated as:
    ```
    clientWidth() - paddingLeft() - paddingRight()
    ```
    - ClientWidth and clientHeight represent the interior of an object excluding border and scrollbar.

2. The elements width is:
    - The **width** style attribute. 
    - It will be calculated as an absolute value by computing the percentage of the container width.
    - The horizontal borders and paddings are now added.
    - So far this was the calculation of the **preferred width**.

3. The minimum and maximum widths:   
    - If the preferred width is greater then the maximum width, the maximum width is used. 
    - If it is less then the minimum width, the minimum width is used.

The values are cached in case a layout is needed, but the width does not change.

## Line Breaking
When a renderer in the middle of a layout decides that it needs to break, the renderer stops and propagates to the layout's parent that it needs to be broken. The parent creates the extra renderers and calls layout on them.