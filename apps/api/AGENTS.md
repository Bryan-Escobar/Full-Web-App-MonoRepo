# AGENTS.md - Developer Guidelines for Planify-be-avanzado

## Project Overview

TypeScript/Express REST API with Prisma ORM, PostgreSQL, Supabase Auth, and Zod validation. Uses clean architecture with presentation/domain/services layers.

## Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server with hot reload (ts-node-dev) |
| `npm run build` | Compile TypeScript to `dist/` (runs `rimraf ./dist && tsc`) |
| `npm run start` | Build and run production server |

**Note:** No test framework is currently configured. Do not add test-related functionality without consulting the user first.

## Code Style Guidelines

### TypeScript

- **Strict mode is enabled** in `tsconfig.json` - all strict checks apply
- Use explicit types for function parameters and return types
- Prefer interfaces over types for object shapes
- Use `unknown` type for catch block errors, then narrow with type guards

### Imports

- Use relative paths: `../../services/usuario.service`
- Avoid absolute imports (no path aliases configured)
- Order imports: external (express, zod, prisma) → internal modules
- Use `import { X } from 'module'` (named imports) over default imports

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files | kebab-case | `usuario.service.ts` |
| Classes | PascalCase | `UsuarioController` |
| Interfaces | PascalCase | `Usuario`, `ApiResponse` |
| Functions/methods | camelCase | `getUsuarios()`, `createUsuario()` |
| Database tables | snake_case | `usuario_facultad` |

### Folder Structure

```
src/
├── app.ts                      # Entry point
├── config/                     # Configuration (envs, supabase)
├── domain/
│   ├── errors/                # CustomError class
│   ├── interfaces/            # TypeScript interfaces
│   └── validators/            # Zod schemas
├── lib/                       # Shared utilities (errorHandler)
├── presentation/
│   ├── middlewares/           # Express middlewares
│   ├── [Entity]/              # Entity-specific (controller + routes)
│   ├── routes.ts              # Main router aggregator
│   └── server.ts              # Express server setup
└── services/                  # Business logic layer
```

### Response Format

All API responses MUST use the `ApiResponse<T>` interface:

```typescript
interface ApiResponse<T = unknown> {
  statusCode: number;
  success: boolean;
  message: string;
  data?: T;
  errors?: ValidationError[];
}
```

Example in controller:
```typescript
const response: ApiResponse = {
  statusCode: 200,
  success: true,
  message: "Usuarios obtenidos exitosamente",
  data: usuarios
};
res.status(200).json(response);
```

### Error Handling

1. **Throw CustomError** in services/controllers:
   ```typescript
   throw new CustomError("Usuario no encontrado", 404);
   ```
   
2. **Use factory methods** for common errors:
   ```typescript
   CustomError.badRequest("Mensaje")
   CustomError.unauthorized()
   CustomError.notFound("Recurso no encontrado")
   CustomError.internal("Error interno")
   ```

3. **Handle in controller** with `handleError` helper:
   ```typescript
   } catch (error) {
     console.error('Error en ControllerName.method:', error);
     const { statusCode, message } = handleError(error);
     const response: ApiResponse = { statusCode, success: false, message };
     res.status(statusCode).json(response);
   }
   ```

### Validation with Zod

- Create validators in `src/domain/validators/[entity].validator.ts`
- Use Zod schemas for request validation
- Export inferred types: `export type CreateUsuarioInput = z.infer<typeof CreateUsuarioSchema>`

Example schema:
```typescript
export const CreateUsuarioSchema = z.object({
  nombre: z.string().min(1, "El nombre es requerido"),
  email: z.string().email("Email inválido"),
  // ...
});
```

### Routes Pattern

Use the static getter pattern with JSDoc comments:

```typescript
export class UsuarioRoutes {
  static get routes(): Router {
    const router = Router();
    const controller = new UsuarioController();

    /**
     * GET /api/usuarios
     * Descripción del endpoint
     */
    router.get('/', controller.getUsuarios);

    return router;
  }
}
```

### Database Access

- Create a new `PrismaClient` instance in each service file
- Use transactions (`prisma.$transaction`) for multi-table operations
- Follow naming: `prisma.usuario`, `prisma.facultad` (singular, matches model names)

### Prisma Usage

- Generated client is at `src/generated/prisma/client`
- Import: `import { PrismaClient } from "../generated/prisma/client";`
- Use `include` for relations (e.g., `include: { facultad: true }`)

### Middlewares

- Validation middlewares available: `validateBody`, `validateParams`, `validateQuery`
- All return 400 on validation failure with formatted errors

Example:
```typescript
router.post(
  '/',
  validateBody(CreateUsuarioSchema),
  controller.createUsuario
);
```

## What NOT to Do

- Do not commit `.env` files or secrets
- Do not use `any` type unless absolutely necessary
- Do not skip strict null checks
- Do not add new dependencies without approval
- Do not create tests without consulting the user first
- Do not use console.log in production code (use proper logging)

## Environment Variables

Required in `.env`:
- `PORT` - Server port
- `DATABASE_URL` - PostgreSQL connection string
- Supabase credentials (`SUPABASE_URL`, `SUPABASE_KEY`, etc.)

See `src/config/envs.ts` for complete list.
