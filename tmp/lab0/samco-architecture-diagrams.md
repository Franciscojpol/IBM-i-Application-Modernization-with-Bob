# SAMCO Architecture Diagrams (Lab 0 Step 4)

## 1. Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    CUSTOMER {
        int CUID PK
        string CUSTNM
        string CUCOUN
        decimal CULIMCRE
        decimal CUCREDIT
        int CULASTORD
        string CUDEL
    }

    ORDER {
        int ORID PK
        int ORYEAR PK
        int ORCUID FK
        int ORDATE
        int ORDATDEL
        int ORDATCLO
    }

    DETORD {
        int ODLINE PK
        int ODORID PK
        int ODYEAR PK
        string ODARID FK
        int ODQTY
        int ODQTYLIV
        decimal ODPRICE
        decimal ODTOT
        decimal ODTOTVAT
    }

    ARTICLE {
        string ARID PK
        string ARDESC
        decimal ARSALEPR
        decimal ARWHSPR
        string ARTIFA FK
        int ARSTOCK
        string ARVATCD FK
        string ARDEL
    }

    FAMILLY {
        string FAID PK
        string FADESC
        string FAVATCD FK
        string FADEL
    }

    VATDEF {
        string VATCODE PK
        decimal VATRATE
        string VATDESC
        string VATDEL
    }

    PROVIDER {
        int PRID PK
        string PROVNM
        string PRCOUN FK
        string PRDEL
    }

    ARTIPROV {
        string APARID PK
        int APPRID PK
        decimal APPRICE
        string APREF
        string APDEL
    }

    COUNTRY {
        string COID PK
        string COUNTR
        string COISO
    }

    PARAMETER {
        string PACODE PK
        string PASUBCODE PK
        string PARM1
        string PARM2
    }

    CUSTOMER ||--o{ ORDER : places
    ORDER ||--o{ DETORD : contains
    ARTICLE ||--o{ DETORD : appears_in

    FAMILLY ||--o{ ARTICLE : categorizes
    VATDEF ||--o{ ARTICLE : taxes
    VATDEF ||--o{ FAMILLY : default_tax

    COUNTRY ||--o{ CUSTOMER : resides_in
    COUNTRY ||--o{ PROVIDER : resides_in

    ARTICLE ||--o{ ARTIPROV : sourced_by
    PROVIDER ||--o{ ARTIPROV : supplies
```

## 2. Application Layer Architecture

```mermaid
flowchart TB
    UI[Presentation Layer\nMenus + DSPF + PRTF]
    BL[Business Logic Layer\nRPG/SQLRPGLE + CL + Service Programs]
    DA[Data Access Layer\nRLA and Embedded SQL]
    DB[(Db2 for i\nPF/LF + SQL Objects)]

    UI --> BL
    BL --> DA
    DA --> DB
```

## 3. Business Process Flow: Order Processing

```mermaid
flowchart TD
    A[Menu option ORD100C2] --> B[Create QTEMP DETORD copy]
    B --> C[Override TMPDETORD to QTEMP]
    C --> D[Call ORD100]
    D --> E[Select customer]
    E --> F[Add article lines]
    F --> G[Calculate line total and VAT]
    G --> H{Confirm order?}
    H -- No --> I[Cancel and exit\nNo final DETORD write]
    H -- Yes --> J[Lock LASTORDNO]
    J --> K[Increment order number]
    K --> L[Write ORDER header]
    L --> M[Copy TMPDETORD to DETORD]
    M --> N[Call ORD500 print]
    N --> O[End]
```

## 4. Data Flow Diagram: Inventory and Sales Cycle

```mermaid
flowchart LR
    A[Provider] -->|buy price| B[ARTIPROV]
    B --> C[ARTICLE]
    C --> D[Order Line DETORD]
    D --> E[Order Header ORDER]
    E --> F[Customer]

    C --> G[Stock fields\nARSTOCK ARMINQTY]
    D --> H[Delivered quantity\nODQTYLIV]

    I[FAMILLY] --> C
    J[VATDEF] --> C
    J --> I
```

## 5. System Integration Architecture

```mermaid
flowchart TB
    User[5250 User] --> Menu[SAMMNU Menu]
    Menu --> RPG[RPG Programs]
    RPG --> DB2[(Db2 for i PF/LF)]

    RPG --> CL[CL Orchestration\nORD100C2 ORD500C]
    CL --> Job[IBM i Job Runtime]

    RPG --> SRV[Service Programs\nFARTICLE FCUSTOMER etc]
    SRV --> DB2

    RPG --> IWS[IWS/REST Exposure\nART400]
    IWS --> Client[Web/API Client]

    RPG --> IFS[IFS Output Utility\nPAR201]
```

## 6. Menu Structure Hierarchy

```mermaid
flowchart TD
    M[SAMMNU Main Menu]

    M --> MF[Master files]
    MF --> MF1[ART200 Work with Articles]
    MF --> MF2[CUS200 Work with Customers]
    MF --> MF3[ORD201 Work with Customer Orders]
    MF --> MF4[PRO200 Work with Providers]
    MF --> MF5[PRO201 Display Providers]
    MF --> MF6[ORD100C2 Create Customer Order]

    M --> RP[Reports]
    RP --> RP1[PRO203 Article to purchase]
    RP --> RP2[CUSQRY Customer with open order]
    RP --> RP3[ARTQRY Article by last order date]

    M --> UT[Utilities]
    UT --> UT1[PAR200 Parameters]
    UT --> UT2[COU200 Countries]
    UT --> UT3[ORD900 Reset LASTORDNO]
    UT --> UT4[ORD901 Reset order dates]
    UT --> UT5[ART801 Reset summary fields]
    UT --> UT6[PAR201 IFS output]
    UT --> UT7[Display application log]
    M --> SO[Signoff]
```

## 7. Technology Stack Overview

```mermaid
mindmap
  root((SAMCO Stack))
    IBM i Platform
      ILE runtime
      Job model
      QTEMP usage
    Languages
      RPGLE
      SQLRPGLE
      CLLE
      DDS
    Data
      Db2 for i PF
      Db2 for i LF
      Reference fields from SAMREF
      VATDEF lookup
    UI
      5250 DSPF
      Menu panels
      Printer files
    Integration
      IWS REST module ART400
      QMQRY reports
      IFS utility flow
    Build
      makei
      Rules.mk
      iproj.json
      Bob and Copilot workflow
```
