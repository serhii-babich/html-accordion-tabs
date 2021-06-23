# Accordion Tabs with pure CSS

Yep, this is yet another “article” about how amazing pure CSS is and how you can do something cool without a line of JavaScript. Why am I bothering to write it?

Because I am still fascinated by how amazing pure CSS is and how you can do something cool without a line of JavaScript.

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

The main idea is to use the very simple yet powerful ability of HTML form controls to have a state and the ability to access this state with CSS pseudo-classes. To be more precise, I’ll use `:checked` pseudo-class here. This means I can style adjacent siblings of checked input using + combinator.

To emulate tabs behaviour I need to be able to display only active tab content, and by active I mean one that is the closest adjacent to checked radio.

Radio-button itself also should be hidden, leaving only the linked label to be visible and interactive. Long story short, this is how these tabs are intended to work. Let’s write some basic CSS code for our tabs.

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

First of all, I decided to create some CSS variables for order property, so this is what is inside `:root` rule. We’ll get back to this a bit later.

```css
.tab-container {
  display: flex;
  flex-wrap: wrap;
}
```

We are using flex layout as it’ll allow us to use an unknown amount of tabs because flex layout distributes its children automatically against fixed-width values we’ll need to put manually otherwise.

By default all flex items are cramped in one line, but we need our tab buttons to be placed at top and content at bottom. Using flex-wrap: wrap will allow flex layout to put large elements to next row.

```html
<input type="radio" id="tab-1" name="tabs" class="tab-actor" checked />
<label for="tab-1" class="tab-button">Lorem ipsum</label>
```

This is how we link label to input — using `id` attribute for input and for attribute for label. When _input-label_ pair has same value for these attributes, clicking on label will activate input as we would click directly on input. This will allow us hide input:

```css
.tab-actor {
  display: none;
}
```

Next, we will add some black flex magic to achieve layout we want.

HTML we wrote would result in next:

```
[tab]
[content]
[tab]
```

But what we need is following:

```
[tab][tab]
[content]
```

To achieve our goal we will use an `order` CSS property which orders (no puns intended) elements inside flex layout despite actual position in DOM-tree. The following code sets the order for `.tab-button` elements to be at the start of layout and `.tab-content` to be at:

```css
.tab-button {
  order: var(--tab-button-order);
}

.tab-content {
  order: var(--tab-content-order);
  display: none;
}
```

`.tab-content` is hidden by default. We will unhide active tab content using following code:

```css
.tab-actor:checked + .tab-button + .tab-content {
  display: block;
}
```

It’s a big selector, for sure, but it does all the magic. All content is hidden and we want to display only content corresponding to the activated tab button. This selector literally says following:

> Display content after button which follows checked input

`+` combinator selects immediately adjacent elements, this is why our HTML code should follow this exact order.

There is another approach, using `~` combinator, which is also adjacent, but not strict and selects **all** matching adjacent elements. Using `~` at first glance will shorten the selector to:

```css
.tab-actor:checked ~ .tab-content {
  display: block;
}
```

but in this case first tab will activate all adjacent content, and to avoid this we’ll need to specify which tab displays which content:

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

Ok, now we have not that pretty, but actually working tabs with pure CSS and HTML.

## Music time!

Or, to be precise — time to convert tabs to the accordion.

Why?

On small screens tabs can be not the best approach to display content and exactly same layout we were trying to avoid at the beginning can come in handy:

```
Desktop:
[tab][tab]
[content]

Mobile:
[tab]
[tab]
[content]
```

All we need to do is just revert flex order and adjust button width for small screens:

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

That’s it. It just works.

## But wait! There is more!

It’s all cool and great, but these tabs are booooring. Let’s give them a bit of fanciness.

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

We added some paddings, colours and a bit of animation. Nice! Looks great now! But, as you can notice, content in «mobile» mode is switching extremely boring, without a spark of joy. Let’s add this spark then:

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

What happens here? We added a `.full-height` class to our `.tab-container` and sprinkled some fun CSS over it.

```css
.tab-container.full-height {
  height: 100vh;
  flex-direction: column;
}
```

Right here we are telling our accordion to occupy exactly full-screen height and order all children in column flex layout.

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

Now we are casting some magic on `.tab-content`, allowing it to expand and collapse with a neat animation.

## Epilogue

That’s, my friends, is how I met… Ah, sorry, it is how we can make responsive tabs which switch to accordion layout on the fly without a single line of JS.
