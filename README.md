# BranchGraph – Product Requirements Document (PRD)

## 1  Purpose & Problem Statement

Current chat interfaces force conversations down a single linear path, making it hard to explore tangents without losing the original thread. BranchGraph lets users converse with large language models (LLMs) on a canvas where each prompt/response pair is a node in a knowledge graph. Users can highlight any part of a response to spawn a new branch, preserving context while diving deeper.

## 2  Goals & Success Criteria

| Objective                          | Success Metric                                            |
| ---------------------------------- | --------------------------------------------------------- |
| Ship an MVP in ≤ 4 weeks           | Deployed, usable by friends on a public URL               |
| Validate branching workflow        | ≥ 50% of sessions include ≥ 1 branch action               |
| Demonstrate engineering competence | Recruiter feedback ≥ 3 positive mentions; repo stars ≥ 25 |

## 3  Target Users

* Knowledge workers & students researching complex topics
* Hobbyist developers exploring code/ideas with LLMs
* Portfolio reviewers (potential employers) assessing full‑stack skills

## 4  User Stories (MVP)

1. **US‑1** As a guest, I can sign in with Google so my graphs persist.
2. **US‑2** As a signed‑in user, I can type a prompt, choose the model (e.g., GPT‑4o, GPT‑3.5), and see the response as a node.
3. **US‑3** I can pan/zoom the canvas to navigate nodes.
4. **US‑4** I can highlight text in any node and click “Branch” to create a child node pre‑populated with that text as additional context.
5. **US‑5** I can continue the current thread without branching by simply asking a follow‑up.
6. **US‑6** My graphs autosave so refreshing the page restores the state.

## 5  Functional Requirements (MVP)

* **Authentication:** Google OAuth via NextAuth.js (extendable to other providers).
* **Prompt Input:** Text area + model dropdown (models fetched from OpenAI list; default GPT‑3.5‑turbo).
* **LLM Call:** Server route proxies request to OpenAI with user‑supplied API key (encrypted at rest).
* **Graph Data Model:**

  * `nodes(id, user_id, content, role, model, created_at)`
  * `edges(parent_id, child_id)`
* **Canvas UI:** Powered by React Flow; supports draggable nodes, selectable edges.
* **Branch Action:** On text selection, show context menu “Branch”; sends selected text back to server to seed `system` or `user` context for new node.
* **Persistence:** Prisma ORM with Postgres (Neon free tier for SaaS‑friendly serverless Postgres).
* **Deployment:** Vercel (frontend + edge functions) or Railway (monorepo) for rapid MVP.

## 6  Non‑Functional Requirements

* **Performance:** Response latency ≤ 3 s (network + OpenAI) 90th percentile.
* **Security:** Store API keys encrypted; do NOT log raw keys or prompts.
* **Accessibility:** Keyboard navigation for node selection; sufficient contrast.
* **Observability:** Console logging (pino) + Sentry for exceptions.

## 7  Technical Stack Rationale

| Layer       | Chosen Tooling                                                    | Why                                                                                                    |
| ----------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Frontend    | **Next.js 14** + React Flow + Tailwind CSS                        | Familiar TypeScript stack; server components + hot reload; React Flow gives node‑canvas out of the box |
| State Mgmt  | TanStack Query (server sync) + Zustand (client)                   | Simple and familiar                                                                                    |
| Backend/API | Next.js Route Handlers (Express‑like) **or** Fastify if separated | Minimal boilerplate; easy deployment                                                                   |
| Database    | Postgres (Neon) + Prisma                                          | Prior experience; rich relational queries for graphs                                                   |
| Auth        | NextAuth.js (Google provider)                                     | Quick setup; extensible                                                                                |
| LLM         | OpenAI Node SDK                                                   | Direct, typed interface                                                                                |
| Deployment  | Vercel                                                            | Free, fast CI/CD, edge functions                                                                       |

### Alternative Options Considered

* **Clerk** for auth (more polished UI, but NextAuth is free and enough for MVP).
* **Supabase** replacing Postgres + auth, but Prisma familiarity wins.
* **tRPC** for typesafe end‑to‑end APIs; adopt post‑MVP if routes grow.

## 8  Milestones & Timeline

| Week | Milestone                             |
| ---- | ------------------------------------- |
| 0    | Project setup, repo, CI/CD on Vercel  |
| 1    | Auth + simple prompt → response node  |
| 2    | React Flow canvas, node persistence   |
| 3    | Text‑selection branching logic        |
| 4    | Polish UI, metrics, invite beta users |

## 9  Future Enhancements (Post‑MVP)

* Real‑time collaborative graphs (Liveblocks or Yjs)
* Export graph to Markdown / JSON
* Support Anthropic, open‑source models via Ollama
* Prompt templates & few‑shot memory per node
* Shareable read‑only links

## 10  Risks & Mitigations

| Risk                                | Mitigation                                        |
| ----------------------------------- | ------------------------------------------------- |
| OpenAI rate limits or cost overruns | Enforce per‑user token quota; cache responses     |
| Canvas complexity/performance       | Virtualization in React Flow; limit visible nodes |
| Security of user API keys           | AES‑GCM encryption, rotate secret key             |

## 11  Open Questions

1. Multi‑tenant architecture vs single‑user (for MVP)?
2. Will graphs ever be public/shareable?
3. Need chat‑history token trimming strategy?

## 12  Detailed Project Roadmap

| Week | Theme                      | Key Tasks                                                                                                                                                                                                                                                                                    | Deliverable                                                |
| ---- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| 0    | **Project Bootstrap**      | T0‑1 Create GitHub repo, enable Vercel CI/CD · T0‑2 Initialize Next.js 14 + pnpm monorepo · T0‑3 Add Tailwind, ESLint, Prettier, Husky pre‑commit · T0‑4 Provision Neon Postgres, set DATABASE\_URL secret · T0‑5 Scaffold Prisma schema (`users`, `nodes`, `edges`) and run first migration | Repo skeleton deploys "Hello World" page on Vercel preview |
| 1    | **Auth & Data Layer**      | T1‑1 Integrate NextAuth.js (Google) · T1‑2 Encrypt & store user OpenAI key (env + edge config) · T1‑3 Implement `/api/nodes` CRUD routes (Prisma) · T1‑4 Write Cypress smoke test for sign‑in & node creation                                                                                | Authenticated user can create a dummy node that persists   |
| 2    | **Prompt → Response Flow** | T2‑1 Proxy `/api/chat` to OpenAI SDK with selected model · T2‑2 Build PromptBar component with model dropdown · T2‑3 Render response node in React Flow; autosave graph · T2‑4 Pan/zoom & fit‑view helpers (React Flow hooks)                                                                | Users converse linearly; graph reloads after refresh       |
| 3    | **Branching Mechanics**    | T3‑1 Add text‑selection “Branch” context menu · T3‑2 Generate child node pre‑filled with selection as system/user context · T3‑3 Create `edges` table entries; draw arrow · T3‑4 Handle token trimming for long histories                                                                    | Users branch off any snippet without losing main thread    |
| 4    | **Polish & Alpha Release** | T4‑1 UI polish (markdown, syntax highlight, dark mode) · T4‑2 Rate limiting & per‑user token quota · T4‑3 Error monitoring (Sentry) & structured logging (pino) · T4‑4 Add simple analytics (PostHog) · T4‑5 Create onboarding email + share link                                            | Public alpha live; friends invited                         |

### Task ID Legend

* **T0‑x / T1‑x ...** = Atomic tasks you can track as GitHub issues.
* Prefix links nicely to User Stories (e.g., `US-2` maps to `T2‑1, T2‑2, T2‑3`).

### Working Rhythm Suggestion

* **Daily goal‑setting:** Pick 1‑2 tasks each morning; PR by evening.
* **Weekly demo:** Record a 2‑minute Loom walkthrough Friday; gather feedback.

### After Alpha (Post‑MVP)

Pull items from section 9 Future Enhancements into a new roadmap cycle based on user feedback and personal learning goals.
