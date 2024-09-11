# pg-compat

Patch to make pg work with both Node.js and Cloudflare.

Target version `pg@8.12.0`

# usage

- wrangler.toml

```toml
compatibility_date = "2024-08-21"
compatibility_flags = ["nodejs_compat_v2"]
```

```tsx
import { LoaderFunctionArgs } from "@remix-run/cloudflare";
import pg from "pg";
import { PrismaClient } from "@prisma/client";
import { useLoaderData } from "@remix-run/react";
import { PrismaPg } from "@prisma/adapter-pg";

export default function Index() {
  const values = useLoaderData<string[]>();
  return (
    <div>
      {values.map((v) => (
        <div key={v}>{v}</div>
      ))}
    </div>
  );
}

export async function loader({
  context,
}: LoaderFunctionArgs): Promise<string[]> {
  const url = new URL(context.cloudflare.env.DATABASE_URL);
  const schema = url.searchParams.get("schema") ?? undefined;
  const pool = new pg.Pool({
    connectionString: context.cloudflare.env.DATABASE_URL,
  });
  const adapter = new PrismaPg(pool, { schema });
  const prisma = new PrismaClient({ adapter });
  await prisma.test.create({ data: {} });
  return prisma.test.findMany().then((r) => r.map(({ id }) => id));
}
```
