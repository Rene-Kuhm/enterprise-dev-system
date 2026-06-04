# FRONTEND MASTERCLASS
## The Ultimate Frontend Stack for 2026
### Next.js 16 · React 19.2 · React Compiler · Cache Components

---

> **Versión:** 3.0
> **Fecha:** Junio 2026
> **Stack target:** Next.js 16.2+ · React 19.2 · Node.js 22 LTS (mínimo 20.9) · TypeScript 5.6+

---

## Índice

- [Parte I: Next.js 16 + React 19.2 — The Modern Stack](#parte-i-nextjs-16---react-192--the-modern-stack)
- [Parte II: React Compiler — Auto-Memoization](#parte-ii-react-compiler--auto-memoization)
- [Parte III: Cache Components — Explicit Caching](#parte-iii-cache-components--explicit-caching)
- [Parte IV: Performance — Core Web Vitals 2026](#parte-iv-performance--core-web-vitals-2026)
- [Parte V: State Management en 2026](#parte-v-state-management-en-2026)
- [Parte VI: Styling — Tailwind 4.1](#parte-vi-styling--tailwind-41)
- [Parte VII: Frontend Security — Defense in Depth](#parte-vii-frontend-security--defense-in-depth)
- [Parte VIII: SEO — The Complete Strategy](#parte-viii-seo--the-complete-strategy)
- [Parte IX: AI-Powered Frontend (Bonus)](#parte-ix-ai-powered-frontend-bonus)
- [Parte X: Checklists Completos](#parte-x-checklists-completos)

---

## Parte I: Next.js 16 + React 19.2 — The Modern Stack

### 1. Requisitos y Setup Inicial

```bash
# Node.js 22 LTS recomendado (mínimo 20.9 para Next 16)
nvm install 22
nvm use 22

# Crear proyecto
npx create-next-app@latest mi-app \
  --typescript \
  --tailwind \
  --app \
  --src-dir \
  --turbopack \
  --import-alias "@/*"

cd mi-app
pnpm dev
```

### 2. Cambios clave de Next.js 16 vs 15

| Feature | Next 15 | Next 16 |
|---|---|---|
| Bundler default | Webpack/Turbopack opcional | **Turbopack default** (dev + prod) |
| Caching | `fetch` cache implícito | **`'use cache'` explícito** |
| React Compiler | Beta | **Stable, integrado** |
| Routing | Prefetch estático | **Smarter routing** (predictive prefetch) |
| Deploy | Solo Vercel optimizado | **Adapter API estable** (Cloudflare, Deno, Bun) |
| File system cache | No | **`turbopackFileSystemCacheForDev`** |
| `dynamicParams` | Config por ruta | **Default `false`** (más explícito) |
| Node.js mínimo | 18.18 | **20.9** |

### 3. Turbopack: Default en Producción

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Turbopack es el default en Next 16 — no requiere opt-in
  // Para volver a webpack (no recomendado):
  // $ next dev --webpack
  // $ next build --webpack

  // File system cache: acelera restarts en monorepos
  experimental: {
    turbopackFileSystemCacheForDev: true,
  },

  // Cache Components (la nueva forma de hacer caching)
  cacheComponents: true,

  // React Compiler (stable)
  reactCompiler: true,

  // Typed Routes (sigue disponible, ahora más rápido)
  typedRoutes: true,
};

export default nextConfig;
```

**Benchmarks reales Next 16 vs 15 (Vercel, prod build):**
- Build production: **2-5× más rápido**
- Fast Refresh: **hasta 10× más rápido**
- Restart con file system cache: **~80% reducción de tiempo**

### 4. React Server Components (RSC) en 2026

```typescript
// app/posts/[id]/page.tsx
// Server Component — nunca llega al cliente como JS
import { db } from '@/lib/database';
import { Comments } from './comments';
import { notFound } from 'next/navigation';

interface PostPageProps {
  params: Promise<{ id: string }>; // Next 16: params es Promise
}

// ⚠️ Next 16: params y searchParams ahora son PROMISES (rompe con Next 15)
async function PostPage({ params }: PostPageProps) {
  const { id } = await params; // Hay que awaitear

  // Fetch en paralelo
  const [post, relatedPosts] = await Promise.all([
    db.posts.findUnique({
      where: { id },
      include: { author: true, tags: true }
    }),
    db.posts.findMany({ where: { authorId: id }, take: 5 })
  ]);

  if (!post) notFound();

  return (
    <article>
      <h1>{post.title}</h1>
      <PostContent content={post.content} />

      {/* Boundary de cliente para interactividad */}
      <Comments postId={post.id} initialComments={post.comments} />
    </article>
  );
}

export default PostPage;
```

### 5. Server Actions — Sin API Routes

```typescript
// app/actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const createPostSchema = z.object({
  title: z.string().min(3).max(200),
  content: z.string().min(10),
  tags: z.array(z.string()).max(5),
});

export async function createPost(formData: FormData) {
  const validated = createPostSchema.parse({
    title: formData.get('title'),
    content: formData.get('content'),
    tags: formData.getAll('tags'),
  });

  const post = await db.post.create({
    data: {
      ...validated,
      authorId: (await auth()).userId,
    },
  });

  // Revalidación por path
  revalidatePath('/posts');
  // O por tag (mejor para Cache Components)
  revalidateTag('posts');

  redirect(`/posts/${post.id}`);
}
```

```tsx
// components/post-form.tsx
'use client';

import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';
import { createPost } from '@/app/actions';

const initialState = { error: null, success: false };

export function PostForm() {
  const [state, action] = useActionState(createPost, initialState);

  return (
    <form action={action}>
      <input name="title" placeholder="Título" required minLength={3} />
      <textarea name="content" placeholder="Contenido" required minLength={10} />

      <SubmitButton />

      {state?.error && (
        <p role="alert" className="text-red-500">{state.error}</p>
      )}
    </form>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending} aria-busy={pending}>
      {pending ? 'Publicando...' : 'Publicar'}
    </button>
  );
}
```

### 6. useOptimistic — Actualizaciones Instantáneas

```tsx
'use client';

import { useOptimistic, useTransition } from 'react';
import { toggleLike } from '@/app/actions';

interface Post {
  id: string;
  likes: number;
  liked: boolean;
}

export function LikeButton({ post }: { post: Post }) {
  const [isPending, startTransition] = useTransition();
  const [optimisticPost, setOptimisticPost] = useOptimistic(
    post,
    (current) => ({
      ...current,
      likes: current.liked ? current.likes - 1 : current.likes + 1,
      liked: !current.liked,
    })
  );

  const handleLike = () => {
    startTransition(async () => {
      setOptimisticPost({ ...optimisticPost });
      await toggleLike(post.id);
    });
  };

  return (
    <button
      onClick={handleLike}
      disabled={isPending}
      aria-pressed={optimisticPost.liked}
      aria-label={`${optimisticPost.likes} likes`}
    >
      {optimisticPost.liked ? '❤️' : '🤍'} {optimisticPost.likes}
    </button>
  );
}
```

### 7. El nuevo `use()` Hook de React 19

```tsx
// Lee Promises o Context directamente en render
import { use, Suspense } from 'react';

function PostPage({ postPromise }: { postPromise: Promise<Post> }) {
  // use() suspende el componente hasta que la Promise resuelva
  const post = use(postPromise);

  return (
    <article>
      <h1>{post.title}</h1>
    </article>
  );
}

// Wrapper con Suspense
export default function Page({ postId }: { postId: string }) {
  const postPromise = fetch(`/api/posts/${postId}`).then(r => r.json());

  return (
    <Suspense fallback={<PostSkeleton />}>
      <PostPage postPromise={postPromise} />
    </Suspense>
  );
}

// use() también funciona con Context condicionalmente
function ThemeToggle({ themePromise }: { themePromise: Promise<Theme> }) {
  // Esto NO se puede hacer con useContext
  if (Math.random() > 0.5) {
    const theme = use(themePromise); // ✅ permitido condicional
    return <button>{theme.name}</button>;
  }
  return <button>Default</button>;
}
```

### 8. Smarter Routing en Next 16

```typescript
// app/dashboard/layout.tsx
// Next 16: prefetching predictivo basado en viewport y mouse hover
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <Sidebar /> {/* Links se prefetchan cuando entran al viewport */}
      <main>{children}</main>
    </div>
  );
}

// Configuración explícita cuando necesitás control fino
export const prefetch = {
  // Solo en hover (default en Next 16)
  strategy: 'hover',
  // Opciones:
  // 'viewport' — cuando el link entra al viewport
  // 'hover' — cuando el usuario hace hover
  // 'never' — deshabilitar prefetch
  // 'auto' — Next.js decide
};
```

---

## Parte II: React Compiler — Auto-Memoization

### 9. ¿Qué es React Compiler?

Es una herramienta de build-time que **memoiza automáticamente** componentes y hooks. Reemplaza el 90% de los casos donde usarías `useMemo`, `useCallback` o `React.memo`.

```typescript
// next.config.js
const nextConfig = {
  reactCompiler: true, // Habilita el compilador
};

// .babelrc (alternativa para proyectos sin Next)
{
  "presets": [
    ["babel-preset-react-compiler", {
      "target": "19" // Target React 19+
    }]
  ]
}
```

### 10. Antes vs Después del Compiler

```typescript
// ❌ ANTES (Next 15 sin Compiler)
const ProductList = ({ products, filter }) => {
  const filtered = useMemo(
    () => products.filter(p => p.category === filter),
    [products, filter]
  );

  const handleClick = useCallback((id: string) => {
    console.log('clicked', id);
  }, []);

  return (
    <div>
      {filtered.map(p => (
        <ProductCard
          key={p.id}
          product={p}
          onClick={handleClick}  // Referencia estable
        />
      ))}
    </div>
  );
};

const ProductCard = React.memo(({ product, onClick }) => {
  // Solo re-renderiza si product u onClick cambian
  return <div onClick={() => onClick(product.id)}>{product.name}</div>;
});

// ✅ DESPUÉS (Next 16 con Compiler)
const ProductList = ({ products, filter }) => {
  // El compiler memoiza automáticamente
  const filtered = products.filter(p => p.category === filter);

  const handleClick = (id: string) => {
    console.log('clicked', id);
  };

  return (
    <div>
      {filtered.map(p => (
        <ProductCard
          key={p.id}
          product={p}
          onClick={handleClick}  // Compiler lo memoiza
        />
      ))}
    </div>
  );
};

// ProductCard no necesita React.memo — el compiler lo hace internamente
const ProductCard = ({ product, onClick }) => {
  return <div onClick={() => onClick(product.id)}>{product.name}</div>;
};
```

### 11. Cuándo SÍ necesitás memoización manual

```typescript
// ⚠️ El compiler no optimiza:
// 1. Efectos secundarios con dependencias externas
useEffect(() => {
  const handler = () => console.log(count);
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}, [count]); // Sigue necesitando deps

// 2. Refs que deben mantener identidad
const mapRef = useRef(new Map()); // OK, useRef no se memoiza

// 3. Cálculos que deben correr OBLIGATORIAMENTE en cada render
// (raro, pero existe — ej: timestamps, random)

// Para "desactivar" el compiler en un componente:
const MyComponent = () => {
  "use no memo"; // Directive del compiler
  // ...
};
```

### 12. React Compiler DevTools

```typescript
// Instalar extensión
// Chrome/Edge: "React Developer Tools" → tab "Components" → activar "Highlight updates"
// Firefox: idem

// En código, marcar componentes para debugging
const MyComponent = () => {
  "use memo trace"; // Log cuando se compila
  return <div>...</div>;
};
```

---


## Parte III: Cache Components — Explicit Caching

> **El cambio más importante de Next.js 16.** Reemplaza el caching implícito de Next 15.

### 13. El problema del caching implícito

```typescript
// ❌ Next 15: el caching dependía del método HTTP, headers, etc.
// Era confuso y propenso a bugs
const data = await fetch('https://api.example.com/posts');
// ¿Esto se cachea? ¿Por cuánto? ¿Cómo lo invalido?
// Depende de Next.js, no de vos
```

```typescript
// ✅ Next 16: caching EXPLÍCITO con 'use cache'
// Si no ponés la directive, NO se cachea (dynamic by default)
import { unstable_cacheTag as cacheTag } from 'next/cache';

async function getPosts() {
  'use cache';
  cacheTag('posts', 'posts-list');

  const res = await fetch('https://api.example.com/posts');
  return res.json();
}
```

### 14. Tres niveles de `'use cache'`

```typescript
// ──── NIVEL 1: A nivel de archivo ────
// app/posts/page.tsx
'use cache';  // Cachea TODA la página

export default async function PostsPage() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}

// ──── NIVEL 2: A nivel de componente ────
// app/posts/featured.tsx
export async function FeaturedPosts() {
  'use cache';
  cacheTag('featured-posts');
  cacheLife('hours');  // Vida del cache

  const posts = await db.posts.findMany({ where: { featured: true } });
  return <section>{/* ... */}</section>;
}

// ──── NIVEL 3: A nivel de función ────
// lib/api/products.ts
import { unstable_cacheTag as cacheTag, unstable_cacheLife as cacheLife } from 'next/cache';

export async function getProductById(id: string) {
  'use cache';
  cacheTag(`product-${id}`, 'products');
  cacheLife('days');  // Reutilizable

  return db.products.findUnique({ where: { id } });
}
```

### 15. `cacheLife` — Cuánto vive el cache

```typescript
import { unstable_cacheLife as cacheLife } from 'next/cache';

// Perfiles predefinidos
async function getData() {
  'use cache';
  cacheLife('minutes');  // 5 min fresh, 5 min stale
  cacheLife('hours');    // 1 hora fresh, 1 día stale
  cacheLife('days');     // 1 día fresh, 1 semana stale
  cacheLife('weeks');    // 1 semana fresh, 30 días stale
  cacheLife('max');      // Lo más largo posible

  // O custom:
  cacheLife({
    stale: 3600,        // 1 hora — servir cache viejo mientras se revalida
    revalidate: 7200,   // 2 horas — frecuencia de revalidación
    expire: 86400,      // 1 día — tiempo máximo de vida
  });
}
```

### 16. `cacheTag` + `revalidateTag` — Invalidación quirúrgica

```typescript
// app/actions.ts
'use server';

import { revalidateTag } from 'next/cache';

export async function updatePost(id: string, data: PostData) {
  await db.posts.update({ where: { id }, data });

  // Invalida SOLO los caches con estos tags
  revalidateTag(`post-${id}`);  // El cache específico de este post
  revalidateTag('posts');        // El listado de posts
  revalidateTag('featured-posts'); // Si estaba en featured
}

// Incluso desde un webhook
// app/api/webhooks/post-update/route.ts
export async function POST(req: Request) {
  const { postId } = await req.json();

  // Re-fetch del post para invalidar su cache
  revalidateTag(`post-${postId}`);

  return Response.json({ ok: true });
}
```

### 17. Combinando Cache Components + PPR

```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react';

// PPR (Partial Prerendering) ahora usa Cache Components
export default function Dashboard() {
  return (
    <div>
      {/* Shell estático — prerenderizado en build */}
      <DashboardHeader />
      <DashboardNav />

      {/* Parte dinámica — renderizada on-demand */}
      <Suspense fallback={<MetricsSkeleton />}>
        <LiveMetrics />
      </Suspense>

      {/* Parte cacheada explícitamente — prerenderizada + revalidable */}
      <Suspense fallback={<ReportsSkeleton />}>
        <CachedReports />
      </Suspense>
    </div>
  );
}

async function CachedReports() {
  'use cache';
  cacheTag('reports', 'monthly-reports');
  cacheLife('hours');

  const reports = await db.reports.findMany();
  return <ReportList reports={reports} />;
}

async function LiveMetrics() {
  // No cache — siempre fresh
  const metrics = await fetch('https://api.example.com/metrics', {
    cache: 'no-store',
  }).then(r => r.json());

  return <MetricsView data={metrics} />;
}
```

### 18. Errores comunes con `'use cache'`

```typescript
// ❌ ERROR 1: La directive debe estar AL INICIO de la función
function wrapper() {
  async function getData() {
    'use cache'; // ⚠️ No funciona acá, está "escondida"
    return fetch('/api');
  }
}

// ✅ CORRECTO
async function getData() {
  'use cache';  // Primera línea ejecutable
  return fetch('/api');
}

// ❌ ERROR 2: revalidateTag sin segundo argumento (Next 16 cambió la API)
// En Next 16, revalidateTag REQUIERE especificar qué invalidar
revalidateTag('posts'); // ❌ No hace nada en Next 16

// ✅ CORRECTO
revalidateTag('posts', 'infinite'); // Invalida todo el cache con este tag
revalidateTag('posts', 'hours');   // Invalida solo el cache de "hours" life

// ❌ ERROR 3: Asumir que un fetch se cachea solo
const data = await fetch('/api/data');
// Por defecto, en Next 16, fetch NO se cachea

// ✅ CORRECTO: Usar 'use cache' explícitamente
async function getData() {
  'use cache';
  cacheTag('data');
  const res = await fetch('/api/data');
  return res.json();
}
```

### 19. Debugging de Cache Components

```typescript
// En dev, los errores de 'use cache' se loguean en consola
// Para debugging más fino:

// next.config.js
const nextConfig = {
  experimental: {
    // Muestra logs detallados de cada cache hit/miss
    cacheComponentsDebugLogs: true,
  },
};

// O desde código:
import { unstable_cacheTag as cacheTag } from 'next/cache';

async function getData() {
  'use cache';
  cacheTag('data');
  console.log('[CACHE MISS] getData ejecutándose');
  return db.data.findMany();
}
```

---

## Parte IV: Performance — Core Web Vitals 2026

### 20. Targets 2026 (sin cambios desde 2024, pero ahora son más estrictos en SEO)

| Métrica | Good | Needs Improvement | Poor |
|---------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5s - 4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200ms - 500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1 - 0.25 | > 0.25 |
| **TTFB** (Time to First Byte) | ≤ 800ms | 800ms - 1800ms | > 1800ms |
| **FCP** (First Contentful Paint) | ≤ 1.8s | 1.8s - 3.0s | > 3.0s |

> **Nota 2026:** Google ya considera CWV como factor de ranking más fuerte. Sitios con "poor" CWV pierden 30-40% de tráfico orgánico vs sitios con "good".

### 21. LCP Optimization

```typescript
// components/hero-image.tsx
import Image from 'next/image';

// AVIF + WebP automático (Next 16 mantiene soporte)
export function HeroImage({ src, alt, priority = true }: HeroProps) {
  return (
    <Image
      src={src}
      alt={alt}
      width={1920}
      height={1080}
      priority={priority}  // CRÍTICO para LCP
      fetchPriority="high"
      placeholder="blur"
      blurDataURL={generateBlurPlaceholder(src)}
      sizes="
        (max-width: 640px) 100vw,
        (max-width: 1024px) 80vw,
        1920px
      "
      quality={85}  // Sweet spot quality/size
    />
  );
}
```

```tsx
// app/layout.tsx — Preload de la imagen LCP
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="es">
      <head>
        <link
          rel="preload"
          href="/hero.avif"
          as="image"
          type="image/avif"
          fetchPriority="high"
        />
        <link
          rel="preload"
          href="/fonts/inter-var.woff2"
          as="font"
          type="font/woff2"
          crossOrigin="anonymous"
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### 22. Font Optimization (2026)

```css
/* app/globals.css */
@font-face {
  font-family: 'Inter';
  font-display: swap;
  font-weight: 100 900;
  font-style: normal;
  src: url('/fonts/inter-var.woff2') format('woff2-variations');
}

/* Size-adjust para evitar CLS durante el swap */
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  size-adjust: 107%;
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}

@font-face {
  font-family: 'Inter';
  font-display: swap;
  src: url('/fonts/inter-var.woff2') format('woff2-variations');
}

body {
  font-family: 'Inter', 'Inter Fallback', sans-serif;
}
```

### 23. INP Optimization

```typescript
// lib/scheduler.ts
// Yielding al main thread para evitar Long Tasks
export function yieldToMain() {
  return new Promise(resolve => {
    // setTimeout con 0ms cede el control al browser
    setTimeout(resolve, 0);
  });
}

export async function processInChunks<T>(
  items: T[],
  processor: (item: T) => Promise<void>,
  chunkSize = 10
): Promise<void> {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    await Promise.all(chunk.map(processor));
    await yieldToMain();
  }
}
```

```typescript
// workers/csv-processor.ts
// Next 16 tiene first-class support para Web Workers
self.addEventListener('message', ({ data }) => {
  const { rows, transformations } = data;

  const processed = rows.map(row =>
    transformations.reduce((acc, fn) => fn(acc), row)
  );

  self.postMessage({ processed });
});
```

```tsx
// components/data-table.tsx
'use client';

import { useRef, useEffect, useState } from 'react';

export function DataTable({ data, transformations }) {
  const workerRef = useRef<Worker | null>(null);
  const [processed, setProcessed] = useState([]);

  useEffect(() => {
    // Next.js 16: Web Workers con type-safe imports
    workerRef.current = new Worker(
      new URL('../workers/csv-processor.ts', import.meta.url),
      { type: 'module' }
    );

    workerRef.current.onmessage = (e) => setProcessed(e.data.processed);
    workerRef.current.postMessage({ rows: data, transformations });

    return () => workerRef.current?.terminate();
  }, [data, transformations]);

  return <table>{/* render processed */}</table>;
}
```

### 24. CLS Optimization

```tsx
// REGLA: SIEMPRE dimensiones explícitas en imágenes y videos

// ✅ Imagen responsive con aspect-ratio
<div style={{ aspectRatio: '16/9', position: 'relative' }}>
  <Image
    src={hero}
    fill
    sizes="100vw"
    alt="Hero"
    style={{ objectFit: 'cover' }}
  />
</div>

// ✅ Video con dimensiones
<video
  width={1920}
  height={1080}
  poster="/video-poster.jpg"
  preload="metadata"
  style={{ aspectRatio: '16/9', width: '100%', height: 'auto' }}
>
  <source src="/video.mp4" type="video/mp4" />
</video>

// ✅ Skeleton para contenido dinámico
export function AdBanner() {
  return (
    <div
      className="skeleton"
      style={{
        minHeight: 250,
        aspectRatio: '300/250',
        background: 'linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%)',
        backgroundSize: '200% 100%',
        animation: 'shimmer 1.5s infinite',
      }}
      aria-hidden="true"
    />
  );
}
```

```css
/* Reservar espacio para fuentes (evita CLS por FOUT) */
.font-loading {
  visibility: hidden;
}

.fonts-loaded .font-loading {
  visibility: visible;
}

/* Reservar espacio para ads/widgets antes de que carguen */
.ad-slot {
  min-height: 250px;
  min-width: 300px;
  contain: layout;
}
```

### 25. Web Vitals Monitoring

```typescript
// lib/web-vitals.ts
import { onCLS, onINP, onLCP, onTTFB, onFCP } from 'web-vitals';
import type { Metric } from 'web-vitals';

const ENDPOINT = '/api/vitals';

export function reportWebVital(metric: Metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating,
    delta: metric.delta,
    id: metric.id,
    navigationType: metric.navigationType,
    // Attribution data (Next 16 + web-vitals 4)
    attribution: metric.attribution,
  });

  // keepalive: envía incluso en unload
  if (navigator.sendBeacon) {
    navigator.sendBeacon(ENDPOINT, body);
  } else {
    fetch(ENDPOINT, {
      method: 'POST',
      body,
      keepalive: true,
    });
  }
}

export function initWebVitals() {
  onCLS(reportWebVital);
  onINP(reportWebVital);
  onLCP(reportWebVital);
  onTTFB(reportWebVital);
  onFCP(reportWebVital);
}
```

```typescript
// app/api/vitals/route.ts
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const vital = await request.json();

  // Guardar en DB para análisis
  await db.vitals.create({
    data: {
      name: vital.name,
      value: vital.value,
      rating: vital.rating,
      url: request.headers.get('referer') ?? 'unknown',
      userAgent: request.headers.get('user-agent') ?? 'unknown',
      deviceType: detectDevice(vital),
      timestamp: new Date(),
    },
  });

  // Alerta si el rating es poor
  if (vital.rating === 'poor') {
    await fetch(process.env.SLACK_WEBHOOK_URL!, {
      method: 'POST',
      body: JSON.stringify({
        text: `🚨 Performance Alert: ${vital.name} = ${vital.value} (poor)\nURL: ${vital.url}`,
      }),
    });
  }

  return Response.json({ success: true });
}
```

---

## Parte V: State Management en 2026

### 26. Zustand 5 (sigue vigente en 2026 — v5.0.8+)

```typescript
// store/useStore.ts
import { create } from 'zustand';
import { persist, devtools, subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface Store {
  // State
  cart: CartItem[];
  user: { id: string; name: string } | null;

  // Actions
  addToCart: (item: CartItem) => void;
  removeFromCart: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  setUser: (user: Store['user']) => void;

  // Computed (usar selectors)
  get total(): number;
}

export const useStore = create<Store>()(
  devtools(
    persist(
      subscribeWithSelector(
        immer((set, get) => ({
          cart: [],
          user: null,

          addToCart: (item) => set((state) => {
            const existing = state.cart.find(c => c.id === item.id);
            if (existing) {
              existing.quantity += item.quantity;
            } else {
              state.cart.push(item);
            }
          }),

          removeFromCart: (id) => set((state) => {
            state.cart = state.cart.filter(c => c.id !== id);
          }),

          updateQuantity: (id, quantity) => set((state) => {
            const item = state.cart.find(c => c.id === id);
            if (item) item.quantity = quantity;
          }),

          clearCart: () => set({ cart: [] }),
          setUser: (user) => set({ user }),

          get total() {
            return get().cart.reduce(
              (sum, item) => sum + item.price * item.quantity,
              0
            );
          },
        }))
      ),
      {
        name: 'app-storage',
        partialize: (state) => ({ cart: state.cart, user: state.user }),
      }
    ),
    { name: 'AppStore' }
  )
);
```

```tsx
// Uso con selectores (evita re-renders innecesarios)
'use client';

import { useStore } from '@/store/useStore';

// ✅ SELECTOR: solo re-renderiza si cart cambia
export function CartCount() {
  const count = useStore((state) => state.cart.length);
  return <span>{count}</span>;
}

// ✅ SELECTOR con shallow comparison para arrays/objetos
import { useShallow } from 'zustand/react/shallow';

export function CartSummary() {
  const { items, total } = useStore(
    useShallow((state) => ({
      items: state.cart,
      total: state.cart.reduce((sum, i) => sum + i.price * i.quantity, 0),
    }))
  );
  return <div>{items.length} items, ${total}</div>;
}
```

### 27. TanStack Query v5 (sigue siendo v5 en React — v6 es solo Svelte)

```typescript
// lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,     // 5 min
      gcTime: 30 * 60 * 1000,       // 30 min
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: false,
    },
  },
});
```

```typescript
// hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useProducts(category?: string) {
  return useQuery({
    queryKey: ['products', { category }],
    queryFn: () => fetchProducts(category),
    staleTime: 5 * 60 * 1000,
  });
}

export function useProduct(id: string) {
  return useQuery({
    queryKey: ['product', id],
    queryFn: () => fetchProduct(id),
    // Next 16: integrar con Cache Components
    // No hacer fetch duplicado si ya está cacheado en el server
  });
}

export function useCreateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createProduct,
    onSuccess: (newProduct) => {
      // Invalidación quirúrgica
      queryClient.invalidateQueries({ queryKey: ['products'] });
      // O actualización optimista
      queryClient.setQueryData(['products'], (old: Product[] = []) => [
        ...old,
        newProduct,
      ]);
    },
  });
}
```

```tsx
// Integración con Next.js 16 App Router
// app/providers.tsx
'use client';

import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryStreamedHydration } from '@tanstack/react-query-next-experimental';
import { queryClient } from '@/lib/queryClient';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      <ReactQueryStreamedHydration>
        {children}
      </ReactQueryStreamedHydration>
    </QueryClientProvider>
  );
}
```

---

## Parte VI: Styling — Tailwind 4.1

### 28. Setup y novedades Tailwind 4.1

```bash
# Tailwind 4.1 — instalación simplificada
pnpm add tailwindcss@latest @tailwindcss/postcss
```

```css
/* app/globals.css — CSS-first config */
@import "tailwindcss";

/* Design tokens via @theme (nuevo en v4) */
@theme {
  --font-display: "Satoshi", "sans-serif";
  --font-sans: "Inter", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", monospace;

  --color-brand-50: oklch(0.97 0.02 240);
  --color-brand-500: oklch(0.55 0.20 240);
  --color-brand-900: oklch(0.25 0.15 240);

  --color-accent-500: oklch(0.65 0.25 50);

  --breakpoint-3xl: 1920px;

  /* Animaciones */
  --animate-fade-in: fadeIn 0.3s ease-out;
  --animate-slide-up: slideUp 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
```

### 29. Container Queries (Tailwind 4.1)

```tsx
// Componentes que responden al TAMAÑO de su contenedor, no del viewport
export function ProductCard({ product }: { product: Product }) {
  return (
    <div className="@container">
      <div className="flex flex-col @md:flex-row @lg:grid @lg:grid-cols-3 gap-4">
        <img
          src={product.image}
          className="w-full @md:w-48 @lg:w-full aspect-square object-cover rounded-lg"
        />
        <div className="flex-1">
          <h3 className="text-lg @xl:text-2xl font-bold">{product.name}</h3>
          <p className="text-sm @xl:text-base text-gray-600">
            {product.description}
          </p>
        </div>
      </div>
    </div>
  );
}
```

### 30. Nuevas utilities de Tailwind 4.1

```css
/* Text shadows (nuevo en v4.1) */
.heading {
  text-shadow: 0 2px 4px oklch(0 0 0 / 0.3);
}

/* Mask utilities (nuevo en v4.1) */
.masked-image {
  mask-image: linear-gradient(to bottom, black 60%, transparent);
}

/* Field sizing (nuevo) */
input {
  field-sizing: content; /* textarea que crece con el contenido */
}

/* Color scheme (nuevo) */
:root {
  color-scheme: light dark;
}

/* not-* variant */
button:not(:disabled) {
  /* Solo aplicar a botones habilitados */
}

/* @starting-style para animaciones de entrada */
.modal {
  @starting-style {
    opacity: 0;
    transform: scale(0.95);
  }
}
```

---


## Parte VII: Frontend Security — Defense in Depth

> **CVE Crítico 2026 a tener en cuenta:** CVE-2026-44578 (SSRF en Next.js < 15.5.16 y < 16.2.5). Actualizar a **Next.js 16.2.5+** o **15.5.16+**.

### 31. Headers de Seguridad Completos (Next 16)

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Generar nonce único por request
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64');

  // CSP estricto (con nonce, no unsafe-inline)
  const csp = [
    `default-src 'self'`,
    `script-src 'self' 'nonce-${nonce}' 'strict-dynamic' https://www.google-analytics.com`,
    `style-src 'self' 'unsafe-hashes' 'sha256-...' https://fonts.googleapis.com`,
    `img-src 'self' data: https: blob:`,
    `font-src 'self' https://fonts.gstatic.com`,
    `connect-src 'self' https://api.example.com wss:`,
    `media-src 'self'`,
    `object-src 'none'`,
    `frame-ancestors 'none'`,
    `base-uri 'self'`,
    `form-action 'self'`,
    `manifest-src 'self'`,
    `worker-src 'self' blob:`,
    `upgrade-insecure-requests`,
    // Report-URI para monitorear violaciones
    `report-uri /api/csp-report`,
    `report-to csp-endpoint`,
  ].join('; ');

  response.headers.set('Content-Security-Policy', csp);
  response.headers.set('x-nonce', nonce);

  // Headers adicionales
  const securityHeaders = {
    // Prevenir MIME sniffing
    'X-Content-Type-Options': 'nosniff',

    // Prevenir clickjacking
    'X-Frame-Options': 'DENY',

    // XSS filter legacy
    'X-XSS-Protection': '1; mode=block',

    // Control de referrer
    'Referrer-Policy': 'strict-origin-when-cross-origin',

    // Permissions Policy
    'Permissions-Policy': 'camera=(), microphone=(), geolocation=(), payment=(), usb=(), magnetometer=(), gyroscope=(), accelerometer=()',

    // HSTS — activar DESPUÉS de verificar todo funciona
    // 'Strict-Transport-Security': 'max-age=63072000; includeSubDomains; preload',

    // Cross-Origin policies
    'Cross-Origin-Opener-Policy': 'same-origin',
    'Cross-Origin-Resource-Policy': 'same-origin',
    'Cross-Origin-Embedder-Policy': 'require-corp',

    // Report-To endpoint
    'Report-To': '{"group":"csp-endpoint","max_age":10886400,"endpoints":[{"url":"/api/csp-report"}]}',
  };

  Object.entries(securityHeaders).forEach(([key, value]) => {
    response.headers.set(key, value);
  });

  return response;
}

export const config = {
  matcher: [
    // Aplicar a todas las rutas excepto assets estáticos
    '/((?!_next/static|_next/image|favicon.ico|robots.txt|sitemap.xml).*)',
  ],
};
```

### 32. CSP Reporting

```typescript
// app/api/csp-report/route.ts
export async function POST(request: NextRequest) {
  const report = await request.json();

  // Loggear violaciones
  console.warn('[CSP Violation]', {
    'blocked-uri': report['blocked-uri'],
    'document-uri': report['document-uri'],
    'violated-directive': report['violated-directive'],
    'original-policy': report['original-policy'],
    timestamp: new Date().toISOString(),
  });

  // En producción, enviar a Sentry/Datadog
  if (process.env.NODE_ENV === 'production') {
    await fetch(process.env.SECURITY_WEBHOOK!, {
      method: 'POST',
      body: JSON.stringify(report),
    });
  }

  return new Response(null, { status: 204 });
}
```

### 33. XSS Prevention

```typescript
// lib/sanitize.ts
import DOMPurify from 'isomorphic-dompurify'; // Funciona server + client

const ALLOWED_TAGS = [
  'p', 'br', 'strong', 'em', 'u', 's', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
  'ul', 'ol', 'li', 'a', 'blockquote', 'code', 'pre', 'img', 'figure',
  'figcaption', 'table', 'thead', 'tbody', 'tr', 'th', 'td',
];

const ALLOWED_ATTR = ['href', 'title', 'alt', 'src', 'class', 'target', 'rel'];

export function sanitizeUserInput(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS,
    ALLOWED_ATTR,
    ALLOW_DATA_ATTR: false,
    FORBID_TAGS: ['style', 'script', 'iframe', 'form', 'input', 'object', 'embed'],
    FORBID_ATTR: ['style', 'onerror', 'onload', 'onclick', 'onmouseover'],
    // Sanitizar URLs (bloquear javascript:, data: excepto imágenes, etc.)
    ALLOWED_URI_REGEXP: /^(?:(?:https?|mailto):|[^a-z]|[a-z+.\-]+(?:[^a-z+.\-:]|$))/i,
  });
}
```

```tsx
// components/safe-content.tsx
import { sanitizeUserInput } from '@/lib/sanitize';

// ❌ PELIGROSO
function Dangerous({ html }: { html: string }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// ✅ SEGURO: sanitizar primero
export function SafeContent({ html }: { html: string }) {
  const sanitized = sanitizeUserInput(html);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

// ✅ MEJOR: usar texto plano cuando sea posible
export function UserComment({ text }: { text: string }) {
  return <p>{text}</p>; // React escapa automáticamente
}

// ✅ Link con whitelist de protocolos
export function SafeLink({ href, children }: { href: string; children: React.ReactNode }) {
  let safeHref = href;
  try {
    const url = new URL(href, 'https://example.com');
    const allowedProtocols = ['https:', 'http:', 'mailto:', 'tel:'];
    if (allowedProtocols.includes(url.protocol)) {
      safeHref = url.href;
    } else {
      return <span>{children}</span>; // Renderizar como texto
    }
  } catch {
    return <span>{children}</span>;
  }

  return (
    <a
      href={safeHref}
      target="_blank"
      rel="noopener noreferrer nofollow"
    >
      {children}
    </a>
  );
}
```

### 34. CSRF Protection (Next 16 + Server Actions)

```typescript
// lib/csrf.ts
import { cookies } from 'next/headers';
import { randomBytes, timingSafeEqual } from 'crypto';

const CSRF_COOKIE = 'csrf-token';
const CSRF_HEADER = 'x-csrf-token';

export async function generateCSRFToken(): Promise<string> {
  const token = randomBytes(32).toString('hex');
  const cookieStore = await cookies();

  cookieStore.set(CSRF_COOKIE, token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    path: '/',
    maxAge: 60 * 60 * 24, // 24 horas
  });

  return token;
}

export async function validateCSRFToken(token: string | null): Promise<boolean> {
  if (!token) return false;

  const cookieStore = await cookies();
  const storedToken = cookieStore.get(CSRF_COOKIE)?.value;

  if (!storedToken) return false;

  return timingSafeEqual(
    Buffer.from(token),
    Buffer.from(storedToken)
  );
}
```

```typescript
// app/actions.ts
'use server';

import { headers } from 'next/headers';
import { validateCSRFToken } from '@/lib/csrf';

export async function sensitiveAction(formData: FormData) {
  // Validar CSRF en acciones sensibles
  const csrfToken = formData.get('_csrf') as string;
  if (!(await validateCSRFToken(csrfToken))) {
    throw new Error('CSRF token inválido');
  }

  // Validar Origin (defensa adicional)
  const headersList = await headers();
  const origin = headersList.get('origin');
  const host = headersList.get('host');

  if (origin && new URL(origin).host !== host) {
    throw new Error('Origin inválido');
  }

  // ... lógica de la acción
}
```

```tsx
// components/csrf-form.tsx
'use client';

import { useEffect, useState } from 'react';

export function CSRFForm() {
  const [token, setToken] = useState('');

  useEffect(() => {
    fetch('/api/csrf')
      .then(r => r.json())
      .then(d => setToken(d.token));
  }, []);

  return (
    <form action="/api/submit" method="POST">
      <input type="hidden" name="_csrf" value={token} />
      <input name="email" type="email" required />
      <button type="submit">Enviar</button>
    </form>
  );
}
```

### 35. Authentication Segura en 2026

```typescript
// lib/auth/session.ts
import { cookies } from 'next/headers';
import { SignJWT, jwtVerify } from 'jose';
import { randomBytes, createHash } from 'crypto';

const SECRET = new TextEncoder().encode(process.env.AUTH_SECRET!);
const ALG = 'ES256'; // ECDSA — más rápido que RSA

export interface Session {
  userId: string;
  role: 'user' | 'admin';
  // NO incluir info sensible en el JWT
}

export async function createSession(userId: string, role: Session['role']) {
  const sessionId = randomBytes(32).toString('hex');
  const token = await new SignJWT({ userId, role })
    .setProtectedHeader({ alg: ALG })
    .setIssuedAt()
    .setExpirationTime('15m') // Access token: 15 min
    .sign(SECRET);

  // Refresh token hasheado en DB
  const refreshTokenHash = createHash('sha256')
    .update(randomBytes(64).toString('hex'))
    .digest('hex');

  await db.sessions.create({
    data: {
      id: sessionId,
      userId,
      refreshTokenHash,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 días
      userAgent: (await headers()).get('user-agent') ?? '',
      ipAddress: (await headers()).get('x-forwarded-for') ?? '',
    },
  });

  const cookieStore = await cookies();
  cookieStore.set('session', sessionId, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    path: '/',
    maxAge: 15 * 60, // 15 min (access token)
  });

  cookieStore.set('refresh', refreshTokenHash, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    path: '/api/auth/refresh',
    maxAge: 7 * 24 * 60 * 60, // 7 días
  });

  return token;
}

export async function verifySession(token: string): Promise<Session | null> {
  try {
    const { payload } = await jwtVerify(token, SECRET);
    return {
      userId: payload.userId as string,
      role: payload.role as Session['role'],
    };
  } catch {
    return null;
  }
}
```

### 36. Subresource Integrity (SRI) y Trusted Types

```tsx
// next.config.js — SRI automático para scripts
const nextConfig = {
  experimental: {
    sri: {
      algorithm: 'sha384', // Algoritmo hash
    },
  },
};
```

```html
<!-- SRI manual para scripts externos -->
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9qRrYnlHk8bN+f3TLGJwGlbWY1T1iUwGIuU3jD"
  crossorigin="anonymous"
></script>
```

```typescript
// Trusted Types (defensa contra DOM XSS)
// Habilitar en CSP: require-trusted-types-for 'script'

// app/policy.ts (Trusted Types policy)
if (typeof window !== 'undefined' && window.trustedTypes) {
  window.trustedTypes.createPolicy('app', {
    createHTML: (string) => DOMPurify.sanitize(string),
    createScriptURL: (string) => {
      const url = new URL(string, location.origin);
      if (url.origin === location.origin) return url.href;
      throw new Error('Script URL no permitida');
    },
    createScript: (string) => string,
  });
}
```

### 37. Dependency Scanning Automatizado

```yaml
# .github/workflows/security.yml
name: Security Audit

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'  # Diario

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run npm audit
        run: pnpm audit --audit-level=high

      - name: Run Snyk
        uses: snyk/actions@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

      - name: Run Trivy (filesystem)
        uses: aquasecurity/trivy-action@master
        with:
          scanType: 'fs'
          severity: 'CRITICAL,HIGH'

      - name: OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: 'http://localhost:3000'
```

---

## Parte VIII: SEO — The Complete Strategy

### 38. Metadata API (Next 16)

```typescript
// app/products/[id]/page.tsx
import type { Metadata } from 'next';
import { getProductSchema } from '@/lib/schema/product';

export async function generateMetadata(
  { params }: { params: Promise<{ id: string }> }
): Promise<Metadata> {
  const { id } = await params;
  const product = await getProduct(id);

  return {
    title: `${product.name} | Tienda Online`,
    description: product.shortDescription,
    keywords: product.tags,

    // Open Graph
    openGraph: {
      title: product.name,
      description: product.shortDescription,
      url: `https://example.com/products/${product.slug}`,
      siteName: 'Tienda Online',
      images: [
        {
          url: product.image,
          width: 1200,
          height: 630,
          alt: product.name,
        },
      ],
      locale: 'es_AR',
      type: 'website',
    },

    // Twitter Card
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.shortDescription,
      images: [product.image],
      creator: '@tecnoDespegue',
    },

    // Canonical
    alternates: {
      canonical: `https://example.com/products/${product.slug}`,
      languages: {
        'es-AR': `https://example.com/es-AR/products/${product.slug}`,
        'en-US': `https://example.com/en-US/products/${product.slug}`,
        'pt-BR': `https://example.com/pt-BR/products/${product.slug}`,
      },
    },

    // Robots
    robots: {
      index: true,
      follow: true,
      googleBot: {
        index: true,
        follow: true,
        'max-image-preview': 'large',
        'max-snippet': -1,
        'max-video-preview': -1,
      },
    },

    // Verification
    verification: {
      google: process.env.GOOGLE_SITE_VERIFICATION,
      yandex: process.env.YANDEX_VERIFICATION,
    },
  };
}
```

### 39. Schema.org JSON-LD (todos los tipos importantes)

```typescript
// lib/schema/product.ts
import type { Product } from '@/types';

export function getProductSchema(product: Product) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.images.map(img =>
      typeof img === 'string' ? img : img.url
    ),
    sku: product.sku,
    mpn: product.mpn,
    brand: {
      '@type': 'Brand',
      name: product.brand.name,
    },
    offers: {
      '@type': 'Offer',
      price: product.price,
      priceCurrency: product.currency,
      priceValidUntil: product.priceValidUntil,
      availability: product.inStock
        ? 'https://schema.org/InStock'
        : 'https://schema.org/OutOfStock',
      itemCondition: 'https://schema.org/NewCondition',
      seller: {
        '@type': 'Organization',
        name: 'Tienda Online',
      },
      shippingDetails: {
        '@type': 'OfferShippingDetails',
        shippingDestination: {
          '@type': 'DefinedRegion',
          addressCountry: 'AR',
        },
        deliveryTime: {
          '@type': 'ShippingDeliveryTime',
          handlingTime: {
            '@type': 'QuantitativeValue',
            minValue: 1,
            maxValue: 2,
            unitCode: 'DAY',
          },
          transitTime: {
            '@type': 'QuantitativeValue',
            minValue: 3,
            maxValue: 7,
            unitCode: 'DAY',
          },
        },
      },
    },
    aggregateRating: product.reviewsCount > 0 ? {
      '@type': 'AggregateRating',
      ratingValue: product.averageRating,
      reviewCount: product.reviewsCount,
      bestRating: 5,
      worstRating: 1,
    } : undefined,
  };
}
```

```typescript
// lib/schema/faq.ts
export function getFAQSchema(faqs: Array<{ q: string; a: string }>) {
  return {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map(faq => ({
      '@type': 'Question',
      name: faq.q,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.a,
        // Para respuestas con código o HTML
        ...(faq.aHtml && { text: faq.aHtml }),
      },
    })),
  };
}
```

```typescript
// lib/schema/howto.ts
export function getHowToSchema(steps: Array<{
  title: string;
  description: string;
  image?: string;
  duration?: string; // ISO 8601: PT5M
}>) {
  return {
    '@context': 'https://schema.org',
    '@type': 'HowTo',
    name: 'Cómo hacer X',
    description: 'Guía paso a paso',
    totalTime: 'PT30M',
    estimatedCost: {
      '@type': 'MonetaryAmount',
      currency: 'USD',
      value: '0',
    },
    step: steps.map((step, index) => ({
      '@type': 'HowToStep',
      position: index + 1,
      name: step.title,
      text: step.description,
      ...(step.image && { image: step.image }),
      ...(step.duration && {
        video: {
          // Si tenés video
          '@type': 'VideoObject',
          name: step.title,
          thumbnailUrl: step.image,
          uploadDate: '2026-01-01T08:00:00-03:00',
          duration: step.duration,
        },
      }),
    })),
  };
}
```

```typescript
// lib/schema/breadcrumb.ts
export function getBreadcrumbSchema(items: Array<{ name: string; url: string }>) {
  return {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: item.name,
      item: `https://example.com${item.url}`,
    })),
  };
}
```

```typescript
// lib/schema/article.ts
export function getArticleSchema(article: Article) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: article.title,
    description: article.excerpt,
    image: [article.featuredImage, ...article.galleryImages],
    datePublished: article.publishedAt,
    dateModified: article.updatedAt,
    author: {
      '@type': 'Person',
      name: article.author.name,
      url: `https://example.com/author/${article.author.slug}`,
      sameAs: article.author.socialLinks,
    },
    publisher: {
      '@type': 'Organization',
      name: 'TecnoDespegue',
      logo: {
        '@type': 'ImageObject',
        url: 'https://example.com/logo.png',
        width: 600,
        height: 60,
      },
    },
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': `https://example.com/blog/${article.slug}`,
    },
    articleSection: article.category,
    keywords: article.tags.join(', '),
    wordCount: article.wordCount,
    inLanguage: 'es-AR',
  };
}
```

```typescript
// lib/schema/organization.ts
export function getOrganizationSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'TecnoDespegue',
    url: 'https://example.com',
    logo: 'https://example.com/logo.png',
    description: 'Soluciones de desarrollo de software enterprise',
    foundingDate: '2020-01-01',
    sameAs: [
      'https://github.com/Rene-Kuhm',
      'https://twitter.com/tecnoDespegue',
      'https://linkedin.com/company/tecnoDespegue',
    ],
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+54-11-1234-5678',
      contactType: 'customer service',
      areaServed: ['AR', 'US', 'ES'],
      availableLanguage: ['Spanish', 'English', 'Portuguese'],
    },
  };
}
```

```tsx
// components/json-ld.tsx
interface JsonLdProps {
  data: object | object[];
}

export function JsonLd({ data }: JsonLdProps) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data),
      }}
    />
  );
}

// Uso
export default function ProductPage({ product }: { product: Product }) {
  return (
    <>
      <JsonLd
        data={[
          getProductSchema(product),
          getBreadcrumbSchema([
            { name: 'Home', url: '/' },
            { name: 'Productos', url: '/products' },
            { name: product.name, url: `/products/${product.slug}` },
          ]),
        ]}
      />
      {/* ... resto de la página */}
    </>
  );
}
```

### 40. Sitemap Dinámico (Next 16)

```typescript
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = process.env.NEXT_PUBLIC_SITE_URL!;

  // Fetch en paralelo de todas las fuentes
  const [products, posts, categories, staticPages] = await Promise.all([
    db.products.findMany({
      where: { published: true },
      select: { slug: true, updatedAt: true, images: { select: { url: true } } },
    }),
    db.posts.findMany({
      where: { published: true },
      select: { slug: true, updatedAt: true, category: true },
    }),
    db.categories.findMany({ select: { slug: true } }),
    Promise.resolve([
      { path: '/', priority: 1.0, freq: 'daily' as const },
      { path: '/products', priority: 0.9, freq: 'daily' as const },
      { path: '/blog', priority: 0.8, freq: 'daily' as const },
      { path: '/about', priority: 0.5, freq: 'monthly' as const },
      { path: '/contact', priority: 0.5, freq: 'monthly' as const },
    ]),
  ]);

  // Sitemap index con múltiples sub-sitemaps (recomendado > 50k URLs)
  return [
    {
      url: `${baseUrl}/sitemap.xml`,
      lastModified: new Date(),
    },
  ];
}

// Para sitios grandes, generar sub-sitemaps
// app/products/sitemap.ts
export default async function productsSitemap(): Promise<MetadataRoute.Sitemap> {
  const products = await db.products.findMany({
    where: { published: true },
    select: { slug: true, updatedAt: true },
  });

  return products.map(p => ({
    url: `${process.env.NEXT_PUBLIC_SITE_URL}/products/${p.slug}`,
    lastModified: p.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.7,
    // Image sitemap (Google)
    images: [`${process.env.NEXT_PUBLIC_SITE_URL}/products/${p.slug}/image.jpg`],
  }));
}
```

### 41. robots.txt

```typescript
// app/robots.ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  const baseUrl = process.env.NEXT_PUBLIC_SITE_URL!;

  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: [
          '/api/',
          '/admin/',
          '/private/',
          '/_next/',
          '/checkout/',
          '/account/',
        ],
      },
      {
        // Googlebot-Image: permite imágenes
        userAgent: 'Googlebot-Image',
        allow: '/images/',
        disallow: ['/private-images/'],
      },
      {
        // Bloquear scrapers conocidos
        userAgent: ['AhrefsBot', 'SemrushBot', 'MJ12bot', 'DotBot'],
        disallow: '/',
      },
    ],
    sitemap: [
      `${baseUrl}/sitemap.xml`,
      `${baseUrl}/products/sitemap.xml`,
      `${baseUrl}/blog/sitemap.xml`,
    ],
    host: baseUrl,
  };
}
```

### 42. hreflang para i18n

```tsx
// app/[locale]/layout.tsx
export async function generateStaticParams() {
  return [
    { locale: 'es-AR' },
    { locale: 'en-US' },
    { locale: 'pt-BR' },
  ];
}

export default async function LocaleLayout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Promise<{ locale: string }>;
}) {
  const { locale } = await params;
  const baseUrl = process.env.NEXT_PUBLIC_SITE_URL!;
  const path = (await headers()).get('x-pathname') ?? '';

  return (
    <html lang={locale}>
      <head>
        {/* Self-referencing */}
        <link rel="alternate" hrefLang={locale} href={`${baseUrl}/${locale}${path}`} />
        {/* x-default para selector de idioma */}
        <link rel="alternate" hrefLang="x-default" href={`${baseUrl}/es-AR${path}`} />
        {/* Todas las variantes */}
        <link rel="alternate" hrefLang="es-AR" href={`${baseUrl}/es-AR${path}`} />
        <link rel="alternate" hrefLang="en-US" href={`${baseUrl}/en-US${path}`} />
        <link rel="alternate" hrefLang="pt-BR" href={`${baseUrl}/pt-BR${path}`} />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### 43. Lighthouse CI en Pipeline

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Build app
        run: |
          pnpm install
          pnpm build
          pnpm start &
          sleep 10

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          urls: |
            http://localhost:3000/
            http://localhost:3000/products
            http://localhost:3000/blog
          budgetPath: ./lighthouse-budget.json
          uploadArtifacts: true
```

```json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "resourceSizes": [
      { "resourceType": "script", "budget": 200 },
      { "resourceType": "image", "budget": 500 },
      { "resourceType": "total", "budget": 1000 }
    ],
    "timings": [
      { "metric": "first-contentful-paint", "budget": 1800 },
      { "metric": "largest-contentful-paint", "budget": 2500 },
      { "metric": "interactive", "budget": 3500 },
      { "metric": "cumulative-layout-shift", "budget": 0.1 },
      { "metric": "total-blocking-time", "budget": 200 }
    ],
    "scores": [
      { "metric": "performance", "score": 0.9 },
      { "metric": "accessibility", "score": 0.95 },
      { "metric": "best-practices", "score": 0.95 },
      { "metric": "seo", "score": 0.95 }
    ]
  }
]
```

---

## Parte IX: AI-Powered Frontend (Bonus)

### 44. AI-Powered Features en el Frontend

```typescript
// lib/ai/search.ts
import { openai } from '@ai-sdk/openai';
import { generateText, embed } from 'ai';

export async function semanticSearch(query: string, products: Product[]) {
  // 1. Generar embedding de la query
  const { embedding: queryEmbedding } = await embed({
    model: openai.textEmbeddingModel('text-embedding-3-small'),
    value: query,
  });

  // 2. Calcular similitud coseno con cada producto
  const results = products
    .map(p => ({
      product: p,
      score: cosineSimilarity(queryEmbedding, p.embedding),
    }))
    .sort((a, b) => b.score - a.score)
    .slice(0, 10);

  return results;
}
```

```tsx
// components/ai-search.tsx
'use client';

import { useChat } from 'ai/react';

export function AISearch() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat',
  });

  return (
    <div>
      <div className="messages">
        {messages.map(m => (
          <div key={m.id} className={`message ${m.role}`}>
            {m.content}
          </div>
        ))}
      </div>
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Buscá productos con lenguaje natural..."
          disabled={isLoading}
        />
        <button type="submit" disabled={isLoading}>
          {isLoading ? 'Pensando...' : 'Buscar'}
        </button>
      </form>
    </div>
  );
}
```

```typescript
// app/api/chat/route.ts
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'),
    messages,
    tools: {
      // Tool calling: el modelo puede buscar productos
      searchProducts: {
        description: 'Busca productos en el catálogo',
        parameters: z.object({
          query: z.string().describe('Query de búsqueda'),
          category: z.string().optional(),
        }),
        execute: async ({ query, category }) => {
          return await searchProducts(query, category);
        },
      },
    },
  });

  return result.toDataStreamResponse();
}
```

### 45. AI-Powered Personalization

```typescript
// lib/recommendations.ts
// Collaborative filtering + AI ranking
export async function getPersonalizedRecommendations(
  userId: string,
  context: { currentProductId?: string; limit?: number }
) {
  // Combinar: historial + comportamiento + similitud semántica
  const [userHistory, similarUsers, contentBased] = await Promise.all([
    getUserHistory(userId),
    findSimilarUsers(userId),
    getContentBasedRecommendations(userId, context.currentProductId),
  ]);

  // AI model para re-ranking
  const ranked = await rankWithAI({
    candidates: [...contentBased, ...similarUsers],
    userContext: { history: userHistory, demographics: await getUserDemo(userId) },
  });

  return ranked.slice(0, context.limit ?? 10);
}
```

---


## Parte X: Checklists Completos

### Checklist Next.js 16

```markdown
## Setup

- [ ] Node.js 22 LTS (mínimo 20.9)
- [ ] Next.js 16.2.5+ (CVE-2026-44578 fix)
- [ ] Turbopack default (sin flags)
- [ ] turbopackFileSystemCacheForDev habilitado
- [ ] Adapter API configurado para tu plataforma de deploy

## React 19.2

- [ ] React Compiler activado en next.config.js
- [ ] useActionState en formularios
- [ ] useOptimistic para feedback instantáneo
- [ ] use() para Promises y Context condicional
- [ ] refs como prop (no más forwardRef)
- [ ] <Context value={...}> en vez de <Context.Provider>

## Cache Components

- [ ] 'use cache' explícito en funciones caras
- [ ] cacheTag definido para invalidación quirúrgica
- [ ] cacheLife con perfil apropiado (minutes/hours/days)
- [ ] revalidateTag con segundo argumento ('infinite' o profile)
- [ ] PPR habilitado para páginas mixtas
- [ ] Auditoría: ¿hay fetches que deberían ser cacheados?
```

### Checklist Performance

```markdown
## Core Web Vitals (2026)

- [ ] LCP ≤ 2.5s (hero image con priority + fetchPriority="high")
- [ ] INP ≤ 200ms (yieldToMain, Web Workers, no Long Tasks)
- [ ] CLS ≤ 0.1 (dimensiones explícitas, size-adjust fonts)
- [ ] TTFB ≤ 800ms (edge functions, ISR cuando sea posible)
- [ ] FCP ≤ 1.8s

## Optimizaciones

- [ ] AVIF/WebP para todas las imágenes
- [ ] Preload de la LCP image y critical fonts
- [ ] Variable fonts con font-display: swap
- [ ] size-adjust para evitar CLS en font swap
- [ ] Code splitting automático (Turbopack)
- [ ] Dynamic imports para componentes pesados
- [ ] Brotli/Zstd compression habilitada
- [ ] HTTP/3 habilitado
- [ ] Web Vitals monitoreándose en producción (web-vitals lib)
- [ ] Alertas en Slack/Datadog cuando CWV es "poor"
```

### Checklist Seguridad

```markdown
## CVE-2026-44578

- [ ] Next.js actualizado a 16.2.5+ o 15.5.16+

## Headers

- [ ] CSP estricto con nonces (no unsafe-inline)
- [ ] Report-To configurado y endpoint funcional
- [ ] HSTS activado (después de verificar todo HTTPS)
- [ ] X-Frame-Options: DENY
- [ ] Permissions-Policy deshabilitando features no usadas
- [ ] Cross-Origin policies (COOP, CORP, COEP)

## XSS

- [ ] DOMPurify para todo HTML de usuario
- [ ] NO dangerouslySetInnerHTML sin sanitizar
- [ ] Whitelist de protocols en URLs (https:, mailto:, tel:)
- [ ] Trusted Types policy (donde sea posible)
- [ ] SRI para scripts externos (sha384)

## CSRF

- [ ] CSRF tokens en Server Actions sensibles
- [ ] SameSite=Strict en cookies de sesión
- [ ] Validación de Origin en acciones

## Auth

- [ ] JWT con expiración corta (15 min access, 7 días refresh)
- [ ] Refresh tokens hasheados en DB
- [ ] httpOnly + secure + sameSite en cookies
- [ ] Sesiones invalidadas en logout
- [ ] MFA para cuentas admin

## Dependencias

- [ ] pnpm audit en CI
- [ ] Snyk / Trivy semanal
- [ ] Dependabot / Renovate activo
- [ ] Lockfile commiteado
```

### Checklist SEO

```markdown
## Metadata

- [ ] Title único por página (50-60 chars)
- [ ] Meta description compelling (150-160 chars)
- [ ] Open Graph completo (image 1200x630)
- [ ] Twitter Card (summary_large_image)
- [ ] Canonical URL en cada página
- [ ] hreflang para i18n
- [ ] Robots meta apropiado (noindex en páginas privadas)

## Schema.org JSON-LD

- [ ] Organization en layout principal
- [ ] Product en cada producto
- [ ] Article/BlogPosting en posts
- [ ] FAQPage donde haya FAQs
- [ ] HowTo en tutoriales
- [ ] BreadcrumbList en páginas con jerarquía
- [ ] LocalBusiness si es negocio local
- [ ] Validar con Google Rich Results Test
- [ ] Validar con Schema.org Validator

## Sitemap & Robots

- [ ] sitemap.xml dinámico
- [ ] Sitemap index si > 50k URLs
- [ ] Sub-sitemaps por sección
- [ ] robots.txt con reglas correctas
- [ ] Bloquear scrapers (AhrefsBot, SemrushBot)
- [ ] Image sitemaps

## Performance SEO

- [ ] CWV "good" en todas las páginas
- [ ] Mobile-friendly (responsive)
- [ ] No render-blocking resources
- [ ] Lighthouse score > 90 en Performance, SEO, A11y
- [ ] HTTPS everywhere
```

### Checklist de Migración Next 15 → Next 16

```markdown
## Cambios Breaking

- [ ] params y searchParams ahora son PROMISES (agregar await)
- [ ] cookies() y headers() ahora son async (agregar await)
- [ ] dynamicParams default cambió a `false` (revisar rutas dinámicas)
- [ ] Revalidar uso de useContext (puede ser use() ahora)
- [ ] Actualizar a 16.2.5+ por CVE-2026-44578

## Oportunidades

- [ ] Habilitar React Compiler (eliminar useMemo/useCallback innecesarios)
- [ ] Migrar a 'use cache' explícito
- [ ] Configurar Adapter API para tu plataforma
- [ ] Adoptar useOptimistic en formularios
- [ ] Adoptar useActionState
- [ ] Probar Cache Components en rutas con data fetching pesado
```

---

## Recursos y Referencias

### Documentación oficial
- [Next.js 16 Docs](https://nextjs.org/docs)
- [React 19 Docs](https://react.dev)
- [React Compiler](https://react.dev/learn/react-compiler)
- [Tailwind CSS v4](https://tailwindcss.com/docs)
- [TanStack Query](https://tanstack.com/query/latest)
- [Zustand](https://zustand.docs.pmnd.rs/)

### Estándares y Guías
- [web.dev — Core Web Vitals](https://web.dev/vitals/)
- [Google Search Central](https://developers.google.com/search/docs)
- [OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)
- [Schema.org](https://schema.org/)
- [MDN Web Docs](https://developer.mozilla.org/)
- [W3C Web Standards](https://www.w3.org/standards/)

### Performance & Security
- [Snyk Vulnerability DB](https://snyk.io/vuln)
- [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci)
- [Mozilla Observatory](https://observatory.mozilla.org/)
- [Security Headers](https://securityheaders.com/)

### Comunidad (2026)
- [r/nextjs](https://reddit.com/r/nextjs)
- [r/reactjs](https://reddit.com/r/reactjs)
- [Next.js Discord](https://nextjs.org/discord)
- [Reactiflux Discord](https://www.reactiflux.com/)

---

## Apéndice: Stack Recomendado por Tipo de Proyecto

### E-commerce de alta performance
```
Next.js 16.2+ (App Router) + Turbopack + Cache Components
React 19.2 + React Compiler + Server Actions
Tailwind 4.1 + Radix UI
TanStack Query + Zustand
PostgreSQL + Prisma + Edge cache (Vercel KV / Cloudflare KV)
Stripe para pagos
Sentry + Datadog para observabilidad
```

### SaaS B2B Dashboard
```
Next.js 16.2+ + React Compiler
React 19.2 + Server Components
TanStack Table + TanStack Query + Zustand
shadcn/ui + Tailwind 4.1
NextAuth.js (Auth.js) + RBAC
PostgreSQL + Drizzle ORM + Redis
Posthog + Sentry
```

### Blog / Content site
```
Astro 6+ o Next.js 16.2+ (Static Export)
MDX para contenido
Tailwind 4.1
Schema.org JSON-LD completo
Cloudflare CDN
Plausible o Umami analytics
```

### AI-powered app
```
Next.js 16.2+ + React Compiler
Vercel AI SDK + OpenAI / Anthropic
TanStack Query para data fetching
Edge functions para baja latencia
Streaming responses con AI SDK
Vector DB (Pinecone / pgvector)
```

---

## Conclusión

El stack frontend de 2026 se caracteriza por:

1. **Caching explícito** — `'use cache'` reemplazó el caching implícito
2. **Auto-optimización** — React Compiler elimina la necesidad de memoización manual
3. **Edge-first** — Adapter API permite deploy en cualquier plataforma
4. **Turbopack everywhere** — Bundler default en dev y prod
5. **Server-first** — Server Components + Server Actions + Streaming SSR
6. **AI-native** — Streaming AI responses, semantic search, AI-powered personalization
7. **Performance by default** — Core Web Vitals como factor SEO más fuerte
8. **Security first** — CSP estricta, Trusted Types, SRI por defecto

La clave: **adoptar las nuevas APIs de a poco**, empezando por React Compiler y `'use cache'`, que dan los mejores resultados con menos esfuerzo.

---

**Mantenedor:** TecnoDespegue
**Última actualización:** Junio 2026
**Versión:** 3.0
**Licencia:** MIT

> Si encontrás algo desactualizado, abrí un PR en [github.com/Rene-Kuhm/enterprise-dev-system](https://github.com/Rene-Kuhm/enterprise-dev-system)
