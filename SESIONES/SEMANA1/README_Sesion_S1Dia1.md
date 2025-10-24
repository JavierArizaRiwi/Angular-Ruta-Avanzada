# Entrenamiento Angular – **Día 1** (Standalone + Vite)  
_Guía completa paso a paso, con imports, scripts y troubleshooting_

> Objetivo del día: **crear, ejecutar y depurar** un proyecto Angular **standalone** (sin NgModules) con **Vite**, entendiendo el arranque (bootstrap), rutas, `HttpClient` y cómo evitar errores típicos como **NG0401** y “**ENOSPC: file watchers**”.

---

## 0) Requisitos previos

- **Node.js 18+** (recomendado LTS):  
  ```bash
  node -v
  npm -v
  ```
- **Angular CLI 17/18/19/20** (cualquiera reciente):  
  ```bash
  npm i -g @angular/cli
  ng version
  ```

> Si usas Linux/WSL y abres muchos proyectos, mira la sección **Troubleshooting** para ampliar watchers.

---

## 1) Crear el proyecto (CSR con Vite)

```bash
ng new mi-primer-proyecto --ssr=false --routing=true --style=css
cd mi-primer-proyecto
```

La CLI generará una app **standalone** con **router** y **Vite**.

**Scripts en `package.json`** (la CLI los añade automáticamente):
```json
{
  "scripts": {
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test"
  }
}
```

**Ejecutar en desarrollo:**
```bash
npm start
# Abre http://localhost:4200
```

---

## 2) Estructura mínima (standalone)

```
src/
  app/
    app.component.ts
    app.component.html
    app.routes.ts
    app.config.ts
  main.ts
```

- **main.ts** hace el `bootstrapApplication`.
- **app.config.ts** registra **router** y **HttpClient** (proveedores).
- **app.routes.ts** define rutas.
- **app.component.ts** es standalone e importa `RouterOutlet` para renderizar rutas.

### 2.1 `main.ts`
```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err));
```

### 2.2 `app.config.ts` (proveedores globales)
```ts
import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),      // logs de errores del navegador (opcional)
    provideZoneChangeDetection({ eventCoalescing: true }), // perf opcional
    provideRouter(routes),                     // rutas
    provideHttpClient()                        // HttpClient para toda la app
  ]
};
```

> Alternativa “zoneless”: `provideZonelessChangeDetection()` si dominas CD sin zone.

### 2.3 `app.routes.ts` (rutas)
```ts
import { Routes } from '@angular/router';
import { AppComponent } from './app.component'; // solo si vas a usarlo en ruta raíz (no necesario)

export const routes: Routes = [
  { path: '', loadComponent: () => import('./usuario/usuario.component').then(m => m.UsuarioComponent) },
  // { path: '**', loadComponent: () => import('./not-found/not-found.component').then(m => m.NotFoundComponent) }
];
```

### 2.4 `app.component.ts` (host + router-outlet)
```ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  template: `
    <nav class="nav">
      <a routerLink="/" routerLinkActive="active" [routerLinkActiveOptions]="{ exact: true }">Inicio</a>
    </nav>
    <router-outlet></router-outlet>
  `,
  styles: [`.nav{display:flex;gap:12px;margin-bottom:16px}`]
})
export class AppComponent {}
```

---

## 3) Crear el **feature** Usuario (component + service)

### 3.1 Componente `UsuarioComponent`
Crea carpeta `src/app/usuario/` y dentro dos archivos:

**`usuario.component.ts`**
```ts
// Importaciones necesarias para el componente Usuario
import { Component, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';      // [(ngModel)]
import { CommonModule } from '@angular/common';    // *ngIf, *ngFor
import { UsuarioService } from '../service/usuario.service';

/**
 * Componente para gestionar usuarios en la aplicación
 * Permite listar y agregar usuarios utilizando el servicio UsuarioService
 */
@Component({
  selector: 'app-usuario',
  standalone: true, // Necesario para que 'imports' funcione en componentes independientes
  imports: [FormsModule, CommonModule],
  templateUrl: './usuario.html',
  styleUrls: ['./usuario.css'], // 👈 plural correcto
})
export class UsuarioComponent implements OnInit {
  /** Nombre del usuario a agregar (enlazado al formulario) */
  nombre = 'Coder';

  /** Lista de usuarios obtenida desde la API */
  listUsuarios: any[] = [];

  /** Inyección del servicio UsuarioService para operaciones CRUD */
  constructor(private svc: UsuarioService) {}

  /** Método del ciclo de vida que se ejecuta al inicializar el componente */
  ngOnInit(): void {
    this.listarUsuarios();
  }

  agregarUsuario() {
    const nuevo = { nombre: this.nombre?.trim() };
    if (!nuevo.nombre) { alert('El nombre es requerido'); return; }

    this.svc.create(nuevo).subscribe({
      next: () => this.listarUsuarios(),      // recarga después de crear
      error: (e) => console.error('Error al crear usuario:', e),
    });
  }

  listarUsuarios() {
    this.svc.getAll().subscribe({
      next: (usuarios) => {
        console.log('Usuarios obtenidos:', usuarios);
        this.listUsuarios = Array.isArray(usuarios) ? usuarios : (usuarios?.content ?? []);
      },
      error: (e) => console.error('Error al listar usuarios:', e),
    });
  }
}
```

**`usuario.html`**
```html
<h2>Gestión de Usuarios</h2>

<input [(ngModel)]="nombre" name="nombre" placeholder="Nombre" />
<button (click)="agregarUsuario()">Agregar</button>
<button (click)="listarUsuarios()">Listar</button>

<ul *ngIf="listUsuarios.length > 0; else noData">
  <li *ngFor="let u of listUsuarios">{{ u.nombre }}</li>
</ul>

<ng-template #noData>
  <p>No hay usuarios registrados.</p>
</ng-template>
```

**`usuario.css`**
```css
h2{margin-bottom:12px}
input{margin-right:8px}
```

### 3.2 Servicio `UsuarioService`
Crea `src/app/service/usuario.service.ts`:

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface Usuario { id?: number; nombre: string; }

@Injectable({ providedIn: 'root' })
export class UsuarioService {
  // Usa proxy en dev o habilita CORS en el backend:
  // - con proxy: '/api/v1/usuario'
  // - sin proxy: 'http://localhost:8080/api/v1/usuario'
  private readonly apiUrl = '/api/v1/usuario';

  constructor(private http: HttpClient) {}

  getAll(): Observable<Usuario[]> {
    return this.http.get<Usuario[]>(this.apiUrl);
  }
  create(usuario: Usuario): Observable<Usuario> {
    return this.http.post<Usuario>(this.apiUrl, usuario);
  }
}
```

---

## 4) (Opcional) Proxy Angular para evitar CORS en dev

Crea `proxy.conf.json` en la raíz del proyecto:
```json
{
  "/api": {
    "target": "http://localhost:8080",
    "secure": false,
    "changeOrigin": true,
    "logLevel": "debug"
  }
}
```

Lanza con:
```bash
ng serve --proxy-config proxy.conf.json
```

Y en el servicio usa rutas relativas: `'/api/v1/usuario'`.

---

## 5) Troubleshooting esencial

### 5.1 `NG0401: Missing Platform / NgFor not found`
- Asegúrate de **standalone: true** y en el componente **imports: [CommonModule, FormsModule]**.
- Alternativa moderna: usa `@for`/`@if` (Angular 17+) y no necesitas `CommonModule`.

### 5.2 ENOSPC: System limit for file watchers
Linux alcanzó el límite de watchers:
```bash
sudo sysctl fs.inotify.max_user_watches=524288
sudo sysctl fs.inotify.max_user_instances=1024
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
echo fs.inotify.max_user_instances=1024 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 5.3 CORS / 403 en POST
- Usa **proxy Angular** (sección 4) **o** habilita CORS/CSRF en Spring Security.

### 5.4 500 – Tabla “usuario” no encontrada (cuando back es H2)
- En Spring: `spring.jpa.hibernate.ddl-auto=update`
- Ver consola H2 y que la entidad tenga `@GeneratedValue(strategy=IDENTITY)` con `Long id`.

---

## 6) (Opcional) SSR con Vite

Para añadir SSR en un proyecto nuevo:
```bash
ng new app-ssr --ssr=true
```
Revisa que existan `main.server.ts`, `app.config.server.ts` y scripts adicionales. Si ves errores como **NG0401** en SSR, confirma que los imports server estén correctos (p. ej., no usar APIs del navegador en server).

---

## 7) Prueba final

1. Arranca el backend (si aplica).  
2. `npm start` (o `ng serve --proxy-config proxy.conf.json`)  
3. Ve a **/usuario** (ruta raíz en esta guía).  
4. Crea usuarios y verifica en la consola que se listan.

¡Día 1 completado! 