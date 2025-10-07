# Entrenamiento Angular – Sesión 7: RxJS Esencial en Angular

## Objetivos
- Comprender Observables, Subjects y BehaviorSubject.
- Aplicar operadores clave en flujos de UI y HTTP.
- Implementar una búsqueda reactiva con typeahead.

## Conceptos clave
- Observable: secuencia asíncrona.
- Subject: observable + observer para multicasting.
- BehaviorSubject: subject con valor inicial y último valor disponible.
- Operadores: map, switchMap, debounceTime, distinctUntilChanged, catchError, finalize.

## Preparación
```bash
ng g component features/tareas/components/buscador
```

## Servicio con búsqueda
```ts
// tareas.service.ts (fragmento)
search(q: string){
  return this.http.get<any[]>(`${this.apiUrl}?q=${encodeURIComponent(q)}`);
}
```

## Componente con búsqueda reactiva
```ts
// buscador.component.ts
import { Component } from '@angular/core';
import { FormControl } from '@angular/forms';
import { TareasService } from '../../tareas.service';
import { debounceTime, distinctUntilChanged, switchMap, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

@Component({
  selector: 'app-buscador',
  template: `
    <input [formControl]="q" placeholder="Buscar tareas">
    <ul><li *ngFor="let t of resultados">{{ t.titulo }}</li></ul>
  `
})
export class BuscadorComponent {
  q = new FormControl('');
  resultados: any[] = [];

  constructor(private svc: TareasService){
    this.q.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(text => this.svc.search(text || '').pipe(
        catchError(() => of([]))
      ))
    ).subscribe(res => this.resultados = res);
  }
}
```

## BehaviorSubject para estado local
```ts
// tareas.store.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

interface TareaState { list: any[]; loading: boolean; }

@Injectable({ providedIn: 'root' })
export class TareasStore {
  private _state = new BehaviorSubject<TareaState>({ list: [], loading: false });
  readonly state$ = this._state.asObservable();
  get state(){ return this._state.value; }
  setState(p: Partial<TareaState>){ this._state.next({ ...this.state, ...p }); }
}
```

## Ejercicio
- Implementar búsqueda reactiva con debounce y cancelación (switchMap).
- Mostrar resultados en tiempo real.
- Manejar errores devolviendo una lista vacía.

## Resultado esperado
- Búsqueda fluida y resistente a errores, sin saturar el backend.