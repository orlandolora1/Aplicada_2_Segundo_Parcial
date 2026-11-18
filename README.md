# Vice City Motors — App móvil

App en React Native con **Expo Router** para explorar el catálogo de vehículos de lujo de Vice City Motors y gestionar compras. Consume la API oficial `https://vicecity-motors-api.vercel.app`.

## Funcionalidades

| # | Funcionalidad | Dónde está |
|---|---|---|
| 1 | Registro (nombre, email, contraseña), login, redirección automática si hay sesión y cerrar sesión | `app/registro.js`, `app/login.js`, `app/index.js`, botón **Salir** en el catálogo |
| 2 | Catálogo con marca, modelo, año, precio y estado, más filtro **Todos / Disponibles** | `app/catalogo/index.js` |
| 3 | Detalle con todos los datos y botón **Comprar** con confirmación (`metodoPago: "transferencia"`) | `app/catalogo/[id].js` |
| 4 | Historial de compras con vehículo, fecha y monto | `app/compras.js` |

## Arquitectura por capas

```
app/                  Pantallas (Expo Router)
  _layout.js          Navegación Stack y estilo del encabezado
  index.js            Revisa si hay token y redirige a /catalogo o /login
  login.js
  registro.js
  catalogo/index.js   Lista de vehículos + filtro
  catalogo/[id].js    Detalle + compra
  compras.js          Mis compras
services/
  api.js              TODAS las llamadas HTTP a la API
  storage.js          Token en AsyncStorage (clave "userToken")
components/           VehiculoCard, VehiculoImagen, EstadoBadge, MensajeError
utils/                Formatos de precio/fecha, estado del vehículo, sesión expirada
constants/            Colores y estilos compartidos
```

Las pantallas nunca llaman a `fetch` directamente: todo pasa por `services/api.js`. Esa capa agrega el token, maneja el tiempo de espera y convierte cualquier error en un mensaje legible.

### Endpoints usados

| Método | Ruta | Uso |
|---|---|---|
| POST | `/auth/registro` | Crear cuenta |
| POST | `/auth/login` | Iniciar sesión (devuelve `token`) |
| GET | `/vehiculos` | Catálogo completo |
| GET | `/vehiculos?estado=disponible` | Filtro de disponibles |
| GET | `/vehiculos/:id` | Detalle |
| POST | `/ventas` | Comprar: `{ "vehiculoId": 1, "metodoPago": "transferencia" }` |
| GET | `/ventas` | Mis compras (el cliente solo ve las suyas) |

### Manejo de errores

- Todos los errores se muestran en pantalla: con `Alert` para acciones como el login, el registro o la compra, y con un mensaje y un botón **Reintentar** cuando falla una carga.
- Si la API responde **401** (token vencido), la app borra el token y vuelve al login.
- Si no hay internet o el servidor tarda más de 20 segundos, la app muestra un mensaje en vez de quedarse cargando.
- Casos que se manejan sin cerrar la app: email duplicado (409), credenciales inválidas (401), vehículo ya vendido (409), vehículo inexistente (404).

## Instalación y ejecución

> Crea el proyecto **fuera de OneDrive**, por ejemplo en `C:\proyectos\`. OneDrive bloquea los archivos de `node_modules`.

1. Descomprime la carpeta en `C:\proyectos\vicecity-motors` y ábrela en VS Code.
2. En la terminal, corre uno a la vez:
   ```bash
   npm install
   npm run setup
   npx expo start -c
   ```
3. Escanea el QR con la cámara del iPhone (Expo Go) o con la app Expo Go en Android.

No hace falta `.env`: la URL de la API está en `services/api.js`.

## Cuenta personal

Cada estudiante crea su propia cuenta desde la pantalla **Crear cuenta** de la app, que usa `POST /auth/registro`. No se usan las cuentas de prueba de la colección de Postman.

## Pruebas manuales

| Caso | Resultado esperado |
|---|---|
| Registrarse con un email nuevo | "Cuenta creada" y vuelve al login con el email ya escrito |
| Registrarse con el mismo email otra vez | Mensaje de email duplicado (409) |
| Login con contraseña incorrecta | Mensaje de credenciales inválidas (401) |
| Cerrar la app y abrirla de nuevo con sesión | Entra directo al catálogo |
| Filtro **Disponibles** | Solo vehículos con estado `disponible` |
| Comprar → **Cancelar** | No pasa nada |
| Comprar → **Comprar** | "¡Compra realizada!" y el vehículo pasa a `vendido` |
| Abrir un vehículo vendido | No aparece el botón **Comprar** |
| Mis compras | Vehículo, fecha y monto de cada compra |
| **Salir** | Borra el token y vuelve al login |
| Modo avión y deslizar para recargar | Mensaje de conexión con **Reintentar**, sin cierre |
