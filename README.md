# Code-Styleguide Mirror
Small styleguide for css, less and more.

Hugely inspierd by [Harry Roberts](http://www.cssguidelin.es)


## Table of Contents
1. [Syntax and Formatting](#syntax-and-formatting)
1. [Naming Conventions](#naming-conventions)
1. [Nesting](#nesting)
1. [!important](#important)
1. [Other](#other)
1. [Quasi-Qualified Selectors](#quasi-qualified-selectors)
1. [Selector Performance](#selector-performance)
1. [IDs in CSS](#ids-in-css)



## Syntax and Formatting
#### At a very high-level, we want:
- Tab indents (4). (No spaces!)
	- Why? Because!
- Multi-line CSS
- Multiple files
- Indenting
- Meaningful use of whitespace.

But, as with anything, the specifics are somewhat irrelevant—consistency is key!

### Multiple files
With the meteoric rise of preprocessors of late, more often is the case that developers are splitting CSS across multiple files.

### Indenting Less (or Sass)
Less (or Sass) provides nesting functionality. That is to say, by writing this:

```less

.foo {
    color: red;
    .bar {
        color: blue;
    }
}
```

…we will be left with this compiled CSS:

```less
.foo { color: red; }
.foo .bar { color: blue; }
```

When indenting Less (or Sass), we stick to the same four (4) tab size, and we also leave a blank line before and after the nested ruleset.

Nesting in Less (or Sass) should be avoided wherever possible. See the Specificity section for more details.

### Meaningful Whitespace
As well as indentation, we can provide a lot of information through liberal and judicious use of whitespace between rulesets. We use:

- One (1) empty line between closely related rulesets.
- Two (2) empty lines between loosely related rulesets.
- Five (5) empty lines between entirely new sections.


**[Back to top](#table-of-contents)**


## Naming Conventions
Naming conventions in CSS are hugely useful in making your code more strict, more transparent, and more informative.
A good naming convention will tell you and your team
- what type of thing a class does;
- where a class can be used;
- what (else) a class might be related to.

The naming convention I follow is very simple: hyphen (-) delimited strings.

It’s worth noting that a naming convention is not normally useful CSS-side of development; they really come into their own when viewed in HTML.

### Hyphen Delimited
All strings in classes are delimited with a hyphen (-).

```html
<div class="mega-awesome-super-class"></div>
```

### JavaScript Hooks
As a rule, it is unwise to bind your CSS and your JS onto the same class in your HTML. This is because doing so means you can’t have (or remove) one without (removing) the other. It is much cleaner, much more transparent, and much more maintainable to bind your JS onto specific classes.

I have known occasions before when trying to refactor some CSS has unwittingly removed JS functionality because the two were tied to each other—it was impossible to have one without the other.

Typically, these are classes that are prepended with js-, for example:

```html
<input type="submit" class="btn  js-btn" value="Follow" />
```

This means that we can have an element elsewhere which can carry with style of .btn {}, but without the behaviour of .js-btn.

### data-* Attributes
A common practice is to use data-* attributes as JS hooks, but this is incorrect. data-* attributes, as per the spec, are used to store custom data private to the page or application (emphasis mine). data-* attributes are designed to store data, not be bound to.

### CSS Selectors
Perhaps somewhat surprisingly, one of the most fundamental, critical aspects of writing maintainable and scalable CSS is selectors. Their specificity, their portability, and their reusability all have a direct impact on the mileage we will get out of our CSS, and the headaches it might bring us.

### Selector Intent
It is important when writing CSS that we scope our selectors correctly, and that we’re selecting the right things for the right reasons. Selector Intent is the process of deciding and defining what you want to style and how you will go about selecting it. For example, if you are wanting to style your website’s main navigation menu, a selector like this would be incredibly unwise:

```less
header ul {}
```

This selector’s intent is to style any ul inside any header element, whereas our intent was to style the site’s main navigation. This is poor Selector Intent: you can have any number of header elements on a page, and they in turn can house any number of uls, so a selector like this runs the risk of applying very specific styling to a very wide number of elements. This will result in having to write more CSS to undo the greedy nature of such a selector.

A better approach would be a selector like:

```less
.site-nav {}
```

An unambiguous, explicit selector with good Selector Intent. We are explicitly selecting the right thing for exactly the right reason.
Poor Selector Intent is one of the biggest reasons for headaches on CSS projects. Writing rules that are far too greedy—and that apply very specific treatments via very far reaching selectors—causes unexpected side effects and leads to very tangled stylesheets, with selectors overstepping their intentions and impacting and interfering with otherwise unrelated rulesets.
CSS cannot be encapsulated, it is inherently leaky, but we can mitigate some of these effects by not writing such globally-operating selectors: your selectors should be as explicit and well reasoned as your reason for wanting to select something.




## Nesting

We’ve already looked at how nesting can lead to location dependent and potentially inefficient code, but now it’s time to take a look at another of its pitfalls: it makes selectors more specific.
When we talk about nesting, we don’t necessarily mean preprocessor nesting, like so:

```less
.foo {
    .bar {}
}
```

We’re actually talking about descendant or child selectors; selectors which rely on a thing within a thing. That could look like any one of the following:

```less
/**
 * An element with a class of `.bar` anywhere inside an element with a class of
 * `.foo`.
 */
.foo .bar {}

/**
 * An element with a class of `.module-title` directly inside an element with a
 * class of `.module`.
 */
.module > .module-title {}

/**
 * Any `li` element anywhere inside a `ul` element anywhere inside a `nav`
 * element
 */
nav ul li {}
```

Whether you arrive at this CSS via a preprocessor or not isn’t particularly important, but it is worth noting that preprocessors tout this as a feature, where is actually to be avoided wherever possible.
Generally speaking, each part in a compound selector adds specificity. Ergo, the fewer parts to a compound selector then the lower its overall specificity, and we always want to keep specificity low. To quote Jonathan Snook:
…whenever declaring your styles, use the least number of selectors required to style an element.
Let’s look at an example:

```less
.widget {
    padding: 10px;
}
    .widget > .widget-title {
        color: red;
    }
```

To style an element with a class of .widget-title, we have a selector that is twice as specific as it needs to be. That means that if we want to make any modifications to .widget__title, we’ll need another at-least-equally specific selector:

```less
.widget { ... }
    .widget > .widget-title { ... }
    .widget > .widget-title-sub {
        color: blue;
    }
```

Not only is this entirely avoidable—we caused this problem ourselves—we have a selector that is literally double the specificity it needs to be. We used 200% of the specificity actually required. And not only that, but this also leads to needless verbosity in our code—more to send over the wire.
As a rule, if a selector will work without it being nested then do not nest it.

## !important

The word !important sends shivers down the spines of almost all front-end developers. !important is a direct manifestation of problems with specificity; it is a way of cheating your way out of specificity wars, but usually comes at a heavy price. It is often viewed as a last resort—a desperate, defeated stab at patching over the symptoms of a much bigger problem with your code.

The general rule is that !important is always a bad thing, but, to quote Jamie Mason:
Rules are the children of principles.

That is to say, a single rule is a simple, black-and-white way of adhering to a much larger principle. When you’re starting out, the rule never use !important is a good one.
However, once you begin to grow and mature as a developer, you begin to understand that the principle behind that rule is simply about keeping specificity low. You’ll also learn when and where the rules can be bent…
!important does have a place in CSS projects, but only if used sparingly and proactively.
Proactive use of !important is when it is used before you’ve encountered any specificity problems; when it is used as a guarantee rather than as a fix. For example:

```less
.one-half {
    width: 50% !important;
}
.hidden {
    display: none !important;
}
```

These two helper, or utility, classes are very specific in their intentions: you would only use them if you wanted something to be rendered at 50% width or not rendered at all. If you didn’t want this behaviour, you would not use these classes, therefore whenever you do use them you will definitely want them to win.

Here we proactively apply !important to ensure that these styles always win. This is correct use of !important to guarantee that these trumps always work, and don’t accidentally get overridden by something else more specific.
Incorrect, reactive use of !important is when it is used to combat specificity problems after the fact: applying !important to declarations because of poorly architected CSS. For example, let’s imagine we have this HTML:

```html
<div class="content">
    <h2 class="heading-sub">...</h2>
</div>
```

…and this CSS:

```less
.content h2 {
    font-size: 2em;
}

.heading-sub {
    font-size: 1.5em !important;
}
```

Here we can see how we’ve used !important to force our .heading-sub {} styles to reactively override our .content h2 {} selector. This could have been circumvented by any number of things, including using better Selector Intent, or avoiding nesting.
In these situations, it is preferable that you investigate and refactor any offending rulesets to try and bring specificity down across the board, as opposed to introducing such specificity heavyweights.
Only use !important proactively, not reactively.

## Quasi-Qualified Selectors
One thing that qualified selectors can be useful for is signalling where a class might be expected or intended to be used, for example:
```less
ul.nav {}
```
Here we can see that the .nav class is meant to be used on a ul element, and not on a nav. By using quasi-qualified selectors we can still provide that information without actually qualifying the selector:

```less
/*ul*/.nav {}
```
By commenting out the leading element, we can still leave it to be read, but avoid qualifying and increasing the specificity of the selector.


##Selector Performance
A topic which is—with the quality of today’s browsers—more interesting than it is important, is selector performance. That is to say, how quickly a browser can match the selectors your write in CSS up with the nodes it finds in the DOM.
Generally speaking, the longer a selector is (i.e. the more component parts) the slower it is, for example:
```less
body.home div.header ul {}
```
…is a far less efficient selector than:
```less
.primary-nav {}
```
This is because browsers read CSS selectors right-to-left. A browser will read the first selector as
- find all ul elements in the DOM;
- now check if they live anywhere inside an element with a class of .header;
- next check that .header class exists on a div element;
- now check that that all lives anywhere inside any elements with a class of .home;
- finally, check that .home exists on a body element.

The second, in contrast, is simply a case of the browser reading
- find all the elements with a class of .primary-nav.

To further compound the problem, we are using descendant selectors (e.g. .foo .bar {}). The upshot of this is that a browser is required to start with the rightmost part of the selector (i.e. .bar) and keep looking up the DOM indefinitely until it finds the next part (i.e. .foo). This could mean stepping up the DOM dozens of times until a match is found.

This is just one reason why nesting with preprocessors is often a false economy; as well as making selectors unnecessarily more specific, and creating location dependency, it also creates more work for the browser.

By using a child selector (e.g. .foo > .bar {}) we can make the process much more efficient, because this only requires the browser to look one level higher in the DOM, and it will stop regardless of whether or not it found a match.


##IDs in CSS

If we want to keep specificity low, which we do, we have one really quick-win, simple, easy-to-follow rule that we can employ to help us: avoid using IDs in CSS.

Not only are IDs inherently non-reusable, they are also vastly more specific than any other selector, and therefore become specificity anomalies. Where the rest of your selectors are relatively low specificity, your ID-based selectors are, comparatively, much, much higher.

In fact, to highlight the severity of this difference, see how one thousand chained classes cannot override the specificity of a single ID: jsfiddle.net/0yb7rque. (Please note that in Firefox you may see the text rendering in blue: this is a known bug, and an ID will be overridden by 256 chained classes.)

N.B. It is still perfectly okay to use IDs in HTML and JavaScript; it is only in CSS that they prove troublesome.

It is often suggested that developers who choose not to use IDs in CSS merely don’t understand how specificity works. This is as incorrect as it is offensive: no matter how experienced a developer you are, this behaviour cannot be circumvented; no amount of knowledge will make an ID less specific.
Opting into this way of working only introduces the chance of problems occurring further down the line, and—particularly when working at scale—all efforts should be made to avoid the potential for problems to arise. In a sentence:

It is just not worth introducing the risk.



## General Rules
Your selectors are fundamental to writing good CSS. To very briefly sum up the above sections:
- Select what you want explicitly, rather than relying on circumstance or coincidence. Good Selector Intent will rein in the reach and leak of your styles.
- Write selectors for reusability, so that you can work more efficiently and reduce waste and repetition.
- Do not nest selectors unnecessarily, because this will increase specificity and affect where else you can use your styles.
- Do not qualify selectors unnecessarily, as this will impact the number of different elements you can apply styles to.
- Keep selectors as short as possible, in order to keep specificity down and performance up.
Focussing on these points will keep your selectors a lot more sane and easy to work with on changing and long-running projects.

## Specificity
As we’ve seen, CSS isn’t the most friendly of languages: globally operating, very leaky, dependent on location, hard to encapsulate, based on inheritance… But! None of that even comes close to the horrors of specificity.

No matter how well considered your naming, regardless of how perfect your source order and cascade are managed, and how well you’ve scoped your rulesets, just one overly-specific selector can undo everything. It is a gigantic curveball, and undermines CSS’ very nature of the cascade, inheritance, and source order.

The problem with specificity is that it sets precedents and trumps that cannot simply be undone. If we take a real example that I was responsible for some years ago:

```less
#content table {}
```
Not only does this exhibit poor Selector Intent—I didn’t actually want every table in the #content area, I wanted a specific type of table that just happened to live there—it is a hugely over-specific selector. This became apparent a number of weeks later, when I needed a second type of table:

```less
#content table {}

/**
 * Uh oh! My styles get overwritten by `#content table {}`.
 */
.my-new-table {}
```
The first selector was trumping the specificity of the one defined after it, working against CSS’ source-order based application of styles. In order to remedy this, I had two main options. I could

refactor my CSS and HTML to remove that ID;
write a more specific selector to override it.
Unfortunately, refactoring would have taken a long time; it was a mature product and the knock-on effects of removing this ID would have been a more substantial business cost than the second option: just write a more specific selector.

```less
#content table {}
#content .my-new-table {}
```
Now I have a selector that is even more specific still! And if I ever want to override this one, I will need another selector of at least the same specificity defined after it. I’ve started on a downward spiral.
Specificity can, among other things,
- limit your ability to extend and manipulate a codebase;
- interrupt and undo CSS’ cascading, inheriting nature;
- cause avoidable verbosity in your project;
- prevent things from working as expected when moved into different environments;
- lead to serious developer frustration.

All of these issues are greatly magnified when working on a larger project with a number of developers contributing code.

## Keep It Low at All Times
The problem with specificity isn’t necessarily that it’s high or low; it’s the fact it is so variant and that it cannot be opted out of: the only way to deal with it is to get progressively more specific—the notorious specificity wars we looked at above.

One of the single, simplest tips for an easier life when writing CSS—particularly at any reasonable scale—is to keep always try and keep specificity as low as possible at all times. Try to make sure there isn’t a lot of variance between selectors in your codebase, and that all selectors strive for as low a specificity as possible.

Doing so will instantly help you tame and manage your project, meaning that no overly-specific selectors are likely to impact or affect anything of a lower specificity elsewhere. It also means you’re less likely to need to fight your way out of specificity corners, and you’ll probably also be writing much smaller stylesheets.

Simple changes to the way we work include, but are not limited to,
- not using IDs in your CSS;
- not nesting selectors;
- not qualifying classes;
- not chaining selectors.

Specificity can be wrangled and understood, but it is safer just to avoid it entirely.