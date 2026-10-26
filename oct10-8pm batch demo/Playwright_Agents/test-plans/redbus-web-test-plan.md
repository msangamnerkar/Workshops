# redBus Public Website Test Plan

## Application Overview

A browser-based functional test plan for redBus.com covering the public bus-booking journey from the home-page search form through route results and seat selection, plus booking lookup, locale/currency controls, and key navigation. Observed on https://www.redbus.com/: source and destination textboxes, departure and optional return-date fields, SEARCH button, INR selector, Manage Booking, Help, and popular-route links. Run scenarios independently from a fresh browser context; use only test data and stop before authorizing payment or creating a real booking. Availability, labels, and validations can vary by market and date, so expected outcomes should be checked against the selected market's current behavior.

## Test Scenarios

### 1. Home-page search and route discovery

**Seed:** `tests/seed.spec.ts`

#### 1.1. Reject or guide incomplete bus searches

**File:** `tests/redbus/home-search-validation.spec.ts`

**Steps:**
  1. Open https://www.redbus.com/ in a fresh browser context and leave all search fields blank.
    - expect: The home page shows origin, destination, departure date, optional return date, and SEARCH controls.
  2. Click SEARCH without entering any values.
    - expect: The site does not navigate to misleading search results.
    - expect: Required fields are identified or the search remains blocked with clear validation guidance.
  3. Enter an origin only and click SEARCH; then enter a destination only and click SEARCH in a separate fresh run.
    - expect: Each incomplete search remains blocked and clearly indicates the missing required input.

#### 1.2. Search for a valid one-way bus trip

**File:** `tests/redbus/one-way-search.spec.ts`

**Steps:**
  1. Open the home page in a fresh browser context.
    - expect: The bus search form is available with no prefilled trip values.
  2. Enter a supported origin and choose the matching autocomplete result; enter a different supported destination and choose its matching result.
    - expect: Selected cities are displayed as recognized values, not merely uncommitted free text.
  3. Choose a valid future departure date and leave return date empty.
    - expect: The selected departure date is displayed and the optional return date remains unset.
  4. Click SEARCH.
    - expect: A results view loads for the selected origin, destination, and date, or a clear no-service/no-buses state is shown.
    - expect: Displayed route and travel date match the submitted search.

#### 1.3. Search for a round trip and validate date ordering

**File:** `tests/redbus/round-trip-search.spec.ts`

**Steps:**
  1. In a fresh browser context, select valid supported origin and destination cities.
    - expect: Origin and destination are selected and distinct.
  2. Set a future departure date and a later return date, then search.
    - expect: The round-trip search proceeds and retains both dates in the results or trip summary.
  3. In a separate run, set the return date earlier than the departure date, then attempt to search.
    - expect: The site prevents the invalid date order or explains the date constraint; it does not silently submit an invalid round trip.

#### 1.4. Handle city autocomplete, unmatched input, and identical cities

**File:** `tests/redbus/city-autocomplete.spec.ts`

**Steps:**
  1. Focus the origin field and type a supported city name using mixed case and a partial spelling.
    - expect: Relevant city suggestions appear and can be selected with pointer or keyboard interaction.
  2. Select a suggestion, then focus destination and type an unsupported random string.
    - expect: The unsupported value is not accepted as a valid city; an empty/no-match state or useful guidance is presented.
  3. Select the same supported city for origin and destination and attempt search.
    - expect: The site prevents an identical-city route or clearly reports that the route is invalid.

#### 1.5. Validate departure and return date boundaries

**File:** `tests/redbus/date-picker-boundaries.spec.ts`

**Steps:**
  1. Open the departure date control from a fresh home page.
    - expect: The date picker or date-entry interaction opens and communicates selectable dates.
  2. Attempt to select a past departure date, then attempt to enter a malformed date directly if typing is supported.
    - expect: Past dates and malformed values cannot be submitted as valid departures; feedback is understandable.
  3. Select a valid future departure date and inspect the optional return-date control.
    - expect: Return-date choices respect the departure-date constraint and a valid later return date can be selected.

### 2. Search results and booking flow

**Seed:** `tests/seed.spec.ts`

#### 2.1. Review and refine bus search results

**File:** `tests/redbus/search-results-filters.spec.ts`

**Steps:**
  1. Run a valid search for a route and date that have inventory in the test environment.
    - expect: Results identify the searched route and date and show available services or an explicit empty state.
  2. Inspect available sort, filter, operator, timing, or bus-type controls; apply each available filter individually and clear it.
    - expect: Results update to match active filters, and clearing filters restores the broader result set.
  3. Change sort order using available sort controls.
    - expect: Result ordering changes consistently with the selected sort option without changing the route or travel date.

#### 2.2. Select seats and boarding/drop-off points

**File:** `tests/redbus/seat-selection.spec.ts`

**Steps:**
  1. From an available trip result, open the seat-selection or trip-details flow.
    - expect: Seat availability and any selection instructions are displayed for the chosen service.
  2. Select one available seat, then select a second seat if multi-seat booking is supported.
    - expect: Chosen seats are visibly marked and the fare/selection summary updates accurately.
  3. Attempt to select a seat already marked unavailable and continue without choosing required boarding or drop-off points, if these are required.
    - expect: Unavailable seats cannot be selected and required journey points are validated before proceeding.
  4. Choose valid boarding and drop-off points and continue to passenger details.
    - expect: The selected seats and journey points remain consistent in the next-step summary.

#### 2.3. Validate passenger details and stop before payment

**File:** `tests/redbus/passenger-checkout-safety.spec.ts`

**Steps:**
  1. Proceed from seat selection using test data only and inspect the passenger/contact form.
    - expect: Required passenger/contact fields and any consent or policy notices are clearly identified.
  2. Submit the form with required details missing and then with malformed email/phone values, where those fields are present.
    - expect: Invalid or missing details are blocked with field-level guidance; entered values are preserved where appropriate.
  3. Enter valid synthetic test details and proceed only as far as the payment/review page.
    - expect: The trip, passenger, seats, and fare summary are internally consistent.
  4. Stop without entering payment credentials or authorizing payment; exit the flow.
    - expect: No real purchase or payment authorization is made during testing.

### 3. Account and site navigation

**Seed:** `tests/seed.spec.ts`

#### 3.1. Validate manage-booking lookup and invalid references

**File:** `tests/redbus/manage-booking.spec.ts`

**Steps:**
  1. From a fresh home page, open Manage Booking.
    - expect: A booking lookup or management screen opens and explains the required lookup information.
  2. Submit the lookup form empty and then with clearly invalid/nonexistent test reference data.
    - expect: Empty or invalid lookups are blocked or return a clear not-found/error message without exposing another passenger's booking.
  3. If a valid sandbox booking is available, retrieve it and inspect cancellation/change actions without confirming any irreversible action.
    - expect: Only the matching booking is displayed and any cancellation/change operation requires an explicit confirmation.

#### 3.2. Change currency or market and validate navigation

**File:** `tests/redbus/locale-currency-navigation.spec.ts`

**Steps:**
  1. Open the currency/market selector visible in the site header.
    - expect: Available selections are understandable and the current setting is indicated.
  2. Choose another available currency or market and inspect the page and fare presentation.
    - expect: The selected setting is reflected consistently where supported; route and date fields remain usable.
  3. Use browser back and return to the default market if needed.
    - expect: Navigation remains functional and the user can return to the original home-page state.

#### 3.3. Navigate to help and popular route pages

**File:** `tests/redbus.help-and-popular-routes.spec.ts`

**Steps:**
  1. From a fresh home page, open Help.
    - expect: The Help destination loads successfully and presents relevant support content.
  2. Return to the home page and open a visible popular-route link, such as Vienna to Bratislava when displayed.
    - expect: The route page corresponds to the selected route and offers relevant trip information or search results.
  3. Use the site's home navigation or browser back to return to the booking form.
    - expect: The home page is reachable and the main search controls remain available.
