---
name: create a ui-components library
description: Use when creating a reusable UI component that appears on more than one page or feature. Covers dumb component pattern — receives data via @Input(), reads auth state via @Select(), dispatches store actions. Component lives in feature-core/ui-components/.
---

## When to create a shared UI component

Extract a component into the shared `ui-components` library when:
- The same UI element appears on **more than one page or feature**
- The component has no business logic of its own — it only renders data and dispatches store actions
- Examples: favorite button, share button, item card, avatar, rating stars, badge, modal trigger

Do **not** extract into `ui-components` when:
- The component is specific to one feature and will never be reused
- The component contains complex business logic tied to a specific flow

---

## Where the library lives

```
libs/
  feature-core/
    ui-components/
      src/lib/
        favorite-btn/
        item-card/
        share-btn/
        ...
      src/index.ts
```

All reusable UI components for the web app live in a single `feature-core/ui-components` library. Feature libraries import from it — never the other way around.

---

## Component folder structure

```
favorite-btn/
  favorite-btn.component.ts
  favorite-btn.component.html
  favorite-btn.component.less    ← only if styles are needed
  favorite-btn.module.ts
  index.ts
```

---

## Component — dumb component pattern

UI components are **dumb components**: they receive data via `@Input()`, read global state via `@Select()`, and dispatch store actions. No HTTP calls, no routing logic beyond redirects, no local business state.

```ts
@UntilDestroy()
@Component({
    selector: 'app-favorite-btn',
    templateUrl: './favorite-btn.component.html',
    styleUrls: ['./favorite-btn.component.less'],
    changeDetection: ChangeDetectionStrategy.OnPush,
})
export class FavoriteButtonComponent {
    @Select(AccountState.account)
    user$: Observable<Account | null>;

    @Input() item: ItemDto;
    @Input() btnType: 'large' | 'normal' = 'normal';

    private user: Account;

    constructor(
        private readonly router: Router,
        private readonly store: Store,
    ) {
        this.user$.pipe(untilDestroyed(this)).subscribe((user) => (this.user = user));
    }

    onLikeClick(event: Event): void {
        event.stopPropagation();
        event.preventDefault();

        if (!this.user) {
            this.router.navigate(['', { outlets: { modal: ['auth', 'login'] } }]);
        } else if (this.item.isFavorite) {
            this.store.dispatch(new RemoveFavorite({ listingId: this.item.id }));
        } else {
            this.store.dispatch(new AddFavorite({ listingId: this.item.id }));
        }
    }
}
```

Key rules:
- `@Input()` for data the parent provides
- `@Select()` for global state the component needs independently
- `store.dispatch()` for mutations — never call HTTP services directly
- `ChangeDetectionStrategy.OnPush` — always
- `@UntilDestroy()` + `untilDestroyed(this)` — clean up subscriptions

---

## Module

```ts
@NgModule({
    declarations: [FavoriteButtonComponent],
    exports: [FavoriteButtonComponent],
    imports: [CommonModule, RouterModule, NzButtonModule],
})
export class FavoriteButtonModule {}
```

## index.ts

```ts
export * from './favorite-btn.module';
```

Root `src/index.ts`:

```ts
export * from './lib/favorite-btn';
export * from './lib/item-card';
export * from './lib/share-btn';
```

---

## What NOT to do

- Don't put feature-specific components into `ui-components` — keep them in the feature library
- Don't add HTTP calls or business logic — receive data and dispatch actions only
- Don't skip `ChangeDetectionStrategy.OnPush`
- Don't import from feature libraries inside `ui-components` — dependency direction is one-way
- Don't create a new `ui-components` library per feature — one shared library for all
