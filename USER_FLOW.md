# KatalogWA — User Flow Diagram & Specification
**Local Business Product Catalog & WhatsApp Direct Ordering System**

---

## 1. High-Level Architecture & User Roles

KatalogWA operates on two distinct user planes:
- **Customer Plane (Public / Zero Friction):** No login, registration, or session state required. Direct catalog browsing, product filtering, quick modal order capture, and pre-formatted WhatsApp redirect (`wa.me`).
- **Admin Back-Office Plane (Protected / JWT-Secured):** Role-based access control, category taxonomy management, product catalog CRUD, and inventory status toggle ("Tersedia" / "Habis").

```mermaid
flowchart LR
    subgraph Roles["System Access Points"]
        User["Customer (Public)"]
        Admin["Admin (Store Owner)"]
    end

    subgraph KatalogWA["KatalogWA Platform"]
        Catalog["Public Web Catalog\n& Search / Filters"]
        Modal["Order Details Modal\n& Message Builder"]
        Auth["JWT Auth / Login\n(/login)"]
        Dashboard["Admin Dashboard\n(/admin)"]
        ProductCRUD["Product CRUD &\nStock Switch"]
        CategoryCRUD["Category CRUD\nManagement"]
    end

    subgraph External["External Services"]
        WA["WhatsApp Web / App\n(wa.me API)"]
    end

    User --> Catalog --> Modal --> WA
    Admin --> Auth --> Dashboard
    Dashboard --> ProductCRUD
    Dashboard --> CategoryCRUD
```

---

## 2. Comprehensive User Flow Diagram

The diagram below details both flows with designated color palettes:
- **Emerald Green (`#10B981`)** for Customer Experience & WhatsApp Actions
- **Slate & Dark Cyan (`#0F766E` / `#1E293B`)** for Admin Back-Office Operations
- **Decision Diamonds** for validation and authentication checkpoints

```mermaid
flowchart TD
    %% ==========================================
    %% SUBGRAPH 1: CUSTOMER EXPERIENCE
    %% ==========================================
    subgraph CustomerExp["Customer Experience (Public - Zero Login)"]
        direction TB
        C_Start(["Start: Customer opens KatalogWA Web Page"])
        C_Catalog["Screen: Product Catalog\n(Responsive Grid, Search Bar, Category Pills)"]
        C_Browse["Action: Search keyword or click category pill\n(Makanan | Minuman | Camilan)"]
        C_Select["Action: Customer clicks '+ Pesan via WA' on Product Card"]
        C_Modal["Modal: Order Details Form\n(Input: Name*, Notes/Address)"]
        C_Check{"Is Form Complete?\n(Name provided & valid)"}
        C_Error["System Alert: Highlight required fields\n(e.g., 'Nama pemesan wajib diisi')"]
        C_Preview["System: Generate formatted WhatsApp text preview\n'Halo Toko Berkah, saya [Nama] mau pesan...'"]
        C_Submit["Action: Customer clicks 'Kirim Pesanan ke WhatsApp'"]
        C_Redirect["System Action: Open WhatsApp API URL\n(wa.me/628xxx?text=...)"]
        C_End(["End: Land on WhatsApp Chat with Merchant"])

        C_Start --> C_Catalog
        C_Catalog --> C_Browse
        C_Browse --> C_Catalog
        C_Catalog --> C_Select
        C_Select --> C_Modal
        C_Modal --> C_Check
        C_Check -- "NO" --> C_Error
        C_Error --> C_Modal
        C_Check -- "YES" --> C_Preview
        C_Preview --> C_Submit
        C_Submit --> C_Redirect
        C_Redirect --> C_End
    end

    %% ==========================================
    %% SUBGRAPH 2: ADMIN BACK-OFFICE
    %% ==========================================
    subgraph AdminBackOffice["Admin Back-Office (Authenticated Experience)"]
        direction TB
        A_Start(["Start: Admin navigates to /login"])
        A_LoginScreen["Screen: Admin Login Form\n(Email & Password Inputs)"]
        A_SubmitLogin["Action: Admin submits credentials"]
        A_CheckAuth{"Are Credentials\nValid?"}
        A_LoginFail["Error Alert: 'Email atau Password salah'\n(Clear password, keep email)"]
        A_AuthSuccess["System: Issue JWT Token in HttpOnly / Secure Storage\nRedirect to /admin/dashboard"]
        A_Dashboard["Screen: Admin Main Dashboard\n(Navigation Sidebar, Metric Overview)"]

        %% Navigation Choices
        A_NavCategory["Action: Select 'Category Management'"]
        A_NavProduct["Action: Select 'Product Management'"]

        %% Category Module
        subgraph ModCategory["Module 1: Category Management"]
            Cat_List["View Category List\n(Name, Item Count, Actions)"]
            Cat_Add["Action: Create New Category\n(Input Name -> Save)"]
            Cat_Edit["Action: Edit Category Name\n(Inline / Modal Edit)"]
            Cat_Del["Action: Delete Category\n(Confirmation prompt)"]
        end

        %% Product Module
        subgraph ModProduct["Module 2: Product Management"]
            Prod_Table["View Product Table\n(Image, Name, Category, Price, Stock Status, Actions)"]
            Prod_Filter["Action: Search or Filter Table"]
            Prod_Toggle["Action: Toggle Stock Status Switch\n('Tersedia' <--> 'Habis')"]
            Prod_Modal["Action: Open 'Tambah / Edit Produk' Modal\n(Name, Cat, Price, Stock, Desc, Photo)"]
            Prod_Save["Action: Click 'Simpan Produk'\n(Validate & Update Storefront)"]
            Prod_Delete["Action: Click 'Hapus Produk'"]
            Prod_Confirm{"Confirm Deletion\nPrompt?"}
            Prod_DelSuccess["System: Remove product from DB"]
        end

        A_Logout["Action: Admin clicks 'Logout'"]
        A_End(["End: Clear Session/Token & Redirect to /login"])

        %% Flow connections
        A_Start --> A_LoginScreen
        A_LoginScreen --> A_SubmitLogin
        A_SubmitLogin --> A_CheckAuth
        A_CheckAuth -- "NO" --> A_LoginFail
        A_LoginFail --> A_LoginScreen
        A_CheckAuth -- "YES" --> A_AuthSuccess
        A_AuthSuccess --> A_Dashboard

        A_Dashboard --> A_NavCategory
        A_Dashboard --> A_NavProduct

        A_NavCategory --> Cat_List
        Cat_List --> Cat_Add --> Cat_List
        Cat_List --> Cat_Edit --> Cat_List
        Cat_List --> Cat_Del --> Cat_List

        A_NavProduct --> Prod_Table
        Prod_Table --> Prod_Filter --> Prod_Table
        Prod_Table --> Prod_Toggle --> Prod_Table
        Prod_Table --> Prod_Modal --> Prod_Save --> Prod_Table
        Prod_Table --> Prod_Delete --> Prod_Confirm
        Prod_Confirm -- "YES" --> Prod_DelSuccess --> Prod_Table
        Prod_Confirm -- "NO" --> Prod_Table

        A_Dashboard --> A_Logout
        A_Logout --> A_End
    end

    %% Class Definitions
    classDef emeraldPrimary fill:#10B981,stroke:#059669,stroke-width:2px,color:#FFFFFF,font-weight:600;
    classDef emeraldLight fill:#ECFDF5,stroke:#10B981,stroke-width:2px,color:#065F46;
    classDef emeraldAction fill:#047857,stroke:#065F46,stroke-width:2px,color:#FFFFFF;
    classDef customerDecision fill:#FEF3C7,stroke:#D97706,stroke-width:2px,color:#92400E,font-weight:600;

    classDef cyanPrimary fill:#0F766E,stroke:#115E59,stroke-width:2px,color:#FFFFFF,font-weight:600;
    classDef slateDark fill:#1E293B,stroke:#475569,stroke-width:2px,color:#F8FAFC,font-weight:600;
    classDef cyanLight fill:#F0FDFA,stroke:#0F766E,stroke-width:2px,color:#134E4A;
    classDef adminDecision fill:#E0E7FF,stroke:#4338CA,stroke-width:2px,color:#312E81,font-weight:600;
    classDef errorNode fill:#FEE2E2,stroke:#DC2626,stroke-width:2px,color:#991B1B;

    %% Apply Classes to Customer Flow
    class C_Start,C_End,C_Redirect emeraldPrimary;
    class C_Catalog,C_Modal,C_Preview emeraldLight;
    class C_Browse,C_Select,C_Submit emeraldAction;
    class C_Check customerDecision;
    class C_Error errorNode;

    %% Apply Classes to Admin Flow
    class A_Start,A_End,A_AuthSuccess slateDark;
    class A_Dashboard,A_LoginScreen,Cat_List,Prod_Table cyanLight;
    class A_SubmitLogin,A_NavCategory,A_NavProduct,Cat_Add,Cat_Edit,Cat_Del,Prod_Filter,Prod_Toggle,Prod_Modal,Prod_Save,Prod_Delete,Prod_DelSuccess,A_Logout cyanPrimary;
    class A_CheckAuth,Prod_Confirm adminDecision;
    class A_LoginFail errorNode;
```

---

## 3. Detailed Step-by-Step Flow Specifications

### 3.1 Flow 1: Customer Order via WhatsApp

| Step # | Screen / State | User Action | System Processing & Validation | Outcome / Next State |
| :--- | :--- | :--- | :--- | :--- |
| **C1** | Landing (`/`) | Customer visits URL | Loads public catalog with real-time stock status from API | Render grid of product cards with photos, prices, category badges, and stock switches |
| **C2** | Catalog Grid | Type search or tap category pill (`Makanan`, `Minuman`, `Camilan`) | Client-side or server query filtering products matching criteria | Filtered grid view |
| **C3** | Product Card | Clicks `+ Pesan via WA` | Checks if product stock status is "Tersedia" (disabled if "Habis") | Opens Order Details Modal with pre-selected product info |
| **C4** | Order Modal | Fills `Nama Pemesan` and optional `Catatan / Alamat Pengiriman` | Form validation: `Name` must be non-empty (minimum 2 chars) | Dynamic preview updates below the form |
| **C5** | Validation Check | Clicks `Kirim Pesanan ke WhatsApp` | • **Invalid:** Highlights required fields with alert red boundary<br>• **Valid:** Encodes text using RFC 3986 `encodeURIComponent()` | Generates WhatsApp deep link |
| **C6** | Redirect | System executes redirection | Constructs `https://wa.me/{STORE_PHONE}?text={ENCODED_MESSAGE}` | Browser/Mobile OS launches WhatsApp application with pre-filled message |

#### WhatsApp Message Template Specification
```text
Halo Toko Berkah, saya [Nama Pemesan] mau pesan:
- Produk: [Nama Produk]
- Harga: Rp [Harga Produk]
- Catatan / Alamat: [Catatan atau "-"]

Terima kasih!
```

---

### 3.2 Flow 2: Admin Authentication & Product Management

| Step # | Module / Screen | Actor Action | Backend / Security Rules | Outcome |
| :--- | :--- | :--- | :--- | :--- |
| **A1** | `/login` | Inputs Email + Password and submits | POST to `/api/auth/login` | Validates credentials against hashed password |
| **A2** | Auth Decision | Credential verification | • **Failure:** Returns 401 Unauthorized<br>• **Success:** Returns Signed JWT token & user profile | Display alert if failed; store token in Secure Cookie / LocalStorage and route to `/admin` |
| **A3** | Dashboard (`/admin`) | Sidebar navigation | Verifies `Authorization: Bearer <token>` in auth guard | Displays Dashboard Overview |
| **A4** | Category Admin (`/admin/categories`) | Add / Edit / Delete category | Validate category unique slug and check for dependent products before deletion | Category table updates instantly |
| **A5** | Product Table (`/admin/products`) | Toggle Stock Switch (`Tersedia` / `Habis`) | Instant optimistic UI update + PATCH request to `/api/products/:id/stock` | Changes live availability immediately on public catalog |
| **A6** | Product Modal Form | Add / Edit product (Name, Category, Price, Stock, Description, Photo) | Form schema validation + multi-part image upload handling | Product is committed to database and reflected in table |
| **A7** | Product Deletion | Click delete -> Confirm alert | DELETE `/api/products/:id` with cascade cleanup | Product removed from catalog |
| **A8** | Logout | Click `Logout` button | Invalidate local token and trigger auth logout API | Redirects back to `/login` |

---

## 4. Key UX Best Practices for KatalogWA

1. **Zero-Friction Conversion for Customers:**
   - No account creation or OTP verification required for buyers.
   - WhatsApp redirect happens in one click after entering their name.
2. **Instant Visual Feedback:**
   - Out-of-stock items ("Habis") display a muted badge and disabled "+ Pesan via WA" button, preventing dead orders.
3. **One-Tap Quick Actions for Admin:**
   - Fast toggle for stock availability directly from the product table without opening the full edit modal.
   - Real-time search and category filtering in the admin table for fast inventory lookups during busy hours.
