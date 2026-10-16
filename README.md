# Loan Application System (Core Java + JDBC + MySQL, console only)

Covers all 5 stages of the SIT final project in one application.

| Stage | What it does | Where |
|---|---|---|
| 1 | Register, login (email or username), logout, forgot password, lock account after 3 failed logins | `ui/App`, `service/AuthService` |
| 2 | Predefined admin, search/view/edit/activate/deactivate/delete users, audit log, email notifications, role check | `ui/AdminSession` |
| 3 | User applies for loan; admin views, approves or rejects (with reason), email sent | `ui/UserSession`, `ui/AdminSession` |
| 4 | PAN validation, random CIBIL (600-900), interest rate by CIBIL, auto-generated username/password emailed | `ui/App.cibilCheck`, `service/CibilService`, `service/CredentialGenerator` |
| 5 | EMI, total interest, total payable stored on approval and emailed | `service/EmiCalculator`, `ui/AdminSession.approve` |

## Setup

1. **MySQL**: run the script `db/schema.sql` (creates `loan_db`, tables, and the admin user).
   `mysql -u root -p < db/schema.sql`
2. **Config**: edit `src/main/resources/config.properties`
   (or put a `config.properties` next to the jar to override it):
   - `db.user` / `db.password` for MySQL
   - SMTP settings. For Gmail, turn on 2-Step Verification and create an **App password**
     (Google Account > Security > App passwords), then use it as `mail.password`.
     Without SMTP configured the app still works; it prints a warning instead of sending.
3. **Build & run** (Java 17+, Maven):
   ```
   mvn clean package
   java -jar target/loan-application-system-1.0.jar
   ```
   In Eclipse/IntelliJ: import as a Maven project and run `com.sit.loan.ui.App`.

## Admin login
Username `admin`, password `Admin@123` (stored as a salted hash in the DB).
Admins are not subject to the failed-login lockout.

## Design notes
- Passwords are stored as salted SHA-256 hashes, not plain text. Generated passwords are
  shown to the user only in the email (or on screen if the email cannot be sent).
- Interest rate = linear within the loan type's range: CIBIL 900 gets the lowest rate, 600 the highest.
  e.g. Home Loan at 720 gives 7.70%.
- EMI = P x r x (1+r)^n / ((1+r)^n - 1), with r = annual rate / 12 / 100 and n = months.
- Loan duration is entered in months.
- A user's PAN/CIBIL is requested the first time they apply for a loan, then reused.
- Locked users (status Inactive) can only be re-activated by an admin (which also resets failed attempts).
- Deleting a user also deletes their loans (foreign key cascade) and writes a row to `audit_log`.
