# Guía de Trabajo — DevLDF

Manual operativo para Lautaro y Francisco. Seguir este documento en cada proyecto, siempre.

---

## Índice

1. [Crear un proyecto nuevo](#1-crear-un-proyecto-nuevo)
2. [Qué es un branch y por qué importa](#2-qué-es-un-branch-y-por-qué-importa)
3. [Flujo diario de trabajo](#3-flujo-diario-de-trabajo)
4. [Cómo hacer buenos commits](#4-cómo-hacer-buenos-commits)
5. [Pull Requests paso a paso](#5-pull-requests-paso-a-paso)
6. [Trabajar en paralelo sin conflictos](#6-trabajar-en-paralelo-sin-conflictos)
7. [Reglas de oro](#7-reglas-de-oro)

---

## 1. Crear un proyecto nuevo

Seguir este checklist en orden. No saltear pasos.

### Paso 1 — Crear el repo desde el vertical correspondiente

```bash
# En la terminal, con gh instalado:
gh repo create DevLDF/cliente-[nombre] --template DevLDF/vertical-[rubro] --private
```

Ejemplos:
```bash
gh repo create DevLDF/cliente-rodriguez --template DevLDF/vertical-inmobiliaria --private
gh repo create DevLDF/cliente-laprima  --template DevLDF/vertical-restaurante   --private
```

### Paso 2 — Clonar el repo en tu máquina

```bash
git clone https://github.com/DevLDF/cliente-[nombre].git
cd cliente-[nombre]
```

### Paso 3 — Configurar el cliente (los ÚNICOS 2 archivos que cambian)

Abrir `config/site.ts` y completar:
```ts
export const siteConfig = {
  name: "Inmobiliaria Rodríguez",   // ← nombre del cliente
  description: "Gestión de contratos de alquiler",
  url: "https://rodriguez.vercel.app",
  primaryColor: "#1A3A5C",          // ← color de la marca si tienen
}
```

Abrir `config/features.ts` y activar solo los módulos que el cliente va a usar:
```ts
export const features = {
  hasContracts: true,    // ← activar
  hasProperties: true,   // ← activar
  hasTenants: false,     // ← desactivar si no lo necesitan todavía
}
```

### Paso 4 — Crear el proyecto en Supabase

1. Ir a [supabase.com](https://supabase.com) → New project
2. Nombre: `cliente-[nombre]`
3. Guardar la contraseña de la base de datos
4. Ir a **SQL Editor** → pegar y ejecutar el contenido de `supabase/schema.sql`
5. Ir a **Settings → API** → copiar las keys

### Paso 5 — Configurar variables de entorno

```bash
cp .env.example .env.local
```

Completar `.env.local` con las keys de Supabase:
```
NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...   ← NUNCA compartir ni commitear
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### Paso 6 — Instalar y verificar que corre

```bash
npm install
npm run dev
```

Abrir `http://localhost:3000` — tiene que cargar sin errores.

### Paso 7 — Proteger el branch main en GitHub

1. Ir al repo en GitHub → **Settings → Branches**
2. Click en **Add branch protection rule**
3. Branch name pattern: `main`
4. Activar:
   - ✅ Require a pull request before merging
   - ✅ Require approvals: 1
   - ✅ Do not allow bypassing the above settings
5. Guardar

### Paso 8 — Conectar a Vercel

1. Ir a [vercel.com](https://vercel.com) → New Project → Import from GitHub
2. Seleccionar `DevLDF/cliente-[nombre]`
3. En **Environment Variables** agregar las mismas del `.env.local`
4. Deploy

**✅ Proyecto listo.** Avisar al socio por Slack con el link del repo y el link de Vercel.

---

## 2. Qué es un branch y por qué importa

Un **branch** es una copia paralela del código donde podés trabajar sin afectar lo que está en producción.

```
main ──────────────────────────────────────── (producción, siempre estable)
          │
          └── feat/login ──── commits ──── PR ──── merge a main
                    │
                    └── fix/error-formulario ──── commits ──── PR ──── merge
```

**La regla es simple:** `main` es sagrado. Nadie trabaja ahí directamente. Todo el trabajo ocurre en branches separados que después se integran vía PR.

### Cómo nombrar los branches

| Tipo de trabajo | Prefijo | Ejemplo |
|---|---|---|
| Feature nueva | `feat/` | `feat/pagina-contratos` |
| Corrección de bug | `fix/` | `fix/error-al-guardar` |
| Cambio de configuración | `config/` | `config/variables-prod` |
| Documentación | `docs/` | `docs/actualizar-readme` |

---

## 3. Flujo diario de trabajo

### Antes de empezar (siempre)

```bash
# 1. Asegurarse de estar en main
git checkout main

# 2. Bajar los últimos cambios del equipo
git pull origin main
```

> Si no hacés esto, vas a trabajar sobre código viejo y habrá conflictos.

### Empezar una feature nueva

```bash
# Crear y moverse al nuevo branch
git checkout -b feat/nombre-de-la-feature
```

Ejemplo:
```bash
git checkout -b feat/pagina-lista-contratos
```

### Mientras trabajás

Commitear seguido (cada vez que algo funciona):
```bash
git add .
git commit -m "feat: agrego listado de contratos con paginación"
```

### Cuando terminás la feature

```bash
# Subir el branch a GitHub
git push origin feat/nombre-de-la-feature
```

Después crear el PR (ver sección 5) y avisar al socio por Slack:
> "PR listo para review: [link del PR] — agrega listado de contratos"

### Después de que el PR se mergea

```bash
# Volver a main y actualizarlo
git checkout main
git pull origin main

# Eliminar el branch que ya no se necesita
git branch -d feat/nombre-de-la-feature
```

---

## 4. Cómo hacer buenos commits

### Formato

```
tipo: descripción corta en minúsculas
```

### Tipos disponibles

| Tipo | Cuándo usarlo |
|---|---|
| `feat` | Agregás algo nuevo (página, componente, acción) |
| `fix` | Corregís un bug |
| `refactor` | Reorganizás código sin cambiar lo que hace |
| `config` | Cambiás configuración (env, next.config, etc.) |
| `docs` | Modificás documentación o comentarios |
| `style` | Cambios de UI/CSS sin cambios de lógica |

### Ejemplos concretos

```bash
# ✅ Buenos commits
git commit -m "feat: agregar formulario de nuevo contrato"
git commit -m "fix: corregir scroll en preview del contrato"
git commit -m "config: activar feature hasContracts para cliente Rodriguez"
git commit -m "style: ajustar colores del header al branding del cliente"

# ❌ Malos commits (no hacen esto)
git commit -m "cambios"
git commit -m "arregle cosas"
git commit -m "wip"
git commit -m "asdasd"
```

### Cuándo commitear

- Cada vez que una parte pequeña funciona
- Antes de irte a almorzar o terminar el día
- Antes de probar algo experimental
- **No** esperar a tener todo terminado para hacer un solo commit gigante

---

## 5. Pull Requests paso a paso

### Crear el PR (quien hizo la feature)

```bash
# Subir el branch
git push origin feat/mi-feature

# Crear el PR desde la terminal
gh pr create --title "feat: [descripción clara]" --body "$(cat <<'EOF'
## Qué hace este PR
- Agrega X
- Modifica Y
- Corrige Z

## Cómo probarlo
1. Ir a /ruta-nueva
2. Hacer X acción
3. Verificar que Y ocurre
EOF
)"
```

O desde GitHub web: después del push aparece un botón "Compare & pull request".

**Avisar al socio por Slack con el link del PR.**

### Revisar el PR (quien NO hizo la feature)

1. Abrir el link del PR en GitHub
2. Ir a la pestaña **"Files changed"** para ver qué cambió
3. Leer el código — si algo no se entiende, preguntar en el PR con un comentario
4. Probar los cambios en local si es necesario:
```bash
git fetch origin
git checkout feat/mi-feature
npm run dev
```
5. Si todo está bien: click en **"Review changes" → "Approve"**
6. Si hay algo a corregir: click en **"Request changes"** y dejar un comentario claro

### Mergear (quien hizo la feature, solo con aprobación)

1. Verificar que el PR tiene ✅ aprobación del socio
2. Click en **"Merge pull request"**
3. Elegir **"Squash and merge"** (junta todos los commits en uno limpio)
4. Confirmar
5. Eliminar el branch (GitHub lo sugiere automáticamente)

---

## 6. Trabajar en paralelo sin conflictos

### Regla principal

> Cada uno trabaja en archivos distintos al mismo tiempo.

Si Lautaro está en `components/contratos/Editor.tsx`, Francisco trabaja en `actions/contratos.actions.ts` o en una feature completamente diferente.

**Antes de tocar un archivo que el otro podría estar usando:** mandar un mensaje por Slack:
> "Voy a trabajar en components/layout/Header.tsx, ¿estás tocando algo ahí?"

### Archivos de alto riesgo de conflicto

Estos archivos los tocan frecuentemente los dos — coordinar antes de modificarlos:

- `app/layout.tsx`
- `components/layout/Header.tsx`
- `config/site.ts`
- `config/features.ts`
- `package.json`

### Qué hacer si hay un merge conflict

Un conflict aparece cuando los dos modificaron el mismo archivo. Git lo marca así:

```
<<<<<<< HEAD
  // tu versión
=======
  // versión del socio
>>>>>>> feat/su-feature
```

**Pasos para resolverlo:**

```bash
# 1. Actualizar main
git checkout main
git pull origin main

# 2. Volver a tu branch e integrar main
git checkout feat/tu-feature
git merge main
```

Si hay conflictos, Git te dice qué archivos tienen el problema. Abrirlos en VS Code — aparecen con marcadores rojos. Elegir qué versión conservar (o combinar las dos) y guardar.

```bash
# 3. Después de resolver
git add .
git commit -m "fix: resolver conflictos con main"
git push origin feat/tu-feature
```

Si hay dudas, **no adivinés** — llamar al socio y resolverlo juntos.

---

## 7. Reglas de oro

Estas reglas nunca se rompen, sin excepciones:

| # | Regla | Por qué |
|---|---|---|
| 1 | **Nunca pushear directo a `main`** | `main` es producción. Un error rompe la app del cliente. |
| 2 | **Nunca mergear tu propio PR** | Siempre necesita aprobación del otro. Dos ojos ven más que uno. |
| 3 | **Siempre hacer `git pull` antes de empezar** | Si no, trabajás sobre código viejo y habrá conflictos. |
| 4 | **Nunca commitear `.env.local`** | Tiene keys secretas. Si se sube a GitHub, hay que rotar todas las keys. |
| 5 | **Cambios al CORE del base-template → PR + review obligatorio** | Un error en el core se propaga a todos los clientes. |
| 6 | **Avisar por Slack antes de tocar archivos compartidos** | Previene el 90% de los conflictos. |
| 7 | **Documentar en `/migrations/` todo cambio al CORE** | Permite propagarlo a otros repos de forma controlada. |

---

## Comandos de referencia rápida

```bash
# Ver en qué branch estás
git status

# Ver todos los branches
git branch

# Actualizar main
git checkout main && git pull origin main

# Crear branch nuevo
git checkout -b feat/nombre

# Ver qué cambios hiciste
git diff

# Agregar y commitear
git add .
git commit -m "tipo: descripción"

# Subir branch a GitHub
git push origin feat/nombre

# Crear PR
gh pr create

# Ver PRs abiertos
gh pr list

# Mergear con main (si hay conflictos)
git merge main
```
