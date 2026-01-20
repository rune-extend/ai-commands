# Prompt: Generate Unit Tests

## Context

You are tasked with creating comprehensive unit tests using Jest. The context can be either:

- **React**: Testing React hooks using React Testing Library
- **NestJS**: Testing services, classes, modules, and utilities

Follow the patterns, conventions, and best practices established in this codebase.

## Requirements

### 1. Test File Structure

- **File naming**: Create test files with the pattern `*.test.ts` (e.g., `myHook.test.ts`, `myService.test.ts`)
- **Location**: Place test files next to the source file being tested
- **Imports**:
  - React: Always import `act` and `renderHook` from `@testing-library/react`
  - NestJS: Import the class/service being tested and its dependencies
- **Documentation**: Add JSDoc comments (`/** ... */`) for each test suite (`describe` block) explaining what the code does

### 2. Test Suite Organization

#### React Hooks Pattern

```typescript
/**
 * Test suite for the [HookName] hook.
 *
 * [Brief description of what the hook does and its main purpose]
 * [Additional context about behavior, edge cases, or special considerations]
 */
describe('HookName', () => {
  beforeEach(() => {
    // Setup code (e.g., jest.useFakeTimers())
  });

  afterEach(() => {
    // Cleanup code (e.g., jest.useRealTimers())
  });

  it('should [expected behavior]', () => {
    // Test implementation
  });
});
```

#### NestJS Services/Classes Pattern

```typescript
/**
 * Test suite for the [ServiceName] class.
 *
 * [Brief description of what the class does and its main purpose]
 * [Additional context about behavior, edge cases, or special considerations]
 */
describe('ServiceName', () => {
  let service: ServiceName;
  let mockDependency: jest.Mocked<Dependency>;

  beforeEach(() => {
    jest.clearAllMocks();
    // Setup mocks and service instance
  });

  afterEach(() => {
    if (service) {
      service.onModuleDestroy?.();
    }
  });

  it('should [expected behavior]', () => {
    // Test implementation
  });
});
```

### 3. Testing Patterns by Context

#### A. React Hooks with Timers (Debounce, Throttle, Intervals)

```typescript
describe('useDebounce', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    act(() => {
      jest.runOnlyPendingTimers();
    });
    jest.useRealTimers();
  });

  it('should update after delay', () => {
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      { initialProps: { value: 'initial', delay: 500 } },
    );

    rerender({ value: 'updated', delay: 500 });

    // Value should not change immediately
    expect(result.current).toBe('initial');

    // Advance time
    act(() => {
      jest.advanceTimersByTime(500);
    });

    expect(result.current).toBe('updated');
  });
});
```

**Key points:**

- Always use `jest.useFakeTimers()` in `beforeEach`
- Wrap `jest.runOnlyPendingTimers()` in `act()` in `afterEach` to avoid React warnings
- Use `act()` when advancing timers with `jest.advanceTimersByTime()`
- Test that values don't change immediately before the delay completes

#### B. React Hooks with Event Listeners

```typescript
describe('useMousePosition', () => {
  it('should track mouse position', () => {
    const { result } = renderHook(() => useMousePosition());

    act(() => {
      const mouseEvent = new MouseEvent('mousemove', {
        clientX: 100,
        clientY: 200,
      });
      window.dispatchEvent(mouseEvent);
    });

    expect(result.current.x).toBe(100);
    expect(result.current.y).toBe(200);
  });

  it('should clean up event listeners on unmount', () => {
    const removeEventListenerSpy = jest.spyOn(window, 'removeEventListener');
    const { unmount } = renderHook(() => useMousePosition());

    unmount();

    expect(removeEventListenerSpy).toHaveBeenCalledWith(
      'mousemove',
      expect.any(Function),
    );
    removeEventListenerSpy.mockRestore();
  });
});
```

#### C. NestJS Services

```typescript
// Mock dependencies BEFORE imports
jest.mock('@nestjs/common', () => {
  const actual = jest.requireActual('@nestjs/common');
  return {
    ...actual,
    Logger: {
      ...actual.Logger,
      error: jest.fn(),
      log: jest.fn(),
    },
  };
});
jest.mock('../dependencies');

import { Logger } from '@nestjs/common';
import { DependencyService } from '../dependencies';
import { MyService } from './my.service';

/**
 * Test suite for the MyService class.
 *
 * This service handles [description of functionality].
 * [Additional context]
 */
describe('MyService', () => {
  let service: MyService;
  let mockDependency: jest.Mocked<DependencyService>;

  beforeEach(() => {
    jest.clearAllMocks();

    mockDependency = {
      method: jest.fn().mockResolvedValue('value'),
    } as unknown as jest.Mocked<DependencyService>;

    service = new MyService(mockDependency);
  });

  afterEach(() => {
    if (service) {
      service.onModuleDestroy?.();
    }
  });

  describe('constructor', () => {
    it('should create an instance', () => {
      expect(service).toBeInstanceOf(MyService);
    });
  });

  describe('methodName', () => {
    it('should execute the method successfully', async () => {
      const result = await service.methodName('param');

      expect(result).toBeDefined();
      expect(mockDependency.method).toHaveBeenCalledWith('param');
    });

    it('should handle errors', async () => {
      mockDependency.method.mockRejectedValue(new Error('Test error'));

      await expect(service.methodName('param')).rejects.toThrow('Test error');
    });
  });
});
```

**Key points:**

- Mock dependencies BEFORE imports using `jest.mock()`
- Use `jest.requireActual()` to preserve original module when needed
- Create mock instances in `beforeEach`
- Test constructor, methods, error handling, and edge cases
- Clean up in `afterEach` if service implements `onModuleDestroy()`

#### D. NestJS Modules and Factories

```typescript
// Mock dependencies BEFORE imports
jest.mock('./dependency');
jest.mock('@nestjs/core');

import { ModuleFactory } from './module-factory';
import { Dependency } from './dependency';

describe('ModuleFactory', () => {
  let factory: ModuleFactory;
  let mockDependency: jest.Mocked<Dependency>;

  beforeEach(() => {
    jest.clearAllMocks();

    mockDependency = {
      method: jest.fn(),
    } as unknown as jest.Mocked<Dependency>;

    factory = new ModuleFactory(mockDependency);
  });

  describe('createModule', () => {
    it('should create a module with the provided configuration', () => {
      const config = { name: 'test' };
      const module = factory.createModule(config);

      expect(module).toBeDefined();
      expect(mockDependency.method).toHaveBeenCalledWith(config);
    });
  });
});
```

#### E. Simple State/Value Hooks (React)

```typescript
describe('usePrevious', () => {
  it('should return undefined for the first value', () => {
    const { result } = renderHook(() => usePrevious('initial'));
    expect(result.current).toBeUndefined();
  });

  it('should return the previous value after a change', () => {
    const { result, rerender } = renderHook(({ value }) => usePrevious(value), {
      initialProps: { value: 'first' },
    });

    expect(result.current).toBeUndefined();

    rerender({ value: 'second' });
    expect(result.current).toBe('first');

    rerender({ value: 'third' });
    expect(result.current).toBe('second');
  });
});
```

### 4. Essential Test Cases to Cover

#### For React Hooks:

1. **Initial state/behavior**: What happens on first render?
2. **Core functionality**: Does the hook do what it's supposed to do?
3. **State updates**: How does the hook respond to prop/state changes?
4. **Edge cases**: Null/undefined values, empty arrays, boundary conditions
5. **Cleanup**: Are event listeners, timers, and subscriptions properly cleaned up on unmount?
6. **Multiple rapid calls**: How does the hook handle rapid successive calls?
7. **Type safety**: Does it work with different value types (if applicable)?

#### For NestJS Services/Classes:

1. **Constructor**: Does it create an instance correctly?
2. **Core functionality**: Does each method do what it's supposed to do?
3. **Dependencies**: Are dependencies called correctly?
4. **Error handling**: How does the service handle errors?
5. **Edge cases**: Null/undefined values, empty arrays, boundary conditions
6. **Cleanup**: Is `onModuleDestroy()` implemented and tested?
7. **Async operations**: Are promises and async/await handled correctly?
8. **State management**: Does the service maintain state correctly?

### 5. Code Quality Standards

#### Language

- **All tests (React and NestJS)**: All test descriptions, comments, and console logs must be in **English**
  - Use clear, descriptive test names: `'should [expected behavior]'`
  - Use present tense: `'should return'` not `'should returns'`
  - All JSDoc comments must be in English
  - All inline comments must be in English
  - All console logs and error messages must be in English

#### Documentation

- Add JSDoc comments for each `describe` block explaining the code's purpose
- Add inline comments (`//`) for complex test logic or non-obvious assertions
- Document why certain test steps are necessary

#### Assertions

- Use specific matchers: `toBe()`, `toEqual()`, `toContain()`, `toHaveBeenCalled()`
- Prefer `toBe()` for primitives, `toEqual()` for objects/arrays
- Use `expect.any(Function)` when checking function types
- Use `expect.any(Constructor)` when checking instance types
- Use `expect.any(String)` when checking string types
- For NestJS: Use `toBeInstanceOf()` for class instances

#### Cleanup

- Always restore spies: `spy.mockRestore()`
- Remove DOM elements: `document.body.removeChild(element)` (React)
- Reset fake timers: `jest.useRealTimers()`
- Wrap timer cleanup in `act()` to avoid React warnings
- Call `service.onModuleDestroy?.()` in `afterEach` for NestJS services

### 6. Common Patterns

#### React: Testing with rerender

```typescript
const { result, rerender } = renderHook(({ value }) => useHook(value), {
  initialProps: { value: 'initial' },
});

rerender({ value: 'updated' });
expect(result.current).toBe('updated');
```

#### React: Testing cleanup

```typescript
const cleanupSpy = jest.spyOn(global, 'clearTimeout');
const { unmount } = renderHook(() => useHook());

unmount();

expect(cleanupSpy).toHaveBeenCalled();
cleanupSpy.mockRestore();
```

#### NestJS: Mocking dependencies

```typescript
// At the top of the file, BEFORE imports
jest.mock('@nestjs/common', () => {
  const actual = jest.requireActual('@nestjs/common');
  return {
    ...actual,
    Logger: {
      ...actual.Logger,
      error: jest.fn(),
    },
  };
});

jest.mock('../dependency', () => ({
  DependencyService: jest.fn(),
}));
```

#### NestJS: Testing async methods

```typescript
it('should handle async operations', async () => {
  mockDependency.method.mockResolvedValue('result');

  const result = await service.asyncMethod('param');

  expect(result).toBe('result');
  expect(mockDependency.method).toHaveBeenCalledWith('param');
});
```

#### NestJS: Testing error handling

```typescript
it('should propagate errors', async () => {
  const error = new Error('Test error');
  mockDependency.method.mockRejectedValue(error);

  await expect(service.method('param')).rejects.toThrow(error);
});
```

#### Testing with fake timers (React)

```typescript
beforeEach(() => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2024-01-15T12:00:00Z')); // If needed
});

afterEach(() => {
  act(() => {
    jest.runOnlyPendingTimers();
  });
  jest.useRealTimers();
});
```

#### Testing event handlers (React)

```typescript
act(() => {
  const event = new MouseEvent('click', { bubbles: true });
  element.dispatchEvent(event);
});
```

### 7. Configuration Requirements

#### React Context

- **Jest config** (`jest.config.js`):
  - `preset: 'ts-jest'`
  - `testEnvironment: 'jsdom'`
  - `tsconfig: './tsconfig.test.json'` in transform config
  - `moduleNameMapper` for React version resolution

- **TypeScript config** (`tsconfig.test.json`):
  - Extends source config
  - `moduleResolution: "node"` for proper module resolution
  - Includes test files: `"**/*.test.ts"`

- **Dependencies** (`package.json`):
  - `@testing-library/react`
  - `jest`
  - `jest-environment-jsdom`
  - `ts-jest`
  - `@types/jest`

#### NestJS Context

- **Jest config** (`jest.config.js`):
  - `preset: 'ts-jest'`
  - `testEnvironment: 'node'`
  - `tsconfig: './tsconfig.test.json'` in transform config
  - `moduleNameMapper` for path aliases (`@core`, `@interfaces`, etc.)

- **TypeScript config** (`tsconfig.test.json`):
  - Extends source config (`tsconfig.src.json`)
  - `moduleResolution: "node"`
  - `types: ["jest", "node"]`
  - Includes test files: `"**/*.test.ts"`

- **Dependencies** (`package.json`):
  - `jest`
  - `ts-jest`
  - `@types/jest`

### 8. Examples: Complete Test Files

#### React Hook Example

```typescript
import { act, renderHook } from '@testing-library/react';

import { useMyHook } from './my-hook';

/**
 * Test suite for the useMyHook hook.
 *
 * This hook [description of what it does].
 * [Additional context about behavior, edge cases, or special considerations]
 */
describe('useMyHook', () => {
  beforeEach(() => {
    // Setup if needed (e.g., jest.useFakeTimers())
  });

  afterEach(() => {
    // Cleanup if needed (e.g., jest.useRealTimers())
  });

  it('should return initial value', () => {
    const { result } = renderHook(() => useMyHook('initial'));
    expect(result.current).toBe('initial');
  });

  it('should update when props change', () => {
    const { result, rerender } = renderHook(({ value }) => useMyHook(value), {
      initialProps: { value: 'first' },
    });

    rerender({ value: 'second' });
    expect(result.current).toBe('second');
  });

  it('should clean up on unmount', () => {
    const cleanupSpy = jest.spyOn(global, 'clearTimeout');
    const { unmount } = renderHook(() => useMyHook('value'));

    unmount();

    expect(cleanupSpy).toHaveBeenCalled();
    cleanupSpy.mockRestore();
  });
});
```

#### NestJS Service Example

```typescript
// Mock dependencies BEFORE imports
jest.mock('@nestjs/common', () => {
  const actual = jest.requireActual('@nestjs/common');
  return {
    ...actual,
    Logger: {
      ...actual.Logger,
      error: jest.fn(),
      log: jest.fn(),
    },
  };
});
jest.mock('../dependency');

import { Logger } from '@nestjs/common';
import { DependencyService } from '../dependency';
import { MyService } from './my.service';

/**
 * Test suite for the MyService class.
 *
 * This service handles [description of functionality].
 * [Additional context about behavior, edge cases, or special considerations]
 */
describe('MyService', () => {
  let service: MyService;
  let mockDependency: jest.Mocked<DependencyService>;

  beforeEach(() => {
    jest.clearAllMocks();

    mockDependency = {
      method: jest.fn().mockResolvedValue('value'),
    } as unknown as jest.Mocked<DependencyService>;

    service = new MyService(mockDependency);
  });

  afterEach(() => {
    if (service) {
      service.onModuleDestroy?.();
    }
  });

  describe('constructor', () => {
    it('should create an instance', () => {
      expect(service).toBeInstanceOf(MyService);
    });
  });

  describe('methodName', () => {
    it('should execute the method successfully', async () => {
      const result = await service.methodName('param');

      expect(result).toBeDefined();
      expect(mockDependency.method).toHaveBeenCalledWith('param');
    });

    it('should handle errors', async () => {
      mockDependency.method.mockRejectedValue(new Error('Test error'));

      await expect(service.methodName('param')).rejects.toThrow('Test error');
    });
  });
});
```

## Instructions

When generating tests:

1. **Identify the context**: Determine if you're testing React hooks or NestJS services/classes
2. **Read the source file** to understand the implementation
3. **Identify the type** (timer-based, event-based, service, module, etc.)
4. **Follow the appropriate pattern** from section 3
5. **Cover all essential test cases** from section 4
6. **Add documentation** following section 5
7. **Use the correct language**:
   - **English** for all tests (React and NestJS)
   - All test descriptions, comments, JSDoc, and console logs must be in English
8. **Verify cleanup** is tested for timers, event listeners, subscriptions, and `onModuleDestroy()`
9. **Test edge cases** and different data types where applicable
10. **Use `act()`** for all React state updates, timer advances, and event dispatches
11. **Mock dependencies** BEFORE imports for NestJS tests
12. **Run tests and fix errors**:
    - After generating the test file, **ALWAYS** run the tests immediately: `pnpm test <test-file-name>`
    - If any test fails, analyze the error message and fix the issue
    - Common issues to check:
      - Missing mocks or incorrect mock setup
      - Incorrect imports or missing dependencies
      - Test isolation issues (state persisting between tests)
      - Incorrect assertions or expectations
      - Missing cleanup in `afterEach` hooks
      - Timer-related issues (use `jest.useFakeTimers()` and `jest.useRealTimers()`)
      - Event listener cleanup issues
      - Async/await handling problems
    - **Continue fixing errors until ALL tests pass** - do not stop until the test suite is completely green
    - If errors persist, examine similar test files in the codebase to understand the correct patterns

## Output Format

Generate a complete test file with:

- Proper imports (with mocks BEFORE imports for NestJS)
- JSDoc documentation for the test suite (in English)
- All essential test cases
- Proper setup/teardown
- Cleanup verification
- Correct language (English for all tests, comments, and console logs)
- Comments explaining complex logic (in English)

## Testing and Debugging Workflow

**CRITICAL**: After generating the test file, you MUST:

1. **Run the tests immediately**:

   ```bash
   pnpm test <test-file-name>
   ```

   Or for specific file patterns:

   ```bash
   pnpm test my-service.test.ts
   ```

2. **Analyze any failures**:
   - Read the error messages carefully
   - Check stack traces to identify the exact failing line
   - Understand what the test expected vs what actually happened

3. **Fix errors iteratively**:
   - Fix one error at a time
   - Re-run tests after each fix
   - Common fixes include:
     - Adding missing mocks or correcting mock implementations
     - Fixing test isolation (ensuring clean state between tests)
     - Correcting assertion expectations
     - Adding proper cleanup in `afterEach` hooks
     - Fixing async/await handling
     - Resolving timer-related issues
     - Correcting import statements

4. **Verify all tests pass**:
   - **DO NOT stop until ALL tests pass**
   - The test suite must show: `Tests: X passed, X total`
   - No failures should remain

5. **Common Error Patterns and Solutions**:

   **Mock not found**:
   - Ensure mocks are defined BEFORE imports
   - Check mock path and structure
   - Verify `jest.mock()` calls are correct

   **State persisting between tests**:
   - Add `middleware.removeAllListeners()` in `beforeEach` or `afterEach`
   - Clear all mocks with `jest.clearAllMocks()` in `beforeEach`
   - Ensure proper cleanup in `afterEach` hooks

   **Timer-related errors**:
   - Use `jest.useFakeTimers()` in `beforeEach`
   - Use `jest.useRealTimers()` in `afterEach`
   - For React: wrap timer cleanup in `act()`

   **clearTimeout/setTimeout not defined**:
   - Use `global.clearTimeout` or check if using fake timers
   - Ensure timer functions are properly mocked for tests

   **Listener/event-related errors**:
   - Ensure all listeners are removed in `afterEach`
   - Use unique event names in tests to avoid collisions
   - Verify event emitter cleanup

6. **Final verification**:
   - Run the full test suite if needed: `pnpm test`
   - Ensure no test file is left with failures
   - All assertions should pass

**Remember**: A test file with failing tests is not complete. You must fix all errors before considering the task done.
