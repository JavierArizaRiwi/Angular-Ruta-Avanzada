# Guía de Transición y Comparativa: AngularJS (v1.x) vs Angular 2+ (moderno)

---

##  Objetivo
Entender las **diferencias clave** entre **AngularJS (v1.x)** y **Angular 2+ (moderno con TypeScript)** para que un equipo que mantiene proyectos legacy pueda **comprender ambos** y orientarse hacia el stack moderno sin necesidad de ver un proyecto completo.

---

##  Contexto general
| Aspecto | AngularJS (v1.x) | Angular 2+ (moderno) |
|---|---|---|
| Año de salida | 2010 | 2016 (re-escritura total) |
| Lenguaje principal | JavaScript ES5/ES6 | TypeScript (superset de JS) |
| Paradigma | MVC/MVVM basado en **$scope** | Arquitectura **basada en componentes** |
| Data binding | **Bidireccional automático** con `ng-model` | **Unidireccional por defecto** + `[(ngModel)]` para 2 vías |
| Enrutamiento | `ngRoute` / `ui-router` | `@angular/router` |
| Inyección de dependencias | `$injector` / anotación por nombre | DI tipada con decoradores (`@Injectable`) |
| HTTP/REST | `$http` | `HttpClient` |
| Formularios | `ng-model` (template-driven) | Template-Driven y **Reactive Forms** |
| Plantillas | Directivas (`ng-repeat`, `ng-if`, etc.) | Plantillas de componentes + directivas estructurales (`*ngFor`, `*ngIf`) |
| CLI / Build | Grunt/Gulp/Browserify (no oficial) | **Angular CLI** (oficial) + Webpack |
| Testing | Karma + Jasmine | Karma/Jasmine y soporte integrado en CLI |
| Performance | Watchers y digest cycle | Change detection más predecible (zones) |
| Soporte | **Legado** / mantenimiento mínimo | **Activo** y con releases regulares |

> **Idea clave**: *AngularJS* y *Angular 2+* son **frameworks diferentes**. El segundo no es “versión nueva” sino **re-escritura** con conceptos y herramientas modernas.

---

##  Arquitectura y organización

### AngularJS (v1.x)
- Estructura típica por **roles**: `controllers/`, `services/`, `directives/`, `views/`.
- Controladores y `$scope` conectan la lógica con la vista.
- Módulo global: `angular.module('app', [...])`.
- Plantillas HTML con directivas (`ng-*`).

### Angular 2+ (moderno)
- Estructura **modular** por **feature**: cada “feature” tiene su **módulo, componentes, servicios y rutas**.
- **Componentes** como unidad fundamental de UI (clase + template + estilos).
- **Decoradores**: `@Component`, `@NgModule`, `@Injectable`.
- **Angular CLI** estandariza el árbol de archivos y genera scaffolding.

---

##  Data Binding y Change Detection

| Tema | AngularJS (v1.x) | Angular 2+ |
|---|---|---|
| Data binding | 2-way binding automático con `ng-model` | 1-way por defecto; 2-way con `[(ngModel)]` |
| Motor | `$digest` + watchers | Zone.js + change detection |
| Implicación | Fácil pero puede degradar performance con muchos watchers | Más predecible, optimiza redibujo y escalabilidad |

---

##  Componentes, Directivas y Plantillas

**AngularJS:**  
- Controladores (`ng-controller`) + vistas (HTML) con directivas como `ng-repeat`, `ng-if`.  
- Directivas personalizadas para encapsular UI/funcionalidad.

**Angular 2+:**  
- **Componentes** con `@Component` (clase TS, template y estilos).  
- **Directivas estructurales** (`*ngIf`, `*ngFor`) y de atributo (`[class.x]`, `[style.x]`).  
- **Pipes** para formateo (`date`, `currency`, pipes personalizados).

---

##  Inyección de dependencias (DI)

| Tema | AngularJS (v1.x) | Angular 2+ |
|---|---|---|
| Registro | `angular.module().service()/.factory()` | Decoradores y providers en módulos |
| Inyección | Por nombre (con posibles problemas de minificación) | Tipada y segura con TypeScript |
| Scope | `$rootScope`, `$scope` | Árbol de inyectores por módulo/componente |

---

##  Enrutamiento

- **AngularJS:** `ngRoute` (simple) o `ui-router` (más potente con estados anidados).  
- **Angular 2+:** `@angular/router` con **módulos de rutas**, **guards**, **lazy loading** y **resolvers**.

---

##  Comunicación con APIs

| Tema | AngularJS (v1.x) | Angular 2+ |
|---|---|---|
| HTTP client | `$http` (promesas) | `HttpClient` (Observables + interceptores) |
| Interceptores | Implementados sobre `$httpProvider` | Interceptores nativos (`HTTP_INTERCEPTORS`) |
| Manejo de errores | `.then().catch()` | RxJS `pipe(catchError(...))` |

---

##  Formularios

| Tema | AngularJS (v1.x) | Angular 2+ |
|---|---|---|
| Modelo | `ng-model` | Template-driven (`ngModel`) o **Reactive Forms** |
| Validación | Atributos HTML + clases de estado (`ng-valid`) | **FormControl/FormGroup** (reactivo) y validaciones síncronas/asincrónicas |
| Escalabilidad | Limitada en formularios complejos | Altamente escalable con **Reactive Forms** |

---

##  Herramientas y Build

- **AngularJS:** No existía una CLI oficial; se usaban **Grunt/Gulp**, `bower`, `browserify`.  
- **Angular 2+:** **Angular CLI** (comandos `ng new`, `ng serve`, `ng generate`, `ng build`) y **Webpack** bajo el capó. Integración nativa con testing.

---

##  Rendimiento y escalabilidad

| Criterio | AngularJS (v1.x) | Angular 2+ |
|---|---|---|
| Detección de cambios | Basada en watchers y digest cycle | Basada en árbol de componentes (más eficiente) |
| Lazy Loading | No nativo (workarounds) | **Nativo** en router/módulos |
| SSR / pre-render | No nativo | **Angular Universal** |
| Reutilización | Directivas y servicios | **Componentes**, módulos y librerías reutilizables |

---

##  Seguridad (alto nivel)

- **AngularJS:** protección básica XSS vía binding; cuidado con `$sce` para contenido confiable.
- **Angular 2+:** binding seguro por defecto, **DomSanitizer** para casos especiales, **HttpClient** con interceptores para auth (JWT), **guards** en rutas.

---

##  Patrones de migración (mental model)

1. **De Controlador + Vista → Componente**  
   - Lo que antes estaba en `controller + $scope` ahora vive dentro de una **clase de componente** con propiedades y métodos.
2. **De Directiva personalizada → Componente/Directiva**  
   - Si la directiva representaba UI, ahora suele ser **componente**. Si alteraba comportamiento, seguirá siendo **directiva**.
3. **De `$http` → `HttpClient`**  
   - Cambia a **Observables** y **interceptores**.
4. **De `ngRoute/ui-router` → `@angular/router`**  
   - Define **módulos de rutas**, usa **guards**, aplica **lazy loading** por feature.
5. **De `ng-model` → FormsModule/ReactiveFormsModule**  
   - Formularios complejos: **Reactive Forms** con `FormGroup` y `FormControl`.

> Consejo: Migrar **por features** (módulos funcionales) y no por archivos; usar **AngularJS + Angular híbrido** con **ngUpgrade** sólo si es imprescindible.

---

##  Tabla resumen final

| Área | AngularJS (v1.x) | Angular 2+ (moderno) |
|---|---|---|
| Paradigma | MVC/MVVM con `$scope` | Componentes + Módulos |
| Lenguaje | JavaScript | TypeScript |
| Datos | 2 vías automático | 1 vía + 2 vías explícita |
| Router | `ngRoute` / `ui-router` | `@angular/router` con guards y lazy load |
| HTTP | `$http` (promesas) | `HttpClient` (Observables, interceptores) |
| Formularios | `ng-model` | Template y **Reactive Forms** |
| DI | `$injector` | DI tipada con decoradores |
| Builds | Grunt/Gulp (no oficial) | **Angular CLI** (oficial) |
| Rendimiento | Watchers/digest | Change detection optimizado |
| Soporte | **Legado** | **Activo** |

---

##  Recomendaciones prácticas para tu equipo

- Si mantienen **AngularJS**: estabilicen dependencias, limiten watchers, usen `component()` API cuando sea posible y aíslen features.  
- Para avanzar en **Angular 2+**: disciplinen el uso de **módulos por feature**, **lazy loading**, **interceptores**, **Reactive Forms** y **Angular CLI** para estandarizar.  
- Capacitación mínima: **TypeScript** (tipos, clases, interfaces), **RxJS** (Observables, `pipe`, `map`, `catchError`), y **Angular Router**.

---

##  Apéndice: ejemplos mínimos (conceptuales)

### AngularJS (v1.x) – componente “mental” con controller
```html
<div ng-app="app" ng-controller="MainCtrl as vm">
  <h3>{{ vm.titulo }}</h3>
  <input ng-model="vm.nombre">
  <button ng-click="vm.saludar()">Saludar</button>
  <p>{{ vm.mensaje }}</p>
</div>
<script>
  angular.module('app', [])
    .controller('MainCtrl', function() {
      var vm = this;
      vm.titulo = 'AngularJS v1';
      vm.saludar = function() { vm.mensaje = 'Hola ' + vm.nombre; };
    });
</script>
```

### Angular 2+ – componente equivalente (conceptual)
```ts
@Component({
  selector: 'app-main',
  template: \`
    <h3>{{ titulo }}</h3>
    <input [(ngModel)]="nombre">
    <button (click)="saludar()">Saludar</button>
    <p>{{ mensaje }}</p>
  \`
})
export class MainComponent {
  titulo = 'Angular 2+';
  nombre = '';
  mensaje = '';
  saludar(){ this.mensaje = 'Hola ' + this.nombre; }
}
```

---

### Cierre
Con esta guía, tu equipo **comprende ambos mundos**: puede mantener proyectos **AngularJS** con criterio y a la vez **pensar en Angular moderno** con componentes, módulos y herramientas actuales. Si más adelante quieres, puedo convertir este material en **diapositivas** o añadir **checklists** de migración por feature.