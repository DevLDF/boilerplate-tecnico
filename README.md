# Boilerplate Técnico — Consultora

Chasis base para todos los proyectos Next.js de la consultora. Cada proyecto nuevo arranca desde aquí.

---

## Stack

- **Next.js 16** — App Router
- **TypeScript** — modo estricto, sin excepciones
- **Supabase** — base de datos y autenticación
- **Zod** — validación de esquemas en toda la app
- **ZSA (zsa)** — Server Actions tipadas y seguras

---

## Flujo de trabajo

```
Request HTTP
    │
    ▼
app/          → Routing, layouts, pages (solo UI y composición)
    │
    ▼
actions/      → Server Actions con ZSA (lógica de negocio + mutaciones)
    │
    ▼
lib/supabase/ → Cliente Supabase, queries, helpers de DB
    │
    ▼
validations/  → Schemas Zod compartidos (input/output de actions)
```

---

## Estructura de carpetas

### `/app`
Routing de Next.js App Router. Contiene únicamente `page.tsx`, `layout.tsx` y `loading.tsx`. **Sin lógica de negocio.** Cada page delega a components y llama Server Actions.

### `/actions`
Server Actions organizadas por dominio (ej: `user.actions.ts`, `project.actions.ts`). Todas las actions usan **ZSA** con el input validado por un schema Zod de `/validations`. Nunca exponer datos sin validar.

### `/lib/supabase`
- `client.ts` → cliente para componentes de cliente (browser)
- `server.ts` → cliente para Server Components y Server Actions
- `middleware.ts` → cliente para middleware de Next.js
- Queries reutilizables organizadas por entidad

### `/validations`
Schemas Zod para validar inputs de Server Actions y formularios. Un archivo por dominio (ej: `user.schema.ts`). Son la **fuente de verdad** de los tipos en runtime.

### `/components/ui`
Componentes atómicos de UI: botones, inputs, modales. Sin lógica de negocio. Pueden ser de shadcn/ui o propios.

### `/components/shared`
Componentes compuestos reutilizables entre features (ej: `DataTable`, `PageHeader`, `UserAvatar`). Pueden usar hooks y llamar Server Actions.

### `/types`
Tipos TypeScript globales y derivados de schemas Zod:
```ts
export type User = z.infer<typeof userSchema>
```
Nunca definir tipos manualmente si existe un schema Zod equivalente.

### `/hooks`
Custom hooks de React para lógica de UI reutilizable (ej: `useDebounce`, `useLocalStorage`). Solo lógica de cliente, sin llamadas directas a DB.

---

## Convenciones

- Cada Server Action **debe** tener input validado con Zod + ZSA
- Los tipos se infieren desde schemas, nunca se duplican
- Un componente de `/app` no importa directamente desde `/lib/supabase`
- Los errores de actions se manejan con el sistema de errores de ZSA
