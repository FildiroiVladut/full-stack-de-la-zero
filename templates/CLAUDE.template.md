# CLAUDE.md

<!--
Copy this file into the root of each project as CLAUDE.md and fill in the <...> parts.
Claude Code reads it at the start of every session. Keep it short and true:
if something here stops being accurate, fix it.
-->

## Project

<One paragraph: what this app does and for whom. Same as "What it does" in README.md.>

## Stack

<e.g. Vite + React + TypeScript (strict), React Router, TanStack Query, Tailwind CSS, zod. Package manager: npm.>

Do not add dependencies outside this list without asking me first and explaining why.

## Commands

- `npm run dev` — <start the dev server>
- `npm run build` — <production build>
- `npm run lint`
- `npm run typecheck`
- `npm test`

## Who I am

I'm a junior developer learning full-stack development (TypeScript, React, NestJS, Postgres). I studied Cybernetics and wrote C/C++ years ago, then worked about ten years as a professional chef. My goal is to understand every line in this repo well enough to explain it in an interview.

I work in WSL2 (Ubuntu) on Windows, with VS Code.

## How to work with me

- Talk to me in Romanian. Code, comments, commit messages, PR descriptions and docs are in English.
- Before any non-trivial change, plan first: propose the approach, list the files you'll touch, and mention one alternative and why you didn't pick it. Wait for my OK.
- Work in small steps: one feature per branch, small commits. Never commit to `main`. Never commit or push without first showing me the diff and explaining it.
- After each step, tell me:
  1. what changed and why;
  2. the 2–3 concepts this step used, with a link to the matching chapter in my guide: https://github.com/FildiroiVladut/full-stack-de-la-zero;
  3. one question I should be able to answer about this code.
- Prefer simple, boring, readable code over clever code. No abstractions until something repeats three times.
- If you're not sure an API or library usage is current, check the official docs before writing it. Never invent package names.
- Never put secrets in code or in frontend env vars (`VITE_*`, `NEXT_PUBLIC_*`). Secrets live in `.env` (git-ignored); keep `.env.example` updated with placeholder values.
- When something fails, explain the error message to me before fixing it.
- When I say "explain", only explain. Don't change code.
- When I'm doing a code walk, let me explain first, then correct me. Don't lecture before I try.

## Built by hand

<Feature name> — I write this one myself. Don't write code for it: review what I write and give hints only when I ask.

## Conventions

- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`.
- Branch names: `feat/<short-name>`, `fix/<short-name>` (in P3: `feat/<issue-number>-<short-name>`).
- TypeScript strict, no `any`.
- Every screen that loads data handles loading, error and empty states.
- <Project-specific rules, e.g. "Money is stored as integer cents." / "Dates are stored in UTC and shown in Europe/Bucharest.">
