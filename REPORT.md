# Enhancing Testing with Large Language Models

## Basis and transparency

The requirements below come from `Assignment.md`. All analysis in this report is AI-assisted. Step 1 supplies example initial ideas rather than misrepresenting them as human-generated before AI use. The student had no prior ideas to supply. Review decisions and reflection are worked examples, not claims about the student's personal experience. Both model responses were generated in the same conversation after the requirements had been read; they were not independent experiments. No tests were run against the application.

| ID | Requirement |
| --- | --- |
| R1 | A reservation is between 1 and 31 nights. |
| R2 | A reservation includes between 1 and 12 guests. |
| R3 | The price is $200 per night. |
| R4 | Each valid reservation includes a one-time $50 cleaning fee. |
| R5 | Each guest over five costs an additional $25 per night. |
| R6 | Check-in occurs before checkout. |
| R7 | Checkout is not a reserved night. |
| R8 | All requested nights must be available. |
| R9 | Invalid booking information produces an error. |
| R10 | Unavailable dates produce an unavailable error. |

For a valid reservation, the price oracle is `nights × (200 + 25 × max(guests − 5, 0)) + 50`. Reserved nights form the interval from check-in inclusive to checkout exclusive. Dates below use ISO format and refer to calendar nights. Prices exclude any invented taxes, discounts, deposits, or payment rules.

## Step 1 — Initial test ideas (AI-assisted examples)

These five examples fill the initial-ideas section, but cannot retroactively satisfy the requirement to create personal ideas before using an LLM. All dates refer to a chosen test rental, and all requested nights are available unless otherwise stated.

| ID | Requirement | Input or action | Expected result |
| --- | --- | --- | --- |
| B1 | R1, R2, R3, R4, R7 | Book 2027-01-10 to 2027-01-11 for 1 guest. | Valid 1-night reservation; total $250. |
| B2 | R1, R9 | Book 2027-01-01 to 2027-02-02 for 1 guest. | 32 nights is invalid; booking error. |
| B3 | R2, R9 | Book 2027-01-10 to 2027-01-11 for 13 guests. | Guest count is invalid; booking error. |
| B4 | R3, R4, R5 | Book 2027-01-10 to 2027-01-13 for 6 guests. | 3 × ($200 + $25) + $50 = $725. |
| B5 | R1, R3, R4, R7 | Book 2028-02-28 to 2028-03-01 for 1 guest. | Leap-day stay covers two nights; total $450. |

## Step 2 — General prompt and complete response

### Prompt

```text
Create tests for a vacation booking application.
```

### Complete AI response

1. Search for a destination and verify that matching properties appear.
2. Select available dates and a valid guest count, then verify that a booking can be created.
3. Try a checkout date earlier than check-in and verify that an error appears.
4. Request dates that are already booked and verify that the application rejects them.
5. Change the number of nights and guests and verify that the total price updates correctly.
6. Submit the booking form with required fields missing and verify that validation messages appear.
7. Apply a valid promotional code and verify that the expected discount is applied.
8. Complete a payment and verify that a confirmation email is sent.
9. Cancel a reservation and verify that the correct refund is issued.
10. Check the booking flow on desktop and mobile screen sizes for usability problems.

### Evaluation

Categories can overlap: a useful direction may still be too vague to execute.

| Idea | Classification | Explanation |
| --- | --- | --- |
| G1 Search | Unsupported | Search is outside the listed booking requirements, even if the starter includes rental browsing. No destination-matching oracle is provided. |
| G2 Valid booking | Useful, vague | Relevant positive path, but dates, guest count, availability setup, and total are missing. |
| G3 Reversed dates | Useful, incomplete | Matches R6/R9, but needs concrete dates and otherwise valid data. Error text is not specified. |
| G4 Booked dates | Useful, vague | Matches R8/R10; distinguish an unavailable error and identify which reserved night is blocked. |
| G5 Price updates | Useful, vague, partly unsupported | Pricing matters, but “correctly” is not an oracle. Live updates are not required. Need exact arithmetic at the booking result. |
| G6 Missing fields | Useful exploration, underspecified | R9 supports invalid-input errors, but required field definitions and validation presentation are not fully specified. |
| G7 Promo code | Unsupported | No promotion feature or discount formula is required. |
| G8 Payment/email | Unsupported | Neither payment processing nor email delivery is required. |
| G9 Refund | Unsupported | No cancellation or refund policy is supplied. |
| G10 Mobile usability | Unsupported acceptance criterion | Could be exploratory testing, but no supported-device or responsive-layout requirements were supplied. |

The response omits the exact 1/31-night and 1/12-guest boundaries, the $200 rate, the one-time $50 fee, the five-to-six guest surcharge boundary, same-day invalid stays, and checkout exclusion. “Valid,” “correctly,” and “expected” leave important outcomes undefined.

## Step 3 — Contextual prompt and complete response

### Prompt

```text
Act as a software tester assisting me with North Vacation Booking.
I remain responsible for deciding which tests to keep and for verifying all
inputs and expected results. Your output is a proposal, not proof of correctness.

Generate exactly 14 test ideas using boundary-value analysis, equivalence
partitioning, pricing checks, and date-availability risks.

Requirements:
- A reservation must be between 1 and 31 nights.
- A reservation must include between 1 and 12 guests.
- The price is $200 per night.
- Each valid reservation includes a one-time $50 cleaning fee.
- Each guest over five costs an additional $25 per night.
- The check-in date must occur before the checkout date.
- The checkout date is not counted as a reserved night.
- All requested nights must be available.
- Invalid booking information produces an error.
- Unavailable dates produce an unavailable error.

For each idea include an ID, requirement, technique, preconditions, concrete
dates and guest count, action, and an exact expected outcome. Show calculations
for valid bookings. Use one chosen test rental with a controllable availability
fixture; reset it between tests. Specify blocked nights where relevant.
Do not invent taxes, payment, refunds, promo codes, emails, exact error text,
or other unsupported requirements. Identify assumptions and implementation
limitations separately. Keep invalid-input tests to one invalid factor each.
```

### Complete AI response

Use rental A with the stated availability fixture, reset before each test. Submit each reservation once. Dates use year-month-day. “Available” means every requested night is available. Invalid cases require a booking error; unavailable cases require an unavailable error. No exact message wording is assumed.

| ID | Requirement / technique | Preconditions | Inputs and action | Expected outcome |
| --- | --- | --- | --- | --- |
| C1 | R1–R4, R7; lower boundaries | Available | Submit 2027-01-10 → 2027-01-11; 1 guest. | Valid 1 night; 1 × 200 + 50 = $250. |
| C2 | R1, R3, R4; upper night boundary | Available | Submit 2027-01-01 → 2027-02-01; 1 guest. | Valid 31 nights; 31 × 200 + 50 = $6,250. |
| C3 | R1, R6, R9; zero-night boundary | Available | Submit 2027-01-10 → 2027-01-10; 1 guest. | Invalid booking error; no valid reservation. |
| C4 | R1, R9; above upper boundary | Available | Submit 2027-01-01 → 2027-02-02; 1 guest. | Invalid booking error for 32 nights. |
| C5 | R6, R9; invalid ordering | Available | Submit 2027-01-11 → 2027-01-10; 1 guest. | Invalid booking error for reversed dates. |
| C6 | R2, R9; below guest boundary | Available | Submit 2027-01-10 → 2027-01-11; 0 guests. | Invalid booking error. |
| C7 | R2–R5; upper guest boundary | Available | Submit 2027-01-10 → 2027-01-11; 12 guests. | Valid; 200 + 7 × 25 + 50 = $425. |
| C8 | R2, R9; above guest boundary | Available | Submit 2027-01-10 → 2027-01-11; 13 guests. | Invalid booking error. |
| C9 | R3–R5; surcharge threshold | Available | Submit 2027-01-10 → 2027-01-13; 5 guests. | Valid 3 nights; 3 × 200 + 50 = $650. |
| C10 | R3–R5; just above threshold | Available | Submit 2027-01-10 → 2027-01-13; 6 guests. | Valid 3 nights; 3 × (200 + 25) + 50 = $725. |
| C11 | R8, R10; interior overlap | Jan 11 unavailable; Jan 10 and 12 available | Submit 2027-01-10 → 2027-01-13; 1 guest. | Unavailable error because one requested night is blocked. |
| C12 | R7, R8; checkout exclusion | Jan 13 unavailable; Jan 10–12 available | Submit 2027-01-10 → 2027-01-13; 1 guest. | Valid; Jan 13 is not reserved; total $650. |
| C13 | R8, R10; first-night overlap | Jan 10 unavailable; Jan 11–12 available | Submit 2027-01-10 → 2027-01-13; 1 guest. | Unavailable error. |
| C14 | R1–R5; combined upper boundaries | Available | Submit 2027-01-01 → 2027-02-01; 12 guests. | Valid; 31 × (200 + 7 × 25) + 50 = $11,675. |

Assumptions and limitations: rental A and availability controls are test fixtures, not specified product features. A test environment must supply them. Exact error wording, error precedence for multiple invalid factors, tax, payment, and persistence behavior are unspecified. These are design proposals; execution depends on an interface that accepts both dates and exposes validation and pricing results.

## Step 4 — Review of every contextual idea

These are AI-assisted review decisions. Accept means suitable as a requirement-based case, not executed or proven to pass. All cases require the shared fixture and implementation access described in Step 6.

| Idea | Decision | Reason | Required correction |
| --- | --- | --- | --- |
| C1 | Accept | Jan 10 to Jan 11 is exactly one night; one guest is valid; $250 is correct. | None to the business oracle. |
| C2 | Accept | January has 31 nights before Feb 1; $6,250 includes one cleaning fee. | None. |
| C3 | Revise | Correct rejection, but zero nights and equal dates necessarily violate both R1 and R6. It cannot meet a literal one-rule-only interpretation of the prompt. | State the coupled violations and avoid claiming which validation executes first. |
| C4 | Accept | 32 nights exceeds the maximum by one while guests and ordering remain valid. | None. |
| C5 | Revise | Reversed dates also imply an invalid duration. | Expect an invalid-booking error without asserting ordering-versus-duration error precedence. |
| C6 | Accept | Zero is immediately below the guest minimum. | None; do not force a specific message. |
| C7 | Accept | Twelve guests means seven additional guests; $425 is correct. | None. |
| C8 | Accept | Thirteen guests is immediately above the maximum. | None. |
| C9 | Accept | Exactly five guests adds no surcharge; $650 for three nights. | None. |
| C10 | Accept | One additional guest for each of three nights adds $75, not $25; total $725. | None. |
| C11 | Accept | A blocked interior night tests “all nights,” not just endpoints. | None. |
| C12 | Revise | The availability exception is correct, but the checkout boundary deserves an explicit occupancy assertion. | List Jan 10, 11, 12 as reserved; Jan 13 must remain outside this reservation. |
| C13 | Accept | Unavailable check-in night is included and must reject the stay. | None. |
| C14 | Accept | 31 × 375 + 50 = $11,675; both maxima are valid inclusively. | None. |

No contextual idea needs rejection: none invents an additional feature. All ten requirements appear in the set, but requirement presence is not exhaustive coverage. Gaps include an unavailable last reserved night, leap-day counting, and additional near-boundary valid values such as 30 nights and 11 guests. The final set adds the first two. Missing-field and malformed-date behavior needs an agreed interface contract before exact acceptance criteria can be written.

### Implementation feasibility

Static inspection of `airbnb-clone-ts/src/pages/AddBooking.tsx` shows a rental selector, a single `date` field, and a guest input with `min="1"`. Submission calls `addBooking(formData)`. `src/context/BookingContext.tsx` appends the object to in-memory state. These inspected paths have no checkout field, duration calculation, pricing calculation, upper guest bound, or availability check. The standalone `BookingForm.tsx` has similar controls.

Consequently, the complete reservation scenarios below cannot be executed through the supplied UI as written. They are planned acceptance tests for the stated requirements, with execution blocked until an appropriate interface and availability fixture exist. Browser minimum validation alone would not verify the missing reservation rules. This report does not change the application or claim passing application tests.

## Step 5 — Comparison

The contextual prompt produced more useful ideas: concrete dates, counts, fixtures, outcomes, and numerical totals make them reviewable. Supplying the requirements replaced generic payment, refund, and promotion suggestions with boundary tests and availability semantics. The general response's clearest weakness was treating unsupported features as test targets and using “correct” without defining an outcome.

A remaining contextual weakness was the mismatch between proposed tests and the starter's interface: the response described required capabilities but did not identify their absence in the actual source. It also did not include a leap-day case. C3/C5 illustrate why a request to isolate invalid factors cannot always isolate individual rules: date ordering and duration are linked.

Relative to the initial example baseline, C12 introduced an idea not previously listed: an unavailable checkout night should not block the preceding stay. B5 introduced a leap-day counting idea that neither response included. These are comparisons between recorded idea sets, not claims about the student's unrecorded thought process.

## Step 6 — Final test cases

### Shared preconditions and actions

Use a test environment with one identified rental A, an interface accepting check-in, checkout, and guests, a price result, and controllable night-level availability. Reset availability and reservations before each case. Use the specified dates directly, with no unspecified past-date policy. If the environment cannot accept them, record the blocker instead of silently changing the test. “Available” below means all requested nights are available for A. No unrelated invalid input is introduced.

For every case, enter the listed dates and guest count and submit once. Inspect the reservation result, price where applicable, and error category. For invalid or unavailable requests, the request must not be accepted as a valid reservation. No exact error wording, presentation location, payment behavior, or persistence across reloads is required.

**Execution status for every case: Not run; blocked for end-to-end execution in the supplied starter.** The table uses the assignment's source labels. “Verified” here means AI-assisted comparison with the requirements and arithmetic, not student verification or a passing execution result. All final cases are AI-origin; none is falsely labeled Human-generated.

| ID | Requirement | Preconditions | Inputs or actions | Expected result | Source |
| --- | --- | --- | --- | --- | --- |
| TC-01 | R1–R4, R7 | Shared setup; available | 2027-01-10 → 2027-01-11; 1 guest | Valid, one reserved night (Jan 10); $250. | AI-generated and verified |
| TC-02 | R1, R3, R4 | Shared setup; available | 2027-01-01 → 2027-02-01; 1 guest | Valid, 31 nights; $6,250. | AI-generated and verified |
| TC-03 | R1, R6, R9 | Shared setup; available | 2027-01-10 → 2027-01-10; 1 guest | Invalid-booking error; equal dates/zero nights rejected; no asserted error precedence. | AI-generated and revised |
| TC-04 | R1, R9 | Shared setup; available | 2027-01-01 → 2027-02-02; 1 guest | Invalid-booking error; 32-night reservation rejected. | AI-generated and verified |
| TC-05 | R1, R6, R9 | Shared setup; available | 2027-01-11 → 2027-01-10; 1 guest | Invalid-booking error; reversed range rejected; no asserted error precedence. | AI-generated and revised |
| TC-06 | R2, R9 | Shared setup; available | 2027-01-10 → 2027-01-11; 0 guests | Invalid-booking error; reservation rejected. | AI-generated and verified |
| TC-07 | R2–R5 | Shared setup; available | 2027-01-10 → 2027-01-11; 12 guests | Valid; $200 + 7 × $25 + $50 = $425. | AI-generated and verified |
| TC-08 | R2, R9 | Shared setup; available | 2027-01-10 → 2027-01-11; 13 guests | Invalid-booking error; reservation rejected. | AI-generated and verified |
| TC-09 | R3–R5 | Shared setup; available | 2027-01-10 → 2027-01-13; 5 guests | Valid, three nights, no guest surcharge; $650. | AI-generated and verified |
| TC-10 | R3–R5 | Shared setup; available | 2027-01-10 → 2027-01-13; 6 guests | Valid; 3 × $225 + $50 = $725; cleaning charged once. | AI-generated and verified |
| TC-11 | R8, R10 | Shared setup; Jan 11 blocked, Jan 10 and 12 available | 2027-01-10 → 2027-01-13; 1 guest | Unavailable error; reservation rejected due to interior night Jan 11. | AI-generated and verified |
| TC-12 | R3, R4, R7, R8 | Shared setup; Jan 13 blocked, Jan 10–12 available | 2027-01-10 → 2027-01-13; 1 guest | Valid; reserve Jan 10, 11, 12 only; do not reserve Jan 13; $650. | AI-generated and revised |
| TC-13 | R8, R10 | Shared setup; Jan 10 blocked, Jan 11–12 available | 2027-01-10 → 2027-01-13; 1 guest | Unavailable error; reservation rejected due to first night Jan 10. | AI-generated and verified |
| TC-14 | R1–R5 | Shared setup; available | 2027-01-01 → 2027-02-01; 12 guests | Valid, 31 nights; 31 × $375 + $50 = $11,675. | AI-generated and verified |
| TC-15 | R1, R3, R4, R7 | Shared setup; Feb 28 and Feb 29, 2028 available | 2028-02-28 → 2028-03-01; 1 guest | Valid, two nights including leap day; 2 × $200 + $50 = $450. | AI-generated and verified |
| TC-16 | R7, R8, R10 | Shared setup; Jan 12 blocked, Jan 10–11 available | 2027-01-10 → 2027-01-13; 1 guest | Unavailable error; Jan 12 is the last reserved night and cannot be excluded like checkout Jan 13. | AI-generated and verified |

TC-01 through TC-14 derive from C1 through C14 respectively. TC-15 derives from B5. TC-16 was added during the coverage review. All final valid totals were recalculated using the stated price oracle.

### Coverage map

| Requirement | Representative final tests |
| --- | --- |
| R1 | TC-01, TC-02, TC-03, TC-04, TC-14, TC-15 |
| R2 | TC-01, TC-06, TC-07, TC-08, TC-14 |
| R3 | TC-01, TC-02, TC-09, TC-10 |
| R4 | TC-01, TC-02, TC-09, TC-10, TC-14 |
| R5 | TC-07, TC-09, TC-10, TC-14 |
| R6 | TC-03, TC-05 |
| R7 | TC-01, TC-12, TC-15, TC-16 |
| R8 | TC-11, TC-12, TC-13, TC-16 |
| R9 | TC-03, TC-04, TC-05, TC-06, TC-08 |
| R10 | TC-11, TC-13, TC-16 |

## Step 7 — Reflection on human and AI responsibilities

The following answers describe this worked example rather than inventing personal experiences for the student.

1. **What did the LLM help complete more quickly?** It expanded a short baseline into boundary, surcharge, and availability scenarios and organized them into consistent tables. It also supplied concrete arithmetic for review. These are potential efficiency gains; no timing experiment was performed.
2. **What did it misunderstand, overlook, or invent?** The general response introduced promotions, payment emails, and refunds without requirements. It omitted exact limits and prices. The contextual response omitted leap-day coverage and did not establish whether the starter supported its tests. Its request to isolate invalid inputs needed qualification for equal or reversed dates.
3. **What knowledge was needed to evaluate it?** Evaluation required inclusive numeric boundaries, calendar-night counting with checkout excluded, leap-year awareness, the distinction between per-night guest charges and a one-time fee, and source inspection to distinguish specified behavior from implemented behavior. For example, six guests for three nights costs $725, because the extra guest incurs $25 on each night.
4. **What could happen without review?** A tester could spend time on nonexistent refund functionality, accept ambiguous pricing checks, miss an off-by-one availability defect, or report coverage for tests that the interface cannot execute. Incorrect oracles could also label correct software as defective or let pricing defects pass.
5. **Which decisions remain the tester's responsibility?** The tester must confirm the authoritative requirements, resolve ambiguities with the product owner, choose priorities and meaningful coverage, validate fixtures and arithmetic, check execution feasibility, inspect actual outcomes, and decide whether evidence supports a defect or release decision. AI proposals do not replace those responsibilities, and this AI-assisted review does not certify that the student performed them.
