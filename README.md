# 📊 **InvoiceFlow - Sistema de Facturación para PYMES**

## 🎯 **Descripción**

Sistema de facturación e inventario desarrollado con arquitectura de microservicios, diseñado para pequeñas y medianas empresas que necesitan gestionar sus productos, clientes y procesos de facturación de manera eficiente.

Este proyecto refleja mi experiencia trabajando en SENNOVA/SENA desarrollando soluciones empresariales reales para microempresarios en Colombia.

---

## 🛠️ **Stack Tecnológico**

| Tecnología | Versión | Uso |
|------------|---------|-----|
| **Java** | 17 | Lenguaje principal |
| **Spring Boot** | 3.x | Framework backend |
| **Spring Data JPA** | - | Persistencia de datos |
| **Spring Security** | - | Autenticación y autorización |
| **PostgreSQL** | 14+ | Base de datos |
| **JWT** | - | Tokens de autenticación |
| **Swagger/OpenAPI** | 3.0 | Documentación API |
| **iText PDF** | 7.x | Generación de facturas PDF |
| **Maven** | 3.8+ | Gestión de dependencias |
| **Docker** | - | Contenedorización |

---

## ✨ **Características Principales**

### 🔐 **Autenticación y Seguridad**
- Sistema de login con JWT
- Roles de usuario (Admin, Vendedor, Contador)
- Protección de endpoints con Spring Security
- Refresh tokens para sesiones extendidas

### 📦 **Gestión de Productos**
- CRUD completo de productos
- Control de stock en tiempo real
- Alertas de inventario bajo
- Categorización de productos
- Búsqueda y filtros avanzados

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @PostMapping
    public ResponseEntity<ProductDTO> createProduct(@Valid @RequestBody ProductDTO productDTO) {
        Product product = productService.createProduct(productDTO);
        return ResponseEntity.status(HttpStatus.CREATED).body(product);
    }
    
    @GetMapping("/low-stock")
    public ResponseEntity<List<ProductDTO>> getLowStockProducts() {
        return ResponseEntity.ok(productService.findLowStockProducts());
    }
}
```

### 👥 **Gestión de Clientes**
- Registro de clientes con validación
- Historial de compras por cliente
- Datos fiscales (NIT, dirección)
- Sistema de fidelización básico

### 🧾 **Facturación**
- Generación de facturas PDF profesionales
- Numeración automática correlativa
- Cálculo automático de impuestos (IVA)
- Estados de factura (Pagada, Pendiente, Anulada)
- Descuentos y promociones

```java
@Service
public class InvoiceService {
    
    public byte[] generateInvoicePDF(Long invoiceId) {
        Invoice invoice = invoiceRepository.findById(invoiceId)
            .orElseThrow(() -> new ResourceNotFoundException("Invoice not found"));
        
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        PdfWriter writer = new PdfWriter(baos);
        PdfDocument pdf = new PdfDocument(writer);
        Document document = new Document(pdf);
        
        // Header con logo empresa
        document.add(new Paragraph("FACTURA DE VENTA")
            .setFontSize(20)
            .setBold()
            .setTextAlignment(TextAlignment.CENTER));
        
        // Datos del cliente y empresa
        document.add(new Paragraph("Cliente: " + invoice.getClient().getName()));
        document.add(new Paragraph("NIT: " + invoice.getClient().getTaxId()));
        
        // Tabla de productos
        Table table = new Table(5);
        table.addHeaderCell("Producto");
        table.addHeaderCell("Cantidad");
        table.addHeaderCell("Precio Unit.");
        table.addHeaderCell("IVA");
        table.addHeaderCell("Total");
        
        for (InvoiceItem item : invoice.getItems()) {
            table.addCell(item.getProduct().getName());
            table.addCell(String.valueOf(item.getQuantity()));
            table.addCell("$" + item.getUnitPrice());
            table.addCell("$" + item.getTaxAmount());
            table.addCell("$" + item.getSubtotal());
        }
        
        document.add(table);
        document.add(new Paragraph("TOTAL: $" + invoice.getTotal())
            .setFontSize(16)
            .setBold());
        
        document.close();
        return baos.toByteArray();
    }
}
```

### 📈 **Reportes e Inventario**
- Reporte de ventas por período
- Productos más vendidos
- Control de stock con historial de movimientos
- Valorización de inventario
- Exportación a Excel/PDF

### 📚 **Documentación API**
- Swagger UI integrado
- Documentación automática de endpoints
- Ejemplos de request/response
- Testing interactivo desde el navegador

```java
@Configuration
@OpenAPIDefinition(
    info = @Info(
        title = "InvoiceFlow API",
        version = "1.0",
        description = "API REST para sistema de facturación e inventario",
        contact = @Contact(
            name = "Paula Abad",
            email = "paula@paulabad.tech"
        )
    )
)
public class OpenAPIConfig {
    // Configuración de Swagger
}
```

---

## 🏗️ **Arquitectura**

### **Capas del Sistema**

```
┌─────────────────────────────────────┐
│         Controller Layer            │
│    (REST API Endpoints)             │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         Service Layer               │
│    (Business Logic)                 │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         Repository Layer            │
│    (Data Access - JPA)              │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         PostgreSQL Database         │
└─────────────────────────────────────┘
```

### **Modelo de Datos Simplificado**

```sql
-- Tabla de Productos
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INTEGER NOT NULL,
    min_stock INTEGER DEFAULT 10,
    category VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabla de Clientes
CREATE TABLE clients (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    tax_id VARCHAR(50) UNIQUE,
    email VARCHAR(255),
    phone VARCHAR(20),
    address TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabla de Facturas
CREATE TABLE invoices (
    id BIGSERIAL PRIMARY KEY,
    invoice_number VARCHAR(50) UNIQUE NOT NULL,
    client_id BIGINT REFERENCES clients(id),
    subtotal DECIMAL(10,2),
    tax_amount DECIMAL(10,2),
    total DECIMAL(10,2),
    status VARCHAR(20),
    issue_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 📸 **Capturas de Pantalla**

### Dashboard Principal
![Dashboard](https://via.placeholder.com/800x400?text=Dashboard+Principal)
*Vista general del sistema con métricas en tiempo real*

### Gestión de Productos
![Productos](https://via.placeholder.com/800x400?text=Gestion+de+Productos)
*CRUD de productos con alertas de stock bajo*

### Generación de Facturas
![Factura](https://via.placeholder.com/800x400?text=Factura+PDF)
*Factura PDF generada con diseño profesional*

### Documentación Swagger
![Swagger](https://via.placeholder.com/800x400?text=Swagger+API+Docs)
*Documentación interactiva de la API REST*

---

## 🚀 **Instalación y Uso**

### **Prerrequisitos**
```bash
- Java 17+
- Maven 3.8+
- PostgreSQL 14+
- Docker (opcional)
```

### **Configuración**

1. **Clonar el repositorio** (nota: código privado)
```bash
# Este es un proyecto desarrollado en SENA
# El código fuente no está disponible públicamente
```

2. **Configurar base de datos**
```properties
# application.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/invoiceflow
spring.datasource.username=tu_usuario
spring.datasource.password=tu_password

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

3. **Ejecutar con Maven**
```bash
mvn clean install
mvn spring-boot:run
```

4. **Acceder a la aplicación**
```
API: http://localhost:8080
Swagger UI: http://localhost:8080/swagger-ui.html
```

### **Con Docker**
```bash
docker-compose up -d
```

---

## 📋 **Endpoints Principales**

### **Autenticación**
```http
POST /api/auth/login
POST /api/auth/register
POST /api/auth/refresh-token
```

### **Productos**
```http
GET    /api/products              # Listar productos
POST   /api/products              # Crear producto
GET    /api/products/{id}         # Obtener producto
PUT    /api/products/{id}         # Actualizar producto
DELETE /api/products/{id}         # Eliminar producto
GET    /api/products/low-stock    # Productos con stock bajo
```

### **Clientes**
```http
GET    /api/clients               # Listar clientes
POST   /api/clients               # Crear cliente
GET    /api/clients/{id}          # Obtener cliente
PUT    /api/clients/{id}          # Actualizar cliente
GET    /api/clients/{id}/invoices # Historial de compras
```

### **Facturas**
```http
POST   /api/invoices              # Crear factura
GET    /api/invoices/{id}         # Obtener factura
GET    /api/invoices/{id}/pdf     # Descargar PDF
PUT    /api/invoices/{id}/status  # Cambiar estado
GET    /api/invoices/reports      # Reportes de ventas
```

---

## 🧪 **Testing**

```java
@SpringBootTest
@AutoConfigureMockMvc
class ProductControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldCreateProduct() throws Exception {
        String productJson = """
            {
                "name": "Laptop Dell",
                "price": 1500.00,
                "stock": 10,
                "category": "Electrónica"
            }
            """;
        
        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(productJson))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.name").value("Laptop Dell"));
    }
}
```

---

## 🎓 **Aprendizajes Clave**

Durante el desarrollo de este proyecto en SENNOVA, adquirí experiencia en:

✅ **Arquitectura REST** - Diseño de APIs escalables y mantenibles  
✅ **Spring Security** - Implementación de autenticación y autorización  
✅ **JPA/Hibernate** - Mapeo objeto-relacional y consultas optimizadas  
✅ **Generación de PDFs** - iText para documentos empresariales  
✅ **Testing** - JUnit y MockMvc para pruebas unitarias e integración  
✅ **Documentación** - Swagger/OpenAPI para APIs profesionales  
✅ **Buenas prácticas** - Clean Code, SOLID, patrones de diseño  

---

## 🔄 **Mejoras Futuras**

- [ ] Integración con pasarelas de pago (PayU, Mercado Pago)
- [ ] Sincronización con DIAN (facturación electrónica)
- [ ] Dashboard con gráficas en tiempo real
- [ ] App móvil para vendedores
- [ ] Sistema de notificaciones (email/SMS)
- [ ] Multi-tenancy para múltiples empresas
- [ ] Integración con contabilidad
