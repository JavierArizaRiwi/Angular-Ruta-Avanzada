# Entrenamiento Angular – Día 3: Servicios e HttpClient

## Objetivos
- Comprender qué son los servicios y por qué se usan en Angular.
- Crear servicios inyectables para encapsular lógica de datos.
- Configurar y usar HttpClient para consumir APIs REST (GET, POST, PUT, DELETE).
- Manejar errores básicos con RxJS y suscripción.

---

## ¿Qué es un Servicio en Angular?
Un **servicio** es una clase que encapsula lógica reutilizable, como acceder a datos de una API o realizar cálculos.  
Angular permite inyectar servicios en componentes mediante **Dependency Injection**.

---

## Configuración inicial

### Importar HttpClientModule
En `app.module.ts`:

```typescript
import { HttpClientModule } from '@angular/common/http';
import { NgModule } from '@angular/core';

@NgModule({
  imports: [HttpClientModule]
})
export class AppModule {}
```

Esto habilita el uso de `HttpClient` en toda la aplicación.

---

## Crear un Servicio de Datos

Generar servicio con Angular CLI:
```bash
ng generate service tareas
```

Archivo `tareas.service.ts`:
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

---

## Usar el servicio en un componente

Ejemplo en `lista-tareas.component.ts`:

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

---

## Ejercicio del día

1. Crear un servicio `TareasService` que gestione datos desde una API en `http://localhost:8080/api/tareas`.  
2. Inyectar el servicio en el componente `lista-tareas`.  
3. Mostrar tareas al iniciar la app (`ngOnInit`).  
4. Implementar botones para **crear** y **eliminar** tareas conectados al servicio.

### Resultado esperado
- Al abrir la app, se listan tareas desde la API.  
- El usuario puede crear nuevas tareas, que aparecen en la lista.  
- Se pueden eliminar tareas y la lista se actualiza.  

---