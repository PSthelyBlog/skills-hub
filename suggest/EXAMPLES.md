# Real-World Examples

Practical examples of using the `/suggest` skill in common development scenarios.

---

## Scenario 1: Pre-Merge Code Review

**Reviewing a feature branch before merging**

```
You: /suggest src/features/user-profile/

Claude:
## Suggestions for src/features/user-profile/

### 1. [HIGH] Validate email format before database insert
**Category:** bugs
**Why:** Invalid emails will cause downstream failures in notification system

**Before:**
```typescript
async function updateProfile(userId: string, data: ProfileData) {
  await db.users.update(userId, { email: data.email });
}
```

**After:**
```typescript
async function updateProfile(userId: string, data: ProfileData) {
  if (data.email && !isValidEmail(data.email)) {
    throw new ValidationError('Invalid email format');
  }
  await db.users.update(userId, { email: data.email });
}
```

---

### 2. [HIGH] Add rate limiting to profile update endpoint
**Category:** security
**Why:** Prevents abuse and enumeration attacks on user data

**Before:**
```typescript
router.put('/profile', authMiddleware, profileController.update);
```

**After:**
```typescript
router.put('/profile', authMiddleware, rateLimit({ windowMs: 60000, max: 10 }), profileController.update);
```

---

### 3. [MED] Memoize expensive avatar URL generation
**Category:** performance
**Why:** Reduces redundant S3 signed URL generation on repeated renders

**Before:**
```typescript
function getAvatarUrl(user: User) {
  return s3.getSignedUrl('getObject', { Bucket: BUCKET, Key: user.avatarKey });
}
```

**After:**
```typescript
const getAvatarUrl = memoize((user: User) => {
  return s3.getSignedUrl('getObject', { Bucket: BUCKET, Key: user.avatarKey, Expires: 3600 });
}, (user) => user.avatarKey);
```

---

**Summary:** 3 suggestions (2 high, 1 medium)
Run `/suggest --apply` to interactively apply these changes.
```

---

## Scenario 2: Security Audit

**Focused security review of authentication code**

```
You: /suggest --category security --depth thorough src/auth/

Claude:
## Suggestions for src/auth/ (security)

### 1. [HIGH] Use constant-time comparison for password verification
**Category:** security
**Why:** Timing attacks can leak password information through response times

**Before:**
```typescript
function verifyPassword(input: string, hash: string): boolean {
  return bcrypt.compareSync(input, hash);
}
```

**After:**
```typescript
import { timingSafeEqual } from 'crypto';

async function verifyPassword(input: string, hash: string): Promise<boolean> {
  const inputHash = await bcrypt.hash(input, hash.slice(0, 29));
  return timingSafeEqual(Buffer.from(inputHash), Buffer.from(hash));
}
```

---

### 2. [HIGH] Invalidate all sessions on password change
**Category:** security
**Why:** Compromised sessions remain valid after password reset

**Before:**
```typescript
async function changePassword(userId: string, newPassword: string) {
  const hash = await bcrypt.hash(newPassword, 10);
  await db.users.update(userId, { passwordHash: hash });
}
```

**After:**
```typescript
async function changePassword(userId: string, newPassword: string) {
  const hash = await bcrypt.hash(newPassword, 10);
  await db.users.update(userId, { passwordHash: hash });
  await sessionStore.invalidateAllForUser(userId);
}
```

---

### 3. [HIGH] Add CSRF token validation to auth endpoints
**Category:** security
**Why:** Auth forms are vulnerable to cross-site request forgery

**Before:**
```typescript
router.post('/login', loginController.handle);
router.post('/logout', authMiddleware, logoutController.handle);
```

**After:**
```typescript
router.post('/login', csrfProtection, loginController.handle);
router.post('/logout', authMiddleware, csrfProtection, logoutController.handle);
```

---

### 4. [MED] Implement account lockout after failed attempts
**Category:** security
**Why:** Prevents brute force password guessing attacks

**Before:**
```typescript
async function login(email: string, password: string) {
  const user = await db.users.findByEmail(email);
  if (!user || !await verifyPassword(password, user.passwordHash)) {
    throw new AuthError('Invalid credentials');
  }
  return createSession(user);
}
```

**After:**
```typescript
async function login(email: string, password: string) {
  const user = await db.users.findByEmail(email);

  if (user && await isAccountLocked(user.id)) {
    throw new AuthError('Account temporarily locked. Try again later.');
  }

  if (!user || !await verifyPassword(password, user.passwordHash)) {
    if (user) await recordFailedAttempt(user.id);
    throw new AuthError('Invalid credentials');
  }

  await clearFailedAttempts(user.id);
  return createSession(user);
}
```

---

### 5. [MED] Log security-relevant events for audit trail
**Category:** security
**Why:** Enables detection and investigation of security incidents

**Before:**
```typescript
async function login(email: string, password: string) {
  // ... validation logic ...
  return createSession(user);
}
```

**After:**
```typescript
async function login(email: string, password: string) {
  // ... validation logic ...

  securityLog.info('login_success', {
    userId: user.id,
    ip: request.ip,
    userAgent: request.headers['user-agent'],
    timestamp: new Date().toISOString()
  });

  return createSession(user);
}
```

---

**Summary:** 5 suggestions (3 high, 2 medium)
Run `/suggest --apply` to interactively apply these changes.
```

---

## Scenario 3: Performance Optimization

**Finding bottlenecks in a slow API endpoint**

```
You: /explain why is the dashboard API slow
Claude: [explains N+1 queries, missing indexes, client-side aggregation]

You: /suggest --category performance

Claude:
## Suggestions for src/api/dashboard.ts (performance)

### 1. [HIGH] Replace N+1 queries with JOIN
**Category:** performance
**Why:** Reduces 50+ database round trips to a single query

**Before:**
```typescript
async function getDashboard(userId: string) {
  const orders = await db.orders.findByUser(userId);

  // N+1: fetches each product individually
  for (const order of orders) {
    order.products = await db.products.findByOrder(order.id);
  }

  return orders;
}
```

**After:**
```typescript
async function getDashboard(userId: string) {
  const orders = await db.query(`
    SELECT o.*, json_agg(p.*) as products
    FROM orders o
    LEFT JOIN order_products op ON o.id = op.order_id
    LEFT JOIN products p ON op.product_id = p.id
    WHERE o.user_id = $1
    GROUP BY o.id
  `, [userId]);

  return orders;
}
```

---

### 2. [HIGH] Add composite index for date range queries
**Category:** performance
**Why:** Current full table scan takes 1.8s, index reduces to ~50ms

**Before:**
```sql
-- No index on commonly filtered columns
SELECT * FROM orders WHERE user_id = $1 AND created_at > $2;
```

**After:**
```sql
-- Add in migration
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);
```

---

### 3. [MED] Move aggregation from JavaScript to SQL
**Category:** performance
**Why:** Database aggregation is 10x faster than client-side processing

**Before:**
```typescript
const orders = await db.orders.findAll();
const total = orders.reduce((sum, o) => sum + o.amount, 0);
const average = total / orders.length;
```

**After:**
```typescript
const stats = await db.query(`
  SELECT
    COUNT(*) as count,
    SUM(amount) as total,
    AVG(amount) as average
  FROM orders
  WHERE user_id = $1
`, [userId]);
```

---

### 4. [MED] Add Redis caching for dashboard data
**Category:** performance
**Why:** Dashboard data changes infrequently, caching saves ~2s per request

**Before:**
```typescript
async function getDashboard(userId: string) {
  return await computeDashboardData(userId);
}
```

**After:**
```typescript
async function getDashboard(userId: string) {
  const cacheKey = `dashboard:${userId}`;
  const cached = await redis.get(cacheKey);

  if (cached) return JSON.parse(cached);

  const data = await computeDashboardData(userId);
  await redis.setex(cacheKey, 300, JSON.stringify(data)); // 5 min TTL

  return data;
}
```

---

**Summary:** 4 suggestions (2 high, 2 medium)
Expected improvement: ~3s → ~200ms after all optimizations.
```

---

## Scenario 4: Quick Wins Before Deploy

**Fast scan for obvious issues**

```
You: /suggest --depth quick --limit 3 src/

Claude:
## Suggestions for src/ (quick scan)

### 1. [HIGH] Remove console.log statements
**Category:** readability
**Why:** Debug logs should not ship to production

**Before:**
```typescript
console.log('DEBUG: user data', userData);
```

**After:**
```typescript
// Remove or replace with proper logging
logger.debug('User data loaded', { userId: userData.id });
```

---

### 2. [HIGH] Handle rejected promise in async handler
**Category:** bugs
**Why:** Unhandled rejection will crash the server

**Before:**
```typescript
app.get('/users', async (req, res) => {
  const users = await db.users.findAll();
  res.json(users);
});
```

**After:**
```typescript
app.get('/users', async (req, res, next) => {
  try {
    const users = await db.users.findAll();
    res.json(users);
  } catch (error) {
    next(error);
  }
});
```

---

### 3. [MED] Use environment variable for API URL
**Category:** bugs
**Why:** Hardcoded URL will fail in staging/production

**Before:**
```typescript
const API_URL = 'http://localhost:3000/api';
```

**After:**
```typescript
const API_URL = process.env.API_URL || 'http://localhost:3000/api';
```

---

**Summary:** 3 suggestions (2 high, 1 medium)
Quick scan complete. Run with `--depth standard` for more suggestions.
```

---

## Scenario 5: Interactive Apply Mode

**Stepping through suggestions one by one**

```
You: /suggest --apply src/utils/validation.ts

Claude:
Analyzing src/utils/validation.ts...

Found 4 suggestions. Let's go through them:

---

Applying suggestion 1/4: Add type guard for null values

**Before:**
```typescript
function validateEmail(email: string): boolean {
  return email.includes('@');
}
```

**After:**
```typescript
function validateEmail(email: string | null | undefined): boolean {
  if (!email) return false;
  return email.includes('@') && email.includes('.');
}
```

Apply this change?

You: y

Claude: Applied.

---

Applying suggestion 2/4: Use regex for stricter email validation

**Before:**
```typescript
function validateEmail(email: string | null | undefined): boolean {
  if (!email) return false;
  return email.includes('@') && email.includes('.');
}
```

**After:**
```typescript
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

function validateEmail(email: string | null | undefined): boolean {
  if (!email) return false;
  return EMAIL_REGEX.test(email);
}
```

Apply this change?

You: y

Claude: Applied.

---

Applying suggestion 3/4: Extract validation messages to constants

**Before:**
```typescript
throw new Error('Invalid email format');
```

**After:**
```typescript
const VALIDATION_MESSAGES = {
  INVALID_EMAIL: 'Invalid email format',
  REQUIRED_FIELD: 'This field is required',
} as const;

throw new Error(VALIDATION_MESSAGES.INVALID_EMAIL);
```

Apply this change?

You: n

Claude: Skipped.

---

Applying suggestion 4/4: Add JSDoc documentation

**Before:**
```typescript
function validateEmail(email: string | null | undefined): boolean {
```

**After:**
```typescript
/**
 * Validates an email address format
 * @param email - The email address to validate
 * @returns true if valid, false otherwise
 */
function validateEmail(email: string | null | undefined): boolean {
```

Apply this change?

You: skip all

Claude: Skipping remaining suggestions.

---

## Summary

Applied 2/4 suggestions:
- Add type guard for null values ✓
- Use regex for stricter email validation ✓
- Extract validation messages to constants (skipped)
- Add JSDoc documentation (skipped)

File saved. Run tests to verify changes.
```

---

## Scenario 6: Post-Explain Improvement

**Understanding code, then improving it**

```
You: /explain src/services/payment.ts --deep

Claude:
## Payment Service

**Overview**: Handles Stripe integration for one-time and subscription payments...

[comprehensive explanation of payment flow, error handling, etc.]

**Gotchas**:
- No retry logic for transient failures
- Idempotency keys not implemented
- Error messages expose internal details

You: /suggest

Claude:
## Suggestions for src/services/payment.ts

Based on the explanation above, here are targeted improvements:

### 1. [HIGH] Add retry logic for transient Stripe failures
**Category:** bugs
**Why:** Network blips cause payment failures that could succeed on retry

**Before:**
```typescript
async function chargeCard(amount: number, token: string) {
  return await stripe.charges.create({ amount, source: token });
}
```

**After:**
```typescript
async function chargeCard(amount: number, token: string, retries = 2) {
  try {
    return await stripe.charges.create({ amount, source: token });
  } catch (error) {
    if (retries > 0 && isTransientError(error)) {
      await delay(1000);
      return chargeCard(amount, token, retries - 1);
    }
    throw error;
  }
}
```

---

### 2. [HIGH] Implement idempotency keys to prevent duplicate charges
**Category:** bugs
**Why:** Retries without idempotency can charge customers multiple times

**Before:**
```typescript
await stripe.charges.create({ amount, source: token });
```

**After:**
```typescript
await stripe.charges.create(
  { amount, source: token },
  { idempotencyKey: `charge_${orderId}_${Date.now()}` }
);
```

---

### 3. [MED] Sanitize error messages before returning to client
**Category:** security
**Why:** Internal error details can leak sensitive information

**Before:**
```typescript
catch (error) {
  throw new Error(`Payment failed: ${error.message}`);
}
```

**After:**
```typescript
catch (error) {
  logger.error('Payment failed', { error, orderId });
  throw new PaymentError('Payment could not be processed. Please try again.');
}
```

---

**Summary:** 3 suggestions (2 high, 1 medium)
These address the gotchas identified in the explanation.
```

---

## Scenario 7: Refactoring a Legacy File

**Improving a large, messy file**

```
You: /suggest --depth thorough --limit 10 src/legacy/orderProcessor.js

Claude:
## Suggestions for src/legacy/orderProcessor.js

This file is 800 lines with multiple concerns. Here's a prioritized improvement plan:

### 1. [HIGH] Split into separate modules by responsibility
**Category:** readability
**Why:** Single 800-line file violates single responsibility principle

**Before:**
```javascript
// orderProcessor.js - 800 lines doing validation, processing, notifications, logging
```

**After:**
```
src/orders/
├── validator.js       # Input validation (lines 1-150)
├── processor.js       # Core processing logic (lines 151-400)
├── notifier.js        # Email/SMS notifications (lines 401-550)
├── logger.js          # Audit logging (lines 551-650)
└── index.js           # Public API combining above
```

---

### 2. [HIGH] Replace callback pyramid with async/await
**Category:** readability
**Why:** Callback hell makes control flow impossible to follow

**Before:**
```javascript
function processOrder(order, callback) {
  validateOrder(order, function(err, valid) {
    if (err) return callback(err);
    chargePayment(order, function(err, charge) {
      if (err) return callback(err);
      updateInventory(order, function(err) {
        if (err) return callback(err);
        sendConfirmation(order, callback);
      });
    });
  });
}
```

**After:**
```javascript
async function processOrder(order) {
  await validateOrder(order);
  const charge = await chargePayment(order);
  await updateInventory(order);
  await sendConfirmation(order);
  return charge;
}
```

---

### 3. [HIGH] Add error handling for each processing step
**Category:** bugs
**Why:** Failures in one step leave order in inconsistent state

[... continues with more suggestions ...]

---

**Summary:** 10 suggestions (4 high, 4 medium, 2 low)
Recommended approach: Start with suggestions 1-2 (structural), then apply 3-10 to new modules.
```

---

## Pattern Summary

### Quick Mode Usage
- Pre-commit sanity check
- Fast CI integration
- "Did I forget anything obvious?"

### Standard Mode Usage
- Regular code review
- Post-feature cleanup
- Daily improvement habit

### Thorough Mode Usage
- Legacy code refactoring
- Security audits
- Performance deep-dives
- Pre-release reviews

### Apply Mode Usage
- Implementing suggestions safely
- Learning by doing
- Controlled refactoring

### Chaining with /explain
```
/explain [target] --deep    # Understand
/suggest                     # Improve
/suggest --apply            # Implement
```

---

**The best code review catches issues before they become problems. Use `/suggest` early and often.**
