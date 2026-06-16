# behave-ui

**Behavior-first React components.**
Async state, forms, and data fetching — batteries included.

> "Kill one 'annoying thing' perfectly."

---

## Table of Contents

- [Concept](#concept)
- [Components](#components)
- [Who is this for?](#who-is-this-for)
- [Installation](#installation)
- [Usage](#usage)
- [Developer Setup](#developer-setup)
- [Design Principles](#design-principles)
- [License](#license)

---

## Concept

Most UI libraries stop at *appearance*. Are you still writing this every time?

```tsx
// ❌ Boilerplate you write over and over
const [loading, setLoading] = useState(false);
const [error, setError] = useState<Error | null>(null);

async function handleClick() {
  setLoading(true);
  try {
    await api.submit(data);
  } catch (e) {
    setError(e as Error);
  } finally {
    setLoading(false);
  }
}
```

**behave-ui** ships components that own their *behavior* — not just their look.

```tsx
// ✅ behave-ui
<AsyncButton onClick={() => api.submit(data)} loadingText="Submitting...">
  Submit
</AsyncButton>
```

---

## Who is this for?

**Great fit:**
- **Admin panels, internal tools, dashboards** — ship features before worrying about design
- **Startups and solo developers** — spend time on business logic, not boilerplate
- **Projects already using Zod** — turn existing schemas directly into form UIs
- **Teams that want to move fast** — drop in one component at a time, shadcn/ui style

**Not a great fit:**
- Products that require pixel-perfect design systems (you style everything yourself)
- Teams already using TanStack Query (DataFetch is a lighter alternative, not a replacement)
- Complex form layouts like multi-column grids (React Hook Form directly is more flexible)

---

## Components

| Component | What it solves |
|---|---|
| `<AsyncButton />` | Manages pending / success / error state automatically |
| `<AutoForm />` | **Zod v4 ready** — generates a full form UI from a schema |
| `<DataFetch />` | Handles loading / error / empty / data in one tag |
| `useAsyncState` | The core async state hook powering the above |

### ✨ Zod v4 + Discriminated Union Support

behave-ui is one of the few form libraries with **full Zod v4 support**.

**What's in v0.4.0:**
- 🔀 **Discriminated Union** — conditional fields that switch automatically (NEW!)
- 🔢 **Number fields** — `z.number()` values are correctly typed as `number`, not `string`
- 📋 **Enum selects** — `z.enum()` options render correctly
- 🧬 **Type safety** — fully compatible with Zod v4's new internal API
- 🎯 **Zero breaking changes** — v3 schemas continue to work

---

## Installation

### Option A — Copy into your project (recommended)

No hidden dependencies. You own the code. Customize freely.

```bash
# npm / pnpm
npx @behave-ui/cli@latest add async-button
npx @behave-ui/cli@latest add auto-form
npx @behave-ui/cli@latest add data-fetch

# yarn
yarn dlx @behave-ui/cli@latest add async-button
yarn dlx @behave-ui/cli@latest add auto-form
yarn dlx @behave-ui/cli@latest add data-fetch

# Add everything at once
npx @behave-ui/cli@latest add async-button auto-form data-fetch

# List available components
npx @behave-ui/cli@latest list
```

Files are added to `src/components/ui/<ComponentName>/`.

### Option B — npm package

```bash
npm install @behave-ui/react
# or
yarn add @behave-ui/react
```

**Current versions: CLI v1.0.2 / React v0.4.0**

---

## Usage

### AsyncButton

```tsx
import { AsyncButton } from '@behave-ui/react';

<AsyncButton
  onClick={async () => await api.submitForm(data)}
  loadingText="Submitting..."
  successText="Done!"
  errorText="Something went wrong"
  onSuccess={() => router.push('/done')}
  onError={(err) => toast.error(err.message)}
>
  Submit
</AsyncButton>
```

**State machine**

```
idle ──(click)──► pending ──(resolve)──► success ──(resetDelay)──► idle
                      └───(reject)───► error ─────(click)──────► idle
```

**Props**

| Prop | Type | Default | Description |
|------|----|-----------|------|
| `onClick` | `() => Promise<T>` | **required** | Async function to run on click |
| `loadingText` | `string` | — | Label while pending |
| `successText` | `string` | — | Label after success |
| `errorText` | `string` | — | Label after error |
| `onSuccess` | `(data: T) => void` | — | Called with the resolved value |
| `onError` | `(err: Error) => void` | — | Called with the caught error |
| `resetDelay` | `number` | `2000` | ms before auto-reset to idle. `0` to disable |
| `renderContent` | `(status) => ReactNode` | — | Full render control based on status |

---

### AutoForm

```tsx
import { z } from 'zod';
import { AutoForm } from '@behave-ui/react';

const schema = z.object({
  name:     z.string().min(1, 'Required'),
  email:    z.string().email(),
  age:      z.number().int().positive().max(120),
  role:     z.enum(['admin', 'user', 'viewer']),
  isActive: z.boolean().default(true),
  bio:      z.string().optional(),
});

<AutoForm
  schema={schema}
  onSubmit={async (values) => await api.createUser(values)}
  fieldConfig={{
    age:      { label: 'Age', type: 'number', description: 'Between 1 and 120' },
    role:     { label: 'Role', type: 'select' },
    isActive: { label: 'Active', type: 'checkbox' },
    bio:      { label: 'Bio', type: 'textarea' },
  }}
/>
```

**🔀 Discriminated Union (v0.4.0+)**

Fields switch dynamically based on the selected value:

```tsx
const schema = z.discriminatedUnion('accountType', [
  z.object({
    accountType: z.literal('personal'),
    name: z.string().min(1),
    age: z.number().int().positive(),
  }),
  z.object({
    accountType: z.literal('company'),
    companyName: z.string().min(1),
    taxId: z.string().regex(/^\d{10}$/),
    employees: z.number().int().positive(),
  }),
]);

<AutoForm schema={schema} onSubmit={handleSubmit} />
// Selecting "personal" shows age
// Selecting "company" shows taxId and employees
```

**Field auto-mapping**

| Zod type | Default UI | Override with `type` |
|---------|-------------|-----------------|
| `z.string()` | `<input type="text">` | `textarea`, `password`, `url`, `email` |
| `z.number()` | `<input type="number">` | `range` |
| `z.boolean()` | `<input type="checkbox">` | `toggle` |
| `z.enum()` | `<select>` | `radio-group` |
| `z.date()` | `<input type="date">` | `datetime-local` |
| `z.discriminatedUnion()` | Dynamic field switching | — |

---

### DataFetch

```tsx
import { DataFetch } from '@behave-ui/react';

<DataFetch
  queryKey={['user', userId]}
  queryFn={() => api.getUser(userId)}
  loadingFallback={<UserSkeleton />}
  errorFallback={({ error, retry }) => (
    <div>
      <p>{error.message}</p>
      <button onClick={retry}>Retry</button>
    </div>
  )}
  emptyFallback={<p>No user found.</p>}
>
  {(user) => <UserCard user={user} />}
</DataFetch>
```

**Props**

| Prop | Type | Default | Description |
|------|----|-----------|------|
| `queryKey` | `readonly unknown[]` | **required** | Cache key (changes trigger re-fetch) |
| `queryFn` | `() => Promise<T>` | **required** | Data fetching function |
| `children` | `(data: NonNullable<T>) => ReactNode` | **required** | Render on success |
| `loadingFallback` | `ReactNode` | built-in skeleton | Shown while loading |
| `errorFallback` | `({ error, retry }) => ReactNode` | built-in error UI | Shown on error |
| `emptyFallback` | `ReactNode` | — | Shown when data is null / empty array |
| `staleTime` | `number` | `60000` | Cache lifetime in ms |
| `retry` | `number \| false` | `3` | Auto-retry count on error |

---

### useAsyncState (hook)

The core hook powering AsyncButton. Use it when you need async state management without the button UI.

```tsx
import { useAsyncState } from '@behave-ui/react';

const { execute, isPending, isSuccess, isError, error, reset } = useAsyncState({
  onSuccess: () => toast.success('Done!'),
  onError: (err) => toast.error(err.message),
  resetDelay: 3000,
});

<button onClick={() => execute(() => uploadFile(file))} disabled={isPending}>
  {isPending ? 'Uploading...' : 'Upload'}
</button>
```

---

## Developer Setup

### Requirements

| Tool | Version |
|--------|----------|
| Node.js | 20+ (LTS recommended) |
| yarn | 4.x (managed automatically via Corepack) |

### Getting Started

```bash
# 1. Clone
git clone https://github.com/HaruyaFujii/behave-ui.git
cd behave-ui

# 2. Enable Corepack (first time only)
#    Reads "packageManager" from package.json and uses the correct yarn version automatically.
corepack enable

# 3. Install dependencies
yarn install

# 4. Build
yarn build
```

### Running Tests

```bash
# Run all tests across all packages
yarn test

# react package only (faster during development)
yarn workspace @behave-ui/react test

# Watch mode
yarn workspace @behave-ui/react test:watch

# With coverage report
yarn workspace @behave-ui/react test:coverage

# Filter by component
yarn workspace @behave-ui/react test src/components/AsyncButton
yarn workspace @behave-ui/react test src/components/AutoForm
yarn workspace @behave-ui/react test src/components/DataFetch
yarn workspace @behave-ui/react test src/hooks

# Filter by test name
yarn workspace @behave-ui/react test -t "shows loadingText"
```

**Test suite**

| File | Tests | Coverage |
|---------|---------|-----------|
| `useAsyncState.test.ts` | 9 | State transitions, double-submit prevention, callbacks |
| `AsyncButton.test.tsx` | 15 | All 4 states, accessibility |
| `AutoForm.test.tsx` | 22 | Field inference, validation, accessibility |
| `DataFetch.test.tsx` | 15 | loading/success/empty/error, cache, retry |
| **Total** | **61** | |

### Build

```bash
# All packages (ESM + CJS dual output)
yarn build

# react package only
yarn workspace @behave-ui/react build

# CLI package only
yarn workspace @behave-ui/cli build

# Type check only (no build)
yarn workspace @behave-ui/react typecheck

# Test CLI locally
yarn workspace @behave-ui/cli build
node packages/cli/dist/index.js list
node packages/cli/dist/index.js add async-button --out-dir ./test-output
```

### Directory Structure

```
behave-ui/
├── CLAUDE.md                     # Claude Code project settings
├── README.md                     # This file
├── package.json                  # yarn workspaces root
├── .yarnrc.yml                   # yarn berry config
├── tsconfig.base.json            # Shared TypeScript config
├── .changeset/                   # Version management (changesets)
├── .github/workflows/ci.yml      # CI: test, typecheck, npm publish
│
├── .claude/
│   └── rules/
│       ├── coding-style.md       # TypeScript / React coding conventions
│       ├── testing.md            # Test strategy and timer gotchas
│       └── plan-template.md      # Plan mode template
│
└── packages/
    ├── react/                    # @behave-ui/react v0.4.0
    │   └── src/
    │       ├── index.ts
    │       ├── hooks/
    │       │   └── useAsyncState.ts
    │       └── components/
    │           ├── AsyncButton/
    │           ├── AutoForm/     # ✨ Zod v4 + Discriminated Union
    │           └── DataFetch/
    └── cli/                      # @behave-ui/cli v1.0.2
        └── src/
            ├── index.ts
            ├── registry.ts
            └── commands/add.ts
```

---

## Design Principles

1. **Behavior-first** — components own their state machine, not just their style
2. **Zero magic** — `data-status` attributes keep internal state always inspectable
3. **Type-safe** — generics propagate through; `onSuccess` knows the type of your data
4. **Non-destructive** — no global providers required; adopt one component at a time
5. **One problem, done right** — no aspirations to be a full design system

---

## Roadmap

| Phase | Status | Description |
|---------|------|------|
| Phase 0 | ✅ Done | Monorepo setup |
| Phase 1 | ✅ Done | AsyncButton + useAsyncState |
| Phase 2 | ✅ Done | AutoForm (Zod v4 support) |
| Phase 3 | ✅ Done | DataFetch (cache + retry) |
| Phase 4 | ✅ Done | CLI, npm publish, GitHub templates |
| Phase 5 | ✅ Done | Discriminated Union (v0.4.0) |
| Phase 6 | 🔲 Next | Performance optimization, SSR support |

---

## Contributing

Issues and PRs are welcome! Please check the existing issues before opening a new one.

---

## License

MIT
