# CLAUDE.md — Reglas del Proyecto

Estas reglas son **obligatorias** en todo el proyecto. No hay excepciones.

---

## Stack obligatorio

- **Next.js 16** con App Router
- **TypeScript** en modo estricto (`strict: true`)
- **Zod** para toda validación de datos
- **ZSA** (`zsa`) para todas las Server Actions

---

## TypeScript — Reglas estrictas

```json
// tsconfig.json debe tener siempre:
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

- **Prohibido** usar `any`. Usar `unknown` y tipar correctamente.
- **Prohibido** usar `as TipoX` para castear sin validación previa.
- **Prohibido** ignorar errores con `// @ts-ignore` o `// @ts-expect-error` sin comentario justificado.
- Los tipos de dominio **siempre** se infieren desde schemas Zod:
  ```ts
  // ✅ Correcto
  export type User = z.infer<typeof userSchema>

  // ❌ Prohibido
  export type User = { id: string; name: string }
  ```

---

## Zod — Uso obligatorio

- **Todo input de Server Action** debe tener un schema Zod en `/validations`.
- **Todo dato externo** (API, DB, env vars) debe pasar por `schema.parse()` o `schema.safeParse()`.
- Los schemas viven en `/validations`, nunca inline en components o actions.
- Nombrar schemas con sufijo `Schema`: `userSchema`, `projectSchema`.

```ts
// ✅ Correcto
import { userCreateSchema } from '@/validations/user.schema'

// ❌ Prohibido — validación inline
const parsed = z.object({ name: z.string() }).parse(data)
```

---

## Server Actions — ZSA obligatorio

**Todas** las Server Actions usan `createServerAction` de `zsa`:

```ts
// ✅ Patrón correcto
'use server'
import { createServerAction } from 'zsa'
import { userCreateSchema } from '@/validations/user.schema'

export const createUserAction = createServerAction()
  .input(userCreateSchema)
  .handler(async ({ input }) => {
    // input ya está tipado y validado
    const user = await db.user.create({ data: input })
    return user
  })
```

- **Prohibido** crear Server Actions sin ZSA (`'use server'` solo, sin `createServerAction`).
- El `handler` recibe `input` ya validado — no re-validar dentro del handler.
- Los errores se propagan con `throw new Error()` o retornando el error estructurado de ZSA.

---

## Estructura de archivos

```
/app             → Solo routing, layouts, pages
/actions         → Server Actions (ZSA). Un archivo por dominio.
/lib/supabase    → Clientes Supabase (client, server, middleware)
/validations     → Schemas Zod. Un archivo por dominio.
/components/ui   → Componentes atómicos sin lógica de negocio
/components/shared → Componentes compuestos reutilizables
/types           → Tipos inferidos de schemas Zod
/hooks           → Custom hooks de React (solo cliente)
```

### Reglas de importación

- `/app` puede importar desde `/components`, `/actions`, `/types`
- `/actions` puede importar desde `/lib/supabase`, `/validations`, `/types`
- `/components` **NO** importa desde `/lib/supabase` directamente
- `/lib/supabase` no importa desde `/actions` ni `/components`

---

## Next.js 16 — Reglas

- **App Router siempre**. No usar Pages Router.
- Los Server Components son el default. Agregar `'use client'` solo cuando sea necesario.
- Los datos se fetchean en Server Components o Server Actions, nunca con `useEffect` + `fetch` del cliente.
- Las rutas dinámicas usan `generateStaticParams` cuando sea posible.

---

## Supabase — Reglas

- Usar el cliente correcto según el contexto:
  - Server Components/Actions → `lib/supabase/server.ts`
  - Client Components → `lib/supabase/client.ts`
  - Middleware → `lib/supabase/middleware.ts`
- **Nunca** usar la service key en código de cliente.
- Las queries reutilizables van en `lib/supabase/`, no inline en components.

---

## Lo que Claude NO debe hacer

- Generar tipos TypeScript manuales si existe un schema Zod equivalente
- Crear Server Actions sin ZSA
- Usar `fetch` en `useEffect` para obtener datos de Supabase
- Ignorar errores de TypeScript
- Agregar dependencias no aprobadas sin preguntar
- Crear archivos fuera de la estructura definida sin justificación
