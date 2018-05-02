---
published: true
title: Tumblr Custom Theme FAQ
---

## How to edit the Theme's HTML (CSS)?
1. Click on _User_ icon (_Account_)
2. Click _Settings_
3. Near the _Website Theme_ you will find a button "_Edit Theme_"
4. If it is a Custom theme, you will see a button "_Edit HTML_" on top. If you have that - Edit your HTML and CSS there.
5. Read the [Custom theme guide](https://www.tumblr.com/docs/en/custom_themes)
6. If you don't have the ability to edit the HTML, go down, and click "_Advanced options_", there, you will find "_Add custom CSS_"

## How can I see what CSS styles are applied to the Element X on a page?

![HTML Inspector](http://i.imgur.com/5jFE37r.png)

1. Use a Safari or a Chrome.
2. Right click on the element
3. Select "_Inspect element_"
4. In the HTML inspector you will see your element tag is selected
5. Check thet a "_Style_" button is selected in the Inspector
5. On the right side, you will see what CSS styles are applied to the Element, and you can play with them by enabling or editing each style.

## How can I change the font (color) of the Element X on a page
1. Inspect the element, check it's style
2. Note the element's `id`, `class`, `style`. Also note the parent element's `id` and `class`
3. Find it in a CSS of a theme. Example: `id="load"` in CSS will be `#load`
4. Note that some elements can have a `:hover`, which is used when the mouse is hovering on top of the element.
5. If you don't know what is a CSS selector for font or color - click [here](http://lmgtfy.com/?q=CSS+change+font)

## How can I use custom fonts?
1. Read [this](http://lmgtfy.com/?q=CSS+custom+font)
2. Select a custom font, for example from [Google Fonts](https://www.google.com/fonts/)
2. Add the custom font loading in HTML and CSS. Google fonts will show you the code.

## Is it possible to align Element X centred to the page?
1. Sure, inspect the element, search for "_css center_" and edit the css.
2. For a lazy ones: `text-align: center;`

## Text in Element X looks too bold
1. Inspect the element
2. Play with the `font-size`, `font-family`, `font-weight`

## Can we put scrolling images at the top?
1. Sure
2. Search for a [scrolling images java script](http://lmgtfy.com/?q=jQuery+scroller)
3. Read it's documentation, it will say exactly how to add it
