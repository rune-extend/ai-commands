# Git Commit Message Generator Prompt

Analyze currently staged files and generate a commit message following conventional commits conventions, automatically create changesets for modified packages, and update README.md files when changes require documentation updates.

## Instructions

1. Use `git diff --cached --name-only` to list staged files
2. Use `git diff --cached` to see the differences
3. **Detect modified packages**: Check if any files in the following directories are staged:
   - `packages/*/` → packages
   - `apps/*/` → applications (rx, portal, bots)
   - `apps/portal/*/` → portal sub-applications
   - `apps/bots/*/` → bot applications
   - `www/*/` → web applications
   - `documentation/` → documentation package
4. **Create changeset if needed**: If any of these spaces are modified, automatically create changesets:
   - For each modified space, read the `package.json` to get the exact package name
   - Determine the changeset type based on commit type:
     - `feat` → `minor` (new feature)
     - `fix` → `patch` (bug fix)
     - `refactor`, `perf`, `style` → `patch` (internal changes)
     - `docs`, `test`, `chore`, `ci`, `build` → `patch` (non-user-facing)
     - If commit message contains `BREAKING CHANGE` or `!` → `major`
   - Create changeset file using the package name from `package.json`
   - Use the commit message body as the changeset description
5. **Update README.md if needed**: For each modified space, check if a `README.md` file exists and determine if changes require documentation updates:
   - **New features (`feat`)**: Add to Features section, update Usage/API sections if public interface changed
   - **Bug fixes (`fix`)**: Update Troubleshooting section if relevant, update Features if behavior changed
   - **Breaking changes (`BREAKING CHANGE` or `!`)**: Add Breaking Changes section, update Configuration/Usage as needed, add migration guide if applicable
   - **API/Interface changes**: Update Usage, API, or GraphQL sections
   - **Configuration changes**: Update Configuration or Environment Variables sections
   - **Dependency changes**: Update Prerequisites or Installation sections
   - **New scripts**: Update Available Scripts section
   - **Internal changes (`refactor`, `style`)**: Only update if public interface changed
   - Skip README updates for: `test`, `chore` (unless dependencies change), `ci`, `build` (unless build process changed significantly), `docs` (unless README itself is being modified)
6. Analyze the changes and categorize them (feat, fix, refactor, style, docs, test, chore, etc.)
7. Generate a commit message in English with:
   - A short subject line (50-72 characters) in the format: `type(scope): description`
   - A body detailing the main changes (bullet list if necessary)
   - **NO** exhaustive list of modified files in the message

## Changeset Creation Logic

When any space with a `package.json` is modified:

1. **Detect modified spaces** by checking staged file paths:
   - `packages/package-name/` → read `packages/package-name/package.json` for package name
   - `apps/app-name/` → read `apps/app-name/package.json` for package name
   - `apps/portal/client/` → read `apps/portal/client/package.json` for package name
   - `apps/portal/api/` → read `apps/portal/api/package.json` for package name
   - `apps/bots/support/` → read `apps/bots/support/package.json` for package name
   - `apps/bots/status/` → read `apps/bots/status/package.json` for package name
   - `documentation/` → read `documentation/package.json` for package name
2. **Extract package name** from the `package.json` file's `"name"` field
3. **Map commit type to changeset version type**:
   - `feat` → `minor`
   - `fix` → `patch`
   - `refactor`, `perf`, `style` → `patch`
   - `docs`, `test`, `chore`, `ci`, `build` → `patch`
   - `BREAKING CHANGE` or `!` in message → `major`
4. **Create changeset file** in `.changeset/` directory with format:

   ```markdown
   ---
   'package-name-from-package.json': patch|minor|major
   ---

   Description from commit message body
   ```

5. **Stage the changeset file** along with other changes

## README Update Logic

When any space with a `package.json` is modified and a `README.md` exists:

1. **Detect README.md existence** for each modified space:
   - Check if `README.md` exists in the same directory as `package.json`
   - If it exists, read it to understand its structure
2. **Analyze changes to determine if README update is needed**:
   - Read `git diff --cached` to understand what changed
   - Categorize changes: new features, bug fixes, breaking changes, API changes, config changes, etc.
   - Determine which sections of README need updates
3. **Update appropriate sections** based on commit type and changes:
   - **Features section**: Add new features with brief description
   - **Usage/API section**: Document new public APIs, endpoints, or methods
   - **Configuration section**: Add new environment variables or configuration options
   - **Troubleshooting section**: Add solutions for fixed bugs or common issues
   - **Breaking Changes section**: Create if breaking changes exist, document migration path
   - **Available Scripts section**: Add new npm/pnpm scripts
   - **Prerequisites/Installation**: Update if dependencies or installation steps change
4. **Update format**:
   - Maintain existing README structure and style
   - Use consistent markdown formatting (emojis, lists, code blocks)
   - Keep descriptions concise but informative
   - Update "Last updated" date if present at the bottom
5. **Stage the updated README.md** along with other changes

## Expected Format

### Commit Message

```
type(scope): short description

- Main change 1
- Main change 2
- Main change 3
```

### Changeset File (auto-generated)

```markdown
---
'@rune-extend/package-name': patch|minor|major
---

- Main change 1
- Main change 2
- Main change 3
```

## Commit Types (Conventional Commits)

- `feat`: New feature → Changeset: `minor`
- `fix`: Bug fix → Changeset: `patch`
- `refactor`: Refactoring without behavior change → Changeset: `patch`
- `style`: Formatting changes (indentation, etc.) → Changeset: `patch`
- `docs`: Documentation only → Changeset: `patch`
- `test`: Adding/modifying tests → Changeset: `patch`
- `chore`: Maintenance tasks, dependencies, etc. → Changeset: `patch`
- `perf`: Performance improvement → Changeset: `patch`
- `ci`: CI/CD changes → Changeset: `patch`
- `build`: Build system changes → Changeset: `patch`
- `revert`: Reverting a previous commit → Changeset: `patch`
- `BREAKING CHANGE` or `!`: Breaking changes → Changeset: `major`

## Examples

### Example 1: Package modification

**Staged files:**

- `packages/dto/src/utils.ts`
- `packages/dto/src/validators.ts`

**Package detected:** `packages/dto/package.json` → `"name": "@rune-extend/dto"`

**Generated changeset:** `.changeset/refactor-dto-utils.md`

```markdown
---
'@rune-extend/dto': patch
---

- Refactor utility functions for better type safety
- Improve validator error messages
```

**README.md update:** `packages/dto/README.md` (if it exists)

- No update needed (internal refactoring, no public API changes)

**Commit message:**

```
refactor(dto): modernize utility functions and validators

- Refactor utility functions for better type safety
- Improve validator error messages
- Add JSDoc comments for better IDE support
```

### Example 2: Application modification (RX)

**Staged files:**

- `apps/rx/node/frontend/components/Menu.tsx`

**Package detected:** `apps/rx/package.json` → `"name": "@rune-extend/rx"`

**Generated changeset:** `.changeset/feat-rx-menu.md`

```markdown
---
'@rune-extend/rx': minor
---

- Add new menu component with improved navigation
- Implement keyboard shortcuts support
```

**Generated changeset:** `.changeset/feat-rx-menu.md`

```markdown
---
'@rune-extend/rx': minor
---

- Add new menu component with improved navigation
- Implement keyboard shortcuts support
```

**README.md update:** `apps/rx/README.md` (if it exists)

- **Features section**: Add "🎹 **Keyboard Shortcuts** : Support for keyboard navigation in menu component"
- **Usage section**: Document keyboard shortcut commands if applicable

**Commit message:**

```
feat(rx): add new menu component with keyboard shortcuts

- Add new menu component with improved navigation
- Implement keyboard shortcuts support
- Add accessibility improvements
```

### Example 3: Portal client modification

**Staged files:**

- `apps/portal/client/src/main/window.ts`

**Package detected:** `apps/portal/client/package.json` → `"name": "@rune-extend/portal-client"`

**Generated changeset:** `.changeset/fix-portal-client-window.md`

```markdown
---
'@rune-extend/portal-client': patch
---

- Fix window resize handling on macOS
- Improve window state persistence
```

**Generated changeset:** `.changeset/fix-portal-client-window.md`

```markdown
---
'@rune-extend/portal-client': patch
---

- Fix window resize handling on macOS
- Improve window state persistence
```

**README.md update:** `apps/portal/client/README.md` (if it exists)

- **Troubleshooting section**: Add "Window resize issues on macOS: Fixed in vX.X.X" or remove if issue was documented
- **Features section**: Update if window state persistence is a new feature

**Commit message:**

```
fix(portal-client): resolve window resize issues on macOS

- Fix window resize handling on macOS
- Improve window state persistence
- Add proper error handling for window operations
```

### Example 4: Multiple packages modified

**Staged files:**

- `packages/dto/src/api.ts`
- `apps/rx/node/backend/api.ts`

**Packages detected:**

- `packages/dto/package.json` → `"name": "@rune-extend/dto"`
- `apps/rx/package.json` → `"name": "@rune-extend/rx"`

**Generated changesets:**

- `.changeset/feat-dto-api.md`
- `.changeset/feat-rx-api.md`

**Commit message:**

```
feat(api): add new GraphQL mutations for user management

- Add createUser and updateUser mutations in dto
- Implement user management endpoints in rx backend
- Add proper validation and error handling
```

### Example 5: Documentation modification

**Staged files:**

- `documentation/docs/api/getting-started.md`

**Package detected:** `documentation/package.json` → `"name": "@rune-extend/documentation"`

**Generated changeset:** `.changeset/docs-documentation-getting-started.md`

```markdown
---
'@rune-extend/documentation': patch
---

- Update getting started guide with new setup instructions
- Add troubleshooting section
```

**README.md update:** `documentation/README.md` (if it exists)

- No update needed (documentation changes don't require README update unless structure changed)

**Commit message:**

```
docs(documentation): update getting started guide

- Update getting started guide with new setup instructions
- Add troubleshooting section
- Fix broken links in API documentation
```

### Example 6: Breaking change with README update

**Staged files:**

- `apps/portal/api/src/modules/auth/resolvers/login.resolver.ts`
- `apps/portal/api/src/modules/auth/services/auth.service.ts`

**Package detected:** `apps/portal/api/package.json` → `"name": "@rune-extend/portal-api"`

**Generated changeset:** `.changeset/breaking-auth-api.md`

```markdown
---
'@rune-extend/portal-api': major
---

- BREAKING CHANGE: Authentication API now requires email instead of username
- Migration: Update all login requests to use email field
```

**README.md update:** `apps/portal/api/README.md` (if it exists)

- **Breaking Changes section**: Create new section if it doesn't exist:

  ```markdown
  ## ⚠️ Breaking Changes

  ### vX.X.X (YYYY-MM-DD)

  - **Authentication API**: The `login` mutation now requires `email` instead of `username`
    - Old: `mutation { login(username: "user123", password: "...") }`
    - New: `mutation { login(email: "user@example.com", password: "...") }`
  ```

- **GraphQL API section**: Update example queries to use `email` instead of `username`
- **Migration guide**: Add migration steps if applicable

**Commit message:**

```
feat(portal-api)!: change authentication to use email instead of username

BREAKING CHANGE: The login mutation now requires an email field instead of username

- Update login resolver to accept email parameter
- Modify AuthService to query users by email
- Update authentication validation logic
- Migrate existing username-based authentication to email-based

Migration guide:
- Update all GraphQL login mutations to use `email` field
- Update frontend forms to request email instead of username
- Update any stored usernames to their corresponding email addresses
```

### Example 7: Configuration change with README update

**Staged files:**

- `apps/bots/status/src/config/index.ts`
- `.env.example` (if exists)

**Package detected:** `apps/bots/status/package.json` → `"name": "@rune-extend/bot-status"`

**Generated changeset:** `.changeset/feat-bot-status-config.md`

```markdown
---
'@rune-extend/bot-status': minor
---

- Add support for custom check interval configuration
- Add RETRY_COUNT environment variable for service checks
```

**README.md update:** `apps/bots/status/README.md` (if it exists)

- **Environment Variables section**: Add new optional variable:
  ```markdown
  | `CHECK_INTERVAL` | Service check interval in minutes | `5` (default) |
  | `RETRY_COUNT` | Number of retries for failed checks | `3` (default) |
  ```
- **Configuration section**: Update service configuration explanation to mention custom interval

**Commit message:**

```
feat(bot-status): add configurable check interval and retry count

- Add CHECK_INTERVAL environment variable (default: 5 minutes)
- Add RETRY_COUNT environment variable (default: 3 retries)
- Update service check logic to respect custom interval
- Implement retry mechanism for failed service checks
```

## Important Notes

- **Create changesets for ALL spaces with `package.json` files**, including:
  - `packages/*/` (all packages)
  - `apps/*/` (all applications: rx, portal, bots)
  - `apps/portal/*/` (portal sub-applications)
  - `apps/bots/*/` (bot applications)
  - `documentation/` (documentation package)
- **Read the `package.json` file to get the exact package name** (don't assume the name from the directory)
- **If multiple spaces are modified, create separate changeset entries for each**
- **If no spaces with `package.json` are modified, skip changeset creation**
- **Always stage the generated changeset file(s) before committing**
- **Package names may vary** (e.g., `@rune-extend/dto`, `@rune-extend/rx`, `bot-support`, `showcase`)
- **Update README.md files when changes affect user-facing behavior**:
  - Always update README for: new features, breaking changes, API changes, configuration changes
  - Update README for bug fixes only if they change documented behavior or add troubleshooting info
  - Skip README updates for: internal refactoring (unless public API changes), test changes, style changes, documentation-only changes (unless README itself)
  - Maintain existing README structure, style, and formatting conventions
  - Read the existing README to understand its structure before making updates
  - If README doesn't exist, do not create it automatically (only update existing ones)
