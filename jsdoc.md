# Generate Complete JSDoc Documentation

## Instructions

You are tasked with generating comprehensive JSDoc documentation in English for the provided TypeScript/JavaScript file. The documentation must be complete, detailed, and follow JSDoc standards.

## General Requirements

1. **Language**: All documentation must be written in English
2. **Completeness**: Document ALL exported interfaces, types, functions, components, hooks, and constants
3. **Detail Level**: Provide maximum information about each element including:
   - Purpose and behavior
   - All parameters and their descriptions
   - Return values and types
   - Usage examples
   - Important notes and edge cases
   - Relationships with other code elements

## JSDoc Format Standards

### Important Rules

- **DO NOT** include TypeScript types in JSDoc tags (e.g., `@param {string} name` → `@param name`)
- TypeScript already infers types from function signatures and interfaces
- Use JSDoc tags for descriptions only, not type annotations
- Format: `@param paramName - Description` (not `@param {Type} paramName`)

### Required Tags

Use standard JSDoc tags appropriately:

- `@param` - For function/component parameters (without type annotations)
- `@returns` - For return value descriptions (without type annotations)
- `@example` - For code examples showing usage
- `@remarks` - For important notes, warnings, or additional context
- `@interface` - For interface declarations
- `@property` - For interface/type properties (without type annotations)
- `@extends` - For extended interfaces (without type annotations)
- `@throws` - For documented exceptions (if applicable)

## Documentation Requirements by Element Type

### Interfaces and Types

For each interface or type:

- Provide a comprehensive description explaining its purpose at the interface/type level
- **CRITICAL**: Each property MUST have its own individual JSDoc comment inline, placed directly before the property declaration
- This ensures IDEs can display property-specific documentation when accessing individual properties
- Document every property using `@property` tags in the interface-level comment (for summary)
- Explain relationships if it extends other interfaces
- Include examples showing how to create/use instances

**Example structure:**

````typescript
/**
 * [Comprehensive description of the interface's purpose and usage]
 *
 * @interface InterfaceName
 * @extends ParentInterface (if applicable)
 * @property propertyName - Description of the property
 * @property optionalProperty - Description of optional property
 *
 * @example
 * ```typescript
 * const instance: InterfaceName = {
 *   propertyName: 'value',
 *   optionalProperty: 'optional value'
 * };
 * ```
 */
export interface InterfaceName {
  /**
   * Detailed description of this specific property.
   * This comment will be displayed by the IDE when accessing this property.
   *
   * @remarks
   * - Additional notes about this property
   * - Usage constraints or requirements
   */
  propertyName: string;

  /**
   * Detailed description of this optional property.
   * This comment will be displayed by the IDE when accessing this property.
   *
   * @remarks
   * - When this property should be used
   * - Default behavior if not provided
   */
  optionalProperty?: string;
}
````

**Important**: The interface-level comment provides the overview, while individual property comments provide detailed documentation that IDEs can display when hovering over or accessing specific properties.

### Functions and Methods

For each function or method:

- Provide a detailed description of what it does
- Document all parameters with `@param` (without types)
- Document the return value with `@returns` (without types)
- Include practical usage examples
- Add `@remarks` for important behavior notes

**Example structure:**

````typescript
/**
 * [Detailed description of the function's purpose and behavior]
 *
 * @param paramName - Description of the parameter
 * @param optionalParam - Description of optional parameter
 *
 * @returns Description of what is returned
 *
 * @remarks
 * - Important note about behavior
 * - Edge case or limitation
 *
 * @example
 * ```typescript
 * const result = functionName('value1', 'value2');
 * ```
 */
````

### React Components

For React components:

- Describe the component's purpose and when to use it
- Document all props (if using PropsWithChildren, document children)
- Explain the return value (JSX.Element)
- Include complete usage examples
- Document any special behavior, lifecycle, or side effects

**Example structure:**

````typescript
/**
 * [Comprehensive description of the component's purpose and usage]
 *
 * [Additional context about when and how to use this component]
 *
 * @param props - The component props
 * @param props.propName - Description of the prop
 * @param props.children - Description of children (if applicable)
 *
 * @returns The rendered component
 *
 * @remarks
 * - Important behavior notes
 * - Performance considerations
 * - Usage constraints
 *
 * @example
 * ```tsx
 * <ComponentName propName="value">
 *   <ChildComponent />
 * </ComponentName>
 * ```
 */
````

### React Hooks

For custom hooks:

- Explain what the hook does and when to use it
- Document all parameters
- Document the return value (usually an object with multiple properties)
- Include complete usage examples
- Note any dependencies or requirements

**Example structure:**

````typescript
/**
 * [Description of the hook's purpose and functionality]
 *
 * [Additional context about when and how to use this hook]
 *
 * @param paramName - Description of the parameter
 *
 * @returns Object containing hook values and methods
 * @returns returns.propertyName - Description of returned property
 *
 * @remarks
 * - Requirements or constraints
 * - Important behavior notes
 *
 * @example
 * ```tsx
 * function MyComponent() {
 *   const { propertyName, methodName } = useHookName(param);
 *   // usage
 * }
 * ```
 */
````

### React Contexts

For React contexts:

- Explain the context's purpose and what it provides
- Document the context value type and all its properties
- Explain when and how to use the context
- Include examples showing provider and consumer usage
- Note any requirements (e.g., must be used within a provider)

**Example structure:**

````typescript
/**
 * [Description of the context's purpose and what it provides]
 *
 * @interface ContextValueType
 * @property propertyName - Description of the context property
 *
 * @example
 * ```tsx
 * // Provider usage
 * <ContextProvider>
 *   <App />
 * </ContextProvider>
 *
 * // Consumer usage
 * const context = useContext(ContextName);
 * ```
 */
````

### Constants and Variables

For exported constants:

- Explain the constant's purpose and value
- Note if it's used in specific contexts
- Include usage examples if applicable

## Analysis Instructions

When analyzing the provided file:

1. **Identify all exports**: List every exported interface, type, function, component, hook, constant
2. **Check existing documentation**: Identify what's already documented and what's missing
3. **Analyze dependencies**: Understand relationships with imported types and modules
4. **Document missing elements**: Focus on elements with no or incomplete JSDoc
5. **For interfaces/types**: Ensure EACH property has its own inline JSDoc comment (not just @property tags in the interface comment)
6. **Enhance existing docs**: Improve incomplete documentation with more details
7. **Add examples**: Include practical, real-world usage examples for each element
8. **Add remarks**: Include important notes about behavior, edge cases, or usage patterns

## Output Format

Provide the complete JSDoc documentation in the following format:

1. For each element that needs documentation, show:
   - The complete JSDoc comment block
   - The code element it documents (for context)

2. If an element already has documentation but it's incomplete:
   - Show the improved/complete version
   - Explain what was added or enhanced

3. Group documentation by element type for clarity

## Quality Checklist

Before finalizing, ensure:

- [ ] All exports are documented
- [ ] No TypeScript types in JSDoc tags
- [ ] All parameters are documented
- [ ] Return values are documented
- [ ] **All interface/type properties have individual inline JSDoc comments**
- [ ] Examples are included and practical
- [ ] Important notes are in @remarks
- [ ] Descriptions are clear and comprehensive
- [ ] All documentation is in English
- [ ] Code examples are syntactically correct

## Example Output Structure

```typescript
/**
 * [Complete interface documentation here]
 *
 * @interface ExampleInterface
 * @property propertyName - Summary description
 */
export interface ExampleInterface {
  /**
   * [Detailed property documentation here]
   * This will be displayed by the IDE when accessing this property.
   */
  propertyName: string;
}

/**
 * [Complete documentation here]
 */
export function exampleFunction() {
  // ...
}
```

---

**Now, analyze the provided file and generate complete JSDoc documentation for all exported elements, following all the guidelines above.**
