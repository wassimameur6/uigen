# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. It uses Claude AI to generate React components based on user descriptions, featuring real-time streaming, a virtual file system (no disk writes), Monaco code editor, and optional user authentication with project persistence.

**Stack:** Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS v4, Prisma/SQLite, Vercel AI SDK + Anthropic SDK, Radix UI, Monaco Editor

## Commands

```bash
# Setup
npm run setup          # Install deps, generate Prisma client, run migrations
npm run db:reset       # Reset database (force reset)

# Development
npm run dev            # Start dev server with Turbopack (port 3000)
npm run build          # Production build
npm start              # Start production server

# Quality
npm run lint           # ESLint
npm run test           # Vitest (jsdom environment)
```

## Architecture

### Core Flow
1. User submits chat request → `/api/chat` endpoint
2. Server streams Claude response with tool calls
3. Tools (`str_replace_editor`, `file_manager`) modify VirtualFileSystem
4. FileSystemContext updates React state
5. PreviewFrame transforms JSX via Babel and renders in sandboxed iframe

### Key Abstractions

**VirtualFileSystem** (`/src/lib/file-system.ts`): In-memory Map-based tree structure with serialization for persistence. All file operations go through this class.

**React Contexts** (`/src/lib/contexts/`):
- `FileSystemContext` - VirtualFileSystem state, selected file, tool call handling
- `ChatContext` - AI chat messages, streaming state via useChat hook

**AI Tools** (`/src/lib/tools/`):
- `str_replace_editor` - view, create, str_replace, insert operations
- `file_manager` - rename, delete operations

**JSX Transformer** (`/src/lib/transform/jsx-transformer.ts`): Client-side Babel transformation, import resolution, Tailwind CSS injection for live preview.

**Provider** (`/src/lib/provider.ts`): Returns real Anthropic model (claude-haiku-4-5) or mock generator when no API key.

### Data Models (Prisma/SQLite)
See `prisma/schema.prisma` for the full schema.
- `User`: id, email, password (bcrypt), projects
- `Project`: id, name, userId?, messages (JSON), data (JSON), user relation

### Authentication
JWT-based sessions (HS256, 7-day expiry) stored in httpOnly cookies. Middleware protects `/api/projects` and `/api/filesystem` routes. Anonymous mode supported.

## Conventions

- Use `@/` path alias for all internal imports
- `"use client"` for interactive components, `"use server"` for actions
- Tailwind CSS only for styling
- Generated components use `/App.jsx` as entry point with `@/` imports
- TypeScript strict mode enabled
- Only add comments when code is difficult to understand
