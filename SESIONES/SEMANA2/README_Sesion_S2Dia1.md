# Entrenamiento Angular – Día 3: Servicios e HttpClient

## Objetivos
- Comprender qué son los servicios y por qué se usan en Angular.
- Crear servicios inyectables para encapsular lógica de datos.
- Configurar y usar HttpClient para consumir APIs REST (GET, POST, PUT, DELETE).
- Manejar errores básicos con RxJS y suscripción.
- Comprender el ciclo de vida y la inyección de dependencias.

---

## 1) ¿Qué es un Servicio en Angular?

Un **servicio** es una clase que encapsula lógica reutilizable y desacoplada de la vista.  
Su propósito es centralizar operaciones como:
- Comunicación con APIs externas.
- Manejo de datos compartidos entre componentes.
- Lógica de negocio o transformaciones de datos.

Angular usa **Dependency Injection (DI)** para proveer servicios a componentes, asegurando bajo acoplamiento y testabilidad.

---

## 2) Configuración inicial

Para poder consumir APIs, debemos importar el módulo **HttpClientModule**.

### En proyectos standalone modernos

Si estás usando Angular 17+ (sin `AppModule`), importa el módulo en `app.config.ts`:

```typescript
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    importProvidersFrom(HttpClientModule)
  ]
};
```

### En proyectos clásicos (con módulos)

Si tu proyecto aún usa módulos tradicionales (`app.module.ts`):

```typescript
import { HttpClientModule } from '@angular/common/http';
import { NgModule } from '@angular/core';

@NgModule({
  imports: [HttpClientModule]
})
export class AppModule {}
```

Esto habilita el uso global de `HttpClient` en toda la aplicación.

---

## 3) Crear un Servicio de Datos

Generar un servicio usando Angular CLI:
```bash
ng generate service tareas
```

Esto crea dos archivos:
- `tareas.service.ts` → código del servicio.
- `tareas.service.spec.ts` → pruebas automáticas.

### Código base del servicio

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class TareasService {
  private apiUrl = 'http://localhost:8080/api/tareas';

  constructor(private http: HttpClient) {}

  get(): Observable<any[]> {
    return this.http.get<any[]>(this.apiUrl);
  }

  create(t: any): Observable<any> {
    return this.http.post(this.apiUrl, t);
  }

  update(id: number, t: any): Observable<any> {
    return this.http.put(`${this.apiUrl}/${id}`, t);
  }

  remove(id: number): Observable<any> {
    return this.http.delete(`${this.apiUrl}/${id}`);
  }
}
```

**Claves:**
- `@Injectable({ providedIn: 'root' })` → hace que el servicio esté disponible en toda la app sin declararlo en módulos.
- `HttpClient` → maneja las peticiones HTTP.
- Cada método retorna un `Observable` (de **RxJS**) al cual el componente se suscribe.

---

## 4) Inyección de dependencias (Dependency Injection)

Angular inyecta automáticamente los servicios en los componentes que los declaran en el constructor:

```typescript
constructor(private svc: TareasService) {}
```

Esto se denomina **inyección por constructor** y evita usar `@Autowired` o instanciar manualmente con `new`.

Ventajas:
- Desacopla la lógica del componente.
- Permite reutilizar código.
- Facilita las pruebas unitarias (mocks o spies).

---

## 5) Usar el servicio en un componente

Ejemplo de uso en `lista-tareas.component.ts`:

```typescript
import { Component, OnInit } from '@angular/core';
import { TareasService } from '../tareas.service';

@Component({
  selector: 'app-lista-tareas',
  templateUrl: './lista-tareas.component.html'
})
export class ListaTareasComponent implements OnInit {
  tareas: any[] = [];
  nuevaTarea: string = "";

  constructor(private svc: TareasService) {}

  ngOnInit() {
    this.cargar();
  }

  cargar() {
    this.svc.get().subscribe({
      next: data => this.tareas = data,
      error: err => console.error('Error cargando tareas', err)
    });
  }

  guardar() {
    if (this.nuevaTarea.trim()) {
      this.svc.create({ titulo: this.nuevaTarea }).subscribe({
        next: () => this.cargar(),
        error: err => console.error('Error creando tarea', err)
      });
      this.nuevaTarea = "";
    }
  }

  eliminar(id: number) {
    this.svc.remove(id).subscribe(() => this.cargar());
  }
}
```

**Puntos importantes:**
- `ngOnInit()` carga las tareas al iniciar el componente.
- Cada método del servicio se **suscribe** para recibir datos o errores.
- La UI se actualiza automáticamente al cambiar el arreglo `tareas`.

---

## 6) Manejo de errores con RxJS

Angular usa **Observables** para manejar datos asincrónicos.  
Podemos capturar errores usando `catchError` de **RxJS**.

```typescript
import { catchError, throwError } from 'rxjs';

get(): Observable<any[]> {
  return this.http.get<any[]>(this.apiUrl).pipe(
    catchError(err => {
      console.error('Error en la petición', err);
      return throwError(() => new Error('Error al obtener las tareas'));
    })
  );
}
```

También puedes manejar errores globales con un **interceptor HTTP**, que veremos en sesiones posteriores.

---

## 7) Ejemplo de vista (HTML)

```html
<h2>Gestión de Tareas (con Servicio)</h2>

<input [(ngModel)]="nuevaTarea" placeholder="Nueva tarea">
<button (click)="guardar()">Agregar</button>

<ul>
  <li *ngFor="let tarea of tareas">
    {{ tarea.titulo }}
    <button (click)="eliminar(tarea.id)">Eliminar</button>
  </li>
</ul>
```

**Conceptos aplicados:**
- Two-way binding para el input (`[(ngModel)]`).
- Event binding para botones (`(click)`).
- Uso de `*ngFor` para renderizar las tareas dinámicamente.

---

## 8) Ejercicio del día

1. Crear el servicio `TareasService` con los métodos `get`, `create`, `update`, `remove`.
2. Inyectarlo en `lista-tareas.component.ts`.
3. Mostrar la lista de tareas obtenida desde una API.
4. Implementar creación y eliminación de tareas desde la interfaz.
5. Agregar manejo de errores en consola con `catchError` o `subscribe(error)`.

### Resultado esperado
- Al iniciar la app, se cargan tareas desde `http://localhost:8080/api/tareas`.
- Se pueden crear nuevas tareas.
- Se pueden eliminar tareas existentes.
- La lista se actualiza automáticamente después de cada operación.

---

## 9) Recomendaciones prácticas

- Mantén las URLs en un archivo de **configuración** o en `environment.ts`.
- Evita repetir código HTTP; reutiliza servicios para cada entidad.
- No manipules `HttpClient` directamente desde los componentes.
- Usa **tipado estricto** en los modelos de datos (ej. `Tarea { id: number; titulo: string; }`).
- Implementa interceptores para manejar autenticación o logs.

---

## 10) Conclusiones

- Los **servicios** permiten un código más limpio, reutilizable y mantenible.
- `HttpClient` facilita la comunicación con APIs RESTful de manera tipada.
- RxJS potencia el manejo de flujos asíncronos con operadores.
- La inyección de dependencias en Angular evita acoplamientos y promueve la modularidad.
- Con estos fundamentos, ya puedes conectar tu frontend Angular con un backend real.

---