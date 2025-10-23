# Entrenamiento Angular – Día 4: Formularios Template Driven y Reactivos

## Objetivos
- Crear formularios template driven (FormsModule) y reactivos (ReactiveFormsModule).
- Aplicar validaciones y mostrar mensajes de error.
- Diseñar un componente de formulario reutilizable.
- Entender buenas prácticas de accesibilidad (a11y), manejo de estado y limpieza.
- Conocer validadores sincrónicos y asincrónicos, y cómo probar formularios.

---

## 0) Formularios en Angular (visión general)

Angular ofrece dos enfoques complementarios:

1) **Template Driven Forms (TDF):**  
   - Declarativos, apoyados en el HTML con `ngModel`.  
   - Aprendizaje rápido; perfectos para formularios simples.  
   - Validador vía atributos (`required`, `minlength`, etc.) y directivas (`ngModel`, `ngForm`).

2) **Reactive Forms (RF):**  
   - Definidos en TypeScript mediante `FormGroup`, `FormControl` y `FormBuilder`.  
   - Más escalables, trazables y testeables.  
   - Validaciones como funciones puras; fácil composición y reusabilidad.

**Regla práctica:** empieza con TDF para formularios chicos; usa RF cuando necesites validaciones complejas, interacción avanzada, testeo fino o escalabilidad.

---

## 1) Configuración de módulos (standalone y clásico)

### 1.1 Proyectos *standalone* (Angular 17+)

En `app.config.ts` (cliente), importa módulos con `importProvidersFrom`:

```ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    importProvidersFrom(FormsModule),
    importProvidersFrom(ReactiveFormsModule),
  ],
};
```

> Si usas SSR, recuerda agregar `provideServerRendering()` en `app.config.server.ts` (ver guía del Día 1) — los formularios no requieren cambios especiales para SSR.

### 1.2 Proyectos con `AppModule` (clásico)

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { App } from './app/app';

@NgModule({
  declarations: [],
  imports: [BrowserModule, FormsModule, ReactiveFormsModule],
  bootstrap: [App]
})
export class AppModule {}
```

---

## 2) Template Driven Forms (TDF)

### 2.1 Estructura mínima

**Componente (TS):**
```ts
export class TareaTdfComponent {
  tarea = { titulo: '' };
  guardar() {
    console.log('Guardando tarea (TDF):', this.tarea);
  }
}
```

**Plantilla (HTML):**
```html
<form #f="ngForm" (ngSubmit)="guardar()">
  <label for="titulo">Título</label>
  <input id="titulo" name="titulo" [(ngModel)]="tarea.titulo" required minlength="3">
  
  <div *ngIf="f.submitted && (!f.controls['titulo']?.valid)">
    <small *ngIf="f.controls['titulo']?.errors?.['required']">El título es obligatorio.</small>
    <small *ngIf="f.controls['titulo']?.errors?.['minlength']">Mínimo 3 caracteres.</small>
  </div>

  <button type="submit" [disabled]="f.invalid">Guardar</button>
</form>
```

**Claves TDF:**
- `#f="ngForm"` expone el estado del formulario (pristine, dirty, valid, invalid, submitted, etc.).
- Cada control debe tener `name` para registrarse en el `ngForm`.
- Los validadores nativos (`required`, `minlength`, etc.) funcionan con `ngModel`.

### 2.2 Estilos por estado del control

Angular aplica clases automáticas: `ng-touched`, `ng-untouched`, `ng-dirty`, `ng-pristine`, `ng-valid`, `ng-invalid`.

```css
input.ng-invalid.ng-touched { border-color: #e74c3c; }
input.ng-valid.ng-touched   { border-color: #2ecc71; }
```

---

## 3) Reactive Forms (RF)

### 3.1 Estructura mínima con FormBuilder

**Componente (TS):**
```ts
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
    console.log('Guardando (RF):', this.form.value);
    this.form.reset({ titulo: '', descripcion: '', completada: false });
  }
}
```

**Plantilla (HTML):**
```html
<form [formGroup]="form" (ngSubmit)="submit()">
  <label for="titulo">Título</label>
  <input id="titulo" formControlName="titulo" placeholder="Título">

  <div *ngIf="form.controls['titulo'].touched && form.controls['titulo'].invalid">
    <small *ngIf="form.controls['titulo'].errors?.['required']">Título requerido.</small>
    <small *ngIf="form.controls['titulo'].errors?.['minlength']">Mínimo 3 caracteres.</small>
  </div>

  <label for="descripcion">Descripción</label>
  <textarea id="descripcion" formControlName="descripcion" placeholder="Descripción"></textarea>

  <label>
    <input type="checkbox" formControlName="completada"> Completada
  </label>

  <button type="submit" [disabled]="form.invalid">Guardar</button>
</form>
```

**Claves RF:**
- El estado y las validaciones están en el **modelo** (TS). La plantilla solo refleja.
- Fácil de testear y de componer validaciones.
- Ideal para formularios complejos o de gran tamaño.

### 3.2 Validadores personalizados (sincrónicos)

```ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

export function noPalabrasProhibidas(...prohibidas: string[]): ValidatorFn {
  const lista = prohibidas.map(p => p.toLowerCase());
  return (control: AbstractControl): ValidationErrors | null => {
    const valor = (control.value ?? '').toLowerCase();
    return lista.some(p => valor.includes(p))
      ? { palabraProhibida: true }
      : null;
  };
}
```

Uso en el formulario:
```ts
form = this.fb.group({
  titulo: ['', [Validators.required, Validators.minLength(3), noPalabrasProhibidas('spam','fake')]]
});
```

### 3.3 Validadores asíncronos (ejemplo con API)

```ts
import { AsyncValidatorFn, AbstractControl, ValidationErrors } from '@angular/forms';
import { map, catchError, of } from 'rxjs';
import { HttpClient } from '@angular/common/http';

export function tituloDisponibleValidator(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl) => {
    return http.get<{ disponible: boolean }>(`/api/tareas/disponible?titulo=${encodeURIComponent(control.value)}`)
      .pipe(
        map(res => (res.disponible ? null : { tituloOcupado: true })),
        catchError(() => of(null)) // No bloquea si hay error en la API
      );
  };
}
```

Uso:
```ts
form = this.fb.group({
  titulo: ['', {
    validators: [Validators.required],
    asyncValidators: [tituloDisponibleValidator(this.http)],
    updateOn: 'blur' // valida al perder foco
  }]
});
```

---

## 4) Componente de formulario reutilizable

Ejemplo de componente genérico que emite valores válidos:

```ts
import { Component, EventEmitter, Output } from '@angular/core';
import { FormBuilder, Validators } from '@angular/forms';

@Component({
  selector: 'app-form-tarea',
  template: \`
  <form [formGroup]="form" (ngSubmit)="submit()">
    <input formControlName="titulo" placeholder="Título">
    <button type="submit" [disabled]="form.invalid">Guardar</button>
  </form>
  \`,
  standalone: true
})
export class FormTareaComponent {
  @Output() guardado = new EventEmitter<{ titulo: string }>();

  constructor(private fb: FormBuilder) {}

  form = this.fb.group({ titulo: ['', [Validators.required, Validators.minLength(3)]] });

  submit() {
    if (this.form.valid) {
      this.guardado.emit(this.form.value as { titulo: string });
      this.form.reset({ titulo: '' });
    }
  }
}
```

Uso en un contenedor:
```html
<app-form-tarea (guardado)="crearTarea($event)"></app-form-tarea>
```

---

## 5) Accesibilidad (a11y) y UX

- Asocia siempre `label` con `for` e `id` del `input`.
- Añade `aria-invalid="true"` cuando el control sea inválido y `touched`.
- Provee mensajes de error cercanos al campo y con `role="alert"` si corresponde.
- Usa placeholders como ayuda, **no** como reemplazo del label.

Ejemplo rápido:
```html
<label for="titulo">Título</label>
<input id="titulo" formControlName="titulo" [attr.aria-invalid]="form.controls['titulo'].invalid && form.controls['titulo'].touched">
<div role="alert" *ngIf="form.controls['titulo'].touched && form.controls['titulo'].invalid">
  <!-- mensajes -->
</div>
```

---

## 6) Manejo de estado y reseteo

- `form.reset()` restablece valores y estado (pristine, untouched).  
- Puedes pasar valores por defecto: `form.reset({ titulo: '', descripcion: '', completada: false })`.
- Para TDF, resetea el objeto de modelo o usa `f.resetForm()` si tienes la referencia al formulario.

---

## 7) Pruebas básicas de formularios

Ejemplo (Jasmine/Karma) para RF:
```ts
it('debería ser inválido cuando título está vacío', () => {
  const ctrl = component.form.get('titulo');
  ctrl?.setValue('');
  expect(ctrl?.invalid).toBeTrue();
});

it('debería ser válido con título >= 3', () => {
  const ctrl = component.form.get('titulo');
  ctrl?.setValue('abc');
  expect(ctrl?.valid).toBeTrue();
});
```

---

## 8) Ejercicio del día

1. Crea el componente `tarea-form`.  
2. Implementa un **formulario reactivo** con: `titulo` (requerido, min 3), `descripcion` (opcional), `completada` (checkbox).  
3. Muestra mensajes de error al tocar el campo y fallar validaciones.  
4. Emite los datos válidos a un componente contenedor.  
5. Limpia el formulario después de guardar.

### Resultado esperado
- El formulario **no** permite enviar datos inválidos.  
- Con datos válidos, se emiten al contenedor y se reinicia el formulario.  
- La UI muestra errores de forma clara y accesible.

---

## 9) Conclusiones

- **TDF** es rápido para formularios simples; **RF** brilla en escalabilidad, testeo y validaciones complejas.  
- Usa validadores personalizados (sincrónicos/asincrónicos) para reglas de negocio reales.  
- Mantén formularios accesibles y con buen feedback visual.  
- Encapsula formularios en componentes reutilizables y emite eventos al contenedor.

---