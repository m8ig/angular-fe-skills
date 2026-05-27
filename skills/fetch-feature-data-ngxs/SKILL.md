---
name: fetch and store feature data (ngxs)
description: Use when deciding how to fetch and store data in an Angular app. Covers when to use NGXS store vs plain service, store folder structure (one per feature), actions/state/selectors patterns, lazy-load-triggered data fetching, and the rule that each route loads only what it needs.
---

**Rule of thumb:**
- Data that is used in more than one place, or that persists while the user navigates → **store (NGXS)**
- Data that is only needed locally within one component and is not reused → **service** (simple `@Injectable`, no store)

---

## When to use the store

Use NGXS store for:
- Lists and paginated data (items, portfolio, feed)
- Single entity detail loaded by ID/slug (`current`)
- Anything that multiple components or routes read from the same source
- Actions that mutate shared state (add/remove favorite, toggle notification)
- Bootstrap data loaded once on app start (user profile, config)

Use a plain service (no store) for:
- One-off form submissions (contact us, newsletter subscribe)
- Local UI state that never leaves the component

---

## Structure — one feature, one store folder

Each feature has its own store folder inside `libs/web/data-access/src/lib/store/`:

```
libs/web/data-access/src/lib/store/
  item/
    actions.ts        ← action classes with static type strings
    messages.ts       ← user-facing notification strings
    state.ts          ← @State class: model, selectors, action handlers
    state.module.ts   ← NgModule that registers the state + data-access modules
    index.ts          ← re-exports everything
```

The `data-access` library owns all stores. Feature libraries (`feature-item`, `feature-portfolio`, etc.) import `ItemStateModule` from `data-access` — they never define their own state.

---

## actions.ts — action classes

Each action is a class with a static `type` string. Namespace format: `[Feature] Action Name`.

```ts
export class GetItem {
    static readonly type = '[Item] Get Item';
    constructor(public item: number | string) {}
}

export class GetItems {
    static readonly type = '[Item] Get Items';
}

export class ClearItems {
    static readonly type = '[Item] Clear Items';
}

export class ChangeItemsPage {
    static readonly type = '[Item] Change Items Page';
    constructor(public index: number) {}
}

export class AddFavorite {
    static readonly type = '[Item] Add Favorite';
    constructor(public itemRequest: FavoriteIncludeRequest) {}
}

export class RemoveFavorite {
    static readonly type = '[Item] Remove Favorite';
    constructor(public itemRequest: FavoriteExcludeRequest) {}
}
```

---

## state.ts — state model, selectors, action handlers

```ts
export interface ItemStateModel {
    current?: ItemDto;           // single loaded entity
    items: ItemDto[];            // paginated list
    favorites: ItemDto[];        // filtered subset
    paging: Paginator;           // pagination metadata
}

@Injectable()
@State<ItemStateModel>({
    name: 'item',
    defaults: {
        current: undefined,
        items: undefined,
        favorites: undefined,
        paging: { index: 1, size: 12, total: 0 },
    },
})
export class ItemState {
    constructor(
        private readonly dataAccess: ItemsDataAccess,
        private readonly effectsUtil: EffectsUtil,
        private readonly notification: NzNotificationService,
    ) {}

    // --- Selectors ---

    @Selector()
    static current(state: ItemStateModel) {
        return state.current;
    }

    @Selector()
    static items(state: ItemStateModel) {
        return state?.items || [];
    }

    @Selector()
    static paging(state: ItemStateModel) {
        return state.paging;
    }

    // --- Action handlers ---

    @Action(ClearItems)
    clearItems(ctx: StateContext<ItemStateModel>) {
        return ctx.patchState({
            items: [],
            paging: { index: 1, size: 12, total: 0 },
        });
    }

    @Action(GetItem)
    getItem(ctx: StateContext<ItemStateModel>, action: GetItem) {
        ctx.patchState({ current: null });

        return this.dataAccess.getBySlug({ slug: action.item.toString() }).pipe(
            tap((response) => ctx.patchState({ current: response })),
            catchError(() => { /* handle error */ return EMPTY; }),
        );
    }

    @Action(GetItems)
    getItems(ctx: StateContext<ItemStateModel>) {
        const { paging } = ctx.getState();

        return this.dataAccess.search({ index: paging.index, size: paging.size }).pipe(
            tap((response) => ctx.patchState({
                items: response.items,
                paging: response.pager,
            })),
            catchError(() => EMPTY),
        );
    }

    @Action(ChangeItemsPage)
    changeItemsPage(ctx: StateContext<ItemStateModel>, action: ChangeItemsPage) {
        const state = ctx.getState();
        ctx.patchState({ paging: { ...state.paging, index: action.index } });
        return this.getItems(ctx);
    }
}
```

Key rules:
- Always reset `current` to `null` before loading a new entity — prevents stale data flash
- When mutating, update ALL copies of the entity: `current`, `items`, `favorites` — keep the store consistent

---

## Updating multiple state slices after a mutation

After add/remove favorite, update `current`, `items`, and `favorites` in one action handler:

```ts
@Action(AddFavorite)
addFavorite(ctx: StateContext<ItemStateModel>, action: AddFavorite) {
    return this.favoriteDataAccess.include(action.itemRequest).pipe(
        tap(() => {
            // 1. Update current entity if it's the same item
            const current = ctx.getState().current;
            if (current?.id === action.itemRequest.listingId) {
                ctx.patchState({ current: { ...current, isFavorite: true } });
            }

            // 2. Update the paginated list
            const items = ctx.getState().items.map((item) =>
                item.id === action.itemRequest.listingId
                    ? { ...item, isFavorite: true }
                    : item
            );
            ctx.patchState({ items });

            // 3. Update favorites list
            const favorites = ctx.getState().favorites?.map((item) =>
                item.id === action.itemRequest.listingId
                    ? { ...item, isFavorite: true }
                    : item
            );
            ctx.patchState({ favorites });
        }),
        catchError(() => EMPTY),
    );
}

@Action(RemoveFavorite)
removeFavorite(ctx: StateContext<ItemStateModel>, action: RemoveFavorite) {
    return this.favoriteDataAccess.exclude(action.itemRequest).pipe(
        tap(() => {
            // Use patch + removeItem operator for clean array removal
            ctx.setState(
                patch({
                    favorites: removeItem<ItemDto>((fav) => fav.id === action.itemRequest.listingId),
                }),
            );
            // Also update current and items list as above
        }),
        catchError(() => EMPTY),
    );
}
```

---

## messages.ts — notification strings

Keep all user-facing strings out of state.ts:

```ts
export const ITEM_NOT_FOUND = 'Item not found.';
export const ADD_FAVORITE_SUCCESS = 'Item successfully added to Favorites';
export const ADD_FAVORITE_FAIL = 'There was an error during the process.';
export const REMOVE_FAVORITE_SUCCESS = 'Item successfully removed from Favorites';
export const REMOVE_FAVORITE_FAIL = 'There was an error during the process.';
```

---

## state.module.ts — registration

```ts
@NgModule({
    imports: [
        NgxsModule.forFeature([ItemState]),
        ItemsDataAccessModule,          // HTTP client for this feature
        FavoriteDataAccessModule,       // additional data-access if needed
    ],
})
export class ItemStateModule {}
```

---

## How the feature library consumes the store

### Lazy loading — data is fetched only when needed

The key pattern: the store state and its HTTP requests are registered inside a lazy-loaded module (or standalone component). This means the API call only happens when the user actually navigates to that route — not on app startup.

```
App loads
  └── user navigates to /items
        └── ItemModule is lazy-loaded          ← module loaded here for the first time
              └── ItemStateModule registered   ← state registered
                    └── ItemComponent created  ← constructor dispatches GetItems
                          └── HTTP request fires ← only now
```

Data is never fetched eagerly. If the user never visits `/items`, no request is made.

---

### Registering the state in the feature module

In `feature-item/src/lib/item.module.ts` — import `ItemStateModule`:

```ts
@NgModule({
    imports: [
        ItemStateModule,    // ← registers NGXS state + data-access HTTP module
        // ...other imports
    ],
})
export class ItemModule {}
```

For standalone components, import `ItemStateModule` directly in the component's `imports: []`.

---

### Dispatching actions from the root route component

The root route component (the one mapped to the route path) is responsible for triggering the initial data load in its constructor. Child components and sub-pages read from the already-populated store without re-fetching.

```ts
// feature-item/src/lib/item.component.ts
@Component({
    selector: 'app-item',
    templateUrl: './item.component.html',
    changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ItemComponent {
    // Read state as Observables via @Select
    @Select(ItemState.items)
    items$: Observable<ItemDto[]>;

    @Select(ItemState.paging)
    paging$: Observable<Paginator>;

    constructor(private readonly store: Store) {
        // Dispatch on construction — fires when the route is first entered
        this.store.dispatch(new ClearItems());  // reset pagination state from previous visit
        this.store.dispatch(new GetItems());    // trigger the HTTP request
    }

    onPageSelect(index: number): void {
        window.scroll(0, 0);
        this.store.dispatch(new ChangeItemsPage(index));
    }
}
```

`ClearItems` before `GetItems` — prevents the previous page-N state from flashing when the user navigates back to the list.

---

### Detail page — load by route param

```ts
// feature-item/src/lib/page/page.component.ts
@Component({ ... })
export class ItemPageComponent {
    @Select(ItemState.current)
    current$: Observable<ItemDto>;

    constructor(
        private readonly store: Store,
        private readonly route: ActivatedRoute,
    ) {
        const slug = this.route.snapshot.paramMap.get('slug');
        this.store.dispatch(new GetItem(slug));  // load by slug/id from URL
    }
}
```

---

### Each route loads ONLY what it needs — strictly

Every route component is responsible for its own data. It **must** dispatch actions only for data that is actually required on that specific page. This is non-negotiable.

Loading something "just in case" or "it might be useful" is not acceptable — it causes unnecessary HTTP requests, slows down the page, and makes the data flow hard to trace.

If a page genuinely requires data from multiple stores — that's fine. Dispatch all of them. But every single dispatch must have a clear reason tied to a visible element on that page.

```ts
constructor(private readonly store: Store) {
    // ✅ This page shows a list of items AND a sidebar with favorites
    this.store.dispatch(new GetItems());
    this.store.dispatch(new GetFavorites());

    // ❌ Do NOT preload data for a different page "while we're here"
    // this.store.dispatch(new GetPortfolio());  ← loaded only on /portfolio
}
```

The `/portfolio` route dispatches `GetPortfolio` when its own component loads — not before.

---

### Rules

- Components never call data-access services directly — always go through `store.dispatch()`
- Only the **root route component** dispatches the initial load — sub-pages and child components just `@Select` from the store
- Use `@Select()` to read state as Observables — pair with `async` pipe in templates
- `ClearItems` before `GetItems` on route entry — prevents stale paginated state from a previous visit
- Each route loads **only** the data it needs — no speculative or "convenience" preloading for other routes

---

## What NOT to do

- Don't put state in feature libraries — all state lives in `data-access`
- Don't call HTTP services directly from components — dispatch an action instead
- Don't create a store for single-use form submissions (contact form, newsletter) — a simple service is enough
- Don't forget to update all copies of an entity after a mutation (`current`, `items`, `favorites`)
- Don't share one big global state for unrelated features — one feature, one `@State` class
