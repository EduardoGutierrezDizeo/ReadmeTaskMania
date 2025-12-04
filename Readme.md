##  Documentación: Exportación de Tareas (Excel y PDF)

### 1.  Introducción

En este proyecto se implementó la **exportación de tareas** en dos formatos populares:

* **Excel (.xlsx):** Utilizando la librería **Apache POI**.
* **PDF (.pdf):** Utilizando la librería **OpenPDF** (una alternativa _open source_ de iText 2).

Ambas funcionalidades permiten exportar la información de las tareas almacenadas en la base de datos y descargar los archivos generados directamente desde el _frontend_ mediante peticiones HTTP específicas.

---

### 2.  Dependencias necesarias

Las siguientes dependencias fueron añadidas al archivo `pom.xml` para habilitar las funcionalidades de exportación en el _backend_.

#### 2.1. Apache POI (Exportación a Excel)

Esta dependencia es necesaria para trabajar con formatos Office Open XML (OOXML), incluyendo archivos `.xlsx`.

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

## 2.2.  OpenPDF / iText (Exportación a PDF)

Esta librería se utiliza para la **generación de documentos PDF** de manera sencilla y eficiente, siendo compatible con las versiones antiguas y licencias de iText 2 (una alternativa de código abierto).

---

### Dependencia XML

La siguiente entrada se añadió al archivo `pom.xml` para incluir la librería **OpenPDF**:

```xml
<dependency>
    <groupId>com.github.librepdf</groupId>
    <artifactId>openpdf</artifactId>
    <version>1.3.39</version>
</dependency>
```

## 3.  Arquitectura de Exportación

La implementación de la funcionalidad de exportación sigue el principio de **separación de preocupaciones** (Separation of Concerns, SOC), organizándose en tres capas bien definidas. Esto garantiza una arquitectura limpia, modular y altamente mantenible. 

---

### Capas de la Arquitectura

1.  **`Controller`** (Capa de Presentación):
    * **Función:** Recibe la **petición HTTP** (ej. `/task/export/excel`).
    * **Responsabilidad:** Prepara el objeto de respuesta HTTP (`HttpServletResponse`), establece cabeceras (Content-Type, Content-Disposition) y delega al `Service`.
2.  **`Service`** (Capa de Lógica de Negocio):
    * **Función:** Actúa como intermediario.
    * **Responsabilidad:** Obtiene los datos necesarios (`List<Task>`) desde la **base de datos** y pasa esta lista al `Exporter`.
3.  **`Exporter`** (Capa de Utilidad/Generación):
    * **Función:** Es la capa de bajo nivel que maneja la lógica de formato.
    * **Responsabilidad:** **Genera el archivo físico** (Excel o PDF) a partir de la lista de datos y lo escribe en el `ServletOutputStream` de la respuesta HTTP.

---

##  4. Exportación a Excel con Apache POI

La exportación de datos al formato Excel (`.xlsx`) se maneja mediante la librería **Apache POI**.

### 4.1. Clase `TaskExcelExporter`

Esta clase encapsula toda la lógica necesaria para construir un libro de trabajo Excel. Recibe una lista de objetos `Task` y genera el archivo `XSSFWorkbook`.

* **Librería:** Apache POI (`XSSFWorkbook` y `XSSFSheet` manejan el formato `.xlsx`).
* **Mecanismo:** El archivo se construye en memoria y se escribe directamente en el flujo de salida de la respuesta HTTP.

### Código Principal: `TaskExcelExporter.java`

```java
package com.example.demo.export;

import com.example.demo.model.Task;
import jakarta.servlet.ServletOutputStream;
import jakarta.servlet.http.HttpServletResponse;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;

public class TaskExcelExporter {

    private final XSSFWorkbook workbook;
    private Sheet sheet;
    private final List<Task> taskList;

    private static final String[] HEADERS = {"ID", "Nombre", "Descripción", "Estado"};

    public TaskExcelExporter(List<Task> taskList) {
        this.taskList = taskList;
        this.workbook = new XSSFWorkbook();
    }

    private CellStyle getHeaderStyle() {
        CellStyle style = workbook.createCellStyle();
        Font font = workbook.createFont();
        font.setBold(true);
        font.setFontHeightInPoints((short) 14);
        style.setFont(font);
        style.setFillForegroundColor(IndexedColors.LIGHT_BLUE.getIndex());
        style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
        style.setBorderBottom(BorderStyle.MEDIUM);
        style.setBorderTop(BorderStyle.MEDIUM);
        style.setBorderLeft(BorderStyle.MEDIUM);
        style.setBorderRight(BorderStyle.MEDIUM);
        return style;
    }

    private CellStyle getTitleStyle() {
        CellStyle style = workbook.createCellStyle();
        Font font = workbook.createFont();
        font.setBold(true);
        font.setFontHeightInPoints((short) 18);
        style.setFont(font);
        style.setAlignment(HorizontalAlignment.CENTER);
        return style;
    }

    private void writeHeader() {
        sheet = workbook.createSheet("Tareas");

        // Título
        Row titleRow = sheet.createRow(0);
        Cell titleCell = titleRow.createCell(0);
        titleCell.setCellValue("REPORTE DE TAREAS - TaskMania");
        titleCell.setCellStyle(getTitleStyle());
        sheet.addMergedRegion(new org.apache.poi.ss.util.CellRangeAddress(0, 0, 0, HEADERS.length - 1));

        // Fecha
        Row dateRow = sheet.createRow(1);
        Cell dateCell = dateRow.createCell(0);
        dateCell.setCellValue("Fecha: " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss")));

        // Cabecera de tabla
        Row headerRow = sheet.createRow(3);
        for (int col = 0; col < HEADERS.length; col++) {
            Cell cell = headerRow.createCell(col);
            cell.setCellValue(HEADERS[col]);
            cell.setCellStyle(getHeaderStyle());
        }
    }

    private void writeDataLines() {
        int rowCount = 4;

        CellStyle style = workbook.createCellStyle();
        style.setBorderBottom(BorderStyle.THIN);
        style.setBorderTop(BorderStyle.THIN);
        style.setBorderLeft(BorderStyle.THIN);
        style.setBorderRight(BorderStyle.THIN);

        for (Task task : taskList) {
            Row row = sheet.createRow(rowCount++);

            // ID
            Cell idCell = row.createCell(0);
            idCell.setCellValue(task.getId());
            idCell.setCellStyle(style);

            // Nombre
            Cell nombreCell = row.createCell(1);
            nombreCell.setCellValue(task.getNombre());
            nombreCell.setCellStyle(style);

            // Descripción
            Cell descCell = row.createCell(2);
            descCell.setCellValue(task.getDescripcion());
            descCell.setCellStyle(style);

            // Estado
            Cell estadoCell = row.createCell(3);
            estadoCell.setCellValue(task.isEstado() ? "Completada" : "Pendiente");
            estadoCell.setCellStyle(style);
        }

        // Autoajustar columnas
        for (int i = 0; i < HEADERS.length; i++) {
            sheet.autoSizeColumn(i);
        }
    }

    public void export(HttpServletResponse response) throws IOException {
        try {
            writeHeader();
            writeDataLines();

            ServletOutputStream outputStream = response.getOutputStream();
            workbook.write(outputStream);
            workbook.close();
            outputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
            throw new IOException("Error al generar Excel: " + e.getMessage());
        }
    }
}
```

## 5.  Exportación a PDF con OpenPDF

La exportación de datos al formato PDF (`.pdf`) se realiza utilizando la librería **OpenPDF**.

### 5.1. Clase `TaskPDFExporter`

Esta clase encapsula la lógica para la construcción del documento PDF. Utiliza clases clave de la librería como `Document`, `PdfWriter`, y `PdfPTable` para estructurar el contenido. 

* **Librería:** OpenPDF (basada en las API de iText 2).
* **Mecanismo:** El documento se crea en memoria y se escribe directamente en el flujo de salida (`OutputStream`) de la respuesta HTTP, permitiendo la descarga.

### Código Principal: `TaskPDFExporter.java`

```java
package com.example.demo.export;

import com.example.demo.model.Task;
import com.lowagie.text.*;
import com.lowagie.text.pdf.PdfPCell;
import com.lowagie.text.pdf.PdfPTable;
import com.lowagie.text.pdf.PdfWriter;
import jakarta.servlet.http.HttpServletResponse;

import java.awt.Color;
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;

public class TaskPDFExporter {

    private final List<Task> taskList;

    public TaskPDFExporter(List<Task> taskList) {
        this.taskList = taskList;
    }

    private void writeTableHeader(PdfPTable table) {
        String[] headers = {"ID", "Nombre", "Descripción", "Estado"};

        for (String header : headers) {
            PdfPCell cell = new PdfPCell();
            cell.setBackgroundColor(Color.LIGHT_GRAY);
            cell.setPadding(5);
            cell.setPhrase(new Phrase(header, FontFactory.getFont(FontFactory.HELVETICA_BOLD, 12)));
            table.addCell(cell);
        }
    }

    private void writeTableData(PdfPTable table) {
        Font font = FontFactory.getFont(FontFactory.HELVETICA, 10);

        for (Task task : taskList) {
            table.addCell(new Phrase(String.valueOf(task.getId()), font));
            table.addCell(new Phrase(task.getNombre(), font));
            table.addCell(new Phrase(task.getDescripcion(), font));
            table.addCell(new Phrase(task.isEstado() ? "Completada" : "Pendiente", font));
        }
    }

    public void export(HttpServletResponse response) throws IOException {
        Document document = new Document(PageSize.A4);

        try {
            PdfWriter.getInstance(document, response.getOutputStream());
            document.open();

            // Título
            Font titleFont = FontFactory.getFont(FontFactory.HELVETICA_BOLD, 20);
            Paragraph title = new Paragraph("REPORTE DE TAREAS", titleFont);
            title.setAlignment(Element.ALIGN_CENTER);
            document.add(title);
            document.add(Chunk.NEWLINE);

            // Fecha
            Font dateFont = FontFactory.getFont(FontFactory.HELVETICA, 10);
            Paragraph date = new Paragraph(
                    "Generado: " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss")),
                    dateFont
            );
            date.setAlignment(Element.ALIGN_RIGHT);
            document.add(date);
            document.add(Chunk.NEWLINE);

            // Tabla
            PdfPTable table = new PdfPTable(4);
            table.setWidthPercentage(100);
            table.setSpacingBefore(10f);
            table.setWidths(new float[]{1f, 3f, 4f, 2f});

            writeTableHeader(table);
            writeTableData(table);

            document.add(table);

        } catch (DocumentException e) {
            throw new IOException("Error al generar PDF: " + e.getMessage());
        } finally {
            if (document.isOpen()) {
                document.close();
            }
        }
    }
}
```

## 6.  Controlador con Endpoints de Exportación (`TaskController`)

Esta sección muestra la implementación en el **`TaskController`** para exponer los dos _endpoints_ de descarga. La función principal del controlador es **preparar la respuesta HTTP** (establecer el tipo de contenido y el nombre del archivo) y luego delegar la generación de datos al servicio y a las clases _Exporter_.

---

### 6.1. Endpoint de Exportación a Excel 

Este método maneja la solicitud HTTP para descargar el archivo de tareas en formato **Excel** (`.xlsx`).

* **Ruta:** `GET /task/export/excel`

```java
    @GetMapping("/export/excel")
    @Operation(summary = "Exportar tareas a Excel", description = "Descarga un archivo XLSX con todas las tareas")
    public void exportToExcel(HttpServletResponse response) throws IOException {
        System.out.println("Exportando a Excel...");

        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        DateFormat dateFormatter = new SimpleDateFormat("yyyy-MM-dd_HH-mm-ss");
        String currentDateTime = dateFormatter.format(new Date());

        response.setHeader("Content-Disposition",
                "attachment; filename=tareas_excel_" + currentDateTime + ".xlsx");

        List<Task> listTasks = service.listarEntidades();
        System.out.println("Total tareas encontradas: " + listTasks.size());

        TaskExcelExporter exporter = new TaskExcelExporter(listTasks);
        exporter.export(response);
    }
```

### 6.2. Endpoint de Exportación a PDF 

Este método maneja la solicitud HTTP para descargar el archivo de tareas en formato **PDF** (`.pdf`).

* **Ruta:** `GET /task/export/pdf`

```java
    @GetMapping("/export/pdf")
    @Operation(summary = "Exportar tareas a PDF", description = "Descarga un archivo PDF con todas las tareas")
    public void exportToPDF(HttpServletResponse response) throws DocumentException, IOException {
        System.out.println("Exportando a PDF...");

        response.setContentType("application/pdf");
        DateFormat dateFormatter = new SimpleDateFormat("yyyy-MM-dd_HH-mm-ss");
        String currentDateTime = dateFormatter.format(new Date());

        response.setHeader("Content-Disposition",
                "attachment; filename=tareas_pdf_" + currentDateTime + ".pdf");

        List<Task> listTasks = service.listarEntidades();
        System.out.println("Total tareas encontradas: " + listTasks.size());

        TaskPDFExporter exporter = new TaskPDFExporter(listTasks);
        exporter.export(response);
    }
```

## 7. Resultado Final

Con la implementación de las dependencias (`Apache POI` y `OpenPDF`), la arquitectura de tres capas (`Controller` → `Service` → `Exporter`), y los _endpoints_ configurados en `TaskController`, tu sistema logra los siguientes beneficios y funcionalidades clave:

---

### capturas de pantalla

* **PDF:**

![Imagen7](/img9.png)

* **XLSX:**

![Imagen8](/img10.png)

---

### Beneficios y Funcionalidades Clave

* **Exportar tareas a Excel (.xlsx):** Se puede generar y descargar el listado completo de tareas en un archivo Excel.
* **Exportar tareas a PDF (.pdf):** Se puede generar y descargar el listado completo de tareas en un documento PDF.
* **Descarga correcta desde Angular:** Los _endpoints_ están configurados con las cabeceras HTTP (`Content-Type` y `Content-Disposition`) correctas, asegurando que cualquier cliente _frontend_ (como **Angular**) pueda iniciar la descarga de archivos correctamente. 
* **Mantenimiento de la Arquitectura:** Se mantiene una arquitectura limpia y escalable al separar la lógica de negocio (obtener datos en el Service) de la lógica de presentación/formato (generar el archivo en el Exporter).


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

![Imagen8](/img8.png)