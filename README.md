# Next.js meetballs 🧆 — 2025‑07‑09

Welcome to the 2‑hour mini‑workshop! In just 120 minutes you'll build a tiny but complete web app using **Next.js 15 App Router**, **SQLite**, **Prisma ORM** and **NextAuth.js** (Credentials provider). By the end you'll have:

* A fully‑featured CRUD interface for **Recipes** saved in SQLite.
* Secure authentication with hashed passwords.
* Middleware‑protected routes (`/dashboard`, `/api/recipes`).
* Tailwind‑styled pages ready for deployment.

> **Why “meetballs”?** Because CRUD stands for *Create‑Read‑Update‑Delicious*.

---

## 0 · Prerequisites

| Tool          | Version                | Check           |
| ------------- | ---------------------- | --------------- |
| Node.js       | **18.18 LTS** or later | `node -v`       |
| npm (bundled) | ≥ 10.x                 | `npm -v`        |
| git           | latest                 | `git --version` |

Optional:

* VS Code + Tailwind CSS & Prisma extensions
* GUI for SQLite (TablePlus, DB Browser)

---

## 1 · Kick‑off (10 min)

```bash
npx create-next-app@latest nextjs-meetballs --ts --tailwind --app --eslint
cd nextjs-meetballs
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to verify the scaffold.

---

## 2 · Tailwind polish (5 min)

`tailwind.config.ts`

```ts
export default {
  content: ['./src/**/*.{ts,tsx}'],
  theme: { extend: {} },
  plugins: [],
}
```

---

## 3 · Database & Prisma (15 min)

```bash
npm i -D prisma
npm i @prisma/client
npx prisma init --datasource-provider sqlite
```

`prisma/schema.prisma`

```prisma
datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model Recipe {
  id        Int      @id @default(autoincrement())
  title     String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  servings  Int      @default(4)
  steps     String
}
```

Create DB & open Prisma Studio:

```bash
npx prisma migrate dev --name init
npx prisma studio
```

---

## 4 · App Router crash course (10 min)

* `/app` is your root.
* Folder name = URL segment.
* `page.tsx` → UI, `route.ts` → API.
* Group routes with `(group)` syntax.

---

## 5 · CRUD API (20 min)

`app/api/recipes/route.ts`

```ts
import { NextResponse } from 'next/server'
import { prisma } from '@/lib/prisma'

export async function GET() {
  const data = await prisma.recipe.findMany({ orderBy: { id: 'desc' } })
  return NextResponse.json(data)
}

export async function POST(request: Request) {
  const body = await request.json()
  const data = await prisma.recipe.create({ data: body })
  return NextResponse.json(data, { status: 201 })
}
```

Add `app/api/recipes/[id]/route.ts` with GET, PUT and DELETE.

---

## 6 · Pages & Forms (20 min)

* `app/(public)/recipes/page.tsx` → list view
* `app/(protected)/dashboard/page.tsx` → form, edit & stats
* `@/components/RecipeForm.tsx` for controlled form that calls the API with `fetch`

---

## 7 · Authentication (20 min)

```bash
npm i next-auth bcrypt
```

`app/api/auth/[...nextauth]/route.ts`

```ts
import NextAuth from 'next-auth'
import CredentialsProvider from 'next-auth/providers/credentials'
import bcrypt from 'bcryptjs'
import { prisma } from '@/lib/prisma'

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      credentials: { email: {}, password: {} },
      async authorize(credentials) {
        const user = await prisma.user.findUnique({ where: { email: credentials.email } })
        if (user && bcrypt.compareSync(credentials.password, user.password)) {
          return { id: user.id.toString(), email: user.email }
        }
        return null
      },
    }),
  ],
  session: { strategy: 'jwt' },
})
export { handler as GET, handler as POST }
```

`.env`

```
DATABASE_URL=file:./dev.db
NEXTAUTH_SECRET=yourSuperSecret
```

Seed an admin user:

```bash
node prisma/seed.mjs
```

---

## 8 · Protect routes with middleware (10 min)

`middleware.ts`

```ts
import { withAuth } from 'next-auth/middleware'

export default withAuth({
  pages: { signIn: '/login' },
})

export const config = {
  matcher: ['/dashboard/:path*', '/api/recipes/:path*'],
}
```

---

## 9 · Deploy (optional)

```bash
git init && git add . && git commit -m '🧆 initial'
vercel
```

---

## 🗓️ Two‑Hour Agenda

| Min     | Topic               |
| ------- | ------------------- |
| 0‑10    | Scaffold project    |
| 10‑25   | App Router basics   |
| 25‑45   | Prisma & DB         |
| 45‑65   | CRUD API            |
| 65‑85   | UI + Forms          |
| 85‑105  | Auth                |
| 105‑115 | Middleware / Guards |
| 115‑120 | Q\&A                |

---

### Further reading

* Next.js docs: [https://nextjs.org/docs](https://nextjs.org/docs)
* Prisma docs: [https://www.prisma.io/docs](https://www.prisma.io/docs)
* NextAuth.js docs: [https://next-auth.js.org](https://next-auth.js.org)

Happy coding & buon appetito! 🧆
