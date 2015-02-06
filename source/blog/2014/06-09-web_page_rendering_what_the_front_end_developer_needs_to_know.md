---
title: "WEB-page rendering: what the front-end developer needs to know"
created_at: 2014-06-09
kind: article
tags: [frontend, web-development]
excerpt: How exactly does the browser render a web-page? What every front-end developer needs to know.
---

*This is a translation of an article in Russian published on [Habrahabr](http://habrahabr.ru/users/skutin/topics/).*

Greetings, esteemed readers! Today, I would like to shed some light on the question of rendering in web development. Of course, a lot has been written on the topic, but it appears to me that all of the information is fragmented and disorganized. At least for me, in order to collect a complete picture and to process it, I had to analyze quite a lot of data. That is why I decided to formalize my knowledge into this article and share it with the readers. I think that this information will be useful for novice web-developers, as well as the more experienced ones, to refresh and give structure to your knowledge.
The following directions can and should be used to optimize the process of frontend-development, because, obviously, the mark-up, styles and scripts play a direct role in rendering. For that, the corresponding specialists should know a few subtleties.

I will note that the aim of the article is not to accurately represent the inner workings of browsers, but to explain the basic principles of it, especially since various browser engines differ greatly in the algorithms they use to work, so to cover all of the nuances in one article is not a valid possibility.

## The processing of a WEB-page by the browser

To begin, let’s look at the sequence the browser goes through to display a document:

1. A DOM (Document Object Model) is formed from an HTML-document received from the server.
* The styles are loaded and recognized, the CSSOM is formed (CSS Object Model).
* The render tree is formed based on the DOM and CSSOM – a set of render objects (Webkit uses the term "renderer", or "render object", Gecko uses "frame"). The render tree duplicates the structure of the DOM, but the invisible elements are missing from in (for example - `<head>`, or elements with the `display:none;` style). Also, each line of text is presented in the tree as a single renderer. Each render object contains a corresponding DOM object and a style object, calculated for it. Simply put, a render tree is a visual representation of the DOM.
* For each render tree element a position on the page is calculated – the layout process is taking place. Browsers use the flow method, in which it mostly takes only one passing to place all of the elements (tables need more than one).
* Finally, the displaying of all the calculated objects is occurring – the painting process.

In the process of the user’s interaction with the web page, as well as script execution, it changes, which requires some of the above actions to be performed again.

## Repaint

When a change that occurred to the style of the element does not touch its size and position on the page (for example, background-color, border-color, visibility), the browser just redraws it again, with the new style taken into consideration – a process known as repaint (or restyle).

## Reflow

Reflow (or relayout) is performed when the content, the structure of the document is affected by the changes. The reasons for this usually are:

* DOM manipulation (adding, removing, changing and rearranging the elements);
* The changing of the content, including the text in form fields;
* The calculation or alteration of CSS-properties;
* Adding and removing style tables;
* Manipulations with the "class" attribute;
* Manipulation with the browser window — size changes, scrolling;
* Pseudo-class activation (for example, :hover).

## Browser side optimization

When possible, browsers optimize repaint and reflow within elements affected by the change. For example, altering the size of an element the position of which is absolute or fixed will only affect the element itself and its descendants, while changing a statically positioned element will cause a reflow of all the elements following it.

Another notable characteristic – while performing JavaScript, browsers cache all the changes, and apply them all at one after a block of code has finished executing. For example, only 1 reflow and repaint will occur after this block has been executed:

~~~javascript

var $body = $('body');
$body.css('padding', '1px'); // reflow, repaint
$body.css('color', 'red'); // repaint
$body.css('margin', '2px'); // reflow, repaint
// Actually only 1 reflow and repaint occurring

~~~

However, as described previously, a call to the element’s properties will force a reflow. So if we add a property call to the given block of code, an additional reflow will occur:

~~~javascript

var $body = $('body');
$body.css('padding', '1px');
$body.css('padding'); // this will cause a second reflow
$body.css('color', 'red');
$body.css('margin', '2px');

~~~

In the end we have two reflows instead of one. Because of that, property calls should be grouped in one place in order to optimize performance.

In practice, there are situations when you cannot avoid a forced reflow. Let’s assume we need to apply the same property to an element (let’s take "margin-left") without animation (set it to 100px), and then animate using transition into 50px. You can go and look at this example on [JSBin](http://jsbin.com/qutev/1/edit) (*with comments in Russian*), but I will also explain it here.

We will begin with declaring a class with transition:

~~~javascript

has-transition {
   -webkit-transition: margin-left 1s ease-out;
      -moz-transition: margin-left 1s ease-out;
        -o-transition: margin-left 1s ease-out;
           transition: margin-left 1s ease-out;
}

~~~

Then, we will try to implement the desired effect the following way:

~~~javascript

var $targetElem = $('#targetElemId'); // our element, by default has the class "has-transition"

//removing the class with transition
$targetElem.removeClass('has-transition');

// changing the property, expecting  that transition transition is disabled, since we removed the class
$targetElem.css('margin-left', 100);

// reintroducing the class with transition
$targetElem.addClass('has-transition');

// changing the property
$targetElem.css('margin-left', 50);

~~~


The following solution will not work as intended, since the changes are cached and applied only at the end of a block of code. A forced reflow will aid us in achieving the goal, the resulting code will look like this, and will perform the desired task:

~~~javascript

// removing the class with transition
$(this).removeClass('has-transition');

// changing the property
$(this).css('margin-left', 100);

// forcing a reflow, the change in the class and property will be applied immediately
$(this)[0].offsetHeight; // as an example, any property call can be used

// reintroducing the class with transition
$(this).addClass('has-transition');

// changing the property
$(this).css('margin-left', 50);

~~~

## Practical optimization tips

Based on this article, as well as others, where the client side optimization is covered, the following guidelines can be deduced, which can help in creating effective front-end:

Write valid HTML and CSS, noting the encoding. Styles should be kept inside `<head>`, and the scripts – at the end of `<body>`.
Strive to simplify and optimize CSS selectors (this is often neglected by developers who use pre-processes). The less nesting – the better.  Based on the effectiveness of processing, selectors can be ranked the following way (starting from the fastest):

* Identifier: #id
* Class: .class
* Tag: div
* Adjacent selector: a+ i
* Child selector: ul >li
* Universal selector: *
* Attribute selector: input[type="text"]
* Pseudo-elements and Pseudo-classes: a:hover

It must be noted that the browser is processing selectors from left to right, when selecting a key selector (the far right one) you should choose the most effective – identifier and class.

<% highlight :html do %>

div * {...} // bad
.list li {...} // bad
.list-item {...} // good
#list .list-item {...} // good

~~~

In your scripts, minimize the work with DOM. Cache everything: properties, objects, if a repeated usage is implied. With complicated manipulations it is wise to work with the "offline" element (the one that is not in DOM, but in memory), and subsequently move it into DOM. When using jQuery to select elements always follow the recommendation to making selectors.
To change the styles of elements it is better to only modify the "class" attribute, it is a more correct way both from the developing and the support standpoint (segregating the logic and appearance), and less taxing on the browser.
It is better to animate only the absolute or fixed position elements.
Complicated :hover animations can be turned off during scrolling (for example, adding a `no-hover` class to `<body>`).

For a more detailed study of this topic I recommend the following articles:

* [How browsers work](http://taligarsiel.com/Projects/howbrowserswork1.htm)
* [Rendering: repaint, reflow/relayout, restyle](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/)

I hope that every reader found something useful in this article. In any case – thank you for your attention!
