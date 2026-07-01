# Todo API — Implementation Plan

> **Single source of truth for autonomous execution.**

## Assumptions
- Node.js 18+, npm available
- Express.js for routing
- In-memory storage (no database dependency for this iteration)
- JSON request/response bodies

## Implementation Checklist

- [ ] Initialize npm project with `npm init -y`, set `"type": "module"`
- [ ] Install dependencies: `express`, `uuid` (for generating todo IDs)
- [ ] Install dev dependencies: `jest`, `supertest`
- [ ] Create `src/index.js` — Express app setup, JSON body parser, listen on PORT from env or 3000
- [ ] Create `src/routes/todos.js` — router with in-memory array storage
- [ ] Implement `GET /api/todos` — returns all todos as JSON array
- [ ] Implement `POST /api/todos` — accepts `{ title: string, done?: boolean }`, validates title is non-empty string, returns created todo with `id`, `title`, `done`, `createdAt`
- [ ] Implement `GET /api/todos/:id` — returns single todo or 404 `{ error: "Not found" }`
- [ ] Implement `PATCH /api/todos/:id` — accepts `{ title?, done? }`, returns updated todo or 404
- [ ] Implement `DELETE /api/todos/:id` — removes todo, returns 204 no content, or 404
- [ ] Add global error handler middleware for unexpected errors (500 with generic message)
- [ ] Add 404 handler for unknown routes
- [ ] Write unit tests for route handlers in `tests/todos.test.js`: create, list, get by id (found and not found), update, delete, validation errors
- [ ] Verify all tests pass with `npx jest --verbose`
- [ ] Create `README.md` with setup, run, and API endpoint documentation
