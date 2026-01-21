# Frontend Template

Next.js frontend template with React, Tailwind CSS, and TypeScript. Uses centralized AI rules from the [ai-rules repository](https://github.com/palpito-hunch/ai-rules).

## Stack

- **Framework**: Next.js 16 (App Router)
- **UI**: React 19.x with React Compiler
- **Styling**: Tailwind CSS 4.x
- **Validation**: Zod 4.x
- **Language**: TypeScript 5.x

## Quick Start

### Prerequisites

- Node.js 24+

### Setup

```bash
# Clone the repository
git clone https://github.com/your-org/frontend-template.git my-app
cd my-app

# Install dependencies
npm install

# Copy environment file
cp .env.example .env.local

# Start development server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view the app.

## Project Structure

```
src/
├── app/                # Next.js App Router
│   ├── layout.tsx      # Root layout
│   ├── page.tsx        # Home page
│   └── globals.css     # Global styles
├── components/         # React components
├── hooks/              # Custom React hooks
├── lib/                # Utility functions
└── types/              # TypeScript types

.kiro/                  # AI rules (synced from ai-rules)
├── standards/          # Coding standards
├── steering/           # Kiro steering files
├── validation/         # Validation rules
└── memory/decisions/   # ADRs
```

## Available Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start development server with Turbopack |
| `npm run build` | Build for production |
| `npm start` | Start production server |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Fix linting errors |
| `npm run format` | Format code with Prettier |
| `npm run type-check` | Run TypeScript type checking |
| `npm run validate` | Run lint, format check, and type check |
| `npm test` | Run tests |

## AI Rules Sync

This project automatically syncs AI coding standards from the ai-rules repository:

- **Automatic**: Daily at 6am UTC-6
- **Manual**: Actions → Sync Standards from AI Rules → Run workflow

When updates are available, a PR is created for review.

## Template Infrastructure Sync

This project syncs infrastructure configuration (Node.js version, tooling) from the frontend-template:

- **Automatic**: Monthly on the 1st
- **Manual**: Actions → Sync Infrastructure from Template → Run workflow

Synced items:
- Node.js version (engines, README)
- DevDependencies: `@types/node`, `eslint`, `typescript`, `prettier`

## Key Features

### React Compiler

Next.js 16 includes stable React Compiler support, providing automatic memoization without manual `useMemo`/`useCallback`.

### Tailwind CSS v4

Uses the new CSS-first configuration with automatic content detection and faster builds.

### App Router

Full support for:
- Server Components (default)
- Client Components (`'use client'`)
- Server Actions
- Streaming and Suspense

## Architecture

```
┌─────────────────────────────────────────┐
│              App Router                 │
│  (Server Components by default)         │
└─────────────────┬───────────────────────┘
                  │
    ┌─────────────┴─────────────┐
    ▼                           ▼
┌─────────────┐         ┌─────────────┐
│   Server    │         │   Client    │
│ Components  │         │ Components  │
│ (data fetch)│         │ (interactiv)│
└─────────────┘         └─────────────┘
```

See `.kiro/memory/decisions/` for Architecture Decision Records.

## License

MIT
