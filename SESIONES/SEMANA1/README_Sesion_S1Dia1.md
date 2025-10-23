
# Entrenamiento Angular – Día 1: Introducción Moderna a Angular

## Objetivos
- Comprender la arquitectura moderna de Angular (componentes, configuración y renderizado).
- Instalar Angular CLI y crear el primer proyecto con Vite.
- Analizar la estructura real de carpetas generadas.
- Entender la diferencia entre arranque de cliente (Client Bootstrap) y servidor (SSR Bootstrap).
- Crear y renderizar un componente con datos dinámicos.

---

## ¿Qué es Angular?
Angular es un framework frontend mantenido por Google, basado en TypeScript y enfocado en la creación de aplicaciones SPA (Single Page Applications).  
A partir de Angular 17+, el framework usa un enfoque standalone + Vite que simplifica la configuración y el rendimiento.

Sus pilares principales son:
- Componentes: piezas visuales y lógicas reutilizables.
- Servicios: clases para compartir datos o lógica entre componentes.
- Configuraciones (`app.config.ts`, `app.config.server.ts`): reemplazan el antiguo `app.module.ts`.
- Bootstrap Application: nueva forma de iniciar la app sin necesidad de módulos.

---

## Instalación y verificación del entorno

1. Verifica Node.js y npm:
   ```bash
   node -v
   npm -v
   ```
   Se recomienda Node 18 o superior.

2. Instala Angular CLI:
   ```bash
   npm install -g @angular/cli
   ng version
   ```

3. Crea un nuevo proyecto con Vite (modo cliente):
   ```bash
   ng new mi-proyecto --ssr=false
   cd mi-proyecto
   npm run start
   ```

   Si deseas habilitar SSR (Server Side Rendering):
   ```bash
   ng new mi-proyecto --ssr
   npm run dev:ssr
   ```

4. Abre el navegador:
   ```
   http://localhost:4200
   ```

---

## Diferencia entre Vite y Servidor Angular Universal

| Característica | Con Vite (Cliente) | Con SSR (Servidor) |
|----------------|--------------------|---------------------|
| Renderizado | En el navegador | En el servidor (Node) |
| Velocidad de desarrollo | Más rápida, recarga instantánea | Más lenta, requiere build y contexto |
| SEO (indexación) | Limitado (contenido generado en cliente) | Completo (contenido prerenderizado) |
| Archivos clave | `main.ts`, `app.config.ts` | `main.server.ts`, `app.config.server.ts` |
| Contexto `BootstrapContext` | No requerido | Obligatorio |
| Ideal para | Apps SPA estándar | Apps con SEO o contenido dinámico público |

---

## Estructura de carpetas en Angular 20

| Carpeta / Archivo | Descripción |
|--------------------|-------------|
| src/app/ | Contiene los componentes, servicios y rutas. |
| app.component.ts | Lógica del componente raíz. |
| app.component.html | Plantilla HTML del componente raíz. |
| main.ts | Punto de entrada del cliente (CSR). |
| main.server.ts | Punto de entrada del servidor (SSR). |
| app.config.ts | Configuración de proveedores del cliente. |
| app.config.server.ts | Configuración del servidor SSR. |
| angular.json | Configura la compilación, scripts y estilos. |
| vite.config.ts | Configuración del empaquetador Vite. |

---

## Explicación de los archivos clave

### main.ts
Punto de entrada de la aplicación cliente.
```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { App } from './app/app';

bootstrapApplication(App, appConfig)
  .catch(err => console.error(err));
```
Se ejecuta solo en el navegador.

---

### app.config.ts
Configura el entorno del cliente:
```ts
import { ApplicationConfig } from '@angular/core';
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

### main.server.ts
Se usa solo en proyectos SSR, recibe el BootstrapContext para evitar el error NG0401.
```ts
import { bootstrapApplication, BootstrapContext } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfigServer } from './app/app.config.server';

export default function bootstrap(context: BootstrapContext) {
  return bootstrapApplication(App, appConfigServer, context);
}
```

---

### app.config.server.ts
Configura el renderizado en servidor.
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

## Crear el primer componente

```bash
ng generate component saludo
```

Esto crea una carpeta src/app/saludo con:
- saludo.component.ts → lógica
- saludo.component.html → vista
- saludo.component.css → estilos

### Ejemplo básico:
```html
<h2>Bienvenido a Angular</h2>
<p>Este es mi primer componente.</p>
```

Usa el selector en app.component.html:
```html
<app-saludo></app-saludo>
```

---

## Ejercicio práctico

1. En saludo.component.ts:
```ts
export class SaludoComponent {
  nombre: string = "Coder";
}
```

2. En saludo.component.html:
```html
<input [(ngModel)]="nombre" placeholder="Escribe tu nombre">
<p>Hola {{ nombre }}!</p>
```

3. Importa FormsModule:
```ts
import { FormsModule } from '@angular/forms';
export const appConfig: ApplicationConfig = {
  providers: [importProvidersFrom(FormsModule)],
};
```

**Resultado:** El nombre se actualiza en tiempo real mediante two-way binding.

---

## Errores comunes y soluciones

| Error | Causa | Solución |
|--------|--------|----------|
| NG0401: Missing Platform | Falta pasar BootstrapContext en main.server.ts | Usa `bootstrapApplication(App, appConfigServer, context)` |
| ReferenceError: document is not defined | Código del navegador ejecutado en SSR | Usa `isPlatformBrowser()` o desactiva SSR |
| Module not found: vite | Falta de dependencias | Reinstala con `npm install` |
| Component not found | Selector mal escrito | Verifica el nombre en el HTML |

---

## Conclusiones

- Angular 20 unifica el desarrollo cliente y SSR mediante standalone y configuración directa.
- Para SSR es obligatorio el contexto BootstrapContext.
- Vite acelera el desarrollo, mientras que SSR mejora el SEO y el prerenderizado.
- Conocer los archivos y flujo de arranque evita errores NG0401 o conflictos de plataforma.