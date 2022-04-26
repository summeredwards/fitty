# Fitty Text Resizing

Scales text so it fits to its parent container. Ideal for flexible and responsive websites.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/rikschennink/fitty/blob/gh-pages/LICENSE)
[![npm version](https://badge.fury.io/js/fitty.svg)](https://badge.fury.io/js/fitty)

## Installation

Install Webpack and Fitty from npm:

```
npm install --save-dev webpack webpack-cli webpack-dev-serve fitty
```
```
npm install
```

If you run into errors like "ReferenceError: require is not defined" or "Cannot use import statement outside a module" when including fitty into your .js file, **[Download](https://github.com/rikschennink/fitty)** `dist/fitty.min.js` from Rik's Fitty Git Repo and include the script on your page like shown below.

```html
<body>
<script src="fitty.min.js"></script>
</body>
```

## Usage

Create an HTML document like normal. Choose the parent container that houses text that you want to be responsive. For example, `<div>` with `id="my-element"` can be passed into the `fitty()` function in your .js file. 
Use the example below to help you get started. Include any .js scripts before the closing `</body>` element.

In your HTML:
```html
<div id="my-element">Responsive Text</div>

<script src="fitty.min.js"></script>
```

In your .js file:
```javascript
fitty('#my-element');
```

You can pass the following into the `fitty` method.

| Option             | Default                | Description                                                                                                                                                                                                                                                                                       |
| ------------------ | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `minSize`          | `16`                   | The minimum font size in pixels                                                                                                                                                                                                                                                                   |
| `maxSize`          | `512`                  | The maximum font size in pixels                                                                                                                                                                                                                                                                   |
| `multiLine`        | `true`                 | Wrap lines when using minimum font size.                                                                                                               

If you want to set a min or max font size, here is the syntax:

```javascript
fitty('#my-element', {
    minSize: 24,
    maxSize: 250,
});
```

If you are using a class to define multiple fitty elements (ex: `<div class="fit">`), calling `var fitties = fitty('.fit');` will return every fitty element with class `.fit` in an array. If you pass a single element reference, the function will return a single fitty instance. 

The following methods can be used on a fitty instance.

| Method          | Description                                                                        |
| --------------- | ---------------------------------------------------------------------------------- |
| `fit()`         | Force a redraw of the current fitty element                                        |
| `freeze()`      | No longer update this fitty on changes                                             |
| `unfreeze()`    | Resume updates to this fitty                                                       |
| `unsubscribe()` | Remove the fitty element from the redraw loop and restore it to its original state |

| Properties | Description                      |
| ---------- | -------------------------------- |
| `element`  | Reference to the related element |

If fitty returns an array, you can select a single element within the array using `.element`; for example `var fittyElement = fitties[0].element;`. If fitty returns a single fitty instance, you do not need to use `.element`. Use the example below to better understand what these methods would look like in your .js file. 

```javascript
// fitty returns an array of fitty instances and sets it to var fitties
var fitties = fitty('.fit');

// get first fitty element from array
var fittyElement = fitties[0].element;

// force refit 
fitties[0].fit();

// stop updating this fitty and return to original state
fitties[0].unsubscribe();
```

You can also call `fitty.fitAll()` if you want all fitty instances to match their parent containers. This is similar to the `fit()` method above, but instead of only redrawing one fitty, you are requesting to redraw all fitties. 

| Method           | Description                                                                                               |
| ---------------- | --------------------------------------------------------------------------------------------------------- |
| `fitty.fitAll()` | Refits all fitty instances to match their parent containers. Essentially a request to redraw all fitties. |

## HTML

If you want to add any sort of horizontal padding or margins to your fitty element, you will need to wrap it within another element. Otherwise Fitty's width calculation between the text and parent containers will be incorrect. 

In your HTML:
```html
<div class="styling">
    <div><h1 class="fit">I'm a fitty text</h1></div>
</div>
```
In your CSS;
```css
.styling{
    margin: 0 auto;
    padding-left: 10px;
    padding-right: 10px;
}
```

I assume you want your website to be thoroughly styled using CSS. I suggest that every fitty element should be wrapped within another element like `<div>`. Like shown above, that `<div>` can be given the class `.styling`. Use `.styling` in your CSS to style your fitty element and NOT your fitty element class (like `.fit` above).

## CSS

Assume all fitty instances are housed within class `.fit`. Fitty instances within class `.fit` should have the following in your CSS file. Doing so will help Fitty perform better.

```css
.fit {
    display: inline-block;
    white-space: nowrap;
}
```

### Do Not Upscale Text

If you do not want Fitty to upscale certain text, you can do either of the following:

```css
.dontUpscale{
    display: block !important;
}
```

or 

```css
.dontUpscale{
    width: 100%;
}
```

Since Fitty works by calculating the width between the text and parent container, setting the width of the text container to be equal to its parent container will cause Fitty to not upscale the text. 

NOTE: If you want your text to upscale but it is failing to do so, double check if the text container width equal to its parent container. 


## Fonts

If you want to use custom fonts, using Google fonts or Adobe fonts is preferred. Importing fonts this way would look something like this `@import url("https://use.typekit.net/blank.css");`, and would not need any special redraw from Fitty. These fonts would work as normal. However, if you want to load your own fonts into your CSS file, redrawing Fitty text will be required. 

If you need to use fitty on browsers that don't have [CSS Font Loading](http://caniuse.com/#feat=font-loading) support (Edge and Internet Explorer) use [FontFaceObserver by Bram Stein](https://github.com/bramstein/fontfaceobserver).

An example of custom font implementation is below. Assume fitty has been called on all elements with class name `fit`.

In your CSS:
```css
/* Only set the custom font if it's loaded and ready */
    .fonts-loaded .fit {
        font-family: 'Oswald', serif;
    }
```

In your .js file:
```javascript
(function () {
    // no promise support (<=IE11)
    if (!('Promise' in window)) {
        return;
    }

    // called when all fonts loaded
    function redrawFitty() {
        document.documentElement.classList.add('fonts-loaded');
        fitty.fitAll();
    }

    // CSS Font Loading API
    function native() {
        // load our custom Oswald font
        var fontOswald = new FontFace('Oswald', 'url(assets/oswald.woff2)', {
            style: 'normal',
            weight: '400',
        });
        document.fonts.add(fontOswald);
        fontOswald.load();

        // if all fonts loaded redraw fitty
        document.fonts.ready.then(redrawFitty);
    }

    // FontFaceObserver
    function fallback() {
        var style = document.createElement('style');
        style.textContent =
            '@font-face { font-family: Oswald; src: url(assets oswald.woff2) format("woff2");}';
        document.head.appendChild(style);

        var s = document.createElement('script');
        s.src =
            'https://cdnjs.cloudflare.com/ajax/libs/fontfaceobserver/2.0.13/fontfaceobserver.standalone.js';
        s.onload = function () {
            new FontFaceObserver('Oswald').load().then(redrawFitty);
        };
        document.body.appendChild(s);
    }

    // Does the current browser support the CSS Font Loading API?
    if ('fonts' in document) {
        native();
    } else {
        fallback();
    }
})();
```

## License

MIT
