# PhysioHome / PhysioGhar End-to-End Manual QA Report

**Test date:** 2026-10-04  
**Website:** https://physiohome.tech/  
**Mobile device:** realme RMX2161, Android 12  
**Android package:** `com.sahayatri.sahayatri_care_connect`  
**Documentation:** Markdown report with linked screenshots under `test-evidence/physiohome/`

## Executive Summary

This pass exercised the public website, patient/therapist/admin web workspaces, and patient/therapist Android app workflows using the supplied demo accounts. It covered discovery, booking through payment review, patient account/support/health views, therapist schedule/requests, admin management pages, and selected form validation. It is broad manual coverage, not exhaustive testing of every combination, device size, API response, or payment-provider transaction.

The most serious findings are incorrect financial state after a cash-on-visit booking (the app immediately reports payment received), and a patient complaint being stored under a different patient's identity. Additional issues include inconsistent patient session/progress counts, therapist lists that classify July appointments as upcoming in October, conflicting platform-fee configuration and checkout totals, an unclear mobile address requirement, and unfinished upload/wallet/care-progress features.

A QA cash-on-visit booking was created and canceled. No online payment was authorized. A clearly marked QA complaint was submitted; the admin queue stored it under the wrong patient, the finding was captured, and that QA complaint was then deleted. The canceled booking remains in the demo booking history as a cancellation, which is useful evidence of the lifecycle result.

## Test Scope

### Website

- Public home, services, clinics, packages, testimonials, about, contact, FAQ, blog, privacy, and terms pages.
- Therapist discovery search, city filter, lowest-price sorting, therapist booking modal, date/slot availability, address, fee breakdown, and payment selection up to the gateway step.
- Patient login and Overview, Sessions, Reports, Rewards, Notifications, Complaints, Profile, Help, and Settings routes.
- Therapist login, Overview, report form, and account dashboard.
- Admin login, Overview, therapists, patients, bookings, service areas, schedules, leave, payments, refunds, complaints, verification, performance, safety incidents, notifications, analytics, admin team, activity log, appearance, and settings pages.

### Android App

- Install and launch on a physical Android 12 device.
- Patient login; home, therapist discovery, sessions, notifications, account, profile, saved addresses, reports, recovery progress, payment methods, rewards, FAQ, and support.
- Therapist login; home, schedule, booking requests, and account.
- Patient booking through confirmation with cash-on-visit, then cancellation; no online payment was made.

### Not Performed

- No eSewa/Khalti transaction, refund, or real-money payment was initiated.
- No therapist report or patient medical file was uploaded; mobile self-upload explicitly reports that it is unavailable.
- Admin mobile testing is out of scope because the APK exposes patient and therapist roles only.
- Destructive account deletion, admin settings changes, availability changes, patient/therapist profile edits, and unrelated test-data modifications were not performed.

## Findings

### FIN-01: Cash-on-visit booking is immediately marked as paid

**Priority:** P1 (High)  
**Platform:** Android app / patient notifications and admin finance  
**Area:** Booking, payment state, notifications

**Description:** After selecting **Cash on visit** and confirming a test booking, the app immediately created a “Payment received” notification stating that the Rs 1,700 cash payment was confirmed. No cash had been collected. The booking was subsequently canceled, but the false payment notice remained. After the test flow, the admin dashboard's platform-earnings card also changed from the earlier Rs 0 baseline to Rs 1,700 even though no payment was collected and the appointment was canceled.

**Steps to reproduce:**
1. Sign in to the mobile app as `patient@test.com`.
2. Select a therapist, a future date/time, and enter a valid address.
3. Select **Cash on visit** and confirm the booking.
4. Inspect Notifications and the admin Overview before any visit/payment takes place.
5. Cancel the booking and inspect the payment notification again.

**Expected:** A cash-on-visit booking is recorded as unpaid/pending collection until the therapist records the actual cash receipt. Cancellation should not create payment-received or earned-fee records.

**Actual:** A payment-received notice was generated immediately; admin earnings rose despite no funds being collected. The patient then received a cancellation notice as well.

**Evidence:** [Mobile booking confirmation](test-evidence/physiohome/mobile-qa-booking-confirmed.png) · [False cash-payment notification](test-evidence/physiohome/mobile-qa-payment-notification.png) · [Admin overview after QA booking](test-evidence/physiohome/web-admin-post-qa-booking.png)

### FIN-02: Platform fee differs between admin configuration, web checkout, and mobile

**Priority:** P1 (High)  
**Platform:** Website, admin, Android app  
**Area:** Pricing and payment transparency

**Description:** Admin Settings shows a configured platform fee of **15%**. Web checkout showed **5%** (for example, Rs 65 on a Rs 1,300 session; total Rs 1,365). The mobile checkout showed a Rs 1,700 session as the full amount and stated “No hidden charges,” with no platform fee. This creates three different fee representations.

**Steps to reproduce:**
1. Sign in as admin and open Settings; inspect Platform fee.
2. Start a patient web booking and advance to Payment; inspect the itemized fee.
3. Start the equivalent flow in the mobile app and inspect the session summary.

**Expected:** The configured fee rate and total are consistent across clients and disclosed before a user confirms a booking.

**Actual:** Admin setting says 15%; web checkout calculates 5%; mobile shows no additional fee.

**Evidence:** [Mobile booking summary](test-evidence/physiohome/mobile-patient-booking-summary.png) · [Web payment step](test-evidence/physiohome/web-booking-payment-step.png). Admin Settings showed 15% during this pass; the setting was not changed.

### SEC-01: Submitted complaint is attributed to a different patient

**Priority:** P1 (High)  
**Platform:** Website / patient complaint and admin queue  
**Area:** Identity, privacy, trust and safety

**Description:** A complaint submitted while authenticated as `patient@test.com` (patient dashboard identifies the account as John Doe) appeared in the admin queue as filed by **Nabin Khadka**. The stored complaint body exactly matched the unique QA-only text entered in the John Doe session, confirming that this was the newly submitted record rather than an older fixture. The complaint was deleted after evidence capture.

**Steps to reproduce:**
1. Sign in to the website as `patient@test.com`.
2. Open Complaints, select Dr. Anil Shakya and category Other, enter a unique test description, and submit.
3. Sign in as admin and open Complaints > Filed by Patients.
4. Open the newest complaint and compare the displayed owner with the authenticated patient.

**Expected:** The complaint is stored against the authenticated patient's account and displayed to admins with the correct identity.

**Actual:** The exact QA complaint appeared under Nabin Khadka, while the session was John Doe. This misroutes the complaint and corrupts the admin identity/audit record. Whether the other named patient can access the complaint was not tested.

**Cleanup:** The QA complaint was permanently deleted through the admin interface after capturing its details. Other complaints were not changed.

**Evidence:** [Admin complaint queue](test-evidence/physiohome/web-admin-complaints-qa.png) · [Admin complaint detail](test-evidence/physiohome/web-admin-qa-complaint-detail.png)

### DATA-01: Patient session totals and progress disagree between screens and web/mobile

**Priority:** P1 (High)  
**Platform:** Android app and website / patient  
**Area:** Care history and progress

**Description:** The mobile Account card showed 20 completed sessions; mobile Sessions showed Past (20) and included rows still labeled CONFIRMED. Mobile Recovery Progress displayed 10/36 sessions and 9 upcoming, while mobile home and Sessions showed no upcoming bookings. On the web, the patient Overview showed 37 total and 10 completed after the QA booking was created and canceled; the Completed list had 10 records. Even allowing for the extra canceled QA record, the mobile and web progress/status groupings do not reconcile.

**Steps to reproduce:**
1. Sign in as the same patient in mobile and web.
2. Compare Account, Sessions, Recovery Progress, and web Overview/Completed.
3. Inspect status labels on past/cancelled session rows.

**Expected:** Aggregates and status labels are derived consistently from the same bookings and update predictably across clients.

**Actual:** Conflicting counts, including 0 versus 9 upcoming and 20 versus 10 completed; some past rows remain labeled CONFIRMED. A cancelled address also contained the malformed text `Kathmandu dhdhdnnd`.

**Evidence:** [Patient mobile account](test-evidence/physiohome/mobile-patient-account.png) · [Patient cancelled/past sessions](test-evidence/physiohome/mobile-patient-cancelled.png) · [Patient recovery progress](test-evidence/physiohome/mobile-patient-progress.png) · [Patient web overview](test-evidence/physiohome/web-patient-overview-current.png)

### DATA-02: Therapist Upcoming list contains appointments dated in the past

**Priority:** P1 (High)  
**Platform:** Android app / therapist  
**Area:** Schedule and session management

**Description:** On October 4, the therapist app's **Upcoming (23)** list showed appointments dated Friday, July 17, including CONFIRMED and IN PROGRESS sessions, with actions such as Mark complete and Reschedule. These dates were already in the past.

**Steps to reproduce:**
1. Sign in as `therapist@test.com`.
2. Open Requests > Upcoming.
3. Inspect the first visible appointment dates and states.

**Expected:** Past appointments are classified as completed, canceled, or explicitly overdue, not Upcoming.

**Actual:** July 17 records were still in the upcoming queue on October 4.

**Evidence:** [Therapist upcoming sessions](test-evidence/physiohome/mobile-therapist-upcoming.png)

### UX-01: Mobile booking rejects the saved city-only address with a generic error

**Priority:** P2 (Medium)  
**Platform:** Android app / patient booking

**Description:** The account's primary address is stored as “Chitwan.” Booking continued to a summary, but attempting to submit it displayed “Add a little more detail — at least 10 characters.” The app does not identify which address detail is missing. The address editor/account says the current single address can be changed, while multiple addresses are “coming soon.”

**Steps to reproduce:**
1. Sign in as patient and open Saved addresses to confirm the primary address.
2. Select a therapist, date and time, then continue with the saved address.
3. Attempt to continue from the booking summary.

**Expected:** The field explains required address parts, identifies the error beside the address input, and provides a clear edit path.

**Actual:** Generic minimum-length message appears after carrying forward the saved short address.

**Evidence:** [Saved patient address](test-evidence/physiohome/mobile-patient-addresses.png) · [Booking validation](test-evidence/physiohome/mobile-patient-booking-validation.png)

### UX-02: Mobile booking summary splits “Tomorrow” across lines

**Priority:** P2 (Medium)  
**Platform:** Android app / patient booking

**Description:** The narrow date tile wraps “Tomorrow” as “Tomorro” and a lone “w,” making the appointment date look malformed.

**Expected:** The date label remains intact or uses a compact format such as “Mon, Oct 5.”

**Evidence:** [Mobile booking summary](test-evidence/physiohome/mobile-patient-booking-summary.png)

### APP-01: Pain tracking, milestones, wallet linking, and patient file upload are incomplete

**Priority:** P2 (Medium; product scope confirmation needed)  
**Platform:** Android app / patient account

**Description:** Recovery Progress says milestones and pain tracking are “coming soon.” Payment methods says saved wallets are coming soon and that eSewa/Khalti are not connected for in-app payment. Reports & Files shows an “Upload my own report” action, but tapping it displays “Uploading your own files is not available yet.” These controls are discoverable but incomplete.

**Expected:** Features represented by visible actions are either implemented or clearly presented as unavailable before the user enters the workflow; roadmap placeholders should not look like active capabilities.

**Actual:** The app exposes entry points that resolve to “coming soon”/unavailable notices, limiting self-management and care tracking.

**Evidence:** [Patient reports](test-evidence/physiohome/mobile-patient-reports.png) · [Upload unavailable notice](test-evidence/physiohome/mobile-patient-upload-unavailable.png) · [Recovery progress](test-evidence/physiohome/mobile-patient-progress.png) · [Payment methods](test-evidence/physiohome/mobile-patient-payment-methods.png)

### WEB-01: Nepali complaint form exposes an untranslated internal key

**Priority:** P2 (Medium)  
**Platform:** Website / patient localization

**Description:** In Nepali, suggested complaint priority renders as `ADMIN_DASHBOARD.NORMAL` rather than a localized value. Other navigation and supporting text also remain partly English.

**Steps to reproduce:** Sign in as patient, switch to Nepali, open Complaints, inspect suggested priority.

**Expected:** All form labels, priorities, validation, and navigation use the selected language consistently.

**Evidence:** [Priority key close-up](test-evidence/physiohome/untranslated-key-closeup.png) · [Complaint page context](test-evidence/physiohome/complaint-localization-context.png)

### WEB-02: Therapist image intermittently returns HTTP 404

**Priority:** P2 (Medium)  
**Platform:** Public website / therapist discovery

**Description:** The responsive image request for Dr. Sushmita Rai returned 404 at the 640px optimized width in a prior pass, leaving the therapist card without a photograph. The image loaded through a different responsive request during another pass, so the failure appears variant-dependent or intermittent.

**Expected:** All responsive image variants load, or a deliberate fallback is shown.

**Evidence:** [Failed therapist image](test-evidence/physiohome/broken-therapist-image.png)

### WEB-03: App-store badges link to the current page anchor

**Priority:** P2 (Medium)  
**Platform:** Public website / Services footer

**Description:** Google Play and App Store badges both resolve to `#` on the current page rather than an app-store listing.

**Expected:** Badges open the valid application listing or are hidden until store links are live.

**Actual:** Activating either badge only targets the current page anchor.

**Evidence:** [Public Services page](test-evidence/physiohome/web-public-services.png)

### WEB-04: Empty therapist report submission provides no visible field guidance

**Priority:** P2 (Medium)  
**Platform:** Website / therapist report upload

**Description:** Clicking Upload & notify patient with patient, note, type, and attachment unset leaves the dashboard form unchanged and provides no visible field-level feedback.

**Expected:** Required fields are clearly indicated, with an error message and focus on the first missing field.

**Actual:** No visible feedback appeared; no report was uploaded.

**Evidence:** [Therapist empty report form](test-evidence/physiohome/web-therapist-empty-report-form.png)

## Confirmed Working

- Patient, therapist, and admin demo logins succeeded on the website; patient and therapist logins succeeded in the Android app.
- Public therapist city filtering and lowest-price sort worked. Searching Dr. Jane Smith eventually returned one matching card.
- Website booking modal opened, allowed date advancement, showed available slots on Oct 10 for the tested therapist, and reached the payment screen. The test stopped before payment.
- Web checkout showed a 5% platform fee and disabled Pay & confirm until a Nepal payment method/required eSewa number was selected. An invalid short number left the payment action disabled.
- Mobile therapist schedule and therapist requests/account screens loaded; patient support showed email, phone, service hours, and a send-message action.
- Mobile patient Sessions supported booking cancellation; the QA cash booking was canceled and Upcoming returned to zero.
- Admin bookings page loaded 69 records and admin management pages rendered for the routes listed in Test Scope.
- Mobile APK archive was readable (378 entries) and contained Flutter assets/native libraries for arm64-v8a, armeabi-v7a, and x86_64. No signature verification was performed in this pass.

## Mobile UI/UX Recommendations

### Typography

Use Noto Sans for Latin and Noto Sans Devanagari for Nepali. Keep body copy around 16sp, support system font scaling, and avoid splitting words in fixed date tiles. Give labels readable contrast and reserve monospaced type for short metadata only.

### Color and status

Use deep teal `#0B5D57` as the primary action color and warm coral `#D96C4A` sparingly for secondary emphasis. Use white and cool mist `#F4F7F6` surfaces with charcoal `#24343A` text. Status must be communicated in text plus color; never infer paid/confirmed/canceled from color alone. In particular, make “cash due” visually and semantically distinct from “payment received.”

### Layout and navigation

Keep patient and therapist role workspaces distinct, with stable labeled bottom navigation. The patient home should prioritize next session, Book a session, care plan, and today's exercises. Therapist home should prioritize today's schedule and actionable requests. Use compact cards, predictable page titles, one clear primary action per screen, and touch targets of at least 44x44pt.

### Booking and care

Use a clear sequence: therapist, date/time, visit type, complete address, transparent price breakdown, then confirmation. Keep date text on one line and show time zone. Validate address inline with specific requirements and allow multiple saved addresses. Show fees consistently with the configured rate. Treat cash-on-visit as unpaid until collection is recorded. Add cancellation terms before confirmation and maintain synchronized states between patient, therapist, and admin views.

### Health information and trust

Implement patient-owned report upload with file type/size guidance, clinician notes with safe access controls, recovery milestones, pain tracking, accessible exercise instructions, and reminders. Clearly label roadmap items as unavailable rather than presenting them as active controls. Keep complaint identity association correct and auditable, and show the patient exactly who can view sensitive reports and complaints.

## Test Data and Cleanup

- One mobile QA booking was created for Dr. Anil Shakya for Oct 5 at 09:00, with a QA-only address and cash-on-visit selected; it was canceled through the patient app. No eSewa/Khalti payment occurred. The cancellation remains as an audit/history record.
- One web QA complaint was submitted with unique QA-only text. Admin stored it under Nabin Khadka instead of the authenticated John Doe patient; evidence was captured, then the exact QA complaint was deleted. No other complaint was modified.
- No report, medical file, unrelated booking, profile, schedule, fee setting, or account was changed.

## Evidence Index

- [Patient mobile home](test-evidence/physiohome/mobile-patient-home.png)
- [Patient mobile discovery](test-evidence/physiohome/mobile-patient-discovery.png)
- [Patient booking summary](test-evidence/physiohome/mobile-patient-booking-summary.png)
- [Patient booking validation](test-evidence/physiohome/mobile-patient-booking-validation.png)
- [QA booking confirmation](test-evidence/physiohome/mobile-qa-booking-confirmed.png)
- [QA booking in Upcoming](test-evidence/physiohome/mobile-qa-booking-upcoming.png)
- [False cash-payment notification](test-evidence/physiohome/mobile-qa-payment-notification.png)
- [Patient notifications](test-evidence/physiohome/mobile-patient-notifications.png)
- [Patient account and profile](test-evidence/physiohome/mobile-patient-account.png)
- [Patient saved address](test-evidence/physiohome/mobile-patient-addresses.png)
- [Patient reports](test-evidence/physiohome/mobile-patient-reports.png)
- [Patient upload unavailable notice](test-evidence/physiohome/mobile-patient-upload-unavailable.png)
- [Patient recovery progress](test-evidence/physiohome/mobile-patient-progress.png)
- [Patient payment methods](test-evidence/physiohome/mobile-patient-payment-methods.png)
- [Patient rewards](test-evidence/physiohome/mobile-patient-rewards.png)
- [Patient support](test-evidence/physiohome/mobile-patient-support.png)
- [Patient FAQ](test-evidence/physiohome/mobile-patient-faq.png)
- [Therapist mobile home](test-evidence/physiohome/mobile-therapist-home.png)
- [Therapist mobile schedule](test-evidence/physiohome/mobile-therapist-schedule.png)
- [Therapist mobile upcoming records](test-evidence/physiohome/mobile-therapist-upcoming.png)
- [Therapist mobile cancelled records](test-evidence/physiohome/mobile-therapist-cancelled.png)
- [Therapist mobile account](test-evidence/physiohome/mobile-therapist-account.png)
- [Patient web overview](test-evidence/physiohome/web-patient-overview-current.png)
- [Therapist web overview](test-evidence/physiohome/web-therapist-overview-current.png)
- [Admin web overview](test-evidence/physiohome/web-admin-overview-current.png)
- [Admin dashboard after QA booking](test-evidence/physiohome/web-admin-post-qa-booking.png)
- [Admin bookings (69 records)](test-evidence/physiohome/web-admin-bookings-empty.png)
- [Admin therapist schedule](test-evidence/physiohome/web-admin-schedule-selected.png)
- [Admin complaint queue](test-evidence/physiohome/web-admin-complaints-qa.png)
- [Admin complaint detail](test-evidence/physiohome/web-admin-qa-complaint-detail.png)
- [Web booking payment step](test-evidence/physiohome/web-booking-payment-step.png)
- [Public Services page](test-evidence/physiohome/web-public-services.png)
- [Therapist web empty report form](test-evidence/physiohome/web-therapist-empty-report-form.png)
- [Untranslated localization key](test-evidence/physiohome/untranslated-key-closeup.png)
- [Intermittently broken therapist image](test-evidence/physiohome/broken-therapist-image.png)

## Submission Note

This report and its screenshots are local workspace files. ClickUp and Google Docs are not configured in this environment, so no public share link was created. To share externally, publish this Markdown and the linked evidence folder to an approved public/shared location; screenshots must be uploaded alongside the report so relative links continue to work.
