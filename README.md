# Accordion Tabs with pure CSS

Yep, this is yet another “article” about how amazing pure CSS is, and how you can do something cool without a single line of JavaScript. Why do I bother to write it? Because I am still fascinated by how amazing pure CSS is, and how you can do something cool without a single line of JavaScript.

<small>The final tiny disclaimer: all this is just for fun and to show you how amazing CSS is.</small>

## HTML structure

We will use very simple HTML markup to define our accordion tabs:

- `.tabs-container` — wrapper element to host all tab related elements;
- `input.tab-actor` — hidden radio-button to control tab content visibility;
- `label.tab-button` — label linked to input, serving as tab button;
- `.tab-content` — wrapper for any content you’ll feel worth putting into;

The tiniest example ever will look like this:

```html
<div class="tab-container">
  <input type="radio" id="tab-1" name="tabs" class="tab-actor" checked />
  <label for="tab-1" class="tab-button">Lorem ipsum</label>
  <section class="tab-content">
    <div class="content">…</div>
  </section>
</div>
```

## How this works

The main idea is to use a very simple yet powerful ability of HTML form controls to have a state and the ability to access this state with CSS pseudo-classes. Namely, I use `:checked` pseudo-class here. This means I style adjacent siblings of the checked input using `+` combinator.

To emulate tabs behaviour, I need to display only active tab content. By active I mean the closest adjacent to the checked radio.

The radio-button also should be hidden, leaving only the linked label visible and interactive.

Long story short, this is how these tabs are intended to work. Let’s write some basic CSS code for the tabs.

## Some basic code

```css
:root {
  --tab-button-order: 1;
  --tab-content-order: 10;
}

.tab-container {
  display: flex;
  flex-wrap: wrap;
}

.tab-actor {
  display: none;
}

.tab-button {
  order: var(--tab-button-order);
}

.tab-content {
  order: var(--tab-content-order);
  display: none;
}

.tab-actor:checked + .tab-button + .tab-content {
  display: block;
}
```

Let’s go through each rule to understand what happens.

First, I create some CSS variables for an order property, and this is what the inside `:root` rule is. We’ll get back to this a bit later.

```css
.tab-container {
  display: flex;
  flex-wrap: wrap;
}
```

We employ flex layout. It allows us to use an unknown number of tabs because it automatically distributes its children. Otherwise, we need to put the fixed-width values manually.

By default, all flex items are cramped in one line, but we need the tab buttons placed at the top and content at the bottom. Using flex-wrap: allows flex layout to put large elements to the next row.

```html
<input type="radio" id="tab-1" name="tabs" class="tab-actor" checked />
<label for="tab-1" class="tab-button">Lorem ipsum</label>
```

We link label to input using id attribute for the input and for attribute for the label. When the input-label pair has the same values of the attributes, clicking the label activates the input as we click directly on the input.

This allows us to hide input:

```css
.tab-actor {
  display: none;
}
```

Next, we add some black flex magic to get the layout we want.

HTML we have already written results in this:

```
[tab]
[content]
[tab]
```

But what we need is the following:

```
[tab][tab]
[content]
```

To achieve our goal we should use an `order` CSS property that orders (no pun intended) elements inside flex layout despite the actual position in the DOM-tree. The following code sets the order for `.tab-button` elements to be at the start of layout and `.tab-content` to be at:

```css
.tab-button {
  order: var(--tab-button-order);
}

.tab-content {
  order: var(--tab-content-order);
  display: none;
}
```

`.tab-content` is hidden by default. We unhide active tab content using the code:

```css
.tab-actor:checked + .tab-button + .tab-content {
  display: block;
}
```

It’s a big selector, for sure, but it does all the magic. All content is hidden, and we want to display only the content corresponding to the activated tab button. This selector literally says the following:

> Display the content after the button that follows the checked input

`+` combinator selects immediately adjacent elements, that’s why the HTML code should follow this exact order.

Another approach is to use `~` combinator. It is also adjacent but not strict and selects **all** matching adjacent elements. Using `~` shortens the selector to:

```css
.tab-actor:checked ~ .tab-content {
  display: block;
}
```

Though in this case, the first tab activates all adjacent content. To avoid this, we need to specify which tab displays which content:

```css
/* Don't write code like this. Please. */
.tab-actor.tab-1:checked ~ .tab-content.tab-1,
.tab-actor.tab-2:checked ~ .tab-content.tab-2,
.tab-actor.tab-3:checked ~ .tab-content.tab-3,
.tab-actor.tab-4:checked ~ .tab-content.tab-4 {
  display: block;
}
```

Not that much optimization, to be honest.

Ok, now we have not that pretty but working tabs made with pure CSS and HTML.

## Music time!

Or, to be precise, it’s time to convert tabs into the accordion.
Why?

On small screens, tabs aren’t the best option to display content, and the very layout we tried to avoid at the beginning comes in handy here:

```
Desktop:
[tab][tab]
[content]

Mobile:
[tab]
[tab]
[content]
```

All we need is just to revert flex order and adjust the button width to small screens:

```css
@media screen and (max-width: 480px) {
  .tab-button,
  .tab-content {
    order: initial;
  }

  .tab-button {
    width: 100%;
  }
}
```

That’s it. It works.

## Wait! There is more!

It’s all cool and great, but these tabs are so booooring. Let’s glam them up!

```css
* {
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, "Helvetica Neue", Helvetica, sans-serif;
  max-width: 1280px;
  margin-inline: auto;
}

.tab-container {
  box-shadow: rgba(0, 0, 0, 0.5) 0 1px 2px;
  justify-content: center;
}

.tab-button {
  padding: 8px 16px;
  border-bottom: transparent 4px solid;
  transition: border-bottom-color 0.4s;
}

.content {
  padding: 16px;
}

.tab-actor:checked + .tab-button {
  border-bottom-color: rgb(82, 2, 136);
}
```

We add some paddings, colours and a bit of animation. Looks great now! Though, as you can notice, content in the “mobile” mode switches dully, without a single spark of joy. Let’s add some sparks then:

```css
@media screen and (max-width: 480px) {
  .tab-button {
    border-bottom: 1px solid #ccc;
    transition: none;
  }

  .tab-content {
    background-color: ivory;
  }

  .tab-container.full-height {
    height: 100vh;
    flex-direction: column;
  }

  .tab-container.full-height .tab-content {
    display: block;
    height: auto;
    flex: 0;
    overflow: hidden;

    transition: 300ms flex;
  }

  .tab-container.full-height .tab-actor:checked + .tab-button + .tab-content {
    flex: 1;
  }
}
```

What happens here? We add a `.full-height` class to our `.tab-container` and sprinkled some fun CSS over it.

```css
.tab-container.full-height {
  height: 100vh;
  flex-direction: column;
}
```

Right here we tell our accordion to occupy exactly the full-screen height and order all children in column flex layout.

```css
.tab-container.full-height .tab-content {
  display: block;
  height: auto;
  flex: 0;
  overflow: hidden;

  transition: 300ms flex;
}

.tab-container.full-height .tab-actor:checked + .tab-button + .tab-content {
  flex: 1;
}
```

Now we cast some magic on `.tab-content`, allowing it to expand and collapse with a neat animation.

## Epilogue

That’s, my friends, is how I met… Ah, sorry, it is how we can make responsive tabs that switch to the accordion layout on the fly without a single line of JS.

---

Edited by [@Ulyanka_A](https://twitter.com/Ulyanka_A)
