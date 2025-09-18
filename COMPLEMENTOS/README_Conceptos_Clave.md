# Directivas y Data Binding en Angular (Guía Práctica)

Este documento explica las **directivas principales de Angular** y las formas de **presentar datos** en plantillas mediante **interpolación** y distintos tipos de **data binding**. Incluye ejemplos, buenas prácticas y notas de rendimiento/seguridad.

---

## 1. Conceptos clave

- **Data Binding**: mecanismo para sincronizar datos entre la clase del componente (TypeScript) y la vista (HTML).
- **Directivas**: instrucciones que Angular aplica al DOM para modificar su estructura o apariencia.
  - **Estructurales**: cambian el árbol del DOM (`*ngIf`, `*ngFor`, `*ngSwitch`).
  - **De atributo**: cambian apariencia o comportamiento de elementos (`[ngClass]`, `[ngStyle]`, `NgModel`).

---

## 2. Formas de Data Binding

### 2.1 Interpolación (`{{ }}`)
Muestra valores de propiedades o expresiones simples en la vista.

```html
<h1>Hola {{ nombre }}</h1>
<p>El doble de 4 es {{ 4 * 2 }}</p>
<p>Fecha: {{ hoy | date:'longDate' }}</p>
```

**Regla**: usar expresiones **puras** (sin efectos secundarios).

### 2.2 Property Binding (`[prop]`)
Asigna valores de la clase a **propiedades** de elementos/Componentes/Directivas.

```html
<img [src]="urlImagen" [alt]="descripcion">
<button [disabled]="deshabilitado">Enviar</button>
```

### 2.3 Event Binding (`(evento)`)
Escucha **eventos** del DOM y ejecuta métodos del componente.

```html
<button (click)="guardar()">Guardar</button>
<input (keyup.enter)="buscar(termino.value)" #termino>
```

### 2.4 Two‑Way Binding (`[(ngModel)]`)
Combina property y event binding. Requiere `FormsModule`.

```html
<input [(ngModel)]="nombre" placeholder="Escribe tu nombre">
<p>Hola {{ nombre }}</p>
```

> **Nota**: para formularios complejos, preferir **Reactive Forms** (escalable y testeable).

---

## 3. Directivas estructurales

### 3.1 `*ngIf`
Renderiza un bloque si la expresión es verdadera. Permite `else` y `then`.

```html
<p *ngIf="cargando; else contenido">Cargando...</p>
<ng-template #contenido>
  <div>Datos listos</div>
</ng-template>
```

**Con `as` (captura de valor):**
```html
<div *ngIf="usuario$ | async as user; else login">
  Bienvenido, {{ user.nombre }}
</div>
<ng-template #login>Inicia sesión</ng-template>
```

### 3.2 `*ngFor`
Itera colecciones. Expone variables de contexto: `index`, `first`, `last`, `even`, `odd`.

```html
<li *ngFor="let item of items; index as i; first as esPrimero">
  {{ i }} - {{ item }} <span *ngIf="esPrimero">(primero)</span>
</li>
```

**Rendimiento con `trackBy`:**
```html
<li *ngFor="let t of tareas; trackBy: trackById">
  {{ t.id }} - {{ t.titulo }}
</li>
```
```ts
trackById(_index: number, item: { id: number }) { return item.id; }
```

### 3.3 `*ngSwitch`
Alterna vistas según un valor.

```html
<div [ngSwitch]="estado">
  <p *ngSwitchCase="'nuevo'">Nuevo</p>
  <p *ngSwitchCase="'proceso'">En proceso</p>
  <p *ngSwitchDefault>Desconocido</p>
</div>
```

### 3.4 `ng-container` y `ng-template`
- `<ng-container>`: contenedor **lógico** sin generar un nodo en el DOM.
- `<ng-template>`: define plantillas que se renderizan bajo demanda (usado por `ngIf`, `ngFor`, etc.).

```html
<ng-container *ngIf="items?.length; else vacio">
  <li *ngFor="let i of items">{{ i }}</li>
</ng-container>
<ng-template #vacio>No hay elementos</ng-template>
```

---

## 4. Directivas de atributo

### 4.1 `[ngClass]`
Aplica clases dinámicamente.

```html
<div [ngClass]="{ 'error': hayError, 'ok': !hayError }">Estado</div>
<div [ngClass]="['card', tema]"></div>
```

### 4.2 `[ngStyle]`
Aplica estilos de forma reactiva.

```html
<div [ngStyle]="{ color: hayError ? 'red' : 'black', 'font-weight': negrita ? 'bold' : 'normal' }">
  Texto
</div>
```

### 4.3 Clases y estilos abreviados
```html
<div [class.activo]="seleccionado"></div>
<div [style.width.px]="ancho"></div>
```

### 4.4 `NgModel` (two‑way en formularios simples)
```html
<input [(ngModel)]="filtro">
<ul>
  <li *ngFor="let u of usuarios | filtroNombre:filtro">{{ u.nombre }}</li>
</ul>
```

---

## 5. Pipes (tuberías) para presentación
Transforman valores **solo en la vista**.

```html
<p>{{ precio | currency:'USD':'symbol' }}</p>
<p>{{ fecha | date:'short' }}</p>
<p>{{ texto | uppercase }}</p>
```

**Pipes puros vs impuros**: los **puros** (por defecto) se ejecutan sólo cuando cambia la referencia del dato; los impuros (`pure: false`) se recalculan con más frecuencia (cuidado con el rendimiento).

---

## 6. Seguridad y sanitización

### 6.1 `innerHTML` y contenido dinámico
Angular **sanitiza** automáticamente HTML para prevenir XSS. Si necesitas insertar HTML:
```html
<div [innerHTML]="htmlSeguro"></div>
```
```ts
import { DomSanitizer, SafeHtml } from '@angular/platform-browser';

htmlSeguro: SafeHtml;
constructor(private s: DomSanitizer) {
  this.htmlSeguro = this.s.bypassSecurityTrustHtml('<b>Negrita</b>');
}
```
Usar `bypassSecurityTrust...` **con extremo cuidado** y sólo con contenido confiable.

### 6.2 URLs y recursos
Para URLs dinámicas, preferir bindings seguros y evitar concatenación insegura. Angular aplica sanitización a `[href]`, `[src]`, etc.

---

## 7. Patrón de presentación y buenas prácticas

1. **Expresiones simples** en plantillas: delegar lógica compleja al componente/pipe.
2. Usar `trackBy` en `*ngFor` cuando renders/actualizaciones sean frecuentes.
3. Evitar `any`: tipar datos y métodos.
4. Preferir **Reactive Forms** para formularios medianos y grandes.
5. Dividir componentes grandes en subcomponentes reutilizables.
6. Evitar crear objetos/funciones en línea dentro de la plantilla (genera nuevas referencias y re-render).

---

## 8. Ejemplos integrados

### 8.1 Lista filtrada con contador
```html
<input [(ngModel)]="filtro" placeholder="Filtrar...">

<p *ngIf="filtrados.length > 0; else sinResultados">
  Resultados: {{ filtrados.length }}
</p>
<ng-template #sinResultados>No hay coincidencias</ng-template>

<ul>
  <li *ngFor="let p of filtrados; trackBy: trackById">
    {{ p.nombre }} - {{ p.precio | currency:'USD' }}
  </li>
</ul>
```
```ts
filtro = '';
productos = [{id:1,nombre:'Teclado',precio:30},{id:2,nombre:'Mouse',precio:20}];
get filtrados() {
  const f = this.filtro.toLowerCase().trim();
  return !f ? this.productos : this.productos.filter(p => p.nombre.toLowerCase().includes(f));
}
trackById(_i: number, it: any) { return it.id; }
```

### 8.2 Microsintaxis de `*ngIf` con `as` y `else`
```html
<ng-container *ngIf="perfil$ | async as perfil; else cargando">
  <h2>{{ perfil.nombre }}</h2>
  <p>Rol: {{ perfil.rol }}</p>
</ng-container>

<ng-template #cargando>Cargando perfil...</ng-template>
```

### 8.3 `*ngSwitch` con estados
```html
<div [ngSwitch]="pedido.estado">
  <span *ngSwitchCase="'nuevo'">Nuevo</span>
  <span *ngSwitchCase="'enviado'">Enviado</span>
  <span *ngSwitchCase="'entregado'">Entregado</span>
  <span *ngSwitchDefault>Desconocido</span>
</div>
```

---

## 9. Crear una directiva personalizada (básico)

```ts
import { Directive, ElementRef, HostBinding, HostListener, Input } from '@angular/core';

@Directive({ selector: '[appResaltar]' })
export class ResaltarDirective {
  @Input('appResaltar') color = 'yellow';

  @HostBinding('style.backgroundColor') bg?: string;

  constructor(private el: ElementRef) {}

  @HostListener('mouseenter') onEnter() { this.bg = this.color; }
  @HostListener('mouseleave') onLeave()  { this.bg = undefined; }
}
```
Uso:
```html
<p [appResaltar]="'lightblue'">Texto con directiva personalizada</p>
```

---

## 10. Errores comunes y cómo evitarlos

- Usar métodos con cálculos pesados directamente en `{{ }}`: mover a `getter` memoizable o pipe.
- Olvidar `FormsModule` para `[(ngModel)]`: provocará error de binding.
- No usar `trackBy` en listas grandes: provoca renders innecesarios.
- Mezclar lógica de negocio en la plantilla: trasladar al componente/servicios.
- No manejar `null/undefined`: usar **navegación segura** `usuario?.perfil?.nombre`.

---

## 11. Checklist rápido

- ¿Las expresiones en la vista son simples?
- ¿Se usan directivas estructurales y de atributo de forma clara?
- ¿Hay `trackBy` en `*ngFor` cuando corresponde?
- ¿Se validan y sanitizan contenidos dinámicos (HTML/URLs)?
- ¿Se usa el enfoque de formularios adecuado (Template vs Reactive)?

---

## Referencias de la documentación oficial
- Angular Docs – Templates & Data Binding
- Angular Docs – Built-in directives
- Angular Docs – Pipes
- Angular Docs – Security (XSS, sanitization)
