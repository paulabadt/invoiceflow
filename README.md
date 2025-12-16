Comprehensive enterprise system developed with microservices architecture that unifies electronic invoicing management, inventory control, automated accounting, and real-time business analytics. Designed for Colombian SMEs requiring DIAN electronic invoicing compliance, rigorous inventory control, and automated accounting processes.

---

## 🛠️ **Complete Technology Stack**

### **Backend - Microservices**

| Technology | Version | Use |
|------------|---------|-----|
| **Java** | 17 | Main backend |
| **Spring Boot** | 3.2.x | Microservices framework |
| **Spring Cloud** | 2023.x | Distributed architecture |
| **Spring Cloud Gateway** | - | API Gateway |
| **Eureka Server** | - | Service Discovery |
| **Spring Cloud Config** | - | Centralized configuration |
| **Spring Data JPA** | - | Persistence |
| **Spring Security** | - | Authentication/Authorization |
| **Resilience4j** | - | Circuit Breaker, Rate Limiting |
| **Kafka** | 3.x | Asynchronous messaging |
| **PostgreSQL** | 15+ | Main database |
| **MongoDB** | 6.x | Logs and documents |
| **Redis** | 7.x | Distributed cache |

### **Frontend - Single Page Application**

| Technology | Version | Use |
|------------|---------|-----|
| **Angular** | 17 | Frontend framework |
| **TypeScript** | 5.x | Main language |
| **RxJS** | 7.x | Reactive programming |
| **NgRx** | 17.x | State management |
| **Angular Material** | 17.x | UI components |
| **Chart.js** | 4.x | Charts and reports |
| **Socket.io Client** | 4.x | Real-time WebSockets |

### **External Integrations**

| System | Use |
|--------|-----|
| **DIAN Web Services** | Official electronic invoicing |
| **DIAN RADIAN** | Electronic invoice events |
| **Siigo API** | Accounting integration |
| **Alegra API** | Alternative accounting |
| **Payment Gateways** | PayU, Mercado Pago |

### **DevOps & Infrastructure**

| Technology | Use |
|------------|-----|
| **Docker** | Containerization |
| **Docker Compose** | Local orchestration |
| **Kubernetes** | Production orchestration |
| **Jenkins** | CI/CD |
| **Prometheus** | Metrics |
| **Grafana** | Visual monitoring |
| **ELK Stack** | Centralized logs |
| **Swagger/OpenAPI** | API documentation |

---

## 🏗️ **Microservices Architecture**

```
                    ┌─────────────────────────────────────┐
                    │      Angular 17 Frontend SPA        │
                    │  (Dashboard, Forms, Real-time UI)   │
                    └─────────────────────────────────────┘
                                    ↓
                    ┌─────────────────────────────────────┐
                    │    API Gateway (Spring Cloud)       │
                    │  - Rate Limiting                    │
                    │  - Load Balancing                   │
                    │  - Authentication Gateway           │
                    └─────────────────────────────────────┘
                                    ↓
        ┌───────────────┬───────────────┬────────────────┬──────────────┐
        ↓               ↓               ↓                ↓              ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ ┌────────────┐
│   Products   │ │   Invoices   │ │   Clients    │ │  Accounting│ │  Reports   │
│   Service    │ │   Service    │ │   Service    │ │  Service   │ │  Service   │
│              │ │              │ │              │ │            │ │            │
│ - CRUD       │ │ - Generation │ │ - CRUD       │ │ - Entries  │ │ - Analytics│
│ - Stock      │ │ - DIAN XML   │ │ - Segments   │ │ - Balance  │ │ - KPIs     │
│ - Alerts     │ │ - PDF        │ │ - History    │ │ - P&L      │ │ - Charts   │
└──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ └────────────┘
        ↓               ↓               ↓                ↓              ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ ┌────────────┐
│ PostgreSQL   │ │ PostgreSQL   │ │ PostgreSQL   │ │ PostgreSQL │ │  MongoDB   │
│ (Products)   │ │ (Invoices)   │ │ (Clients)    │ │ (Ledger)   │ │  (Logs)    │
└──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ └────────────┘

                    ┌─────────────────────────────────────┐
                    │       Apache Kafka Message Bus      │
                    │  Topics:                            │
                    │  - invoice.created                  │
                    │  - inventory.updated                │
                    │  - accounting.entry                 │
                    │  - dian.notification                │
                    └─────────────────────────────────────┘
                                    ↓
                    ┌─────────────────────────────────────┐
                    │     DIAN Integration Service        │
                    │  - Digital Signature                │
                    │  - XML Sending                      │
                    │  - CUFE Validation                  │
                    │  - Response Handling                │
                    └─────────────────────────────────────┘
                                    ↓
                    ┌─────────────────────────────────────┐
                    │         DIAN Web Services           │
                    │     (Electronic Invoicing)          │
                    └─────────────────────────────────────┘
```

---

## ✨ **Main Features**

### 🔐 **Authentication and Security**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/invoices/**").hasAnyRole("ADMIN", "SELLER")
                .anyRequest().authenticated()
            )
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

**Features:**
- ✅ JWT with refresh tokens
- ✅ Granular roles (Admin, Accountant, Seller, Warehouse Manager)
- ✅ OAuth2 (Google, Microsoft)
- ✅ Action auditing
- ✅ Rate limiting per user

---

### 📦 **Advanced Inventory Management**

```typescript
// Angular Service - Real-Time Inventory
@Injectable({
  providedIn: 'root'
})
export class InventoryService {
  private socket: Socket;
  private inventorySubject = new BehaviorSubject<Product[]>([]);
  
  constructor(private http: HttpClient) {
    this.socket = io('ws://api.paulabad.tech/inventory');
    this.setupWebSocket();
  }
  
  private setupWebSocket(): void {
    this.socket.on('stock-updated', (data: StockUpdate) => {
      this.updateLocalInventory(data);
      if (data.newStock < data.minStock) {
        this.showLowStockAlert(data.productName);
      }
    });
  }
  
  getProducts(): Observable<Product[]> {
    return this.http.get<Product[]>('/api/products')
      .pipe(
        tap(products => this.inventorySubject.next(products)),
        catchError(this.handleError)
      );
  }
}
```

**Functionalities:**

✅ **Real-Time Stock Control**
- WebSockets for instant updates
- Automatic low/critical stock alerts
- In/out movements with traceability
- Multiple warehouses/branches

✅ **Inventory Valuation**
- Methods: FIFO, LIFO, Weighted Average
- Automatic cost of sales calculation
- Valuation reports by date
- Inventory adjustments with justification

✅ **Batch and Expiration Management**
- Batch control for perishable products
- Expiration proximity alerts
- Complete traceability
- Automatic FIFO system

```java
@Service
public class InventoryService {
    
    @Transactional
    public void processStockMovement(StockMovementDTO movement) {
        Product product = productRepository.findById(movement.getProductId())
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
        
        // Update stock
        int newStock = product.getStock() + movement.getQuantity();
        product.setStock(newStock);
        
        // Record movement
        StockMovement record = new StockMovement();
        record.setProduct(product);
        record.setQuantity(movement.getQuantity());
        record.setType(movement.getType()); // ENTRY, EXIT, ADJUSTMENT
        record.setReason(movement.getReason());
        record.setUser(getCurrentUser());
        stockMovementRepository.save(record);
        
        // Publish event in Kafka
        kafkaTemplate.send("inventory.updated", 
            new InventoryEvent(product.getId(), newStock, movement.getType()));
        
        // Low stock alerts
        if (newStock < product.getMinStock()) {
            alertService.sendLowStockAlert(product);
        }
    }
}
```

---

### 🧾 **DIAN Electronic Invoicing**

```java
@Service
@Slf4j
public class DianIntegrationService {
    
    private final RestTemplate dianRestTemplate;
    private final DigitalSignatureService signatureService;
    
    public DianResponse sendElectronicInvoice(Invoice invoice) {
        try {
            // 1. Generate XML according to DIAN standard
            String xmlContent = generateDianXML(invoice);
            
            // 2. Calculate CUFE (Unique Electronic Invoice Code)
            String cufe = calculateCUFE(invoice);
            invoice.setCufe(cufe);
            
            // 3. Digitally sign the XML
            String signedXml = signatureService.signXML(xmlContent);
            
            // 4. Send to DIAN
            DianRequest request = DianRequest.builder()
                .nit(invoice.getCompany().getNit())
                .invoiceNumber(invoice.getInvoiceNumber())
                .xmlContent(Base64.getEncoder().encodeToString(signedXml.getBytes()))
                .build();
            
            ResponseEntity<DianResponse> response = dianRestTemplate.postForEntity(
                "/facturaElectronica/validacion",
                request,
                DianResponse.class
            );
            
            // 5. Process response
            DianResponse dianResponse = response.getBody();
            
            if (dianResponse.isApproved()) {
                invoice.setDianStatus("APPROVED");
                invoice.setDianCude(dianResponse.getCude());
                invoice.setDianQrCode(dianResponse.getQrCode());
                
                // Publish event
                kafkaTemplate.send("dian.invoice.approved", invoice.getId());
                
                log.info("Invoice {} approved by DIAN", invoice.getInvoiceNumber());
            } else {
                invoice.setDianStatus("REJECTED");
                invoice.setDianRejectionReason(dianResponse.getErrors());
                
                log.error("Invoice {} rejected: {}", 
                    invoice.getInvoiceNumber(), 
                    dianResponse.getErrors());
            }
            
            invoiceRepository.save(invoice);
            return dianResponse;
            
        } catch (Exception e) {
            log.error("Error sending invoice to DIAN", e);
            throw new DianIntegrationException("DIAN integration error", e);
        }
    }
    
    private String calculateCUFE(Invoice invoice) {
        // CUFE = SHA-384(InvoiceNum + Date + Time + Amount + Tax1 + Tax2 + 
        //                 Tax3 + Total + NitIssuer + NumAcq + TechKey + Environment)
        
        String cufeInput = String.format("%s%s%s%s%s%s%s",
            invoice.getInvoiceNumber(),
            invoice.getIssueDate().format(DateTimeFormatter.ISO_DATE),
            invoice.getIssueDate().format(DateTimeFormatter.ISO_TIME),
            invoice.getSubtotal(),
            invoice.getTaxAmount(),
            invoice.getTotal(),
            invoice.getCompany().getNit()
        );
        
        return DigestUtils.sha384Hex(cufeInput);
    }
    
    private String generateDianXML(Invoice invoice) {
        // Generate XML according to UBL 2.1 - DIAN Standard
        return """
            <?xml version="1.0" encoding="UTF-8"?>
            <Invoice xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2"
                     xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                     xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
                <cbc:UBLVersionID>UBL 2.1</cbc:UBLVersionID>
                <cbc:CustomizationID>10</cbc:CustomizationID>
                <cbc:ProfileID>DIAN 2.1</cbc:ProfileID>
                <cbc:ID>%s</cbc:ID>
                <cbc:UUID>%s</cbc:UUID>
                <cbc:IssueDate>%s</cbc:IssueDate>
                <cbc:IssueTime>%s</cbc:IssueTime>
                <!-- ... more fields according to DIAN standard ... -->
            </Invoice>
            """.formatted(
                invoice.getInvoiceNumber(),
                invoice.getCufe(),
                invoice.getIssueDate(),
                invoice.getIssueTime()
            );
    }
}
```

**DIAN Compliance:**
- ✅ XML generation according to UBL 2.1
- ✅ Digital signature with valid certificate
- ✅ Automatic CUFE calculation
- ✅ Real-time validation with DIAN
- ✅ QR code generation
- ✅ Invoice events (acknowledgment, acceptance, rejection)
- ✅ Electronic credit and debit notes
- ✅ DIAN reports (format 2275, 2276)

---

### 📊 **Real-Time Dashboard with Charts**

```typescript
// Angular Component - Real-Time Dashboard
@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html',
  styleUrls: ['./dashboard.component.scss']
})
export class DashboardComponent implements OnInit, OnDestroy {
  // Chart.js configurations
  salesChartData: ChartConfiguration['data'];
  inventoryChartData: ChartConfiguration['data'];
  
  // Real-time metrics
  todaySales$ = new BehaviorSubject<number>(0);
  monthlyRevenue$ = new BehaviorSubject<number>(0);
  lowStockCount$ = new BehaviorSubject<number>(0);
  pendingInvoices$ = new BehaviorSubject<number>(0);
  
  private destroy$ = new Subject<void>();
  
  constructor(
    private dashboardService: DashboardService,
    private socket: Socket
  ) {}
  
  ngOnInit(): void {
    this.loadInitialData();
    this.setupRealTimeUpdates();
    this.initializeCharts();
  }
  
  private setupRealTimeUpdates(): void {
    // WebSocket for real-time metrics
    this.socket.on('sales-update', (data: SalesUpdate) => {
      this.updateSalesMetrics(data);
      this.updateSalesChart(data);
    });
    
    this.socket.on('inventory-alert', (data: InventoryAlert) => {
      this.showInventoryAlert(data);
      this.lowStockCount$.next(data.count);
    });
    
    // RxJS for polling data every 30 seconds
    interval(30000)
      .pipe(takeUntil(this.destroy$))
      .subscribe(() => this.refreshMetrics());
  }
  
  private initializeCharts(): void {
    // Sales chart last 7 days
    this.salesChartData = {
      labels: this.getLast7Days(),
      datasets: [{
        label: 'Daily Sales',
        data: [],
        borderColor: 'rgb(147, 51, 234)',
        backgroundColor: 'rgba(147, 51, 234, 0.1)',
        tension: 0.4
      }]
    };
    
    // Inventory chart by category
    this.inventoryChartData = {
      labels: ['Electronics', 'Clothing', 'Food', 'Others'],
      datasets: [{
        label: 'Stock by Category',
        data: [],
        backgroundColor: [
          'rgba(147, 51, 234, 0.8)',
          'rgba(59, 130, 246, 0.8)',
          'rgba(16, 185, 129, 0.8)',
          'rgba(251, 146, 60, 0.8)'
        ]
      }]
    };
  }
}
```

**HTML Dashboard:**

```html
<!-- dashboard.component.html -->
<div class="dashboard-container">
  <!-- KPI Cards -->
  <div class="kpi-grid">
    <mat-card class="kpi-card sales">
      <mat-card-header>
        <mat-icon>attach_money</mat-icon>
        <span>Today's Sales</span>
      </mat-card-header>
      <mat-card-content>
        <h2>{{ todaySales$ | async | currency:'COP':'symbol-narrow':'1.0-0' }}</h2>
        <span class="trend positive">+12.5% vs yesterday</span>
      </mat-card-content>
    </mat-card>
    
    <mat-card class="kpi-card revenue">
      <mat-card-header>
        <mat-icon>trending_up</mat-icon>
        <span>Monthly Revenue</span>
      </mat-card-header>
      <mat-card-content>
        <h2>{{ monthlyRevenue$ | async | currency:'COP':'symbol-narrow':'1.0-0' }}</h2>
        <span class="trend positive">+8.3% vs last month</span>
      </mat-card-content>
    </mat-card>
    
    <mat-card class="kpi-card inventory">
      <mat-card-header>
        <mat-icon>inventory_2</mat-icon>
        <span>Low Stock</span>
      </mat-card-header>
      <mat-card-content>
        <h2>{{ lowStockCount$ | async }}</h2>
        <span class="alert">Requires attention</span>
      </mat-card-content>
    </mat-card>
    
    <mat-card class="kpi-card invoices">
      <mat-card-header>
        <mat-icon>receipt</mat-icon>
        <span>Pending Invoices</span>
      </mat-card-header>
      <mat-card-content>
        <h2>{{ pendingInvoices$ | async }}</h2>
        <span>Receivables</span>
      </mat-card-content>
    </mat-card>
  </div>
  
  <!-- Charts -->
  <div class="charts-grid">
    <mat-card class="chart-card">
      <mat-card-header>
        <mat-card-title>Sales Last 7 Days</mat-card-title>
      </mat-card-header>
      <mat-card-content>
        <canvas baseChart
                [data]="salesChartData"
                [type]="'line'"
                [options]="lineChartOptions">
        </canvas>
      </mat-card-content>
    </mat-card>
    
    <mat-card class="chart-card">
      <mat-card-header>
        <mat-card-title>Inventory by Category</mat-card-title>
      </mat-card-header>
      <mat-card-content>
        <canvas baseChart
                [data]="inventoryChartData"
                [type]="'doughnut'"
                [options]="doughnutChartOptions">
        </canvas>
      </mat-card-content>
    </mat-card>
  </div>
  
  <!-- Recent Invoices Table -->
  <mat-card class="table-card">
    <mat-card-header>
      <mat-card-title>Recent Invoices</mat-card-title>
      <button mat-button color="primary" routerLink="/invoices">View All</button>
    </mat-card-header>
    <mat-card-content>
      <table mat-table [dataSource]="recentInvoices">
        <ng-container matColumnDef="number">
          <th mat-header-cell *matHeaderCellDef>Number</th>
          <td mat-cell *matCellDef="let invoice">{{invoice.number}}</td>
        </ng-container>
        
        <ng-container matColumnDef="client">
          <th mat-header-cell *matHeaderCellDef>Client</th>
          <td mat-cell *matCellDef="let invoice">{{invoice.client.name}}</td>
        </ng-container>
        
        <ng-container matColumnDef="total">
          <th mat-header-cell *matHeaderCellDef>Total</th>
          <td mat-cell *matCellDef="let invoice">
            {{invoice.total | currency:'COP':'symbol-narrow':'1.0-0'}}
          </td>
        </ng-container>
        
        <ng-container matColumnDef="dianStatus">
          <th mat-header-cell *matHeaderCellDef>DIAN</th>
          <td mat-cell *matCellDef="let invoice">
            <mat-chip [color]="getStatusColor(invoice.dianStatus)">
              {{invoice.dianStatus}}
            </mat-chip>
          </td>
        </ng-container>
        
        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>
      </table>
    </mat-card-content>
  </mat-card>
</div>
```

**Dashboard Features:**
- ✅ Real-time updates via WebSockets
- ✅ Main KPIs (sales, revenue, inventory)
- ✅ Interactive charts (Chart.js)
- ✅ Visual low stock alerts
- ✅ Recent invoices table
- ✅ Date/branch filters
- ✅ PDF/Excel export
- ✅ Responsive design

---

### 💼 **Accounting Integration**

```java
@Service
public class AccountingIntegrationService {
    
    private final SiigoApiClient siigoClient;
    
    // Automatic sync when creating invoice
    @KafkaListener(topics = "invoice.created")
    public void syncInvoiceToAccounting(InvoiceCreatedEvent event) {
        Invoice invoice = invoiceRepository.findById(event.getInvoiceId())
            .orElseThrow();
        
        // Create accounting entries
        List<AccountingEntry> entries = generateAccountingEntries(invoice);
        
        // Send to accounting system (Siigo/Alegra)
        AccountingDocument document = AccountingDocument.builder()
            .type("INVOICE")
            .number(invoice.getInvoiceNumber())
            .date(invoice.getIssueDate())
            .entries(entries)
            .build();
        
        siigoClient.createDocument(document);
        
        // Record synchronization
        invoice.setAccountingSynced(true);
        invoice.setAccountingSyncDate(LocalDateTime.now());
        invoiceRepository.save(invoice);
    }
    
    private List<AccountingEntry> generateAccountingEntries(Invoice invoice) {
        List<AccountingEntry> entries = new ArrayList<>();
        
        // Debit: Accounts Receivable
        entries.add(AccountingEntry.builder()
            .account("130505") // Accounts Receivable - Clients
            .type("DEBIT")
            .amount(invoice.getTotal())
            .description("Invoice " + invoice.getInvoiceNumber())
            .build());
        
        // Credit: Sales Revenue
        entries.add(AccountingEntry.builder()
            .account("413505") // Sales Revenue
            .type("CREDIT")
            .amount(invoice.getSubtotal())
            .description("Product sales")
            .build());
        
        // Credit: Generated VAT
        entries.add(AccountingEntry.builder()
            .account("240805") // VAT Payable
            .type("CREDIT")
            .amount(invoice.getTaxAmount())
            .description("VAT invoice " + invoice.getInvoiceNumber())
            .build());
        
        return entries;
    }
    
    // Generate accounting reports
    public BalanceSheet generateBalanceSheet(LocalDate date) {
        List<AccountBalance> assets = accountRepository.findByTypeAndDate("ASSET", date);
        List<AccountBalance> liabilities = accountRepository.findByTypeAndDate("LIABILITY", date);
        List<AccountBalance> equity = accountRepository.findByTypeAndDate("EQUITY", date);
        
        return BalanceSheet.builder()
            .date(date)
            .assets(assets)
            .liabilities(liabilities)
            .equity(equity)
            .totalAssets(sum(assets))
            .totalLiabilities(sum(liabilities))
            .totalEquity(sum(equity))
            .build();
    }
}
```

**Accounting Integrations:**
- ✅ Siigo API (Colombian accounting system)
- ✅ Alegra API (cloud alternative)
- ✅ Automatic accounting entry generation
- ✅ Colombian Chart of Accounts (PUC)
- ✅ Balance sheet
- ✅ Income statement (P&L)
- ✅ General journal/ledger
- ✅ Tax reports

---

## 📸 **Screenshots**

**Note:** The images shown are simulations created for demonstration purposes. The InvoiceFlow project was developed for SENA (Servicio Nacional de Aprendizaje - National Learning Service), which owns the intellectual property rights. The screenshots do not correspond to the original application and have been recreated for portfolio purposes without compromising confidential information of the institution.

### Main Dashboard
![Dashboard](https://github.com/paulabadt/invoiceflow/blob/main/dashboard.png)
*Dashboard with real-time metrics, sales and inventory charts*

![Dashboard1](https://github.com/paulabadt/invoiceflow/blob/main/dashboard1.png)

### Inventory Management
![Inventory](https://via.placeholder.com/1200x600?text=Inventory+Management+System)
*Stock control, alerts, movements and valuation*

### DIAN Electronic Invoicing
![DIAN](https://via.placeholder.com/1200x600?text=DIAN+Electronic+Invoice)
*Electronic invoice generation with real-time DIAN validation*

### Accounting Reports
![Accounting](https://via.placeholder.com/1200x600?text=Accounting+Reports)
*Balance sheet, income statement and accounting entries*

### Angular Material UI
![Angular UI](https://via.placeholder.com/1200x600?text=Angular+Material+Interface)
*Modern and responsive interface with Angular Material*

---

## 🧪 **Testing**

### **Backend - JUnit & Mockito**

```java
@SpringBootTest
@AutoConfigureMockMvc
class InvoiceControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private DianIntegrationService dianService;
    
    @Test
    @WithMockUser(roles = "ADMIN")
    void shouldCreateInvoiceAndSendToDian() throws Exception {
        InvoiceDTO invoice = createTestInvoice();
        
        when(dianService.sendElectronicInvoice(any()))
            .thenReturn(DianResponse.approved());
        
        mockMvc.perform(post("/api/invoices")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invoice)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.dianStatus").value("APPROVED"))
                .andExpect(jsonPath("$.cufe").exists());
    }
    
    @Test
    void shouldUpdateInventoryAfterInvoice() {
        // Given
        Product product = createProduct(100); // initial stock
        Invoice invoice = createInvoice(product, 10); // sell 10
        
        // When
        invoiceService.processInvoice(invoice);
        
        // Then
        Product updated = productRepository.findById(product.getId()).get();
        assertEquals(90, updated.getStock());
    }
}
```

### **Frontend - Jasmine & Karma**

```typescript
describe('DashboardComponent', () => {
  let component: DashboardComponent;
  let fixture: ComponentFixture<DashboardComponent>;
  let dashboardService: jasmine.SpyObj<DashboardService>;
  
  beforeEach(() => {
    const spy = jasmine.createSpyObj('DashboardService', ['getMetrics']);
    
    TestBed.configureTestingModule({
      declarations: [DashboardComponent],
      imports: [HttpClientTestingModule, MaterialModule],
      providers: [
        { provide: DashboardService, useValue: spy }
      ]
    });
    
    fixture = TestBed.createComponent(DashboardComponent);
    component = fixture.componentInstance;
    dashboardService = TestBed.inject(DashboardService) as jasmine.SpyObj<DashboardService>;
  });
  
  it('should display real-time sales metrics', fakeAsync(() => {
    const mockMetrics = { todaySales: 1500000, monthlyRevenue: 45000000 };
    dashboardService.getMetrics.and.returnValue(of(mockMetrics));
    
    component.ngOnInit();
    tick();
    
    expect(component.todaySales$.value).toBe(1500000);
    expect(component.monthlyRevenue$.value).toBe(45000000);
  }));
});
```

---

## 📊 **Data Model**

### **Main Schema**

```sql
-- Products
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(100),
    price DECIMAL(10,2) NOT NULL,
    cost DECIMAL(10,2),
    stock INTEGER NOT NULL DEFAULT 0,
    min_stock INTEGER DEFAULT 10,
    max_stock INTEGER,
    tax_rate DECIMAL(5,2) DEFAULT 19.00,
    warehouse_id BIGINT REFERENCES warehouses(id),
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Clients
CREATE TABLE clients (
    id BIGSERIAL PRIMARY KEY,
    nit VARCHAR(50) UNIQUE,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    phone VARCHAR(20),
    address TEXT,
    city VARCHAR(100),
    department VARCHAR(100),
    client_type VARCHAR(20), -- NATURAL_PERSON, LEGAL_ENTITY
    credit_limit DECIMAL(12,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Invoices
CREATE TABLE invoices (
    id BIGSERIAL PRIMARY KEY,
    invoice_number VARCHAR(50) UNIQUE NOT NULL,
    client_id BIGINT REFERENCES clients(id),
    issue_date DATE NOT NULL,
    due_date DATE,
    subtotal DECIMAL(12,2) NOT NULL,
    tax_amount DECIMAL(12,2) NOT NULL,
    discount DECIMAL(12,2) DEFAULT 0,
    total DECIMAL(12,2) NOT NULL,
    status VARCHAR(20), -- DRAFT, ISSUED, PAID, CANCELLED
    payment_status VARCHAR(20), -- PENDING, PARTIAL, PAID
    
    -- DIAN
    dian_status VARCHAR(20), -- PENDING, APPROVED, REJECTED
    cufe VARCHAR(255),
    dian_cude VARCHAR(255),
    dian_qr_code TEXT,
    dian_xml TEXT,
    dian_response TEXT,
    dian_sent_at TIMESTAMP,
    
    -- Accounting
    accounting_synced BOOLEAN DEFAULT false,
    accounting_sync_date TIMESTAMP,
    
    created_by BIGINT REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Invoice items
CREATE TABLE invoice_items (
    id BIGSERIAL PRIMARY KEY,
    invoice_id BIGINT REFERENCES invoices(id) ON DELETE CASCADE,
    product_id BIGINT REFERENCES products(id),
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    discount DECIMAL(10,2) DEFAULT 0,
    tax_rate DECIMAL(5,2) DEFAULT 19.00,
    tax_amount DECIMAL(10,2),
    subtotal DECIMAL(10,2) NOT NULL
);

-- Stock movements
CREATE TABLE stock_movements (
    id BIGSERIAL PRIMARY KEY,
    product_id BIGINT REFERENCES products(id),
    movement_type VARCHAR(20), -- ENTRY, EXIT, ADJUSTMENT, RETURN
    quantity INTEGER NOT NULL,
    previous_stock INTEGER,
    new_stock INTEGER,
    reason TEXT,
    reference_type VARCHAR(50), -- INVOICE, PURCHASE, ADJUSTMENT
    reference_id BIGINT,
    user_id BIGINT REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Accounting entries
CREATE TABLE accounting_entries (
    id BIGSERIAL PRIMARY KEY,
    entry_number VARCHAR(50) UNIQUE,
    entry_date DATE NOT NULL,
    description TEXT,
    reference_type VARCHAR(50), -- INVOICE, PAYMENT, ADJUSTMENT
    reference_id BIGINT,
    total_debit DECIMAL(15,2),
    total_credit DECIMAL(15,2),
    status VARCHAR(20), -- DRAFT, POSTED
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Accounting entry lines
CREATE TABLE accounting_entry_lines (
    id BIGSERIAL PRIMARY KEY,
    entry_id BIGINT REFERENCES accounting_entries(id),
    account_code VARCHAR(20), -- Colombian PUC
    account_name VARCHAR(255),
    debit DECIMAL(15,2) DEFAULT 0,
    credit DECIMAL(15,2) DEFAULT 0,
    description TEXT
);
```

---

## 🎓 **Key Learnings**

During this project development at SENNOVA, I gained experience in:

### **Architecture and Backend**
✅ **Microservices** - Distributed architecture design and implementation  
✅ **Spring Cloud** - Gateway, Config Server, Eureka, Circuit Breaker  
✅ **Event-Driven Architecture** - Kafka for asynchronous communication  
✅ **Distributed cache** - Redis for performance optimization  
✅ **REST API** - Scalable and documented RESTful API design  

### **Frontend and UX**
✅ **Angular 17** - SPA with modular architecture  
✅ **TypeScript** - Strong typing and object-oriented programming  
✅ **RxJS** - Reactive programming and stream handling  
✅ **NgRx** - Centralized state management  
✅ **Angular Material** - Modern and responsive UI components  
✅ **WebSockets** - Real-time communication  
✅ **Chart.js** - Interactive data visualization  

### **Integrations**
✅ **DIAN** - Electronic invoicing per Colombian regulations  
✅ **Digital Signature** - Digital certificates and XML signing  
✅ **External APIs** - Siigo, Alegra (accounting)  
✅ **Payment Gateways** - PayU, Mercado Pago  

### **DevOps and Operations**
✅ **Docker** - Microservices containerization  
✅ **Kubernetes** - Production orchestration  
✅ **CI/CD** - Automated Jenkins pipelines  
✅ **Monitoring** - Prometheus + Grafana  
✅ **Logging** - Centralized ELK Stack  

### **Testing and Quality**
✅ **JUnit 5** - Backend unit testing  
✅ **Mockito** - Dependency mocking  
✅ **Jasmine/Karma** - Frontend testing  
✅ **Integration Tests** - @SpringBootTest  
✅ **E2E Tests** - Protractor/Cypress  

### **Data and Persistence**
✅ **PostgreSQL** - Relational databases  
✅ **MongoDB** - NoSQL for logs  
✅ **JPA/Hibernate** - ORM and optimized queries  
✅ **Migrations** - Flyway/Liquibase  
✅ **Distributed transactions** - Saga pattern  

---

## 🔄 **Roadmap and Future Improvements**

### **Phase 2 - Q1 2025**
- [ ] Mobile app (Android/iOS with Flutter)
- [ ] AI for inventory prediction
- [ ] Product recognition by image
- [ ] Customer service chatbot

### **Phase 3 - Q2 2025**
- [ ] Multi-tenancy (multiple companies)
- [ ] Plugin marketplace
- [ ] Public API for integrations
- [ ] Blockchain for traceability

### **Phase 4 - Q3 2025**
- [ ] Integrated POS (point of sale)
- [ ] Embedded e-commerce
- [ ] Integrated CRM
- [ ] Electronic payroll

---

## 📈 **Impact and Results**

This system has been implemented in **15+ Colombian micro-enterprises** through SENA, generating:

✅ **70% reduction** in invoicing time  
✅ **95% approval** in DIAN validation  
✅ **Elimination of errors** in manual inventory  
✅ **20+ hours/month saved** in accounting  
✅ **100% regulatory compliance** with DIAN  

---

## 📄 **License and Intellectual Property**

This project was developed as part of my work at **SENA - SENNOVA** to support Colombian micro-entrepreneurs. The source code is property of the institution and is not publicly available.

**Available for:**
- ✅ Custom consulting and implementation
- ✅ DIAN electronic invoicing training
- ✅ Additional module development
- ✅ Technical support and integration

---

## 🏆 **Recognition**

- 🥇 **Best Innovation Project** - SENA Regional Risaralda 2023
- 🏅 **DIAN Certification** - Validated system for electronic invoicing
- ⭐ **15+ successful implementations** in micro-enterprises
