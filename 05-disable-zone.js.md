# Exercise 5: Disable zone.js completely

We already know how to disable zone.js partially.
Let's go wild and fully out of zone.

As in previous exercise we're focusing on LCP, TTI, TBT metrics, but impact will be much bigger.

## Switch to `noop` zone

To disable zone.js in the entire application we need to provide `noop` zone at bootstrap time.

Go to `main.ts` and add second argument to `bootstrapModule` function:

```typescript
// Exercise 5: Add {ngZone: 'noop'} as second argument
.bootstrapModule(AppModule, {ngZone: 'noop'})
```

## Remove `zone.js` import

It's hard to believe but we don't need zone.js in the application **at all**.

Let's remove `zone.js` import from `polyfills.ts`:

```typescript
// Exercise 5: Remove zone js import ;)

import "zone.js"; // Included with Angular CLI.
```

Since our polyfills are irrelevant now, we can remove them from `angular.json` `build` options:

```json
     "options": {
        "outputPath": "dist/movies/browser",
        "index": "projects/movies/src/index.original.html",
        "main": "projects/movies/src/main.ts",
        "tsConfig": "projects/movies/tsconfig.app.json",
        "inlineStyleLanguage": "scss",
        "namedChunks": true,
        "assets": [
            "projects/movies/src/favicon.ico",
            "projects/movies/src/manifest.json",
            "projects/movies/src/manifest.webmanifest",
            "projects/movies/src/assets"
        ],
        "styles": ["projects/movies/src/styles.scss"],
        "scripts": []
    },
```

Done. Our app is now running fully outside of zone.js.
However we need to fix couple of things now.

## Fix navigation

Unfortunately `zone.js` is critical for Angular `Router`.
If you try to navigate between pages now there is a high chance that navigation will not work.
Let's fix it with routing handling service that will trigger change detection when `NavigationEnd` event fires.

In `shared/zone-less` create a `zone-less-routing.service.ts` with following content:

```typescript
import { ApplicationRef, ErrorHandler, Injectable } from "@angular/core";
import { NavigationEnd, Router } from "@angular/router";
import { isZonePresent } from "./is-zone-present";
import { RxEffects } from "@rx-angular/state/effects";

/**
 * A small service encapsulating the hacks needed for routing (and bootstrapping) in zone-less applications
 */
@Injectable({
  providedIn: "root",
  deps: [RxEffects],
})
export class ZonelessRouting extends RxEffects {
  constructor(
    private router: Router,
    private appRef: ApplicationRef,
    errorHandler: ErrorHandler
  ) {
    super(errorHandler);
  }

  init() {
    /**
     * **🚀 Perf Tip:**
     *
     * In zone-less applications we have to trigger CD on every `NavigationEnd` event that changes the view.
     * This is a necessity to make it work zone-less, but does not make the app faster.
     */
    if (!isZonePresent()) {
      this.register(
        // Filter relevant navigation events for change detection
        this.router.events,
        // In a service we have to use `ApplicationRef#tick` to trigger change detection.
        // In a component we use `ChangeDetectorRef#detectChanges()` as it is less work compared to `ApplicationRef#tick` as it's less work.
        (e) => {
          if (e instanceof NavigationEnd) {
            this.appRef.tick();
          }
        }
      );
    }
  }
}
```

Go to `app.component.ts` and add import of our service:

```typescript
// Exercise 5: Import zone-les routing here.

import { ZonelessRouting } from "./shared/zone-less/zone-less-routing.service";
```

Create constructor and initialize it:

```typescript
// Exercise 5: Use zone-less routing

 constructor(private zonelessRouting: ZonelessRouting) {
       this.zonelessRouting.init();
     }
```

Another problem is that all our `async` pipes are broken because it can not work without `zone.js`.
We will fix it in next exercise.
