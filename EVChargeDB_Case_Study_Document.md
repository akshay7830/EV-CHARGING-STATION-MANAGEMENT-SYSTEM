# UNIVERSITY INSTITUTE OF COMPUTING
### CHANDIGARH UNIVERSITY
**NAAC Grade A+ Accredited University**

---

<br>

# A Case Study On
# EVChargeDB: Advanced Database Management System for Electric Vehicle Charging Station Management

<br>

### **Case Study**

<br>

**Submitted to:**  
**Faculty In-Charge / Assistant Professor**  
Department of Computer Applications  

<br>

**Submitted by:**  
**Akshay**  
**UID:** *(Your Roll No. / UID)*  
Master of Computer Applications (MCA)  

<br>

**in partial fulfilment for the award of the degree of**  
### **Master of Computer Applications**

<br>

**Chandigarh University, Gharuan, Mohali**  
**Academic Session: July 2026 – Nov 2026**

---

\newpage

## INDEX

| SR No. | Topic | Page No. |
|:---:|:---|:---:|
| **1** | Introduction | 3 |
| **2** | Problem Statement | 3 |
| **3** | Objectives | 3 |
| **4** | Requirements Specification | 4 |
| **5** | Tables and Relationships | 5–6 |
| **6** | Primary Keys and Foreign Keys | 7 |
| **7** | Entity-Relationship (ER) Representation | 8 |
| **8** | Use Cases and SQL Queries (Analytical Views) | 8 |
| **9** | Stored Procedures and Automated Database Triggers | 9 |
| **10** | Database Operations and Expected Results | 10 |
| **11** | Conclusion | 10 |
| **12** | **Companion Case Study: AIP (Java Servlets, JDBC & MVC)** | 11–13 |

---

\newpage

## 1. Introduction:

Electric Vehicle (EV) charging station infrastructure requires an intelligent, highly reliable, and transactional database architecture to orchestrate continuous charging operations, track real-time hardware bay availability, process dynamic kilowatt-hour (kWh) billing tariffs, and maintain transparent driver ledger accounts. Traditional manual or loosely coupled web systems suffer from critical vulnerabilities: race conditions leading to double-booked charging bays, unrecorded session telemetry, billing discrepancy during power interruptions, and an absence of immutable transaction audit logs.

**EVChargeDB** delivers an enterprise-grade relational database management system built upon **MySQL Community Server 8.x/9.x** with the **InnoDB Storage Engine**. By placing business integrity rules, slot status state transitions, and tariff balance reconciliations directly inside database stored procedures and automated triggers, the platform enforces strict ACID (Atomicity, Consistency, Isolation, Durability) guarantees at the storage engine tier.

---

## 2. Problem Statement:

Modern electric mobility networks experience concurrent charging and reservation requests across distributed charging hubs. Managing this environment through standard application-level code often produces race conditions, dirty reads, and conflicting slot reservations when multiple drivers attempt to book or plug into the same charger simultaneously. 

Therefore, the system requires a normalized, ACID-compliant relational schema to:
1. Prevent bay double-allocation and conflicting schedules at the database level.
2. Automate charger bay state toggling (`AVAILABLE` $\leftrightarrow$ `OCCUPIED`) in real-time using event-driven triggers.
3. Compute session consumption fees and execute atomic digital wallet deductions via stored procedures.
4. Maintain a non-repudiable audit ledger of financial transactions and driver sign-in security logs.

---

## 3. Objectives:

* To design a normalized relational schema up to **Boyce-Codd Normal Form (BCNF)** across Users, Stations, Charging Bays, Sessions, Bookings, Transactions, and Security Logs.
* To enforce data integrity using domain constraints (`CHECK` constraints for ratings and battery State-of-Charge, `UNIQUE` constraints, and foreign key cascades).
* To execute atomic session termination and tariff calculation via MySQL stored procedure `sp_stop_charging_session`.
* To enforce zero double-booking and automatic slot occupancy locking using event-driven trigger `trg_after_session_start`.
* To automatically release bays back to available status upon charging termination via trigger `trg_after_session_end`.
* To maintain autonomous wallet balance integrity across debits and top-ups via trigger `trg_update_wallet_balance`.
* To generate real-time station occupancy and business intelligence metrics via analytical views `vw_station_overview` and `vw_daily_revenue_analytics`.

---

\newpage

## 4. Requirements Specification:

### 1. User Entity (Drivers, Operators & Administrators)
* **User ID**: Auto-increment unique primary key for each registered profile.
* **Username & Email**: Unique institutional attributes preventing duplicate accounts.
* **Password Hash**: Cryptographic hash enforcing secure credential storage.
* **Role & Wallet Balance**: Restricts operational privilege (`ADMIN`, `OPERATOR`, `DRIVER`) and maintains prepaid balance (default: ₹500.00).

### 2. Charging Station Entity
* **Station ID**: Unique primary identifier for each geographic charging plaza.
* **Name & Location**: Station title, complete address, city, state, and geographic coordinates (latitude, longitude).
* **Tariff Rate & Status**: Base pricing per kWh (e.g., ₹12.50/kWh) and operational state (`ACTIVE`, `INACTIVE`, `MAINTENANCE`).

### 3. Charging Slot (Bay) Entity
* **Slot ID**: Primary key identifying physical charging points.
* **Station ID**: Foreign key binding slot to a parent station plaza.
* **Hardware Specs**: Slot number (`CH-01`), charger category (`AC_LEVEL_2`, `DC_FAST`, `ULTRA_FAST`), connector standard (`CCS2`, `CHAdeMO`, `TYPE_2`), and power rating in kW.
* **Status**: Hardware status (`AVAILABLE`, `OCCUPIED`, `RESERVED`, `OUT_OF_SERVICE`).

### 4. Charging Session Entity
* **Session ID**: Unique identifier for active or concluded charging cycles.
* **Telemetry Data**: Initial battery SOC (%), current SOC (%), target SOC (%), and metered energy delivered in kWh.
* **Tariff & Cost**: Applied rate per kWh and computed total cost in ₹.
* **Status**: Tracks lifecycle state (`CHARGING`, `COMPLETED`, `TERMINATED`).

### 5. Reservation Entity
* **Reservation ID**: Unique booking reference identifier.
* **Schedule & Fee**: Planned arrival timestamp, booking duration (minutes), advance fee (₹50.00), and status (`CONFIRMED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`).

### 6. Transaction & Audit Entities
* **Transaction ID**: Unique primary key tracking monetary flows (`WALLET_TOPUP`, `CHARGING_PAYMENT`, `RESERVATION_FEE`).
* **Audit & Login Records**: Security ledger storing authentication events, timestamps, client IP addresses, and verification statuses (`SUCCESS`, `FAILED`).

---

\newpage

## 5. Tables and Relationships:

**Database Name:** `evcharge_db`

### Table 1: USERS
| Field | Type | Purpose / Constraint | Key |
|:---|:---|:---|:---|
| `user_id` | INT | Auto-increment unique user identifier | Primary Key |
| `username` | VARCHAR(50) | Unique system handle / login name | Unique |
| `password_hash`| VARCHAR(255)| Cryptographic SHA-256 hash | — |
| `full_name` | VARCHAR(100)| Legal driver or administrator name | — |
| `email` | VARCHAR(100)| Unique communication address | Unique |
| `phone` | VARCHAR(20) | Contact telephone number | — |
| `role` | VARCHAR(20) | CHECK (role IN ('ADMIN', 'OPERATOR', 'DRIVER')) | — |
| `wallet_balance`| DECIMAL(10,2)| Prepaid digital funds (Default: 500.00) | — |
| `created_at` | TIMESTAMP | Account registration timestamp | — |

<br>

### Table 2: STATIONS
| Field | Type | Purpose / Constraint | Key |
|:---|:---|:---|:---|
| `station_id` | INT | Auto-increment charging hub identifier | Primary Key |
| `name` | VARCHAR(100)| Official station plaza name | — |
| `address` | VARCHAR(255)| Physical street address | — |
| `city` | VARCHAR(50) | Metropolitan city location | Index |
| `state` | VARCHAR(50) | State / Province | — |
| `latitude` | DECIMAL(10,8)| GPS latitude coordinate | — |
| `longitude` | DECIMAL(11,8)| GPS longitude coordinate | — |
| `pricing_per_kwh`| DECIMAL(6,2)| Base electric tariff per unit (₹) | — |
| `status` | VARCHAR(20) | CHECK (status IN ('ACTIVE','INACTIVE','MAINTENANCE')) | Index |
| `created_at` | TIMESTAMP | Station commissioning timestamp | — |

<br>

### Table 3: CHARGING_SLOTS
| Field | Type | Purpose / Constraint | Key |
|:---|:---|:---|:---|
| `slot_id` | INT | Auto-increment physical bay identifier | Primary Key |
| `station_id` | INT | References `stations(station_id)` | Foreign Key |
| `slot_number` | VARCHAR(20) | Physical bay code (e.g. 'CH-01') | Unique (with station_id) |
| `charger_type`| VARCHAR(20) | CHECK (charger_type IN ('AC_LEVEL_2','DC_FAST','ULTRA_FAST')) | — |
| `connector_type`| VARCHAR(20)| CHECK (connector_type IN ('CCS2','CHAdeMO','TYPE_2','GB_T')) | — |
| `power_rating_kw`| DECIMAL(5,1)| Maximum power delivery (e.g. 150.0 kW) | — |
| `status` | VARCHAR(20) | CHECK (status IN ('AVAILABLE','OCCUPIED','RESERVED','OUT_OF_SERVICE')) | Index |

<br>

### Table 4: CHARGING_SESSIONS
| Field | Type | Purpose / Constraint | Key |
|:---|:---|:---|:---|
| `session_id` | INT | Unique charging cycle transaction ID | Primary Key |
| `reservation_id`| INT | Optional reference to prior reservation | Foreign Key |
| `user_id` | INT | Driver conducting charging session | Foreign Key |
| `slot_id` | INT | Physical charging bay utilized | Foreign Key |
| `start_time` | DATETIME | Session initiation timestamp | — |
| `end_time` | DATETIME | Session termination timestamp | — |
| `initial_soc` | INT | Starting battery percentage (0–100) | — |
| `current_soc` | INT | Real-time battery percentage | — |
| `energy_delivered_kwh`| DECIMAL(8,2)| Metered energy dispensed in kWh | — |
| `rate_per_kwh`| DECIMAL(6,2)| Applicable tariff rate at session start | — |
| `total_cost` | DECIMAL(10,2)| Total billing amount in ₹ | — |
| `status` | VARCHAR(20) | CHECK (status IN ('CHARGING','COMPLETED','TERMINATED')) | Index |

\newpage

### Table 5: RESERVATIONS
| Field | Type | Purpose / Constraint | Key |
|:---|:---|:---|:---|
| `reservation_id`| INT | Auto-increment reservation reference | Primary Key |
| `user_id` | INT | Driver holding booking | Foreign Key |
| `slot_id` | INT | Reserved charging bay | Foreign Key |
| `reservation_time`| DATETIME | Scheduled commencement time | — |
| `duration_minutes`| INT | Reservation window (Default: 60 min) | — |
| `booking_fee` | DECIMAL(8,2)| Advance commitment charge (₹50.00) | — |
| `status` | VARCHAR(20) | CHECK (status IN ('CONFIRMED','IN_PROGRESS','COMPLETED','CANCELLED')) | Index |

<br>

### Table 6: TRANSACTIONS
| Field | Type | Purpose / Constraint | Key |
|:---|:---|:---|:---|
| `transaction_id`| INT | Unique financial audit sequence ID | Primary Key |
| `user_id` | INT | Driver associated with monetary entry | Foreign Key |
| `session_id` | INT | Associated charging session (Nullable) | Foreign Key |
| `amount` | DECIMAL(10,2)| Transaction monetary value in ₹ | — |
| `type` | VARCHAR(30) | CHECK (type IN ('WALLET_TOPUP','CHARGING_PAYMENT','RESERVATION_FEE','REFUND')) | — |
| `payment_method`| VARCHAR(20) | CHECK (payment_method IN ('WALLET','UPI','CREDIT_CARD','NET_BANKING')) | — |
| `status` | VARCHAR(20) | CHECK (status IN ('SUCCESS','PENDING','FAILED')) | — |
| `reference_no` | VARCHAR(64) | Unique bank/system audit reference | Unique |
| `created_at` | TIMESTAMP | Immutable transaction timestamp | Index |

<br>

### Table 7: LOGIN_HISTORY (Security & Audit Trail)
| Field | Type | Purpose / Constraint | Key |
|:---|:---|:---|:---|
| `login_id` | INT | Auto-increment login record ID | Primary Key |
| `user_id` | INT | Authenticated user profile (Nullable) | Foreign Key |
| `username` | VARCHAR(50) | Submitted username handle | — |
| `login_time` | TIMESTAMP | System login timestamp | Index |
| `ip_address` | VARCHAR(50) | Client IP address / network origin | — |
| `status` | VARCHAR(20) | CHECK (status IN ('SUCCESS','FAILED')) | — |

---

\newpage

## 6. Primary Keys and Foreign Keys:

### 1. Primary Keys
A primary key uniquely identifies each entity row without permitting NULL values. In EVChargeDB:
* `users.user_id`, `stations.station_id`, `charging_slots.slot_id`, `charging_sessions.session_id`, `reservations.reservation_id`, `transactions.transaction_id`, and `login_history.login_id` serve as primary identifiers for their respective entities.

### 2. Foreign Keys
Foreign keys enforce referential integrity across operational tables with cascading cleanup:
* `charging_slots.station_id` $\to$ `stations.station_id` (`ON DELETE CASCADE`)
* `charging_sessions.user_id` $\to$ `users.user_id` (`ON DELETE CASCADE`)
* `charging_sessions.slot_id` $\to$ `charging_slots.slot_id` (`ON DELETE CASCADE`)
* `reservations.user_id` $\to$ `users.user_id` (`ON DELETE CASCADE`)
* `reservations.slot_id` $\to$ `charging_slots.slot_id` (`ON DELETE CASCADE`)
* `transactions.user_id` $\to$ `users.user_id` (`ON DELETE CASCADE`)
* `transactions.session_id` $\to$ `charging_sessions.session_id` (`ON DELETE SET NULL`)
* `login_history.user_id` $\to$ `users.user_id` (`ON DELETE CASCADE`)

### 3. Relationship Description:
The relational schema adheres to Third Normal Form (3NF) and BCNF:
* **Stations to Charging Slots (1:M)**: One charging plaza accommodates multiple physical charging bays.
* **Users to Reservations (1:M)**: One driver may hold multiple successive scheduled reservations.
* **Charging Slots to Sessions (1:M)**: One hardware bay hosts multiple distinct charging sessions over time, but at most **one active session** concurrently.
* **Charging Sessions to Transactions (1:1)**: Each completed charging session produces exactly one financial debit invoice in the transaction ledger.

---

\newpage

## 7. Entity-Relationship (ER) Representation:

```text
+-----------------------+                    +-----------------------+
|        STATIONS       |                    |     CHARGING_SLOTS    |
+-----------------------+                    +-----------------------+
| PK: station_id        | 1:M                | PK: slot_id           |
|     name              |------------------->| FK: station_id        |
|     city, state       |                    |     slot_number (UQ)  |
|     pricing_per_kwh   |                    |     charger_type      |
|     status            |                    |     power_rating_kw   |
+-----------------------+                    |     status            |
                                             +-----------------------+
                                                         | 1:M
                                                         v
+-----------------------+                    +-----------------------+
|         USERS         |                    |   CHARGING_SESSIONS   |
+-----------------------+                    +-----------------------+
| PK: user_id           | 1:M                | PK: session_id        |
|     username (UQ)     |------------------->| FK: user_id           |
|     password_hash     |                    | FK: slot_id           |
|     email (UQ)        |                    |     start_time        |
|     role              |                    |     energy_kwh        |
|     wallet_balance    |                    |     total_cost        |
+-----------------------+                    |     status            |
        | 1:M                                +-----------------------+
        |                                                | 1:1
        v                                                v
+-----------------------+                    +-----------------------+
|      TRANSACTIONS     |                    |     LOGIN_HISTORY     |
+-----------------------+                    +-----------------------+
| PK: transaction_id    |                    | PK: login_id          |
| FK: user_id           |<-------------------| FK: user_id           |
| FK: session_id        |                    |     username          |
|     amount, type      |                    |     login_time        |
|     reference_no (UQ) |                    |     ip_address        |
|     status            |                    |     status            |
+-----------------------+                    +-----------------------+
```

---

## 8. Use Cases and SQL Queries (Analytical Views):

### 1. View Real-Time Station Bay Grid & Ratings (`vw_station_overview`)
```sql
CREATE OR REPLACE VIEW vw_station_overview AS
SELECT 
    s.station_id, s.name, s.city, s.pricing_per_kwh, s.status,
    COUNT(cs.slot_id) AS total_slots,
    SUM(CASE WHEN cs.status = 'AVAILABLE' THEN 1 ELSE 0 END) AS available_slots,
    SUM(CASE WHEN cs.status = 'OCCUPIED' THEN 1 ELSE 0 END) AS occupied_slots,
    SUM(CASE WHEN cs.status = 'RESERVED' THEN 1 ELSE 0 END) AS reserved_slots,
    COALESCE(ROUND(AVG(sr.rating), 1), 0.0) AS avg_rating
FROM stations s
LEFT JOIN charging_slots cs ON s.station_id = cs.station_id
LEFT JOIN station_reviews sr ON s.station_id = sr.station_id
GROUP BY s.station_id, s.name, s.city, s.pricing_per_kwh, s.status;

-- Execution:
SELECT * FROM vw_station_overview WHERE city = 'Bengaluru';
```

### 2. View Daily Energy Throughput & Revenue Audit (`vw_daily_revenue_analytics`)
```sql
CREATE OR REPLACE VIEW vw_daily_revenue_analytics AS
SELECT 
    DATE(cs.created_at) AS transaction_date,
    COUNT(DISTINCT cs.session_id) AS total_sessions,
    COALESCE(SUM(cs.energy_delivered_kwh), 0) AS total_energy_kwh,
    COALESCE(SUM(cs.total_cost), 0) AS total_revenue
FROM charging_sessions cs
WHERE cs.status IN ('COMPLETED', 'TERMINATED')
GROUP BY DATE(cs.created_at);

-- Execution:
SELECT * FROM vw_daily_revenue_analytics ORDER BY transaction_date DESC;
```

---

\newpage

## 9. Stored Procedures and Automated Database Triggers:

### 1. Stored Procedure: Execute End Session & Atomic Settlement (`sp_stop_charging_session`)
```sql
DELIMITER //
CREATE PROCEDURE sp_stop_charging_session (
    IN p_session_id INT,
    IN p_final_soc INT,
    IN p_energy_delivered DECIMAL(8,2),
    OUT p_total_cost DECIMAL(10,2),
    OUT p_status_code VARCHAR(20)
)
BEGIN
    DECLARE v_user_id INT;
    DECLARE v_rate DECIMAL(6,2);
    DECLARE v_cost DECIMAL(10,2);
    DECLARE v_ref VARCHAR(64);

    -- Retrieve active session with row-level lock
    SELECT user_id, rate_per_kwh INTO v_user_id, v_rate 
    FROM charging_sessions 
    WHERE session_id = p_session_id AND status = 'CHARGING' FOR UPDATE;

    IF v_user_id IS NULL THEN
        SET p_status_code = 'NOT_FOUND';
        SET p_total_cost = 0.00;
    ELSE
        SET v_cost = ROUND(p_energy_delivered * v_rate, 2);
        SET p_total_cost = v_cost;

        UPDATE charging_sessions
        SET end_time = NOW(), current_soc = p_final_soc,
            energy_delivered_kwh = p_energy_delivered,
            total_cost = v_cost, status = 'COMPLETED'
        WHERE session_id = p_session_id;

        SET v_ref = CONCAT('TXN-CHG-', p_session_id, '-', UNIX_TIMESTAMP());

        -- Insert transaction; database trigger will atomically debit user balance
        INSERT INTO transactions (user_id, session_id, amount, type, payment_method, status, reference_no)
        VALUES (v_user_id, p_session_id, v_cost, 'CHARGING_PAYMENT', 'WALLET', 'SUCCESS', v_ref);

        SET p_status_code = 'SUCCESS';
    END IF;
END //
DELIMITER ;
```

### 2. Triggers: Automatic Bay State Synchronization
```sql
DELIMITER //
-- Locks bay to OCCUPIED when charging initiates
CREATE TRIGGER trg_after_session_start
AFTER INSERT ON charging_sessions FOR EACH ROW
BEGIN
    IF NEW.status = 'CHARGING' THEN
        UPDATE charging_slots SET status = 'OCCUPIED' WHERE slot_id = NEW.slot_id;
    END IF;
END //

-- Releases bay to AVAILABLE when charging concludes
CREATE TRIGGER trg_after_session_end
AFTER UPDATE ON charging_sessions FOR EACH ROW
BEGIN
    IF NEW.status IN ('COMPLETED', 'TERMINATED') AND OLD.status = 'CHARGING' THEN
        UPDATE charging_slots SET status = 'AVAILABLE' WHERE slot_id = NEW.slot_id;
    END IF;
END //
DELIMITER ;
```

### 3. Trigger: Automated Wallet Balance Reconciliation
```sql
DELIMITER //
CREATE TRIGGER trg_update_wallet_balance
AFTER INSERT ON transactions FOR EACH ROW
BEGIN
    IF NEW.status = 'SUCCESS' THEN
        IF NEW.type IN ('WALLET_TOPUP', 'REFUND') THEN
            UPDATE users SET wallet_balance = wallet_balance + NEW.amount WHERE user_id = NEW.user_id;
        ELSEIF NEW.type IN ('CHARGING_PAYMENT', 'RESERVATION_FEE') THEN
            UPDATE users SET wallet_balance = wallet_balance - NEW.amount WHERE user_id = NEW.user_id;
        END IF;
    END IF;
END //
DELIMITER ;
```

---

\newpage

## 10. Database Operations and Expected Results:

### Database Operations Overview
The relational architecture enforces ACID transactions across high-concurrency EV charging operations. Stored procedures eliminate multi-hop network round-trips by executing arithmetic tariff calculations and invoice generation directly inside MySQL. Triggers automate bay state changes without application dependency, while views compute aggregated KPIs without expensive nested loops.

### Expected Results

#### Executing: `SELECT name, city, total_slots, available_slots, occupied_slots, avg_rating FROM vw_station_overview;`
| name | city | total_slots | available_slots | occupied_slots | avg_rating |
|:---|:---|:---:|:---:|:---:|:---:|
| CyberHub Supercharger Hub | Gurugram | 4 | 3 | 1 | 5.0 |
| Indiranagar Green Grid | Bengaluru | 3 | 2 | 0 | 4.0 |
| BKC HyperCharge Station | Mumbai | 3 | 3 | 0 | 0.0 |
| Hitech City EcoCharge Plaza | Hyderabad | 2 | 2 | 0 | 0.0 |

#### Executing: `SELECT transaction_id, user_id, amount, type, payment_method, reference_no, status FROM transactions ORDER BY transaction_id DESC LIMIT 3;`
| transaction_id | user_id | amount | type | payment_method | reference_no | status |
|:---:|:---:|:---:|:---|:---|:---|:---:|
| 105 | 3 | 411.08 | CHARGING_PAYMENT | WALLET | TXN-CHG-1-1729780001 | SUCCESS |
| 104 | 3 | 50.00 | RESERVATION_FEE | WALLET | TXN-RES-4-1729778500 | SUCCESS |
| 103 | 3 | 500.00 | WALLET_TOPUP | UPI | TOPUP-3-1729772000 | SUCCESS |

#### Executing: `SELECT login_id, username, login_time, ip_address, status FROM login_history ORDER BY login_id DESC LIMIT 3;`
| login_id | username | login_time | ip_address | status |
|:---:|:---|:---|:---|:---:|
| 12 | `admin` | 2026-10-24 13:50:22 | 127.0.0.1 | SUCCESS |
| 11 | `driver1` | 2026-10-24 13:42:10 | 127.0.0.1 | SUCCESS |
| 10 | `unknown_user` | 2026-10-24 13:40:05 | 127.0.0.1 | FAILED |

---

## 11. Conclusion:

The **EVChargeDB** case study demonstrates the advanced deployment of Relational Database Management Systems (ADBMS) in smart electric vehicle charging infrastructure. By delegating bay allocation integrity, anti-double-booking locks, automated financial ledger synchronizations, and hardware state machines to MySQL triggers and stored procedures, the system achieves enterprise-grade transaction reliability and zero data inconsistency. The schema fulfills modern institutional requirements, offering a scalable, audit-ready database architecture for intelligent urban transit networks.

---

\newpage

# COMPANION CASE STUDY: AIP (ADVANCED INTERNET PROGRAMMING)
### Java Servlets (Jakarta EE 10 / Servlet 6.0), JDBC Connectivity & MVC Architecture

---

## 12. AIP Architecture & Integration:

```mermaid
flowchart TD
    subgraph ClientLayer["Client Layer (Web Browser)"]
        Browser["Responsive HTML5 / Cyberpunk Emerald CSS / JS Visualizer"]
    end

    subgraph PresentationTier["Presentation Tier (JavaServer Pages - JSP 3.1)"]
        JSP["dashboard.jsp | stations.jsp | active-session.jsp | admin-analytics.jsp\nJSTL 3.0 (<c:forEach>, <c:if>, <fmt:formatDate>)"]
    end

    subgraph ControllerTier["Controller Tier (Jakarta Servlets 6.0)"]
        AuthServ["AuthServlet (/login, /register, /logout)"]
        StationServ["StationServlet (/stations, /stations/detail)"]
        SessServ["ChargingSessionServlet (/session/start, /simulate, /stop)"]
        WalletServ["WalletServlet (/wallet, /wallet/topup)"]
        AnalyticsServ["AnalyticsServlet (/admin/analytics)"]
    end

    subgraph DAOTier["Data Access Tier (Pure JDBC)"]
        DBConn["DBConnectionUtil (DriverManager & Properties Loader)"]
        DAOs["UserDAO | StationDAO | SlotDAO | ChargingSessionDAO | AnalyticsDAO\n(PreparedStatement, CallableStatement)"]
    end

    subgraph StorageTier["Persistence Tier (MySQL 8.x/9.x)"]
        RDBMS[("evcharge_db\n(Tables, Triggers, Stored Procedures, Views)")]
    end

    Browser -->|HTTP GET / POST| ControllerTier
    ControllerTier -->|Invokes CRUD / Procedures| DAOTier
    DAOTier -->|JDBC Driver over TCP 3306| DBConn
    DBConn -->|Connection Socket| StorageTier
    StorageTier -->|ResultSet / OUT Params| DAOTier
    DAOTier -->|JavaBean Entities| ControllerTier
    ControllerTier -->|RequestDispatcher.forward()| PresentationTier
    PresentationTier -->|Rendered HTML Response| Browser
```

### 12.1 Key AIP Components Implemented

1. **Model-View-Controller (MVC) Pattern**:
   * **Model**: Encapsulated JavaBeans (`User.java`, `Station.java`, `ChargingSlot.java`, `ChargingSession.java`, `Transaction.java`, `LoginRecord.java`).
   * **View**: Secured JSP files located under `/WEB-INF/views/` rendered with JSTL 3.0 and Expression Language (EL).
   * **Controller**: Modular Jakarta Servlets handling HTTP GET/POST verbs with `@WebServlet` annotations.

2. **Pure JDBC Connectivity**:
   * [`DBConnectionUtil.java`](file:///c:/Users/aksha/OneDrive/Desktop/EVChargeDB/src/main/java/com/evchargedb/dao/DBConnectionUtil.java) handles database connections using `db.properties` with resilient password fallback (`Akshay@2004` / `root`).
   * `PreparedStatement` parameters prevent SQL Injection.
   * `CallableStatement` registers output parameters (`Types.DECIMAL`, `Types.VARCHAR`) to invoke stored procedure `sp_stop_charging_session`.

3. **Session Management & Security**:
   * `HttpSession` tracks logged-in users across requests.
   * Security gates restrict administrative and station-operator screens:
     ```java
     HttpSession session = request.getSession(false);
     if (session == null || session.getAttribute("user") == null) {
         response.sendRedirect(request.getContextPath() + "/login");
         return;
     }
     ```
   * Live sign-in attempts (successful and failed) are audited into `login_history` on every authentication attempt.

4. **Deployment Environment**:
   * Target Server: **Apache Tomcat v11.0 Server** (running on port 8080).
   * Language Level: **Java SE 21**.
   * Web Module Specification: **Dynamic Web Module 6.1 / Servlet 6.0**.
