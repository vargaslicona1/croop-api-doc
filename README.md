# Documentación Técnica API CROOP Panel Cross

## Índice

1. [Introducción](#introducción)
2. [Información General](#información-general)
   - 2.1 [URL Base y Ambiente](#url-base-y-ambiente)
   - 2.2 [Autenticación y Seguridad](#autenticación-y-seguridad)
   - 2.3 [Variables Globales](#variables-globales)
3. [Módulo de Autenticación](#módulo-de-autenticación)
   - 3.1 [Login - Inicio de Sesión](#login---inicio-de-sesión)
4. [Módulo de Gestión de Clientes](#módulo-de-gestión-de-clientes)
   - 4.1 [Consultar ID de Cliente por Email](#consultar-id-de-cliente-por-email)
   - 4.2 [Consultar Clientes General](#consultar-clientes-general)
   - 4.3 [Consultar Cliente por PKey](#consultar-cliente-por-pkey)
   - 4.4 [Catálogo de Países](#catálogo-de-países)
   - 4.5 [Insertar Nuevo Cliente](#insertar-nuevo-cliente)
5. [Módulo de Gestión de Créditos](#módulo-de-gestión-de-créditos)
   - 5.1 [Consultar Productos Disponibles](#consultar-productos-disponibles)
   - 5.2 [Consultar Catálogos de Carteras](#consultar-catálogos-de-carteras)
   - 5.3 [Consultar Catálogos de Plazos](#consultar-catálogos-de-plazos)
   - 5.4 [Generar Cotización y Tabla de Amortización](#generar-cotización-y-tabla-de-amortización)
   - 5.5 [Insertar Crédito](#insertar-crédito)
   - 5.6 [Consultar Créditos por Cliente](#consultar-créditos-por-cliente)
6. [Módulo de Consultas de Contratos](#módulo-de-consultas-de-contratos)
   - 6.1 [Consultar Tabla de Amortización](#consultar-tabla-de-amortización)
   - 6.2 [Consultar Detalles de Contrato](#consultar-detalles-de-contrato)
7. [Módulo de Gestión de Pagos](#módulo-de-gestión-de-pagos)
   - 7.1 [Consultar Lista de Pagos](#consultar-lista-de-pagos)
   - 7.2 [Insertar Pago](#insertar-pago)
   - 7.3 [Ejecución Manual de Cobro](#ejecución-manual-de-cobro)
8. [Catálogos Adicionales](#catálogos-adicionales)
   - 8.1 [Consultar Carteras de Empresa](#consultar-carteras-de-empresa)
9. [Módulo de Movimientos y Transacciones](#módulo-de-movimientos-y-transacciones)
   - 9.1 [Consultar Movimientos de Contrato](#consultar-movimientos-de-contrato)
10. [Consideraciones Técnicas](#consideraciones-técnicas)
11. [Códigos de Respuesta](#códigos-de-respuesta)
12. [Mejores Prácticas](#mejores-prácticas)

---

## 1. Introducción

El presente documento describe la especificación técnica de la API REST del sistema **CROOP Panel Cross**, una plataforma integral para la gestión de créditos y servicios financieros. Esta API proporciona una interfaz programática que permite la integración con sistemas externos y la automatización de procesos relacionados con la administración de clientes, créditos, pagos y transacciones financieras.

### Alcance del Documento

Este documento técnico está dirigido a desarrolladores, arquitectos de software y equipos de integración que requieren implementar o consumir los servicios expuestos por la API de CROOP Panel Cross. Se detallan los endpoints disponibles, métodos HTTP soportados, parámetros requeridos, formatos de respuesta y consideraciones de seguridad.

### Versión de la API

La documentación corresponde a la versión actual de la API desplegada en el ambiente de desarrollo.

---

## 2. Información General

### 2.1 URL Base y Ambiente

La API está disponible en el siguiente endpoint base:

| Ambiente | URL Base | Descripción |
|----------|----------|-------------|
| Desarrollo | `https://apides.croop.mx:8085/api` | Ambiente de desarrollo y pruebas |

**Protocolo de Comunicación:** HTTPS (Puerto 8085)  
**Formato de Datos:** JSON  
**Codificación de Caracteres:** UTF-8

### 2.2 Autenticación y Seguridad

El sistema implementa un modelo de autenticación basado en **tokens JWT** con las siguientes características:

#### Headers Requeridos

Todos los requests a la API deben incluir los siguientes headers:

| Header | Tipo | Descripción | Requerido |
|--------|------|-------------|-----------|
| `AppID` | String | Identificador único de la aplicación cliente | Sí |
| `Token` | String | Token JWT obtenido del proceso de login | Sí (excepto para login) |
| `Content-Type` | String | Tipo de contenido del request | Según endpoint |
| `Cache-Control` | String | Control de caché HTTP | Opcional |

#### AppID de Aplicación

```
AppID: eae76274-0727-e811-9456-22000a244a86
```

Este identificador es único para la aplicación y debe ser incluido en todos los requests.

### 2.3 Variables Globales

El sistema maneja las siguientes variables globales que son utilizadas a través de múltiples endpoints:

| Variable | Tipo | Descripción | Origen |
|----------|------|-------------|---------|
| `base_url` | String | URL base de la API | Configuración de ambiente |
| `app_id` | String | Identificador de aplicación | Archivo de configuración |
| `token` | String | Token de autenticación JWT | Respuesta del endpoint de login |
| `fk_empresa` | Integer | ID de la empresa en contexto | Configuración de usuario (valor por defecto: 4) |
| `fk_login_name` | Integer | ID del usuario autenticado | Respuesta del endpoint de login (campo UserPkey) |

---

## 3. Módulo de Autenticación

Este módulo gestiona los procesos de autenticación y autorización de usuarios en el sistema.

### 3.1 Login - Inicio de Sesión

Endpoint para autenticar usuarios y obtener el token de acceso necesario para consumir los demás servicios de la API.

#### Especificación del Endpoint

- **URL:** `/access/Signin`
- **Método HTTP:** `POST`
- **Autenticación Requerida:** No

#### Headers del Request

| Header | Valor | Descripción |
|--------|-------|-------------|
| Content-Type | application/json | Formato del body del request |
| AppID | {app_id} | Identificador de la aplicación |
| Cache-Control | no-cache | Evita el almacenamiento en caché |

#### Body del Request

```json
{
    "Email": "string",
    "Password": "string",
    "IPAddress": "string",
    "UserAgent": "string",
    "ServerURL": "string"
}
```

#### Parámetros del Body

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|-----------|-------------|---------|
| `Email` | String | Sí | Correo electrónico del usuario | moises@croop.mx |
| `Password` | String | Sí | Contraseña del usuario | Web1234$$ |
| `IPAddress` | String | Sí | Dirección IP del cliente | 192.168.1.1 |
| `UserAgent` | String | Sí | Información del navegador/cliente | Mozilla/5.0... |
| `ServerURL` | String | Sí | URL del servidor de origen | localhost |

#### Response Exitoso (200 OK)

```json
{
    "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "UserPkey": 96524,
    "Success": true,
    "Message": "Autenticación exitosa"
}
```

#### Campos de Respuesta

| Campo | Tipo | Descripción | Uso Posterior |
|-------|------|-------------|---------------|
| `Token` | String | Token JWT para autenticación | Se almacena en variable `token` para requests subsecuentes |
| `UserPkey` | Integer | Identificador único del usuario | Se almacena en variable `fk_login_name` |
| `Success` | Boolean | Indicador de éxito de la operación | Validación del resultado |
| `Message` | String | Mensaje descriptivo del resultado | Información al usuario |

#### Proceso Post-Autenticación

Tras una autenticación exitosa, el sistema automáticamente:
1. Almacena el token en la variable de colección `token`
2. Almacena el UserPkey en la variable `fk_login_name`
3. Estos valores se utilizarán en todos los requests posteriores

---

## 4. Módulo de Gestión de Clientes

Este módulo proporciona funcionalidades completas para la administración del ciclo de vida de los clientes en el sistema.

### 4.1 Consultar ID de Cliente por Email

Permite obtener el identificador único de un cliente mediante su dirección de correo electrónico.

#### Especificación del Endpoint

- **URL:** `/clsUsuario/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Valor | Descripción |
|-----------|------|-------|-------------|
| `Bandera` | Integer | 401 | Identificador de la operación de consulta por email |
| `Email` | String | Variable | Dirección de correo electrónico del cliente a buscar |

#### Ejemplo de Request

```
GET {{base_url}}/clsUsuario/Listar?Bandera=401&Email=moises@croop.mx
```

#### Response

El endpoint retorna la información completa del usuario asociado al email proporcionado, incluyendo su PKey (identificador único).

### 4.2 Consultar Clientes General

Permite obtener un listado paginado de clientes de la empresa con filtros avanzados por tipo de usuario.

#### Especificación del Endpoint

- **URL:** `/clsUsuario/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Valor/Origen |
|-----------|------|-------------|--------------|
| `Bandera` | Integer | Identificador de operación | Valor fijo: 408 |
| `FK_Empresa` | Integer | ID de la empresa | Variable global `fk_empresa` |
| `fJSON` | String | Filtros adicionales en formato JSON | Filtro por tipo de usuario |
| `RegistrosPorPag` | Integer | Cantidad de registros por página | 10-100 registros |
| `NumPag` | Integer | Número de página a consultar | Inicia en 1 |

#### Estructura del parámetro fJSON

```json
{
    "FK_TipoUsuario": [6]
}
```

- `FK_TipoUsuario`: Array de tipos de usuario a filtrar (6 = Cliente)

#### Ejemplo de Request

```
GET {{base_url}}/clsUsuario/Listar?Bandera=408&FK_Empresa={{fk_empresa}}&fJSON={ "FK_TipoUsuario": [6] }&RegistrosPorPag=10&NumPag=1
```

#### Response

Retorna un objeto con:
- Lista paginada de clientes
- Total de registros encontrados
- Información completa de cada cliente (nombre, email, RFC, CURP, dirección, etc.)
- Metadata de paginación

### 4.3 Consultar Cliente por PKey

Obtiene la información detallada de un cliente utilizando su identificador único (PKey).

#### Especificación del Endpoint

- **URL:** `/clsUsuario/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Valor | Descripción |
|-----------|------|-------|-------------|
| `Bandera` | Integer | 402 | Identificador de la operación de consulta por PKey |
| `PKey` | Integer | Variable | Identificador único del cliente |

#### Ejemplo de Request

```
GET {{base_url}}/clsUsuario/Listar?Bandera=402&PKey=96524
```

### 4.4 Catálogo de Países

Endpoint para obtener el catálogo completo de países disponibles en el sistema, utilizado principalmente en formularios de registro y actualización de datos.

#### Especificación del Endpoint

- **URL:** `/clsUsuario/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Valor | Descripción |
|-----------|------|-------|-------------|
| `Bandera` | Integer | 403 | Identificador para obtener el catálogo de países |

### 4.5 Insertar Nuevo Cliente

Endpoint para el registro de nuevos clientes en el sistema, incluyendo toda la información personal, fiscal y de contacto.

#### Especificación del Endpoint

- **URL:** `/access/Signup`
- **Método HTTP:** `POST`
- **Autenticación Requerida:** Sí

#### Body del Request

```json
{
    "Bandera": 100,
    "Nombre": "string",
    "APP": "string",
    "APM": "string",
    "Email": "string",
    "RFC": "string",
    "CURP": "string",
    "Fecha_Nac": "string",
    "FK_TipoPersona": integer,
    "FK_PaisNacimiento": integer,
    "FK_PaisResidencia": integer,
    "FK_Genero": integer,
    "Celular": "string",
    "Calle_Numero": "string",
    "FK_Nacionalidad": "string",
    "FK_Estado": "string",
    "FK_Delegacion": "string",
    "Codigo_Postal": "string",
    "FK_Empresa": integer,
    "Ciudad": "string",
    "Password": "string"
}
```

#### Descripción Detallada de Parámetros

| Parámetro | Tipo | Requerido | Descripción | Validaciones/Formato |
|-----------|------|-----------|-------------|---------------------|
| `Bandera` | Integer | Sí | Identificador de operación de inserción | Valor fijo: 100 |
| `Nombre` | String | Sí | Nombre(s) del cliente | Máximo 50 caracteres |
| `APP` | String | Sí | Apellido paterno | Máximo 50 caracteres |
| `APM` | String | No | Apellido materno | Máximo 50 caracteres |
| `Email` | String | Sí | Correo electrónico único | Formato email válido |
| `RFC` | String | Sí | Registro Federal de Contribuyentes | 10-13 caracteres |
| `CURP` | String | Sí | Clave Única de Registro de Población | 18 caracteres exactos |
| `Fecha_Nac` | String | Sí | Fecha de nacimiento | Formato: DD/MM/YYYY |
| `FK_TipoPersona` | Integer | Sí | Tipo de persona (1=Física, 2=Moral) | Catálogo interno |
| `FK_PaisNacimiento` | Integer | Sí | ID del país de nacimiento | Referencia catálogo países |
| `FK_PaisResidencia` | Integer | Sí | ID del país de residencia | Referencia catálogo países |
| `FK_Genero` | Integer | Sí | Género del cliente | 1=Masculino, 2=Femenino |
| `Celular` | String | Sí | Número de teléfono celular | Incluir código de país |
| `Calle_Numero` | String | Sí | Dirección completa | Máximo 200 caracteres |
| `FK_Nacionalidad` | String | Sí | Código de nacionalidad | Código ISO de 3 letras |
| `FK_Estado` | String | Sí | Código del estado | Catálogo de estados |
| `FK_Delegacion` | String | Sí | Código de delegación/municipio | Catálogo de municipios |
| `Codigo_Postal` | String | Sí | Código postal | 5 dígitos |
| `FK_Empresa` | Integer | Sí | ID de empresa asociada | Obtenido de variable global |
| `Ciudad` | String | Sí | Ciudad de residencia | Máximo 100 caracteres |
| `Password` | String | Sí | Contraseña del usuario | Mínimo 8 caracteres, incluir mayúsculas, números y caracteres especiales |

---

## 5. Módulo de Gestión de Créditos

Este módulo gestiona todo el ciclo de vida de los productos crediticios, desde la consulta de productos disponibles hasta la creación y administración de créditos.

### 5.1 Consultar Productos Disponibles

Obtiene el catálogo de productos crediticios disponibles para una empresa específica.

#### Especificación del Endpoint

- **URL:** `/clsAdminProducto/listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Origen del Valor |
|-----------|------|-------------|------------------|
| `Bandera` | Integer | Identificador de operación | Valor fijo: 417 |
| `FK_Empresa` | Integer | ID de la empresa | Variable global `fk_empresa` |
| `FK_TipoProducto` | Integer | Tipo de producto crediticio | 1 = Crédito simple |

#### Respuesta

Retorna un arreglo con los productos disponibles, cada uno conteniendo:
- ID del producto
- Nombre del producto
- Tasas de interés
- Plazos disponibles
- Montos mínimos y máximos
- Condiciones específicas

### 5.2 Consultar Catálogos de Carteras Accesibles para un Vendedor

Obtiene las carteras de crédito a las que un vendedor específico tiene acceso según su perfil y permisos.

#### Especificación del Endpoint

- **URL:** `/clsEmpresaCarteras/listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Origen del Valor |
|-----------|------|-------------|------------------|
| `Bandera` | Integer | Identificador de operación | Valor fijo: 401 |
| `FK_Empresa` | Integer | ID de la empresa | Variable global `fk_empresa` |
| `FK_Rama` | Integer | Rama o línea de negocio | 1 = Rama principal |
| `FK_Usuario` | Integer | ID del vendedor/usuario | Variable `fk_login_name` del token |

### 5.3 Consultar Catálogos de Plazos

Obtiene el catálogo de plazos disponibles para un producto específico, permitiendo conocer las opciones de financiamiento.

#### Especificación del Endpoint

- **URL:** `/clsCredito/listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Origen del Valor |
|-----------|------|-------------|------------------|
| `Bandera` | Integer | Identificador de operación | Valor fijo: 438 |
| `PKey` | Integer/String | ID de la empresa | Variable global `fk_empresa` |
| `FK_Producto` | Integer | ID del producto a consultar | Obtenido del catálogo de productos |

#### Ejemplo de Request

```
GET {{base_url}}/clsCredito/listar?Bandera=438&PKey={{fk_empresa}}&FK_Producto=17364
```

#### Response

Retorna un arreglo con los plazos disponibles para el producto, conteniendo:
- Número de periodos disponibles
- Tipo de plazo (diario, semanal, mensual)
- Tasas de interés asociadas a cada plazo
- Monto mínimo y máximo por plazo
- Condiciones especiales si aplican

### 5.4 Generar Cotización y Tabla de Amortización

Genera una cotización detallada y la tabla de amortización para un crédito potencial sin crear el crédito en el sistema.

#### Especificación del Endpoint

- **URL:** `/clsCredito/listar`
- **Método HTTP:** `POST`
- **Autenticación Requerida:** Sí

#### Body del Request

```json
{
    "Bandera": "437",
    "FK_LoginName": integer,
    "Plazo": integer,
    "Monto": decimal,
    "FK_TipoPlazo": integer,
    "FK_Producto": integer,
    "PKey": "string"
}
```

#### Descripción de Parámetros

| Parámetro | Tipo | Descripción | Validaciones |
|-----------|------|-------------|--------------|
| `Bandera` | String | Identificador de operación de cotización | Valor fijo: "437" |
| `FK_LoginName` | Integer | ID del usuario solicitante | Obtenido del token |
| `Plazo` | Integer | Número de periodos del crédito | Mínimo 1, máximo según producto |
| `Monto` | Decimal | Monto del crédito solicitado | Según límites del producto |
| `FK_TipoPlazo` | Integer | Tipo de plazo | 1=Diario, 2=Semanal, 3=Mensual, 4=Anual |
| `FK_Producto` | Integer | ID del producto seleccionado | Obtenido de consulta de productos |
| `PKey` | String | ID de la empresa | Variable `fk_empresa` como string |

#### Response

La respuesta incluye:
- Tabla de amortización completa con fechas de pago
- Desglose de capital e intereses por periodo
- Monto total a pagar
- CAT (Costo Anual Total)
- Comisiones aplicables

### 5.5 Insertar Crédito Tipo Producto 1

Crea un nuevo crédito en el sistema con todas sus características y parámetros.

#### Especificación del Endpoint

- **URL:** `/clsCredito/Insertar`
- **Método HTTP:** `POST`
- **Autenticación Requerida:** Sí

#### Body del Request Completo

```json
{
    "Bandera": 100,
    "Nombre": "string",
    "Plazo": integer,
    "FK_TipoPlazo": integer,
    "Interes": decimal,
    "FK_Empresa": "string",
    "FK_Usuario_Sol": integer,
    "Monto": decimal,
    "Status": integer,
    "FK_Cartera": integer,
    "FK_ConductoCobro": integer,
    "FK_TipoCredito": integer,
    "Detalle": integer,
    "FK_LoginName": integer,
    "FK_Producto": integer,
    "fJSON": "string",
    "FechaSolicitud": "string"
}
```

#### Descripción Detallada de Parámetros de Crédito

| Parámetro | Tipo | Descripción | Valores/Origen |
|-----------|------|-------------|----------------|
| `Bandera` | Integer | Operación de inserción | Valor fijo: 100 |
| `Nombre` | String | Nombre del titular del crédito | Nombre completo del cliente |
| `Plazo` | Integer | Duración del crédito | Número de periodos |
| `FK_TipoPlazo` | Integer | Periodicidad de pagos | 1=Diario, 2=Semanal, 3=Mensual |
| `Interes` | Decimal | Tasa de interés override | 0 = usar tasa del producto |
| `FK_Empresa` | String | ID de empresa | Variable `fk_empresa` como string |
| `FK_Usuario_Sol` | Integer | Usuario solicitante | Variable `fk_login_name` |
| `Monto` | Decimal | Monto del crédito | Validar contra límites del producto |
| `Status` | Integer | Estado inicial del crédito | 17 = En proceso |
| `FK_Cartera` | Integer | Cartera asignada | Obtenido de catálogo de carteras |
| `FK_ConductoCobro` | Integer | Método de cobro | 1 = Cobro directo |
| `FK_TipoCredito` | Integer | Clasificación del crédito | 17 = Crédito simple |
| `Detalle` | Integer | Nivel de detalle | 2 = Detalle completo |
| `FK_LoginName` | Integer | Usuario que registra | Variable `fk_login_name` |
| `FK_Producto` | Integer | Producto crediticio | ID del catálogo de productos |
| `fJSON` | String | Configuraciones adicionales | JSON con parámetros extra |
| `FechaSolicitud` | String | Fecha de la solicitud | Formato: DD/MM/YYYY |

#### Parámetros del campo fJSON

El campo `fJSON` contiene configuraciones adicionales en formato JSON string:

```json
{
    "BoolConsultarBuro": false
}
```

- `BoolConsultarBuro`: Indica si se debe realizar consulta a buró de crédito

### 5.6 Consultar Créditos por Cliente (Rama 1)

Obtiene la lista de créditos asociados a un cliente específico con filtros avanzados.

#### Especificación del Endpoint

- **URL:** `/clsCredito/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters Completos

| Parámetro | Tipo | Descripción | Valor/Origen |
|-----------|------|-------------|--------------|
| `Bandera` | Integer | Operación de consulta | 432 |
| `FK_LoginName` | Integer | Usuario consultante | Variable `fk_login_name` |
| `FK_Empresa` | Integer | ID de empresa | Variable `fk_empresa` |
| `FK_Rama` | Integer | Rama de negocio | 1 = Principal |
| `FK_TipoProducto` | Integer | Filtro por tipo de producto | 1 = Crédito simple |
| `NumPag` | Integer | Número de página | Paginación de resultados |
| `FK_Usuario_Sol` | Integer | Usuario solicitante del crédito | Para filtrar por solicitante |
| `Detalle` | Integer | Nivel de información | 4 = Detalle extendido para consulta de usuarios de la empresa 4 (opcional) |

---

## 6. Módulo de Consultas de Contratos

Este módulo proporciona acceso a la información detallada de contratos de crédito existentes.

### 6.1 Consultar Tabla de Amortización

Obtiene la tabla de amortización completa de un contrato de crédito específico.

#### Especificación del Endpoint

- **URL:** `/clsCredito/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `Bandera` | Integer | Operación de consulta de amortización | 401 |
| `PKey` | Integer | ID único del contrato de crédito | 158818 |

#### Response

Retorna un objeto con:
- Tabla de amortización detallada por periodo
- Fechas de vencimiento
- Montos de capital
- Montos de interés
- Saldo insoluto
- Estado de cada pago

### 6.2 Consultar Detalles de Contrato

Obtiene información completa y detallada de un contrato de crédito.

#### Especificación del Endpoint

- **URL:** `/clsCredito/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `Bandera` | Integer | Operación de consulta detallada | 445 |
| `PKey` | Integer | ID único del contrato | 158818 |

#### Información Retornada

- Datos del cliente
- Condiciones del crédito
- Estado actual
- Historial de modificaciones
- Documentación asociada
- Garantías si aplican

---

## 7. Módulo de Gestión de Pagos

Este módulo administra todos los procesos relacionados con pagos, abonos y cobros de los créditos.

### 7.1 Consultar Lista de Pagos de un Contrato

Obtiene el historial completo de pagos realizados a un contrato específico.

#### Especificación del Endpoint

- **URL:** `/clsPagos/Listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `Bandera` | Integer | Operación de consulta de pagos | 404 |
| `FK_Credito` | Integer | ID del contrato de crédito | 158798 |

#### Response

Lista de pagos conteniendo:
- Fecha de pago
- Monto pagado
- Método de pago utilizado
- Aplicación (capital/intereses)
- Referencia de pago
- Usuario que registró

### 7.2 Insertar Pago Rama 1 Tipo Producto 1

Registra un nuevo pago o abono a cuenta en el sistema.

#### Especificación del Endpoint

- **URL:** `/clsCuentaInversion/Insertar`
- **Método HTTP:** `POST`
- **Autenticación Requerida:** Sí
- **Content-Type:** `multipart/form-data`

#### Parámetros del Form Data

| Campo | Tipo | Descripción | Validaciones |
|-------|------|-------------|--------------|
| `Bandera` | Text | Operación de inserción | Valor fijo: "100" |
| `FK_Usuario` | Text | ID del usuario que paga | Variable `fk_login_name` |
| `Saldo_Total` | Text | Monto del pago | Formato decimal con 2 decimales |
| `Referencia` | Text | Referencia única del pago | Entre comillas, alfanumérico |
| `FK_MetodoPago` | Text | Método de pago | 1=Efectivo, 2=Transferencia, 3=Tarjeta |
| `FK_TipoMovimiento` | Text | Tipo de movimiento | 1=Abono a cuenta |
| `fJSON` | Text | Datos adicionales en JSON | Ver estructura abajo |
| `FK_Vendedor` | Text | Vendedor que registra | Variable `fk_login_name` |

#### Estructura del campo fJSON

```json
{
    "Moneda": "MXN",
    "FK_Producto": 18443
}
```

- `Moneda`: Código ISO de la moneda (MXN, USD)
- `FK_Producto`: ID del producto al que aplica el pago

### 7.3 Ejecución Manual de Cobro

Ejecuta un proceso de cobro manual para un crédito específico, proceso posterior al abono para que aplique de forma inmediata.

#### Especificación del Endpoint

- **URL:** `/clsPagos/Insertar`
- **Método HTTP:** `POST`
- **Autenticación Requerida:** Sí

#### Body del Request

```json
{
    "Bandera": 101,
    "FK_Credito": 158798
}
```

#### Parámetros

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `Bandera` | Integer | Operación de cobro manual (101) |
| `FK_Credito` | Integer | ID del crédito a cobrar |

#### Proceso de Cobro

Este endpoint ejecuta:
1. Validación del estado del crédito
2. Cálculo de monto a cobrar
3. Aplicación del cobro según método configurado
4. Actualización de saldos
5. Generación de comprobante

---

## 8. Catálogos Adicionales

### 8.1 Consultar Carteras de Empresa

Obtiene el catálogo completo de carteras disponibles para una empresa.

#### Especificación del Endpoint

- **URL:** `/clsEmpresaCarteras/listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Valor |
|-----------|------|-------------|-------|
| `Bandera` | Integer | Operación de consulta general | 400 |
| `FK_Empresa` | Integer | ID de la empresa | Variable `fk_empresa` |

#### Uso de las Carteras

Las carteras son utilizadas para:
- Segmentación de créditos
- Asignación de cobranza
- Reportes segmentados
- Control de accesos por vendedor

---

## 9. Módulo de Movimientos y Transacciones

### 9.1 Consultar Movimientos de Contrato

Obtiene el historial completo de movimientos y transacciones de un contrato.

#### Especificación del Endpoint

- **URL:** `/clsMovimientos/listar`
- **Método HTTP:** `GET`
- **Autenticación Requerida:** Sí

#### Query Parameters

| Parámetro | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `Bandera` | Integer | Operación de consulta | 403 |
| `FK_Usuario` | Integer | Usuario consultante | Variable `fk_login_name` |
| `FK_Credito` | Integer | ID del contrato | 158798 |

#### Tipos de Movimientos Retornados

- Pagos realizados
- Ajustes de saldo
- Cargos por mora
- Bonificaciones
- Reestructuras
- Cambios de estado
- Notas y observaciones

---

## 10. Consideraciones Técnicas

### Manejo de Errores

El sistema implementa un esquema consistente de manejo de errores:

#### Estructura de Error Estándar

```json
{
    "Success": false,
    "ErrorCode": "string",
    "Message": "string",
    "Details": "string"
}
```

### Límites y Restricciones

| Aspecto | Límite | Descripción |
|---------|--------|-------------|
| Tamaño máximo de request | 10 MB | Para requests JSON |
| Timeout de conexión | 30 segundos | Tiempo máximo de espera |
| Rate limiting | 100 requests/minuto | Por token de usuario |
| Paginación | 100 registros | Máximo por página |
| Vigencia del token | 24 horas | Requiere renovación |

### Formato de Fechas

Todos los endpoints manejan fechas en formato: `DD/MM/YYYY`

Ejemplo: `18/09/2025`

### Tipos de Datos Monetarios

- Utilizar tipo `decimal` con 2 decimales de precisión
- No incluir símbolos de moneda en los valores
- Separador decimal: punto (.)
- Sin separadores de miles

---

## 11. Códigos de Respuesta

### Códigos HTTP Utilizados

| Código | Significado | Descripción |
|--------|------------|-------------|
| 200 | OK | Operación exitosa |
| 201 | Created | Recurso creado exitosamente |
| 400 | Bad Request | Error en parámetros o formato |
| 401 | Unauthorized | Token inválido o expirado |
| 403 | Forbidden | Sin permisos para la operación |
| 404 | Not Found | Recurso no encontrado |
| 422 | Unprocessable Entity | Validación de negocio fallida |
| 500 | Internal Server Error | Error interno del servidor |
| 503 | Service Unavailable | Servicio temporalmente no disponible |

### Códigos de Bandera por Módulo

#### Módulo de Usuarios
- 100: Inserción
- 401: Consulta por email
- 402: Consulta por PKey
- 403: Catálogo de países
- 408: Consulta general de clientes con paginación

#### Módulo de Créditos
- 100: Inserción de crédito
- 401: Consulta tabla de amortización
- 417: Consulta de productos disponibles
- 432: Consulta de créditos por cliente
- 437: Generación de cotización
- 438: Consulta de catálogos de plazos
- 445: Detalles de contrato

#### Módulo de Pagos
- 100: Inserción de pago
- 101: Ejecución de cobro
- 404: Consulta de pagos

#### Módulo de Carteras
- 400: Consulta general de carteras
- 401: Carteras accesibles por vendedor

#### Módulo de Movimientos
- 403: Consulta de movimientos por contrato

---

## 12. Mejores Prácticas

### Seguridad

1. **Gestión de Tokens**
   - Almacenar tokens de forma segura
   - Renovar tokens antes de expiración
   - No compartir tokens entre aplicaciones
   - Implementar logout adecuado

2. **Validación de Datos**
   - Validar todos los inputs antes de enviar
   - Sanitizar datos de entrada
   - Verificar tipos de datos
   - Respetar límites de caracteres

3. **Manejo de Información Sensible**
   - No registrar passwords en logs
   - Encriptar datos sensibles en tránsito
   - Implementar HTTPS siempre
   - Ofuscar datos personales en ambientes de prueba

### Optimización de Performance

1. **Uso de Caché**
   - Cachear catálogos que cambian poco
   - Implementar ETags cuando sea posible
   - Utilizar Cache-Control headers apropiados

2. **Paginación**
   - Siempre paginar consultas grandes
   - Usar límites razonables (20-50 registros)
   - Implementar scroll infinito cuando aplique

3. **Requests Concurrentes**
   - Limitar requests paralelos a 5
   - Implementar retry con backoff exponencial
   - Manejar throttling apropiadamente

### Mantenibilidad

1. **Versionado**
   - Mantener compatibilidad hacia atrás
   - Documentar cambios breaking
   - Usar versionado semántico

2. **Logging y Monitoreo**
   - Registrar todos los requests/responses
   - Implementar correlation IDs
   - Monitorear métricas de uso
   - Alertas para errores recurrentes

3. **Documentación**
   - Mantener documentación actualizada
   - Incluir ejemplos de uso
   - Documentar casos edge
   - Proveer colección de Postman actualizada

### Flujos de Trabajo Recomendados

#### Flujo de Alta de Cliente y Crédito

1. **Autenticación**
   - Login → Obtener token y fk_login_name

2. **Preparación de Catálogos**
   - Consultar países
   - Consultar productos disponibles
   - Consultar plazos disponibles por producto
   - Consultar carteras accesibles

3. **Gestión de Cliente**
   - Consultar clientes existentes (general o por email)
   - Validar si existe por email
   - Si no existe, insertar nuevo cliente
   - Guardar PKey del cliente

4. **Cotización**
   - Seleccionar producto y plazo del catálogo
   - Generar cotización con parámetros deseados
   - Revisar tabla de amortización
   - Confirmar con cliente

5. **Creación de Crédito**
   - Insertar crédito con datos confirmados
   - Obtener número de contrato

6. **Seguimiento**
   - Consultar estado del crédito
   - Registrar pagos según se reciban
   - Monitorear movimientos

---

## Anexos

### A. Ejemplo de Flujo Completo

```javascript
// 1. Login
POST /api/access/Signin
{
    "Email": "vendedor@croop.mx",
    "Password": "SecurePass123$",
    "IPAddress": "192.168.1.100",
    "UserAgent": "PostmanRuntime/7.29.0",
    "ServerURL": "api.croop.mx"
}

// Response: { "Token": "eyJhbG...", "UserPkey": 12345 }

// 2. Consultar productos
GET /api/clsAdminProducto/listar?Bandera=417&FK_Empresa=4&FK_TipoProducto=1
Headers: { "Token": "eyJhbG...", "AppID": "eae76274..." }

// 3. Consultar plazos disponibles para el producto
GET /api/clsCredito/listar?Bandera=438&PKey=4&FK_Producto=17364
Headers: { "Token": "eyJhbG...", "AppID": "eae76274..." }

// 4. Generar cotización
POST /api/clsCredito/listar
{
    "Bandera": "437",
    "FK_LoginName": 12345,
    "Plazo": 12,
    "Monto": 50000,
    "FK_TipoPlazo": 3,
    "FK_Producto": 17364,
    "PKey": "4"
}

// 5. Consultar o crear cliente
GET /api/clsUsuario/Listar?Bandera=408&FK_Empresa=4&fJSON={ "FK_TipoUsuario": [6] }&RegistrosPorPag=10&NumPag=1

// 6. Crear crédito
POST /api/clsCredito/Insertar
{
    "Bandera": 100,
    "Nombre": "Juan Pérez García",
    "Plazo": 12,
    // ... resto de parámetros
}
```

### B. Glosario de Términos

| Término | Descripción |
|---------|-------------|
| **PKey** | Primary Key - Identificador único de registros |
| **FK** | Foreign Key - Referencia a otro catálogo |
| **Bandera** | Identificador de operación específica en endpoint multiuso |
| **CAT** | Costo Anual Total |
| **Token JWT** | JSON Web Token para autenticación |
| **Cartera** | Agrupación lógica de créditos |
| **Rama** | Línea de negocio o división operativa |
| **FK_TipoUsuario** | Identificador del tipo de usuario (6=Cliente, etc.) |
| **fJSON** | Campo para parámetros adicionales en formato JSON |

### C. Contacto y Soporte

Para soporte técnico o consultas sobre la API:

- **Documentación completa en línea (centrada en los procesos documentados en este documento y otros):** https://api.croop.mx/
- **Soporte técnico:** soporte@croop.mx
- **Horario de atención:** Lunes a Viernes, 9:00 - 18:00 hrs (CST)

---

*Documento generado para la versión de API desplegada en ambiente de desarrollo.*  
*Última actualización: Septiembre 2025 - Versión 1.1*  
*Cambios: 18-09-2025 - Carga de documentación inicial*

*Esta documentación estará en constante actualización si se ameritan correcciones o actualizaciones de endpoints*

