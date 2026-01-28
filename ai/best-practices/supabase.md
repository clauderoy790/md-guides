# Supabase Best Practices

## Migrations

1. **Always create migration files** — When creating tables, policies, functions, or any database changes, create a SQL migration file in `supabase/migrations/` with a numbered prefix (e.g., `001_feature_name.sql`, `002_another_feature.sql`).

2. **Clearly instruct the user to run the migration** — After creating a migration file, explicitly tell the user:
   - The file path
   - That they need to run it in the Supabase SQL Editor
   - Example: "Run the SQL from `supabase/migrations/001_business_examples.sql` in your Supabase SQL Editor."

3. **Include all related schema in one migration** — A single feature's migration should include:
   - Table creation
   - RLS enablement
   - All RLS policies
   - Any indexes or constraints
   - Related functions/triggers if needed

## Row Level Security (RLS)

1. **Always enable RLS** on user-facing tables
2. **Create policies for all CRUD operations** the app needs (select, insert, update, delete)
3. **Use `auth.uid()` for user-scoped data** — Standard pattern: `using (auth.uid() = user_id)`

## Error Handling

1. **Never throw raw Supabase errors** in server actions — Supabase errors are objects that can't be serialized across the server/client boundary in Next.js
2. **Wrap errors in plain Error objects**: `throw new Error(error.message)` instead of `throw error`
