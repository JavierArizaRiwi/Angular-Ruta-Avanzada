# AngularJS 1.8 + json-server — Guía completa (desde cero)

> **Stack**: AngularJS **1.8.x** (legacy) + **@uirouter/angularjs** + **json-server**.  
> **Meta**: Este README funciona como instructivo paso a paso y plantilla base para tu proyecto. Incluye **comentarios** en el código para entender cada parte.

---

## 0) Objetivo

Levantar una SPA en **AngularJS 1.8** que consuma una API REST simulada con **json-server**, implementando:

- Enrutamiento con **ui-router** (`@uirouter/angularjs`).
- Arquitectura por **componentes** (desde AngularJS 1.5), **servicios** e **interceptores**.
- **CRUD** (listar, crear, editar, eliminar).
- **Búsqueda, orden y paginación** usando query params de json-server.
- **Manejo de errores** con `$httpInterceptor`.
- **Estructura de carpetas** clara y mantenible.

> **Nota**: Aunque AngularJS está en modo legado, la arquitectura por componentes facilita una futura migración a Angular moderno.

---

## 1) Requisitos

- Node.js LTS (18+)
- npm 9+
- Sin build complejo: usaremos **CDN** para AngularJS + ui-router y un **servidor estático**.

Verifica versiones:
```bash
node -v
npm -v
```

---

## 2) Estructura del proyecto

```
angularjs-tienda/
├─ app/
│  ├─ components/
│  │  └─ products/
│  │     ├─ products-list.component.js
│  │     ├─ products-list.component.html
│  │     ├─ product-form.component.js
│  │     └─ product-form.component.html
│  ├─ services/
│  │  ├─ api.service.js
│  │  └─ products.service.js
│  ├─ interceptors/
│  │  └─ http-error.interceptor.js
│  ├─ app.module.js
│  └─ app.routes.js
├─ db.json
├─ index.html
└─ package.json
```

> **Idea**: `components/` aloja vistas reutilizables con su HTML + JS; `services/` encapsula llamadas HTTP y lógica; `interceptors/` centraliza manejo de errores; `app.module.js` y `app.routes.js` componen el núcleo.

---

## 3) Inicializar y dependencias (dev)

```bash
mkdir angularjs-tienda && cd angularjs-tienda
npm init -y
npm i -D json-server http-server concurrently
```

- **json-server**: API falsa en `http://localhost:3000`.
- **http-server**: sirve `index.html` y `app/` en `http://localhost:4200`.
- **concurrently**: corre ambos en paralelo (backend falso + frontend).

---

## 4) Base de datos falsa (`db.json`)

Crea `db.json` en la raíz:

```json
{
  "products": [
    { "id": 1, "name": "Teclado Mecánico", "price": 220000, "stock": 8, "category": "perifericos" },
    { "id": 2, "name": "Monitor 24\"", "price": 650000, "stock": 5, "category": "pantallas" },
    { "id": 3, "name": "Mouse Inalámbrico", "price": 90000, "stock": 12, "category": "perifericos" }
  ],
  "categories": [
    { "id": 1, "code": "perifericos", "label": "Periféricos" },
    { "id": 2, "code": "pantallas", "label": "Pantallas" }
  ]
}
```

**Consultas útiles json-server**

- **Paginación**: `?_page=1&_limit=5`
- **Orden**: `?_sort=price&_order=asc`
- **Búsqueda global**: `?q=teclado`
- **Filtro por campo**: `?category=perifericos`

> **Tip**: `json-server` usa almacenamiento en memoria; persiste solo en `db.json`. Cambios de datos se escriben ahí.

---

## 5) Scripts en `package.json`

Agrega estos scripts:

```json
{
  "scripts": {
    "api": "json-server --watch db.json --port 3000",
    "start": "http-server -p 4200 -c-1 .",
    "dev": "concurrently \"npm run api\" \"npm start\""
  }
}
```

- `-c-1` desactiva caché en `http-server` para ver cambios al instante.
- `dev` levanta **API** y **app** a la vez.

---

## 6) `index.html` (CDNs + bootstrap de la app)

Usamos CDN de AngularJS 1.8 y de ui-router para AngularJS. Incluimos estilos sencillos y cargamos los archivos de la app en orden.

```html
<!doctype html>
<html lang="es" ng-app="tiendaApp">
<head>
  <meta charset="utf-8" />
  <title>Tienda (AngularJS 1.8)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <!-- AngularJS 1.8 (core) -->
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.3/angular.min.js"></script>
  <!-- UI-Router para AngularJS -->
  <script src="https://unpkg.com/@uirouter/angularjs/release/angular-ui-router.min.js"></script>
  <!-- Normalización CSS ligera -->
  <link rel="stylesheet" href="https://unpkg.com/modern-normalize/modern-normalize.css">
  <style>
    body { font-family: system-ui, Arial, sans-serif; margin: 20px; }
    header { display:flex; gap:16px; align-items:center; margin-bottom: 16px; }
    nav a { margin-right: 8px; }
    table { width: 100%; border-collapse: collapse; margin-top: 12px; }
    th, td { border: 1px solid #ddd; padding: 8px; }
    th { cursor: pointer; }
    .toolbar { display:flex; gap:8px; align-items:center; }
    .pager { margin-top: 12px; display:flex; gap:8px; align-items:center; }
    form > label { display:block; margin: 8px 0; }
    .actions { display:flex; gap:8px; margin-top: 12px; }
  </style>
</head>
<body>
  <!-- Encabezado básico con navegación a estados -->
  <header>
    <h1>Tienda (AngularJS 1.8)</h1>
    <nav>
      <!-- ui-sref crea enlaces basados en estados de ui-router -->
      <a ui-sref="products">Productos</a>
      <a ui-sref="products.new">Nuevo</a>
    </nav>
  </header>

  <!-- ui-view es el contenedor donde ui-router inyecta la vista del estado activo -->
  <div ui-view></div>

  <!-- App core -->
  <script src="app/app.module.js"></script>
  <script src="app/app.routes.js"></script>

  <!-- Núcleo: interceptores y servicios -->
  <script src="app/interceptors/http-error.interceptor.js"></script>
  <script src="app/services/api.service.js"></script>
  <script src="app/services/products.service.js"></script>

  <!-- Componentes -->
  <script src="app/components/products/products-list.component.js"></script>
  <script src="app/components/products/product-form.component.js"></script>
</body>
</html>
```

> **Comentarios clave**:  
> - `ng-app="tiendaApp"` arranca AngularJS con el módulo `tiendaApp`.  
> - `ui-view` es el outlet de vistas de ui-router.  
> - El orden de `<script>` importa: primero módulo/rutas, luego interceptores/servicios, y finalmente componentes.

---

## 7) Módulo principal (`app/app.module.js`)

```js
(function() {
  'use strict';

  // Declaramos el módulo principal y su dependencia con ui.router
  angular
    .module('tiendaApp', ['ui.router'])
    .config(config);

  // Inyectamos $httpProvider para registrar interceptores globales
  config.$inject = ['$httpProvider'];
  function config($httpProvider) {
    // Registrar el interceptor HTTP global para manejo de errores
    $httpProvider.interceptors.push('httpErrorInterceptor');
  }
})();
```

> **Por qué un interceptor**: centraliza el manejo de errores de red/servidor en un solo lugar en lugar de duplicar `catch` en todos los servicios.

---

## 8) Rutas (ui-router) (`app/app.routes.js`)

```js
(function () {
  'use strict';

  angular
    .module('tiendaApp')
    .config(routes);

  routes.$inject = ['$stateProvider', '$urlRouterProvider', '$locationProvider'];
  function routes($stateProvider, $urlRouterProvider, $locationProvider) {
    // Redirigir cualquier URL no reconocida a /products
    $urlRouterProvider.otherwise('/products');

    // Definición de estados (rutas) con ui-router
    $stateProvider
      .state('products', {
        // URL con parámetros de consulta para lista/paginación/búsqueda/orden
        url: '/products?q&_page&_limit&_sort&_order',
        component: 'productsList' // Nombre del componente a renderizar
      })
      .state('products.new', {
        url: '/new',
        component: 'productForm',
        resolve: { productId: () => null } // En modo "crear", no hay id
      })
      .state('products.edit', {
        url: '/:id/edit', // :id es parámetro de ruta
        component: 'productForm',
        resolve: {
          // Usamos resolve para pasar el id ya parseado como binding del componente
          productId: ['$transition$', function($transition$) {
            return parseInt($transition$.params().id, 10);
          }]
        }
      });

    // Opcional: HTML5 mode si configuras tu servidor con fallback a index.html
    // $locationProvider.html5Mode(true);
  }
})();
```

> **Tip**: Mover el estado de la UI a la URL (query params) permite refrescar/compartir el enlace y mantener búsqueda/paginación/sort.

---

## 9) Interceptor HTTP (`app/interceptors/http-error.interceptor.js`)

```js
(function () {
  'use strict';

  angular
    .module('tiendaApp')
    .factory('httpErrorInterceptor', httpErrorInterceptor);

  httpErrorInterceptor.$inject = ['$q', '$window'];
  function httpErrorInterceptor($q, $window) {
    return {
      responseError: function (rejection) {
        var status = rejection.status;
        var msg = (rejection.data && rejection.data.message) || rejection.statusText || 'Error HTTP';

        // Log técnico para desarrolladores
        console.error('HTTP Error:', status, msg);

        // UI mínima para usuarios (podrías integrar toasts o un servicio de notificaciones)
        if (status === 0) {
          $window.alert('No hay conexión con el servidor API.');
        } else if (status >= 500) {
          $window.alert('Error del servidor. Intenta más tarde.');
        } else if (status === 404) {
          $window.alert('Recurso no encontrado.');
        }

        // Rechazamos para que el flujo que llamó pueda decidir qué hacer
        return $q.reject(rejection);
      }
    };
  }
})();
```

> **Extensión**: puedes detectar 401/403 e inyectar `$state` para redirigir a login, etc.

---

## 10) Servicio genérico de API (`app/services/api.service.js`)

```js
(function () {
  'use strict';

  angular
    .module('tiendaApp')
    .factory('apiService', apiService);

  apiService.$inject = ['$http'];
  function apiService($http) {
    var baseUrl = 'http://localhost:3000'; // Endpoint de json-server

    // Utilidad para construir params limpiando vacíos
    function buildParams(params) {
      var p = {};
      angular.forEach(params || {}, function (v, k) {
        if (v !== undefined && v !== null && v !== '') {
          p[k] = v;
        }
      });
      return p;
    }

    return {
      // GET con parámetros opcionales (lista/consulta)
      get: function (endpoint, params) {
        return $http.get(baseUrl + '/' + endpoint, { params: buildParams(params) })
                    .then(function(r){ return r.data; });
      },
      // POST para crear recursos
      post: function (endpoint, body) {
        return $http.post(baseUrl + '/' + endpoint, body).then(function(r){ return r.data; });
      },
      // PUT para actualizar recursos
      put: function (endpoint, body) {
        return $http.put(baseUrl + '/' + endpoint, body).then(function(r){ return r.data; });
      },
      // DELETE para eliminar recursos
      delete: function (endpoint) {
        return $http.delete(baseUrl + '/' + endpoint).then(function(r){ return r.data; });
      },
      // HEAD para obtener cabeceras (X-Total-Count) sin traer payload
      head: function (endpoint, params) {
        return $http.head(baseUrl + '/' + endpoint, { params: buildParams(params) });
      }
    };
  }
})();
```

> **Por qué `head`**: `json-server` incluye `X-Total-Count` en respuestas paginadas; con HEAD evitas descargar el listado completo solo para contar.

---

## 11) Servicio de dominio (`app/services/products.service.js`)

```js
(function () {
  'use strict';

  angular
    .module('tiendaApp')
    .factory('productsService', productsService);

  productsService.$inject = ['apiService', '$http'];
  function productsService(api, $http) {
    var endpoint = 'products';

    // Lista con soporte para búsqueda/página/orden
    function list(query) {
      return api.get(endpoint, query);
    }

    // Conteo eficiente usando cabecera X-Total-Count (requiere _page y _limit)
    function countViaHead(query) {
      var q = angular.copy(query || {});
      // para que json-server devuelva X-Total-Count, _page y _limit deben existir
      q._page = q._page || 1;
      q._limit = q._limit || 1;
      return $http.head('http://localhost:3000/' + endpoint, { params: q })
        .then(function (resp) {
          return parseInt(resp.headers('X-Total-Count') || '0', 10);
        });
    }

    // Alternativa simple: contar por largo del arreglo (menos eficiente en grandes volúmenes)
    function countSimple(query) {
      var q = angular.copy(query || {});
      delete q._page; delete q._limit; delete q._sort; delete q._order;
      return api.get(endpoint, q).then(function (arr) { return arr.length; });
    }

    function getById(id) { return api.get(endpoint + '/' + id); }
    function create(payload) { return api.post(endpoint, payload); }
    function update(id, payload) { return api.put(endpoint + '/' + id, payload); }
    function remove(id) { return api.delete(endpoint + '/' + id); }

    return {
      list: list,
      count: countViaHead, // Usa la versión eficiente por defecto
      getById: getById,
      create: create,
      update: update,
      remove: remove
    };
  }
})();
```

> **Decisión**: `countViaHead` evita descargar toda la colección para contar; en training/poCs es perfecto.

---

## 12) Lista de productos (componente)

**JS** — `app/components/products/products-list.component.js`

```js
(function () {
  'use strict';

  angular
    .module('tiendaApp')
    .component('productsList', {
      templateUrl: 'app/components/products/products-list.component.html',
      controller: ProductsListController
    });

  ProductsListController.$inject = ['$state', '$stateParams', 'productsService', '$window'];
  function ProductsListController($state, $stateParams, productsSvc, $window) {
    var vm = this;

    // Estado de UI alimentado desde la URL (permite compartir enlaces con búsqueda/paginación/sort)
    vm.q     = $stateParams.q     || '';
    vm.page  = parseInt($stateParams._page || 1, 10);
    vm.limit = parseInt($stateParams._limit || 5, 10);
    vm.sort  = $stateParams._sort || 'price';
    vm.order = $stateParams._order || 'asc';
    vm.total = 0;
    vm.loading = false;

    vm.products = [];

    vm.$onInit = function () {
      fetch();
      fetchCount();
    };

    // Dispara una búsqueda, reseteando a la primera página
    vm.search = function () {
      vm.page = 1;
      goWithParams();
    };

    // Alterna orden asc/desc al hacer click en la misma columna
    vm.sortBy = function (field) {
      if (vm.sort === field) {
        vm.order = vm.order === 'asc' ? 'desc' : 'asc';
      } else {
        vm.sort = field;
        vm.order = 'asc';
      }
      goWithParams();
    };

    // Paginación
    vm.pagePrev = function () {
      vm.page = Math.max(1, vm.page - 1);
      goWithParams();
    };

    vm.pageNext = function () {
      var maxPage = Math.max(1, Math.ceil(vm.total / vm.limit));
      vm.page = Math.min(maxPage, vm.page + 1);
      goWithParams();
    };

    // Eliminar con confirmación básica
    vm.delete = function (id) {
      if (!id) return;
      if ($window.confirm('¿Eliminar producto?')) {
        productsSvc.remove(id).then(fetch);
      }
    };

    // --- Internos ---
    function fetch() {
      vm.loading = true;
      productsSvc.list({
        q: vm.q || undefined,
        _page: vm.page,
        _limit: vm.limit,
        _sort: vm.sort,
        _order: vm.order
      }).then(function (res) {
        vm.products = res;
      }).finally(function () {
        vm.loading = false;
      });
    }

    function fetchCount() {
      // Para obtener X-Total-Count, countViaHead requiere _page/_limit (seteamos por defecto)
      productsSvc.count({
        q: vm.q || undefined,
        _page: 1,
        _limit: 1
      }).then(function (n) {
        vm.total = n;
      });
    }

    // Sincroniza el estado actual con la URL y recarga el estado
    function goWithParams() {
      $state.go('products', {
        q: vm.q || null,
        _page: vm.page,
        _limit: vm.limit,
        _sort: vm.sort,
        _order: vm.order
      }, { reload: true });
    }
  }
})();
```

**HTML** — `app/components/products/products-list.component.html`

```html
<section class="toolbar">
  <!-- ng-model + ng-change: búsqueda reactiva (puedes añadir debounce con una directiva si lo prefieres) -->
  <input type="text" placeholder="Buscar..."
         ng-model="$ctrl.q"
         ng-change="$ctrl.search()" />
  <a ui-sref="products.new"><button>+ Nuevo</button></a>
</section>

<table ng-if="!$ctrl.loading">
  <thead>
    <tr>
      <!-- Las columnas llaman a sortBy para alternar orden -->
      <th ng-click="$ctrl.sortBy('name')">Nombre</th>
      <th ng-click="$ctrl.sortBy('price')">Precio</th>
      <th ng-click="$ctrl.sortBy('stock')">Stock</th>
      <th ng-click="$ctrl.sortBy('category')">Categoría</th>
      <th>Acciones</th>
    </tr>
  </thead>
  <tbody>
    <tr ng-repeat="p in $ctrl.products">
      <td>{{ p.name }}</td>
      <td>{{ p.price | currency:'COP':0 }}</td>
      <td>{{ p.stock }}</td>
      <td>{{ p.category }}</td>
      <td>
        <a ui-sref="products.edit({ id: p.id })">Editar</a>
        <button ng-click="$ctrl.delete(p.id)">Eliminar</button>
      </td>
    </tr>
  </tbody>
</table>

<p ng-if="$ctrl.loading">Cargando...</p>

<footer class="pager">
  <button ng-click="$ctrl.pagePrev()">«</button>
  <span>
    Página {{ $ctrl.page }} / {{ Math.max(1, Math.ceil(($ctrl.total || 0) / $ctrl.limit)) }}
  </span>
  <button ng-click="$ctrl.pageNext()">»</button>
</footer>
```

> **Accesibilidad**: añade `aria-sort` dinámico en `<th>` si necesitas soporte a lectores de pantalla.

---

## 13) Formulario de producto (componente)

**JS** — `app/components/products/product-form.component.js`

```js
(function () {
  'use strict';

  angular
    .module('tiendaApp')
    .component('productForm', {
      templateUrl: 'app/components/products/product-form.component.html',
      bindings: { productId: '<' }, // Recibe productId desde resolve en rutas
      controller: ProductFormController
    });

  ProductFormController.$inject = ['$state', 'productsService', '$window'];
  function ProductFormController($state, productsSvc, $window) {
    var vm = this;

    // Modelo del formulario (ng-model bindea a estos campos)
    vm.model = {
      name: '',
      price: 0,
      stock: 0,
      category: ''
    };
    vm.isEdit = false;
    vm.sending = false;

    vm.$onInit = function () {
      // Si el estado trae productId, estamos en modo edición
      if (vm.productId) {
        vm.isEdit = true;
        productsSvc.getById(vm.productId).then(function (p) {
          vm.model = angular.copy(p || {});
        });
      }
    };

    vm.save = function (form) {
      // Validaciones nativas de AngularJS con form.$invalid y $touched
      if (form.$invalid) {
        $window.alert('Por favor corrige los errores del formulario.');
        return;
      }
      vm.sending = true;

      // Elegimos create/update según el modo
      var req = vm.isEdit
        ? productsSvc.update(vm.model.id, vm.model)
        : productsSvc.create(vm.model);

      req.then(function () {
        // Volver a la lista tras guardar
        $state.go('products', {}, { reload: true });
      }).finally(function () {
        vm.sending = false;
      });
    };

    vm.cancel = function () {
      $state.go('products');
    };
  }
})();
```

**HTML** — `app/components/products/product-form.component.html`

```html
<form name="form" novalidate ng-submit="$ctrl.save(form)">
  <label>Nombre
    <input type="text" name="name" ng-model="$ctrl.model.name" required ng-minlength="3" />
  </label>
  <div ng-show="form.name.$touched && form.name.$invalid">
    Debe tener al menos 3 caracteres.
  </div>

  <label>Precio
    <input type="number" name="price" ng-model="$ctrl.model.price" min="0" required />
  </label>

  <label>Stock
    <input type="number" name="stock" ng-model="$ctrl.model.stock" min="0" required />
  </label>

  <label>Categoría
    <input type="text" name="category" ng-model="$ctrl.model.category" required />
  </label>

  <div class="actions">
    <button type="submit" ng-disabled="$ctrl.sending">
      {{$ctrl.isEdit ? 'Actualizar' : 'Guardar'}}
    </button>
    <button type="button" ng-click="$ctrl.cancel()">Cancelar</button>
  </div>
</form>
```

> **Mejoras posibles**: validar categoría contra `categories` del backend y ofrecer `<select>` (consulta previa con un servicio `categoriesService`).

---

## 14) Probar la API desde consola (opcional)

```bash
# Listar
curl "http://localhost:3000/products"

# Buscar
curl "http://localhost:3000/products?q=Mouse"

# Paginación
curl "http://localhost:3000/products?_page=1&_limit=2"

# Crear
curl -X POST "http://localhost:3000/products" \
  -H "Content-Type: application/json" \
  -d '{"name":"Headset","price":180000,"stock":7,"category":"perifericos"}'

# Actualizar
curl -X PUT "http://localhost:3000/products/1" \
  -H "Content-Type: application/json" \
  -d '{"name":"Teclado Mecánico RGB","price":250000,"stock":6,"category":"perifericos"}'

# Eliminar
curl -X DELETE "http://localhost:3000/products/3"
```

---

## 15) Ejecutar el proyecto

```bash
npm run dev
# App: http://localhost:4200
# API: http://localhost:3000
```

- Navega a **Productos**.
- Prueba **buscar**, **ordenar**, **paginación**.
- Crea/edita/elimina desde la UI.

---

## 16) Buenas prácticas (AngularJS 1.8)

- **Componentes (1.5+)**: evita `$scope`, usa bindings (`<`) y controladores cohesivos.
- **Servicios**: encapsulan llamadas HTTP y reglas del dominio.
- **Interceptores**: centralizan manejo de errores (y también tokens de auth si los necesitas).
- **Rutas con estado**: reflejar estado de UI (q, page, sort) en URL mejora UX y compartibilidad.
- **Validación**: usa `form.$invalid`, `$touched`, `required`, `ng-minlength`, etc.
- **Migración futura**: esta estructura por componentes reduce el costo de migrar a Angular moderno.

---

## 17) Troubleshooting

- **La app no carga**: asegúrate de abrir `http://localhost:4200` servido por `http-server` (no el archivo `index.html` directo).
- **CORS**: AngularJS desde `http-server` puede llamar a `json-server` sin CORS estricto; si alojas en dominios distintos con políticas duras, considera un proxy.
- **Paginación rara**: revisa `_page` y `_limit`. Para `X-Total-Count`, usa `HEAD` con `_page/_limit` definidos.
- **IDs duplicados**: no envíes `id` manual si ya existe. Deja que json-server asigne o coordina IDs únicos.

---

## 18) Checklist de entrega

- [ ] `db.json` con datos iniciales.
- [ ] `index.html` con AngularJS 1.8 + ui-router.
- [ ] `app.module.js`, `app.routes.js`.
- [ ] Interceptor HTTP registrado.
- [ ] `api.service.js` + `products.service.js`.
- [ ] `products-list` y `product-form` funcionando.
- [ ] Scripts npm: `api`, `start`, `dev`.

---

## 19) Extensiones (opcional)

- **CategoriesService** + `<select>` de categorías (valida `category`).
- **Directiva debounce** para el input de búsqueda.
- **Directiva sort** que gestione iconos y `aria-sort`.
- **Guards** simples con ui-router (onEnter) para confirmar salida de formularios con cambios.
- **Notificaciones** con lib de toasts (o propia).
- **Tests** con Karma/Jasmine (unit tests para servicios y controladores).

---

### Arranque rápido (TL;DR)

```bash
npm i -D json-server http-server concurrently
npm run dev
# App: http://localhost:4200
# API: http://localhost:3000
```

> Con esto tienes una base sólida, **listo para enseñanza o PoC**, con comentarios que explican el “por qué” de cada pieza.