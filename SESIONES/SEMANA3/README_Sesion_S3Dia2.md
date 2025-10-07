# Entrenamiento Angular – Sesión 6: Comunicación entre Componentes

## Objetivos
- Separar componentes contenedores y de presentación.
- Comunicar datos con @Input y eventos con @Output.
- Usar ViewChild y proyección de contenido con ng-content.

## Conceptos clave
- Componentes presentacionales: reciben datos por @Input y emiten eventos por @Output.
- Componentes contenedores: obtienen datos del servicio y coordinan.
- ViewChild: referencia a un hijo para invocar métodos o leer propiedades.
- ng-content: inserción de contenido personalizado dentro de un componente.

## Ejemplo: Tarjeta de Tarea (presentacional)
```ts
// tarea-card.component.ts
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-tarea-card',
  template: `
    <article class="card">
      <header><ng-content select="[card-title]"></ng-content></header>
      <p>{{ tarea?.titulo }}</p>
      <button (click)="onEliminar()">Eliminar</button>
    </article>
  `
})
export class TareaCardComponent {
  @Input() tarea: any;
  @Output() eliminar = new EventEmitter<number>();

  onEliminar() {
    if (this.tarea?.id) this.eliminar.emit(this.tarea.id);
  }
}
```

## Uso en el contenedor
```html
<!-- tarea-list.component.html -->
<section *ngFor="let t of tareas">
  <app-tarea-card [tarea]="t" (eliminar)="eliminar($event)">
    <h4 card-title>{{ t.titulo }}</h4>
  </app-tarea-card>
</section>
```

```ts
// tarea-list.component.ts
import { Component } from '@angular/core';
import { TareasService } from '../../tareas.service';

@Component({
  selector: 'app-tarea-list',
  templateUrl: './tarea-list.component.html'
})
export class TareaListComponent {
  tareas: any[] = [];
  constructor(private svc: TareasService){ this.cargar(); }
  cargar(){ this.svc.get().subscribe(res => this.tareas = res); }
  eliminar(id: number){ this.svc.remove(id).subscribe(() => this.cargar()); }
}
```

## ViewChild para invocar métodos del hijo
```ts
@ViewChild(TareaCardComponent) card!: TareaCardComponent;
// this.card.onEliminar();
```

## Ejercicio
- Convertir elementos de la lista en tarjetas presentacionales.
- Emitir el evento eliminar desde la tarjeta al contenedor.
- Usar ng-content para personalizar el título.

## Resultado esperado
- Componentes reusables, con responsabilidades claras y comunicación fluida.