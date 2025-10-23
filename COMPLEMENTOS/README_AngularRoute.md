# Entrenamiento Angular – Día 1 (Guía Completa y Corregida)

**Introducción Moderna a Angular 20, CSR vs SSR con Vite y prevención de NG0401**

Esta guía te lleva **desde cero** a un proyecto Angular 20 funcionando en **CSR** (cliente) y **SSR** (servidor) con **Vite**, usando **componentes standalone** y **router**. Incluye:

- Diferencias prácticas CSR vs SSR (con Vite).
- Estructura de archivos y para qué sirve cada uno.
- Configuración correcta de `main.ts`, `main.server.ts`, `app.config.ts`, `app.config.server.ts`.
- **Registro del Router** y renderizado de componentes por ruta.
- Scripts mínimos en `package.json` y targets en `angular.json`.
- Checklist y troubleshooting paso a paso (**NG0401: Missing Platform** incluido).
- Comandos y **salidas típicas** de la consola para que sepas qué elegir en cada prompt.

---

## 0) Objetivos

- Comprender el **arranque standalone** en Angular 20 (sin módulos).
- Crear proyectos con y sin **SSR**, sabiendo qué comandos usar en cada caso.
- Evitar y solucionar **NG0401: Missing Platform** en SSR.
- Configurar **router** con rutas **lazy (loadComponent)** y `RouterOutlet`.
- Dejar una **plantilla mínima** y funcional para CSR y SSR.

---

## 1) ¿Qué es Angular hoy?

Angular es un framework frontend (TypeScript) para construir SPA y apps con SSR. Desde Angular 17+:

- Se fomenta el uso de **componentes standalone** (no necesitas `AppModule`).
- **Vite** es el empaquetador por defecto.
- Puedes elegir **CSR** (render en navegador) o **SSR** (render en Node + hidratación en cliente).

**Piezas clave:**

- `main.ts` → arranque en cliente.
- `main.server.ts` → arranque en servidor (**solo SSR**).
- `app.config.ts` y `app.config.server.ts` → configuración de providers para cliente y servidor.
- `app.routes.ts` → **rutas** (standalone, con `loadComponent` recomendado).
- Comandos distintos para ejecutar CSR y SSR.

---

## 2) CSR vs SSR con Vite (diferencias reales)

| Aspecto | CSR (Cliente) | SSR (Servidor) |
|---|---|---|
| Renderizado | En el navegador | En Node (prerender en servidor y luego hidrata en cliente) |
| Comando típico | `ng serve` | `npm run dev:ssr` (o `ng run <app>:serve-ssr`) |
| Archivos obligatorios | `main.ts`, `app.config.ts` | `main.ts`, `app.config.ts`, `main.server.ts`, `app.config.server.ts` |
| `BootstrapContext` | No aplica | **Obligatorio** en `main.server.ts` |
| SEO | Limitado | Mejor **SEO** (HTML del servidor) |
| Performance inicial | Depende del cliente | Mejor **TTFB** |
| Complejidad | Menor | Mayor (dos pipelines: browser + server) |

---

## 3) Instalación del entorno

1) Verifica **Node y npm** (recomendado Node 18+):
```bash
node -v
npm -v
```

2) Instala **Angular CLI** (global):
```bash
npm install -g @angular/cli
ng version
```

**Salida típica al correr `ng version`** (resumen):
```
Angular CLI: 20.x.x
Node: 18.x.x
Package Manager: npm x.x.x
...
```

---

## 4) Crear proyecto

### 4.1 Solo Cliente (CSR)

```bash
ng new mi-proyecto --ssr=false
# Prompts típicos del CLI (elige):
# ? Which stylesheet format would you like to use? CSS
# ? Do you want to enable server-side rendering (SSR) and static site generation (SSG/Prerendering)? No
cd mi-proyecto
ng serve
# Abre: http://localhost:4200
```

**Salida típica de `ng serve`:**
```
✔ Browser application bundle generation complete.
Initial chunk files ... 
... 
✔ Compiled successfully.
```

**Ventaja:** más simple para empezar.

---

### 4.2 Con Servidor (SSR)

```bash
ng new mi-proyecto --ssr
# Prompts típicos del CLI (elige):
# ? Stylesheet format: CSS
# ? Enable SSR? Yes
cd mi-proyecto
```

Si tu `package.json` **no incluye** los scripts SSR, agrégalos (ver sección 8). Luego ejecuta:

```bash
npm run dev:ssr
# o:
ng run mi-proyecto:serve-ssr
```

> **Importante:** No arranques SSR con `ng serve` porque eso lanza solo el cliente.

**Salida típica de `npm run dev:ssr`:**
```
✔ Server application bundle generation complete.
✔ Browser application bundle generation complete.
[SSR Dev Server] Listening on http://localhost:4200
```

---

## 5) Estructura de carpetas (Angular 20 + Vite)

| Archivo/Carpeta | Uso |
|---|---|
| `src/app/` | Componentes, servicios y rutas. |
| `src/app/app.ts` | **Componente raíz** standalone. |
| `src/app/app.routes.ts` | **Definición de rutas** (standalone). |
| `src/app/app.config.ts` | Configuración de **cliente** (providers, router, hidratación). |
| `src/app/app.config.server.ts` | Configuración de **servidor** (añade `provideServerRendering()`). |
| `src/main.ts` | Punto de **entrada del cliente**. |
| `src/main.server.ts` | **Entrada del servidor** (recibe y pasa `BootstrapContext`). |
| `angular.json` | Targets de build/serve para browser y server. |
| `package.json` | Scripts para CSR/SSR. |
| `vite.config.ts` | Configuración de Vite. |

---

## 6) Archivos base (copiar/pegar)

> **Nota:** Si vienes de CLI con SSR, algunos archivos ya existirán. Revisa y **ajusta** según este patrón.

### 6.1 `src/app/app.ts` — componente raíz standalone

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet], // RouterOutlet para render de rutas
  template: `
    <nav style="display:flex; gap:.75rem; padding:.5rem 0">
      <a routerLink="/">Inicio</a>
      <a routerLink="/acerca">Acerca</a>
    </nav>

    <h1>Angular 20 – CSR/SSR</h1>
    <p>Proyecto base funcionando.</p>

    <!-- Aquí se renderiza la ruta activa -->
    <router-outlet></router-outlet>
  `,
})
export class App {}
```

---

### 6.2 `src/app/app.routes.ts` — **Router** (standalone con loadComponent)

```ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    loadComponent: () =>
      import('./saludo/saludo.component').then(m => m.SaludoComponent),
  },
  {
    path: 'acerca',
    loadComponent: () =>
      import('./acerca/acerca.component').then(m => m.AcercaComponent),
  },
  {
    path: '**',
    loadComponent: () =>
      import('./not-found/not-found.component').then(m => m.NotFoundComponent),
  },
];
```

> Puedes empezar con solo la ruta `''` y el wildcard `**`. Se recomienda **lazy** con `loadComponent` para standalone.

---

### 6.3 `src/app/app.config.ts` — configuración de **cliente**

```ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter, withInMemoryScrolling, withPreloading } from '@angular/router';
import { PreloadAllModules } from '@angular/router';
import { provideClientHydration } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withInMemoryScrolling({ scrollPositionRestoration: 'enabled', anchorScrolling: 'enabled' }),
      withPreloading(PreloadAllModules)
    ),
    provideClientHydration(),
    importProvidersFrom(FormsModule),
  ],
};
```

---

### 6.4 `src/app/app.config.server.ts` — configuración de **servidor**

```ts
import { ApplicationConfig } from '@angular/core';
import { provideServerRendering } from '@angular/platform-server';
import { appConfig } from './app.config';

export const appConfigServer: ApplicationConfig = {
  ...appConfig,
  providers: [
    ...appConfig.providers!,
    provideServerRendering(),
  ],
};
```

---

### 6.5 `src/main.ts` — **cliente**

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { App } from './app/app';

bootstrapApplication(App, appConfig)
  .catch((err) => console.error(err));
```

---

### 6.6 `src/main.server.ts` — **servidor** (evita NG0401)

```ts
import { bootstrapApplication, BootstrapContext } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfigServer } from './app/app.config.server';

export default function bootstrap(context: BootstrapContext) {
  // CLAVE: pasar el 'context' como tercer argumento
  return bootstrapApplication(App, appConfigServer, context);
}
```

**Alternativa** (sin `app.config.server.ts`):

```ts
import { bootstrapApplication, BootstrapContext } from '@angular/platform-browser';
import { provideServerRendering } from '@angular/platform-server';
import { App } from './app/app';
import { appConfig } from './app/app.config';

export default function bootstrap(context: BootstrapContext) {
  return bootstrapApplication(
    App,
    {
      ...appConfig,
      providers: [...(appConfig.providers ?? []), provideServerRendering()],
    },
    context
  );
}
```

---

## 7) Componentes de ejemplo (standalone)

### 7.1 Generación por CLI

```bash
ng generate component saludo --standalone
ng generate component acerca --standalone
ng generate component not-found --standalone
```

**Salida típica:**
```
CREATE src/app/saludo/saludo.component.ts ...
CREATE src/app/saludo/saludo.component.html ...
...
```

### 7.2 `src/app/saludo/saludo.component.ts`

```ts
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-saludo',
  standalone: true,
  imports: [FormsModule],
  template: `
    <h2>Bienvenido a Angular</h2>
    <input [(ngModel)]="nombre" placeholder="Escribe tu nombre">
    <p>Hola {{ nombre }}!</p>
  `
})
export class SaludoComponent {
  nombre = 'Coder';
}
```

### 7.3 `src/app/acerca/acerca.component.ts` (mínimo)

```ts
import { Component } from '@angular/core';

@Component({
  standalone: true,
  template: `
    <h2>Acerca</h2>
    <p>Esta es una página de ejemplo para probar el router.</p>
  `
})
export class AcercaComponent {}
```

### 7.4 `src/app/not-found/not-found.component.ts` (mínimo)

```ts
import { Component } from '@angular/core';

@Component({
  standalone: true,
  template: `
    <h2>404 – No encontrado</h2>
    <p>La ruta solicitada no existe.</p>
  `
})
export class NotFoundComponent {}
```

---

## 8) `package.json` — scripts mínimos SSR/CSR

Ejemplo para un proyecto llamado `mi-proyecto` (ajusta el nombre según tu `angular.json`):

```json
{
  "name": "mi-proyecto",
  "scripts": {
    "start": "ng serve",
    "build": "ng build",
    "dev:ssr": "ng run mi-proyecto:serve-ssr",
    "build:ssr": "ng run mi-proyecto:build-ssr",
    "serve:ssr": "node dist/mi-proyecto/server/server.mjs"
  }
}
```

Si `npm run dev:ssr` dice **“Missing script”**, añade los scripts o ejecuta:

```bash
ng add @angular/ssr
```

> Suele completar la configuración automáticamente.

---

## 9) `angular.json` — targets esperados para SSR

Bloques relevantes (puede variar según CLI):

```json
{
  "projects": {
    "mi-proyecto": {
      "projectType": "application",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:application",
          "options": {
            "outputPath": "dist/mi-proyecto/browser",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": ["zone.js"],
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": ["src/styles.css"]
          }
        },
        "server": {
          "builder": "@angular-devkit/build-angular:server",
          "options": {
            "outputPath": "dist/mi-proyecto/server",
            "main": "src/main.server.ts",
            "tsConfig": "tsconfig.server.json"
          }
        },
        "serve-ssr": {
          "builder": "@nguniversal/builders:ssr-dev-server",
          "options": {
            "browserTarget": "mi-proyecto:build",
            "serverTarget": "mi-proyecto:server"
          }
        },
        "build-ssr": {
          "builder": "@nguniversal/builders:ssr",
          "options": {
            "browserTarget": "mi-proyecto:build:production",
            "serverTarget": "mi-proyecto:server:production"
          }
        }
      }
    }
  }
}
```

Si faltan `server`, `serve-ssr` o `build-ssr`, ejecuta:

```bash
ng add @angular/ssr
```

---

## 10) Cómo arrancar y verificar

### Solo cliente (CSR)

```bash
ng serve
# http://localhost:4200
# Navega: Inicio (ruta '' con SaludoComponent) y /acerca
```

### Con servidor (SSR)

```bash
npm run dev:ssr
# o:
ng run mi-proyecto:serve-ssr
# Abre http://localhost:4200
```

**Verificación rápida:**

- En `/` ves `<h2>Bienvenido a Angular</h2>` y el input con `[(ngModel)]`.
- El `<nav>` permite ir a `/acerca`, donde verás “Esta es una página de ejemplo…”. 
- Rutas inexistentes → `404 – No encontrado`.

---

## 11) Troubleshooting (incluye NG0401)

### NG0401: Missing Platform (SSR)

**Causa:** `main.server.ts` no pasa `BootstrapContext` a `bootstrapApplication` o falta `provideServerRendering()`.

**Arreglo (correcto):**

```ts
export default function bootstrap(context: BootstrapContext) {
  return bootstrapApplication(App, appConfigServer, context);
}
```

y en `app.config.server.ts` asegúrate de agregar `provideServerRendering()`.

**Alternativa inline:** añadir `provideServerRendering()` directo en `main.server.ts` (ver 6.6 alternativa).

---

### “Missing script: dev:ssr”

**Causa:** `package.json` no tiene scripts SSR.

**Arreglo:** añade los scripts (sección 8) o ejecuta `ng add @angular/ssr`.

---

### Cambié archivos y sigue fallando (cache sucia)

**Arreglo (limpia y reinstala):**
```bash
rm -rf .angular/cache node_modules
npm i
# luego corre el comando correcto según CSR o SSR
```

---

### `document is not defined` en SSR

**Causa:** código que asume DOM en el servidor.

**Arreglo:** condiciona con `isPlatformBrowser()`, por ejemplo:
```ts
import { inject, PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser } from '@angular/common';

const platformId = inject(PLATFORM_ID);
if (isPlatformBrowser(platformId)) {
  // aquí sí puedes usar window/document
}
```

---

### Router no renderiza / errores de plantilla

- Asegúrate de **importar `RouterOutlet`** en el root standalone (`imports: [RouterOutlet]`).
- Verifica que `provideRouter(routes)` está en `app.config.ts`.
- Si usas `[(ngModel)]`, recuerda `importProvidersFrom(FormsModule)` en `app.config.ts` o importar `FormsModule` en el componente standalone.

---

## 12) Checklist rápido

- [ ] ¿Elegiste CSR o SSR y usas el comando correcto? (`ng serve` vs `npm run dev:ssr`)
- [ ] `main.server.ts` **pasa** el `BootstrapContext`.
- [ ] `app.config.server.ts` **incluye** `provideServerRendering()`.
- [ ] `package.json` tiene `dev:ssr`, `build:ssr`, `serve:ssr`.
- [ ] `angular.json` contiene `server`, `serve-ssr`, `build-ssr`.
- [ ] `app.routes.ts` exporta `routes: Routes` (con `loadComponent` recomendado).
- [ ] El root `App` **importa** `RouterOutlet` (standalone).
- [ ] `FormsModule` disponible donde uses `[(ngModel)]`.
- [ ] Limpiada la caché (`.angular/cache`) tras cambios de arranque.

---

## 13) Apéndice A — Variante de rutas sin lazy (para pruebas rápidas)

```ts
import { Routes } from '@angular/router';
import { SaludoComponent } from './saludo/saludo.component';
import { AcercaComponent } from './acerca/acerca.component';
import { NotFoundComponent } from './not-found/not-found.component';

export const routes: Routes = [
  { path: '', component: SaludoComponent, pathMatch: 'full' },
  { path: 'acerca', component: AcercaComponent },
  { path: '**', component: NotFoundComponent },
];
```

> Funciona, pero en standalone moderno se prefiere `loadComponent` para carga diferida.

---

## 14) Apéndice B — Rutas con parámetros y query params

```ts
export const routes: Routes = [
  {
    path: 'saludo/:id',
    loadComponent: () => import('./saludo/saludo.component').then(m => m.SaludoComponent),
  },
];
```

En el componente:
```ts
import { inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

const route = inject(ActivatedRoute);
route.paramMap.subscribe(p => console.log('id:', p.get('id')));
route.queryParamMap.subscribe(q => console.log('q:', q.get('q')));
```

---

## 15) Conclusiones

- En Angular 20, el **standalone** simplifica el arranque, pero **SSR** requiere dos entradas (cliente/servidor) y `BootstrapContext` explícito.
- Si empiezas, **CSR** es más simple; activa **SSR** cuando necesites **SEO**, prerender o mejor **TTFB**.
- Esta plantilla te permite alternar entre CSR y SSR sin caer en **NG0401**, con un **router** correcto y componentes standalone **lazy**.

---

### Fin de la guía