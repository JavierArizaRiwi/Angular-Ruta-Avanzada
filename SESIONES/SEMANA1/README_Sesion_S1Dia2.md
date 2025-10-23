# Entrenamiento Angular – Día 2: Data Binding y Directivas

## Objetivos
- Comprender y aplicar los 4 tipos de data binding en Angular.
- Usar directivas estructurales (`*ngIf`, `*ngFor`) y de atributos (`[ngClass]`, `[ngStyle]`).
- Construir un componente dinámico que gestione una lista de tareas.
- Comprender el flujo de datos unidireccional y bidireccional en Angular.
- Dominar la interacción entre vista (HTML) y lógica (TypeScript).

---

## 1) Fundamentos de Data Binding en Angular

El **data binding** (enlace de datos) conecta el modelo de datos del componente con la vista (plantilla HTML).  
Permite sincronizar información de manera declarativa, sin manipular directamente el DOM.

Angular ofrece **4 tipos de binding** principales:

| Tipo | Sintaxis | Dirección de datos | Descripción |
|------|-----------|--------------------|--------------|
| Interpolación | `{{ variable }}` | Componente → Vista | Muestra valores en HTML |
| Property Binding | `[propiedad]="variable"` | Componente → Vista | Enlaza atributos HTML a datos |
| Event Binding | `(evento)="método()"` | Vista → Componente | Escucha eventos del usuario |
| Two-Way Binding | `[(ngModel)]="variable"` | Bidireccional | Sincroniza vista y modelo |

---

## 2) Interpolación ({{ }})

Permite mostrar el valor de una propiedad del componente dentro del HTML.

```typescript
export class AppComponent {
  nombre = "Coder";
}
```

```html
<h1>Hola {{ nombre }}</h1>
<p>Bienvenido al curso de Angular</p>
```

**Claves:**
- Solo puede contener **expresiones** (no estructuras de control ni asignaciones).
- Evalúa dentro del **contexto del componente**.
- Angular actualiza automáticamente el valor cuando el modelo cambia.

---

## 3) Property Binding ([ ])

Permite **enlazar propiedades del DOM** (como `src`, `alt`, `disabled`, etc.) a variables del componente.

```typescript
export class AppComponent {
  urlImagen = "https://angular.io/assets/images/logos/angular/angular.png";
  descripcion = "Logo oficial de Angular";
}
```

```html
<img [src]="urlImagen" [alt]="descripcion" width="200">
```

**Claves:**
- Se usa para establecer valores dinámicos.
- Angular sustituye el valor en tiempo de ejecución.

---

## 4) Event Binding (( ))

Permite **escuchar eventos del usuario** (clics, cambios, teclado, etc.) y ejecutar métodos del componente.

```typescript
export class AppComponent {
  saludar() {
    alert("¡Hola desde Angular!");
  }
}
```

```html
<button (click)="saludar()">Saludar</button>
```

**Claves:**
- El evento se pasa entre paréntesis.
- Puede usarse también con `$event` para obtener información del evento.

```html
<input (input)="onInputChange($event)">
```

---

## 5) Two-way Binding ([( )])

Combina **property binding** y **event binding** en un solo mecanismo.  
Requiere importar `FormsModule` en la configuración del proyecto.

```typescript
export class AppComponent {
  nombre: string = "";
}
```

```html
<input [(ngModel)]="nombre" placeholder="Escribe tu nombre">
<p>Bienvenido {{ nombre }}</p>
```

**Claves:**
- `[(ngModel)]` sincroniza el valor del input y la propiedad.
- Ideal para formularios y edición de datos en tiempo real.

---

## 6) Directivas en Angular

Las **directivas** son instrucciones que extienden el comportamiento de elementos HTML.

### 6.1 Directivas estructurales (`*`)

Modifican la estructura del DOM (añaden o eliminan elementos).

#### *ngIf
Muestra o elimina elementos según una condición.

```typescript
export class AppComponent {
  visible = true;
}
```

```html
<p *ngIf="visible">Este texto se muestra solo si 'visible' es true.</p>
<button (click)="visible = !visible">Alternar visibilidad</button>
```

#### *ngFor
Itera sobre un arreglo para generar elementos dinámicamente.

```typescript
export class AppComponent {
  items = ["Angular", "React", "Vue"];
}
```

```html
<ul>
  <li *ngFor="let item of items; index as i">{{ i + 1 }} - {{ item }}</li>
</ul>
```

---

### 6.2 Directivas de atributos

Alteran la apariencia o el comportamiento de un elemento existente.

#### [ngClass]
Aplica clases CSS condicionalmente.

```typescript
export class AppComponent {
  hayError = true;
}
```

```html
<div [ngClass]="{ error: hayError, correcto: !hayError }">Estado actual</div>
```

#### [ngStyle]
Aplica estilos en función de valores dinámicos.

```html
<div [ngStyle]="{ color: hayError ? 'red' : 'green' }">
  Texto con estilo dinámico
</div>
```

---

## 7) Ejemplo práctico: Lista de Tareas

Generar un componente con Angular CLI:

```bash
ng generate component lista-tareas
```

### `lista-tareas.component.ts`
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

### `lista-tareas.component.html`
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

**Conceptos aplicados:**
- `[(ngModel)]` → two-way binding para capturar el input.
- `(click)` → event binding para agregar tareas.
- `*ngIf` → mostrar u ocultar la lista.
- `*ngFor` → iterar sobre las tareas.

---

## 8) Ejercicio del día

1. Crea el componente `lista-tareas` usando Angular CLI.  
2. Implementa la lógica para añadir nuevas tareas.  
3. Muestra las tareas en una lista con `*ngFor`.  
4. Añade un botón para alternar visibilidad con `*ngIf`.  
5. Usa `[ngClass]` para resaltar tareas completadas (opcional).

### Resultado esperado
- El usuario escribe tareas en un input y las añade con un botón.  
- Las tareas se muestran dinámicamente.  
- Se puede mostrar/ocultar la lista.  
- Se refuerza el uso de los 4 tipos de data binding y directivas principales.

---

## 9) Recomendaciones finales

- Usa **interpolación** para mostrar valores simples.
- Usa **property binding** cuando necesites manipular atributos del DOM.
- Usa **event binding** para responder a acciones del usuario.
- Usa **two-way binding** en formularios o inputs interactivos.
- Combina directivas estructurales y de atributos para lograr componentes dinámicos y expresivos.
- Refactoriza tu código usando **componentes reutilizables** con inputs y outputs (ver Día 3).

---

## 10) Conclusiones

- Los tipos de binding definen cómo fluyen los datos entre la lógica y la vista.
- Las directivas hacen que el HTML sea **reactivo y declarativo**.
- El enfoque standalone facilita el aprendizaje: cada componente es autónomo.
- Con Angular puedes crear interfaces reactivas sin manipular directamente el DOM.

---