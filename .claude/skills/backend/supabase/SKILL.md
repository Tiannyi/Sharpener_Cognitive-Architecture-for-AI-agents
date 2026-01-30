---
description: Supabase Postgres, row-level security, Edge Functions, real-time subscriptions.
---

# Backend Sub-Skill: Supabase

> **Load when the task involves Postgres, Edge Functions, or row-level security policies.**

## Project Setup

```bash
# Install Supabase CLI
npm install -g supabase

# Initialize
supabase init
supabase start  # Local development
```

## Data Modeling (Postgres)

```sql
-- Users table (extends Supabase auth.users)
CREATE TABLE public.profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  display_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Enable Row Level Security
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;
```

## Row-Level Security (RLS)

```sql
-- Users can read any profile
CREATE POLICY "Public profiles are viewable by everyone"
  ON public.profiles FOR SELECT
  USING (true);

-- Users can only update their own profile
CREATE POLICY "Users can update own profile"
  ON public.profiles FOR UPDATE
  USING (auth.uid() = id);

-- Users can only insert their own profile
CREATE POLICY "Users can insert own profile"
  ON public.profiles FOR INSERT
  WITH CHECK (auth.uid() = id);
```

**Rule: Always enable RLS on every table.** Forgetting this exposes all data to any authenticated user.

## Edge Functions (Deno/TypeScript)

```typescript
// supabase/functions/my-function/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get("SUPABASE_URL")!,
    Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!
  );

  // Business logic here
  const { data, error } = await supabase
    .from("profiles")
    .select("*");

  return new Response(JSON.stringify({ data }), {
    headers: { "Content-Type": "application/json" },
  });
});
```

## Real-Time Subscriptions

```typescript
const channel = supabase
  .channel('public:messages')
  .on('postgres_changes',
    { event: 'INSERT', schema: 'public', table: 'messages' },
    (payload) => {
      console.log('New message:', payload.new);
    }
  )
  .subscribe();
```

## Common Patterns

- **Database functions:** Use Postgres functions for complex logic (runs server-side)
- **Foreign keys:** Use proper relational modeling — this is Postgres, leverage it
- **Indexes:** Add indexes on columns you filter/sort by frequently
- **Migrations:** Use `supabase migration new` to version schema changes

## What NOT to Build

- Custom REST API — Supabase auto-generates REST from your schema (PostgREST)
- Custom auth — use Supabase Auth
- Custom real-time server — use Supabase Realtime
- Custom file storage API — use Supabase Storage
