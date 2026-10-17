# 1. unstable cache

```typescript
import { unstable_cache } from "next/cache";
import prisma from "../prisma";

const getStats = unstable_cache(
  async () => {
    const totalUsers = await prisma.user.count();
    const totalOrders = await prisma.order.count();
    return { totalUsers, totalOrders };
  },
  ["dashboard-stats"], // مفتاح الكاش
  { revalidate: 60 }, // كل 60 ثانية يجدد البيانات
);

export async function GET() {
  const stats = await getStats();
  return Response.json(stats);
}
```

```

```

# 2. Fetches everything

**Query only what you need**

❌ **Fetches everything**

```typescript
const users = await prisma.user.findMany();
```

✅ **Fetch only what you need**

```typescript
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true,
  },
});
```

```

```

# 3. Indexing

**Indexes turn lookups instant**

**Seq Scan**

no index — slow

**Index Scan**

jumps straight there — fast

```prisma
model User {
  id    String @id @default(cuid())
  email String @unique
  name  String

  @@index([name])
}

```

```sql
EXPLAIN ANALYZE
SELECT * FROM "User"
WHERE email = 'test@gmail.com';

-> Index Scan using User_email_key

```

```

```

# 4. The n+1 query problem

❌ **N+1**

```typescript
// ❌ N+1 problem
const users = await prisma.user.findMany();

for (const user of users) {
  await prisma.post.findMany({
    where: { userId: user.id },
  });
}
```

✅ **Fixed — 1-2 queries total**

```typescript
const users = await prisma.user.findMany({
  include: {
    posts: true,
  },
});
```

# 5. Connect poole

```

```

# Summary

**Before you ship, check this**

- 💹 Do I need this request at all?
- 💹 Can I cache it?
- 💹 Is the query selecting only what's needed?
- 💹 Do I have the right index?
- 💹 Is there a hidden N+1 loop?
- 💹 Is connection pooling configured correctly?
