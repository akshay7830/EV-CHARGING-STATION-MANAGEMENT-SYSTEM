# EVChargeDB: Electric Vehicle Charging Station Management System

An advanced, end-to-end Database Management and Web Operations platform for Electric Vehicle (EV) Charging Networks, built using **Jakarta Servlets, JSP/JSTL, JDBC, and MySQL 8.x/9.x on Eclipse IDE**.

Designed as a joint comprehensive academic project combining:
1. **ADBMS (Advanced Database Management Systems)**: 3NF/BCNF relational modeling, automated triggers, stored procedures with transactions, analytical database views, composite indexing, and integrity constraints.
2. **AIP (Advanced Internet Programming / Web Technology)**: Model-View-Controller (MVC) architecture, Jakarta Servlets 6.0, JDBC connectivity, `HttpSession` session management, role-based access control, responsive Cyberpunk-Emerald EV UI, and dynamic JSTL rendering.

---

## ⚡ Key Highlights & Subject Integration

### 1. ADBMS (Advanced Database Concepts)
* **Relational Schema (3NF / BCNF)**: 8 normalized tables modeling Users, Stations, Charging Slots (AC/DC Fast/Ultra-Fast), Vehicle Profiles, Reservations, Charging Telemetry Sessions, Transactions, and Station Reviews.
* **Automated Database Triggers**:
  * `trg_after_session_start`: Automatically changes slot status to `OCCUPIED` upon start of an active charging session.
  * `trg_after_session_end`: Automatically releases charging slot to `AVAILABLE` upon session completion.
  * `trg_update_wallet_balance`: Automatically credits or debits user balances upon financial transaction insertion (`WALLET_TOPUP`, `CHARGING_PAYMENT`, `RESERVATION_FEE`).
* **Stored Procedures with Atomic Logic**:
  * `sp_stop_charging_session`: Computes total energy delivered, applies station per-kWh tariff, marks session completed, releases slot, and generates an audit transaction atomically.
  * `sp_topup_wallet`: Atomically credits funds and records an audit reference log.
* **Database Views for Analytics**:
  * `vw_station_overview`: Aggregates active, occupied, and reserved bay counts with average driver ratings.
  * `vw_active_charging_sessions`: Real-time streaming telemetry view.
  * `vw_daily_revenue_analytics`: Grouped financial and energy throughput metrics for executive auditing.
* **ACID Transactions**: Atomic reservation booking and payment debiting via JDBC `Connection.setAutoCommit(false)`, `commit()`, and `rollback()`.
* **Indexing & Performance**: Composite indexes (`idx_city_status`, `idx_station_status`, `idx_user_res`, `idx_user_session`, `idx_user_trans`) for fast queries.

### 2. AIP (Advanced Internet Programming)
* **MVC Pattern**:
  * **Model**: JavaBeans (`User`, `Station`, `ChargingSlot`, `Reservation`, `ChargingSession`, `Transaction`, `Review`, `VehicleProfile`).
  * **View**: JSP templates with JSTL 3.0 tags, custom responsive styling (`style.css`), FontAwesome icons, and interactive JavaScript (`app.js`).
  * **Controller**: Clean Jakarta Servlets with `@WebServlet` annotations (`AuthServlet`, `StationServlet`, `SlotServlet`, `ReservationServlet`, `ChargingSessionServlet`, `WalletServlet`, `AnalyticsServlet`, `DashboardServlet`).
* **JDBC Data Access Objects (DAO)**:
  * `DBConnectionUtil`: Robust connection manager reading `db.properties` with fallback support.
  * `PreparedStatement` & `CallableStatement`: Prepared queries to prevent SQL Injection and CallableStatements to invoke MySQL Stored Procedures.
* **Session Management**: Secure session tracking (`HttpSession`), preventing unauthorized access to driver bookings and administrative dashboards.

---

## 📂 Project Structure

```
EVChargeDB/
├── pom.xml                                 # Maven project configuration (Java 21, Servlet 6.0)
├── .project                                # Eclipse project metadata
├── .classpath                              # Eclipse build classpath (Tomcat 11, JavaSE-21)
├── .settings/                              # Eclipse Dynamic Web Project facets (Servlet 6.1)
├── sql/
│   └── schema.sql                          # Database schema, triggers, views, stored procedures & seed data
├── src/main/java/com/evchargedb/
│   ├── controller/                         # Jakarta Servlets (MVC Controllers)
│   │   ├── AuthServlet.java                # Login, Register, Logout
│   │   ├── DashboardServlet.java           # Role-based dashboard (Driver vs Admin)
│   │   ├── StationServlet.java             # Station catalog, search & review
│   │   ├── SlotServlet.java                # Charger bay management
│   │   ├── ReservationServlet.java         # Slot booking & cancellation
│   │   ├── ChargingSessionServlet.java     # Live telemetry & charging simulation
│   │   ├── WalletServlet.java              # Digital wallet & statement
│   │   └── AnalyticsServlet.java           # Network reports & revenue audit
│   ├── dao/                                # Data Access Objects & JDBC
│   │   ├── DBConnectionUtil.java           # JDBC Connection manager
│   │   ├── UserDAO.java
│   │   ├── StationDAO.java
│   │   ├── SlotDAO.java
│   │   ├── ReservationDAO.java
│   │   ├── ChargingSessionDAO.java
│   │   └── AnalyticsDAO.java
│   ├── model/                              # JavaBeans (Entities)
│   │   ├── User.java
│   │   ├── Station.java
│   │   ├── ChargingSlot.java
│   │   ├── VehicleProfile.java
│   │   ├── Reservation.java
│   │   ├── ChargingSession.java
│   │   ├── Transaction.java
│   │   └── Review.java
│   └── util/
│       ├── PasswordUtil.java               # SHA-256 password hashing
│       └── DateUtil.java                   # Timestamp formatters
├── src/main/resources/
│   └── db.properties                       # MySQL connection settings
└── src/main/webapp/                        # Web Presentation Tier
    ├── WEB-INF/
    │   ├── web.xml                         # Deployment descriptor (Servlet 6.0)
    │   ├── classes/                        # Pre-compiled classes
    │   ├── lib/                            # MySQL Driver (8.4.0) & Jakarta JSTL (3.0.1) JARs
    │   └── views/                          # Secure JSP templates
    │       ├── includes/                   # Header, Navbar, Footer
    │       ├── login.jsp
    │       ├── register.jsp
    │       ├── dashboard.jsp
    │       ├── stations.jsp
    │       ├── station-detail.jsp
    │       ├── book-slot.jsp
    │       ├── start-session.jsp
    │       ├── active-session.jsp
    │       ├── my-reservations.jsp
    │       ├── wallet.jsp
    │       ├── admin-stations.jsp
    │       ├── admin-slots.jsp
    │       ├── admin-reservations.jsp
    │       ├── admin-analytics.jsp
    │       └── session-history.jsp
    ├── css/style.css                       # Modern EV cyberpunk-green styling
    ├── js/app.js                           # Interactive helpers & demo autofills
    └── index.jsp                           # Root entrypoint
```

---

## 🚀 Setup & Execution Guide in Eclipse

### Step 1: Database Setup in MySQL
The database `evcharge_db` has already been created and verified on your MySQL Server (`localhost:3306`).
If you ever need to re-import it:
1. Open MySQL Workbench or MySQL Command Line:
   ```cmd
   mysql -u root -pAkshay@2004 < "C:\Users\aksha\OneDrive\Desktop\EVChargeDB\sql\schema.sql"
   ```

### Step 2: Database Configuration (`db.properties`)
Located in `src/main/resources/db.properties`:
```properties
db.driver=com.mysql.cj.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/evcharge_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
db.user=root
db.password=Akshay@2004
```

### Step 3: Open in Eclipse IDE
The project is already configured with Eclipse project metadata and placed in two convenient locations:
1. Desktop: `C:\Users\aksha\OneDrive\Desktop\EVChargeDB`
2. Eclipse Workspace: `C:\Users\aksha\eclipse-workspace\EVChargeDB`

**To open in Eclipse**:
1. Open **Eclipse IDE**.
2. If your workspace is `C:\Users\aksha\eclipse-workspace`:
   - Go to **File** &rarr; **Import...** &rarr; **General** &rarr; **Existing Projects into Workspace**.
   - Choose `C:\Users\aksha\eclipse-workspace\EVChargeDB` (or `C:\Users\aksha\OneDrive\Desktop\EVChargeDB`).
   - Click **Finish**.
3. Eclipse will automatically recognize the project facets:
   - Dynamic Web Module 6.1
   - Java 21
   - Apache Tomcat v11.0 runtime

### Step 4: Run on Server
1. In Eclipse **Project Explorer**, right-click on `EVChargeDB`.
2. Select **Run As** &rarr; **Run on Server**.
3. Select **Apache Tomcat v11.0 Server** and click **Finish**.
4. Open your browser and navigate to:
   ```
   http://localhost:8080/EVChargeDB/
   ```

*(Alternatively, the ready-to-deploy `EVChargeDB.war` is also present inside `EVChargeDB/` and can be dropped directly into `C:\apache-tomcat-11.0.24-windows-x64\apache-tomcat-11.0.24\webapps` if desired).*

---

## 🔑 Pre-Configured Demo Accounts

For fast evaluation and demonstration, the login screen includes **one-click autofill buttons**:

| Role | Username | Password | Purpose |
|---|---|---|---|
| **Admin** | `admin` | `admin123` | Full access: Station CRUD, Slot management, Analytics reports, User roster |
| **Station Operator** | `operator1` | `operator123` | Station operations, bay maintenance, live telemetry monitor |
| **EV Driver** | `driver1` | `driver123` | Search stations, reserve bays, simulate live charging, digital wallet |
| **EV Driver 2** | `driver2` | `driver123` | Secondary test driver |

*New drivers can also be registered at any time from the `/register` page with an automatic **₹500 welcome credit**.*

---

## 💡 Key Application Workflows to Demonstrate

1. **Find & Filter Stations**:
   - Navigate to `/stations`.
   - Filter by city (Gurugram, Bengaluru, Mumbai, Hyderabad) or search keyword.
   - Observe live slot availability badges calculated dynamically from the database.
2. **Reserve a Charging Bay**:
   - Click on any station &rarr; Choose an available bay &rarr; Click **Reserve**.
   - Pick a date, arrival time, and duration.
   - Confirm booking (₹50 advance fee debited atomically from wallet with rollback on error).
3. **Plug-In & Live Charging Simulation**:
   - Go to `/session/start` or your reservations &rarr; Click **Charge Now**.
   - Set battery initial SOC (e.g., 20%) and target limit (80%).
   - Experience the **Live Telemetry Stream**: Click **"Simulate +10% Charge"** to see battery percentage increase, energy deliver in real-time, and cost calculate dynamically!
   - Click **"Stop & Pay Bill"** &rarr; Invokes stored procedure `sp_stop_charging_session`, releases bay via trigger `trg_after_session_end`, and creates a financial transaction.
4. **Digital Wallet & Audit Trail**:
   - Go to `/wallet`.
   - View updated balance and instant top-ups (₹200, ₹500, ₹1000) using stored procedure `sp_topup_wallet`.
   - Review complete financial audit log with unique transaction references.
5. **Admin Master Console & ADBMS Reports**:
   - Sign in as `admin`.
   - Add/edit charging stations or toggle charging bays between `AVAILABLE`, `OCCUPIED`, and `MAINTENANCE`.
   - Visit `/admin/analytics` to view daily revenue and energy throughput reports generated directly from database views `vw_daily_revenue_analytics` and `vw_station_overview`.
