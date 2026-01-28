# Real-World Examples

Practical examples of using the `/update` skill in common development scenarios.

---

## Scenario 1: React Query v3 to v5 Migration

**The problem**: Your codebase uses React Query v3, but v5 has breaking changes including a package rename and API changes.

```
You: /update src/hooks/useUserData.ts

Claude:
🔍 Scanning imports...
📦 Detected: react-query@3.39.0

   Update this library? [Y/n]

You: Y

📚 Fetching documentation from Context7...
   Library ID: /tanstack/query

## Update Opportunities

### react-query → @tanstack/react-query (v5)
⚠️ BREAKING: Package renamed in TanStack Query v5

📊 Summary:
- 🟢 1 import path change
- 🟡 2 API signature changes
- 🔴 0 behavioral changes

---

### Change 1 of 3: Package rename (🟢 Low)

```diff
- import { useQuery, useMutation } from 'react-query'
+ import { useQuery, useMutation } from '@tanstack/react-query'
```

**Reason**: React Query was renamed to TanStack Query in v5
**Reference**: https://tanstack.com/query/latest/docs/react/guides/migrating-to-v5

[Accept] [Reject] [Accept All] [Show More]

You: Show More

### Change 2 of 3: useQuery signature (🟡 Medium)

```diff
  const { data, isLoading } = useQuery(
-   'users',
-   fetchUsers,
-   { staleTime: 5000 }
+   {
+     queryKey: ['users'],
+     queryFn: fetchUsers,
+     staleTime: 5000
+   }
  )
```

**Reason**: v5 uses object syntax for all options
**Reference**: https://tanstack.com/query/latest/docs/react/guides/migrating-to-v5#usequery

[Accept] [Reject] [Accept All] [Reject All]

You: Accept All

## Update Complete

✅ Applied 3 changes to src/hooks/useUserData.ts
📁 Backup: .update-backups/src_hooks_useUserData.ts.1706140800.bak

### Changes Applied:
1. ✅ Import: react-query → @tanstack/react-query
2. ✅ useQuery: Array key → Object syntax
3. ✅ useMutation: Callback options → Object syntax

Next steps:
- Run `npm install @tanstack/react-query` if not installed
- Update your QueryClientProvider import
- Run tests to verify functionality
```

---

## Scenario 2: Express 4 to Express 5 Migration

**The problem**: Express 5 changed how middleware errors are handled and removed some deprecated methods.

```
You: /update src/api/routes.ts --lib express --verbose

Claude:
📚 Fetching documentation for express...
   Library ID: /expressjs/express
   Current version: 4.18.2 → Latest: 5.x

## Update Opportunities

### Express 5 Migration
⚠️ BREAKING: Error handling and async middleware changes

---

### Change 1 of 4: Async error handling (🟡 Medium)

```diff
- app.get('/users', async (req, res) => {
-   const users = await fetchUsers();
-   res.json(users);
- });
+ app.get('/users', async (req, res, next) => {
+   try {
+     const users = await fetchUsers();
+     res.json(users);
+   } catch (err) {
+     next(err);
+   }
+ });
```

**Reason**: Express 5 requires explicit error forwarding in async routes
**Reference**: https://expressjs.com/en/guide/migrating-5.html

---

### Change 2 of 4: req.param() removed (🔴 High)

```diff
- const id = req.param('id');
+ const id = req.params.id || req.body.id || req.query.id;
```

**Reason**: req.param() was deprecated in Express 4, removed in Express 5
**Reference**: https://expressjs.com/en/guide/migrating-5.html#removed

---

### Change 3 of 4: res.send() behavior (🟡 Medium)

```diff
- res.send(status, body);  // Deprecated positional args
+ res.status(status).send(body);  // Chained method
```

**Reason**: Positional arguments to res.send() removed
**Reference**: https://expressjs.com/en/guide/migrating-5.html#res.send

---

### Change 4 of 4: app.router removed (🟢 Low)

```diff
- app.use(app.router);  // No longer needed
+ // Router is mounted automatically
```

**Reason**: Explicit router mounting removed in Express 5
**Reference**: https://expressjs.com/en/guide/migrating-5.html#app.router

---

Apply changes? [Accept All] [Review Each] [Cancel]

You: Review Each
You: Accept (changes 1, 3, 4)
You: Reject (change 2 - needs manual review)

## Update Complete

✅ Applied 3 changes
❌ Rejected 1 change (needs manual review)

### Rejected Change Details:
Change 2 (req.param): This change affects multiple code paths.
Manual review recommended to determine correct replacement.

📁 Backup: .update-backups/src_api_routes.ts.1706140800.bak
```

---

## Scenario 3: Zod Schema Migration

**The problem**: Zod 3.22 deprecated some patterns and introduced new refinement syntax.

```
You: /update src/schemas/user.ts --lib zod

Claude:
📦 Current: zod@3.21.0
📚 Fetching docs for latest patterns...

## Update Opportunities

### Zod Best Practices Update

📊 Summary:
- 🟢 2 syntax improvements
- 🟡 1 deprecated pattern
- 🔴 0 breaking changes

---

### Change 1 of 3: Improved error messages (🟢 Low)

```diff
  const userSchema = z.object({
-   email: z.string().email(),
+   email: z.string().email({ message: "Invalid email format" }),
    age: z.number().min(0),
  });
```

**Reason**: Explicit error messages improve validation UX
**Reference**: https://zod.dev/?id=strings

---

### Change 2 of 3: Use z.coerce for parsing (🟡 Medium)

```diff
- const ageFromString = z.preprocess(
-   (val) => parseInt(val as string, 10),
-   z.number()
- );
+ const ageFromString = z.coerce.number();
```

**Reason**: z.coerce is the modern approach for type coercion
**Reference**: https://zod.dev/?id=coercion-for-primitives

---

### Change 3 of 3: Refine syntax update (🟢 Low)

```diff
  const passwordSchema = z.string()
-   .refine((val) => val.length >= 8, "Password too short")
+   .min(8, "Password too short")
    .refine((val) => /[A-Z]/.test(val), "Need uppercase letter");
```

**Reason**: Built-in validators preferred over custom refine for common cases
**Reference**: https://zod.dev/?id=strings

---

[Accept All] [Review Each]

You: Accept All

## Update Complete

✅ Applied 3 changes to src/schemas/user.ts
📁 Backup: .update-backups/src_schemas_user.ts.1706140800.bak
```

---

## Scenario 4: Project-Wide Migration with --scope all

**The problem**: You want to update all files using a library across the entire project.

```
You: /update src/ --lib react-router --scope all --dry-run

Claude:
🔍 Scanning project for react-router imports...

Found 12 files using react-router:
- src/App.tsx
- src/routes/index.tsx
- src/routes/PrivateRoute.tsx
- src/components/Navigation.tsx
- src/hooks/useNavigate.ts
... and 7 more

📦 Current: react-router-dom@5.3.0
📚 Fetching docs for react-router v6...

## Update Opportunities (DRY RUN - No changes will be applied)

### react-router-dom v6 Migration
⚠️ BREAKING: Major API changes

### Summary Across All Files

| File | 🟢 Low | 🟡 Medium | 🔴 High | ⚠️ Breaking |
|------|--------|----------|---------|-------------|
| App.tsx | 1 | 2 | 0 | 2 |
| routes/index.tsx | 2 | 5 | 1 | 3 |
| routes/PrivateRoute.tsx | 0 | 3 | 0 | 1 |
| components/Navigation.tsx | 1 | 1 | 0 | 1 |
| hooks/useNavigate.ts | 0 | 2 | 0 | 1 |
| ... | ... | ... | ... | ... |
| **Total** | **8** | **24** | **2** | **15** |

---

### Key Changes Detected:

**1. Switch → Routes (all files)**
```diff
- <Switch>
-   <Route path="/home" component={Home} />
- </Switch>
+ <Routes>
+   <Route path="/home" element={<Home />} />
+ </Routes>
```

**2. useHistory → useNavigate (5 files)**
```diff
- const history = useHistory();
- history.push('/dashboard');
+ const navigate = useNavigate();
+ navigate('/dashboard');
```

**3. Redirect → Navigate (3 files)**
```diff
- <Redirect to="/login" />
+ <Navigate to="/login" replace />
```

---

This is a DRY RUN. No files were modified.

To apply changes:
- Run without --dry-run to apply interactively
- Or use --yes to apply all changes
- Consider --max-risk low for safe changes only

Recommended approach:
1. /update src/App.tsx --lib react-router  (start with root)
2. /update src/routes/ --lib react-router --scope all  (then routes)
3. /update src/components/ --lib react-router --scope all  (then components)
```

---

## Scenario 5: Safe CI Updates with --max-risk low

**The problem**: You want to apply only safe updates in CI without human review.

```
You: /update src/ --lib axios --scope all --max-risk low --yes

Claude:
📦 Detected: axios@0.27.2
📚 Fetching documentation...

## Applying Low-Risk Changes Only

Filtering: --max-risk low (excluding medium and high risk changes)

### Changes Applied:

**src/api/client.ts:**
✅ Import syntax update (axios default export)

**src/utils/http.ts:**
✅ Type import added for AxiosResponse

**src/hooks/useApi.ts:**
✅ Type import added for AxiosError

---

## Summary

- Files scanned: 45
- Files modified: 3
- Changes applied: 3 (all 🟢 low risk)
- Changes skipped: 7 (4 🟡 medium, 3 🔴 high - filtered by --max-risk)

Backups created:
- .update-backups/src_api_client.ts.1706140800.bak
- .update-backups/src_utils_http.ts.1706140801.bak
- .update-backups/src_hooks_useApi.ts.1706140802.bak

### Skipped Changes (require manual review):

**Medium Risk (4):**
- Error handling pattern change in src/api/client.ts
- Response interceptor update in src/api/client.ts
- Request config typing in src/utils/http.ts
- Cancel token → AbortController in src/hooks/useApi.ts

**High Risk (3):**
- Timeout behavior change
- Retry logic update
- Default headers handling

To apply skipped changes:
/update src/ --lib axios --scope all --max-risk high
```

---

## Scenario 6: Handling Library Not Found

**The problem**: The library isn't indexed in Context7.

```
You: /update src/utils/date.ts

Claude:
🔍 Scanning imports...
📦 Detected: luxon@3.2.0

   Update this library? [Y/n]

You: Y

📚 Fetching documentation...

⚠️ Library Not Found

Could not find "luxon" in Context7 documentation index.

**Suggestions:**
1. Try the exact npm package name: `--lib luxon`
2. Check for alternative names: `moment-luxon`, `@types/luxon`
3. The library may not be indexed yet

**Options:**
[Retry with different name] [Skip this library] [Cancel]

You: Skip this library

No updates applied. The file remains unchanged.
```

---

## Scenario 7: Reviewing Breaking Changes Carefully

**The problem**: High-risk changes need careful manual review.

```
You: /update src/auth/oauth.ts --lib passport --verbose

Claude:
📦 Current: passport@0.6.0
📚 Fetching documentation...

## Update Opportunities

### Passport.js Session Handling Changes
⚠️ BREAKING: Session serialization API changed

---

### Change 1 of 2: serializeUser signature (🔴 High)

```diff
- passport.serializeUser((user, done) => {
-   done(null, user.id);
- });
+ passport.serializeUser((user, done) => {
+   process.nextTick(() => {
+     done(null, { id: user.id, email: user.email });
+   });
+ });
```

**Reason**: Passport 0.6+ recommends async serialization and storing more user data
**Breaking**: Changes what data is stored in session
**Impact**: All existing sessions may need to be invalidated
**Reference**: https://www.passportjs.org/concepts/authentication/sessions/

⚠️ **Manual Review Recommended**: This change affects session storage.
   Existing users may need to re-authenticate.

---

### Change 2 of 2: Strategy callback (🟡 Medium)

```diff
  passport.use(new GoogleStrategy({
    clientID: CLIENT_ID,
    clientSecret: CLIENT_SECRET,
-   callbackURL: "/auth/google/callback"
+   callbackURL: "/auth/google/callback",
+   scope: ['profile', 'email']  // Now required to be explicit
  }, verifyCallback));
```

**Reason**: Scopes should be explicit rather than relying on defaults
**Reference**: https://www.passportjs.org/packages/passport-google-oauth20/

---

[Accept All] [Review Each] [Reject All] [Show Impact Analysis]

You: Show Impact Analysis

## Impact Analysis

### Change 1 (Session Serialization):

**Files that may be affected:**
- src/auth/oauth.ts (direct)
- src/middleware/auth.ts (uses req.user)
- src/routes/api.ts (uses req.isAuthenticated())
- tests/auth.test.ts (mocks session)

**Database/Storage impact:**
- Session store will contain different data structure
- Existing sessions become invalid

**Recommended migration steps:**
1. Deploy with backwards-compatible serialization first
2. Allow grace period for session refresh
3. Then deploy strict new format
4. Clear old sessions from store

**Risk if applied directly:**
All logged-in users will be logged out on next request.

---

[Accept with Warning] [Reject and Note for Manual] [Cancel]

You: Reject and Note for Manual

## Update Complete

✅ Applied 1 change (scope addition)
❌ Rejected 1 change (session serialization - flagged for manual migration)

📁 Backup: .update-backups/src_auth_oauth.ts.1706140800.bak

### Flagged for Manual Review:

Session serialization change rejected.
See .update-audit/2026-01-24.jsonl for details and migration guide.

Recommended manual approach:
1. Plan session migration strategy
2. Test with subset of users
3. Implement gradually with rollback plan
```

---

## Pattern Summary

### When to Use Different Flags

| Scenario | Recommended Command |
|----------|-------------------|
| Exploring changes | `/update [file] --dry-run --verbose` |
| Single file update | `/update [file]` |
| Project migration | `/update src/ --lib [lib] --scope all` |
| CI automation | `/update src/ --yes --max-risk low` |
| Careful migration | `/update [file] --verbose` (review each) |

### Progressive Update Strategy

For large migrations:

```bash
# Step 1: Preview all changes
/update src/ --lib [library] --scope all --dry-run

# Step 2: Apply safe changes
/update src/ --lib [library] --scope all --max-risk low --yes

# Step 3: Review medium risk
/update src/ --lib [library] --scope all --max-risk medium

# Step 4: Carefully handle high risk
/update [specific-file] --lib [library] --verbose
```

### After Any Update

1. Run your test suite
2. Check for runtime errors
3. Review rejected changes manually
4. Delete backups after confirming
5. Commit with clear migration message

---

**The best update is one that leaves your code working and you understanding what changed.**

Use `--dry-run` to explore. Use `--verbose` to understand. Use `--max-risk` to control.
