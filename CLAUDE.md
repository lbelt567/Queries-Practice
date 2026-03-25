# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an e-commerce data utilities project that provides query functions for a SQLite database. The project uses TypeScript with ES2022 modules (`"type": "module"` in package.json).

## Development Commands

```bash
# Install dependencies and initialize Claude hooks
npm run setup

# Run a TypeScript file directly (no compilation step needed)
npx tsx src/main.ts

# Run the Claude Agent SDK integration
npm run sdk
```

There is no test command configured yet.

## Architecture

`src/main.ts` opens `ecommerce.db` (SQLite file at project root) and runs `createSchema()` to initialize all tables. All query modules under `src/queries/` receive a `Database` instance (from the `sqlite` promise wrapper) as their first argument.

The `sqlite` package wraps `sqlite3` with a Promise-based API. All query functions must use `async/await` with `db.get()` (single row) or `db.all()` (multiple rows):

```typescript
import { Database } from "sqlite";

export async function getOrderById(
  db: Database,
  orderId: number,
): Promise<any> {
  return db.get(`SELECT * FROM orders WHERE id = ?`, [orderId]);
}
```

Do **not** use the raw `sqlite3` callback pattern — use the `sqlite` promise wrapper throughout.

## Hooks

The project uses Claude Code hooks defined in `.claude/settings.example.json` (copy to `.claude/settings.json` to activate):

- **PreToolUse (Write/Edit/MultiEdit)** — `hooks/query_hook.js`: Uses the Claude Agent SDK to detect if new query functions duplicate existing ones in `src/queries/`. Blocks writes that introduce duplicates.
- **PostToolUse (Write/Edit/MultiEdit)** — runs `prettier` on the modified file, then `hooks/tsc.js` for TypeScript type-checking against `tsconfig.json`. Writes will be blocked if they introduce type errors.

## Database Schema

Defined in `src/schema.ts`. Primary key column is `id` (INTEGER AUTOINCREMENT) on all tables. Key tables:

- `customers` — status: `active | inactive | suspended`
- `orders` — status: `pending | processing | shipped | delivered | cancelled | refunded`; links to `customers`, `addresses`
- `order_items` — links `orders` to `products`
- `inventory` — per-warehouse stock for each product; unique on `(product_id, warehouse_id)`
- `promotions` — discount_type: `percentage | fixed | free_shipping`

## Critical Guidance

- All database queries must be written in `src/queries/`
- The `tsconfig.json` sets `rootDir: "./"` (project root), so imports like `import { createSchema } from "./schema"` from inside `src/` resolve correctly relative to root
