---
name: create a new angular component
description: "Use when creating any Angular component. Covers Angular 21+ patterns: standalone components, signals, input(), inject(), async/await with run() helper, ngModel forms, @if/@for template syntax. No OnPush, no NgModules, no UntilDestroy."
---

Based on: Angular 21+, standalone components, signals.

---

## Component types

**1. Route component** — one component, one route. Loads its own data via injected service. No `@Input`.

**2. Section component** — a named section within a page (e.g., a modal, a list block, a form panel). May receive `input()` from the parent or inject a service directly.

**3. Card / item component** — renders a single data item. Always receives data via `input()`, no service injection.

---

## Folder and file structure

Each component lives in its own folder:

```
feature-name/
  src/lib/
    feature-name.component.ts      ← standalone, inline template (preferred)
    section-name/
      section-name.component.ts
    item-card/
      item-card.component.ts
```

For all components — separate `.html` template from `.ts` component file. If there are any styles used for the component, they should be placed in separated file as well. No styles, no file.

Naming conventions:
- `page-*` prefix for sub-views of a detail page: `page-settings`, `page-details`
- No prefix for feature-level sections: `header`, `filter`, `card`
- Selector: `app-component-name`

---

## Standalone component — minimal boilerplate

```ts
import { Component, inject, signal } from '@angular/core';

@Component({
    selector: 'app-example',
    imports: [FormsModule],     // only what the template actually uses
    template: `...`,
})
export class ExampleComponent {
    // services via inject(), not constructor
    private readonly state = inject(AppStateService);
}
```

No `NgModule`. No `ChangeDetectionStrategy.OnPush` — signals handle reactivity automatically.

---

## Signals — state management

```ts
export class ApartmentsComponent {
    // local UI state
    formOpen = signal(false);
    saving = signal(false);
    error = signal('');
    editing = signal<Apartment | null>(null);

    // global app state — read directly from service signals
    readonly state = inject(AppStateService);
    // template: state.apartments(), state.loading(), state.error()
}
```

Rules:
- Local ephemeral state (modal open, loading, form errors) → `signal()` in the component
- Shared/persistent state → lives in `AppStateService` (or feature service), read directly via `state.fieldName()`
- No `@Select()`, no `store.dispatch()`, no NGXS
- No `| async` pipe — signals are read synchronously in templates: `{{ value() }}`

---

## Input signals (Angular 17+)

```ts
import { Component, input } from '@angular/core';

@Component({ ... })
export class ItemCardComponent {
    item = input.required<ItemDto>();        // required input
    user = input<Account | null>(null);      // optional with default
}
```

Use `input()` instead of `@Input()` decorator for new components.

---

## Async operations — async/await pattern

```ts
export class AuthComponent {
    busy = signal(false);
    error = signal('');

    private readonly state = inject(AppStateService);

    async login(): Promise<void> {
        if (!this.email || !this.password) {
            this.error.set('Введите email и пароль');
            return;
        }
        await this.run(() => this.state.login(this.email, this.password));
        if (!this.error()) this.router.navigate(['/']);
    }

    // reusable wrapper: sets busy, clears error, catches exceptions
    private async run(fn: () => Promise<void>): Promise<void> {
        this.error.set('');
        this.busy.set(true);
        try {
            await fn();
        } catch (err) {
            this.error.set(err instanceof Error ? err.message : 'Ошибка');
        } finally {
            this.busy.set(false);
        }
    }
}
```

- Always extract the `run()` helper when a component has multiple async operations
- No Observable chains for user actions — use `async/await`
- Services return `Promise`, not `Observable`

---

## Forms — FormsModule + plain properties

For most forms, use `[(ngModel)]` with plain class properties — no FormBuilder, no ReactiveFormsModule:

```ts
import { FormsModule } from '@angular/forms';

@Component({
    imports: [FormsModule],
    template: `
        <input type="email" [(ngModel)]="email" placeholder="Email" [disabled]="busy()" />
        <input type="password" [(ngModel)]="password" (keydown.enter)="login()" />
        <button (click)="login()" [disabled]="busy()">Войти</button>
    `,
})
export class LoginComponent {
    email = '';
    password = '';
    busy = signal(false);
    error = signal('');
}
```

Use ReactiveFormsModule only when you need cross-field validation (e.g. password confirm), dynamic field groups, or programmatic form manipulation.

---

## Multi-step UI — signal with string union

```ts
type Screen = 'form' | 'otp' | 'success';

export class RestorePasswordComponent {
    screen = signal<Screen>('form');

    // switch between states
    // template: @if (screen() === 'form') { ... }
    //           @if (screen() === 'success') { ... }
}
```

No `BehaviorSubject`, no `enum ModalStates`. String union + `signal()` is simpler and more readable.

---

## Template syntax (Angular 17+)

Use the new control flow syntax — never `*ngIf` / `*ngFor`:

```html
@if (state.loading()) {
    <div>Загрузка...</div>
}

@if (error()) {
    <p class="text-red-600">{{ error() }}</p>
}

@for (item of state.items(); track item.id) {
    <app-item-card [item]="item" />
}

@if (formOpen()) {
    <!-- modal -->
} @else {
    <!-- something else -->
}
```

---

## Dependency injection

```ts
export class MyComponent {
    // public: accessed in template
    readonly state = inject(AppStateService);

    // private: internal use only
    private readonly router = inject(Router);
    private readonly route = inject(ActivatedRoute);
}
```

No constructor injection. All dependencies via `inject()` at property level.

---

## What NOT to do

- Don't use `ChangeDetectionStrategy.OnPush` — not needed with signals
- Don't use `@UntilDestroy()` / `untilDestroyed()` — signals auto-clean up
- Don't use `@Select()` / `Store.dispatch()` / NGXS
- Don't use `| async` pipe — read signals directly: `value()`
- Don't use `*ngIf` / `*ngFor` — use `@if` / `@for`
- Don't use `@Input()` decorator — use `input()` signal function instead
- Don't use `ngOnInit` for data loading when constructor + inject() works
- Don't put components in NgModule declarations — standalone only
- Don't create components without their own folder
