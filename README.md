# Foody

Monorepo del proyecto SaaS **Foody**, organizado con npm workspaces.

## Stack

| App | Tecnologías |
|-----|-------------|
| `apps/web` | React 19, Vite, TypeScript, Tailwind CSS, Supabase, Zustand |
| `apps/api` | Node.js, Express, TypeScript, Prisma ORM, PostgreSQL, Zod |

## Estructura

```
Foody/
├── apps/
│   ├── web/        # Frontend React
│   └── api/        # Backend Express + Prisma
├── packages/       # Código compartido (tipos, validadores)
├── package.json    # Workspace raíz
└── .gitignore
```

## Requisitos

- Node.js >= 18
- npm >= 9
- PostgreSQL (o proyecto en Supabase)

## Instalación

```bash
# Instala todas las dependencias de todas las apps desde la raíz
npm install
```

## Scripts

```bash
npm run dev:web     # Inicia el frontend en desarrollo  → http://localhost:5173
npm run dev:api     # Inicia el backend en desarrollo   → http://localhost:3000
npm run build:web   # Compila el frontend
npm run build:api   # Compila el backend
```

## Variables de entorno

Cada app maneja su propio `.env`. Copia los archivos de ejemplo antes de arrancar:

```bash
# Frontend
cp apps/web/.env.example apps/web/.env

# Backend
cp apps/api/.env.template apps/api/.env
```

### `apps/web/.env`

```env
VITE_SUPABASE_URL=https://<tu-proyecto>.supabase.co
VITE_SUPABASE_ANON_KEY=<tu-anon-key>
```

### `apps/api/.env`

```env
PORT=3000
DATABASE_URL="postgresql://user:password@host:6543/postgres?pgbouncer=true"
DIRECT_URL="postgresql://user:password@host:5432/postgres"
```

## Generar cliente Prisma

Ejecutar después de cualquier cambio en `apps/api/prisma/schema.prisma`:

```bash
cd apps/api && npx prisma generate
```

## Tests (API)

```bash
npm test --workspace=apps/api
npm run test:coverage --workspace=apps/api
```

## Licencia

MIT
