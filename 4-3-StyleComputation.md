# Style Computation
Building the render tree requires calculating the visual properties of each render object. This is done by calculating the style properties of each element.

## Origins of Style 
The style includes style sheets of various origins, includes:
1. Browser's default style sheets
2. Provided by the web author
    - Includes inline style elements and visual properties in the HTML.
3. Provided by the browser user (user style sheet)
    - In Firefox, this is done by placing a style sheet in the "Firefox Profile" folder.

## Difficulties of Computation
Style computation brings up a few difficulties:
1. Memory Problems
    - A very large construct.
    - Holding the numerous style properties.
2. Performance Issues  
    - Not optimized style data.
    - Traversing the entire rule list for each element to find matches.
    - Complex structure of selectors can cause the matching process to start on a seemingly promising path that is proven to be futile and another path has to be tried.  
        For example:
        ```
        div div div div{
          ...
        }
        ```
        Means the rules apply to a &lt;div&gt; who is the descendant of 3 divs. You may need to traverse the node tree up just to find out there are only two divs and the rule does not apply, then need to try other paths in the tree.
3. Complex Cascade Rules

Let's see how the browsers face these issues:

## Sharing Style Data
WebKit nodes references style objects (RenderStyle). These objects can be shared by nodes in some conditions. The nodes are siblings or cousins and:
1. The elements must be in the same mouse state
    - For example, one can't be in :hover while the other isn't
2. Neither element should have an id
3. The tag names should match
4. The class attributes should match
5. The set of mapped attributes must be identical
6. The link states must match
7. The focus states must match
8. Neither element should be affected by attribute selectors
9. There must be no inline style attribute on the elements
10. There must be no sibling selectors in use at all
    - This includes the + selector and selectors like :first-child and :last-child
    - WebCore simply throws a global switch when any sibling selector is encountered and disables style sharing

    
## Firefox Rule Tree
Firefox has two extra trees for easier style computation: 
1. The Style Context Tree
2. The Rule Tree

### The Style Context Tree
![Figure : Firefox style context tree](/images/13.png)

The style contexts contain end values. The values are computed by applying all the matching rules in the correct order and performing manipulations that transform them from logical to concrete values. 

For example, if the logical value is a percentage of the screen it will be calculated and transformed to absolute units.

### The Rule Tree
It enables sharing these values between nodes to avoid computing them again. This also saves space.

All the matched rules are stored in a tree. The bottom nodes in a path have higher priority. The tree contains all the paths for rule matches that were found. 

Storing the rules is done lazily. The tree isn't calculated at the beginning for every node, but whenever a node style needs to be computed the computed paths are added to the tree.

Let's see how the tree saves us work.

#### Division Into Structs
The style contexts are divided into structs which contain style information for a certain category like border or color. All the properties in a struct are either inherited or non inherited. 
- Inherited properties 
    - Unless defined by the element, are inherited from its parent. 
- Non inherited properties (Reset properties) 
    - Using default values if not defined.
     
The tree helps us by caching entire structs (containing the computed end values) in the tree. If the bottom node didn't supply a definition for a struct, a cached struct in an upper node can be used.

#### Computing The Style Contexts Using The Rule Tree
When computing the style context for a certain element, 
1. Computing a path 
    - A path in the rule tree or use an existing one. 
2. Applying the rules in the path to fill the structs
    - Starting at the bottom node of the path (the highest precedence, usually the most specific selector) and traverse the tree up until our struct is full. 
        - If there is no specification for the struct in that rule node, then we go up the tree until we find a node that specifies it fully and simply point to it. That's the best optimization, the entire struct is shared. This saves computation of end values and memory.
        - If we find partial definitions we go up the tree until the struct is filled.
        - If we didn't find any definitions then, in case the struct is an "inherited" type, we point to the struct of our parent in the context tree. In this case we also succeeded in sharing structs. 
        - If it's a reset struct then default values will be used.
        - If the most specific node does add values then we need to do some extra calculations for transforming it to actual values. We then cache the result in the tree node so it can be used by children.
        - In case an element has a sibling or a brother that points to the same tree node then the entire style context can be shared between them.
     
Lets see an example: Suppose we have this HTML
```HTML
<html>
    <body>
        <div class="err" id="div1">
            <p>
                this is a <span class="big"> big error </span>
                this is also a
                <span class="big"> very  big  error</span> error
            </p>
        </div>
        <div class="err" id="div2">another error</div>
    </body>
</html>
```

And the following rules:
```CSS
/* 1 */ div {margin:5px;color:black}
/* 2 */ .err {color:red}
/* 3 */ .big {margin-top:3px}
/* 4 */ div span {margin-bottom:4px}
/* 5 */ #div1 {color:blue}
/* 6 */ #div2 {color:green}
```
To simplify things let's say we need to fill out only two structs: 
1. The color struct
    - Only one member.
2. The margin struct
    - Contains the four sides.

The resulting rule tree will look like this (the nodes are marked with the node name: the number of the css rule they point at):
![Figure : The rule tree](/images/14.png)

The context tree will look like this (node name: rule node they point to):
![Figure : The context tree](/images/15.png)

Suppose we parse the HTML and get to the second &lt;div&gt; tag. We will match the rules and discover that the matching rules for the &lt;div&gt; are 1, 2 and 6. This means there is already an existing path in the tree that our element can use, and we just need to add another node to it for rule 6 (node F in the rule tree).
We will create a style context and put it in the context tree. The new style context will point to node F in the rule tree.

We now need to fill the style structs. Since the last rule node (F) doesn't add to the margin struct, we can go up the tree until we find a cached struct computed in a previous node insertion and use it. We will find it on node B, which is the uppermost node that specified margin rules.

We do have a definition for the color struct, so we can't use a cached struct. Since color has one attribute we don't need to go up the tree to fill other attributes. We will compute the end value (convert string to RGB etc) and cache the computed struct on this node.

The work on the second &lt;span&gt; element is even easier. We will match the rules and come to the conclusion that it points to rule G, like the previous span. Since we have siblings that point to the same node, we can share the entire style context and just point to the context of the previous span.

For structs that contain rules that are inherited from the parent, caching is done on the context tree (the color property is actually inherited, but Firefox treats it as reset and caches it on the rule tree).

## WebKit
WebKit doesn't not have a rule tree, the matched declarations are traversed four times. 
1. First non-important high priority properties are applied (properties that should be applied first because others depend on them, such as display).
2. Then high priority important, then normal priority non-important
3. Normal priority important rules. 

This means that properties that appear multiple times will be resolved according to the correct cascade order. The last wins.

So to summarize: sharing the style objects (entirely or some of the structs inside them) solves issues 1 and 3. The Firefox rule tree also helps in applying the properties in the correct order.


## Manipulating The Rules For An Easy Match
There are several sources for style rules:
- CSS rules, either in external style sheets or in style elements.
```CSS
p {color: blue}
```
- Inline style attributes like
```HTML
<p style="color: blue" />
```
- HTML visual attributes (which are mapped to relevant style rules)
```HTML
<p bgcolor="blue" />
```

The last two are easily matched to the element since he owns the style attributes and HTML attributes can be mapped using the element as the key.

### Hash map
After parsing the style sheet, the rules are added to one of several hash maps, according to the selector. There are maps by id, by class name, by tag name and a general map for anything that doesn't fit into those categories. If the selector is an id, the rule will be added to the id map, if it's a class it will be added to the class map etc.

This manipulation makes it much easier to match rules. There is no need to look in every declaration: we can extract the relevant rules for an element from the maps. This optimization eliminates 95+% of the rules.

Let's see for example the following style rules:
```CSS
p.error {color: red}
#messageDiv {height: 50px}
div {margin: 5px}
```
The first rule will be inserted into the class map. The second into the id map and the third into the tag map.

For the following HTML fragment;
```HTML
<p class="error">an error occurred </p>
<div id=" messageDiv">this is a message</div>
```
We will first try to find rules for the p element. The class map will contain an "error" key under which the rule for "p.error" is found. The div element will have relevant rules in the id map (the key is the id) and the tag map. So the only work left is finding out which of the rules that were extracted by the keys really match.

For example if the rule for the div was:
```CSS
table div {margin:5px}
```
It will still be extracted from the tag map, because the key is the rightmost selector, but it would not match our div element, who does not have a table ancestor.

Both WebKit and Firefox do this manipulation.

## Applying The Rules In The Correct Cascade Order
A declaration for a style property can appear in several style sheets, and several times inside a style sheet. So the order of applying the rules is very important. This is called the **cascade** order. 

### Style Sheet Cascade Order
According to CSS2 spec, the cascade order is (from low to high):
1. Browser declarations
2. User normal declarations
3. Author normal declarations
4. Author important declarations
5. User important declarations

### Specificity
The selector specificity is defined by the CSS2 specification as follows:
- count 1 if the declaration it is from is a "style" attribute rather than a rule with a selector, 0 otherwise (= a)
- count the number of ID attributes in the selector (= b)
- count the number of other attributes and pseudo-classes in the selector (= c)
- count the number of element names and pseudo-elements in the selector (= d)

Concatenating the four numbers a-b-c-d gives the specificity.

Some examples:
```CSS
*             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
#x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
```

### Sorting The Rules
After the rules are matched, they are sorted according to the cascade rules. WebKit uses bubble sort for small lists and merge sort for big ones. WebKit implements sorting by overriding the ">" operator for the rules:
```
static bool operator >(CSSRuleData& r1, CSSRuleData& r2)
{
    int spec1 = r1.selector()->specificity();
    int spec2 = r2.selector()->specificity();
    return (spec1 == spec2) : r1.position() > r2.position() : spec1 > spec2;
}
```

## Gradual Process
WebKit uses a flag that marks if all top level style sheets (including @imports) have been loaded. If the style is not fully loaded when attaching, place holders are used and it is marked in the document, and they will be recalculated once the style sheets were loaded.