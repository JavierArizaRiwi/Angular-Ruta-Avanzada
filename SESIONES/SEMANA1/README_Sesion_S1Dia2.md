# Entrenamiento Angular – Día 2: Data Binding y Directivas

## Objetivos
- Comprender y aplicar los 4 tipos de data binding en Angular.
- Usar directivas estructurales (`*ngIf`, `*ngFor`) y de atributos (`[ngClass]`, `[ngStyle]`).
- Construir un componente dinámico que gestione una lista de tareas.

---

## Data Binding en Angular

### 1. Interpolación ({{ }})
Permite mostrar valores de variables en la plantilla.

```html
<h1>Hola {{ nombre }}</h1>
```

### 2. Property Binding ([ ])
Permite enlazar propiedades de HTML a variables del componente.

```html
<img [src]="urlImagen" [alt]="descripcion">
```

### 3. Event Binding (( ))
Escucha eventos del DOM y ejecuta métodos del componente.

```html
<button (click)="saludar()">Saludar</button>
```

```typescript
saludar() {
  alert("Hola desde Angular!");
}
```

### 4. Two-way Binding ([( )])
Combina property y event binding. Requiere `FormsModule`.

```html
<input [(ngModel)]="nombre">
<p>Bienvenido {{ nombre }}</p>
```

---

## Directivas en Angular

### *ngIf
Renderiza un elemento solo si la condición es verdadera.

```html
<p *ngIf="visible">Texto visible solo si 'visible' es true.</p>
```

### *ngFor
Itera sobre un arreglo y genera un elemento por cada item.

```html
<ul>
  <li *ngFor="let item of items; index as i">{{ i }} - {{ item }}</li>
</ul>
```

### [ngClass] y [ngStyle]
Aplican clases y estilos dinámicamente.

```html
<div [ngClass]="{ error: hayError }" [ngStyle]="{ color: hayError ? 'red' : 'black' }">
  Estado actual
</div>
```

---

## Ejemplo práctico: Lista de Tareas

Generar un componente:
```bash
ng generate component lista-tareas
```

Editar `lista-tareas.component.ts`:
```typescript
export class ListaTareasComponent {
  nuevaTarea: string = "";
  tareas: string[] = [];
  visible: boolean = true;

  agregarTarea() {
    if (this.nuevaTarea.trim()) {
      this.tareas.push(this.nuevaTarea);
      this.nuevaTarea = "";
    }
  }

  toggleVisibilidad() {
    this.visible = !this.visible;
  }
}
```

Editar `lista-tareas.component.html`:
```html
<h2>Lista de Tareas</h2>

<input [(ngModel)]="nuevaTarea" placeholder="Nueva tarea">
<button (click)="agregarTarea()">Agregar</button>
<button (click)="toggleVisibilidad()">Mostrar/Ocultar</button>

<ul *ngIf="visible">
  <li *ngFor="let tarea of tareas; index as i">
    {{ i + 1 }} - {{ tarea }}
  </li>
</ul>
```

---

## Ejercicio del día

1. Crea el componente `lista-tareas` usando Angular CLI.  
2. Implementa la lógica para añadir nuevas tareas.  
3. Muestra las tareas en una lista con `*ngFor`.  
4. Añade un botón para alternar visibilidad con `*ngIf`.  

### Resultado esperado
- El usuario puede escribir tareas en el input y añadirlas.  
- Las tareas aparecen listadas.  
- Se puede mostrar u ocultar la lista con un botón.  

---