# Exercise 7: Refactor to RxState

In this exercise we will refactor `app-shell` component and get familiar with RxState.

## Adding RxState

First let's add RxState to our `app-shell.component.ts`:

```typescript
// Exercise 7: Add RxState import here

import { RxState } from "@rx-angular/state";
```

State is a local service, add it to component providers:

```typescript

  // Exercise 7: Add RxState and RxActionFactory here

  providers: [RxEffects, RxState],
```

Provide it in the constructor:

```typescript
 constructor(
    // Exercise 7: Add RxState here
    private state: RxState<{ sideDrawerOpen: boolean }>,
    ...
 ) {}
```

## Initializing state

Now we need to initialize our default value.

Go to `init` function and set the initial state:

```typescript
  init() {
    // Exercise 7: initialize state here
    this.state.set({sideDrawerOpen: false});
    ...
  }
```

## Setting up view model

Let's refactor our view model.

Replace `viewModel$` stream with `select` method.

```typescript
 // Exercise 7: Replace it with state.select()

  readonly viewState$ = this.state.select('sideDrawerOpen');

//   this.state$.pipe(
//     distinctUntilChanged((o, n) => o.sideDrawerOpen === n.sideDrawerOpen),
//     shareReplay({ bufferSize: 1, refCount: true })
//   );
```

Now we can remove our `state$` `BehaviorSubject`:

```typescript
// Exercise 7: Remove state$

//   private readonly state$ = new BehaviorSubject<{ sideDrawerOpen: boolean }>({
//     sideDrawerOpen: false,
//   });
```

## Setup state updates

Create a subject called `sideDrawerOpenToggle`:

```typescript
// Exercise 7: Create ui actions here

readonly sideDrawerOpenToggle = new Subject<boolean>();
```

Connect `Subject` to state in `init` method:

```typescript
init() {
    // Exercise 7: initialize state here
    this.state.set({sideDrawerOpen: false});

    // Exercise 7: connect ui here

    this.state.connect('sideDrawerOpen', this.sideDrawerOpenToggle);
    ...
  }
```

Replace `closeSidenav` call with `Subject.next`.

```typescript
// Exercise 7: replace with ui action

() => this.sideDrawerOpenToggle.next(false);
```

Do a cleanup:

```typescript
// Exercise 7: remove this function

//   closeSidenav = () => {
//     this.state$.next({
//       sideDrawerOpen: false,
//     });
//   };

// Exercise 7: remove this function

//   sideDrawerOpenToggle = (sideDrawerOpen: boolean) => {
//     this.state$.next({
//       sideDrawerOpen,
//     });
//   };
```

## Refactoring template

In `app-shell.component.html` replace calls of `sideDrawerOpenToggle` with `sideDrawerOpenToggle.next(!vs)`

```html
<!-- Exercise 7: replace with actions -->

<ui-side-drawer [opened]="vs" (openedChange)="sideDrawerOpenToggle.next(false)">
  ...
</ui-side-drawer>
```

```html
<!-- Exercise 7: replace with actions -->
<ui-hamburger-button
  data-uf="menu-btn"
  class="ui-toolbar--action"
  (click)="sideDrawerOpenToggle.next(!vs)"
>
  ...
</ui-hamburger-button>
```

## Bonus: Refactoring to RxActions

RxActions is another tool provided by rx-angular.
Main goal is to simplify triggers that causing state updates or side effects.

Add `RxActionFactory` import:

```typescript
// Exercise 7: Add RxActionFactory import here

import { RxActionFactory } from "../shared/rxa-custom/actions";
```

Add it to providers:

```typescript

  // Exercise 7: Add RxState and RxActionFactory here

  providers: [RxEffects, RxState, RxActionFactory],
```

Add actions to the constructor:

```typescript
 constructor(
    // Exercise 7: Add RxState here
    private state: RxState<{ sideDrawerOpen: boolean }>,
    // Exercise 7: Add RxActionFactory here
    private actionsF: RxActionFactory<{sideDrawerOpenToggle: boolean}>,
    ...
 ) {}
```

Replace `sideDrawerOpenToggle` with `ui` property:

```typescript
// Exercise 7: Create ui actions here

readonly ui = this.actionsF.create();
```

Connect `ui.sideDrawerOpenToggle$` to state instead of `Subject`:

```typescript
init() {
    // Exercise 7: initialize state here
    this.state.set({sideDrawerOpen: false});

    // Exercise 7: connect ui here

    this.state.connect('sideDrawerOpen', this.ui.sideDrawerOpenToggle$);
    ...
  }
```

Replace `Subject.next` with a call of `sideDrawerOpenToggle()`:

```typescript
// Exercise 7: replace with ui action

() => this.ui.sideDrawerOpenToggle(false);
```

Remove `sideDrawerOpenToggle` subject:

```typescript
// Exercise 7: Create ui actions here

// readonly sideDrawerOpenToggle = new Subject<boolean>();
```

Refactor template:

```html
<!-- Exercise 7: replace with actions -->

<ui-side-drawer [opened]="vs" (openedChange)="ui.sideDrawerOpenToggle(false)">
  ...
</ui-side-drawer>
```

```html
<!-- Exercise 7: replace with actions -->
<ui-hamburger-button
  data-uf="menu-btn"
  class="ui-toolbar--action"
  (click)="ui.sideDrawerOpenToggle(!vs)"
>
  ...
</ui-hamburger-button>
```
