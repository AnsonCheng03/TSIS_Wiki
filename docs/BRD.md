# BRD (Business Requirement Document) of TSIS

## Owner Story

1. As an **Owner**, I want to **create tutor center admin accounts**, so that each center can manage their own tutoring system independently.
   - Owner *Anson* logs into the portal via Keycloak SSO.
   - *Anson* navigates to the **Admin Management** section and clicks "Create Admin".
   - *Anson* selects or creates a tutor center (e.g., "Dsessential Company Limited").
   - *Anson* fills in the admin details: name, email, and default role.
   - The system registers the admin under that center in the **user-profile-service**, and links the center ID to their account.
   - An automatic email is sent via **notification-service** with a temporary password.
   - The admin (e.g., *Allan*) receives the email, logs in, and is prompted to change their password.
   - From now on, *Allan* can manage users and classes within "Dsessential Company Limited" only.

2. As an **Owner**, I want to **create tutor centers**, so that I can group users under their respective organizations.
   - *Anson* navigates to the **Tutor Centers** section and clicks "Create Center".
   - He fills in details such as center name, address, timezone, and billing cycle.
   - Upon submission, the center is created in the **user-profile-service** and pricing metadata is initialized in the **billing-service**.
   - This center now becomes selectable when creating new admin accounts or assigning data scopes.

3. As an **Owner**, I want to **set platform subscription fee and AI credit balance for each center**, so that usage-based billing can be automated.
   - *Anson* selects a center (e.g., "Dsessential Company Limited") from the billing dashboard.
   - He sets a **base monthly subscription fee**, which will be charged regardless of student count.
   - He also sets the **monthly AI credit quota** (e.g., 1000 credits), and the **overage rate** (e.g., $0.01 per extra credit).
   - These configurations are stored in the **billing-service** and reflected in monthly invoices.
   - AI-related services (e.g., essay scoring, feedback generation, transcription) deduct credits via the **homework-service**.
   - When credits run low, the **notification-service** alerts the Admin.
   - Admins can later **purchase add-on credits** if needed (future scope).

4. As an **Owner**, I want to **check billing status of each tutor center**, so that I can track unpaid accounts.
   - *Anson* enters the **Billing Overview** page.
   - The system fetches all active invoices per center from the **billing-service**.
   - Status is shown as “Paid”, “Pending”, or “Overdue”.
   - *Anson* can sort by payment status or due date to follow up.

5. As an **Owner**, I want to **view payment history by tutor center**, so that I can analyze financial trends.
   - *Anson* clicks into a center profile and selects “Payment History”.
   - The system lists all past invoices with amount, date, status, and payment method.
   - Graphs and trends are visualized using data from the **report-service**.

6. As an **Owner**, I want to **generate system usage and financial reports**, so that I can monitor business health.
   - *Anson* selects a time range (e.g., Jan to May 2025) and report type.
   - The **report-service** aggregates data from MongoDB (read model) and renders:
     - Active users per center
     - Monthly revenue
     - Class activity count
     - AI usage breakdown
   - Reports can be exported as CSV or PDF.

7. As an **Owner**, I want to **view access and audit logs**, so that I can ensure data integrity and trace user actions.
   - *Anson* visits the **Audit Log** section.
   - The frontend queries the **log-service** for all logs scoped by action type, user role, or service.
   - Examples include “Admin created”, “Homework deleted”, or “Login failed”.
   - Each log entry includes timestamp, IP, user ID, and action.

## Admin Story

1. As an **Admin**, I want to **create student and tutor accounts**, so that I can manage user access within the center.
   - Admin *Allan* logs into the Admin Dashboard for "Dsessential Company Limited".
   - He goes to **User Management > Create Account** and selects either "Student" or "Tutor".
   - He fills in personal info like name, email, phone number, and class assignment (optional).
   - The system creates the user profile in the **user-profile-service** and sends login credentials via **notification-service**.
   - Once logged in, users are scoped to their center and role-based permissions.

2. As an **Admin**, I want to **bulk import student/tutor data via Excel or sync from Google Drive**, so that onboarding is efficient.
   - *Allan* clicks "Import Users" and chooses the upload method.
   - He uploads an Excel sheet or links a Google Sheet containing user info.
   - The system parses and validates data, then displays import preview and errors.
   - Valid entries are saved to the **user-profile-service**, and credentials are batch emailed.

3. As an **Admin**, I want to **assign students and tutors to classes**, so that I can organize teaching schedules.
   - *Allan* navigates to **Class Management > Assignments**.
   - He selects a class and uses a drag-and-drop or search interface to assign tutors and students.
   - The **class-service** updates relational mappings, enabling scheduling, attendance, and homework functionality.

4. As an **Admin**, I want to **create and manage classes**, so that I can offer structured sessions.
   - *Allan* goes to **Class Management > Create Class**.
   - He inputs class name, subject, schedule (time, recurrence), and duration.
   - The system stores class metadata in the **class-service**.
   - Future edits (e.g., time change, tutor replacement) are version-tracked and synced with attendance and notification systems.

5. As an **Admin**, I want to **check and pay platform usage bills**, so that I can maintain access for our center.
   - *Allan* navigates to the **Billing** page.
   - He sees the center’s monthly subscription fee, AI credit usage (with overage if any), and payment history.
   - He clicks "Pay Now" to settle the invoice via credit card or e-wallet.
   - Payment is processed through a third-party gateway, and status updates in the **billing-service**.

6. As an **Admin**, I want to **check, amend, or void student bills**, so that I can ensure accurate financial records.
   - *Allan* opens **Student Billing** and selects a student (e.g., *Richard*).
   - He sees upcoming and past tuition charges generated based on the per-student pricing model.
   - He can edit line items, apply discounts, or void entries if incorrect.
   - Adjustments are recorded in the **billing-service** with audit trails in **log-service**.

7. As an **Admin**, I want to **configure tuition pricing rules (e.g., per-student fee)** for my own center, so that I can align with our internal policies.
   - *Allan* accesses **Billing Settings > Tuition Rules**.
   - He defines fee structures: flat rate, per-class charge, or per-subject plan.
   - Optionally, he sets scholarship policies, discounts, and payment deadlines.
   - These settings are scoped to his center and stored in the **billing-service**.
   - The system uses this to auto-generate student bills every cycle.

8. As an **Admin**, I want to **integrate SSO for users and tutors**, so that login is simplified and secure.
   - *Allan* opens **System Settings > Authentication**.
   - He enables SSO and selects an identity provider (e.g., Google Workspace).
   - He inputs client ID, secret, and domain restrictions.
   - The configuration is stored in **user-profile-service**, and the system switches to SSO flow for login.
   - Users from the allowed domain can now log in directly without a password, reducing friction.

9. As an **Admin**, I want to **configure system-wide settings (e.g., timezone, language, grading rules)**, so that the system can fit our regional or academic policies.
   - *Allan* visits **Center Settings**.
   - He selects options such as:
     - Default timezone (e.g., UTC+8)
     - Default language (e.g., Traditional Chinese)
     - Grading scheme: percentage-based, letter grade, or pass/fail
   - These settings are saved to **user-profile-service** and applied across scheduling, reports, and UI labels.

10. As an **Admin**, I want to **view audit logs within my center**, so that I can track internal user actions for accountability.

- *Allan* goes to **Logs > Activity History**.
- He filters by action (e.g., “homework edited”, “bill voided”), user, or date range.
- The frontend fetches results from **log-service**, scoped only to his center.
- Log entries include timestamp, actor, IP, and action performed.

11. As an **Admin**, I want to **send notifications to students and tutors**, so that I can communicate important updates or reminders.

- *Allan* opens **Communications > Send Notification**.
- He selects a target group (e.g., students of Class A, or all tutors).
- He writes a message and selects a delivery method: email, in-app, or push.
- The message is queued and sent via **notification-service**.
- Delivery status (sent, failed) is tracked in real-time.

12. As an **Admin**, I want to **reschedule or cancel a class**, so that I can respond to emergencies.

- *Allan* visits **Class Schedule** and selects a class occurrence.
- He edits the date/time or marks the session as canceled.
- Changes are written to **class-service** and cascade to **attendance-service**.
- Students and tutors are automatically notified via **notification-service**.

13. As an **Admin**, I want to **set school holidays**, so that regular classes can be adjusted accordingly.

- *Allan* opens **Academic Calendar > Holidays**.
- He adds dates or date ranges for public holidays or breaks.
- These dates are stored in **class-service**, which uses them to skip sessions when auto-generating class schedules.
- Tutors and students will see holiday notices on their dashboards.

14. As an **Admin**, I want to **compare performance across classes**, so that I can improve our teaching methods.

- *Allan* visits **Analytics > Class Comparison**.
- The dashboard shows average attendance, homework completion rates, and AI score averages by class.
- Data is aggregated from **attendance-service**, **homework-service**, and **report-service** (MongoDB read model).
- *Allan* filters by subject or tutor to analyze specific trends.

## Tutor Story

1. As a **Tutor**, I want to **view the list of assigned students**, so that I can prepare for class.
   - Tutor *Crystal* logs into her dashboard.
   - She selects a class (e.g., English Writing – Sat 3PM).
   - The system queries **class-service** and displays the list of enrolled students with name, photo, and recent attendance rate.
   - *Crystal* can click a student to view academic progress and past homework.

2. As a **Tutor**, I want to **take attendance**, so that I can track student participation.
   - At the start of a session, *Crystal* opens the **Attendance** tab for her class.
   - She marks each student as "Present", "Late", "Absent", or "Excused".
   - Submissions are saved to the **attendance-service**, which also logs timestamp and tutor ID.
   - The system auto-notifies Admin if the absence rate is unusually high.

3. As a **Tutor**, I want to **upload or livestream class videos**, so that students can access lessons remotely.
   - *Crystal* opens **Video Library > Upload** and selects a recorded MP4 file.
   - The video is stored to **S3**, metadata is saved via **class-service**, and linked to the class record.
   - Alternatively, *Crystal* can launch a **Live Stream** with a built-in integration (e.g., OBS, Zoom, or WebRTC).
   - After class, the stream is automatically archived and linked to the session.

4. As a **Tutor**, I want to **upload or write class notes**, so that students can revise material afterward.
   - *Crystal* enters **Class Notes > New Note**.
   - She writes Markdown or rich text, attaches images or PDFs, and selects the relevant class.
   - The note is saved and becomes visible to students immediately in their portal or app.

5. As a **Tutor**, I want to **mark student homework or mock exams**, so that I can assess their academic performance.
   - *Crystal* goes to **Homework Inbox**.
   - She opens each submission, writes comments, and assigns a grade or score.
   - All data is saved to the **homework-service** and reflected in student analytics.

6. As a **Tutor**, I want to **request AI to assist in marking homework and preview the feedback before sending it**, so that I can ensure quality control while saving time.
   - *Crystal* clicks "AI Review" on a submitted essay.
   - The **homework-service** invokes the AI model to generate a score and detailed feedback (e.g., grammar, coherence, task response).
   - *Crystal* reviews and edits the AI feedback if needed, then clicks “Send to Student”.
   - AI usage deducts from the center’s credit balance tracked in **billing-service**.

7. As a **Tutor**, I want to **send notifications to students**, so that I can remind them of class activities or deadlines.
   - *Crystal* opens **Student Messaging > New Notification**.
   - She writes a reminder (e.g., “Bring your essay draft next week!”) and selects the class.
   - Messages are dispatched through **notification-service** via in-app alert and email.

8. As a **Tutor**, I want to **see student performance analytics over time**, so that I can identify who needs more help.
   - *Crystal* opens **Performance Dashboard**.
   - The system visualizes trends such as:
     - Average scores by topic
     - Attendance consistency
     - Homework submission rate
   - Data is queried from **report-service** and scoped only to her assigned classes.
   - She uses this insight to plan personalized interventions.

## Student Story

1. As a **Student**, I want to **submit homework**, so that my tutor can provide feedback.
   - Student *Richard* logs into his portal or opens the mobile app.
   - He navigates to **Homework > My Assignments**, and selects an assignment due this week.
   - *Richard* uploads a PDF, types inline text, or pastes Google Doc links.
   - The file is sent to the **homework-service** and marked as "Submitted".
   - *Richard* sees a timestamp confirmation and can edit or resubmit before the deadline.

2. As a **Student**, I want to **receive tutor or AI-generated comments**, so that I can learn from my mistakes.
   - After *Crystal* marks the submission, *Richard* gets a notification.
   - He opens **Homework > Feedback**, where he sees:
     - Tutor’s comments
     - AI-generated rubric feedback (e.g., grammar, content, structure)
     - Score or rating
   - *Richard* can view change history and improve on his next assignment.

3. As a **Student**, I want to **watch video replays or attend live sessions**, so that I can review and participate in class.
   - *Richard* checks **Classroom > Video Lessons**.
   - He finds last week’s class and clicks “Replay”.
   - The video streams from **S3** with a timestamped playlist and playback speed control.
   - For upcoming live classes, *Richard* can join via a “Join Now” button that connects to the integrated video platform.

4. As a **Student**, I want to **view my attendance record**, so that I can stay accountable.
   - *Richard* clicks into **My Attendance**.
   - A calendar view highlights each session with icons (✔ Present, ✖ Absent, ➖ Excused).
   - Records are fetched from **attendance-service** and scoped to his class enrollments.
   - He can download attendance logs or report discrepancies to Admin.

5. As a **Student**, I want to **check my tuition bills**, so that I am aware of what I owe.
   - *Richard* opens **Billing > My Invoices**.
   - He sees a list of current and past tuition charges, payment status, and due dates.
   - The data comes from **billing-service**, scoped to his student ID.
   - If there are discounts or adjustments, he can see line-by-line breakdown.

6. As a **Student**, I want to **pay my tuition online**, so that I can avoid manual transfers.
   - *Richard* clicks "Pay Now" on a pending invoice.
   - The system redirects to a payment gateway (credit card, PayMe, or FPS).
   - Upon success, the payment status updates automatically, and a receipt is generated.
   - He can view his full payment history anytime from his portal.

7. As a **Student**, I want to **view my academic progress and past results**, so that I can understand how I'm improving.
   - *Richard* visits **My Performance > Analytics**.
   - He sees trends like:
     - Average homework score
     - Attendance consistency
     - Class engagement level (if tracked)
   - Data is retrieved from **report-service** and visualized as charts.
   - *Richard* uses this to track improvement before exams.

8. As a **Student**, I want to **submit a class absence request**, so that my tutor and admin are informed in advance.
   - *Richard* opens **Attendance > Request Leave**.
   - He selects the class date and fills in a reason (e.g., sick leave, family trip).
   - The request is sent to **attendance-service**, and a pending status is shown.
   - Admin and tutor are notified to approve or reject.

9. As a **Student**, I want to **see my upcoming class schedule**, so that I don’t miss any sessions.
   - *Richard* visits **My Schedule**.
   - The dashboard shows weekly sessions by day/time, with tutor and subject.
   - Holiday-adjusted dates are handled automatically from **class-service**.
   - Notifications remind him 15 minutes before each session.

## User Story (Admin, Tutor and Student)

1. As a **User**, I want to **edit my personal information and change my password**, so that I can manage my account securely.
   - *Richard* logs into the portal and clicks on his profile avatar at the top right corner.
   - He selects **Account Settings > Profile**.
   - *Richard* updates his name, contact number, and avatar image.
   - Changes are validated and saved to the **user-profile-service**.
   - To change his password, *Richard* navigates to **Account Settings > Security**.
   - He enters his current password, then the new password twice.
   - The system verifies the change through **Keycloak**, and on success, logs him out of all other sessions.
   - Similarly, *Crystal* (Tutor) or *Allan* (Admin) can perform the same actions in their dashboards.
   - Updates are scoped to each user's role but unified under the same settings interface.
