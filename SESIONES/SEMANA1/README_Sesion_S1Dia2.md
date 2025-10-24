# Entrenamiento Angular – **Día 2**: Data Binding y Directivas  
_Guía detallada con imports, ejemplos y comentarios_

> Objetivo del día: dominar los 4 tipos de **data binding** y las **directivas** principales, integrándolos en un componente funcional y **standalone**. Configuraremos correctamente `FormsModule`, `CommonModule`, `Router` y `HttpClient` para que todo compile sin NG errores.

---

## 0) Recordatorio de configuración global (standalone)

### `app.config.ts` (proveedores globales)
```ts
import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient() // Habilita HttpClient para toda la app
  ]
};
```

### `app.component.ts` (host con router)
```ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  template: `
    <nav class="nav">
      <a routerLink="/usuario" routerLinkActive="active">Usuarios</a>
    </nav>
    <router-outlet></router-outlet>
  `
})
export class AppComponent {}
```

> Si no usas rutas, puedes omitir `Router*`, pero mantenemos este setup porque es el habitual.

---

## 1) Los 4 tipos de Data Binding

| Tipo | Sintaxis | Dirección de datos | Descripción |
|------|----------|--------------------|-------------|
| Interpolación | `{{ variable }}` | Componente → Vista | Muestra valores en HTML |
| Property Binding | `[propiedad]="variable"` | Componente → Vista | Enlaza atributos HTML a datos |
| Event Binding | `(evento)="método($event)"` | Vista → Componente | Responde a eventos del usuario |
| Two-Way Binding | `[(ngModel)]="variable"` | Bidireccional | Sincroniza vista y modelo |

**Reglas rápidas**
- `[(ngModel)]` requiere **FormsModule** en el **componente standalone**.
- `*ngIf` / `*ngFor` requieren **CommonModule** (o usa `@if`/`@for` en Angular 17+).

---

## 2) Interpolación

```ts
nombre = 'Coder';
contador = 0;
```

```html
<h1>Hola {{ nombre }}</h1>
<p>Clicks: {{ contador }}</p>
```

---

## 3) Property Binding

```ts
img = 'https://angular.io/assets/images/logos/angular/angular.png';
alt = 'Logo de Angular';
deshabilitado = false;
```

```html
<img [src]="img" [alt]="alt" width="180">
<button [disabled]="deshabilitado">Botón</button>
```

---

## 4) Event Binding

```ts
contador = 0;
sumar() { this.contador++; }
```

```html
<button (click)="sumar()">Sumar</button>
```

Con `$event`:
```html
<input (input)="onInput($event)">
```

---

## 5) Two-Way Binding ([(ngModel)])

```ts
modelo = '';
```

```html
<input [(ngModel)]="modelo" name="modelo" placeholder="Escribe algo" />
<p>Valor: {{ modelo }}</p>
```

> **Importante**: el `input` debe tener `name="..."` para que Angular forms no advierta.

---

## 6) Directivas

### 6.1 Estructurales
- `*ngIf` y `*ngFor` → requieren `CommonModule` en `imports` del componente.

```ts
visible = true;
items = ['Angular', 'React', 'Vue'];
```

```html
<p *ngIf="visible">Texto visible</p>
<ul>
  <li *ngFor="let i of items; index as idx">{{ idx + 1 }} - {{ i }}</li>
</ul>
```

**Alternativa moderna (Angular 17+):**
```html
@if (visible) {
  <p>Texto visible</p>
}
@for (i of items; track i) {
  <li>{{ i }}</li>
}
```

### 6.2 De atributo
```ts
hayError = true;
```

```html
<div [ngClass]="{ error: hayError, ok: !hayError }">Estado</div>
<div [ngStyle]="{ color: hayError ? 'red' : 'green' }">Color dinámico</div>
```

---

## 7) Ejemplo práctico — **UsuarioComponent** (completo y documentado)

> Archivos: `src/app/usuario/usuario.component.ts`, `usuario.html`, `usuario.css`

**`usuario.component.ts`**
```ts
import { Component, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';      // Necesario para [(ngModel)]
import { CommonModule } from '@angular/common';    // Necesario para *ngIf y *ngFor
import { UsuarioService } from '../service/usuario.service';

/**
 * Componente para gestionar usuarios (listar y crear).
 * Aplica Interpolación, Event Binding, Two-way Binding y directivas.
 */
@Component({
  selector: 'app-usuario',
  standalone: true,
  imports: [FormsModule, CommonModule],
  templateUrl: './usuario.html',
  styleUrls: ['./usuario.css'],
})
export class UsuarioComponent implements OnInit {
  nombre = 'Coder';
  listUsuarios: any[] = [];
  visible = true; // para *ngIf

  constructor(private svc: UsuarioService) {}

  ngOnInit(): void {
    this.listarUsuarios();
  }

  agregarUsuario() {
    const nuevo = { nombre: this.nombre?.trim() };
    if (!nuevo.nombre) { alert('El nombre es requerido'); return; }

    this.svc.create(nuevo).subscribe({
      next: () => this.listarUsuarios(),
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
<button (click)="visible = !visible">Mostrar/Ocultar</button>

<ul *ngIf="visible && listUsuarios.length > 0; else noData">
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
button{margin-right:6px}
```

---

## 8) Servicio HTTP (`UsuarioService`)

> Archivo: `src/app/service/usuario.service.ts`

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface Usuario { id?: number; nombre: string; }

@Injectable({ providedIn: 'root' })
export class UsuarioService {
  private readonly apiUrl = '/api/v1/usuario'; // usa proxy en dev o habilita CORS

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

## 9) Rutas y navegación

> Archivo: `src/app/app.routes.ts`
```ts
import { Routes } from '@angular/router';
import { UsuarioComponent } from './usuario/usuario.component';

export const routes: Routes = [
  { path: '', redirectTo: 'usuario', pathMatch: 'full' },
  { path: 'usuario', component: UsuarioComponent },
];
```

> Host con `<router-outlet>` ya lo definimos en el **0)**.

---

## 10) Proxy Angular (opcional) para evitar CORS

**`proxy.conf.json`**
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

Lanzar:
```bash
ng serve --proxy-config proxy.conf.json
```

En el servicio usa `'/api/v1/usuario'`.

---

## 11) Errores comunes (y su solución rápida)

- **NgFor/NgIf not found** → falta `CommonModule` en `imports` (o usa `@for/@if`).  
- **Can’t bind to ngModel** → falta `FormsModule` y `name="..."` en el `<input>`.  
- **403 (Forbidden) en POST** → CORS/CSRF backend (usa proxy o ajusta Spring Security).  
- **500 (Table not found)** → habilita `spring.jpa.hibernate.ddl-auto=update` o crea la tabla.  
- **ENOSPC watchers** (Linux) → aumenta límites `inotify` (ver Día 1, §5.2).

---

## 12) Tarea rápida

- Añade `[ngClass]` para marcar usuarios con nombre largo (>10 chars).  
- Cambia `*ngFor` por `@for` y compara rendimiento.  
- Extra: crea un pipe para capitalizar `nombre`.

¡Día 2 completado! 