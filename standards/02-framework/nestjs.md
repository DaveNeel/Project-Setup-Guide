# Framework Standard: NestJS (Tier 1)

## 1. Purpose
Defines the structure, bootstrap lifecycle, and cross-cutting integration contracts for a NestJS application.

---

## 2. Applies when
- `framework: nestjs` is selected in Step 2.

---

## 3. Interaction Notes

| Preceding Decision | Impact on NestJS Standard |
|:---|:---|
| Step 1 (Architecture) | In microservices, applies per service in `services/*`. In monolith, applies to the root app. |
| Step 3 & 4 (DB & ORM) | Configures the database module (`TypeOrmModule`, `MongooseModule`, etc.) in the root `AppModule`. |

---

## 4. Contract

### Folder Layout & Module Architecture
- Root source tree MUST reside in `src/`.
- Entry point MUST be `src/main.ts`.
- Root module MUST be `src/app.module.ts`.
- Feature domains MUST be encapsulated in NestJS modules: `src/modules/<feature>/<feature>.module.ts`.
- File naming MUST follow kebab-case with strict suffixes: `.controller.ts`, `.service.ts`, `.repository.ts`, `.dto.ts`, `.module.ts`.

### Bootstrap Order
In `src/main.ts`, the bootstrap sequence MUST execute in this exact sequence:
1. **Config Validation:** Load and validate configuration schema before starting the Nest factory.
2. **Nest Application Creation:** Instantiate via `NestFactory.create(AppModule, { bufferLogs: true })`.
3. **Structured Logger:** Wire custom Pino logger via `app.useLogger(app.get(PinoLoggerService))`.
4. **Global Middleware:** Wire correlation ID middleware via `app.use(correlationIdMiddleware)`.
5. **Global Validation Pipe:** Wire `ValidationPipe` with `{ whitelist: true, forbidNonWhitelisted: true, transform: true }`.
6. **Global Exception Filter:** Wire `AllExceptionsFilter` mapped to the cross-cutting error envelope.
7. **Global Interceptors:** Wire response transformation interceptor ensuring `{ success: true, data }`.
8. **Port Listen:** Bind to `config.PORT`.

---

## 5. Illustrative Bootstrap Snippet
<!-- illustrative only -->
```typescript
async function bootstrap() {
  const config = validateConfig(process.env);
  const app = await NestFactory.create(AppModule, { bufferLogs: true });
  app.useLogger(app.get(LoggerService));
  app.use(correlationIdMiddleware);
  app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }));
  app.useGlobalFilters(new AllExceptionsFilter());
  app.useGlobalInterceptors(new ResponseTransformInterceptor());
  app.enableShutdownHooks();
  await app.listen(config.PORT);
}
bootstrap();
```

---

## 6. Agent Output Checklist
- [ ] `src/main.ts` executes the exact bootstrap sequence.
- [ ] `AllExceptionsFilter` implements `ExceptionFilter` and returns standard error envelope.
- [ ] Response interceptor wraps successful outputs in `{ success: true, data }`.
- [ ] All controllers use standard HTTP decorators (`@Get`, `@Post`, `@HttpCode`).

---

## 7. Common Mistakes
- **Controllers Calling Repositories Directly:** Injecting `@InjectRepository` or a repository class into a controller.
- **Unvalidated Configuration:** Reading `process.env.PORT` directly without bootstrap validation.
- **Missing Correlation ID:** Creating logs in filters without propagating `req.correlationId`.
