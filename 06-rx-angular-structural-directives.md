# Exercise 6: Rx-angular structural directives

We are already fully zone-less however we need to fix some more issues.
In this exercise you will see how we can fix `async` pipe issue and make a huge performance improvements with rx-angular structural directives.

This improvements focusing on TBT metric. Especially big impact has `rxFor` directive that reduces this metric drastically.

## Fix application shell

First we will fix our application shell.

We need to add 2 imports to `app-shell.module.ts`:

```typescript
// Exercise 6: Add ForModule & LetModule imports here

import { LetModule } from "@rx-angular/template/let";
import { ForModule } from "@rx-angular/template/experimental/for";
```

Provide them:

```typescript
  imports: [
    CommonModule,
    RouterModule,
    HamburgerButtonModule,
    SideDrawerModule,
    SearchBarModule,
    DarkModeToggleModule,
    LazyModule,
    SvgIconModule,
    // Exercise 6: Add ForModule & LetModule here
    ForModule,
    LetModule,
  ],
```

Replace top level async pipe with `rxLet` directive in `app-shell.component.html`:

```
<!-- Exercise 6: Replace it with rxLet  directive -->
<ng-container *rxLet="viewState$; let vs">
```

Replace `ngFor` with `rxFor`:

```html
<!-- Exercise 6: Replace ngFor with rxFor -->
<a
  [attr.data-uf]="'menu-gen-' + genre.id"
  *rxFor="let genre of genres$; trackBy: trackByGenre"
  class="navigation--link"
  [routerLink]="['/list', 'genre', genre.id]"
  routerLinkActive="active"
>
  <div class="navigation--menu-item">
    <svg-icon class="navigation--menu-item-icon" name="genre"></svg-icon>
    {{ genre.name }}
  </div>
</a>
```

As a result of this replacements we enabled local change detection in app shell and enabled chunked rendering for genres list.

## Fix movie list page

Add `LetModule` to `movie-list-page.module.ts`:

```typescript
// Exercise 6: Add LetModule import here

import { LetModule } from "@rx-angular/template/let";
```

Add it to the imports:

```typescript
  imports: [
    CommonModule,
    RouterModule.forChild(ROUTES),
    MovieListModule,
    IfModule,
    LetModule,
  ],
```

Now we need to refactor a template.

Go to `movie-list-page.component.html` and refactor header:

```html
<!-- Exercise 6: Refactor header -->

<header *rxLet="headings$; let heading">
  <h1 data-uf="header-main">{{ heading.main }}</h1>
  <h2 data-uf="header-sub">{{ heading.sub }}</h2>
</header>
```

Remove `async` pipe in `[movies]` input, pass observable directly:

```html
<!-- Exercise 6: Pass observable directly -->

<ui-movie-list (paginate)="paginate()" [movies]="movies$"> </ui-movie-list>
```

Refactor loader:

```html
<!-- Exercise 6: Refactor loader -->

<div *rxLet="loading$" class="loader"></div>
```

### Pro tip

In zone.js application we can reduce 1 CD cycle by passing observable directly.

Usual way of passing observable to child component is next:

```html
<ui-movie-list (paginate)="paginate()" [movies]="movies$ | async">
</ui-movie-list>
```

This means that global change detection will kick in 2 times:

- Form `async` pipe
- When value will be evaluated by child component

We can remove 1 CD cycle if we pass observable directly to child and subscribe to it inside with `rxLet`.

## Fix movie-list component

We need to add 2 imports to `movie-list.module.ts`:

```typescript
// Exercise 6: Add ForModule & LetModule imports here

import { LetModule } from "@rx-angular/template/let";
import { ForModule } from "@rx-angular/template/experimental/for";
```

Provide them:

```typescript
  imports: [
    CommonModule,
    RouterModule,
    StarRatingModule,
    ElementVisibilityModule,
    SvgIconModule,
    GridListModule,
    IfModule,
    ForModule,
    LetModule
  ],
```

Now we will fix top level observable in `movie-list.component.ts` with `rxLet` and suspense template.

Go to `movie-list.component.ts` and do the following:

```html
<ui-grid-list *rxLet="moviesListVisible$; rxSuspense: noData">
  ...
</ui-grid-list>
```

Replace `ngFor` with `rxFor`:

```html
<!-- Exercise 6: Replace ngFor with rxFor -->
<a
  class="ui-grid-list-item"
  *rxFor="
          let movie of movies$;
          index as idx;
          trackBy: trackByMovieId
        "
  [routerLink]="['/detail/movie', movie.id]"
  [attr.data-uf]="'movie-' + idx"
></a>
```

## Measuring results

Now you can go to the browser and measure the end results.
