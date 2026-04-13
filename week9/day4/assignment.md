# Week 9 - Day 4 Assignment

## Title
Testing Basics in React -- Vitest and React Testing Library

## Overview
Day 4 is pre-weekend polish day. Today you write your first React tests. Vitest + React Testing Library are the standard. Your assignment is to set up the tooling, write three tests (one utility function, one component, one hook), and make them run in `npm test`.

## Learning Objectives Assessed
- Install and configure Vitest for a Vite + React project
- Write a test that asserts on pure function output
- Render a component with React Testing Library and find elements by role/text
- Simulate events with `fireEvent` or `userEvent`
- Test a custom hook

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 50/50. **Habit:** AI reads docs, you make decisions. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Test boilerplate after you decided what to test.
- **NOT ALLOWED FOR:** Deciding what is worth testing.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Install Vitest and Testing Library

**What to do:**
```bash
npm install -D vitest @testing-library/react @testing-library/user-event @testing-library/jest-dom jsdom
```

Add to `vite.config.js`:

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: "./src/test-setup.js",
  },
});
```

Create `src/test-setup.js`:

```js
import "@testing-library/jest-dom";
```

Add to `package.json`:

```json
"scripts": {
  "test": "vitest"
}
```

Run `npm test`. It should start in watch mode and say "No tests found".

**Expected output:**
Vitest runs without errors.

### Task 2: Test a pure function

**What to do:**
Pick any utility function you wrote earlier (like `formatCurrency(cents)` from Week 5 or a validator from Week 8). Create `src/utils/format.test.js`:

```js
import { describe, it, expect } from "vitest";
import { formatCurrency } from "./format";

describe("formatCurrency", () => {
  it("formats cents as KSh with comma separators", () => {
    expect(formatCurrency(150000)).toBe("KSh 1,500");
  });

  it("returns KSh 0 for 0", () => {
    expect(formatCurrency(0)).toBe("KSh 0");
  });
});
```

**Expected output:**
Two passing tests.

### Task 3: Test a component

**What to do:**
Test your `Counter` component from Week 7. Create `src/components/Counter.test.jsx`:

```jsx
import { describe, it, expect } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import Counter from "./Counter";

describe("Counter", () => {
  it("starts at 0", () => {
    render(<Counter />);
    expect(screen.getByText(/Count: 0/i)).toBeInTheDocument();
  });

  it("increments on + click", async () => {
    const user = userEvent.setup();
    render(<Counter />);
    await user.click(screen.getByRole("button", { name: "+" }));
    expect(screen.getByText(/Count: 1/i)).toBeInTheDocument();
  });
});
```

**Expected output:**
Two passing component tests.

### Task 4: Test a custom hook

**What to do:**
Test your `useToggle` hook with `renderHook`:

```jsx
import { describe, it, expect } from "vitest";
import { renderHook, act } from "@testing-library/react";
import { useToggle } from "./useToggle";

describe("useToggle", () => {
  it("starts at false by default", () => {
    const { result } = renderHook(() => useToggle());
    expect(result.current[0]).toBe(false);
  });

  it("toggles to true on call", () => {
    const { result } = renderHook(() => useToggle());
    act(() => result.current[1]());
    expect(result.current[0]).toBe(true);
  });
});
```

**Expected output:**
Two passing hook tests.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 9 Day 4 Pre-Weekend Checklist

- [ ] UserList handles all four states
- [ ] Temperature app demonstrates lifting state up
- [ ] Theme Context works in two components
- [ ] useToggle, useLocalStorage, useFetch all working
- [ ] Vitest runs via `npm test`
- [ ] At least 6 tests passing (2 utility, 2 component, 2 hook)
- [ ] AI_AUDIT.md has decisions owned for every day
- [ ] Repo pushed and clean
```

Tick every box honestly.

**Expected output:**
Checklist committed.

## Stretch Goals (Optional - Extra Credit)

- Add a test that mocks `fetch` using Vitest's `vi.fn()`.
- Write a snapshot test with `toMatchSnapshot`.
- Configure test coverage reporting with `vitest --coverage`.

## Submission Requirements

- **What to submit:** Repo with tests, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Vitest installed and configured | 15 | `npm test` runs and finds tests. |
| Utility function tests | 20 | At least 2 tests on a pure function. Both pass. |
| Component tests | 25 | At least 2 tests on a component using Testing Library. Both pass. |
| Custom hook tests | 20 | At least 2 tests on a hook using renderHook. Both pass. |
| Pre-weekend checklist | 10 | All boxes honest. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Testing implementation details.** Good tests assert on what the user sees, not on internal state. Prefer `getByRole`, `getByText` over inspecting state directly.
- **Forgetting `act` around hook state updates.** React warns when state updates happen outside `act` in tests. Wrap setState calls in `act(...)`.
- **Snapshot testing for everything.** Snapshots are tempting but they fail for reasons unrelated to actual bugs. Use them sparingly.

## Resources

- Day 4 reading: [Testing Basics in React.md](./Testing%20Basics%20in%20React.md)
- Week 9 AI boundaries: [../ai.md](../ai.md)
- Vitest docs: https://vitest.dev/
- Testing Library docs: https://testing-library.com/docs/react-testing-library/intro
