# Exercise 1: Optimize network with rel attributes

Network is where every frontend starts. We already have an overview of dev tools "Network" tab and possible values for `rel` attributes.
Optimizations with `rel` attributes are improving [Largest Contentful Paint (LCP) metric](https://web.dev/i18n/en/lcp/).

In this exercise you will learn how to use them. And as a bonus to additional optimizations to improve UX.

## Use `preconnect` to preemptively connect

Movies app is based on [The Movie Database (TMDB)](https://www.themoviedb.org/) API.
We for sure know that user will need many images from `https://image.tmdb.org/` and make API requests to `https://api.themoviedb.org/`.
In our movies app users fetching lot of images from the same origin - `https://image.tmdb.org/`.
We can preemptively connect to this origins using `rel="preconnect"`.

<details>
    <summary>show solution</summary>

Go to `index.original.html` and extend `<head>` tag with following:

```html
<!--Exercise 1: User preconnect here.-->

<link rel="preconnect" href="https://image.tmdb.org/" crossorigin="" />
<link rel="preconnect" href="https://api.themoviedb.org/" crossorigin="" />
```
</details>

## Use `preload` to load critical resources

This value allows browser to fetch resources that browser needs very early in the lifecycle.
Using this attribute we are making sure that requested resources are not blocking main rendering.

Apply `preload` to `link` tags with image sources.

<details>
    <summary>show solution</summary>

Go to `index.original.html` and extend `<head>` tag with following:

```html
<!-- Exercise 1: Use preload here. -->

<link rel="preload" as="image" href="/assets/images/logo.svg" />
<link rel="preload" as="image" href="assets/icons/android-chrome-192x192.png" />
```

</details>

## Use `prefetch` to preemptively fetch resources

You can use `prefetch` to make a hint to the browser that the resource will be needed for other pages.
Links with this attribute will be preemptively fetched and cached by the browser.
We need a backup image for movies that don't have a poster so let's fetch it.

Add a `link` tag which prefetches the `assets/images/no_poster_available.jpg`.


<details>
    <summary>show solution</summary>

Go to `index.original.html` and extend `<head>` tag with following:

```html
<!-- Exercise 1: Use prefetch here. -->

<link rel="prefetch" as="image" href="assets/images/no_poster_available.jpg" />
```

</details>
    
## Lazy loading images

By default images are eagerly (always) loaded. We can change it by setting attribute `loading="lazy"` to images that are not critical.
This avoids bootstrap and template evaluation time and will improve LCP metric.

Go to `movie-list.component.ts` and add `loading="lazy"` to movie poster:

```html
<!-- Exercise 3: Add lazy loading here and improve it -->

<img
  class="aspectRatio-2-3 gradient"
  [src]="movie?.imgUrl || 'assets/images/no_poster_available.jpg'"
  [width]="movie.imgWidth"
  [height]="movie.imgHeight"
  alt="poster movie"
  [title]="movie.title"
  loading="lazy"
/>
```

Our images are rendered in the list with `ngFor` directive. We can reduce layout shifts by loading first image eagerly.

To do this modify img with following:

```html
<!-- Exercise 3: Add lazy loading here and improve it -->

<img
  class="aspectRatio-2-3 gradient"
  [src]="movie?.imgUrl || 'assets/images/no_poster_available.jpg'"
  [width]="movie.imgWidth"
  [height]="movie.imgHeight"
  alt="poster movie"
  [title]="movie.title"
  [attr.loading]="idx === 0 ? '' : 'lazy'"
/>
```

### Pro tip

- `<iframe>` tag also has `loading` attribute and can be optimized in the same way

    
## Bonus inline loader style

To make an additional improvement for LCP and also reduce [Cumulative Layout Shift (CLS)](https://web.dev/i18n/en/cls/) we can inline our app launcher styles.

inline the class `.launcher` into our index.html.

<details>
    <summary>show solution</summary>

Go to `styles.scss` and remove `.launcher` styles:

```scss
/** Exercise 1: Remove .launcher styles **/

.launcher {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100%;
}
```

Go to `index.original.html` and extend `<head>` tag with following:

```html
<!-- Exercise 1: Use inline styles here -->

<style>
  .launcher {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100%;
  }
</style>
```

</details>

## Bonus use fixed size for loader image

Additional improvement can be made for CLS metric if we use fixed width for launcher image.
This will also remove flickering. Try to always stick to fixed size of images.

Search for image tags which always have fixed height and width, and apply fixed values to them.

<details>
    <summary>show solution</summary>

Go to `index.original.html` and add `width` and `height` attributes to launcher image:

```html
<!-- Exercise 1: Use fixed image size -->

<img
  width="192"
  height="192"
  src="assets/icons/android-chrome-192x192.png"
  alt="logo"
/>
```

</details>
