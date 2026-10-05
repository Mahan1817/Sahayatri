# PhysioHome / PhysioGhar End-to-End Manual QA Report

**Test date:** 2026-10-04

**Website:** https://physiohome.tech/

**Mobile device:** realme RMX2161, Android 12

**Android package:** `com.sahayatri.sahayatri_care_connect`

**Documentation:** Markdown report with screenshots linked from `test-evidence/physiohome/`

---

## 1. Test Summary

I tested the main user flows across the PhysioHome website and Android app using the provided demo accounts.

The testing covered the public website, patient and therapist areas, admin dashboard, and the main patient/therapist flows in the mobile app. I also tested therapist search, booking, payment screens, complaints, patient sessions, notifications, reports, and several validation scenarios.

This was a broad manual QA pass. It was not intended to cover every possible device, browser, API response, payment transaction, or data combination.

### Main issues found

The main issues found during testing were:

* A cash-on-visit booking was shown as **payment received even though no payment had been made**.

* A complaint submitted by one patient was shown in the admin panel under another patient's name.

* Patient session and progress counts did not match between different screens and between web and mobile.

* The therapist's Upcoming list contained appointments from July even though the test was performed in October.

* Platform fee information was different in admin, web checkout, and mobile.

* The mobile app gave a generic address validation message.

* Several mobile features such as file upload, pain tracking, milestones, and wallet linking are not available yet.

* A Nepali translation displayed an internal key instead of the actual translated text.

* Some website links and form validations did not behave as expected.

A QA cash-on-visit booking was created and later canceled. No online payment was made. I also submitted a complaint using a QA-only description to verify the complaint flow. After capturing the evidence, that complaint was deleted from the admin panel.

---

# 2. Test Scope

## 2.1 Website

The following areas were checked:

* Public home page

* Services

* Clinics

* Packages

* Testimonials

* About

* Contact

* FAQ

* Blog

* Privacy

* Terms

* Therapist search

* City filter

* Lowest-price sorting

* Therapist booking

* Date and slot availability

* Address selection

* Fee breakdown

* Payment selection up to the gateway step

* Patient login

* Patient Overview

* Sessions

* Reports

* Rewards

* Notifications

* Complaints

* Profile

* Help

* Settings

* Therapist login

* Therapist Overview

* Therapist report form

* Therapist account dashboard

* Admin login

* Admin Overview

* Therapists

* Patients

* Bookings

* Service areas

* Schedules

* Leave

* Payments

* Refunds

* Complaints

* Verification

* Performance

* Safety incidents

* Notifications

* Analytics

* Admin team

* Activity log

* Appearance

* Settings

## 2.2 Android App

The following areas were checked:

* App installation and launch on a physical Android 12 device

* Patient login

* Patient home

* Therapist discovery

* Sessions

* Notifications

* Account

* Profile

* Saved addresses

* Reports

* Recovery progress

* Payment methods

* Rewards

* FAQ

* Support

* Therapist login

* Therapist home

* Schedule

* Booking requests

* Therapist account

* Patient booking

* Cash-on-visit booking

* Booking cancellation

## 2.3 Not Tested

The following were not performed:

* No eSewa or Khalti transaction

* No real-money payment

* No refund transaction

* No therapist report or medical file upload

* Mobile self-upload was checked only to confirm that the feature is currently unavailable

* Admin mobile testing, since the APK only provides patient and therapist roles

* Destructive account deletion

* Admin settings changes

* Availability changes

* Patient/therapist profile edits

* Unrelated test-data changes

---

# 3. Findings

## FIN-01: Cash-on-visit booking is shown as paid

**Priority:** P1 (High)

**Platform:** Android app / Patient notifications / Admin finance

**Area:** Booking and payment

### Description

I created a booking using **Cash on visit** and confirmed it.

Immediately after the booking, the patient received a notification saying that the Rs 1,700 payment had been received. No cash had actually been collected.

The booking was later canceled, but the payment notification was still present.

The admin dashboard also showed Rs 1,700 in platform earnings after the QA booking, even though no payment had been collected and the appointment was canceled.

### Steps to reproduce

1. Log in to the mobile app using `patient@test.com`.

2. Select a therapist.

3. Select a future date and time.

4. Enter a valid address.

5. Select **Cash on visit**.

6. Confirm the booking.

7. Open Notifications.

8. Check the admin Overview.

9. Cancel the booking.

10. Check the payment notification again.

### Expected result

A cash-on-visit booking should remain unpaid until the therapist confirms that the cash was actually received.

Canceling the booking should not create a payment-received notification or add the amount to platform earnings.

### Actual result

The app immediately showed a payment-received notification for Rs 1,700.

The admin earnings also increased even though no payment was collected.

A cancellation notification was later generated as well.

### Evidence

* `mobile-qa-booking-confirmed.png`

* `mobile-qa-payment-notification.png`

* `web-admin-post-qa-booking.png`

---

## FIN-02: Platform fee is different in admin, web checkout, and mobile

**Priority:** P1 (High)

**Platform:** Website / Admin / Android app

**Area:** Pricing and payment

### Description

The platform fee shown in the admin settings was **15%**.

During web booking, the checkout showed a **5% fee**. For example, a Rs 1,300 session showed a Rs 65 platform fee and a total of Rs 1,365.

On mobile, the booking showed the session amount as Rs 1,700 with **"No hidden charges"** and did not show a platform fee.

So the same fee is represented differently in the three areas.

### Steps to reproduce

1. Log in as admin.

2. Open Settings.

3. Check the Platform fee value.

4. Start a patient booking on the website.

5. Continue to the Payment step.

6. Check the fee and total.

7. Repeat the booking flow from the mobile app.

8. Compare the amounts.

### Expected result

The configured platform fee and final amount should be consistent across admin, web, and mobile.

### Actual result

* Admin: 15%

* Web checkout: 5%

* Mobile: No additional platform fee shown

### Evidence

* `mobile-patient-booking-summary.png`

* `web-booking-payment-step.png`

The admin setting showed 15% during testing and was not changed.

---

## SEC-01: Complaint is stored under the wrong patient

**Priority:** P1 (High)

**Platform:** Website

**Area:** Complaints / Patient identity

### Description

I submitted a complaint while logged in as `patient@test.com`.

The patient dashboard showed the account as **John Doe**.

When the complaint was checked from the admin panel, the complaint appeared as being filed by **Nabin Khadka**.

The complaint text was a unique QA-only message that had just been submitted from the John Doe account, so it was possible to confirm that this was the newly created complaint.

### Steps to reproduce

1. Log in to the website using `patient@test.com`.

2. Open Complaints.

3. Select Dr. Anil Shakya.

4. Select the category **Other**.

5. Enter a unique test description.

6. Submit the complaint.

7. Log in as admin.

8. Open **Complaints > Filed by Patients**.

9. Open the newly created complaint.

10. Compare the patient shown in the complaint with the patient account used to submit it.

### Expected result

The complaint should be linked to the patient who submitted it.

### Actual result

The complaint submitted from the John Doe account was displayed under **Nabin Khadka**.

This creates an incorrect patient identity in the admin complaint record.

Whether the other patient could access this complaint was not tested.

### Cleanup

The QA complaint was deleted from the admin panel after the evidence was captured.

### Evidence

* `web-admin-complaints-qa.png`

* `web-admin-qa-complaint-detail.png`

---

## DATA-01: Patient session and progress counts do not match

**Priority:** P1 (High)

**Platform:** Android app / Website

**Area:** Sessions / Recovery progress

### Description

The patient information shown in different parts of the app did not match.

The mobile Account screen showed **20 completed sessions**.

The mobile Sessions screen showed:

* Past: 20

* Some rows in the Past section were still marked **CONFIRMED**

The Recovery Progress screen showed:

* 10/36 sessions

* 9 upcoming

However, the mobile Home and Sessions screens did not show any upcoming bookings.

On the website, the patient Overview showed:

* 37 total

* 10 completed

The Completed list also contained 10 records.

Even after considering the additional QA booking that was created and canceled during testing, the numbers did not line up.

### Steps to reproduce

1. Log in as the same patient on mobile and web.

2. Open Account.

3. Open Sessions.

4. Open Recovery Progress.

5. Check the patient Overview on web.

6. Compare completed and upcoming session counts.

7. Check the status shown on past/canceled sessions.

### Expected result

Session totals and statuses should be based on the same booking data and should match across the different screens and platforms.

### Actual result

Different screens showed different numbers, including:

* 0 upcoming vs 9 upcoming

* 20 completed vs 10 completed

Some past records were also still marked **CONFIRMED**.

One canceled booking contained the address:

`Kathmandu dhdhdnnd`

### Evidence

* `mobile-patient-account.png`

* `mobile-patient-cancelled.png`

* `mobile-patient-progress.png`

* `web-patient-overview-current.png`

---

## DATA-02: Past appointments appear under Therapist Upcoming

**Priority:** P1 (High)

**Platform:** Android app / Therapist

**Area:** Schedule / Requests

### Description

The test was performed on October 4.

The therapist's **Upcoming (23)** list contained appointments dated **Friday, July 17**.

Some of these appointments were marked CONFIRMED or IN PROGRESS and still had actions such as Mark complete and Reschedule.

These appointments were already in the past.

### Steps to reproduce

1. Log in to the mobile app using `therapist@test.com`.

2. Open Requests.

3. Open Upcoming.

4. Check the dates of the listed appointments.

### Expected result

Appointments from the past should not appear under Upcoming.

They should be completed, canceled, or otherwise marked as overdue depending on their state.

### Actual result

July 17 appointments were still shown in Upcoming on October 4.

### Evidence

* `mobile-therapist-upcoming.png`

---

## UX-01: Saved city-only address is rejected with a generic message

**Priority:** P2 (Medium)

**Platform:** Android app / Patient booking

**Area:** Address validation

### Description

The patient's saved primary address was simply:

`Chitwan`

The address could be selected during booking and the flow continued to the booking summary.

When the booking was submitted, the app displayed:

**"Add a little more detail — at least 10 characters."**

The message does not explain what address information is missing.

### Steps to reproduce

1. Log in as patient.

2. Open Saved addresses.

3. Confirm the saved primary address.

4. Select a therapist.

5. Select a date and time.

6. Continue with the saved address.

7. Try to continue from the booking summary.

### Expected result

The app should explain what information is required and show the validation message near the address field.

There should also be a clear way to edit the address.

### Actual result

A generic minimum-length error was displayed after the saved address was already accepted earlier in the flow.

### Evidence

* `mobile-patient-addresses.png`

* `mobile-patient-booking-validation.png`

---

## UX-02: "Tomorrow" is broken into two lines

**Priority:** P2 (Medium)

**Platform:** Android app / Patient booking

**Area:** Booking UI

### Description

On the mobile booking summary, the date tile was too narrow for the word **Tomorrow**.

It appeared as:

`Tomorro`

`w`

This makes the date information look broken.

### Expected result

The date should remain readable on one line or use a shorter format such as:

`Mon, Oct 5`

### Evidence

* `mobile-patient-booking-summary.png`

---

## APP-01: Several patient features are not available yet

**Priority:** P2 (Medium; product scope needs confirmation)

**Platform:** Android app / Patient account

### Description

Several options are visible in the patient app but are not currently available.

Recovery Progress shows that milestones and pain tracking are coming soon.

Payment Methods says saved wallets are coming soon and that eSewa/Khalti are not connected for in-app payment.

Reports & Files contains an **Upload my own report** option, but selecting it displays:

**"Uploading your own files is not available yet."**

### Expected result

Features that are not ready should either be clearly marked as unavailable before entering the flow or should not be presented as active features.

### Actual result

The user can access these options, but they lead to "coming soon" or unavailable messages.

### Evidence

* `mobile-patient-reports.png`

* `mobile-patient-upload-unavailable.png`

* `mobile-patient-progress.png`

* `mobile-patient-payment-methods.png`

---

## WEB-01: Internal translation key is displayed in Nepali

**Priority:** P2 (Medium)

**Platform:** Website / Patient

**Area:** Localization

### Description

When the website was switched to Nepali, the suggested complaint priority displayed:

`ADMIN_DASHBOARD.NORMAL`

instead of a translated value.

Some other parts of the page also remained in English.

### Steps to reproduce

1. Log in as a patient.

2. Change the website language to Nepali.

3. Open Complaints.

4. Check the suggested priority.

### Expected result

The priority and other visible text should be translated according to the selected language.

### Actual result

An internal translation key was visible to the user.

### Evidence

* `untranslated-key-closeup.png`

* `complaint-localization-context.png`

---

## WEB-02: Therapist image sometimes returns 404

**Priority:** P2 (Medium)

**Platform:** Public website

**Area:** Therapist discovery

### Description

The image for Dr. Sushmita Rai returned a **404** for one responsive image request at the 640px width during an earlier pass.

The image loaded successfully through a different responsive request during another pass.

This looks like the issue may depend on the requested image size or image variant.

### Expected result

All responsive image versions should load correctly.

If an image cannot be loaded, a fallback image should be displayed.

### Evidence

* `broken-therapist-image.png`

---

## WEB-03: App Store buttons do not open the app stores

**Priority:** P2 (Medium)

**Platform:** Public website

**Area:** Services footer

### Description

The Google Play and App Store buttons were tested from the public Services page.

Both buttons point to `#` instead of an actual store listing.

### Expected result

The buttons should open the relevant Google Play or App Store listing.

If the apps are not published yet, the buttons should not appear as active links.

### Actual result

Clicking either button only moved to the current page anchor.

### Evidence

* `web-public-services.png`

---

## WEB-04: Therapist report form gives no validation message

**Priority:** P2 (Medium)

**Platform:** Website / Therapist

**Area:** Report upload

### Description

The therapist report form was submitted without selecting a patient, adding a note, selecting a report type, or attaching a file.

After clicking **Upload & notify patient**, nothing visibly changed.

There was no message telling the therapist which fields were required.

### Expected result

The form should show validation messages for required fields and ideally focus the first missing field.

### Actual result

No visible validation message appeared and no report was uploaded.

### Evidence

* `web-therapist-empty-report-form.png`

---

# 4. Areas That Worked

The following flows worked during testing:

* Patient, therapist, and admin demo logins worked on the website.

* Patient and therapist logins worked on the Android app.

* Therapist city filtering worked.

* Lowest-price sorting worked.

* Searching for Dr. Jane Smith eventually returned one matching therapist.

* The website booking modal opened correctly.

* Date navigation worked.

* Available slots were displayed for October 10 for the tested therapist.

* The booking flow reached the payment screen.

* Web checkout showed a 5% platform fee.

* Pay & confirm remained disabled until the required Nepal payment information was selected.

* An invalid short payment number kept the payment action disabled.

* Mobile therapist schedule and Requests screens loaded.

* Therapist account screen loaded.

* Patient support displayed email, phone, service hours, and a send-message option.

* Mobile patient Sessions allowed the test booking to be canceled.

* After cancellation, Upcoming returned to zero.

* Admin Bookings loaded 69 records.

* The main admin management pages loaded successfully.

* The mobile APK archive was readable and contained Flutter assets/native libraries for arm64-v8a, armeabi-v7a, and x86_64.

* APK signature verification was not performed.

---

# 5. Mobile UI/UX Recommendations

## Typography

Use a font that works well for both English and Nepali, such as Noto Sans and Noto Sans Devanagari.

Body text should remain readable at normal Android font sizes and should continue to work when system font scaling is increased.

Avoid fixed-width date tiles that can split words.

## Colors and status

Payment and booking states should not rely only on color.

For example, **Cash due** and **Payment received** should be clearly different both visually and through text.

The current UI recommendations are:

* Primary action: deep teal `#0B5D57`

* Secondary emphasis: warm coral `#D96C4A`

* Surface: `#F4F7F6`

* Text: `#24343A`

## Layout and navigation

Patient and therapist areas should remain clearly separated.

The patient home screen should give priority to:

* Next session

* Book a session

* Care plan

* Today's exercises

The therapist home screen should prioritize:

* Today's schedule

* Booking requests

Keep one main action per screen where possible and use touch targets of at least 44x44pt.

## Booking and care

A simple booking flow would be:

1. Select therapist

2. Select date/time

3. Select visit type

4. Enter/confirm address

5. Review price

6. Confirm booking

The address should be validated before reaching the final confirmation screen.

Fees should be calculated consistently across web and mobile.

For cash-on-visit, the booking should remain unpaid until the cash has actually been collected.

Patient, therapist, and admin booking statuses should also stay synchronized.

## Health information and trust

Patient reports and other health information should have clear access controls.

When file upload becomes available, the app should provide supported file types and size limits.

Recovery milestones and pain tracking should be clearly marked as unavailable if they are still under development.

Complaint records should always remain linked to the correct patient.

---

# 6. Test Data and Cleanup

The following test data was created during this QA pass:

### Mobile booking

One QA booking was created for:

* Therapist: Dr. Anil Shakya

* Date: October 5

* Time: 09:00

* Payment: Cash on visit

* Address: QA-only address

The booking was later canceled through the patient app.

No eSewa or Khalti payment was made.

The cancellation remains in the booking history as an audit record.

### Complaint

One complaint was submitted using unique QA-only text.

The complaint was submitted while logged in as John Doe but appeared in the admin panel under Nabin Khadka.

The evidence was captured and the QA complaint was then deleted.

No other complaints were changed.

### Other data

No report, medical file, unrelated booking, profile, schedule, fee setting, or account was modified.

---

# 7. Evidence Index

* `mobile-patient-home.png`

* `mobile-patient-discovery.png`

* `mobile-patient-booking-summary.png`

* `mobile-patient-booking-validation.png`

* `mobile-qa-booking-confirmed.png`

* `mobile-qa-booking-upcoming.png`

* `mobile-qa-payment-notification.png`

* `mobile-patient-notifications.png`

* `mobile-patient-account.png`

* `mobile-patient-addresses.png`

* `mobile-patient-reports.png`

* `mobile-patient-upload-unavailable.png`

* `mobile-patient-progress.png`

* `mobile-patient-payment-methods.png`

* `mobile-patient-rewards.png`

* `mobile-patient-support.png`

* `mobile-patient-faq.png`

* `mobile-therapist-home.png`

* `mobile-therapist-schedule.png`

* `mobile-therapist-upcoming.png`

* `mobile-therapist-cancelled.png`

* `mobile-therapist-account.png`

* `web-patient-overview-current.png`

* `web-therapist-overview-current.png`

* `web-admin-overview-current.png`

* `web-admin-post-qa-booking.png`

* `web-admin-bookings-empty.png`

* `web-admin-schedule-selected.png`

* `web-admin-complaints-qa.png`

* `web-admin-qa-complaint-detail.png`

* `web-booking-payment-step.png`

* `web-public-services.png`

* `web-therapist-empty-report-form.png`

* `untranslated-key-closeup.png`

* `complaint-localization-context.png`

* `broken-therapist-image.png`