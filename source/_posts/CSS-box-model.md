---
title: CSS box model
date: 2019-05-21 15:21:18
tags:
  - CSS
intro: The tricky parts of CSS box model
---

Every element on the webpage is treated as [Box](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Box_model), which gives the element some attributes we normally work with in our day to day life (padding, margin, border, height, width). Understanding of box model is the foundation of writing CSS.

Box model explain:

<img src="/images/box-model-standard-small.png" alt="Box Model" style="width:400px;" />

The **height** and **width** are actually attibutes of the box content which don't include padding, margin and border width.

**Margin Collapse**: Sometimes, the margins of two boxes might collapse, which is only applied to **vertical margins** (margin-top, margin-bottom).

**Collapse** means the two margins will combine to one, which is larger of the two margin values.

i. The margin of Two adjacent siblings will collapse to larger one.

<img src="/images/sibling.svg" alt="Box Model" style="width:400px;" />

ii. For empty element, if there is no padding, min-height, height, border attributes set, the margin-top and margin-bottom of this element will collapse

<img src="/images/empty.svg" alt="Box Model" style="width:400px;" />

iii. The margin-top of the element and its first child's margin-top will collapse, similarly, the margin-bottom of the element and its last child's margin-bottom will collapse.

Top:

<img src="/images/child.svg" alt="Box Model" style="width:400px;" />

Bottom:

<img src="/images/child2.svg" alt="Box Model" style="width:400px;" />

For situation 3, if the element has top/bottom border or padding, the margin between itself and its first/last child will not collapse. If the element overflow is not visible, or display is inline-block, or position absolute, float, the margins of itself and its first/last child will not collapse.

<!--
<img src="/images/no1.svg" alt="Box Model" style="width:400px;" />

<img src="/images/no2.svg" alt="Box Model" style="width:400px;" />

<img src="/images/no3.svg" alt="Box Model" style="width:400px;" /> -->

More Resources:

[Jonathan Explain](https://www.jonathan-harrell.com/whats-the-deal-with-margin-collapse/)

[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)

**Type of box**: block, inline, inline-block

Set by using display attribute.

The **block** element can use all attributes provide by box model, and which will exhaust the whole width of its parent element by default. Also enforce a line-break between two sibling block elements.

The **inline** element will effect on left/right margin and all padding, the top/bottom margin will not have effect. Can not have width and height, the height will equal to line-height of the parent element. Element can sit on its right or left. The inline element can wrap to two lines.

The **inline-block** element will have all block element attributes and not enforce a line-break. Element can sit on its right or left. Inline-block can not wrap to second lines.

More Resource: [Stackoverflow](https://stackoverflow.com/questions/9189810/css-display-inline-vs-inline-block)
