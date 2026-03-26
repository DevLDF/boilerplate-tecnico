# ⚡ LF Software Studio - El Chasis de Hierro (Master Mold)

Este repositorio es el **núcleo técnico** de nuestra consultora. No es un proyecto final, es la infraestructura estandarizada que nos permite construir, desplegar y escalar software para clientes (Inmobiliarias, locales de ropa, abogados) en tiempo récord usando Inteligencia Artificial.

## 🎯 Nuestra Visión (El Negocio)
Operamos bajo un modelo de **Alta Velocidad y MRR (Ingresos Recurrentes)**.
- **Velocidad:** Usamos este molde para que la IA (Claude Code) no pierda tiempo configurando cañerías y se enfoque en la lógica del negocio del cliente.
- **Calidad de Hierro:** Cada proyecto que sale de aquí es seguro, tipado y escalable.
- **Aislamiento:** Cada cliente tendrá su propio repositorio clonado de este molde y su propia instancia de Supabase.

---

## 🏗️ Arquitectura de Hierro (Stack 2026)
Para que la IA no alucine y el código sea indestructible, estandarizamos:

- **Framework:** Next.js 16 (App Router) - El estándar de rendimiento.
- **Base de Datos:** Supabase (PostgreSQL) - Seguridad a nivel de fila (RLS).
- **Comunicación:** Server Actions + **ZSA** - Acciones de servidor estrictamente tipadas.
- **Validación:** **Zod** - El "Contrato" de datos. Si no pasa Zod, no entra a la base de datos.
- **UI:** **shadcn/ui** + Tailwind CSS - Componentes consistentes que la IA conoce de memoria.
- **TypeScript** — modo estricto, sin excepciones.

---

## 📁 Anatomía del Repositorio (Mapa para el Ingeniero)

| Carpeta | Contenido | Razón de Ser |
| :--- | :--- | :--- |
| **`/app`** | Rutas y Vistas | Estructura de navegación del cliente. |
| **`/actions`** | Lógica de Negocio | Aquí vive el "Cerebro". Nada de lógica dentro de los componentes. |
| **`/validations`** | Esquemas Zod | Los contratos de datos. Evitan que entre "basura" a la DB. |
| **`/lib/supabase`** | Infraestructura | Clientes de conexión. Se configuran una vez y no se tocan. |
| **`/components/ui`** | Átomos de Diseño | Botones, inputs y tablas base de shadcn. |
| **`/components/forms`** | Módulos de Entrada | Formularios complejos generados a partir de Zod. |
| **`/types`** | Tipado Global | Diccionario de TypeScript para evitar errores de compilación. |
| **`/hooks`** | Custom Hooks | Lógica de UI reutilizable. Solo cliente, sin acceso directo a DB. |
| **`/components/shared`** | Componentes Compuestos | Reutilizables entre features. Pueden llamar Server Actions. |

---

## 🔄 Flujo de Datos

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

## 🤖 Workflow con Claude Code
Para mantener la integridad, toda instrucción a la IA debe seguir estas reglas:
1. **Leer `CLAUDE.md`** antes de cualquier tarea.
2. **Validación obligatoria:** Antes de crear una tabla o un form, definir el esquema en `/validations`.
3. **Server-First:** No usar `'use client'` a menos que sea estrictamente necesario para interactividad.

---

## 📐 Convenciones de código

- Cada Server Action **debe** tener input validado con Zod + ZSA.
- Los tipos se infieren desde schemas Zod, nunca se duplican a mano:
  ```ts
  export type User = z.infer<typeof userSchema>
  ```
- Un componente de `/app` no importa directamente desde `/lib/supabase`.
- Los errores de actions se manejan con el sistema de errores de ZSA.

---

## 🛠️ Setup para el Socio (Lauti)
1. Clonar el repositorio.
2. Copiar `.env.example` a `.env.local` y pedir las keys a Francisco.
3. Ejecutar `npm install`.
4. Usar Claude Code para extender la funcionalidad según el rubro del cliente.
