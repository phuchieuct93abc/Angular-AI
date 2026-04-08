# Copilot Instructions for angular-ai

## Build and Test Commands

### Development Server
```bash
npm start
```
Starts the dev server at `http://localhost:4200/`. The app automatically reloads on source file changes.

### Building
```bash
npm run build
```
Produces production build artifacts in `dist/angular-ai/` with optimization enabled.

### Unit Tests
```bash
npm test
```
Runs the full test suite via Karma and Jasmine. Tests are located alongside components as `*.spec.ts` files.

To run a single test file:
```bash
npm test -- --include='**/component-name.spec.ts'
```

### Watch Mode
```bash
npm run watch
```
Builds continuously in development configuration as source files change.

### Code Generation
Angular CLI generates scaffolding with proper conventions:
```bash
ng generate component component-name       # Creates component with .scss styling
ng generate service service-name           # Creates service with .spec.ts
ng generate directive|pipe|guard|etc name  # Generates other Angular artifacts
```

All generated components use the `app` prefix by default (e.g., `app-component-name`).

## Architecture

### Standalone Components (Angular 18+)
This project uses Angular's **standalone component model** exclusively. All components use:
- `standalone: true` in the component decorator
- `imports: []` array to declare dependencies
- **No NgModules**

No `app.module.ts` exists. Configuration is handled through `ApplicationConfig` in `app.config.ts`.

### Bootstrap Flow
1. `main.ts` → Calls `bootstrapApplication(AppComponent, appConfig)`
2. `app.config.ts` → Defines `ApplicationConfig` with providers (zone change detection, router)
3. `app.routes.ts` → Defines the application's route configuration
4. `AppComponent` → Root standalone component with `RouterOutlet` for routing

### Styling
- **Language**: SCSS (configured in `angular.json` schematics)
- **Global styles**: `src/styles.scss`
- **Component styles**: Colocated as `component-name.component.scss`

When generating new components, SCSS is automatically used (not CSS).

### File Structure
```
src/
  app/                    # Application root
    app.component.*       # Root component (HTML, SCSS, TS, spec)
    app.config.ts         # Dependency injection configuration
    app.routes.ts         # Route definitions
  styles.scss             # Global styles
  main.ts                 # Bootstrap entry point
  index.html              # HTML entry point
public/                   # Static assets (copied to dist on build)
```

### Production Budgets
Angular's build optimizer enforces bundle size constraints:
- **Initial bundle**: Max 1MB (warning at 500kB)
- **Component styles**: Max 4kB per component (warning at 2kB)

Keep styles minimal and tree-shakeable.

## Key Conventions

### TypeScript Configuration
- **Strict mode enabled**: `strict: true` in `tsconfig.json`
- **Required return type annotations** on all functions
- **No implicit any** – types required for all parameters
- **Experimental decorators** enabled for Angular metadata
- **Target**: ES2022 with ES2022 modules

### Component Naming and Selectors
- Component files: `component-name.component.ts`
- Component classes: `ComponentNameComponent`
- Selector prefix: `app` (e.g., `<app-component-name>`)
- Styles file: `component-name.component.scss`
- Template file: `component-name.component.html`
- Tests file: `component-name.component.spec.ts`

Generated components follow these conventions automatically via Angular CLI.

### Testing with Jasmine and Karma
- Test files are colocated with their component: `*.spec.ts`
- Use `TestBed` to configure and compile components
- Mock dependencies via Jasmine's `spyOn()` or manual mock implementations
- Run a subset of tests with `--include` or `--exclude` flags

### Router Configuration
Routes are defined in `app.routes.ts` as a flat array. Use lazy loading for feature modules if scalability becomes necessary:
```typescript
{
  path: 'feature',
  loadChildren: () => import('./feature/feature.routes').then(m => m.featureRoutes)
}
```

### Dependency Injection
Providers are declared in `app.config.ts` and made available application-wide via `ApplicationConfig`. Use constructor injection in components and services:
```typescript
constructor(private myService: MyService) {}
```

### Zone Change Detection
Zone.js change detection is optimized with `eventCoalescing: true` to batch change detection runs and improve performance.

## Development Workflow

1. **Create new features** in `src/app/` as standalone components/services
2. **Add routes** to `app.routes.ts`
3. **Configure providers** in `app.config.ts` if adding global services or interceptors
4. **Write tests** alongside each component (`.spec.ts`)
5. **Use SCSS** for all styling
6. **Run `npm test`** before committing to catch regressions
7. **Check bundle size** before production builds to stay within budgets

## Build Output
- Production build output: `dist/angular-ai/`
- Development server serves from memory with source maps enabled
- Static assets from `public/` are included in the final build
