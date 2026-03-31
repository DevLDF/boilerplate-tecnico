# CLAUDE.md — Reglas del Proyecto

Estas reglas son **obligatorias** en todo el proyecto. No hay excepciones.

---

## Stack obligatorio

- **Next.js 15** con App Router (versión 15.5.14)
- **TypeScript** en modo estricto (`strict: true`)
- **Zod** para toda validación de datos
- **ZSA** (`zsa`) para todas las Server Actions
- **Supabase** (PostgreSQL) con Row Level Security siempre habilitado
- **shadcn/ui + Tailwind CSS** para UI
- **Vercel** para deploy

---

## Arquitectura de capas — Regla fundamental

### 🔴 CORE — Nunca modificar
Idénticos en todos los proyectos. Cambiarlos rompe la propagación de updates.
- `/lib/supabase/**` — clientes estandarizados de DB
- `/middleware.ts` — auth refresh automático
- `/components/ui/**` — shadcn/ui intocable
- `/types/index.ts` — tipos base compartidos

### 🟡 CONFIG — Solo estos archivos cambian por cliente
- `/config/site.ts` — nombre, logo, colores, dominio
- `/config/features.ts` — qué módulos están activos

### 🟢 EXTENSIÓN — Agregar libremente
- `/app/[feature]/` — nuevas rutas
- `/actions/[feature].actions.ts` — nuevas server actions
- `/validations/[feature].schema.ts` — nuevos schemas Zod
- `/components/[vertical]/` — nuevos componentes

Si necesitás modificar el CORE para que una feature funcione, la feature está mal diseñada.

---

## TypeScript — Reglas estrictas

- Prohibido usar `any` — usar `unknown` y tipar correctamente
- Prohibido castear con `as TipoX` sin validación previa
- Prohibido `// @ts-ignore` sin comentario justificado
- Los tipos de dominio siempre se infieren desde schemas Zod: `type X = z.infer<typeof xSchema>`

---

## Zod — Uso obligatorio

- Todo input de Server Action necesita un schema en `/validations`
- Todo dato externo (API, DB, env vars) debe pasar por `schema.parse()` o `schema.safeParse()`
- Los schemas viven en `/validations`, nunca inline
- Nombrar con sufijo `Schema`: `userSchema`, `contratoSchema`

---

## Server Actions — ZSA obligatorio

- Todas las Server Actions usan `createServerAction()` de `zsa`
- Prohibido crear Server Actions con `'use server'` solo, sin `createServerAction`
- El handler recibe `input` ya validado — no re-validar dentro del handler
- Los errores se propagan con `throw` o el error estructurado de ZSA

---

## Estructura de archivos

```
/app             → Solo routing, layouts, pages
/actions         → Server Actions (ZSA). Un archivo por dominio.
/config          → Configuración por cliente (site.ts, features.ts)
/lib/supabase    → Clientes Supabase — CORE
/validations     → Schemas Zod. Un archivo por dominio.
/components/ui   → shadcn/ui — CORE, no modificar
/components/shared → Componentes compuestos reutilizables
/types           → Tipos inferidos de schemas Zod
/hooks           → Custom hooks de React (solo cliente)
/migrations      → Changelog de cambios del base-template
```

### Reglas de importación

- `/app` importa desde `/components`, `/actions`, `/types`, `/config`
- `/actions` importa desde `/lib/supabase`, `/validations`, `/types`, `/config`
- `/components` NO importa desde `/lib/supabase` directamente
- `/lib/supabase` no importa desde `/actions` ni `/components`

---

## Next.js 15 — Reglas

- App Router siempre. No usar Pages Router.
- Server Components por defecto. `'use client'` solo cuando sea necesario.
- Datos en Server Components o Server Actions, nunca con `useEffect` + fetch del cliente.
- Usar `generateStaticParams` en rutas dinámicas cuando sea posible.

---

## Supabase — Reglas

- Server Components/Actions → `lib/supabase/server.ts`
- Client Components → `lib/supabase/client.ts`
- Middleware → `lib/supabase/middleware.ts`
- Nunca usar la service key en código de cliente
- Queries reutilizables en `lib/supabase/`, no inline
- Toda tabla nueva requiere RLS activado desde el primer commit

---

## Propagación de cambios del base-template

Cuando se modifica el CORE:
1. Documentar en `/migrations/YYYY-MM-DD-descripcion.md`
2. Propagar a verticales y clientes con Claude Code:
   `"Aplicá el cambio de migrations/YYYY-MM-DD-descripcion.md sin tocar customizaciones del proyecto"`

---

## Lo que Claude NO debe hacer

- Generar tipos TypeScript manuales si existe un schema Zod equivalente
- Crear Server Actions sin ZSA
- Usar `fetch` en `useEffect` para datos de Supabase
- Ignorar errores de TypeScript
- Agregar dependencias sin preguntar
- Modificar archivos del CORE
- Crear archivos fuera de la estructura definida sin justificación
- Hardcodear el nombre del cliente — siempre usar `siteConfig.name`
