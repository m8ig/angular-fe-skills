---
name: create a new feature library
description: Use when creating a new lazy-loaded route group in the NX monorepo. Covers folder structure, routes file, root route component, data-access integration, and naming conventions for Angular 17+ standalone components.
---

A feature library is an independent lazy-loaded route group. Angular 17+ style — standalone components, no NgModule.

---

## Folder structure

```
libs/feature-item/
  CLAUDE.md                      ← product description of the feature
  shared/                        ← DTOs, interfaces, error codes (no framework)
  api/                           ← NestJS backend
  data-access/                   ← Angular HTTP client (Promise-based services)
  web/                           ← Angular web UI (optional)
  mobile/                        ← Angular + Capacitor mobile UI (optional)
```

See `NX — Feature-first library grouping.md` for full folder rules.

---

## Web/mobile library internal structure

```
src/lib/
  item.component.ts              ← root route component (standalone)
  item.routes.ts                 ← route definitions
  page/                          ← detail page component
    page.component.ts
  page-docs/                     ← sub-view of detail page
    page-docs.component.ts
  page-gallery/
    page-gallery.component.ts
```

`page-` prefix = sub-views rendered inside the detail page component (not separate routes).

---

## Routes file

```ts
// item.routes.ts
import { Routes } from '@angular/router';
import { ItemComponent } from './item.component';
import { ItemPageComponent } from './page/page.component';

export const ITEM_ROUTES: Routes = [
    {
        path: '',
        component: ItemComponent,
    },
    {
        path: ':id',
        component: ItemPageComponent,
    },
];
```

No `RouterModule.forChild()`. Export a plain `Routes` array.

---

## Root route component

```ts
import { Component, inject, signal } from '@angular/core';
import { ItemDataAccessService } from '@org/feature-item/data-access';

@Component({
    selector: 'app-item',
    imports: [],
    template: `...`,
})
export class ItemComponent {
    readonly service = inject(ItemDataAccessService);

    constructor() {
        this.service.loadItems();
    }
}
```

---

## Component naming

```
ItemComponent              ← root: /items
ItemPageComponent          ← detail: /items/:id
ItemPageDocsComponent      ← sub-view inside /items/:id (not a route)
ItemPageGalleryComponent   ← sub-view inside /items/:id (not a route)
```

Class name = `{Feature}{PageSection}Component`.
Folder name = same without `Component`: `page/`, `page-docs/`, `page-gallery/`.

---

## data-access library

Services return Promises, not Observables. Use `signal()` inside the service for reactive state:

```ts
@Injectable({ providedIn: 'root' })
export class ItemDataAccessService {
    items = signal<ItemDto[]>([]);
    loading = signal(false);
    error = signal('');

    async loadItems(): Promise<void> {
        this.loading.set(true);
        try {
            const result = await this.http.get<ItemDto[]>('/api/items').toPromise();
            this.items.set(result ?? []);
        } catch (err) {
            this.error.set(err instanceof Error ? err.message : 'Ошибка');
        } finally {
            this.loading.set(false);
        }
    }
}
```

---

## What NOT to do

- Don't create `{feature}.module.ts` — no NgModule
- Don't create `{feature}-routing.module.ts` — use `{feature}.routes.ts` instead
- Don't use `RouterModule.forChild()` — export `Routes` array directly
- Don't put multiple route groups in one library — one lazy-loaded route group per library
