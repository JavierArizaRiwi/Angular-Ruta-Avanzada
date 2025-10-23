# Entrenamiento Angular – Día 1 (Guía Completa y Corregida)
## Introducción Moderna a Angular, CSR vs SSR con Vite y prevención de NG0401

Esta guía ha sido reestructurada para que puedas crear, ejecutar y depurar proyectos Angular 20 con Vite en **modo cliente (CSR)** y **modo servidor (SSR)** sin caer en el error **NG0401: Missing Platform**. Incluye:
- Diferencias prácticas CSR vs SSR con Vite.
- Estructura de archivos y para qué sirve cada uno.
- Configuración correcta de `main.ts`, `main.server.ts`, `app.config.ts`, `app.config.server.ts`.
- Scripts mínimos en `package.json` y *targets* en `angular.json`.
- Checklist y troubleshooting paso a paso (incluye limpieza de caché y reinstalación).

---

## 0) Objetivos
- Comprender el arranque moderno **standalone** (sin módulos) en Angular 20.
- Crear proyectos con y sin SSR, sabiendo qué comandos usar en cada caso.
- Evitar y solucionar **NG0401: Missing Platform**.
- Tener una plantilla mínima y funcional para CSR y SSR.

---

## 1) ¿Qué es Angular hoy?
Angular es un framework frontend (TypeScript) para construir **SPA** y apps con **SSR**. Desde Angular 17+:
- Se fomenta el uso de **componentes standalone** (ya no necesitas `AppModule`).
- **Vite** es el empaquetador por defecto.
- Puedes elegir **CSR** (cliente) o **SSR** (renderizado en servidor con Node).

**Piezas clave:**
- `main.ts` → arranque en cliente.
- `main.server.ts` → arranque en servidor (solo SSR).
- `app.config.ts` y `app.config.server.ts` → configuración de *providers* para cliente y servidor.
- Comandos distintos para ejecutar CSR y SSR.

---

## 2) CSR vs SSR con Vite (diferencias reales)
| Aspecto | CSR (Cliente) | SSR (Servidor) |
|---|---|---|
| Renderizado | En el navegador | En Node (prerender en servidor y luego hidrata en cliente) |
| Comando típico | `ng serve` | `npm run dev:ssr` |
| Archivos obligatorios | `main.ts`, `app.config.ts` | `main.ts`, `app.config.ts`, `main.server.ts`, `app.config.server.ts` |
| `BootstrapContext` | No aplica | **Obligatorio** en `main.server.ts` |
| SEO | Limitado | Mejor SEO (HTML del servidor) |
| Performance inicial | Depende del cliente | Mejor TTFB (primera pintura desde el servidor) |
| Complejidad | Menor | Mayor (dos pipelines: browser + server) |

---

## 3) Instalación del entorno
Verifica Node y npm:
```bash
node -v
npm -v
# Recomendado Node 18+
```
Instala Angular CLI:
```bash
npm install -g @angular/cli
ng version
```

---

## 4) Crear proyecto
### 4.1 Solo Cliente (CSR)
```bash
ng new mi-proyecto --ssr=false
cd mi-proyecto
ng serve
# http://localhost:4200
```
**Ventaja:** más simple para empezar.

### 4.2 Con Servidor (SSR)
```bash
ng new mi-proyecto --ssr
cd mi-proyecto
```
Si tu `package.json` no incluye los scripts SSR, agrégalos (ver sección 7). Luego ejecuta:
```bash
npm run dev:ssr
# o: ng run mi-proyecto:serve-ssr
```
**Importante:** **No** arranques SSR con `ng serve` porque eso lanza solo el cliente.

---

## 5) Estructura de carpetas (Angular 20 + Vite)
| Archivo/Carpeta | Uso |
|---|---|
| `src/app/` | Componentes, servicios, rutas. |
| `src/app/app.ts` | Componente raíz **standalone**. |
| `src/app/app.routes.ts` | Definición de rutas (opcional). |
| `src/app/app.config.ts` | Configuración de **cliente** (providers, router, hidratación). |
| `src/app/app.config.server.ts` | Configuración de **servidor** (añade `provideServerRendering()`). |
| `src/main.ts` | Punto de entrada del cliente. |
| `src/main.server.ts` | Punto de entrada del servidor (recibe y pasa `BootstrapContext`). |
| `angular.json` | Targets de build/serve tanto browser como server. |
| `package.json` | Scripts para CSR/SSR. |
| `vite.config.ts` | Configuración de Vite. |

---

## 6) Archivos base (copiar/pegar)

### 6.1 `src/app/app.ts` (componente raíz standalone)
```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  standalone: true,
  template: \`
    <h1>Angular 20 – CSR/SSR</h1>
    <p>Proyecto base funcionando.</p>
    <router-outlet></router-outlet>
  \`,
})
export class App {}
```

### 6.2 `src/app/app.config.ts` (cliente)
```ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideClientHydration } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';

// Si no usas rutas, puedes poner: const routes = [];
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration(),
    importProvidersFrom(FormsModule),
  ],
};
```

### 6.3 `src/app/app.config.server.ts` (servidor)
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

### 6.4 `src/main.ts` (cliente)
```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { App } from './app/app';

bootstrapApplication(App, appConfig)
  .catch((err) => console.error(err));
```

### 6.5 `src/main.server.ts` (servidor) – evita NG0401
```ts
import { bootstrapApplication, BootstrapContext } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfigServer } from './app/app.config.server';

export default function bootstrap(context: BootstrapContext) {
  // CLAVE: pasar el 'context' como tercer argumento
  return bootstrapApplication(App, appConfigServer, context);
}
```

> Alternativa (sin `app.config.server.ts`):
```ts
import { bootstrapApplication, BootstrapContext } from '@angular/platform-browser';
import { provideServerRendering } from '@angular/platform-server';
import { App } from './app/app';
import { appConfig } from './app/app.config';

export default function bootstrap(context: BootstrapContext) {
  return bootstrapApplication(App, {
    ...appConfig,
    providers: [...(appConfig.providers ?? []), provideServerRendering()],
  }, context);
}
```

---

## 7) `package.json` – scripts mínimos SSR/CSR
Ejemplo para un proyecto llamado `mi-proyecto` (ajusta el nombre si tu paquete se llama distinto):
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

Si `npm run dev:ssr` dice “Missing script”, agrega esos scripts a mano **o** ejecuta:
```bash
ng add @angular/ssr
```
que suele completar la configuración automáticamente.

---

## 8) `angular.json` – targets esperados para SSR
Bloques relevantes (aproximado; el CLI puede variar ligeramente):

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

## 9) Crear el primer componente (ejemplo rápido)

```bash
ng generate component saludo
```

`src/app/saludo/saludo.component.ts`:
```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-saludo',
  standalone: true,
  template: \`
    <h2>Bienvenido a Angular</h2>
    <input [(ngModel)]="nombre" placeholder="Escribe tu nombre">
    <p>Hola {{ nombre }}!</p>
  \`
})
export class SaludoComponent {
  nombre = 'Coder';
}
```

Usa el componente en la plantilla del `App` o en una ruta. Si lo insertas directo en `app.ts`:
```ts
// ...
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [SaludoComponent],
  template: \`
    <h1>Angular 20 – CSR/SSR</h1>
    <app-saludo></app-saludo>
  \`,
})
export class App {}
```

---

## 10) Cómo arrancar correctamente

### Solo cliente (CSR)
```bash
ng serve
# http://localhost:4200
```

### Con servidor (SSR)
```bash
npm run dev:ssr
# o: ng run mi-proyecto:serve-ssr
```

---

## 11) Troubleshooting (incluye NG0401)

### NG0401: Missing Platform
- Causa: `main.server.ts` no pasa `BootstrapContext` a `bootstrapApplication` o falta `provideServerRendering()`.
- Arreglo: Usa exactamente este patrón:
```ts
export default function bootstrap(context: BootstrapContext) {
  return bootstrapApplication(App, appConfigServer, context);
}
```
y en la config del servidor añade `provideServerRendering()` (ya sea en `app.config.server.ts` o inline en `main.server.ts`).

### “Missing script: dev:ssr”
- Causa: tu `package.json` no tiene scripts SSR.
- Arreglo: añade los scripts de la sección 7 o ejecuta `ng add @angular/ssr`.

### Cambié archivos y sigue fallando
- Limpia caché y reinstala:
```bash
rm -rf .angular/cache node_modules
npm i
```
- Luego ejecuta el comando adecuado (CSR o SSR).

### `document is not defined` en SSR
- Causa: código que asume DOM en el servidor.
- Arreglo: usa `isPlatformBrowser()` para condicionar código que toca `window`, `document`, etc.

---

## 12) Checklist rápido
- [ ] ¿Elegiste CSR o SSR y usas el **comando correcto**? (`ng serve` para CSR, `npm run dev:ssr` para SSR).  
- [ ] ¿`main.server.ts` **pasa el `BootstrapContext`**?  
- [ ] ¿Incluiste `provideServerRendering()` en la config de servidor?  
- [ ] ¿`package.json` tiene los scripts `dev:ssr`, `build:ssr`, `serve:ssr`?  
- [ ] ¿`angular.json` contiene los targets `server`, `serve-ssr`, `build-ssr`?  
- [ ] ¿Borraste `.angular/cache` tras cambios de arranque?  

---

## 13) Conclusiones
- En Angular 20 la **configuración standalone** simplifica el arranque, pero SSR requiere **dos entradas** (cliente/servidor) y el **`BootstrapContext`** explícito.
- Si empiezas, **CSR** es más simple. Activa **SSR** cuando necesites SEO, prerender o mejor TTFB.
- Esta plantilla te permite alternar entre CSR y SSR sin caer en NG0401.
