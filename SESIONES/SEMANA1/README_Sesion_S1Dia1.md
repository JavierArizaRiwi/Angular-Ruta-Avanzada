# Entrenamiento Angular – Día 1: Introducción a Angular

## Objetivos
- Entender la arquitectura de Angular (componentes, módulos, servicios).
- Instalar Angular CLI y crear el primer proyecto.
- Explorar la estructura de carpetas.
- Crear y renderizar un componente básico con datos dinámicos.

---

## ¿Qué es Angular?
Angular es un framework de desarrollo frontend creado por Google. Está basado en TypeScript y permite crear aplicaciones web de una sola página (SPA - Single Page Application).  
Sus elementos principales son:
- **Componentes**: bloques básicos de una aplicación. Cada componente tiene su HTML, CSS y lógica en TypeScript.
- **Módulos**: agrupan componentes, servicios y otras dependencias.
- **Servicios**: clases que encapsulan lógica reutilizable, como conexión a APIs o manejo de datos.

---

## Instalación y verificación de entorno

Antes de empezar necesitamos **Node.js** y **npm**. Verifica las versiones:

```bash
node -v
npm -v
```

Instalar Angular CLI (herramienta oficial de Angular):

```bash
npm install -g @angular/cli
ng version
```

Con este comando se debe mostrar la versión de Angular CLI instalada.

---

## Crear un proyecto Angular y servirlo

```bash
ng new mi-proyecto
cd mi-proyecto
ng serve
```

Esto creará una aplicación base. Angular CLI preguntará opciones como si deseas añadir **routing** y el tipo de estilos (CSS, SCSS, etc.).  
Una vez generado, abre en el navegador:

```
http://localhost:4200
```

Deberías ver la aplicación funcionando.

---

## Estructura de carpetas

- **src/app**: contiene los componentes principales.  
- **app.component.ts**: código TypeScript del componente raíz.  
- **app.component.html**: plantilla HTML del componente raíz.  
- **app.module.ts**: módulo raíz que importa otros módulos y declara componentes.

---

## Crear el primer componente

Usamos Angular CLI para generar un nuevo componente:

```bash
ng generate component saludo
```

Esto crea una carpeta `saludo` con:
- `saludo.component.ts` (lógica)
- `saludo.component.html` (vista)
- `saludo.component.css` (estilos)
- `saludo.component.spec.ts` (pruebas)

### Editar saludo.component.html
```html
<h2>Bienvenido a Angular!</h2>
<p>Este es mi primer componente.</p>
```

### Usar el componente en app.component.html
```html
<app-saludo></app-saludo>
```

Al recargar el navegador, se mostrará el contenido del nuevo componente.

---

## Ejercicio del día

1. Edita `saludo.component.ts` para añadir una variable `nombre` inicializada en TypeScript:
```typescript
export class SaludoComponent {
  nombre: string = "Coder";
}
```

2. En el HTML (`saludo.component.html`), muestra el valor con interpolación:
```html
<p>Hola {{ nombre }}!</p>
```

3. Habilita la entrada de texto para que el usuario escriba su nombre.  
Primero importa `FormsModule` en `app.module.ts`:

```typescript
import { FormsModule } from '@angular/forms';
@NgModule({
  imports: [BrowserModule, FormsModule]
})
```

4. Agrega un input y haz un **two-way data binding** con `[(ngModel)]`:

```html
<input [(ngModel)]="nombre" placeholder="Escribe tu nombre">
<p>Hola {{ nombre }}!</p>
```

### Resultado esperado:
Cuando el usuario escriba un nombre en el input, el saludo debe actualizarse automáticamente.

---