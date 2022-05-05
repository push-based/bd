# Exercise 2: Bootstrap optimizations

In this exercise we will see how we can improve Angular application bootstrap time.

## Schedule app bootstrap

First thing that we do is a split of application bootstrap into several tasks. This way we will improve [Total Blocking Time (TBT) metric](https://web.dev/i18n/en/tbt/).
We don't want to trigger style recalculation so we avoid `requestAnimationFrame` and use `setTimeout`.

Search for the `platformBrowserDynamic()` method call and wrap it with a `setTimeout`

<details>
    <summary>show solution</summary>

Go to `main.ts` file and wrap `platformBrowserDynamic` call into `setTimeout`:

<!-- TODO: Check ex number -->

```typescript
// Wrap platformBrowserDynamic into setTimeout
setTimeout(() =>
  platformBrowserDynamic()
    .bootstrapModule(AppModule)
    .catch((err) => console.error(err))
);
```

</details>

## Create scheduled app initializer

We continue to improve TBT metric. Angular allows us to provide one or several initialization functions.
With this approach we load data on application bootstrap time instead of component initialization.

First let's create `SCHEDULED_APP_INITIALIZER_PROVIDER`.

The provider should `provide` an `APP_INITIALIZER` with a factory that returns `() => Promise<void>`. 
You can try different scheduling techniques internally here if you like.

```ts
// default factory fn

() => (): Promise<void> =>
    new Promise<void>((resolve) => {
        setTimeout(() => resolve());
    }),
```

After creating your `SCHEDULED_APP_INITIALIZER_PROVIDER`, import it in the `app.module.ts`

<details>
    <summary>show solution</summary>

Create `chunk-app-initializer.provider.ts` near `app.module.ts` with following content:

```typescript
import { APP_INITIALIZER } from "@angular/core";

/**
 * **🚀 Perf Tip for TBT:**
 *
 * Use `APP_INITIALIZER` and an init method in data services to run data fetching
 * on app bootstrap instead of component initialization.
 */
export const SCHEDULED_APP_INITIALIZER_PROVIDER = [
  {
    provide: APP_INITIALIZER,
    useFactory: () => (): Promise<void> =>
      new Promise<void>((resolve) => {
        setTimeout(() => resolve());
      }),
    deps: [],
    multi: true,
  },
];
```

Add an import of our initializer in `app.module.ts`:

```typescript
// Exercise 2: Include app intializer import here.

import { SCHEDULED_APP_INITIALIZER_PROVIDER } from "./chunk-app-initializer.provider";
```

Provide it in `app.module.ts` providers array:

```typescript
providers: [
    ...
    // Include app intializer import here.

    SCHEDULED_APP_INITIALIZER_PROVIDER,
    ...
```

</details>

## Create global state initializer

We can also create an initializer for our application core state.
Loading of most important state pieces on application init reduces LCP and also improves [Time To Interactive (TTI) metric](https://web.dev/i18n/en/tti/).

<details>
    <summary>show solution</summary>

Near `app.module.ts` create `state-app-initializer.provider.ts` with following content:

```typescript
import { APP_INITIALIZER } from "@angular/core";
import { GenreResource } from "./data-access/api/resources/genre.resource";
import { MovieState } from "./shared/state/movie.state";
import { RouterState } from "./shared/router/router.state";
import { take } from "rxjs";

function initializeState(
  movieState: MovieState,
  routerState: RouterState,
  genreResource: GenreResource
) {
  return (): void => {
    // sideBar prefetch
    genreResource.getGenresCached().pipe(take(1)).subscribe();
    // initial route prefetch
    routerState.routerParams$
      .pipe(take(1))
      .subscribe(({ layout, type, identifier }) => {
        // default route
        layout === "list" &&
          type === "category" &&
          movieState.initialize({ category: identifier });
        // movie detail route
        layout === "detail" &&
          type === "movie" &&
          movieState.initialize({ movieId: identifier });
      });
  };
}

/**
 * **🚀 Perf Tip for LCP, TTI:**
 *
 * Use `APP_INITIALIZER` and an init method in data services to run data fetching
 * on app bootstrap instead of component initialization.
 */
export const GLOBAL_STATE_APP_INITIALIZER_PROVIDER = [
  {
    provide: APP_INITIALIZER,
    useFactory: initializeState,
    deps: [MovieState, RouterState, GenreResource],
    multi: true,
  },
];
```

Add an import of our initializer in `app.module.ts`:

```typescript
// Exercise 2: Include app intializer import here.

import { GLOBAL_STATE_APP_INITIALIZER_PROVIDER } from "./state-app-initializer.provider";
```

Provide it in `app.module.ts` providers array:

```typescript
providers: [
    ...
    // Include state intializer import here.
    GLOBAL_STATE_APP_INITIALIZER_PROVIDER,
    ...
```

</details>

## Optimize initial route

This will improve TTI and TBT metrics.

If you have routes with the same UI but different data implement it with 2 parameters instead of 2 different routes.
This saves creation-time and destruction-time of the component and also render work in the browser.

Try to replace the current route configuration for `MovieListPageComponent` so that it can re-use a single route
instead of having it configured twice.

You should visit `app.routing.ts`. Change the `list` routing configuration so that it accepts `:type/:indefier` as arguments.

<details>
    <summary>show solution</summary>

Go to `app.routing.ts` and replace this routes with single one:

```typescript

    // Replace next 2 routes

    // {
    //     path: 'list/category/:category',
    //     component: MovieListPageComponent,
    // },
    // {
    //     path: 'list/genre/:genre',
    //     component: MovieListPageComponent,
    // }
    {
        path: 'list/:type/:identifier',
        component: MovieListPageComponent,
    },
```

</details>

## Optimize router bootstrap performance

Initially router doing a sync initial navigation.
To improve TBT we can disable this behavior by adding `initialNavigation: 'disabled'` as configuration
for our `RouterModule.forRoot` config.

<details>
    <summary>show solution</summary>

Go to `app.routing.ts` and extend `RouterModule.forRoot()` with following:

```typescript
  RouterModule.forRoot(ROUTES, {
    enableTracing: false,

    // Disable route initial navigation here.

    initialNavigation: 'disabled',
    ...
```

</details>

However app should perform initial navigation anyway, so we should schedule it in router-outlet wrapper component.
In our case it is in `app-shell.component.ts`.

Use any scheduling technique (`setTimeout`, ....) in order to schedule the function
`fallbackRouteToDefault` inside of the `AppShellComponents` constructor.

The utility function can be found in this file `routing-default.utils.ts`.

<details>
    <summary>show solution</summary>

Add import of routing utility function:

```typescript
// Exercise 2: Add fallback util import here

import { fallbackRouteToDefault } from "../routing-default.utils";
```

Extend constructor with following:

```typescript
// Schedule navigation here

setTimeout(() =>
  this.router.navigate([fallbackRouteToDefault(document.location.pathname)])
);
```

</details>
