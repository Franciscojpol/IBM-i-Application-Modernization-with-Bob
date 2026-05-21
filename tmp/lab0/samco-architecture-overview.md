# SAMCO Architecture Overview (Lab 0 Step 4)

## 1. Scope and Objective

This document summarizes the SAMCO application architecture from source artifacts in:
- SAMCO/QPNLSRC/SAMMNU-Main_menu_application_SAMPLE.MENUSRC
- SAMCO/QDDSSRC/ARTICLE-Article_File.PF
- SAMCO/QDDSSRC/CUSTOMER.PF
- SAMCO/QDDSSRC/ORDER.PF
- SAMCO/QDDSSRC/Rules.mk
- SAMCO/QRPGLESRC and SAMCO/QCLSRC program inventory

The goal is to describe database schema, application layers, business workflows, and integration points for modernization planning.

## 2. Application Summary

SAMCO is an IBM i green-screen order management application.

Main functional domains:
- Master files: articles, customers, providers, countries, families, parameters
- Sales orders: order creation, maintenance, status updates, printing
- Reports and utilities: analysis queries and operational reset/maintenance tasks

Main users:
- Back-office order entry clerks
- Customer service and sales administration
- Purchasing/procurement users
- IBM i operators/administrators

## 3. Menu and Functional Areas

From the SAMMNU menu source, the top navigation groups are:
- Master files
  - ART200 Work with Articles
  - CUS200 Work with Customers
  - ORD201 Work with Customer Orders
  - PRO200/PRO201 Provider functions
  - ORD100C2 Create a Customer Order
- Reports
  - PRO203 Article to purchase
  - Query-based customer and article reports
- Utilities
  - PAR200 parameters, COU200 countries
  - ORD900/ORD901 reset utilities
  - ART801 summary reset, PAR201 IFS output utility
  - Application log display and signoff

## 4. Database Schema

## 4.1 Core Physical Files (PF)

### ARTICLE (FARTI)
Purpose: article master and stock/pricing
Key fields:
- ARID article id
- ARDESC article description
- ARSALEPR sale price
- ARWHSPR warehouse price
- ARTIFA family id
- ARSTOCK, ARMINQTY, ARCUSQTY, ARPURQTY
- ARVATCD vat code
- ARDEL soft-delete code

### CUSTOMER (FCUST)
Purpose: customer master and credit profile
Key fields:
- CUID customer id
- CUSTNM name
- CUPHONE, CUVAT, CUMAIL
- CULINE1..3, CUZIP, CUCITY, CUCOUN
- CULIMCRE credit limit
- CUCREDIT current credit usage
- CULASTORD last order date
- CUDEL soft-delete code

### ORDER (FORDE)
Purpose: order header
Key fields:
- ORID order id
- ORYEAR year
- ORCUID customer id
- ORDATE order date
- ORDATDEL delivery date
- ORDATCLO close date

### DETORD (FDETO)
Purpose: order detail lines
Key fields:
- ODORID order id
- ODYEAR year
- ODLINE line number
- ODARID article id
- ODQTY ordered quantity
- ODQTYLIV delivered quantity
- ODPRICE unit price
- ODTOT line total net
- ODTOTVAT line total with vat

### FAMILLY (FFAMI)
Purpose: article family master
Key fields:
- FAID family id
- FADESC description
- FAVATCD default vat code
- FADEL soft-delete code

### COUNTRY (FCOUN)
Purpose: country reference
Key fields:
- COID country code
- COUNTR country name
- COISO, COISO5, COISO1 iso variants

### PROVIDER (FPROV)
Purpose: supplier master
Key fields:
- PRID provider id
- PROVNM provider name
- PRCONT contact
- PRPHONE, PRVAT, PRMAIL
- PRLINE1..3, PRZIP, PRCITY, PRCOUN
- PRDEL soft-delete code

### ARTIPROV (FARPR)
Purpose: article-provider relationship
Key fields:
- APARID article id
- APPRID provider id
- APPRICE buy price
- APREF provider reference
- APDEL soft-delete code

### PARAMETER (FPARAM)
Purpose: configurable application parameters
Key fields:
- PACODE
- PASUBCODE
- PARM1..PARM5

### VATDEF (FVAT)
Purpose: VAT reference table used by VAT services
Key fields:
- VATCODE
- VATRATE
- VATDESC
- VATDEL soft-delete code

## 4.2 Logical Files (LF) and Access Paths

Examples from QDDSSRC:
- ARTICLE1/ARTICLE2 on ARTICLE
- CUSTOME1/CUSTOME2 on CUSTOMER
- ORDER1/ORDER2/ORDER3 on ORDER
- DETORD1 on DETORD
- FAMILL1 on FAMILLY
- PROVIDE1 on PROVIDER
- ARTIPRO1 on ARTIPROV
- COUNTR1 on COUNTRY

These LFs provide alternate keyed access paths for interactive programs and reporting.

## 4.3 Key Relationships

- ORDER.ORCUID -> CUSTOMER.CUID
- DETORD.(ODORID, ODYEAR) -> ORDER.(ORID, ORYEAR)
- DETORD.ODARID -> ARTICLE.ARID
- ARTICLE.ARTIFA -> FAMILLY.FAID
- ARTICLE.ARVATCD -> VATDEF.VATCODE
- FAMILLY.FAVATCD -> VATDEF.VATCODE
- CUSTOMER.CUCOUN -> COUNTRY.COID
- PROVIDER.PRCOUN -> COUNTRY.COID
- ARTIPROV.APARID -> ARTICLE.ARID
- ARTIPROV.APPRID -> PROVIDER.PRID

Notes:
- Soft-delete pattern is used via DLCODE-derived fields (for example ARDEL, CUDEL, PRDEL, FADEL, VATDEL).
- Some relationships are logical/business relationships and are not always enforced as SQL foreign keys in DDS PF/LF design.

## 5. Application Structure

## 5.1 Presentation Layer

- Menu panel group: SAMMNU
- Display files (DSPF): ART200D, CUS200D, CUS301D, ORD100D, ORD101D, ORD200D, ORD201D, ORD202D, PRO200D, PRO201D, PRO202D, COU200D, COU301D, FAM301D, PAR200D, ART301D
- Printer file (PRTF): ORD500O

## 5.2 Business Logic Layer

Program families:
- Order: ORD100, ORD101, ORD200, ORD201, ORD202, ORD500, ORD700, ORD900, ORD901
- Article: ART200, ART201, ART202, ART300, ART301, ART302, ART400
- Customer: CUS200, CUS300, CUS301
- Provider: PRO200, PRO202, PRO203, PRO300
- Master and utilities: COU300, COU301, FAM300, FAM301, PAR200, PAR300, DAT001, DAT002, TXT001, XML001, LOG100, LOG300

CL orchestration:
- ORD100C and ORD100C2 coordinate order entry flow
- ORD500C supports print flow
- PAR201 handles IFS output utility logic

Service programs and binding:
- FARTICLE, FARTICLEAPI, FCUSTOMER, FCOUNTRY, FFAMILLY, FPROVIDER
- Additional utility bindings: TXT, XML
- VAT business functions implemented in functionsVAT (for example ClcVAT)

## 5.3 Data Access Layer

Two styles coexist:
- RLA style in classic RPG programs (SETLL/READ/READE/CHAIN)
- Embedded SQL in SQLRPGLE programs (DECLARE/OPEN/FETCH/CLOSE, joins)

This dual model is important for modernization sequencing.

## 6. Main Business Workflows

## 6.1 Create Customer Order (High-Level)

1. User starts order creation from menu option ORD100C2.
2. CL creates a temporary DETORD copy in QTEMP and overrides TMPDETORD.
3. ORD100 requests customer selection.
4. User adds article lines, calculates totals and VAT.
5. On confirm:
   - LASTORDNO is locked and incremented
   - ORDER header is written
   - TMPDETORD lines are copied to DETORD
   - ORD500 print flow is called

This staging pattern avoids committing partial orders.

## 6.2 Article and VAT Rule Chain

1. Article points to family and VAT code.
2. Family can provide default VAT code.
3. VAT rate is read from VATDEF.
4. VAT amount is calculated as (net * rate) / 100.

## 6.3 Order Lifecycle

States represented by dates:
- Created: ORDATE
- Delivered: ORDATDEL
- Closed: ORDATCLO

## 7. Technical Architecture and Integration Points

Platform and runtime:
- IBM i (ILE RPG, CL, DDS, Db2 for i)
- makei/Bob build pipeline with Rules.mk and iproj.json

Integration points:
- IWS/REST style exposure through SQLRPGLE modules (for example ART400)
- Query management reports via STRQMQRY menu actions
- IFS-oriented utility flow (PAR201)
- Logging integration through LOG programs and SAM log display utility

Modernization-relevant observations:
- Strong separation of UI files and business programs
- Reusable domain functions in service programs
- Existing SQL assets provide a migration bridge from RLA to SQL-first design
