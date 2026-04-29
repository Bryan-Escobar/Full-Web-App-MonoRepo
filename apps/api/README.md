# Foody - Backend

Backend base construido con **Node.js**, **Express**, **TypeScript** y **Prisma ORM**.

## Requisitos Previos

- **Node.js** >= 18.x
- **npm** >= 9.x
- **PostgreSQL**

## Instalación

### 1. Clonar e instalar dependencias

```bash
git clone <repo-url>
cd Foody-be
npm install
```

### 2. Variables de entorno

Crea un archivo `.env` en la raíz del proyecto:

```env
PORT=3000
DATABASE_URL="postgresql://user:password@host:6543/postgres?pgbouncer=true"
DIRECT_URL="postgresql://user:password@host:5432/postgres"
```

- `DATABASE_URL`: conexión con pooler (puerto 6543) para consultas normales
- `DIRECT_URL`: conexión directa (puerto 5432) para introspecciones

### 3. Generar el cliente de Prisma

```bash
npx prisma generate
```

> Ejecutar siempre después de cambios en `prisma/schema.prisma`. No usar `prisma migrate` directamente.

## Scripts

```bash
npm run dev          # Servidor en desarrollo con hot reload
npm run build        # Compila TypeScript → dist/
npm start            # Build + ejecuta el compilado
npm test             # Tests con Mocha
npm run test:coverage  # Tests con reporte de cobertura (nyc)
```

## Estructura del Proyecto

```
src/
├── app.ts                        # Punto de entrada
├── config/
│   ├── envs.ts                   # Variables de entorno (env-var)
│   └── prisma.ts                 # Singleton del cliente Prisma
├── domain/
│   ├── errors/
│   │   ├── CustomError.ts        # Error tipado con statusCode
│   │   └── errorHandler.interface.ts
│   ├── interfaces/
│   │   └── api-response.interface.ts  # Forma estándar de respuesta
│   └── validators/
│       └── health.validator.ts   # Ejemplo de schema Zod
├── lib/
│   └── errorHandler.ts           # Helper centralizado de errores
├── presentation/
│   ├── routes.ts                 # Agregador de rutas
│   ├── server.ts                 # Clase Server (Express)
│   ├── middlewares/
│   │   └── validation.middleware.ts  # validateBody / validateQuery / validateParams
│   └── Health/
│       ├── health.controller.ts
│       └── routes.ts
└── services/
    └── health.service.ts

test/
├── setup.ts                      # Inyecta env vars antes de importar la app
├── helpers/
│   └── app.ts                    # Exporta la instancia Express para supertest
└── presentation/
    └── health.test.ts
```

## Patrones Clave

**Respuesta estándar** — todos los endpoints devuelven `ApiResponse`:
```typescript
{ statusCode, success, message, data?, errors? }
```

**Errores** — los servicios lanzan `CustomError(message, statusCode)`. Los controladores siempre capturan con `handleError(error)`.

**Validación** — schemas Zod en `src/domain/validators/`. Se aplican en las rutas con los middlewares `validateBody`, `validateQuery` o `validateParams`.

**Prisma** — cliente singleton en `src/config/prisma.ts`. El cliente generado vive en `src/generated/prisma/client`.

## Endpoint de ejemplo

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/health` | Estado del servidor |
| POST | `/api/health/echo` | Eco del body (demuestra validación Zod) |

## Tests

Framework: **Mocha + Chai + Sinon + Supertest** (TDD con `suite` / `test` / `teardown`).

Los tests son HTTP-level: golpean el Express app completo y mockean los métodos del servicio con stubs de Sinon. Nunca se conectan a una base de datos real.

```bash
npm test
npm run test:coverage
```

## Dependencias Principales

| Paquete | Uso |
|---------|-----|
| `express` | Framework web |
| `typescript` | Tipado estático |
| `prisma` / `@prisma/client` | ORM |
| `zod` | Validación de esquemas |
| `cors` | Política de origen cruzado |
| `dotenv` / `env-var` | Variables de entorno |

## Licencia

MIT
