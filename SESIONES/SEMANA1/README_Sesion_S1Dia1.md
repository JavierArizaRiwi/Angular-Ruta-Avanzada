# Entrenamiento Angular – Día 1 (Guía Completa y Corregida)

**Introducción Moderna a Angular 20, CSR vs SSR con Vite y Router funcional (Standalone)**

Esta guía te lleva desde cero hasta un proyecto Angular 20 **plenamente funcional** con **componentes standalone**, **router configurado**, y compatibilidad tanto **CSR (cliente)** como **SSR (servidor)**.  
Incluye todos los ajustes necesarios para evitar errores comunes como `NG0401` y `routerLink` inactivos.

---

## 0) Objetivos

- Comprender el **arranque standalone** en Angular 20 (sin módulos).  
- Configurar correctamente `app.ts`, `app.html`, y `app.routes.ts`.  
- Crear proyectos con y sin **SSR**, sabiendo qué comandos usar en cada caso.  
- Implementar un router funcional con navegación `<a routerLink>`.  
- Evitar y solucionar **NG0401: Missing Platform** en SSR.  

---

## 1) ¿Qué es Angular hoy?

Angular 20 es un framework frontend basado en TypeScript para construir **Single Page Applications (SPA)** y aplicaciones **SSR**.  
Desde la versión 17+:

- Se usa **standalone** (no se requiere `AppModule`).  
- **Vite** es el motor de build y servidor de desarrollo.  
- Puedes elegir entre **CSR** o **SSR** según tus necesidades.

**Archivos clave:**

| Archivo | Rol |
|----------|------|
| `main.ts` | Entrada del cliente |
| `main.server.ts` | Entrada del servidor (SSR) |
| `app.ts` | Componente raíz standalone |
| `app.html` | Plantilla principal con `<router-outlet>` |
| `app.routes.ts` | Definición de rutas |
| `app.config.ts` | Configuración global de providers |
| `app.config.server.ts` | Configuración del servidor (SSR) |

---

## 2) CSR vs SSR con Vite

| Aspecto | CSR (Cliente) | SSR (Servidor) |
|---|---|---|
| Renderizado | En navegador | En Node.js |
| Comando | `ng serve` | `npm run dev:ssr` |
| Archivos necesarios | `main.ts`, `app.config.ts` | + `main.server.ts`, `app.config.server.ts` |
| SEO | Limitado | Alto |
| Complejidad | Menor | Mayor |
| Errores típicos | - | `NG0401`, `document is not defined` |

---

## 3) Instalación

1. Verifica Node y npm:

```bash
node -v
npm -v
```

2. Instala Angular CLI:

```bash
npm install -g @angular/cli
ng version
```

---

## 4) Crear proyecto

### CSR (Cliente)

```bash
ng new mi-proyecto --ssr=false
cd mi-proyecto
ng serve
```

### SSR (Servidor)

```bash
ng new mi-proyecto --ssr
cd mi-proyecto
npm run dev:ssr
```

> ⚠️ Si `npm run dev:ssr` no existe, agrega los scripts en el `package.json` o ejecuta `ng add @angular/ssr`.

---

## 5) Estructura general (Angular 20 + Vite)

```
src/
 ├─ app/
 │   ├─ app.ts
 │   ├─ app.html
 │   ├─ app.routes.ts
 │   ├─ app.config.ts
 │   ├─ saludo/
 │   │   ├─ saludo.component.ts
 │   │   └─ saludo.component.html
 │   └─ acerca/
 │       ├─ acerca.component.ts
 │       └─ acerca.component.html
 ├─ main.ts
 ├─ main.server.ts
 └─ index.html
```

---

## 6) Configuración base (Archivos principales)

### 6.1 `index.html`

Asegúrate de incluir el `<base href="/">` en `<head>`:

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <title>Angular 20 App</title>
    <base href="/">
  </head>
  <body>
    <app-root></app-root>
  </body>
</html>
```

---

### 6.2 `src/app/app.html`

```html
<h1>Mi Proyecto Angular</h1>

<nav>
  <a routerLink="/" routerLinkActive="active" [routerLinkActiveOptions]="{ exact: true }">Inicio</a> |
  <a routerLink="/acerca" routerLinkActive="active">Acerca</a>
</nav>

<hr>

<!-- Render dinámico de las rutas -->
<router-outlet></router-outlet>
```

---

### 6.3 `src/app/app.ts` (componente raíz standalone)

```ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  templateUrl: './app.html',
})
export class App {}
```

---

### 6.4 `src/app/app.routes.ts` (rutas funcionales)

```ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    loadComponent: () => import('./saludo/saludo.component').then(m => m.SaludoComponent),
  },
  {
    path: 'acerca',
    loadComponent: () => import('./acerca/acerca.component').then(m => m.AcercaComponent),
  },
];
```

---

### 6.5 `src/app/app.config.ts` (configuración del router y providers)

```ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideClientHydration } from '@angular/platform-browser';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration(),
  ],
};
```

---

### 6.6 `src/main.ts`

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfig } from './app/app.config';

bootstrapApplication(App, appConfig)
  .catch(err => console.error(err));
```

---

## 7) Componentes de ejemplo

```bash
ng generate component saludo --standalone
ng generate component acerca --standalone
```

### `src/app/saludo/saludo.component.ts`

```ts
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
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

### `src/app/acerca/acerca.component.ts`

```ts
import { Component } from '@angular/core';

@Component({
  standalone: true,
  template: `
    <h2>Acerca</h2>
    <p>Aplicación base Angular con navegación funcional.</p>
  `
})
export class AcercaComponent {}
```

---

## 8) Verificación del router

- En `/` debe verse el componente **SaludoComponent**.  
- En `/acerca`, el **AcercaComponent**.  
- Los links en `<nav>` deben cambiar la vista **sin recargar la página**.

Si no navega: revisa `RouterLink` y que `RouterOutlet` esté importado en `App`.

---

## 9) Solución de errores comunes

###`routerLink` no hace nada

**Causa:** `RouterLink` o `RouterOutlet` no importados en `App`.  
**Solución:** agrega en `app.ts`:
```ts
imports: [RouterOutlet, RouterLink, RouterLinkActive]
```

---

###  `NG0401: Missing Platform`

**Causa:** `main.server.ts` no pasa `BootstrapContext` o falta `provideServerRendering()`.  
**Solución:** en `main.server.ts`:

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

## 10) Checklist final

- [x] `<base href="/">` en `index.html`
- [x] `<router-outlet>` en `app.html`
- [x] `imports: [RouterOutlet, RouterLink, RouterLinkActive]` en `App`
- [x] `provideRouter(routes)` en `app.config.ts`
- [x] Rutas correctas en `app.routes.ts`
- [x] `ng serve` o `npm run dev:ssr` según el modo

---

## 11) Conclusión

Esta versión de la guía incorpora todos los ajustes necesarios para que el router de Angular 20 funcione correctamente en **standalone mode**, evitando los errores de navegación y NG0401.  
A partir de esta base, puedes extender el proyecto para los siguientes días de entrenamiento (Data Binding, Servicios, Formularios, etc.).

---