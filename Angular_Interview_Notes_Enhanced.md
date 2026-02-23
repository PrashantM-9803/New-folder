# Angular Interview Notes (Enhanced)

---

## What is Node.js?
- **Definition:** Node.js is a runtime built on Chrome's V8 engine that lets you run JavaScript outside the browser.
- **Key Features:** Non-blocking I/O, event-driven architecture, npm ecosystem for packages.
- **Why It Matters:** Provides tooling (npm, Angular CLI), local dev server, build process for Angular apps.

## Why Node.js is Required in an Angular Project
- Angular CLI depends on Node.js and npm to:
  - Install dependencies (npm/yarn)
  - Run dev server (`ng serve`)
  - Build and bundle (`ng build`)
  - Run tests and linting
- Node.js is **not** required at runtime in the browser, but is essential for development tooling.

## Angular — Overview
- A framework for building client-side Single Page Applications (SPAs) using TypeScript.
- Opinionated structure with powerful tooling (CLI, schematics, testing).

## Core Features of Angular
1. **Component-based architecture:** UI is composed of reusable components (view + logic).
2. **Two-way data binding:** Using `ngModel` (Forms) syncs model ⇄ view.
3. **Dependency Injection (DI):** Services provided via injectors; promotes testability.
4. **Directives:** Structural (e.g., `*ngIf`, `*ngFor`) and Attribute (e.g., `[ngClass]`, `[ngStyle]`).
5. **Routing:** Client-side navigation using `RouterModule`; supports parameters, guards, lazy loading.
6. **Reactive programming:** RxJS Observables, operators, async pipe for async data.
7. **Cross-platform:** Works in browsers; supports SSR (Angular Universal) and PWAs.

## Single Page Application (SPA)
- **Definition:** An app that loads a single HTML shell and updates the view dynamically using JavaScript.
- **How it works:**
  - Router intercepts navigation; only parts of the DOM update.
  - API calls fetch data; no full page reloads.
  - Improves perceived performance and UX.

## Angular CLI — Common Commands
- **Create app:** `ng new first-angular-app`
- **Start dev server:** `ng serve` | `ng serve --open`
- **Create component:** `ng g c my-first-component`
- **Create service:** `ng g s user`
- **Create module:** `ng g m admin --routing`
- **Build prod:** `ng build --configuration production`

## Angular Project Structure — File Meanings
- `package.json`: Project dependencies and scripts
- `angular.json`: Angular workspace configuration (build, serve, test)
- `tsconfig.json`: TypeScript compiler options
- `src/main.ts`: Entry point; bootstraps the app
- `src/index.html`: App host page with root element
- `src/styles.css/scss`: Global styles
- `src/app/`: Application source (components, modules, services)
- `environments/`: Environment-specific settings (dev/prod)

## How a Component Loads in the Browser
1. `main.ts` starts the app (bootstraps root component/module).
2. Root component renders into the root element in `index.html`.
3. Router renders component views into `<router-outlet>`.
4. Change detection updates the DOM when data changes.

## Component
- The basic building block of UI, encapsulating HTML, CSS, and TypeScript logic.

## Standalone Components
- Manage their own dependencies without needing an `@NgModule`.
- As of Angular 17/18, this is the default approach.

## Change Detection Strategy
- Keeps the view (DOM) in sync with the component’s data.
- Angular checks the entire component tree on every browser event.
- Uses Zone.js for change detection.

## Data Binding in Angular
1. **Interpolation (one-way):** `{{ expression }}`
2. **Property Binding (one-way):** `[property]="expression"` (e.g., `<img [src]="avatarUrl">`)
   - **Class Binding:** `[class.active]="isActive"` or `[ngClass]="{active: isActive}"`
   - **Style Binding:** `[style.color]="color"` or `[ngStyle]="{color: color}"`
3. **Event Binding (one-way from view to component):** `(event)="onEvent($event)"`
4. **Two-way Binding:** `[(ngModel)]="value"` (requires FormsModule)

## Directive vs Decorator
- **Directive:** Class that adds behavior to elements (e.g., `*ngIf`, `*ngFor`).
- **Decorator:** TypeScript metadata (e.g., `@Component`, `@Directive`, `@Injectable`) used by Angular to configure classes and properties.

## Angular Directives — Types and Examples
1. **Component Directive:** Has a template and styles, defined with `@Component`.
2. **Structural Directives:** Change the DOM structure (e.g., `*ngIf`, `*ngFor`, `*ngSwitch`).
3. **Attribute Directives:** Change appearance/behavior (e.g., `[ngClass]`, `[ngStyle]`, custom directives).

### Template Elements
- `ng-template`: Defines a fragment, not rendered by default.
- `ng-content`: Projects parent content into a child component.
- `ng-container`: Logical wrapper, does not render extra DOM node.

## Angular Decorators/Annotations
- `@Component`: Define component
- `@Directive`: Define custom directive
- `@Pipe`: Define transformation pipe
- `@NgModule`: Define Angular module
- `@Injectable`: Make class injectable via DI
- `@Input`: Receive data from parent
- `@Output`: Emit events to parent
- `@ViewChild`, `@ViewChildren`, `@ContentChild`, `@ContentChildren`: Query elements/components
- `@HostListener`: Listen to DOM events on the host element
- `@Inject`, `@Optional`, `@Self`, `@SkipSelf`, `@Host`, `@Attribute`: Dependency injection helpers

## Component Relationship — Input & Output
- `@Input()`: Parent → Child data binding
- `@Output()`: Child → Parent event communication via `EventEmitter`

## Template Reference Variables (#var)
- Local handle to a DOM element/component inside template.
- Syntax: `<input #username>`; usable in template: `{{username.value}}`
- With `@ViewChild`: `@ViewChild('username') inputRef: ElementRef;`

## Angular Pipes
- **Built-in:** `date`, `currency`, `percent`, `uppercase`, `lowercase`, `slice`, `json`, `async`
- **Usage:** `{{ total | currency:'INR':'symbol' }}`
- **Custom Pipe:** `@Pipe({name: 'capitalize'}) transform(value: string): string { ... }`

## Component Initialization — Constructor vs ngOnInit
- **Constructor:** Only DI and simple defaults; `@Input` values are NOT set.
- **ngOnInit:** Runs once after inputs are set; ideal for fetching data, complex setup.
- Access `@ViewChild` in `ngAfterViewInit`, and `@ContentChild` in `ngAfterContentInit`.

## Angular Lifecycle Methods — Overview
- **ngOnChanges:** Respond to `@Input` changes
- **ngOnInit:** Called once after inputs are set
- **ngDoCheck:** Custom change detection hook
- **ngAfterContentInit:** Projected content initialized
- **ngAfterContentChecked:** Projected content checked
- **ngAfterViewInit:** View and child views initialized
- **ngAfterViewChecked:** View and child views checked
- **ngOnDestroy:** Cleanup (unsubscribe, detach listeners)

**Lifecycle Order:**
Constructor → ngOnChanges → ngOnInit → ngDoCheck → ngAfterContentInit → ngAfterContentChecked → ngAfterViewInit → ngAfterViewChecked → ngOnDestroy

**Best Practices:**
- Use `ngOnInit` for init and API calls
- Use `ngOnChanges` to react to `@Input` changes
- Use `ngAfterViewInit` for DOM/`@ViewChild` access
- Use `ngAfterContentInit` for `@ContentChild` access
- Use `ngOnDestroy` for cleanup

## Services
- Reusable code with a focused purpose, used across components.
- Easier to test and debug.
- Used for data access, business logic, etc.

## Dependency Injection (DI)
- Design pattern where a class receives its dependencies from an external source.
- Promotes loose coupling.
- **Key concepts:**
  - **Service:** Logic class (marked with `@Injectable`)
  - **Provider:** Tells Angular how to create it
  - **Injector:** Stores and manages instances
  - **Dependency:** The actual object delivered

## Angular Routing and Navigation
- **Steps:**
  1. Define routes: `const routes = [{path: 'home', component: HomeComponent}];`
  2. Add `<router-outlet>` in a host template
  3. Navigate: `<a routerLink="/home">Home</a>` or `this.router.navigate(['home'])`
- **Terms:** router, route, routes, redirectTo, routerOutlet, routerLink, RouterLinkActive, ActivatedRoute, RouterState
- **Route Parameters:** Dynamic values in the URL path (e.g., `:id`)
- **Query Parameters:** Key-value pairs after `?` (e.g., `/products?page=2&sort=desc`)
- **Nested Routes:** Render child components inside a parent’s view
- **Lazy Loading:** Load feature modules/components on demand
- **Wildcard Routes:** `{ path: '**', component: NotFoundComponent }`
- **Route Guards:** Control navigation (CanActivate, CanDeactivate, Resolve, CanLoad, CanMatch)

## Angular Forms
- **Template-Driven Forms:** Quick setup, use `ngModel`, good for simple forms. Module: `FormsModule`.
- **Reactive Forms:** Explicit, scalable, testable. Module: `ReactiveFormsModule`.
  - **FormControl:** Represents a single input field.
  - **FormGroup:** Collection of FormControls.
  - **FormBuilder:** Helper service to create controls/groups concisely.

## RxJS in Angular — Observables
- **Observables:** Multiple values over time; cancellable; rich operators.
- **Promise:** Single value; not cancellable.
- **Observable APIs in Angular:** HTTP, Forms, Router, State, Async pipe
- **Operators:** `map`, `filter`, `tap`, `catchError`, `take`, etc.
- **of() vs from():**
  - `of([1,2,3])` emits `[1,2,3]` as a single value
  - `from([1,2,3])` emits `1`, then `2`, then `3`

## Subject and BehaviorSubject
- **Subject:** Multicasts values to multiple subscribers; can call `.next()`, `.error()`, `.complete()`
- **BehaviorSubject:** Stores latest value and emits it immediately to new subscribers; requires initial value

## State — Types & Approaches
- **Local state:** Component-only
- **Shared state:** Across components (services + RxJS)
- **Global state:** App-wide (NgRx/Akita/NGXS)
- **URL/router state:** Route params, query params
- **Server state:** Data on backend
- **Signals:** Reactive state primitives (Angular 16+)

## HttpClient
- Angular’s built-in HTTP API for making HTTP requests (GET/POST/PUT/PATCH/DELETE).
- Responses are Observables.

## NgRx — Quick Overview
- **Store:** Holds app state as one immutable tree.
- **Actions:** Describe what happened (type + optional payload).
- **Reducers:** Pure functions to produce new state.
- **Selectors:** Functions to read state for components.
- **Effects:** Handle side effects like API calls.
- **Benefits:** Predictable state, time-travel debugging, scalability.

---

*End of Notes*
