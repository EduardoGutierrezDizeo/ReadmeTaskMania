## ⚙️ Documentación de Instalación e Implementación de Swagger UI (OpenAPI 3)

Esta documentación detalla los pasos seguidos para **instalar e implementar Swagger UI** (usando la implementación `springdoc-openapi`) en su proyecto Spring Boot, enfocándose en el controlador `TaskController`.

---

### 1.  Adición de Dependencia Maven (o Gradle)

Se añadió la siguiente dependencia al archivo de configuración de su proyecto (`pom.xml` si usa Maven, o `build.gradle` si usa Gradle). Esta dependencia integra la generación de documentación OpenAPI 3 y la interfaz de usuario de Swagger.

* **Dependencia utilizada:**

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.8</version>
</dependency>
```

## 2 Creación y Configuración de OpenAPI

Se creó el archivo de configuración **`OpenAPIConfig.java`** dentro del paquete `com.example.demo.config` para definir la información básica de la documentación de la API. Este archivo es fundamental para configurar los metadatos globales que se mostrarán en la interfaz de **Swagger UI**.

---

### Detalles del Archivo

* **Ubicación:** `src/main/java/com/example/demo/config/OpenAPIConfig.java`
* **Propósito:** Este `@Bean` configura el objeto `OpenAPI` con el título, versión y descripción de la API.

### Contenido

```java
package com.example.demo.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenAPIConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("Documentación de mi API")
                        .version("1.0.0")
                        .description("Documentación generada automáticamente con OpenAPI 3 y Swagger UI"));
    }
}
```

## 3  Anotaciones en TaskController.java

Se implementaron las anotaciones de OpenAPI 3 (del paquete `io.swagger.v3.oas.annotations.*`) directamente en el controlador `TaskController` para documentar los _endpoints_ y los modelos.

---

### 3.1. Anotaciones a Nivel de Clase

Se utilizó la anotación **`@Tag`** para **agrupar y describir** el conjunto de _endpoints_ de este controlador. Esta etiqueta define el nombre del grupo y una descripción que aparecerá en la interfaz de Swagger UI.

### Implementación

```java
import io.swagger.v3.oas.annotations.tags.Tag;
// ... otras importaciones

@RestController
@RequestMapping("/task")
@Tag(name = "Task Controller", description = "CRUD y exportación de tareas") 
public class TaskController {
// ...
}
```

## 3.2. Anotaciones a Nivel de Método (Endpoints) 📌

Cada método del controlador `TaskController` fue anotado individualmente con **`@Operation`** (para el resumen y la descripción) y **`@ApiResponse`** o **`@Parameter`** cuando fue necesario.

Estas anotaciones proporcionan la documentación detallada de cada operación (endpoint) que será visible en Swagger UI.

---

### Tabla de Documentación de Endpoints

| Endpoint | Anotaciones Clave | Propósito |
| :--- | :--- | :--- |
| **`POST /task`** | `@Operation`, `@ApiResponse` | Documenta la creación de la tarea y el código de respuesta esperado (ej. 200). |
| **`GET /task`** | `@Operation` | Documenta la recuperación y listado de todas las tareas. |
| **`GET /task/id/{id}`** | `@Operation`, `@Parameter` | Documenta la búsqueda por ID y describe el parámetro de la ruta (`id`). |
| **`PUT /task/id/{id}`** | `@Operation`, `@Parameter` | Documenta la actualización de una tarea existente y su parámetro. |
| **`DELETE /task/id/{id}`** | `@Operation`, `@ApiResponse` | Documenta la eliminación de una tarea y el código de respuesta. |

## 4.  Acceso a la Documentación (Swagger UI)

Una vez que la aplicación **Spring Boot** se inicia y se ejecuta correctamente, la documentación interactiva generada por **Swagger UI** (a través de la librería `springdoc-openapi`) estará automáticamente disponible.

No se requiere configuración adicional en archivos como `application.properties` o `application.yml`, ya que la dependencia proporciona una configuración automática por defecto.

---

### Cómo Acceder

La interfaz de usuario de Swagger se encuentra accesible a través de la siguiente ruta:

* **URL de Acceso General:** `http://<host>:<puerto>/swagger-ui/index.html`

* **Ejemplo Común (Ejecución Local):** `http://localhost:8080/swagger-ui/index.html`

Simplemente navegue a esta URL en su navegador web para ver la documentación de su **`Task Controller`** y probar sus _endpoints_.

---

### Pantallazos de Swagger:

![Imagen1](/img1.png)

* **Metodo GET:**

![Imagen6](/img6.png)

* **Metodo GET{id}:**

![Imagen2](/img2.png)

* **Metodo POST:**

![Imagen3](/img3.png)

* **Metodo PUT:**

![Imagen4](/img4.png)

* **Metodo DELETE:**

![Imagen5](/img5.png)

* **Metodo GET PDF:**

![Imagen7](/img7.png)

* **Metodo GET XLSX:**

![Imagen7](/img7.png)