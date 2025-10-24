# Entrenamiento Angular – **Día 3: Servicios e HttpClient**
_Guía paso a paso con imports, configuración standalone y ejemplos comentados_

> Objetivos del día
>
> - Comprender qué son los **servicios** y por qué se usan en Angular.
> - Crear servicios **inyectables** para encapsular lógica de datos.
> - Configurar y usar **HttpClient** para consumir APIs REST (GET/POST/PUT/DELETE).
> - Manejar errores básicos con **RxJS** y `subscribe`/`catchError`.
> - Entender el **ciclo de vida** y la **inyección de dependencias** (DI).

---

## 0) Conceptos clave

- Un **servicio** es una clase reusable, sin UI, que concentra lógica de negocio (por ejemplo, llamadas HTTP).
- Angular usa **Dependency Injection (DI)** para proveer instancias a componentes (por constructor).
- `HttpClient` expone métodos (`get`, `post`, `put`, `delete`) que devuelven **Observables** (RxJS).

---

## 1) Configuración global — Standalone (Angular 17–20)

En proyectos **standalone** no hay `AppModule`. Debes **proveer** Router y HttpClient en `app.config.ts` y arrancar la app en `main.ts`.

### 1.1 `app.config.ts` — Proveedores globales
```ts
// app/app.config.ts
import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';

/**
 * Configuración global de la aplicación (standalone).
 * - Router: navegación entre pantallas.
 * - HttpClient: peticiones HTTP disponibles en toda la app.
 * - Error listeners y coalescing: utilidades opcionales.
 */
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),                    // (opcional) Registro de errores del navegador
    provideZoneChangeDetection({ eventCoalescing: true }),   // (opcional) Optimización de CD
    provideRouter(routes),                                   // Router
    provideHttpClient()                                      // HttpClient disponible globalmente
  ]
};
```

> **Alternativa clásica** (proyectos con `AppModule`): importar `HttpClientModule` en el `@NgModule.imports`.

### 1.2 `main.ts` — Bootstrap
```ts
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err));
```

### 1.3 `app.component.ts` — Host con Router
```ts
// app/app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive } from '@angular/router';

/**
 * Componente raíz (host) que aloja el <router-outlet>.
 * Al ser standalone, debemos declarar explícitamente sus imports.
 */
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  template: `
    <nav class="nav">
      <a routerLink="/usuario" routerLinkActive="active">Usuarios</a>
    </nav>
    <router-outlet></router-outlet>
  `,
  styles: [`.nav{display:flex;gap:12px;margin-bottom:16px}`]
})
export class AppComponent {}
```

---

## 2) Rutas de la app

Creamos una ruta para el **feature** `Usuario`.

```ts
// app/app.routes.ts
import { Routes } from '@angular/router';
import { UsuarioComponent } from './usuario/usuario.component';

export const routes: Routes = [
  { path: '', redirectTo: 'usuario', pathMatch: 'full' },
  { path: 'usuario', component: UsuarioComponent },
  // { path: '**', loadComponent: () => import('./not-found/not-found.component').then(m => m.NotFoundComponent) }
];
```

---

## 3) Servicio de datos (CRUD) — **con tipos y comentarios**

> Archivo: `src/app/service/usuario.service.ts`

```ts
// Importaciones necesarias para el servicio
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

/**
 * Interfaz que representa la estructura de un usuario
 * - Tipar los datos ayuda a detectar errores en tiempo de compilación
 */
export interface Usuario {
  id?: number;    // Identificador único (opcional al crear)
  nombre: string; // Nombre del usuario
}

/**
 * Servicio para gestionar operaciones CRUD de usuarios
 * - @Injectable({ providedIn: 'root' }) expone el servicio como singleton global
 * - Usa HttpClient para llamar a la API
 */
@Injectable({ providedIn: 'root' })
export class UsuarioService {
  /** URL base de la API para usuarios (puedes moverla a environments) */
  private readonly apiUrl = 'http://localhost:8080/api/v1/usuario';
  // Si usas proxy Angular en dev, deja '/api/v1/usuario' y configura proxy.conf.json

  /** Inyección de HttpClient para realizar peticiones HTTP */
  constructor(private http: HttpClient) {}

  /** Obtiene la lista de todos los usuarios */
  getAll(): Observable<Usuario[]> {
    return this.http.get<Usuario[]>(this.apiUrl);
  }

  /** Obtiene un usuario por su ID */
  getById(id: number): Observable<Usuario> {
    return this.http.get<Usuario>(`${this.apiUrl}/${id}`);
  }

  /** Crea un nuevo usuario */
  create(usuario: Usuario): Observable<Usuario> {
    return this.http.post<Usuario>(this.apiUrl, usuario);
  }

  /** Actualiza un usuario existente */
  update(id: number, usuario: Usuario): Observable<Usuario> {
    return this.http.put<Usuario>(`${this.apiUrl}/${id}`, usuario);
  }

  /** Elimina un usuario por su ID */
  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

> **Por qué estas importaciones**:
> - `@angular/core` → `Injectable` para registrar el servicio en el DI container.
> - `@angular/common/http` → `HttpClient` para llamadas REST tipadas.
> - `rxjs` → `Observable` para trabajar asincronía con suscripciones.

---

## 4) Componente standalone que consume el servicio — **con imports explicados**

> Archivos: `src/app/usuario/usuario.component.ts`, `usuario.html`, `usuario.css`

### 4.1 `usuario.component.ts`
```ts
// Importaciones necesarias para el componente Usuario
import { Component, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';   // Requerido por [(ngModel)]
import { CommonModule } from '@angular/common'; // Requerido por *ngIf y *ngFor
import { UsuarioService, Usuario } from '../service/usuario.service';

/**
 * Componente para gestionar usuarios en la aplicación.
 * - Lista y agrega usuarios utilizando el UsuarioService.
 * - Demuestra Interpolación, Event Binding, Two-way Binding y directivas.
 */
@Component({
  selector: 'app-usuario',
  standalone: true,                     // Necesario para que 'imports' funcione
  imports: [FormsModule, CommonModule], // Habilita [(ngModel)], *ngIf, *ngFor en este componente
  templateUrl: './usuario.html',
  styleUrls: ['./usuario.css'],         // Nota: plural correcto
})
export class UsuarioComponent implements OnInit {
  /** Nombre del usuario a agregar (enlazado al input) */
  nombre = 'Coder';

  /** Lista de usuarios obtenida desde la API */
  listUsuarios: Usuario[] = [];

  /** Inyección del servicio UsuarioService para operaciones CRUD */
  constructor(private svc: UsuarioService) {}

  /** Ciclo de vida: carga la lista al iniciar */
  ngOnInit(): void {
    this.listarUsuarios();
  }

  /** Llama al servicio para crear un usuario y vuelve a listar */
  agregarUsuario(): void {
    const nuevo: Usuario = { nombre: this.nombre?.trim() };
    if (!nuevo.nombre) { alert('El nombre es requerido'); return; }

    this.svc.create(nuevo).subscribe({
      next: () => this.listarUsuarios(),
      error: (e) => console.error('Error al crear usuario:', e),
    });
  }

  /** Llama al servicio para obtener todos los usuarios */
  listarUsuarios(): void {
    this.svc.getAll().subscribe({
      next: (usuarios) => {
        console.log('Usuarios obtenidos:', usuarios);
        // Si tu API pagina (ej: {content: []}), adapta:
        this.listUsuarios = Array.isArray(usuarios) ? usuarios : (usuarios as any)?.content ?? [];
      },
      error: (e) => console.error('Error al listar usuarios:', e),
    });
  }
}
```

### 4.2 `usuario.html`
```html
<h2>Gestión de Usuarios</h2>

<!-- Two-way binding: sincroniza input ↔ propiedad 'nombre' -->
<input [(ngModel)]="nombre" name="nombre" placeholder="Nombre" />

<!-- Event binding: ejecuta métodos del componente -->
<button (click)="agregarUsuario()">Agregar</button>
<button (click)="listarUsuarios()">Listar</button>

<!-- *ngIf para mostrar la lista solo si hay elementos -->
<ul *ngIf="listUsuarios.length > 0; else noData">
  <li *ngFor="let u of listUsuarios">{{ u.nombre }}</li>
</ul>

<ng-template #noData>
  <p>No hay usuarios registrados.</p>
</ng-template>
```

### 4.3 `usuario.css`
```css
h2{margin-bottom:12px}
input{margin-right:8px}
button{margin-right:6px}
```

> **Por qué estas importaciones**:
> - `FormsModule` → habilita `[(ngModel)]` para two-way binding en inputs.
> - `CommonModule` → habilita directivas `*ngIf` y `*ngFor`.
> - `standalone: true` + `imports: [...]` → patrón moderno sin `NgModule`.

---

## 5) Manejo de errores con RxJS

Puedes manejar errores en el servicio (para centralizar) o en el componente (por caso).

### 5.1 En el servicio (centralizado)
```ts
import { catchError, throwError } from 'rxjs';

getAll(): Observable<Usuario[]> {
  return this.http.get<Usuario[]>(this.apiUrl).pipe(
    catchError(err => {
      console.error('[UsuarioService] getAll error', err);
      return throwError(() => new Error('Error al obtener usuarios'));
    })
  );
}
```

### 5.2 En el componente (por operación)
```ts
this.svc.create(nuevo).subscribe({
  next: () => this.listarUsuarios(),
  error: (e) => {
    if (e.status === 409) alert('Nombre duplicado.');
    else if (e.status === 403) alert('No autorizado (CORS/CSRF).');
    else alert('Error al crear usuario.');
    console.error(e);
  }
});
```

---

## 6) Environments y URLs

Centraliza URLs en `src/environments` (útil para cambiar host por entorno).

```ts
// environments/environment.ts
export const environment = {
  production: false,
  api: 'http://localhost:8080/api/v1'
};
```

```ts
// usuario.service.ts
private readonly apiUrl = `${environment.api}/usuario`;
```

> Recuerda configurar `fileReplacements` en `angular.json` para `environment.prod.ts`.

---

## 7) Evitar CORS en desarrollo (proxy Angular)

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
Lanza:
```bash
ng serve --proxy-config proxy.conf.json
```
Y usa rutas relativas en el servicio: `private readonly apiUrl = '/api/v1/usuario';`

---

## 8) Troubleshooting rápido

- **`*ngFor/*ngIf not found`** → falta `CommonModule` en `imports` del componente (o usa `@for/@if` en Angular 17+).
- **`Can’t bind to ngModel`** → falta `FormsModule` y `name="..."` en el input.
- **403 (Forbidden)** en POST → CORS/CSRF backend (usa proxy o habilita CORS en Spring Security).
- **409 (Conflict)** → registro duplicado según reglas del backend.
- **500 (Table not found / tipos)** → en H2/MySQL, ajusta entidad y `spring.jpa.hibernate.ddl-auto=update`.
- **ENOSPC watchers (Linux)** → aumenta límites de inotify (ver Día 1).

---

## 9) Ejercicio del día

1. Implementa `getById`, `update`, `delete` en UI (botones y formulario).  
2. Muestra errores legibles en la interfaz (ej. toast/alert) según código HTTP.  
3. Extra: crea un `UsuarioStoreService` (BehaviorSubject) para compartir estado entre componentes.

---

## 10) Resumen

- Los **servicios** encapsulan la lógica de datos y facilitan testeo y reutilización.
- `HttpClient` + `Observable` es el combo base para hablar con APIs REST.
- Con **standalone**, recuerda **proveer** Router/HttpClient en `app.config.ts` y **importar** `FormsModule`/`CommonModule` en cada componente que lo requiera.
- Dominar estos conceptos deja tu app lista para **interceptores**, **guards** y **comunicación entre componentes** (Día 4).

---

### Anexo A — Variante clásica con `HttpClientModule` (AppModule)
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```