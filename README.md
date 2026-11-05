# Lightbox Gallery Example (litelight-js)

This example builds a photo gallery of Glacier National Park. The page shows a grid of small thumbnail images. When you click one, the full-size photo opens in a **lightbox**, an overlay on top of the page.

The lightbox comes from a small JavaScript library called [litelight-js](https://litelightbox.com/). It has no dependencies and is about 10 KB. The example shows **two ways to add a third-party library to a page**:

1. Load it from a **CDN** (Content Delivery Network) on the internet.
2. Load it from a **local copy** saved in your project (called "vendoring").

## What's in this folder

```
gallery/
├── index.html            Landing page with links to both gallery versions
├── gallery-cdn.html      Gallery that loads litelight-js from unpkg.com (CDN)
├── gallery-local.html    Gallery that loads litelight-js from vendor/ (local copy)
├── styles/
│   └── main.css          Our styles: the 3-column grid and the figure boxes
├── images/
│   ├── gnp01.jpg …       Full-size photos (opened in the lightbox)
│   ├── thumb/
│   │   └── gnp01.jpg …   Small thumbnails (shown in the grid)
│   └── README.txt        Photo titles and image credits
└── vendor/
    └── litelight-1.2.0/
        └── dist/         A copy of the library's built files
            ├── lite-light.min.js       ES module version (used here)
            ├── lite-light.umd.min.js   Classic <script> version (not used here)
            ├── lite-light.min.css      The lightbox's styles
            └── lite-light.d.ts         TypeScript type definitions (documents the options)
```

## Suggested reading order

1. **index.html**: the starting point. It links to the two galleries.
2. **gallery-cdn.html**: read the comments here first. This file explains the markup in the most detail.
3. **gallery-local.html**: compare it with the CDN version. Only the two library URLs in `<head>` are different.
4. **styles/main.css**: see how the grid and the captions are laid out.

## How to run it

Open the folder with a local web server. Don't double-click the HTML file.

In VS Code, the **Live Server** extension works well: right-click `index.html` and choose **Open with Live Server**. Then use the links to open each gallery.

> **Why do I need a server?** Both galleries load the library with `<script type="module">`. Browsers refuse to load an ES module *file* from a `file://` URL for security reasons. If you double-click `gallery-local.html`, the thumbnails still appear, but clicking them won't open the lightbox. Look in the browser's DevTools Console to see the error. `gallery-cdn.html` may still work from `file://`, because it imports the module over `https://` from a server that allows it. That difference is itself a good lesson, but using a server for everything avoids the confusion.

## How it works

### 1. The markup: one `<figure>` per photo

```html
<figure>
  <figcaption>Lake McDonald</figcaption>
  <img src="images/thumb/gnp05.jpg"
       data-lightbox="images/gnp05.jpg"
       alt="Lake McDonald">
</figure>
```

| Piece | Purpose |
|---|---|
| `<figure>` | Groups an image and its caption into one self-contained unit. |
| `<figcaption>` | The visible caption for the figure. |
| `src` | The **thumbnail**, a small file so the grid loads quickly. |
| `data-lightbox` | The **full-size** image the lightbox should open. |
| `alt` | A text alternative for screen readers, also shown if the image fails to load. |

`data-lightbox` is a **custom data attribute**. HTML lets you make up any attribute that starts with `data-`. The browser ignores it, but JavaScript can read it. This is how the HTML tells the library which big image belongs to each thumbnail.

### 2. The script: one function call

```html
<script type="module">
  import { init } from 'https://unpkg.com/litelight-js@1.2.0/dist/lite-light.min.js';
  init();
</script>
```

When `init()` runs, it finds every `img[data-lightbox]` on the page and adds a click handler to each one. When you click a thumbnail, the library reads that image's `data-lightbox` value and shows the full-size photo in an overlay. Once the lightbox is open:

- **← / →** arrow keys (or swiping on a touch screen) move between photos.
- **Esc** closes the lightbox.

Because `type="module"` scripts are **deferred**, this script waits until the whole HTML document has been parsed. That's why it can sit in `<head>` and still find the `<img>` elements in `<body>`.

`init()` also accepts options. The two main ones are listed in `vendor/litelight-1.2.0/dist/lite-light.d.ts`:

```js
init({
  imageSelector: 'img[data-lightbox]',   // which elements open the lightbox
  imageUrlAttribute: 'data-lightbox'     // which attribute holds the full-size URL
});
```

### 3. The CSS: a grid of figures

In `styles/main.css`:

- `.gallery` uses `display: grid` with `grid-template-columns: 1fr 1fr 1fr` to make three equal columns, with a `gap` between cells.
- Each `figure` is a flex container with `flex-direction: column-reverse`. In the HTML the `<figcaption>` comes **before** the `<img>`, but `column-reverse` flips the visual order, so the caption appears **below** the photo.
- `figure { margin: 0 }` removes the browser's default figure margins (about 40px on the sides), which would otherwise squeeze the grid.
- `.gallery img { max-width: 100% }` keeps each thumbnail from overflowing its grid cell.

## CDN vs. local copy

| | `gallery-cdn.html` | `gallery-local.html` |
|---|---|---|
| Library URL | `https://unpkg.com/litelight-js@1.2.0/dist/...` | `./vendor/litelight-1.2.0/dist/...` |
| Setup | Nothing to download. Just paste the URL. | Download the files and add them to your project. |
| Works offline? | No | Yes |
| Depends on a third-party server? | Yes (unpkg.com) | No |
| Updating | Change the version number in the URL | Replace the files in `vendor/` |

**unpkg** serves any file from any package published on [npm](https://www.npmjs.com/). Its URL pattern is `https://unpkg.com/<package>@<version>/<file>`. Always **pin an exact version** (like `@1.2.0`) so that a new release of the library can't change or break your page without warning.

## Making Thumbnail Images

The grid shows small **thumbnails** (`images/thumb/`), and only loads a full-size photo when you open it in the lightbox. That's a big savings: the nine originals are 2048×1367 pixels and about 12 MB in total, while the nine 300×300 thumbnails are about 448 KB, roughly 4% of the size.

I made the thumbnails with [ImageMagick](https://imagemagick.org/), a free command-line image tool. Here is [my thumbnail script (GitHub gist)](https://gist.github.com/dpheitmeyer/83981e45e105eb0044d3). The core idea:

```bash
magick input.jpg -thumbnail 300x300^ -gravity center -extent 300x300 thumb/input.jpg
```

- `-thumbnail 300x300^` scales the image to *fill* a 300×300 box and strips metadata to keep the file small.
- `-gravity center -extent 300x300` crops the overflow evenly, so every thumbnail is square and the grid lines up.

## Credits

- Photos: public domain, from the [Glacier National Park photostream on Flickr](https://www.flickr.com/photos/glaciernps/with/23095638434/). Titles are listed in `images/README.txt`.
- Lightbox library: [litelight-js](https://litelightbox.com/) v1.2.0.