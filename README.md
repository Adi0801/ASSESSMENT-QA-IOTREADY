# ASSESSMENT-QA-IOTREADY

# QA Assessment Submission

## Part 1 — Bug Report

### Internal Bug Report

**Title:** Intermittent mobile app failure reported by customer

**Environment:**

* Platform: Mobile app
* Device/OS/App version: Unknown
* Network conditions: Unknown

**Customer Report:**
“The app doesn't work on my phone sometimes.”

**Current Status:**
Unable to reproduce internally.

**Reproduction Attempts:**

* Tested on Android and iOS devices
* Tested on Wi-Fi and mobile data
* Tested login and primary workflows
* Tested app relaunch and background/foreground transitions
* No failure observed

**Impact:**
Potential intermittent blocking issue affecting mobile usability.

**Required Information / Next Steps:**
Need customer-specific details to isolate the issue:

* Exact failing action
* Error message or screenshot
* Device and OS version
* App version
* Network type
* Reproduction steps

**Suggested Engineering Checks:**

1. Review crash analytics and error logs
2. Check backend API failures around reported timeframe
3. Validate session/authentication handling
4. Verify mobile app telemetry for network and lifecycle events

**Priority:** Medium

---

### Follow-Up Questions to Customer

* What exactly stops working when the issue happens?
* Which phone model and OS version are you using?
* Which app version are you on?
* When did this last happen?
* Does it happen on Wi-Fi, mobile data, or both?
* Do you see any error message or blank screen?
* Does reopening the app temporarily fix the issue?
* What were you doing immediately before it happened?

---

# Part 2 — Test Case Design

## Feature: Invite a Team Member by Email

### Functional Test Cases

| ID    | Test Case                              | Expected Result                 |
| ----- | -------------------------------------- | ------------------------------- |
| TC-01 | Send invite with valid email           | Invite sent successfully        |
| TC-02 | Send invite with invalid email         | Validation error displayed      |
| TC-03 | Submit empty email field               | Required field validation       |
| TC-04 | Invite existing team member            | Proper error/prevention message |
| TC-05 | Invite already invited email           | Duplicate invite handling       |
| TC-06 | Open valid invite link within 48 hours | Invite accepted successfully    |
| TC-07 | Open expired invite after 48 hours     | Expired link message shown      |
| TC-08 | Reuse accepted invite link             | Reuse prevented                 |
| TC-09 | Tamper invite token in URL             | Invalid invite error            |
| TC-10 | Rapid multiple clicks on Send Invite   | Only one invite created         |
| TC-11 | Network failure during invite          | User-friendly error shown       |
| TC-12 | Backend API failure                    | Proper error handling           |
| TC-13 | Verify invite email content            | Correct email/link generated    |
| TC-14 | Mobile responsiveness                  | UI works correctly on mobile    |
| TC-15 | Keyboard accessibility                 | Flow usable without mouse       |

### Security / Edge Cases

* XSS payload in email field
* SQL injection attempt
* Rate limiting / spam prevention
* Authorization validation
* HTTPS invite links
* Token uniqueness and expiration validation

---

# Part 3 — Exploratory Testing

Tested application: [SauceDemo](https://www.saucedemo.com?utm_source=chatgpt.com)

Credentials:

* Username: `standard_user`
* Password: `secret_sauce`

## Bugs Found

### Bug 1 — Cart badge flickers during rapid add/remove actions

**Severity:** Low

**Steps to Reproduce:**

1. Login to the application
2. Rapidly add and remove products
3. Observe cart badge

**Expected:**
Cart badge updates consistently.

**Actual:**
Badge briefly displays inconsistent count/flickering.

---

### Bug 2 — Sort/filter state handling is unclear after page refresh

**Severity:** Medium

**Steps to Reproduce:**

1. Change product sort option
2. Refresh the page

**Expected:**
Sort state handling is clear and consistent.

**Actual:**
Behavior may confuse users because sorting state handling lacks clarity.

---

### Bug 3 — Browser back navigation creates confusing checkout flow

**Severity:** Low

**Steps to Reproduce:**

1. Add item to cart
2. Proceed to checkout
3. Use browser back button repeatedly

**Expected:**
Navigation flow remains intuitive.

**Actual:**
User can land in partially completed checkout states.

---

## UX / Product Improvements

* Missing loading indicators during transitions
* Generic validation messages
* Weak empty-cart guidance
* Limited session timeout handling
* Accessibility improvements needed for keyboard navigation

---

## Next Area to Prioritize

I would prioritize testing network resilience and API reliability next:

* Slow network conditions
* Offline/online transitions
* API retry behavior
* Concurrent cart operations

This area is high-risk for production stability and user experience.

---

# Part 4 — Investigation Scenario

## Issue

Some mobile users see sensor readings that are 2–3 minutes stale after the app remains in the background. Web dashboard shows live data. Restarting the app fixes the issue.

---

## Investigation Plan

### Hypothesis 1 — Mobile app stops refreshing after backgrounding

**Reason:**
Issue only appears after app remains in background.

**Validation Steps:**

* Open app and monitor live updates
* Background app for different durations
* Trigger backend sensor events
* Check whether polling/websocket reconnects after resume

**Evidence Confirming:**
No refresh requests after app resumes.

---

### Hypothesis 2 — Cached data incorrectly reused

**Validation Steps:**

* Compare app timestamps vs backend timestamps
* Inspect cache behavior and API responses

**Evidence Confirming:**
Old cached payload shown instead of fresh data.

---

### Hypothesis 3 — Foreground lifecycle event not handled correctly

**Validation Steps:**

* Monitor app lifecycle logs
* Verify refresh logic execution on app resume

**Evidence Confirming:**
Foreground event occurs but refresh logic does not execute.

---

### Hypothesis 4 — Authentication/session refresh issue

**Validation Steps:**

* Leave app idle until token refresh boundary
* Inspect API/websocket auth responses

**Evidence Confirming:**
401/403 responses or silent reconnect failures.

---

### Hypothesis 5 — OS battery optimization suspends background networking

**Validation Steps:**

* Test with battery saver enabled/disabled
* Compare across devices and OS versions

**Evidence Confirming:**
Issue strongly tied to OS power management behavior.

---

## Investigation Order

1. Reproduce issue consistently
2. Compare mobile vs web timestamps
3. Inspect network activity after app resume
4. Verify websocket/polling reconnection
5. Validate lifecycle handling
6. Check authentication refresh logic
7. Test battery optimization scenarios

---

# Part 5 — Something I’m Proud of Testing

One project I’m genuinely proud of testing was a new feature added to a job board application. Previously, the platform loaded jobs for only a single district, but the new feature introduced support for loading and displaying jobs from multiple districts simultaneously. During testing, I found several critical issues: data was not loading consistently, some backend APIs were intermittently returning 500 Internal Server Errors, and certain frontend UI components were not rendering properly because dependent APIs failed or responded slowly. I wrote automated end-to-end test scripts using Playwright to validate district selection, API responses, filtering, and UI rendering flows. Initially, the automation itself became flaky because components were taking longer to load due to unnecessary frontend rendering and repeated API retries. By analyzing network failures, browser logs, and component loading behavior, I helped identify backend instability and frontend performance bottlenecks. The fixes improved API reliability, reduced page loading time, stabilized the UI rendering process, and made the automation suite reliable enough to be integrated into regression testing for future releases.
