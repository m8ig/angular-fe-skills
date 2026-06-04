---
name: create a data-access library
description: Use when creating the Angular HTTP client layer for a new feature in the Baza NX monorepo. Covers Observable-based service wrappers around REST endpoints, file structure, and module setup.
---

A data-access library is the **Angular HTTP client layer** — Observable-based wrappers around REST endpoints. Lives inside `baza-core-data-access/src/libs/`.

Used in: Angular web app, Angular CMS app.

## Where it lives
`libs/baza-core-data-access/src/libs/{feature-name}/src/lib/ng/`

For CMS-specific endpoints:
`libs/baza-core-cms-data-access/src/libs/{feature-name}/src/lib/ng/`

## Folder structure

```
src/
  lib/
    ng/
      {feature}.data-access.ts          ← Angular @Injectable service
      {feature}-cms.data-access.ts      ← CMS variant (if needed, uses CMS HTTP client)
    {feature}.data-access-module.ts     ← NgModule exporting the service
  index.ts
```

## Data-access service pattern

Implements the `*Endpoint` interface from shared. Uses `Observable` (not Promise):

```typescript
@Injectable()
export class ExampleDataAccess implements ExampleEndpoint {
    constructor(private readonly http: BazaDataAccessService) {}

    list(request: ExampleListRequest): Observable<ExampleListResponse> {
        return this.http.get(ExampleEndpointPaths.list, request);
    }

    getById(request: ExampleGetByIdRequest): Observable<ExampleDto> {
        return this.http.get(ExampleEndpointPaths.getById, request);
    }

    create(request: ExampleCreateRequest): Observable<ExampleDto> {
        return this.http.post(ExampleEndpointPaths.create, request);
    }

    update(request: ExampleUpdateRequest): Observable<ExampleDto> {
        return this.http.post(ExampleEndpointPaths.update, request);
    }

    delete(request: ExampleDeleteRequest): Observable<void> {
        return this.http.post(ExampleEndpointPaths.delete, request);
    }
}
```

## CMS variant (uses CMS HTTP client with CMS auth token)

```typescript
@Injectable()
export class ExampleCmsDataAccess implements ExampleCmsEndpoint {
    constructor(private readonly http: BazaCmsDataAccessService) {}

    // same pattern, different http client
}
```

## Module pattern

```typescript
@NgModule({
    imports: [BazaDataAccessModule],
    providers: [ExampleDataAccess],
    exports: [ExampleDataAccess],
})
export class ExampleDataAccessModule {}
```

## Rules
- Always `Observable`, never `Promise` — this is Angular territory
- Use `BazaDataAccessService` for web app, `BazaCmsDataAccessService` for CMS
- HTTP method choice: `get()` for read operations, `post()` for write/mutations
- The service must implement the exact `*Endpoint` interface from shared
- No business logic here — only HTTP calls
- Each service has its own `*DataAccessModule` for clean imports in feature modules
