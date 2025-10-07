# Entrenamiento Angular – Sesión 8: Interceptores, Manejo Global de Errores y Loading

## Objetivos
- Centralizar headers, manejo de errores y loading.
- Implementar un interceptor HTTP con HttpInterceptor.
- Mostrar un spinner global mientras hay solicitudes pendientes.

## Conceptos clave
- HTTP_INTERCEPTORS en providers de AppModule.
- Clonación de requests con req.clone.
- Manejo de HttpErrorResponse con pipe(catchError).
- finalize para ocultar el loading al terminar.

## Servicio de loading
```ts
// loading.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class LoadingService {
  private _active = new BehaviorSubject<boolean>(false);
  readonly active$ = this._active.asObservable();
  show(){ this._active.next(true); }
  hide(){ this._active.next(false); }
}
```

## Interceptor
```ts
// app.interceptor.ts
import { Injectable } from '@angular/core';
import { HttpEvent, HttpHandler, HttpInterceptor, HttpRequest, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, finalize } from 'rxjs/operators';
import { LoadingService } from './loading.service';

@Injectable()
export class AppInterceptor implements HttpInterceptor {
  constructor(private loading: LoadingService) {}
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    this.loading.show();
    const cloned = req.clone({ setHeaders: { 'X-App': 'Tareas' }});
    return next.handle(cloned).pipe(
      catchError((err: HttpErrorResponse) => {
        console.error('HTTP error', err.status, err.message);
        return throwError(() => err);
      }),
      finalize(() => this.loading.hide())
    );
  }
}
```

## Registro del interceptor
```ts
// app.module.ts (providers)
{ provide: HTTP_INTERCEPTORS, useClass: AppInterceptor, multi: true }
```

## Componente de spinner
```html
<!-- app.component.html -->
<app-spinner *ngIf="loading$ | async"></app-spinner>
<router-outlet></router-outlet>
```

```ts
// app.component.ts
loading$ = this.loadingService.active$;
```

## Ejercicio
- Implementar interceptor con header personalizado y manejo de errores.
- Mostrar spinner mientras haya peticiones.
- Loguear códigos 400/500 y notificar al usuario.

## Resultado esperado
- Experiencia consistente: feedback de carga y errores centralizados.