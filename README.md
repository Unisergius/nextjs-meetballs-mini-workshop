# Next.js Meetballs 🧆 — 2025‑07‑09

Welcome to the 2‑hour mini‑workshop! In just 120 minutes you’ll build a small but production‑ready app that shows off **Next.js 15 App Router** advantages—nested layouts, streaming server components, the new Metadata API for SEO, and **shadcn/ui** for beautiful React‑based design. We’ll fetch live data from a public API through a Next.js Route Handler so you leave knowing how to bridge server code and client code—*no database required*.

By the end you’ll have:

* A fully‑featured `/news` page that hydrates from a server‑side fetch.
* Strong on‑page SEO (dynamic `<title>`, canonical URLs, Open Graph images).
* Type‑safe API route (`/api/news`) that proxies a 3rd‑party service.
* Middleware that adds simple request logging & optional token guard.
* A sprinkle of **shadcn/ui** components styled with Tailwind.

---

## 0 · Prerequisites

| Tool          | Version                | Check           |
| ------------- | ---------------------- | --------------- |
| Node.js       | **18.18 LTS** or later | `node -v`       |
| npm (bundled) | ≥ 10.x                 | `npm -v`        |
| git           | latest                 | `git --version` |

Optional:

* VS Code + Tailwind CSS extension
* Vercel CLI for easy deploy (`npm i -g vercel`)

---

## 1 · Kick‑off (10 min)

> **What happens here?** We scaffold a brand‑new Next.js 15 project pre‑wired with TypeScript, Tailwind, the App Router and ESLint. Saying **Yes** to the `src/` directory keeps your code organised; **Turbopack** accelerates local HMR, and sticking with the default `@/*` alias avoids extra config.

```bash
npx create-next-app@latest nextjs-meetballs --ts --tailwind --app --eslint
#   ✔ Where should we create your project? › ./nextjs-meetballs
#   ✔ Would you like to use src/ directory? › Yes
#   ✔ Would you like to use Turbopack? › Yes
#   ✔ Customize the default import alias ( @/* )? › No
#   ✔ Ready to bake your meetballs!

cd nextjs-meetballs
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to verify the scaffold.

---

## 2 · Tailwind polish (5 min)

> **Why?** Tailwind is already installed, but we tighten the `content` glob so the JIT compiler only scans files under `src/`. This shaves compile times and eliminates unused styles.

`tailwind.config.ts`

```ts
export default {
  content: ['./src/**/*.{ts,tsx}'],
  theme: { extend: {} },
  plugins: [],
}
```

---

## 3 · App Router in depth (25 min)

> **Goal:** Understand how folder structure becomes the API.

1. **Layouts & nesting** – create `app/layout.tsx` for the HTML shell; nest `app/(public)/layout.tsx` for the marketing section.
2. **Dynamic routes** – add `app/news/[id]/page.tsx` and implement `generateStaticParams` for SSG.
3. **Server vs Client components** – default to server (no JS shipped) and use `'use client'` only when interactivity is required.
4. **Streaming** – demonstrate incremental rendering by adding an artificial delay to a child component.

---

## 4 · SEO & Metadata API (15 min)

> **Objective:** Make every page discoverable and share‑worthy.

https://nextjs.org/learn/seo

* Use the `metadata` export in `app/news/[id]/page.tsx` to set dynamic titles and descriptions.
* Generate Open Graph & Twitter card images with the built‑in Image Response (`app/news/[slug]/opengraph-image.tsx`).

Example snippet:

```ts
export const generateMetadata: MetadataFactory = async ({ params }) => {
  const article = await fetchArticle(params.slug)
  return {
    title: `${article.title} | Meetballs` ,
    description: article.summary,
    alternates: { canonical: `/news/${params.slug}` },
    openGraph: { ... }
  }
}
```

---

## 5 · shadcn/ui & External API (20 min)

> **Task:** Install shadcn, fetch live data on the server, display it with fancy components.

```bash
npx shadcn@latest init
npx shadcn@latest add card button
```

### 5a • Route Handler

Create `app/api/news/route.ts` that proxies [NewsAPI.org](https://newsapi.org) (or your chosen public API):

```ts
import { NextResponse } from 'next/server'

export async function GET() {
  const res = await fetch(`https://newsapi.org/v2/top-headlines?country=us&apiKey=${process.env.NEWS_KEY}`)
  const json = await res.json()
  return NextResponse.json(json)
}
```

### 5b • Client page

`app/news/page.tsx`

```tsx
'use client'
import useSWR from 'swr'
import { Card, CardContent } from '@/components/ui/card'

const fetcher = (url: string) => fetch(url).then(r => r.json())

export default function NewsPage() {
  const { data, isLoading } = useSWR('/api/news', fetcher)
  if (isLoading) return <p>Loading…</p>
  return (
    <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
      {data.articles.map((a: any) => (
        <Card key={a.url}>
          <CardContent>
            <h2 className="font-semibold mb-2">{a.title}</h2>
            <p className="text-sm opacity-70">{a.description}</p>
          </CardContent>
        </Card>
      ))}
    </div>
  )
}
```

---

## 6 · Middleware (10 min)

> **Why:** Edge‑executed logic before page/API resolution—here we’ll log timing info and optionally block requests without a query token.

`middleware.ts`

```ts
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const start = Date.now()
  const response = Response.next()
  response.headers.set('x-processing-time', `${Date.now() - start}ms`)

  // Simple token gate for protected routes
  if (request.nextUrl.pathname.startsWith('/news') && !request.nextUrl.searchParams.get('token')) {
    return Response.redirect(new URL('/?error=token', request.url))
  }
  return response
}

export const config = {
  matcher: ['/news/:path*', '/api/news/:path*'],
}
```

---

## 7 · Deploy (optional)

> **Bonus:** Push to **Vercel**; Next.js 15 auto‑optimises images, streaming & edge middleware.

```bash
git init && git add . && git commit -m '🧆 initial'
vercel
```

---

## 🗓️ Two‑Hour Agenda

| Min     | Topic                      |
| ------- | -------------------------- |
| 0‑10    | Scaffold project           |
| 10‑15   | Tailwind polish            |
| 15‑40   | App Router deep dive       |
| 40‑55   | SEO & Metadata             |
| 55‑75   | shadcn install + API route |
| 75‑95   | Build News page            |
| 95‑105  | Middleware                 |
| 105‑120 | Deploy & Q\&A              |

---

## Architecture overview

Next.js 15 with the App Router embraces the **“file‑system is the API”** philosophy—folders become routes and nested React Server Components form a streaming UI tree. With no database, the only server state lives in *external APIs* and Edge Middleware.

```text
/app
  layout.tsx        # Root layout (HTML shell & providers)
  (public)/
    page.tsx        # GET /
  news/
    page.tsx        # Client page fetching /api/news
    [slug]/page.tsx # Dynamic route with generateStaticParams
    opengraph-image.tsx # OG image function
  api/
    news/route.ts   # Proxy to NewsAPI
middleware.ts       # Edge guard + metrics
```

| Piece                 | What it does                                                           |
| --------------------- | ---------------------------------------------------------------------- |
| **Layouts**           | Persistent wrappers streamed down; perfect for navbars & global styles |
| **Server Components** | Default—run on the server; no JS shipped unless necessary              |
| **Client Components** | Add `'use client'` for interactive pieces (forms, modals)              |
| **Route Handlers**    | File‑based API: any HTTP verb; lives alongside pages                   |
| **Middleware**        | Edge‑executed logic before page/API resolution (auth, logging)         |

Typical request lifecycle: **Edge Middleware → Server Components/layouts → External API fetch → Client Component hydration**.

### Further reading

* Next.js docs: [https://nextjs.org/docs](https://nextjs.org/docs)
* Metadata API: [https://nextjs.org/docs/app/building-your-application/optimizing/metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
* shadcn/ui: [https://ui.shadcn.com](https://ui.shadcn.com)

Happy coding & buon appetito! 🧆
