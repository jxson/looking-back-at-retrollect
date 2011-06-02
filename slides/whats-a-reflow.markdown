Anything that changes input information used to construct the rendering tree can cause a repaint or a reflow, for example:

* Adding, removing, updating DOM nodes
* Hiding a DOM node with display: none (reflow and repaint) or visibility: hidden (repaint only, because no geometry changes)
* Moving, animating a DOM node on the page
* Adding a stylesheet, tweaking style properties
* User action such as resizing the window, changing the font size, or (oh, OMG, no!) scrolling
