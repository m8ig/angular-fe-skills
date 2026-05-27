---
name: fetch and store global data (ngxs)
description: "Use when data must be loaded once on app start and available across all features — regardless of which route the user visits. Covers the Bootstrap store + Resolver pattern: global state, cache-first resolver, and attaching it to the root route so all lazy-loaded children receive the data before rendering."
---

For feature-specific data loading see `Fetch and store feature data (NGXS).md`. This skill covers a different problem: data that is **not tied to any specific feature** and must be available everywhere — configuration, user profile, app-wide settings, marketing sections, etc.

---

## The problem

Some data is needed on every page: current user, app config, global CMS sections. Loading it inside each feature would cause duplicate requests and race conditions. The solution: load it **once**, before any route renders, and store it globally.

---

## How it works

```
User opens any URL (e.g. /items/some-slug)
  └── Root route resolver runs first         ← BootstrapResolver
        ├── Data already in store?  → skip, return cached value
        └── Not loaded yet?         → dispatch action → HTTP request → store result
              └── Only now the child route renders
                    └── feature-item, feature-portfolio, etc. all read from store
```

The resolver is attached to the **root `path: ''` route** that wraps all other routes. This guarantees the data exists before any lazy-loaded module renders, no matter which URL the user opens.

---

## Store structure

Identical to a feature store but with a single action and no pagination or mutations:

```
store/bootstrap/
  actions.ts
  state.ts
  state.module.ts
  index.ts
```

### actions.ts

One action — no payload needed, the endpoint knows what to return:

```ts
export class RequestAppBootstrapInitData {
    static readonly type = '[Bootstrap] RequestAppBootstrapInitData';
}
```

### state.ts

```ts
export interface BootstrapStateModel {
    initData: AppBootstrapDto | null;
}

@State<BootstrapStateModel>({
    name: 'bootstrap',
    defaults: { initData: null },
})
@Injectable()
export class BootstrapState {
    constructor(private readonly dataAccess: BootstrapDataAccess) {}

    @Selector()
    static initData(state: BootstrapStateModel): AppBootstrapDto {
        return state.initData;
    }

    // cancelUncompleted: true — if resolver fires twice during fast navigation,
    // the first in-flight request is cancelled and only the latest runs
    @Action(RequestAppBootstrapInitData, { cancelUncompleted: true })
    requestAppBootstrapInitData(ctx: StateContext<BootstrapStateModel>) {
        return this.dataAccess.bootstrap().pipe(
            tap((response) => ctx.patchState({ initData: response })),
            catchError(() => EMPTY),
        );
    }
}
```

### state.module.ts

```ts
@NgModule({
    imports: [
        NgxsModule.forFeature([BootstrapState]),
        BootstrapDataAccessModule,
    ],
})
export class BootstrapStateModule {}
```

---

## Resolver — cache-first

The resolver is the critical piece. It checks the store before making any HTTP request:

```ts
@Injectable({ providedIn: 'root' })
export class BootstrapResolver implements Resolve<AppBootstrapDto> {
    constructor(private readonly store: Store) {}

    resolve(): Observable<AppBootstrapDto> {
        const initData = this.store.selectSnapshot(BootstrapState.initData);

        // Already loaded on a previous navigation → return immediately, no request
        return initData
            ? of(initData)
            : this.store.dispatch(new RequestAppBootstrapInitData());
    }
}
```

`selectSnapshot` reads the current store state synchronously — no subscription needed. If data is present (user navigates to a second page), it returns instantly. If not (first load), it dispatches the action and waits for the HTTP response before the route renders.

---

## Attaching to the root route

The resolver is placed on the pathless root route that wraps all feature routes:

```ts
export const routes: Routes = [
    {
        // Maintenance page is outside — it doesn't need bootstrap data
        path: 'maintenance',
        loadChildren: () => import('...').then(m => m.MaintenanceModule),
    },
    {
        path: '',
        resolve: {
            bootstrap: BootstrapResolver,   // ← runs before any child renders
            sections: MarketingResolver,    // ← multiple global resolvers are fine
        },
        children: [
            {
                path: '',
                component: LayoutComponent,
                children: [
                    // All lazy-loaded features get bootstrap data for free
                    { path: '', loadChildren: () => import('...').then(m => m.HomeModule) },
                    { path: 'items', loadChildren: () => import('...').then(m => m.ItemModule) },
                    { path: 'portfolio', loadChildren: () => import('...').then(m => m.PortfolioModule) },
                    // ...all other routes
                ],
            },
        ],
    },
];
```

Key points:
- The root route has `path: ''` with no component of its own — it's a pure organizational wrapper
- Multiple resolvers on the same route run in parallel — `bootstrap` and `sections` fire simultaneously
- Pages that don't need bootstrap (e.g. maintenance) are placed **outside** this wrapper
- The `BootstrapStateModule` is imported at the app level (in `AppModule` or root providers), not inside a feature module

---

## Reading bootstrap data in components

Any component in any feature can read the global data via `@Select`:

```ts
@Component({ ... })
export class SomeFeatureComponent {
    @Select(BootstrapState.initData)
    initData$: Observable<AppBootstrapDto>;
}
```

No need to dispatch anything — the resolver guarantees the data is already in the store.

---

## What NOT to do

- Don't dispatch `RequestAppBootstrapInitData` from a feature component — the resolver handles this
- Don't attach the resolver to individual feature routes — it belongs on the root route only
- Don't store feature-specific data in the bootstrap state — keep global data truly global (config, user, app-wide settings)
- Don't skip the cache check in the resolver (`selectSnapshot`) — without it, every navigation triggers a new HTTP request
- Don't put `BootstrapStateModule` inside a lazy-loaded feature module — it must be available from app start
