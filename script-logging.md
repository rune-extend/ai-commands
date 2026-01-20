# Comprehensive Logging Best Practices

Generate well-structured, organized logging code for Node.js scripts following best practices for readability, debugging, and user experience.

## Instructions

When adding or improving logging in a script:

1. **Analyze the script structure** to identify logical phases/steps
2. **Implement hierarchical grouping** using `console.group()` and `console.groupEnd()`
3. **Separate normal vs debug logs** using a `debugLog()` function
4. **Use appropriate log levels** (console.log, console.warn, console.error)
5. **Add contextual information** (file sizes, durations, URLs, etc.)
6. **Ensure proper cleanup** of groups in error scenarios
7. **Follow emoji conventions** for visual clarity
8. **Add progress indicators** for multi-step operations
9. **Implement conditional colorization** (if terminal supports colors)
10. **Generate log output examples** at the end showing how logs appear in different scenarios

## Core Principles

### 1. Hierarchical Grouping

Organize logs into logical, visually nested groups:

```javascript
console.group('🚀 Main Process');
console.group('⚙️  Initialization');
// Initialization logs
console.groupEnd();

console.group('📋 Discovery Phase');
// Discovery logs
console.groupEnd();

console.group('📦 Processing Phase');
console.group('📱 Item 1');
// Item-specific logs
console.groupEnd();
console.groupEnd();
console.groupEnd();
```

**Rules:**

- Each major phase/step should have its own group
- Use descriptive emojis for quick visual identification
- Always close groups with `console.groupEnd()` in the same order they were opened
- Close groups even in error scenarios (before throwing errors)

### 2. Debug vs Normal Logs

**Normal logs (`console.log`, `console.warn`, `console.error`)**: Information that should always be visible

- Progress indicators
- Success/error messages
- Summary information
- Important state changes

**Debug logs (`debugLog()`)**: Technical details only visible in debug mode

- File paths and directories
- Configuration values
- Internal state details
- Verbose operation details
- Stack traces (in error handlers)

**Implementation:**

```javascript
// In a utility file (e.g., release-utils.js)
function isDebugMode() {
  return !!(
    process.env.DEBUG ||
    process.env.ACTIONS_STEP_DEBUG === 'true' ||
    process.env.ACTIONS_RUNNER_DEBUG === 'true'
  );
}

function debugLog(message) {
  if (isDebugMode()) {
    console.log(`🔍 [DEBUG] ${message}`);
  }
}
```

**When to use `debugLog()`:**

- File system paths and directories
- Environment variable values (sensitive data should be masked)
- Internal calculations or state
- Detailed step-by-step operations
- Verbose filtering or processing details
- API request details (without sensitive tokens)

**When to use `console.log()`:**

- Progress indicators (✅, ⏭️, 🔨, etc.)
- Summary counts ("Found 3 items")
- Success confirmations
- User-facing status updates
- Final results and outputs

### 3. Error Handling with Groups

Always ensure groups are closed before throwing errors:

```javascript
console.group('⚙️  Initialization');
try {
  // Initialization code
  if (!process.env.REQUIRED_VAR) {
    console.groupEnd(); // Close group before error
    throw new Error('REQUIRED_VAR is missing');
  }
  console.groupEnd();
} catch (err) {
  console.groupEnd(); // Ensure cleanup in catch
  throw err;
}
```

**Pattern for early returns:**

```javascript
if (items.length === 0) {
  console.log('ℹ️  No items found');
  console.groupEnd();
  console.groupEnd(); // Close all nested groups
  return;
}
```

### 4. Emoji Conventions

Use consistent emojis for different log types:

| Emoji | Usage                        | Example                                       |
| ----- | ---------------------------- | --------------------------------------------- |
| 🚀    | Main process start           | `console.group('🚀 Process Name')`            |
| ⚙️    | Configuration/initialization | `console.group('⚙️  Initialization')`         |
| 📋    | Discovery/search phase       | `console.group('📋 App Discovery')`           |
| 🔨    | Build/compilation            | `console.group('🔨 Building')`                |
| 📦    | Archive/package creation     | `console.group('📦 Creating Archive')`        |
| 🏷️    | Tag/version checking         | `console.group('🏷️  Tag Checking')`           |
| 📝    | Reading/writing files        | `console.group('📝 Preparing Release Notes')` |
| 📤    | Upload/publish               | `console.group('📤 Creating GitHub Release')` |
| ⬆️    | Upload operation             | `console.group('⬆️  Uploading Asset')`        |
| 📊    | Summary/statistics           | `console.group('📊 Summary')`                 |
| 🔧    | Technical/system             | `console.group('🔧 GitHub Actions Output')`   |
| ✅    | Success                      | `console.log('✅ Operation completed')`       |
| ⏭️    | Skip/skip operation          | `console.log('⏭️  Skipping...')`              |
| ❌    | Error                        | `console.error('❌ Operation failed')`        |
| ⚠️    | Warning                      | `console.warn('⚠️  Warning message')`         |
| ℹ️    | Information                  | `console.log('ℹ️  Info message')`             |
| 🔍    | Debug information            | `debugLog('Debug message')`                   |

### 5. Contextual Information

Add relevant context to logs:

**File sizes:**

```javascript
const fileSize = statSync(filePath).size;
console.log(
  `✅ Asset uploaded: ${fileName} (${(fileSize / 1024 / 1024).toFixed(2)} MB)`,
);
```

**Durations:**

```javascript
const startTime = Date.now();
// ... operation ...
const duration = ((Date.now() - startTime) / 1000).toFixed(2);
debugLog(`Operation completed in ${duration}s`);
```

**URLs and links:**

```javascript
console.log(`✅ Release created: ${release.html_url}`);
```

**Counts and statistics:**

```javascript
console.log(`Found ${items.length} item(s):`);
items.forEach(item => {
  console.log(`  - ${item.name}@${item.version}`);
});
```

### 6. Progress Indicators (Advanced)

Show progress for multi-step operations with clear indicators:

**Basic Progress Counter:**

```javascript
console.group(`📦 Processing ${items.length} item(s)`);
items.forEach((item, index) => {
  const current = index + 1;
  const total = items.length;
  const progress = `${current}/${total}`;
  console.group(`📱 ${item.name} (${progress})`);
  await processItem(item);
  console.log(`✅ Processed (${progress})`);
  console.groupEnd();
});
console.groupEnd();
```

**Progress with Percentage:**

```javascript
console.group(`📦 Processing ${items.length} item(s)`);
items.forEach((item, index) => {
  const current = index + 1;
  const total = items.length;
  const percentage = Math.round((current / total) * 100);
  console.group(`📱 ${item.name} (${current}/${total} - ${percentage}%)`);
  await processItem(item);
  console.log(`✅ Processed (${percentage}%)`);
  console.groupEnd();
});
console.groupEnd();
```

**Visual Progress Bar (Simple ASCII):**

```javascript
function createProgressBar(current, total, width = 20) {
  const percentage = current / total;
  const filled = Math.round(percentage * width);
  const empty = width - filled;
  const bar = '█'.repeat(filled) + '░'.repeat(empty);
  const percent = Math.round(percentage * 100);
  return `[${bar}] ${percent}%`;
}

console.group(`📦 Processing ${items.length} item(s)`);
items.forEach((item, index) => {
  const current = index + 1;
  const progressBar = createProgressBar(current, items.length);
  console.group(`📱 ${item.name} ${progressBar}`);
  await processItem(item);
  console.log(`✅ Completed`);
  console.groupEnd();
});
console.groupEnd();
```

**Progress for Async Operations:**

```javascript
console.group(`📦 Processing ${items.length} item(s)`);
for (let i = 0; i < items.length; i++) {
  const item = items[i];
  const progress = `${i + 1}/${items.length}`;
  console.group(`📱 ${item.name} (${progress})`);

  // Show progress during async operation
  console.log(`⏳ Processing...`);
  await processItem(item);

  console.log(`✅ Completed (${progress})`);
  console.groupEnd();
}
console.groupEnd();
```

### 7. Summary Sections

Always include a summary at the end:

```javascript
console.group('📊 Summary');
console.log(`✅ Successfully processed ${successCount} item(s):`);
successItems.forEach(item => {
  console.log(`  - ${item.name}`);
});

if (failedItems.length > 0) {
  console.error(`❌ Failed to process ${failedItems.length} item(s):`);
  failedItems.forEach(({ item, error }) => {
    console.error(`  - ${item.name}: ${error}`);
  });
}
console.groupEnd();
```

### 8. Conditional Colorization

```javascript
console.group('📊 Summary');
console.log(`✅ Successfully processed ${successCount} item(s):`);
successItems.forEach(item => {
  console.log(`  - ${item.name}`);
});

if (failedItems.length > 0) {
  console.error(`❌ Failed to process ${failedItems.length} item(s):`);
  failedItems.forEach(({ item, error }) => {
    console.error(`  - ${item.name}: ${error}`);
  });
}
console.groupEnd();
```

## Pattern Examples

### Pattern 1: Simple Script with Phases

```javascript
async function main() {
  try {
    console.group('🚀 Script Process');

    // Phase 1: Initialization
    console.group('⚙️  Initialization');
    debugLog('Environment check:');
    debugLog(`  VAR1: ${process.env.VAR1 ? 'SET' : 'NOT SET'}`);

    if (!process.env.REQUIRED_VAR) {
      console.groupEnd();
      console.groupEnd();
      throw new Error('REQUIRED_VAR is required');
    }
    console.groupEnd();

    // Phase 2: Discovery
    console.group('📋 Discovery');
    const items = findItems();
    debugLog(`Found ${items.length} total items`);
    console.log(`Found ${items.length} item(s):`);
    items.forEach(item => console.log(`  - ${item.name}`));
    console.groupEnd();

    // Phase 3: Processing
    console.group(`📦 Processing (${items.length} item(s))`);
    for (const item of items) {
      console.group(`📱 ${item.name}`);
      await processItem(item);
      console.log(`✅ Processed ${item.name}`);
      console.groupEnd();
    }
    console.groupEnd();

    // Summary
    console.group('📊 Summary');
    console.log('🎉 Process completed successfully!');
    console.groupEnd();

    console.groupEnd();
  } catch (err) {
    handleError(err, 'Process failed');
  }
}
```

### Pattern 2: Script with Build Steps

```javascript
async function buildAndRelease() {
  console.group('🚀 Build and Release Process');

  console.group('⚙️  Initialization');
  // Validation
  console.groupEnd();

  console.group('🔨 Building');
  const appDir = resolve('./app');
  debugLog(`Building in directory: ${appDir}`);
  debugLog(`Build command: npm run build`);

  execSync('npm run build', { cwd: appDir, stdio: 'inherit' });
  console.log('✅ Build completed');
  console.groupEnd();

  console.group('📦 Creating Archive');
  const archivePath = await createArchive(appDir);
  const fileSize = statSync(archivePath).size;
  console.log(
    `✅ Archive created: ${basename(archivePath)} (${(fileSize / 1024 / 1024).toFixed(2)} MB)`,
  );
  console.groupEnd();

  console.group('📤 Publishing');
  await publish(archivePath);
  console.log('✅ Published successfully');
  console.groupEnd();

  console.groupEnd();
}
```

### Pattern 3: Error Handling with Multiple Groups

```javascript
async function processWithErrorHandling() {
  try {
    console.group('🚀 Process');

    console.group('⚙️  Initialization');
    if (!validateConfig()) {
      console.groupEnd();
      console.groupEnd();
      throw new Error('Invalid configuration');
    }
    console.groupEnd();

    console.group('📋 Discovery');
    const items = await findItems();
    if (items.length === 0) {
      console.log('ℹ️  No items found');
      console.groupEnd();
      console.groupEnd();
      return;
    }
    console.groupEnd();

    console.group(`📦 Processing (${items.length} item(s))`);
    const results = [];
    for (const item of items) {
      console.group(`📱 ${item.name}`);
      try {
        const result = await processItem(item);
        console.log(`✅ Processed successfully`);
        results.push({ item, success: true });
        console.groupEnd();
      } catch (err) {
        console.error(`❌ Failed: ${err.message}`);
        console.groupEnd();
        results.push({ item, success: false, error: err.message });
      }
    }
    console.groupEnd();

    // Summary with error details
    console.group('📊 Summary');
    const successful = results.filter(r => r.success);
    const failed = results.filter(r => !r.success);

    if (successful.length > 0) {
      console.log(`✅ Successfully processed ${successful.length} item(s)`);
    }

    if (failed.length > 0) {
      console.error(`❌ Failed to process ${failed.length} item(s):`);
      failed.forEach(({ item, error }) => {
        console.error(`  - ${item.name}: ${error}`);
      });
      console.groupEnd();
      console.groupEnd();
      throw new Error(`Process completed with ${failed.length} failure(s)`);
    }

    console.log('🎉 All items processed successfully!');
    console.groupEnd();
    console.groupEnd();
  } catch (err) {
    handleError(err, 'Process failed');
  }
}
```

## Best Practices

### DO

✅ **Use groups for major phases**: Each logical step should have its own group
✅ **Keep normal logs concise**: Users should see progress, not technical details
✅ **Use debugLog for technical details**: Paths, config values, internal state
✅ **Close groups before errors**: Always ensure cleanup in error scenarios
✅ **Add context to success messages**: File sizes, counts, URLs when relevant
✅ **Use consistent emoji patterns**: Makes logs scannable and professional
✅ **Include summary sections**: Users need to see what happened overall
✅ **Format large numbers**: Bytes → MB, milliseconds → seconds with 2 decimals

### DON'T

❌ **Don't show technical details in normal mode**: Keep them in debug
❌ **Don't forget to close groups**: Especially in error paths
❌ **Don't use groups for single log statements**: Groups are for related logs
❌ **Don't expose sensitive data**: Mask tokens, passwords, secrets even in debug
❌ **Don't create too deep nesting**: Max 3-4 levels of groups
❌ **Don't use inconsistent emojis**: Follow the conventions
❌ **Don't skip summary**: Always show what was accomplished
❌ **Don't forget error context**: Include what failed and why

## Advanced Features

Use ANSI color codes to enhance log readability when terminal supports colors:

**Implementation:**

```javascript
// Check if colors are supported (terminal with TTY and not CI without color support)
function supportsColor() {
  if (process.env.NO_COLOR || process.env.FORCE_COLOR === '0') {
    return false;
  }
  if (process.env.FORCE_COLOR === '1' || process.env.FORCE_COLOR === 'true') {
    return true;
  }
  // Check if stdout is a TTY (terminal)
  return process.stdout.isTTY === true;
}

// ANSI color codes
const Colors = {
  Reset: '\x1b[0m',
  Bright: '\x1b[1m',
  Dim: '\x1b[2m',
  Red: '\x1b[31m',
  Green: '\x1b[32m',
  Yellow: '\x1b[33m',
  Blue: '\x1b[34m',
  Magenta: '\x1b[35m',
  Cyan: '\x1b[36m',
  Gray: '\x1b[90m',
  BrightGreen: '\x1b[92m',
  BrightYellow: '\x1b[93m',
  BrightCyan: '\x1b[96m',
};

// Colorize function (only if colors are supported)
function colorize(text, color) {
  return supportsColor() ? `${color}${text}${Colors.Reset}` : text;
}

// Enhanced logging functions
function logSuccess(message) {
  console.log(colorize(`✅ ${message}`, Colors.BrightGreen));
}

function logError(message) {
  console.error(colorize(`❌ ${message}`, Colors.Red));
}

function logWarning(message) {
  console.warn(colorize(`⚠️  ${message}`, Colors.BrightYellow));
}

function logInfo(message) {
  console.log(colorize(`ℹ️  ${message}`, Colors.BrightCyan));
}

function logDebug(message) {
  if (isDebugMode()) {
    console.log(colorize(`🔍 [DEBUG] ${message}`, Colors.Gray));
  }
}
```

**Usage in groups:**

```javascript
console.group(colorize('🚀 Process Name', Colors.Bright + Colors.Cyan));
console.group(colorize('⚙️  Initialization', Colors.Blue));
logSuccess('Configuration loaded');
logInfo('Environment validated');
console.groupEnd();

console.group(colorize('📦 Processing', Colors.Magenta));
logSuccess('Item processed successfully');
logWarning('Some items were skipped');
console.groupEnd();
console.groupEnd();
```

**Color Recommendations:**

- **Success messages (✅)**: `Colors.BrightGreen` or `Colors.Green`
- **Error messages (❌)**: `Colors.Red` or `Colors.BrightRed`
- **Warning messages (⚠️)**: `Colors.BrightYellow` or `Colors.Yellow`
- **Info messages (ℹ️)**: `Colors.BrightCyan` or `Colors.Cyan`
- **Debug messages**: `Colors.Gray`
- **Group headers**: `Colors.Bright` + `Colors.Blue`/`Colors.Magenta`/`Colors.Cyan`
- **Progress indicators**: `Colors.BrightCyan`

**Important Notes:**

- Always check color support before applying colors
- Respect `NO_COLOR` and `FORCE_COLOR` environment variables
- Colors should enhance readability, not distract
- In CI/CD environments, colors may not be supported (GitHub Actions, etc.)
- Fallback gracefully when colors aren't supported

**Complete colorization utility example:**

```javascript
// Utility functions for colorized logging
function supportsColor() {
  if (process.env.NO_COLOR || process.env.FORCE_COLOR === '0') return false;
  if (process.env.FORCE_COLOR === '1' || process.env.FORCE_COLOR === 'true')
    return true;
  return process.stdout.isTTY === true;
}

const Colors = {
  Reset: '\x1b[0m',
  Bright: '\x1b[1m',
  Red: '\x1b[31m',
  Green: '\x1b[32m',
  Yellow: '\x1b[33m',
  Blue: '\x1b[34m',
  Cyan: '\x1b[36m',
  Gray: '\x1b[90m',
  BrightGreen: '\x1b[92m',
  BrightYellow: '\x1b[93m',
  BrightCyan: '\x1b[96m',
};

function colorize(text, color) {
  return supportsColor() ? `${color}${text}${Colors.Reset}` : text;
}

// Enhanced logging with colors
function logSuccess(message) {
  console.log(colorize(`✅ ${message}`, Colors.BrightGreen));
}

function logError(message) {
  console.error(colorize(`❌ ${message}`, Colors.Red));
}

function logWarning(message) {
  console.warn(colorize(`⚠️  ${message}`, Colors.BrightYellow));
}

function logInfo(message) {
  console.log(colorize(`ℹ️  ${message}`, Colors.BrightCyan));
}

function logDebug(message) {
  if (isDebugMode()) {
    console.log(colorize(`🔍 [DEBUG] ${message}`, Colors.Gray));
  }
}

// Usage in groups
console.group(colorize('🚀 Process Name', Colors.Bright + Colors.Cyan));
logSuccess('Operation completed');
logInfo('Processing items');
logWarning('Some items were skipped');
console.groupEnd();
```

### Combining Features

**Progress indicators with colorization:**

```javascript
console.group(colorize(`📦 Processing ${items.length} item(s)`, Colors.Magenta));
items.forEach((item, index) => {
  const current = index + 1;
  const total = items.length;
  const percentage = Math.round((current / total) * 100);
  const progress = colorize(`(${current}/${total} - ${percentage}%)`, Colors.BrightCyan);

  console.group(`📱 ${item.name} ${progress}`);
  await processItem(item);
  logSuccess(`Completed (${percentage}%)`);
  console.groupEnd();
});
console.groupEnd();
```

### Timing Operations

Add timing information in debug mode:

```javascript
console.group('🔨 Building');
const startTime = Date.now();
debugLog(`Build started at ${new Date(startTime).toISOString()}`);

await build();

const duration = ((Date.now() - startTime) / 1000).toFixed(2);
debugLog(`Build completed in ${duration}s`);
console.log('✅ Build completed');
console.groupEnd();
```

### Conditional Summary

Only show summary sections that are relevant:

```javascript
console.group('📊 Summary');
if (createdItems.length > 0) {
  console.log(`✅ Created ${createdItems.length} item(s):`);
  createdItems.forEach(item => console.log(`  - ${item.name}`));
}

if (skippedItems.length > 0) {
  console.log(`⏭️  Skipped ${skippedItems.length} item(s):`);
  skippedItems.forEach(item => console.log(`  - ${item.name}`));
}

if (failedItems.length === 0) {
  console.log('🎉 All operations completed successfully!');
}
console.groupEnd();
```

### Statistics Display

Show aggregated statistics:

```javascript
console.group('📊 Summary');
console.log(
  `Total: ${totalCount} | Success: ${successCount} | Failed: ${failedCount} | Skipped: ${skippedCount}`,
);
if (totalSize > 0) {
  console.log(`Total size: ${(totalSize / 1024 / 1024).toFixed(2)} MB`);
}
console.groupEnd();
```

## Integration with Error Handlers

Ensure error handlers close all groups:

```javascript
function handleError(err, context = null) {
  const message = context ? `${context}: ${err.message}` : err.message;
  console.error(`❌ ${message}`);
  if (err.stack && isDebugMode()) {
    console.error(err.stack);
  }
  // Ensure all groups are closed before exit
  // (groups should be closed before calling handleError)
  process.exit(1);
}
```

## Checklist

When implementing logging:

- [ ] All major phases have groups
- [ ] Groups are properly closed (including error paths)
- [ ] Technical details use `debugLog()`
- [ ] Progress indicators implemented (counters, percentages, or progress bars)
- [ ] Colorization implemented (with proper color support detection)
- [ ] Error messages are clear and actionable
- [ ] Summary section shows overall results
- [ ] Sensitive data is masked (even in debug)
- [ ] File sizes and counts are formatted appropriately
- [ ] Consistent emoji usage throughout
- [ ] Logs are readable in both normal and debug modes
- [ ] Example log outputs generated and displayed for different scenarios

## Example: Complete Script

```javascript
const { debugLog, isDebugMode } = require('./utils');

async function main() {
  try {
    console.group('🚀 Complete Process');

    // Initialization
    console.group('⚙️  Initialization');
    debugLog('Environment check:');
    debugLog(`  NODE_ENV: ${process.env.NODE_ENV || 'development'}`);

    if (!process.env.REQUIRED_VAR) {
      console.groupEnd();
      console.groupEnd();
      throw new Error('REQUIRED_VAR environment variable is required');
    }
    console.groupEnd();

    // Discovery
    console.group('📋 Discovery');
    const items = await discoverItems();
    debugLog(`Found ${items.total} total items`);
    debugLog(`Filtering criteria: ${JSON.stringify(criteria)}`);

    const filteredItems = items.filter(/* ... */);
    console.log(`Found ${filteredItems.length} matching item(s):`);
    filteredItems.forEach(item => {
      debugLog(`  - ${item.name} (path: ${item.path})`);
      console.log(`  - ${item.name}@${item.version}`);
    });
    console.groupEnd();

    // Processing
    if (filteredItems.length === 0) {
      console.log('ℹ️  No items to process');
      console.groupEnd();
      return;
    }

    console.group(`📦 Processing (${filteredItems.length} item(s))`);
    const results = [];

    for (const item of filteredItems) {
      console.group(`📱 ${item.name}@${item.version}`);
      const startTime = Date.now();

      try {
        debugLog(`Processing ${item.name}...`);
        await processItem(item);

        const duration = ((Date.now() - startTime) / 1000).toFixed(2);
        debugLog(`Completed in ${duration}s`);
        console.log(`✅ Processed successfully`);

        results.push({ item, success: true });
      } catch (err) {
        console.error(`❌ Failed: ${err.message}`);
        results.push({ item, success: false, error: err.message });
      }

      console.groupEnd();
    }
    console.groupEnd();

    // Summary
    console.group('📊 Summary');
    const successful = results.filter(r => r.success);
    const failed = results.filter(r => !r.success);

    if (successful.length > 0) {
      console.log(`✅ Successfully processed ${successful.length} item(s):`);
      successful.forEach(({ item }) => {
        console.log(`  - ${item.name}@${item.version}`);
      });
    }

    if (failed.length > 0) {
      console.error(`❌ Failed to process ${failed.length} item(s):`);
      failed.forEach(({ item, error }) => {
        console.error(`  - ${item.name}: ${error}`);
      });
      console.groupEnd();
      console.groupEnd();
      throw new Error(`Process failed for ${failed.length} item(s)`);
    }

    console.log('🎉 All items processed successfully!');
    console.groupEnd();
    console.groupEnd();
  } catch (err) {
    handleError(err, 'Process failed');
  }
}
```

---

## Generating Log Output Examples

After implementing or improving logging in a script, **always generate example log outputs** to demonstrate how the logs will appear to users in different scenarios.

### Required Step: Generate Examples

At the end of the command execution, **you MUST automatically generate and display** example log outputs based on the analyzed script. This helps users understand how the logs will appear.

### Step-by-Step Process

1. **Read and analyze the script file**:
   - Parse the JavaScript/TypeScript file to find all logging statements
   - Identify all `console.group()` calls and their hierarchy/nesting
   - Extract all `console.log()`, `console.warn()`, `console.error()` statements
   - Identify `debugLog()` calls (these will only appear in debug mode)
   - Map conditional branches (if/else, early returns, try/catch blocks)
   - Note loop structures that create multiple log instances

2. **Identify script phases**:
   - Initialization/Setup
   - Discovery/Collection
   - Processing/Execution
   - Summary/Results

3. **Generate examples for different scenarios**:
   - **Success scenario (normal mode)**: Normal execution with all steps completing successfully, no debug logs
   - **Success scenario (debug mode)**: Same as above but including all `debugLog()` output
   - **Early exit scenario**: When conditions cause early return (e.g., no items found, validation fails)
   - **Error scenario**: When an error occurs during execution, showing proper group cleanup
   - **Partial success scenario**: When some operations succeed and others fail (if applicable)

4. **Format examples correctly**:
   - Use markdown code blocks with clear labels (e.g., "### Example 1: Successful Execution (Normal Mode)")
   - Show actual indentation structure (2 spaces per nesting level)
   - Filter logs appropriately:
     - Normal mode: Exclude all `debugLog()` output
     - Debug mode: Include all `debugLog()` output prefixed with `🔍 [DEBUG]`
   - Use realistic example values (version numbers, file sizes, counts, URLs)
   - Show only what would actually be displayed (respect conditional logic)
   - Indicate where groups close (don't repeat group headers)

### Example Output Format

After analyzing and improving a script, provide output like this:

---

## 📊 Log Output Examples

### Example 1: Successful Execution (Normal Mode)

```
🚀 Script Process
  ⚙️  Initialization
  📋 Discovery
    Found 3 item(s):
      - item1@1.0.0
      - item2@2.0.0
      - item3@3.0.0
  📦 Processing (3 item(s))
    📱 item1@1.0.0
      ✅ Processed successfully
    📱 item2@2.0.0
      ✅ Processed successfully
    📱 item3@3.0.0
      ✅ Processed successfully
  📊 Summary
    ✅ Successfully processed 3 item(s)
    🎉 All operations completed successfully!
```

### Example 2: Early Exit Scenario

```
🚀 Script Process
  ⚙️  Initialization
  📋 Discovery
    ℹ️  No items found
```

### Example 3: Error During Processing

```
🚀 Script Process
  ⚙️  Initialization
  📋 Discovery
    Found 2 item(s):
      - item1@1.0.0
      - item2@2.0.0
  📦 Processing (2 item(s))
    📱 item1@1.0.0
      ✅ Processed successfully
    📱 item2@2.0.0
      ❌ Failed: Connection timeout
  📊 Summary
    ✅ Successfully processed 1 item(s):
      - item1@1.0.0
    ❌ Failed to process 1 item(s):
      - item2@2.0.0: Connection timeout
Error: Process completed with 1 failure(s) out of 2 item(s)
```

### Example 4: Debug Mode Enabled (DEBUG=true)

```
🚀 Script Process
  ⚙️  Initialization
    🔍 [DEBUG] Environment check:
    🔍 [DEBUG]   VAR1: SET
    🔍 [DEBUG]   VAR2: NOT SET
  📋 Discovery
    🔍 [DEBUG] Scanning directory: /path/to/items
    🔍 [DEBUG] Found 3 total items before filtering
    Found 3 item(s):
      - item1@1.0.0
      - item2@2.0.0
      - item3@3.0.0
  📦 Processing (3 item(s))
    📱 item1@1.0.0
      🔍 [DEBUG] Processing item1... (path: /path/to/item1)
      🔍 [DEBUG] Operation completed in 1.23s
      ✅ Processed successfully
    📱 item2@2.0.0
      🔍 [DEBUG] Processing item2... (path: /path/to/item2)
      🔍 [DEBUG] Operation completed in 0.87s
      ✅ Processed successfully
    📱 item3@3.0.0
      🔍 [DEBUG] Processing item3... (path: /path/to/item3)
      🔍 [DEBUG] Operation completed in 1.45s
      ✅ Processed successfully
  📊 Summary
    ✅ Successfully processed 3 item(s)
    🎉 All operations completed successfully!
```

### Rules for Generating Examples

1. **Hierarchical structure**: Show actual indentation created by `console.group()` (2 spaces per level)
2. **Filter by mode**:
   - Normal mode: Exclude all `debugLog()` output
   - Debug mode: Include all `debugLog()` output with `🔍 [DEBUG]` prefix
3. **Show conditional paths**: Include examples for different execution paths (if/else branches)
4. **Error scenarios**: Show how groups are closed before errors (clean structure)
5. **Progress indicators**: Show actual progress counters/percentages as they would appear
6. **Realistic data**: Use realistic example values (e.g., `item1@1.2.3`, `45.67 MB`, `https://github.com/owner/repo/releases/tag/v1.2.3`)
7. **No duplication**: Each group header appears once at the start, content is nested underneath
8. **Show loop results**: If processing multiple items in a loop, show 2-3 examples, then indicate if more exist
9. **Colorization**: If colorization is implemented, show examples both with and without colors (if terminal doesn't support)

### Automated Generation Algorithm

When executing this command:

1. **Read the target script file**:
   - Parse JavaScript/TypeScript code
   - Extract function definitions, especially `main()` or entry point

2. **Analyze logging structure**:
   - Find all `console.group('...')` calls → track nesting levels
   - Find all `console.groupEnd()` calls → ensure proper closing
   - Extract `console.log()`, `console.warn()`, `console.error()` statements
   - Identify `debugLog()` calls (search for function name or pattern)
   - Map conditional branches (if/else, try/catch, early returns)

3. **Build execution paths**:
   - Trace through code to identify possible execution flows
   - Identify variables that affect log output (arrays, counts, conditions)
   - Map error points and how groups are closed

4. **Generate scenarios**:
   - **Scenario 1**: Full success path (normal mode) - no debug logs
   - **Scenario 2**: Full success path (debug mode) - with debug logs
   - **Scenario 3**: Early exit (validation failure, empty list, etc.)
   - **Scenario 4**: Error during processing (with proper cleanup)
   - **Scenario 5**: Partial success (if applicable to the script)

5. **Format and display**:
   - Create markdown-formatted examples
   - Use proper indentation (2 spaces per nesting level)
   - Include realistic example values
   - Display examples at the end of command output

### Example Analysis Process

Given a script like:

```javascript
async function main() {
  console.group('🚀 Process');
  console.group('⚙️  Initialization');
  debugLog('Checking env vars');
  if (!process.env.VAR) {
    console.groupEnd();
    console.groupEnd();
    throw new Error('Missing VAR');
  }
  console.groupEnd();

  console.group('📋 Discovery');
  const items = findItems();
  console.log(`Found ${items.length} items`);
  items.forEach(item => console.log(`  - ${item.name}`));
  console.groupEnd();

  console.group(`📦 Processing (${items.length})`);
  for (const item of items) {
    console.group(`📱 ${item.name}`);
    await process(item);
    console.log('✅ Done');
    console.groupEnd();
  }
  console.groupEnd();
  console.groupEnd();
}
```

**Generated examples should show:**

1. Normal execution with 2 items (normal mode)
2. Normal execution with 2 items (debug mode - includes debugLog output)
3. Early exit when VAR is missing
4. Error during processing (if applicable)

**Display format:**

```
---

## 📊 Log Output Examples

[Generated examples here]

---
```

---

**When improving existing scripts:**

1. Identify current logging issues (missing groups, too verbose, etc.)
2. Apply hierarchical grouping to major phases
3. Move technical details to `debugLog()`
4. Add contextual information (sizes, counts, durations)
5. Ensure proper cleanup in error scenarios
6. Add summary sections for better overview
7. Use consistent emoji patterns throughout
8. **Generate example log outputs** showing how logs appear in different scenarios
