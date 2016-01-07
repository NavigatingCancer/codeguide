# (S)CSS Codeguide
> “Every line of code should appear to be written by a single person, no matter the number of contributors.” -@mdo

## Table Of Contents
- [Goals](#goals)
- [Principles](#principles)
  - [BEM](#bem)
  - [Do Not](#do-not)
  - [Spacing](#spacing)
  - [Formatting](#formatting)
- [Sass Specifics](#sass-specifics)
  - [Internal Order Of An .scss File](#internal-order-of-an-scss-file)
  - [Variables](#variables)
  - [Color](#color)
  - [Experiments](#experiments)
  - [Rule Ordering](#rule-ordering)
  - [Nesting](#nesting)
- [Separation Of Concerns](#separation-of-concerns-one-thing-well)
- [(S)CSS ProTips](#scss-protips)
  - [margin-top](#margin-top)
  - [Shorthand Properties](#shorthand-properties)

## Goals
- Keep stylesheets maintainable
- Keep code transparent, sane, and readable
- Keep stylesheets scalable
- Minimize side effects

## Principles

### BEM

**Block:** Unique, meaningful names for a logical unit of style. Avoid excessive shorthand.
- Good: `.alert-box` or `.recents-intro` or `.button`
- Bad: `.feature` or `.content` or `.button`

**Element:** styles that only apply to children of a block. Elements can also be blocks themselves. Class name is a concatenation of the block name, two underscores and the element name. Examples:
- `.alert-box__close`
- `.expanding-section__section`

**Modifier:** override or extend the base styles of a block or element with modifier styles. Class name is a concatenation of the block (or element) name, two hyphens and the modifier name. Examples:
- `.alert-box--success`
- `.expanding-section--expanded`

Absolutely every class in a project fits into one of these categories, which is why BEM is so great—it’s incredibly simple and straightforward.

The point of BEM is to give a lot more transparency and clarity in your markup. BEM tells developers how classes relate to each other, which is particularly useful in complex or deep pieces of DOM. For example, if I were to ask you to delete all of the user-related classes in the following chunk of HTML, which ones would you get rid of?

```html
<div class="media  user  premium">
  <img src="" alt="" class="img  photo  avatar" />
  <p class="body  bio">...</p>
</div>
```

Well we’d definitely start with user, but anything beyond that would have to be a guess, educated or otherwise. However, if we rewrote it with BEM:

```html
<div class="media  user  user--premium">
  <img src="" alt="" class="media__img  user__photo  avatar" />
  <p class="media__body  user__bio">...</p>
</div>
```

Here we can instantly see that user, `user--premium`, `user__photo`, and `user__bio` are all related to each other. We can also see that `media`, `media__img`, and `media__body` are related, and that `avatar` is just a lone Block on its own with no Elements or Modifiers.

This level of detail from our classes alone is great! It allows us to make much safer and more informed decisions about what things do and how we can use, reuse, change, or remove them.

#### BEM Best practices

A modifier should always be included with the base block.
- Good: `<div class="my-block my-block--modifier">`
- Bad: `<div class="my-block--modifier">`

Don't create elements inside elements. If you find yourself needing this, consider converting your element into a block.
- Bad: `.alert-box__close__button`

If you're ever confused, ask for help in the Front-end HipChat room (or ask a friend—we're all in this together).

----------

### Selector Naming

- Try to use [BEM-based](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/) naming for your class selectors
  - When using modifier classes, always require the base/unmodified class is present
- Don't use Sass’s nesting to manage BEM selectors. It makes those selectors non-searchable. Prefer something like this:
  ```scss
  .block {
    [...]
  }

  .block--modifier {
    text-align: center;
  }

  .block__element {
    color: red;
  }

  .block__element--modifier {
    color: blue;
  }
  ```

----------

### Namespaced Classes

The one thing missing from BEM is that it only tells us what classes to in relative terms, as in, how classes are related to each other. They don’t really give us any idea of how things behave, act, or should be implemented in a global and non-relative sense.

Thus, we prefix every class in a codebase with a certain string in order to explain to developers what kind of job it does. This Hungarian notation-like naming allows us to ascertain exactly what kind of job a class might have, how and where we might be able to reuse it (if at all), whether or not we can modify it, and much more.

There are a few reserved namespaces for classes to provide common and globally-available abstractions.

- `.o-`: Signify that something is a CSS Object, and that it may be used in any number of unrelated contexts to the one you can currently see it in. Making modifications to these types of class could potentially have knock-on effects in a lot of other unrelated places. Tread carefully.
- `.c-`: Signify that something is a Component. This is a concrete, implementation-specific piece of UI. All of the changes you make to its styles should be detectable in the context you’re currently looking at. Modifying these styles should be safe and have no side effects. Components are designed pieces of UI—like buttons, inputs, modals, and banners.
- `.u-`: Signify that this class is a Utility class. It has a very specific role (often providing only one declaration) and should not be bound onto or changed. It can be reused and is not tied to any specific piece of UI. Things like floating elements, trimming margins, etc. You will probably recognise this namespace from libraries and methodologies like [SUIT](https://suitcss.github.io/).
- `.is-`, `.has-`: Signify that the piece of UI in question is currently styled a certain way because of a state or condition. This stateful namespace is gorgeous, and comes from [SMACSS](https://smacss.com/). It tells us that the DOM currently has a temporary, optional, or short-lived style applied to it due to a certain state being invoked.
- `._`: Signify that this class is the worst of the worst—a hack! Sometimes, although incredibly rarely, we need to add a class in our markup in order to force something to work. If we do this, we need to let others know that this class is less than ideal, and hopefully **temporary** (i.e. do not bind onto this).
- `.js-`: Signify that this piece of the DOM has some behaviour acting upon it, and that JavaScript binds onto it to provide that behaviour. If you’re not a developer working with JavaScript, leave these well alone.

(This list is to get us started. In the future, we may find the need for another type of namespace.)

From our previous example:

```html
<div class="media  user  user--premium">
  <img src="" alt="" class="media__img  user__photo  avatar" />
  <p class="media__body  user__bio">...</p>
</div>
```

We now have:

```html
<div class="o-media  c-user  c-user--premium">
  <img src="" alt="" class="o-media__img  c-user__photo  c-avatar" />
  <p class="o-media__body  c-user__bio">...</p>
</div>
```

----------

### Avoid HTML tags

- Avoid using HTML tags in CSS selectors.
  - E.g. Prefer `.o-modal {}` over `div.o-modal {}`.
  - Always prefer using a class over HTML tags (with some exceptions like CSS resets and base styles).

For example:

```scss
div.sidebar .login-box a.btn span {
}
```

In this compound selector, the subject is `span`, and the conditions are `IF (inside .btn) AND IF (on a) AND IF (inside .login-box) AND IF (inside .sidebar) AND IF (on div)`.

That is to say, every component part of a selector is an `if` statement—something that needs to be satisfied (or not) before the selector will match.

This subtle shift in the way we look at how we write our selectors can have a huge impact on their quality. Would we really ever write (pseudo code):

```scss
@if exists(span) {

  @if is-inside(.btn) {

    @if is-on(a) {

      @if is-inside(.login-box) {

        @if is-inside(.sidebar) {

          @if is-on(div) {

            # Do this.

          }

        }

      }

    }

  }

}
```
Probably not. That seems so indirect and convoluted. We’d probably just do this:

```scss
@if exists(.btn-text) {

  # Do this.

}
```

Every time we nest or qualify a selector, we are adding another if statement to it. This in turn increases its Cyclomatic Complexity.


----------

### No IDs allowed for CSS

- Don’t use IDs in selectors.
  - `#header` is overly specific compared to, for example `.header` and is much harder to override.
  - Read more about the headaches associated with IDs in CSS [here](http://csswizardry.com/2011/09/when-using-ids-can-be-a-pain-in-the-class/).

----------

### Specificity Wars

- Don’t `!important`.
  - Ever.
  - If you must, leave a comment, and prioritize resolving specificity issues before resorting to `!important`.
  - `!important` greatly increases the power of a CSS rule, making it extremely tough to override in the future. It’s only possible to override with another `!important` rule later in the cascade.

----------

### Spacing

- Two spaces for indenting code
- Put spaces after `:` in property declarations
  - E.g. `color: red;` instead of `color:red;`
- Put spaces before `{` in rule declarations
  - E.g. `.o-modal {` instead of `.o-modal{`
- Write your CSS one line per rule
- Add a line break after `}` closing rule declarations
- When grouping selectors, keep individual selectors on a single line
- Place closing braces `}` on a new line
- No trailing white-space at the ends of lines
- Trim excess whitespace

### Formatting

- All selectors are lower case, hyphen separated aka “spinal case” eg. `.my-class-name`
- Always prefer Sass’s double-slash `//` commenting, even for block comments
- Avoid specifying units for zero values, e.g. `margin: 0;` instead of `margin: 0px;`
- Always add a semicolon to the end of a property/value rule
- Use leading zeros for decimal values `opacity: 0.4;` instead of `opacity: .4;`
- Put spaces before and after child selector `div > span` instead of `div>span`

----------

## Sass Specifics
### Internal Order Of An .scss File

1. Local Variables
2. Base Styles
3. Experiment Styles

Example:

```scss
//------------------------------
// Modal
//------------------------------

$modal-namespace: "c-modal" !default;
$modal-padding: 32px;

$modal-background: #fff !default;
$modal-background-alt: color(gray, x-light) !default;

.c-modal { ... }

// Many lines later...

// EXPERIMENT: experiment-rule-name
.c-modal--experiment { ... }
// END EXPERIMENT: experiment-rule-name
```

### Variables

- Define all variables at the top of the file
- Namespace local variables with the filename (SASS has no doc level scope)
  - eg `business_contact.scss` →`$business_contact_font_size: 14px;`
- Local variables should be `$snake_lowercase`
- Global constants should be `$SNAKE_ALL_CAPS`

### Color

- Use the defined color constants via the color function
- Lowercase all hex values `#ffffff`
- Limit alpha values to a maximum of two decimal places. Always use a leading zero.

Example:

```scss
// Bad
.c-link {
  color: #007ee5;
  border-color: #FFF;
  background-color: rgba(#FFF, .0625);
}

// Good
.c-link {
  color: color(blue);
  border-color: #ffffff;
  background-color: rgba(#ffffff, 0.1);
}
```

### Experiments

Wrap experiment styles with comments:

```scss
// EXPERIMENT: experiment-rule-name
.stuff { ... }
// END EXPERIMENT: experiment-rule-name
```

----------

### Rule Ordering

Properties and nested declarations should appear in the following order, with line breaks between groups:

1. Any `@` rules like include
2. Layout and box-model properties
  - margin, padding, box-sizing, overflow, position, display, width/height, etc.
3. Typographic properties
  - E.g. font-*, line-height, letter-spacing, text-*, etc.
4. Stylistic properties
  - color, background-*, animation, border, etc.
5. UI properties
  - appearance, cursor, user-select, pointer-events, etc.
6. Pseudo-elements
  - ::after, ::before, ::selection, etc.
7. Pseudo-selectors
  - :hover, :focus, :active, etc.
8. Modifier classes
9. Nested elements

Here’s a comprehensive example:

```scss
.c-button {
  @include drop-shadow;

  display: inline-block;
  padding: 5px 10px;

  text-align: center;
  font-weight: 600;

  background-color: color(grey, medium);
  border-radius: 3px;
  color: color(black);

  cursor: pointer;

  &::before {
    content: '';
  }

  &:focus, &:hover {
    background-color: color(grey, dark);
  }
}
```

----------

### Nesting

- As a general rule of thumb, avoid nesting selectors more than 3 levels deep
  - Nesting selectors increases specificity, meaning that overriding any CSS set therein needs to be targeted with an even more specific selector. This quickly becomes a significant maintenance issue.
- Avoid using nesting for anything other than pseudo selectors and state selectors.
  - E.g. nesting `:hover`, `:focus`, `::before`, etc. is OK, but nesting selectors inside selectors should be avoided.

Nesting can be really easily avoided by smart class naming (with the help of BEM) and avoiding bare tag selectors.

If a nested block of Sass is longer than 50 lines, there is a good chance it doesn't fit on one code editor screen, and starts becoming difficult to understand. The whole point of nesting is convenience and to assist in mental grouping. Don't use it if it hurts that.

----------

## Separation of Concerns (One Thing Well™)

You should always try to spot common code—padding, font sizes, layout patterns—and abstract them to reusable, namespaced classes that can be chained to elements and have a single responsibility. Doing so helps prevent overrides and duplicated rules, and encourages a separation of concerns.

```scss
// Bad code
// HTML:
// <div class="modal compact">...</div>
.modal {
  padding: 32px;
  background-color: color(gray, x-light);

  &.compact {
    padding: 24px;
  }
}

// Good code
// HTML:
// <div class="c-modal u-l-large">...</div>
// <div class="c-modal u-l-medium">...</div>

// components/_modal.scss
.c-modal {
  background-color: color(gray, x-light);
}

// helpers/_layout.scss
.u-l-large {
  padding: 32px;
}

.u-l-medium {
  padding: 24px;
}
```

## (S)CSS ProTips™

### Margin-Top
- Don’t use `margin-top`.
  - Vertical margins [collapse](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing). Always prefer `padding-top` or`margin-bottom` on preceding elements.

### Shorthand Properties

- Avoid shorthand properties (unless you really need them)
  - It can be tempting to use, for instance, `background: #fff` instead of `background-color: #fff`, but doing so overrides other values encapsulated by the shorthand property. (In this case, `background-image` and its associative properties are set to “none.”
  - This applies to all properties with a shorthand: border, margin, padding, font, etc.