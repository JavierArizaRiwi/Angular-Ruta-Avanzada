# Entrenamiento Angular – Día 4: Formularios Template Driven y Reactivos

## Objetivos
- Crear formularios template driven (FormsModule) y reactivos (ReactiveFormsModule).
- Aplicar validaciones y mostrar mensajes de error.
- Diseñar un componente de formulario reutilizable.

---

## Formularios en Angular

Angular ofrece dos enfoques:
1. **Template Driven Forms**: más simples, se construyen en el HTML con `ngModel`.  
2. **Reactive Forms**: más escalables, se construyen en TypeScript con `FormGroup` y `FormControl`.

---

## Template Driven Forms

### Configuración
Importar `FormsModule` en `app.module.ts`:

```typescript
import { FormsModule } from '@angular/forms';
@NgModule({
  imports: [FormsModule]
})
export class AppModule {}
```

### Ejemplo
```html
<form (ngSubmit)="guardar()">
  <input name="titulo" [(ngModel)]="tarea.titulo" required>
  <div *ngIf="!tarea.titulo">Título requerido</div>
  <button type="submit" [disabled]="!tarea.titulo">Guardar</button>
</form>
```

Componente:
```typescript
tarea = { titulo: "" };

guardar() {
  console.log("Guardando tarea:", this.tarea);
}
```

---

## Reactive Forms

### Configuración
Importar `ReactiveFormsModule` en `app.module.ts`:

```typescript
import { ReactiveFormsModule } from '@angular/forms';
@NgModule({
  imports: [ReactiveFormsModule]
})
export class AppModule {}
```

### Ejemplo
```typescript
import { Component } from '@angular/core';
import { FormBuilder, Validators } from '@angular/forms';

@Component({
  selector: 'app-tarea-form',
  templateUrl: './tarea-form.component.html'
})
export class TareaFormComponent {
  constructor(private fb: FormBuilder) {}

  form = this.fb.group({
    titulo: ['', [Validators.required, Validators.minLength(3)]],
    descripcion: [''],
    completada: [false]
  });

  submit() {
    if (this.form.invalid) return;
    console.log(this.form.value);
  }
}
```

### Plantilla HTML
```html
<form [formGroup]="form" (ngSubmit)="submit()">
  <input formControlName="titulo" placeholder="Título">
  <div *ngIf="form.controls['titulo'].invalid && form.controls['titulo'].touched">
    Título requerido (mínimo 3 caracteres).
  </div>

  <textarea formControlName="descripcion" placeholder="Descripción"></textarea>

  <label>
    <input type="checkbox" formControlName="completada"> Completada
  </label>

  <button type="submit" [disabled]="form.invalid">Guardar</button>
</form>
```

---

## Ejercicio del día

1. Crear un componente `tarea-form`.  
2. Implementar un formulario **reactivo** con los campos: título, descripción y estado (checkbox).  
3. Aplicar validaciones: título obligatorio y mínimo 3 caracteres.  
4. Mostrar mensajes de error cuando las validaciones fallen.  
5. Tras guardar, limpiar el formulario.

### Resultado esperado
- El formulario no permite enviar datos inválidos.  
- Al ingresar un título válido y guardar, se muestra el objeto en consola y el formulario se reinicia.  

---