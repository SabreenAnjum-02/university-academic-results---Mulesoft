# 🎓 University Academic Results API & Automated Broadcasting System

[![MuleSoft](https://img.shields.io/badge/MuleSoft-4.4%2B-blue.svg)](https://www.mulesoft.com/)
[![CloudHub 2.0](https://img.shields.io/badge/Deployment-CloudHub%202.0-00A1E0.svg)](https://universityacademicresults-uj2n56.5sc6y6-2.usa-e2.cloudhub.io/portal)
[![MUnit](https://img.shields.io/badge/Tests-MUnit%20100%25%20Passed-brightgreen.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An enterprise-grade, event-driven academic grading and broadcast integration built on **MuleSoft Anypoint Platform**, **MySQL Database**, **Java VM Queue**, **Gmail SMTP**, and deployed to **CloudHub 2.0 (Shared Space, US East 2)**.

---

## 🌐 Live CloudHub 2.0 Web Portal

Access the live interactive portal deployed on MuleSoft CloudHub:
👉 **[https://universityacademicresults-uj2n56.5sc6y6-2.usa-e2.cloudhub.io/portal](https://universityacademicresults-uj2n56.5sc6y6-2.usa-e2.cloudhub.io/portal)**

---

## 🏛️ System Architecture

```
                       +-----------------------------------+
                       |    Interactive Web Portal (UI)    |
                       |    (/portal served by MuleSoft)   |
                       +-----------------+-----------------+
                                         |
                                         | POST /api/students/{id}/grade
                                         v
+---------------------------------------------------------------------------------+
|                               MuleSoft ESB Runtime                              |
|                                                                                 |
|  1. APIkit Router & Autodiscovery (ID: 21182871)                                |
|  2. Try Scope with Error Propagate (DB:CONNECTIVITY)                            |
|  3. MySQL Insert (`broadcast_status = 'PENDING'`)                               |
|                                                                                 |
|  4. Fixed-Frequency Scheduler (30s Poller)                                      |
|     `SELECT ... FROM results r JOIN students s WHERE broadcast_status='PENDING'`|
|                                                                                 |
|  5. VM Queue (`pendingResultsQueue`) — Asynchronous Event Decoupling           |
|                                                                                 |
|  6. VM Listener Broadcast Flow:                                                 |
|     ├── Email Connector: Personalized HTML Grade Card to student's inbox        |
|     ├── Email Attachment: Generated CSV Grade Statement document                |
|     ├── File Connector: Local/Cloud audit CSV log appended                      |
|     └── DB Subflow: Status updated to 'SENT'                                    |
+---------------------------------------------------------------------------------+
```

---

## ✨ Key Capabilities & Highlights

1. **Enterprise API-Led Architecture**:
   - Designed using **RAML 1.0** and published to **Anypoint Exchange**.
   - Governed via **API Manager** with Autodiscovery (`apiId: 21182871`) and Rate Limiting policy.

2. **Event-Driven Asynchronous Processing**:
   - Uses **VM Queues** (`pendingResultsQueue`) to buffer and decouple result processing from user transactions, ensuring high throughput and resilience.

3. **Dynamic Multi-Channel Broadcasting**:
   - Queries student records from MySQL (`students` table joined with `results` table).
   - Dynamically resolves student emails and dispatches customized, rich **HTML Grade Statements** with green/red status indicators.
   - Automatically attaches individual downloadable CSV grade reports.

4. **Audit Logging & Fault Tolerance**:
   - File connector logs every broadcasted result with timestamped audit CSVs.
   - Robust error handling using **Try Scope** and `On Error Propagate` targeting `DB:CONNECTIVITY`.

5. **Automated Testing**:
   - Includes **MUnit 2.x** test suite (`university-academic-results-test-suite.xml`) validating error propagation and business logic with 100% pass rate.

---

## 🚀 API Endpoints

| Method | Path | Description |
| :--- | :--- | :--- |
| `GET` | `/portal` | Serves the interactive University Results Web UI |
| `POST` | `/api/students/{studentId}/grade` | Submits/updates a student's grade and initiates pipeline |
| `GET` | `/console/*` | Interactive APIkit Documentation Console |

### Sample Grade Submission Payload
```json
POST /api/students/101/grade
Content-Type: application/json

{
  "subject": "Distributed Cloud Systems",
  "marks": 92,
  "grade": "O"
}
```

### Sample Response
```json
{
  "message": "Grade updated successfully",
  "studentId": 101
}
```

---

## 🛠️ Local Development & Setup

### Prerequisites
- **Anypoint Studio 7.x** (Mule Runtime 4.4+)
- **MySQL 8.0+**
- **Java JDK 11 or 17**

### Configuration
1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/university-academic-results.git
   ```
2. Navigate to `src/main/resources/`:
   - Copy `config-template.properties` to `config.properties`:
     ```bash
     cp config-template.properties config.properties
     ```
   - Open `config.properties` and add your database credentials and Gmail App Password:
     ```properties
     db.host=localhost
     db.port=3306
     db.user=root
     db.password=YOUR_DB_PASSWORD
     db.database=university_results

     email.host=smtp.gmail.com
     email.port=587
     email.user=YOUR_EMAIL@gmail.com
     email.password=YOUR_GMAIL_APP_PASSWORD
     email.from=YOUR_EMAIL@gmail.com
     ```
   *(Note: `config.properties` is protected by `.gitignore` and will never be committed to Git).*

3. Database Setup:
   - Run the initial SQL script in MySQL:
     ```sql
     CREATE DATABASE university_results;
     USE university_results;

     CREATE TABLE students (
         student_id INT PRIMARY KEY,
         name VARCHAR(100) NOT NULL,
         email VARCHAR(150) NOT NULL,
         department VARCHAR(100),
         year INT
     );

     CREATE TABLE results (
         result_id INT PRIMARY KEY AUTO_INCREMENT,
         student_id INT NOT NULL,
         subject VARCHAR(100) NOT NULL,
         marks INT NOT NULL,
         grade VARCHAR(5),
         status VARCHAR(10),
         broadcast_status VARCHAR(20) DEFAULT 'PENDING',
         FOREIGN KEY (student_id) REFERENCES students(student_id)
     );
     ```

4. Run in Studio:
   - Import the project into Anypoint Studio.
   - Right-click project $\rightarrow$ **Run As** $\rightarrow$ **Mule Application**.
   - Open your browser at **`http://localhost:8081/portal`**.

---

## 👩‍💻 Author
**Sabreen Anjum**  
*Vel Tech Rangarajan Dr. Sagunthala R&D Institute of Science and Technology*
