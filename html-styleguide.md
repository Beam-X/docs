# HTML Style Guide

## Table of Contents

1. [Basic Rules](#basic-rules)
2. [Lean markup](#lean-markup)
3. [Proper Tags](#proper-tags)
4. [Proper Input Types](#proper-input-types)
5. [Images](#images)

## Basic Rules

- Try to use as little HTML as possible
- Use flex-boxes
- Use H1-H6 only for page/section meaningful headers (don't use as elements titles)
- Use proper input types
- Use high-density images
- Use webp images

## Lean markup

General rule is: Keep it simple;

Whenever possible, avoid superfluous parent elements when writing HTML. Try to use as little html as possible. Always ask yourself "is this element really required here?". Many times this requires iteration and refactoring, but produces less HTML.

```html
<!-- Not so great -->
<span class="avatar">
  <img src="https://github.com/github.png">
</span>
<!-- Better -->
<img class="avatar" src="https://github.com/github.png">
```

## Proper Tags

- use H1, H2, H3, etc. only for for headings for their sections.
Don't use them just to create an elements title.
  ```html
  // bad
  <div class="profileCard">
    <h2 class="profileName">John Smith</h2>
  </div>

  // good
  <div class="profileCard">
    <div class="title profileName">John Smith</div>
  </div>

  // bad
  <section>
    <div class="heading-1">Page title</div>
    ...
    <div class="heading-2">Page sub heading</div>
  </section>

  // good
  <section>
    <h1>Page title</h1>
    ...
    <h2>Page sub heading</h2>
  </section>
  ```
- use `p` for text elements
- use `ul` for lists
- use `button` for buttons and control elements
- use `a` for links
- use `nav` for navigation

## Proper Input Types

It's important to use a propoer input type cause it affects keyboard which will be used on a mobile device while filling in.

```html
// bad
<input type="text" name="email" />

// good
<input type="email" name="email" />
```

## Images

- Always use inline SVG for any icons/small images (e.g. logos)
- Always optimize PNG/JPG images with [tinypng](https://tinypng.com/)
- Always add `alt` for images
- Use `srcset`/`@media` for high res images support
  ```html
  // in styles
  .background-image {
    background-image: url(image.png);
  }
  @media 
    (-webkit-min-device-pixel-ratio: 2), 
    (min-resolution: 192dpi) {
    .background-image {
      background-image: url(image@2x.png);
    }
  }
  
  // in html
  <picture>
    <source type="image/webp" srcset="image.webp, image@2x.webp 2x" />
    <source type="image/png" srcset="image@2x.png 2x" />
    <img src="logo.png" alt="image description" />
  </picture>
  ```
- Use `webp` images with a fallback to a `png`
  ```html
  <html class="no-webp">
  ...
  <script>
  async function supportsWebp() {
    if (!self.createImageBitmap) return false
    const webpData = 'data:image/webp;base64,UklGRh4AAABXRUJQVlA4TBEAAAAvAAAAAAfQ//73v/+BiOh/AAA='
    const blob = await fetch(webpData).then(r => r.blob())
    return createImageBitmap(blob).then(() => true, () => false)
  }
  (async () => {
    if (await supportsWebp()) {
      document.documentElement.classList.remove("no-webp")
    }
  })()
  </script>
  ...
  <style>
  .background-image {
    background-image: url(image.webp);
  }
  .no-webp .background-image {
    background-image: url(image.png);
  }
  </style>
  ...
  <div class="background-image"></div>
  ...
  <picture>
    <source type="image/webp" srcset="image.webp, image@2x.webp 2x" />
    <source type="image/png" srcset="image@2x.png 2x" />
    <img src="logo.png" alt="image description" />
  </picture>
  ```
- Use `picture` tag
  ```html
  <picture>
    <source type="image/webp" srcset="image.webp, image@2x.webp 2x" />
    <source type="image/png" srcset="image@2x.png 2x" />
    <img src="logo.png" alt="image description" />
  </picture>
  ```

---

Credits: this guide is based on [Primer HTML Principles](https://primer.style/css/principles/html) + our own changes according to principles we use in our company