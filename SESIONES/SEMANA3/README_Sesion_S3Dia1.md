# Entrenamiento Angular – Sesión 5: Routing, Feature Modules y Lazy Loading

## Objetivos
- Estructurar la aplicación por features.
- Configurar rutas principales, rutas hijas y lazy loading.
- Navegar con parámetros y query params.
- Usar RouterLink, RouterLinkActive y ActivatedRoute.

## Conceptos clave
- RouterModule.forRoot(routes) en AppRoutingModule.
- RouterModule.forChild(routes) en módulos de feature.
- Redirecciones y pathMatch.
- Parámetros de ruta (:id) y snapshot vs suscripción.
- Query params y fragment.
- Lazy loading con loadChildren y módulos separados.

## Preparación
Crear un proyecto o continuar con el existente. Se recomienda tener una carpeta `features/`.
Comandos sugeridos:
```bash
ng g module app-routing --flat --module=app
ng g module features/tareas --route tareas --module app-routing
ng g component features/tareas/pages/tarea-list
ng g component features/tareas/pages/tarea-detail
```

## AppRoutingModule básico
```ts
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', redirectTo: 'tareas', pathMatch: 'full' },
  {
    path: 'tareas',
    loadChildren: () =>
      import('./features/tareas/tareas.module').then(m => m.TareasModule)
  },
  { path: '**', redirectTo: 'tareas' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

## Módulo de feature con rutas hijas
```ts
// features/tareas/tareas-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { TareaListComponent } from './pages/tarea-list/tarea-list.component';
import { TareaDetailComponent } from './pages/tarea-detail/tarea-detail.component';

const routes: Routes = [
  { path: '', component: TareaListComponent },
  { path: ':id', component: TareaDetailComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class TareasRoutingModule {}
```

## Navegación en plantillas
```html
<nav>
  <a routerLink="/tareas" routerLinkActive="active" [routerLinkActiveOptions]="{ exact: true }">Listado</a>
  <a [routerLink]="['/tareas', 42]">Detalle 42</a>
</nav>
<router-outlet></router-outlet>
```

## Leer parámetros y query params
```ts
// features/tareas/pages/tarea-detail/tarea-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-tarea-detail',
  template: `
    <h3>Detalle de Tarea {{ id }}</h3>
    <p>Filtro: {{ filtro }}</p>
  `
})
export class TareaDetailComponent implements OnInit {
  id!: number;
  filtro?: string;

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    this.id = Number(this.route.snapshot.paramMap.get('id'));
    this.route.queryParamMap.subscribe(qp => this.filtro = qp.get('filtro') || '');
  }
}
```

## Ejercicio
1. Mover el listado y detalle de tareas a un módulo de feature con rutas hijas.
2. Hacer que el botón “ver detalle” navegue a `/tareas/:id?filtro=...`.
3. Implementar lazy loading con `loadChildren`.

## Resultado esperado
- Estructura por features con rutas perezosas.
- URL con parámetros funcionando.
- Menú con estilo activo mediante RouterLinkActive.