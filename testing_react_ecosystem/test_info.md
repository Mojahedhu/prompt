# AI Testing Protocol & Reference Guide

**System Directive for AI Agent:** You are operating within the Google Antigravity IDE. This document serves as your absolute source of truth for all testing operations within this repository. Whenever you are tasked with writing, updating, or debugging tests, adhere strictly to the rules and conventions defined below.

## 1. Project Architecture & Environment
- **Framework:** React / Next.js
- **Package Manager:** `pnpm`
- **UI Components:** Tailwind CSS & shadcn/ui
- **Database/Backend:** Supabase (ensure data structures and Row-Level Security rules are mocked or handled appropriately in integration tests)
- **Local Dev:** Execution runs locally on the host machine.

## 2. Testing Stack
- **Unit & Integration:** Vitest + React Testing Library (RTL)
- **End-to-End (E2E):** Playwright
- **Mocking:** Vitest standard mocking (`vi.mock()`) and MSW (Mock Service Worker) for network requests.

## 3. Directory Structure & File Naming
- **Unit/Integration Tests:** Co-locate test files with their corresponding components. 
  - *Pattern:* `[filename].test.tsx` or `[filename].spec.ts`
- **E2E Tests:** Keep isolated in a dedicated root `/tests/e2e` directory.
  - *Pattern:* `[feature].e2e.ts`
- **Test Utilities:** Place all custom renders, mock data builders, and setup files in `/tests/utils`.

## 4. Execution Commands
Always use `pnpm` to execute test scripts.
- **Run Unit Tests:** `pnpm test`
- **Run Tests in Watch Mode:** `pnpm test:watch`
- **Run UI Mode (Vitest):** `pnpm test:ui`
- **Run E2E Tests:** `pnpm run test:e2e`

## 5. Core Testing Directives (The SOP)

### A. React Testing Library Rules
1. **Behavior Over Implementation:** Test the user experience, not the internal component state. Never test hooks in isolation unless they are highly complex utility hooks.
2. **Query Priority:** 
   - *Highest:* `getByRole`, `getByLabelText` (accessible queries).
   - *Lowest:* `getByTestId` (only use as a last resort when semantic HTML is unavailable).
3. **Wait for Updates:** Use `waitFor` or `findBy*` queries when asserting state changes that depend on asynchronous actions.

### B. Mocking Strategy
1. **External APIs:** Never hit live endpoints. Use MSW to intercept API calls or mock the fetch/axios client directly.
2. **Third-Party Libraries:** Do not test third-party code. If a component relies on a complex library (like a chart or 3D canvas), mock the library and assert that it receives the correct props.
3. **Database Client:** When testing components interacting with the Supabase client, inject a mocked client that returns deterministic data payloads. 

### C. Styling & UI Framework Assumptions
1. **shadcn/ui & Tailwind:** Assume standard shadcn components (Buttons, Dialogs, Selects) function correctly. Do not write tests verifying that a dialog opens *unless* it is tied to custom business logic.
2. **Visuals:** Do not assert on Tailwind class names. Assert on visibility, accessibility roles, and DOM presence.

## 6. AI Agent Workflow
When instructed to "write tests for [Component/Feature]":
1. Read the component source code to understand its props, context dependencies, and side effects.
2. Identify the necessary mocks (API calls, navigation routers, context providers).
3. Draft the test file utilizing the AAA (Arrange, Act, Assert) pattern.
4. Execute `pnpm test` via the terminal to verify the tests pass.
5. If tests fail, analyze the traceback, correct the test code, and re-run until all tests are green.
