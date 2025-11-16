# Trabajo Final Integrador — Gestión de Productos con Código de Barras

## 1. Descripción general

Este repositorio contiene la segunda parte del Trabajo Final Integrador para **Programación II** y **Bases de Datos I**. Se desarrolló una aplicación Java (JDK 17+) que gestiona un catálogo de **productos** y sus **códigos de barras**, vinculados mediante una relación **1→1 unidireccional**: la clase `Producto` mantiene una referencia obligatoria a `CodigoBarras`, mientras que `CodigoBarras` desconoce a su propietario. La solución emplea **JDBC sin ORM**, respeta el patrón **DAO + Service** y expone un **menú de consola** con operaciones CRUD envueltas en transacciones que ejecutan `commit` o `rollback` según el resultado.

## 2. Cumplimiento detallado de las consignas

La siguiente sección resume cómo se cubre cada requisito del enunciado, con referencias directas al código fuente y a los recursos incluidos.

### 2.1 Diseño y UML

- Se reservaron los archivos de recursos en `doc_resources/`. El diagrama UML se integrará en `doc_resources/uml_relacion_producto_codigo.png` (placeholder) y se vincula en la [Sección 6](#6-diagrama-uml) para incorporarlo apenas se finalice la imagen.
- Las dependencias entre paquetes se reflejan en la estructura bajo `java/src/main/java`, donde cada capa mantiene responsabilidades claras (ver [Sección 3](#3-arquitectura-y-paquetes)).

### 2.2 Entidades y dominio (A → B)

- `entities/Producto.java` define los atributos de negocio (`nombre`, `descripcion`, `categoriaId`, `marcaId`, `precio`, `costo`, `stock`, `fechaAlta`), el identificador `id`, la bandera de baja lógica `eliminado` y la referencia `private CodigoBarras codigoBarras;`, cumpliendo el requisito 1→1 unidireccional.
- `entities/CodigoBarras.java` utiliza `productoId` como clave primaria/foránea compartida, almacena el `gtin13`, el `tipo` (EAN13, UPC, etc.) y el estado `activo`, sin referenciar de vuelta a `Producto`.
- Ambos modelos ofrecen constructores completos y vacíos, getters/setters y un `toString()` legible para apoyo del menú.

### 2.3 Base de datos y scripts SQL

- `scripts/schema.sql` crea la base `producto_barras`, define tablas (`producto`, `codigo_barras`, catálogos auxiliares) e impone la relación 1→1 mediante una clave foránea única (`codigo_barras.producto_id` con `UNIQUE` y `ON DELETE CASCADE`).
- `scripts/sample_data.sql` carga datos reproducibles para categorías, marcas, productos y códigos, facilitando la puesta en marcha desde cero.
- `config/DatabaseConnection` (ver [Sección 3](#3-arquitectura-y-paquetes)) abre conexiones a MySQL reutilizando `database.properties` o overrides por variables/propiedades JVM.

### 2.4 Capa DAO (JDBC + PreparedStatement)

- `dao/GenericDao.java` declara las operaciones básicas (`crear`, `leer`, `leerTodos`, `actualizar`, `eliminar`) comunes a cada entidad.
- `dao/ProductoDao.java` y `dao/CodigoBarrasDao.java` implementan dichas operaciones con `PreparedStatement`, admiten una `Connection` inyectada externamente para compartir transacciones y reutilizan helpers de mapeo para componer entidades completas.
- Ambas clases incluyen búsquedas adicionales: por nombre (`ProductoDao`) y por GTIN (`CodigoBarrasDao`).

### 2.5 Capa Service y transacciones

- `service/ProductoService.java` y `service/CodigoBarrasService.java` validan entradas (campos obligatorios, reglas de negocio), abren transacciones con `setAutoCommit(false)` y aseguran `commit()`/`rollback()` en bloques `try/catch/finally`.
- La lógica impide asignar más de un código a un producto, evita duplicar GTIN y centraliza la baja lógica tanto para productos como para códigos.

### 2.6 Menú de consola y experiencia de uso

- `main/AppMenu.java` arranca desde `Main` y ofrece opciones CRUD completas para productos y códigos de barras, búsquedas específicas y manejo robusto de errores (parseo numérico, IDs inexistentes, entradas vacías).
- Cada opción delega en la capa `service`, capturando mensajes amigables para el usuario.

### 2.7 Entregables adicionales

- Scripts SQL: `schema.sql` + `sample_data.sql` ya disponibles.
- [Video](https://www.youtube.com/watch?v=rH4wfG8qZiA)
- [Informe PDF](https://drive.google.com/file/d/1GonXruvmFCYZCP2xoNPrB0rGGs5IuFn3/view?usp=drive_link)

## 3. Arquitectura y paquetes

La aplicación Java reside en `java/src/main/java` y sigue una arquitectura por capas:

- `config/`: `DatabaseConnection` obtiene los parámetros desde `database.properties`, admite overrides (`DB_PROPERTIES`, propiedades JVM) y expone `getConnection()` reutilizable.
- `entities/`: modelos `Producto` y `CodigoBarras` con atributos de negocio, `id`, banderas de baja lógica y referencia 1→1 desde `Producto`.
- `dao/`: `GenericDao`, `ProductoDao` y `CodigoBarrasDao` con operaciones CRUD, búsquedas específicas y soporte para conexiones externas.
- `service/`: reglas de negocio (`ProductoService`, `CodigoBarrasService`), validaciones de campos, control transaccional con `commit`/`rollback` y administración de la relación 1→1.
- `dto/` y `util/`: componentes auxiliares para encapsular solicitudes/respuestas y validar formatos (por ejemplo, longitud del GTIN).
- `main/`: `AppMenu` y `Main`, responsables de la interacción con el usuario y del ciclo de vida de la aplicación.

## 4. Requisitos previos

- **Java Development Kit (JDK) 17 o superior** (se recomienda 21 para alinearse con la consigna).
- **MySQL 8.0 o compatible**.
- Cliente de línea de comandos para `javac`, `java` y `mysql`.

## 5. Guía paso a paso para reproducir la aplicación

### 🧰 5.0 Preparar el entorno con Maven (macOS y Windows)

El proyecto puede ejecutarse directamente con **Apache Maven**, lo que simplifica la compilación, ejecución y gestión del driver JDBC de MySQL.

## 5.0.1 Instalación de Maven

### 🪟 En Windows

1. Descargar Maven desde:  
   https://maven.apache.org/download.cgi  
   Elegir _Binary zip archive_.
2. Descomprimir en:  
   `C:\Program Files\Apache\maven`
3. Configurar variables de entorno:
   - **MAVEN_HOME** ⇒ `C:\Program Files\Apache\maven`
   - Agregar al **PATH**:  
     `C:\Program Files\Apache\maven\bin`
4. Verificar instalación:
   ```
   mvn -v
   ```

### 🍎 En macOS

1. Verificar Homebrew:
   ```
   brew --version
   ```
   Si no está instalado:
   ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. Instalar Maven:
   ```
   brew install maven
   ```
3. Verificar:
   ```
   mvn -v
   ```

---

## 5.1 Preparar la base de datos

1. Crear la base y todas las tablas requeridas:
   ```
   mysql -u root -p < scripts/schema.sql
   ```
2. Insertar datos de ejemplo:
   ```
   mysql -u root -p < scripts/sample_data.sql
   ```
3. Verificar registros:
   ```
   USE producto_barras;
   SELECT COUNT(*) FROM producto;
   ```
4. (Opcional) Crear un usuario dedicado ejecutando `scripts/E4_seguridad.sql` y ajustar los permisos necesarios.

---

## 5.2 Configurar las credenciales de conexión

El archivo está en:  
`src/main/resources/database.properties`

Ejemplo:

```
jdbc.url=jdbc:mysql://localhost:3306/producto_barras?serverTimezone=America/Argentina/Cordoba&useSSL=false&allowPublicKeyRetrieval=true
jdbc.user=root
jdbc.password=***
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
```

---

## 5.3 Compilar y ejecutar con Maven

```
mvn -q -DskipTests compile
mvn -q -DskipTests exec:java
```

---

## 5.4 Ejecutar sin Maven (opcional)

```
javac -cp "lib/mysql-connector-j-9.0.0.jar" -d out $(find src/main/java -name "*.java")
cp -R src/main/resources/* out/
java -cp "out:lib/mysql-connector-j-9.0.0.jar" main.AppMenu
```

---

Notas:

- El menú imprime las opciones disponibles y continúa hasta que el usuario elige `0` (salir).
- Para recompilar después de cambios, repita `find` + `javac`. Puede eliminar `sources.list` cuando termine.

## 6. Diagrama UML

- El diagrama de clases que refleja la relación 1→1 (paquetes, atributos, métodos y dependencias) se integrará aquí:

  ![Diagrama UML Producto → CodigoBarras](doc_resources/uml_relacion_producto_codigo.png)

  > _Pendiente_: subir la imagen final al repositorio.

## 7. Video de demostración

Enlace al video (10–15 minutos) que presenta al equipo, explica la arquitectura y muestra el flujo CRUD con transacciones:

- [Video de demostración](https://www.youtube.com/watch?v=rH4wfG8qZiA)

## 8. Funcionalidades expuestas por el AppMenu

`AppMenu` ofrece las siguientes acciones, todas respaldadas por la capa `service` y con manejo robusto de entradas inválidas:

1. Crear producto y código de barras en una única transacción.
2. Actualizar producto y código asociado.
3. Dar de baja lógica un producto.
4. Listar todos los productos (incluyendo su código y estado).
5. Buscar productos por coincidencia en el nombre.
6. Buscar producto por GTIN.
7. Crear un código de barras para un producto existente.
8. Consultar código por ID de producto.
9. Listar todos los códigos de barras.
10. Actualizar un código de barras.
11. Baja lógica del código de barras.

Cada opción delega en `ProductoService` o `CodigoBarrasService`, que validan datos, orquestan transacciones (`commit`/`rollback`) y preservan la unicidad de la relación 1→1.

## 9. Estructura del repositorio

La aplicación sigue una arquitectura por capas, con una organización clara y mantenible.  
La estructura del proyecto es la siguiente:

```
📁 **java/**
   └── 📁 **src/main/java/**
       ├── 📁 **config/**
       │     Contiene la clase de conexión JDBC (`DatabaseConnection`),
       │     encargada de leer `database.properties` y proveer `Connection`.
       │
       ├── 📁 **dao/**
       │     Acceso a datos mediante JDBC.
       │     Implementa CRUD con `PreparedStatement` y mapeo a entidades.
       │
       ├── 📁 **dto/**
       │     Objetos de transferencia (request/response) usados por los services.
       │
       ├── 📁 **entities/**
       │     Modelos del dominio: `Producto`, `CodigoBarras`, etc.
       │     Aquí se refleja la relación 1→1 entre entidades.
       │
       ├── 📁 **main/**
       │     Contiene `AppMenu` y la clase principal `Main`.
       │     Gestiona la interfaz de consola y el flujo de uso.
       │
       ├── 📁 **service/**
       │     Lógica de negocio.
       │     Orquesta transacciones (`commit`/`rollback`) y garantiza 1→1.
       │
       └── 📁 **util/**
             Funciones auxiliares: validaciones, formatos, helpers.

📁 **src/main/resources/**
   Archivos de configuración, principalmente:
   - `database.properties` → credenciales y URL de conexión JDBC.

📁 **scripts/**
   ├── `schema.sql` → creación de tablas, claves foráneas y constraints.
   ├── `sample_data.sql` → datos iniciales para pruebas.
   └── Otros scripts (E1...E5) usados para carga masiva o validaciones.

📁 **doc_resources/**
   Diagramas UML, capturas, documentación complementaria para la entrega.

📄 **README.md**
   Documentación principal del proyecto.
```
