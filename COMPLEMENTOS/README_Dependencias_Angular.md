# Guía Práctica: Angular Material y Dependencias Importantes (Angular 17+ standalone & Vite)

> **Objetivo:** Instalar, configurar y usar **Angular Material** (con **Angular 17+**, standalone) junto a dependencias clave (CDK, Forms, Router, HttpClient, i18n/locale, ESLint, etc.), con ejemplos **listos para copiar**: **botones**, **formularios reactivos**, **tabla con paginación/ordenación/filtro**, **diálogo**, **snackbar**, **datepicker** con **locale es-ES**, **temas personalizados** y **buenas prácticas**.

---

## 1) Prerrequisitos y contexto técnico

- **Angular**: 17 o superior (standalone por defecto, builder con **Vite**).
- **Node.js**: >= 18 LTS (recomendado).
- Proyecto creado con CLI:
  ```bash
  ng new mi-app --standalone
  cd mi-app
  ```
- Si tu proyecto es más antiguo (módulos), los ejemplos siguen funcionando con mínimos ajustes (usar `@NgModule` y `imports` en módulos).

> **Nota**: Desde Angular 17, el CLI usa **Vite** como dev server/bundler por defecto. La integración de Material es transparente (no se requieren configuraciones especiales de Webpack).

---

## 2) Instalación rápida: Angular Material + Animations + CDK + Icons

### 2.1 Opción recomendada (automatizada)
```bash
ng add @angular/material
```
Esto:
- Instala `@angular/material` y `@angular/cdk`.
- Activa **BrowserAnimationsModule** (o en standalone, registra el provider `provideAnimations()`).
- Agrega estilos de tema y **Material Icons** a tu proyecto.

### 2.2 Opción manual (si no usas `ng add`)
```bash
npm i @angular/material @angular/cdk
```
En **main.ts** (standalone), registra animaciones:
```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideAnimations } from '@angular/platform-browser/animations';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [provideAnimations()]
});
```
En **index.html**, agrega **Material Icons** (si no lo hizo `ng add`):
```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```
> Alternativa sin CDN: descargar la fuente e incluirla en `assets/`.

---

## 3) Dependencias importantes (Angular 17+)

| Dependencia | ¿Para qué? | Notas |
|---|---|---|
| `@angular/material` | Componentes UI listos (botones, formularios, tablas, diálogos, etc.) | Usa tema + tipografía + CDK |
| `@angular/cdk` | Utilidades base (overlay, a11y, drag-drop, cdk-table) | Muchas features de Material dependen del CDK |
| `@angular/forms` | Formularios reactivos / template-driven | Para Material, preferir **Reactive Forms** |
| `@angular/router` | Navegación SPA, lazy routes | Guardias, resolvers, standalone routes |
| `@angular/common/http` | `HttpClient`, interceptores, JSON | Integrar con snackbars para errores |
| `@angular/platform-browser/animations` | Animaciones (ripple, transitions) | `provideAnimations()` |
| **ESLint** (`@angular-eslint/*`) | Linter y reglas Angular/TS | Reemplaza TSLint (deprecado) |
| **Prettier** | Formateo de código | Integrarlo con ESLint evita conflictos |
| `@ngx-translate/core` (opcional) | i18n en runtime | Útil junto con locales del `DateAdapter` |
| `zod`/`yup` (opcional) | Validaciones schema-driven | Complementa Bean Validation del backend |

Instalación (linter/formatter opcional):
```bash
ng add @angular-eslint/schematics
npm i -D prettier eslint-config-prettier eslint-plugin-prettier
```

---

## 4) Theming: SCSS con Angular Material (Angular 17+)

### 4.1 Estructura de tema (SCSS)
Crea **`src/styles.scss`** (o el que uses globalmente) con un tema personalizado:

```scss
/* styles.scss */
@use '@angular/material' as mat;

/* Tipografías opcionales */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

:root {
  --app-padding: 16px;
}

/* Paletas base */
$primary: mat.define-palette(mat.$indigo-palette, 600);
$accent:  mat.define-palette(mat.$pink-palette, 500, 300, 700);
$warn:    mat.define-palette(mat.$red-palette);

/* Tema claro */
$light-theme: mat.define-light-theme((
  color: (
    primary: $primary,
    accent: $accent,
    warn: $warn,
  ),
  typography: mat.define-typography-config($font-family: 'Inter, Roboto, "Helvetica Neue", Arial, sans-serif'),
  density: 0,
));

/* Tema oscuro (opcional) */
$dark-theme: mat.define-dark-theme((
  color: (
    primary: $primary,
    accent: $accent,
    warn: $warn,
  ),
  typography: mat.define-typography-config($font-family: 'Inter, Roboto, "Helvetica Neue", Arial, sans-serif'),
  density: 0,
));

/* Aplica tema claro por defecto a todos los componentes */
@include mat.all-component-themes($light-theme);

/* Clase para activar tema oscuro */
.dark-theme {
  @include mat.all-component-colors($dark-theme);
}
```

> **Tip**: alterna temas agregando/removiendo la clase `dark-theme` en el `<body>` con un `ThemeService` sencillo.

---

## 5) Angular Material: imports en standalone

En Angular 17+, los componentes son **standalone**. Importa módulos Material **directo en el componente** o en un **layout shell**.

Ejemplo (shell `AppComponent`):
```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatListModule } from '@angular/material/list';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    RouterOutlet,
    MatToolbarModule, MatSidenavModule, MatListModule,
    MatButtonModule, MatIconModule
  ],
  template: `
  <mat-sidenav-container>
    <mat-sidenav mode="side" opened>
      <mat-nav-list>
        <a mat-list-item routerLink="/">Inicio</a>
        <a mat-list-item routerLink="/usuarios">Usuarios</a>
      </mat-nav-list>
    </mat-sidenav>
    <mat-sidenav-content>
      <mat-toolbar color="primary">
        <button mat-icon-button (click)="toggle()"><mat-icon>menu</mat-icon></button>
        <span>Mi App</span>
      </mat-toolbar>
      <main style="padding: var(--app-padding)">
        <router-outlet></router-outlet>
      </main>
    </mat-sidenav-content>
  </mat-sidenav-container>
  `
})
export class AppComponent {
  toggle() {/* noop ejemplo */}
}
```

---

## 6) Formularios Reactivos + Material (con validación)

```ts
// usuarios-form.component.ts
import { Component, inject } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, Validators } from '@angular/forms';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatIconModule } from '@angular/material/icon';
import { MatDatepickerModule } from '@angular/material/datepicker';
import { MatNativeDateModule } from '@angular/material/core';

@Component({
  selector: 'app-usuarios-form',
  standalone: true,
  imports: [
    ReactiveFormsModule,
    MatFormFieldModule, MatInputModule, MatButtonModule, MatCardModule, MatIconModule,
    MatDatepickerModule, MatNativeDateModule
  ],
  template: `
  <mat-card>
    <h2>Crear Usuario</h2>

    <form [formGroup]="form" (ngSubmit)="submit()">
      <mat-form-field appearance="outline" style="width: 100%;">
        <mat-label>Nombre</mat-label>
        <input matInput formControlName="nombre" placeholder="Ej: Javier Ariza">
        <mat-error *ngIf="form.controls.nombre.hasError('required')">
          El nombre es obligatorio
        </mat-error>
        <mat-error *ngIf="form.controls.nombre.hasError('minlength')">
          Mínimo 3 caracteres
        </mat-error>
      </mat-form-field>

      <mat-form-field appearance="outline" style="width: 100%;">
        <mat-label>Email</mat-label>
        <input matInput formControlName="email" type="email" placeholder="correo@dominio.com">
        <mat-error *ngIf="form.controls.email.hasError('email')">
          Email inválido
        </mat-error>
      </mat-form-field>

      <mat-form-field appearance="outline">
        <mat-label>Fecha de nacimiento</mat-label>
        <input matInput [matDatepicker]="picker" formControlName="fechaNac">
        <mat-datepicker-toggle matSuffix [for]="picker"></mat-datepicker-toggle>
        <mat-datepicker #picker></mat-datepicker>
      </mat-form-field>

      <div style="margin-top: 12px;">
        <button mat-raised-button color="primary" type="submit" [disabled]="form.invalid">
          <mat-icon>save</mat-icon> Guardar
        </button>
      </div>
    </form>
  </mat-card>
  `
})
export class UsuariosFormComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    nombre: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.email]],
    fechaNac: [null]
  });

  submit() {
    if (this.form.valid) {
      console.log('Payload:', this.form.value);
    }
  }
}
```

### 6.1 Locale ES para Datepicker
En **main.ts** agrega providers:
```ts
import { MAT_DATE_LOCALE } from '@angular/material/core';

bootstrapApplication(AppComponent, {
  providers: [
    provideAnimations(),
    { provide: MAT_DATE_LOCALE, useValue: 'es-ES' }
  ]
});
```

> Si usas `MatMomentDateModule`, instala `moment` y cambia el adapter. Para producción, `MatNativeDateModule` suele ser suficiente.

---

## 7) Tabla con MatTableDataSource + Paginación + Ordenación + Filtro

```ts
// usuarios-table.component.ts
import { Component, ViewChild } from '@angular/core';
import { MatTableDataSource, MatTableModule } from '@angular/material/table';
import { MatPaginator, MatPaginatorModule } from '@angular/material/paginator';
import { MatSort, MatSortModule } from '@angular/material/sort';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatIconModule } from '@angular/material/icon';
import { MatCardModule } from '@angular/material/card';

interface Usuario {
  id: number;
  nombre: string;
  email: string;
}

const DATA: Usuario[] = [
  { id: 1, nombre: 'Javier Ariza', email: 'javier@ejemplo.com' },
  { id: 2, nombre: 'María Gomez', email: 'maria@ejemplo.com' },
  // ...
];

@Component({
  selector: 'app-usuarios-table',
  standalone: true,
  imports: [
    MatTableModule, MatPaginatorModule, MatSortModule,
    MatFormFieldModule, MatInputModule, MatIconModule, MatCardModule
  ],
  template: `
  <mat-card>
    <h2>Usuarios</h2>

    <mat-form-field appearance="outline">
      <mat-label>Filtrar</mat-label>
      <input matInput (keyup)="applyFilter($event)" placeholder="Buscar por nombre o email">
      <mat-icon matSuffix>search</mat-icon>
    </mat-form-field>

    <table mat-table [dataSource]="dataSource" matSort class="mat-elevation-z2" style="width: 100%;">

      <ng-container matColumnDef="id">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>ID</th>
        <td mat-cell *matCellDef="let row">{{ row.id }}</td>
      </ng-container>

      <ng-container matColumnDef="nombre">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Nombre</th>
        <td mat-cell *matCellDef="let row">{{ row.nombre }}</td>
      </ng-container>

      <ng-container matColumnDef="email">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Email</th>
        <td mat-cell *matCellDef="let row">{{ row.email }}</td>
      </ng-container>

      <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
      <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>
    </table>

    <mat-paginator [pageSize]="5" [pageSizeOptions]="[5, 10, 20]"></mat-paginator>
  </mat-card>
  `
})
export class UsuariosTableComponent {
  displayedColumns = ['id', 'nombre', 'email'];
  dataSource = new MatTableDataSource<Usuario>(DATA);

  @ViewChild(MatPaginator) paginator!: MatPaginator;
  @ViewChild(MatSort) sort!: MatSort;

  ngAfterViewInit() {
    this.dataSource.paginator = this.paginator;
    this.dataSource.sort = this.sort;
  }

  applyFilter(event: Event) {
    const value = (event.target as HTMLInputElement).value;
    this.dataSource.filter = value.trim().toLowerCase();
  }
}
```

---

## 8) Diálogo (MatDialog) + Snackbar (MatSnackBar)

```ts
// confirm-dialog.component.ts
import { Component, inject } from '@angular/core';
import { MatDialogModule, MatDialogRef } from '@angular/material/dialog';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-confirm-dialog',
  standalone: true,
  imports: [MatDialogModule, MatButtonModule],
  template: `
    <h2 mat-dialog-title>Confirmar</h2>
    <mat-dialog-content>¿Deseas continuar?</mat-dialog-content>
    <mat-dialog-actions align="end">
      <button mat-button (click)="close(false)">Cancelar</button>
      <button mat-raised-button color="primary" (click)="close(true)">Aceptar</button>
    </mat-dialog-actions>
  `
})
export class ConfirmDialogComponent {
  dialogRef = inject(MatDialogRef<ConfirmDialogComponent>);
  close(result: boolean) { this.dialogRef.close(result); }
}
```

```ts
// acciones.component.ts
import { Component, inject } from '@angular/core';
import { MatDialog, MatDialogModule } from '@angular/material/dialog';
import { MatSnackBar, MatSnackBarModule } from '@angular/material/snack-bar';
import { MatButtonModule } from '@angular/material/button';
import { ConfirmDialogComponent } from './confirm-dialog.component';

@Component({
  selector: 'app-acciones',
  standalone: true,
  imports: [MatDialogModule, MatSnackBarModule, MatButtonModule, ConfirmDialogComponent],
  template: `
    <button mat-raised-button color="accent" (click)="abrirDialog()">Abrir diálogo</button>
  `
})
export class AccionesComponent {
  dialog = inject(MatDialog);
  snack = inject(MatSnackBar);

  async abrirDialog() {
    const ref = this.dialog.open(ConfirmDialogComponent);
    const ok = await ref.afterClosed().toPromise();
    if (ok) this.snack.open('Acción confirmada', 'Cerrar', { duration: 2000 });
  }
}
```

> **Nota**: Para `toPromise()` en Angular 17+, puedes usar `firstValueFrom(ref.afterClosed())` de `rxjs`.

---

## 9) HttpClient + Interceptor (errores → MatSnackBar)

```ts
// error.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { MatSnackBar } from '@angular/material/snack-bar';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const snack = inject(MatSnackBar);
  return next(req).pipe({
    error: (err) => {
      const msg = err?.error?.message ?? 'Error de red';
      snack.open(msg, 'Cerrar', { duration: 3000 });
      throw err;
    }
  } as any);
};
```

Registrar en **main.ts**:
```ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { errorInterceptor } from './app/core/error.interceptor';

bootstrapApplication(AppComponent, {
  providers: [
    provideAnimations(),
    provideHttpClient(withInterceptors([errorInterceptor]))
  ]
});
```

---

## 10) Router standalone (lazy + guards)

```ts
// app.routes.ts
import { Routes } from '@angular/router';
import { inject } from '@angular/core';
import { CanActivateFn } from '@angular/router';

const canActivateAuth: CanActivateFn = () => {
  const isLogged = true; // demo
  return isLogged;
};

export const routes: Routes = [
  { path: '', loadComponent: () => import('./home/home.component').then(m => m.HomeComponent) },
  {
    path: 'usuarios',
    canActivate: [canActivateAuth],
    loadComponent: () => import('./usuarios/usuarios-table.component').then(m => m.UsuariosTableComponent)
  },
  { path: '**', redirectTo: '' }
];
```
En **main.ts**:
```ts
import { provideRouter } from '@angular/router';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [
    provideAnimations(),
    provideRouter(routes)
  ]
});
```

---

## 11) Accesibilidad (a11y) y rendimiento

- Usa **roles ARIA** cuando proceda; Material ya cuida a11y, pero valida tus plantillas.
- Evita renderizar listas grandes sin **virtual scroll** (`@angular/cdk/scrolling`).
- Usa **ChangeDetectionStrategy.OnPush** en componentes de alto tráfico.
- Prefiere **Reactive Forms** y evita `[(ngModel)]` en formularios complejos.

---

## 12) Checklist de adopción en equipos

1. `ng add @angular/material` → tema + animations + icons.
2. Consolidar **styles.scss** con **tema claro/oscuro**.
3. Crear **shell** (toolbar + sidenav) con componentes standalone.
4. Estándar de **formularios reactivos** + validación + datepicker locale `es-ES`.
5. **Tabla** con `MatTableDataSource` + `MatPaginator` + `MatSort` + filtro.
6. **Diálogo de confirmación** y **snackbar** para feedback.
7. **HttpClient** con **interceptor** de errores.
8. Router **standalone** + **lazy** + **guards**.
9. ESLint + Prettier; reglas mínimas de accesibilidad y code style.
10. Documentar patrón de **imports** (en componente vs en shell) para evitar duplicaciones.

---

## 13) Errores comunes y solución

- **No aparecen estilos de Material** → Revisa que el **tema SCSS** esté cargado y que no haya conflictos de CSS globales.
- **El datepicker no abre** → Falta `MatDatepickerModule` y `MatNativeDateModule` o `provideAnimations()`.
- **Iconos no se ven** → Verifica `<link>` de Material Icons o fuentes locales en `assets/`.
- **Paginador no funciona** → Asigna `paginator` y `sort` en `ngAfterViewInit()`.
- **Standalone missed imports** → Asegúrate de listar todos los módulos Material en `imports` del componente standalone.

---

## 14) Snippets de productividad (copiar/pegar)

**Botón + icono:**
```html
<button mat-raised-button color="primary">
  <mat-icon>add</mat-icon> Nuevo
</button>
```

**Form field básico:**
```html
<mat-form-field appearance="outline">
  <mat-label>Buscar</mat-label>
  <input matInput placeholder="Texto...">
  <mat-icon matSuffix>search</mat-icon>
</mat-form-field>
```

**Card con acciones:**
```html
<mat-card>
  <mat-card-title>Título</mat-card-title>
  <mat-card-content>Contenido…</mat-card-content>
  <mat-card-actions align="end">
    <button mat-button>Cancelar</button>
    <button mat-raised-button color="primary">Aceptar</button>
  </mat-card-actions>
</mat-card>
```

---

## 15) Referencias rápidas (mentales)

- **Material = UI** + tema + tipografía + a11y (sobre CDK).
- **Standalone**: importa lo que uses **en cada componente** o en **shell**.
- **Reactive Forms** > template-driven en apps complejas.
- **Interceptors** + **snackbar** = feedback y manejo de errores consistente.
- **Locales**: `MAT_DATE_LOCALE` y, si aplica, `registerLocaleData` de `@angular/common`.

---

