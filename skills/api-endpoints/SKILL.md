---
description: guides the creation of new API endpoints, use when adding routes, handlers, or controllers to the project
---

# API Endpoint Conventions

Follow these conventions whenever you add or modify an API endpoint.

## Where route files live

- Route files live in `src/routes/`, one file per resource (e.g. `src/routes/users.ts`).
- Handlers and controllers live in `src/controllers/`, one file per resource, mirroring the route file name.
- Shared validation schemas live in `src/schemas/`.
- Register every new route file in `src/routes/index.ts`. A route that is not registered there is not live.
- Tests live next to the code they cover, in `src/routes/__tests__/` and `src/controllers/__tests__/`.

## Naming conventions

- Route files, controller files, and URL paths use kebab-case (`order-items.ts`, `/order-items`).
- URL paths use plural nouns for resources (`/users`, `/users/:id`), never verbs. The HTTP method carries the action.
- Handler functions are camelCase and start with a verb: `getUser`, `listUsers`, `createUser`, `updateUser`, `deleteUser`.
- Path parameters are camelCase (`:userId`), and JSON fields are camelCase.
- Version APIs in the path prefix (`/api/v1/...`). Never make breaking changes to an existing version.

## Error handling pattern

- Handlers never swallow errors and never send error responses by hand. Throw a typed error and let the central error middleware format it.
- Use the shared error classes (`BadRequestError`, `UnauthorizedError`, `ForbiddenError`, `NotFoundError`, `ConflictError`). Each one carries an HTTP status and a stable machine-readable `code`.
- Wrap async handlers so rejected promises reach the error middleware.
- Validate input at the edge, before any business logic. Validation failures throw `BadRequestError` with per-field details.
- Never leak stack traces, SQL, or internal messages to the client. Log the details server-side with the request ID.
- Use the correct status code: 400 invalid input, 401 unauthenticated, 403 unauthorized, 404 missing, 409 conflict, 500 unexpected.

## Response shape

Every response uses the same envelope.

Success:

```json
{
  "data": { "id": "123", "name": "Ada" },
  "meta": {}
}
```

- `data` holds the resource, or an array of resources for list endpoints.
- `meta` holds pagination (`page`, `pageSize`, `total`) and other non-resource information. It is always present, empty if unused.

Error:

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User not found",
    "details": []
  }
}
```

- Use `201` for creation, `204` with no body for deletion, and `200` otherwise.
- Never return a bare array or a bare string at the top level.

## Before committing

An endpoint is not ready to commit until all of these are present:

- [ ] Route registered in `src/routes/index.ts`
- [ ] Input validation schema for params, query, and body
- [ ] Authentication and authorization checks, or an explicit note that the route is public
- [ ] Errors thrown through the shared error classes, with no hand-written error responses
- [ ] Response matches the envelope above
- [ ] Tests covering the success path, validation failure, and not-found or unauthorized cases
- [ ] API documentation updated (OpenAPI spec or equivalent)
- [ ] Linter, type check, and the full test suite pass locally
