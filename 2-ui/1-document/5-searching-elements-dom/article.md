# Searching: getElement*, querySelector* and others

DOM navigation properties are great when elements are close to each other. What if they are not? How to get an arbitrary element of the page?

There are additional searching methods for that.
[cut]

## document.getElementById or just id

If an element has the `id` attribute, then there's a global variable by the name from that `id`.

We can use it to access the element, like this:

```html run
<div id="*!*elem*/!*">
  <div id="*!*elem-content*/!*">Element</div>
</div>

<script>
  alert(elem); // DOM-element with id="elem"
  alert(window.elem); // accessing global variable like this also works

  // for elem-content the name has dash inside, so the "simple" variable access won't work
  alert(window['elem-content']); // but still possible to access with [...]
</script>
```

The behavior is described [in the specification](http://www.whatwg.org/specs/web-apps/current-work/#dom-window-nameditem), but it is supported mainly for compatibility, because relies on global variables. The browser tries to help us by mixing namespaces of JS and DOM, but there may be conflicts.

The better alternative is to use a special method `document.getElementById(id)`.

For instance:

```html run
<div id="*!*elem*/!*">
  <div id="*!*elem-content*/!*">Element</div>
</div>

<script>
  let elem = document.getElementById('elem');

  elem.style.background = 'red';
</script>
```

```smart header="There can be only one"
By the specification the value of `id` must be unique. There can be only one element in the document with the given `id`.

If there are multiple elements with the same `id`, then the behavior is unpredictable. The browser may return any of them at random. So please stick to the rule and keep `id` unique.
```

I will often use `id` to directly reference an element in examples, but that's only to keep things short. In real life `document.getElementById` is definitely the preferred method.

## getElementsBy*

There are also other methods of this kind:

`elem.getElementsByTagName(tag)` looks for elements with the given tag and returns the collection of them. The `tag` can also be `*` for any.

For instance:
```js
// get all divs in the document
let divs = document.getElementsByTagName('div');
```

Unlike `getElementById` that can be called only on `document`, this method can look inside any element.

Let's find all `input` inside the table:

```html run height=50
<table id="table">
  <tr>
    <td>Your age:</td>

    <td>
      <label>
        <input type="radio" name="age" value="young" checked> less than 18
      </label>
      <label>
        <input type="radio" name="age" value="mature"> from 18 to 50
      </label>
      <label>
        <input type="radio" name="age" value="senior"> more than 60
      </label>
    </td>
  </tr>
</table>

<script>
*!*
  let inputs = table.getElementsByTagName('input');
*/!*

  for (let input of inputs) {
    alert( input.value + ': ' + input.checked );
  }
</script>
```

```warn header="Don't forget the `\"s\"` letter!"
Novice developers sometimes forget the letter `"s"`. That is, they try to call `getElementByTagName` instead of <code>getElement<b>s</b>ByTagName</code>.

The `"s"` letter is only absent in `getElementById`, because we look for a single element. But `getElementsByTagName` returns a collection.
```

````warn header="It returns a collection, not an element!"
Another widespread novice mistake is to write like:

```js
// doesn't work
document.getElementsByTagName('input').value = 5;
```

That won't work, because we take a *collection* of inputs and assign the value to it, rather to elements inside it.

We should either iterate over the collection or get an element by the number, and then assign, like this:

```js
// should work (if there's an input)
document.getElementsByTagName('input')[0].value = 5;
```
````

Other methods:

- `document.getElementsByName(name)` returns elements with the given `name` attribute. Rarely used.
- `elem.getElementsByClassName(className)` returns elements that have the given CSS class. Elements may have other classes too.

For instance:

```html run height=50
<form name="my-form">
  <div class="article">Article</div>
  <div class="long article">Long article</div>
</form>

<script>
  // find by name attribute
  let form = document.getElementsByName('my-form')[0];

  let articles = form.getElementsByClassName('article');
  alert( articles.length ); // 2, will find both elements with class "article"
</script>
```

## querySelectorAll [#querySelectorAll]

Now the heavy artillery.

The call to `elem.querySelectorAll(css)` returns all elements inside `elem` matching the given CSS selector. That's the most often used and powerful method.

Here we look for all `<li>` elements that are last children:

```html run
<ul>
  <li>The</li>
  <li>test</li>
</ul>
<ul>
  <li>has</li>
  <li>passed</li>
</ul>
<script>
*!*
  let elements = document.querySelectorAll('ul > li:last-child');
*/!*

  for (let elem of elements) {
    alert(elem.innerHTML); // "test", "passed"
  }
</script>
```

```smart header="Can use pseudo-classes as well"
Pseudo-classes in the CSS selector like `:hover` and `:active` are also supported. For instance, `document.querySelectorAll(':hover')` will return the collection with elements that the pointer is  over now (in nesting order: from the outmost `<html>` to the most nested one).
```

## querySelector [#querySelector]

The call to `elem.querySelector(css)` returns the first element for the given CSS selector.

In other words, the result is the same as `elem.querySelectorAll(css)[0]`, but that's looking for *all* elements and picking the first, much slower than only looking for the first one.

## matches

Previous methods were searching the DOM.

The [elem.matches(css)](http://dom.spec.whatwg.org/#dom-element-matches) does not look for anything, it merely checks if `elem` fits the CSS-selector. It returns `true` or `false`.

The method comes handy when we are walking over elements (like in array or something) and trying to filter those that interest us.

For instance:

```html run
<a href="http://example.com/file.zip">...</a>
<a href="http://ya.ru">...</a>

<script>
  for (let elem of document.body.children) {
*!*
    if (elem.matches('a[href$="zip"]')) {
*/!*
      alert("The archive reference: " + elem.href );
    }
  }
</script>
```

## closest

The method `elem.closest(css)` looks the nearest ancestor (parent or his parent and so on) that matches the CSS-selector. The `elem` itself is also included in the search.

In other words, the method `closest` goes up from the element and checks each of them. If it matches, then stops the search and returns it.

For instance:

```html run
<ul>
  <li class="chapter">Chapter I
    <ul>
      <li class="subchapter">Chapter <span class="num">1.1</span></li>
      <li class="subchapter">Chapter <span class="num">1.2</span></li>
    </ul>
  </li>
</ul>

<script>
  let numSpan = document.querySelector('.num');

  // nearest li
  alert(numSpan.closest('li').className) // subchapter

  // nearest chapter
  alert(numSpan.closest('.chapter').className) // chapter

  // nearest span
  // (the numSpan is the span itself, so it's the result)
  alert(numSpan.closest('span') === numSpan) // true
</script>
```

## Live collections

All methods `getElementsBy*` return a *live* collection. They always reflect the current state of the document.

For instance, here in the first script the length is `1`, because the browser only processed the first div. Then later it's 2:

```html run
<div>First div</div>

<script>
  let divs = document.getElementsByTagName('div');
  alert(divs.length); // 1
</script>

<div>Second div</div>

<script>
*!*
  alert(divs.length); // 2
*/!*
</script>
```

In contrast, `querySelectorAll` returns a *static* collection. It's like a fixed array of elements.

If we use it instead, then both scripts output `1`:


```html run
<div>First div</div>

<script>
  let divs = document.querySelectorAll('div');
  alert(divs.length); // 1
</script>

<div>Second div</div>

<script>
*!*
  alert(divs.length); // 1
*/!*
</script>
```

## Summary

There are 6 main methods to search for nodes in DOM:

<table>
<thead>
<tr>
<td>Method</td>
<td>Finds by...</td>
<td>Can call on element?</td>
<td>Live?</td>
</tr>
</thead>
<tbody>
<tr>
<td><code>getElementById</code></td>
<td><code>id</code></td>
<td>-</td>
<td>-</td>
</tr>
<tr>
<td><code>getElementsByName</code></td>
<td><code>name</code></td>
<td>-</td>
<td>✔</td>
</tr>
<tr>
<td><code>getElementsByTagName</code></td>
<td>tag or <code>'*'</code></td>
<td>✔</td>
<td>✔</td>
</tr>
<tr>
<td><code>getElementsByClassName</code></td>
<td>class</td>
<td>✔</td>
<td>✔</td>
</tr>
<tr>
<td><code>querySelector</code></td>
<td>CSS-selector</td>
<td>✔</td>
<td>-</td>
</tr>
<tr>
<td><code>querySelectorAll</code></td>
<td>CSS-selector</td>
<td>✔</td>
<td>-</td>
</tr>
</tbody>
</table>

Please note that methods `getElementById` and `getElementsByName` can only be called in the context of the document: `document.getElementById(...)`. Other methods can be called on elements, like `elem.querySelectorAll(...)` -- and will search in their subtrees.

Besides:

- There is `elem.matches(css)` to check if `elem` matches the given CSS selector.
- There is `elem.closest(css)` to look for a nearest ancestor that matches the given CSS-selector. The `elem` itself is also checked.