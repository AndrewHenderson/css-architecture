# css-architecture
Proposal for an Inheritance and Composition Approach

This is a proposal for a structured CSS class system which combines both composition and inheritance. It is guided largely by the [Functional CSS](https://blog.colepeters.com/building-and-shipping-functional-css/) approach, however also incorporates the practice of inheritance common to frameworks such as [Twitter Bootstrap](http://getbootstrap.com/).

[tldr;](#the-system)

### Composition: The Functional Approach
Functional CSS employs the use of single purpose CSS classes that each do one thing very well.
It is guided by the an approach known as "composition" and composition can be arduous.
In my opinion, Functional CSS is the equivalent of a rudimentary language in it’s infancy. It requires many words to convey an idea.

For example, if the concepts of Functional CSS are applied to English, rather than using concise wording to describe something, “I bought a dog.” One instead says, “I bought one of those furry, four-legged things with a tail.”
Consider the following HTML with the Functional approach applied:

```html
<button class=“bg-white border pad-5–10 color-black”></button>
```
### Inheritance: The Framework Approach
Defining a more inclusive vernacular allows us to convey an idea more succinctly:

```css
button {
 font-size: 12px;
 background-color: white;
 border: 1px solid black;
 padding: 5px 10px;
 color: black;
}
```
The result is a succinct representation of a button in our application, as seen below.
```html
<button></button>
```
This is a common practice employed by frameworks such as Twitter Bootstrap.

However, we should be careful when introducing this new "component" into our application’s CSS language.

Before creating a new component in our CSS language, the UI should clearly distinguish itself from all others within our application and be common enough that the need for succinct reference becomes apparent to the team.

### Overrides: The Problem with Inheritance
When we discover the following pattern of HTML has become very common
```html
<button class=“background-red border-color-red color-white”></button>
```
we have the option to create what we'll call a "modifier."
```css
button.btn-danger {
 background-color: red;
 border-color: red;
 color: white;
}
```
This returns us to a succinct representation of a "danger" button in our application.
```html
<button class=“btn-danger”></button>
```
Introducing a new modifier has the potential to make our CSS less maintainable. This is because it is difficult for the engineer reading the HTML to know the combination of rules this modifier contains.

Nevertheless, we’ve created a modifier that allows for concise representation of a reoccurring UI pattern in our application. We’re feeling a sense of accomplishment.

Inevitably, more exceptions arise however.
```html
<button class=”btn-danger fsize-24 min-w-400 pad-10–20"></button>
```
So, again we create a modifier. This time however, it's technically a sub-modifier because it's dependent on the button tag as it's foundation.
```css
button.btn-large {
 font-size: 24px;
 min-width: 400px;
}
```
```html
<button class=”btn-danger btn-large”></button>
```
Again, when viewing this HTML, it is difficult for an engineer to know which set of rules the .btn-large modifier owns and which are owned by .btn-danger.
Do the two classes share rules? Do they conflict? If so, which one wins?
He could look at the stylesheet, but if he needs to override the appearance, he’s more likely to simply append a utility and move on.

The preceding button scenario describes what Building and shipping functional CSS describes as a component:
> "Components: Content-specific classes that generate distinct UI appearances."

I believe we have to be disciplined about what we define as components within our CSS language.

As a team, we should define only what all buttons actually share: properties which MUST be present that CANNOT deviate. Otherwise, one can no longer refer to it a “button.”

We’ll probably find these components consist of very little.
```css
button {
 border: 1px solid;
}
```
And if we later choose to create a modifier such as .btn-danger,
```css
.btn-danger {
 color: white !important;
 border-color: red !important;
 background-color: red !important;
}
```
we must agree that the rules contained within this modifier are not meant to be overridden. We can enforce this by using `!imporant`.
If any of these rules change, we must break up the modifier and express the desired appearance as a set of utility classes or create a new modifier
If a new pattern emerges, meaning this set of utilities has become commonplace, we create a new modifier.

### Conflicts
Unless there are rules governing what properties are owned by a particular modifier, we will eventually end up clobbering the very properties which make up the modifier itself.
Allowing an engineer to change any of these introduces exceptions into the system which will lead confusion and maintenance issues.

# The System
## Modifiers: a better approach through categorization
### Prefixing
To signal this to the engineer, we can define different types of modifiers. For example, “.btn-danger” can be a color modifier, as it only defines a set of colors. So instead, we can prefix it with .c-
```css
button.c-danger {
 background-color: red !important;
 border-color: red !important;
 color: white !important;
}
```
.btn-large can be differentiated as a scaling modifier and prefixed with .s-.
```css
button.s-large {
 font-size: 24px !important;
 min-width: 400px !important;
}
```
The result is the following succinct expression:
```html
<button class="c-danger s-large”></button>
```
### Modifiers
It must be documented which rules are available to a particular modifier. We must categorize every possible CSS rule.
If a color modifier is present on an HTML element, the engineer cannot use a color utility to override any of the rules available to that category of modifier.
He must remove the modifier and use a functional approach.
If a new pattern emerges, meaning this set of utilities has become commonplace, he's free to create a new modifier.
### Coexistence
This approach allows for utilities to coexist with modifiers - a mix of Composition and Inheritance.
```html
<button class=”c-danger s-large pad-20 mar-15"></button>
```
### Conditions: Platform, DOM placement, and State

It’s much easier to manage conditions such as media queries in Functional CSS. However, if we are working with a modifier, rather than using a Functional approach, we can leverage a media query modifier: .mq-
This is an override for a particular modifier with a media query modifier which despite the use of !important in both place, will win.
```html
<button class=”c-danger s-large” data-mqs=”mqs-s-large”></button>
```
```css
button.mqs-s-large {
 font-size: 18px !important;
}
```
The preceding button leverages the rules of .s-large, however it also employs an override for that particular modifier on small screens .mqs-s-large. Most importantly, this override is optional.
### Strict Enforcement
I had been debated on whether a utility or modifier would take precedence.
Traditionally the utility would win by using !important on all utilities. This seems obvious because the name is explicit: `.inline-block`
> “I want this one rule to be expressed. No ‘buts’ about it.”

However, this approach encourages engineers to be reckless and requires no prerequisite understanding of the modifier he is overriding.

This can quickly lead to engineers leveraging modifiers such as `.c-danger` and later overriding the majority, perhaps even all of the properties held within that modifier as time goes by.

Based on my experience, this is likely done because the engineer didn’t bother to reference the modifier in the stylesheet beyond the few properties that he needed to override.

Often the engineer might open a file to discover several utilities already present on the element.
In this system there is a heirarchy. Modifiers take precedence over utilities. This is the opposite of the framework approach.

### Example
Consider a case in which two engineers over the course of five months added four utilities have been on top of a modifier within an HTML element.

In this scenario, no single person is accountable for determining which modifier rules are being expressed and which are being overridden by a utility at this point.

At this point, the original modifier may have little, perhaps even no properties in play.

Who’s to know this? Who is going to actually take the time to investigate? The engineer who opens the file today? Most likely not. He’ll probably just add another utility.

In this paradigm, classes just continue to get added to HTML with no accountability.

By adding !important to all of the rules housed by the modifier, rules can be enforced.

If an engineer adds a sizing property and a sizing modifier is present, the modifier's !important won’t allow for the engineer to clobber it.

If he wants to change something about that element that conflicts with the modifier, he needs to remove that modifier and express the desired styling in utilities.

If a new pattern emerges where this set of utilities has now also become commonplace, he’s free to define a new modifier whose rules again will be strictly enforced using !important.

Note: We should keep an eye out for potential drawbacks to this approach as well. I’m sure there are several. However, I think it will help bring order to what may be a chaotic CSS landscape.

## Things to avoid:
### Masquerading
Avoid it. In fact, do not even name your components after HTML tags. This introduces likelihood for the following:
Masquerading HTML Elements
```html
<h3 class=“h1”></h3>
<li class=“button”></li>
<button class=“link”></button>
```
Which is why in the prior examples I chose to use the button tag as the base rather than creating a .btn component. This forces us to only use buttons when they are actually buttons.

### Large Components
Components which wrap large sections of HTML within the app (i.e. summary list, dialog, etc.)
This is just the same set of concerns on a larger scale: HTML, CSS, and JavaScript.

The more complex our component’s HTML structure becomes, the more opportunities arise for exceptions.

### Context and Conditions
Platform, DOM placement, and State all allow for variation from the default composition of your class. It is easy to run into conflicts when rules are conditional.

## Keep In Mind
Lastly, be aware that since all HTML elements have access to the same set of CSS rules, the line between your “components” in your CSS language can become blurred unless strictly defined amongst your team.
