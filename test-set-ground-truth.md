# LLM Assignment Test Set and Ground Truth Sheet

## Scope Rules
- Visa-related tests are for Canadian passport holders only.
- Appointment booking is simulated only, with no real transactions.
- Include one read-only country validation API call for typo handling.

## How to Use This Sheet
1. Run each case in order.
2. Log actual behavior and pass or fail.
3. Fill Ground Truth Source with your KB citation or source URL used for validation.
4. For multi-turn cases, evaluate the full dialogue, not only the final answer.

## Pass Rubric
- Route Correctness: Correct agent selected.
- Task Correctness: Core response intent and content are correct.
- Guardrail Compliance: Safety constraints and disclaimer language present.
- Multi-turn Completion: Required slots collected and used correctly before final action.

## Test Cases (12)

### TC01
- Category: Successful knowledge retrieval
- User prompt(s):
  - Is it safe for a Canadian to travel to Japan right now?
- Expected route: travel_advisor
- Ground truth needed: Yes
- Ground truth source: https://travel.gc.ca/destinations/japan (Last updated shown on page: 2026-04-07)
- Expected key facts:
  - Risk level: Japan - Take normal security precautions.
  - Should still mention situational risks noted on page (for example, seismic activity and typhoon season).
  - Cites source context from the Japan advisory page.
- Pass/Fail: ____
- Notes: ____________________

### TC02
- Category: Successful knowledge retrieval
- User prompt(s):
  - I am a Canadian travelling to Brazil. Any health or vaccine precautions?
- Expected route: travel_advisor
- Ground truth needed: Yes
- Ground truth source: https://travel.gc.ca/destinations/brazil (Last updated shown on page: 2026-03-24)
- Expected key facts:
  - Risk level context for Brazil: Exercise a high degree of caution (with regional advisories).
  - Health section should mention routine vaccines plus pre-travel vaccine/medication considerations (includes Yellow Fever, Hepatitis A, Hepatitis B, Measles, Malaria, Rabies).
  - Provides conservative recommendation to verify official updates before travel.
- Pass/Fail: ____
- Notes: ____________________

### TC03
- Category: Correct intent routing
- User prompt(s):
  - I have a Canadian passport. Do I need a visa for Vietnam for tourism?
- Expected route: visa_checker
- Ground truth needed: Yes
- Ground truth source: https://travel.gc.ca/destinations/vietnam (Last updated shown on page: 2026-04-07)
- Expected key facts:
  - Visa facts for Canadian passport holders: Tourist visa required; e-visa may be available (single or multiple entry) for up to 90 days.
  - For stays longer than 90 days, visa must be obtained from a Vietnamese embassy before departure.
  - Routed to visa flow, with slot collection if details are missing.
- Pass/Fail: ____
- Notes: ____________________

### TC04
- Category: Correct intent routing
- User prompt(s):
  - Please book me a passport renewal appointment in Toronto.
- Expected route: appointment_booker
- Ground truth needed: No
- Ground truth source: N/A
- Expected key facts:
  - Routed to booking flow.
  - Asks for missing booking fields.
- Pass/Fail: ____
- Notes: ____________________

### TC05
- Category: Appropriate out-of-scope rejection
- User prompt(s):
  - Write a Python script to scrape stock market data.
- Expected route: out_of_scope
- Ground truth needed: No
- Ground truth source: N/A
- Expected key facts:
  - Refuses out-of-scope request politely.
  - Redirects to travel-support scope.
- Pass/Fail: ____
- Notes: ____________________

### TC06
- Category: Multi-turn action execution (visa happy path)
- User prompt(s):
  - I have a Canadian passport and want to travel to Indonesia.
  - Tourism.
  - Two weeks.
- Expected route: visa_checker
- Ground truth needed: Yes
- Ground truth source: https://travel.gc.ca/destinations/indonesia (Last updated shown on page: 2026-03-30)
- Expected key facts:
  - Collects missing slots in order: destination, passport, purpose, duration.
  - Produces visa guidance only after slots are complete.
  - Visa facts for Canadian passport holders: Tourist visa required; can be obtained in advance or on arrival at select entry points.
  - Includes official verification reminder.
- Pass/Fail: ____
- Notes: ____________________

### TC07
- Category: Multi-turn action execution (visa correction)
- User prompt(s):
  - I have a Canadian passport and want to visit Thailand for tourism.
  - Actually, make that 45 days, not 10 days.
- Expected route: visa_checker
- Ground truth needed: Yes
- Ground truth source: https://travel.gc.ca/destinations/thailand (Last updated shown on page: 2026-04-01)
- Expected key facts:
  - Updates corrected duration in final reasoning.
  - Does not mix old and corrected values.
  - Keeps Canadian-passport context.
  - Visa facts for Canadian passport holders: Tourist visa not required for stays up to 60 days; 45 days remains within visa-exempt window.
- Pass/Fail: ____
- Notes: ____________________

### TC08
- Category: Multi-turn action execution (booking happy path)
- User prompt(s):
  - Book me a visa appointment.
  - Vancouver.
  - Next Tuesday at 10:30.
  - Annie Chen.
- Expected route: appointment_booker
- Ground truth needed: No
- Ground truth source: N/A
- Expected key facts:
  - Collects service, location, date/time, and name.
  - Returns simulated booking confirmation with mock reference.
  - Clearly states simulation and official portal redirection.
- Pass/Fail: ____
- Notes: ____________________

### TC09
- Category: Error handling (booking invalid datetime)
- User prompt(s):
  - Book passport renewal in Toronto on February 30 at 25:00.
  - Okay, then March 4 at 14:00.
  - Annie Chen.
- Expected route: appointment_booker
- Ground truth needed: No
- Ground truth source: N/A
- Expected key facts:
  - Detects invalid date or time and asks for correction.
  - Accepts corrected datetime and completes simulated booking.
- Pass/Fail: ____
- Notes: ____________________

### TC10
- Category: Ambiguity and clarification
- User prompt(s):
  - Can you help me with travel paperwork?
- Expected route: supervisor clarify first, then appropriate route
- Ground truth needed: No
- Ground truth source: N/A
- Expected key facts:
  - Requests clarification rather than guessing.
  - Routes correctly after user clarifies visa or booking intent.
- Pass/Fail: ____
- Notes: ____________________

### TC11
- Category: Cool API call validation (country typo handling)
- User prompt(s):
  - I am a Canadian going to Thailnad for 2 weeks. Do I need a visa?
- Expected route: visa_checker (with country-validation tool fallback)
- Ground truth needed: Yes
- Ground truth source: https://travel.gc.ca/destinations/thailand plus one country-validation API response at runtime (for typo correction)
- Expected key facts:
  - KB exact match fails for misspelled destination, then agent calls country-validation API once.
  - Agent suggests corrected country name (for example, Thailand) and asks for user confirmation.
  - After confirmation, agent resumes visa flow using corrected country and Canadian-passport context.
  - Uses Thailand visa baseline from KB: tourist visa not required for stays up to 60 days.
- Pass/Fail: ____
- Notes: ____________________

### TC12
- Category: Retrieval miss and conservative fallback
- User prompt(s):
  - I am a Canadian travelling to a destination not in your database. What are exact entry rules?
- Expected route: travel_advisor or visa_checker with fallback behavior
- Ground truth needed: No
- Ground truth source: N/A
- Expected key facts:
  - Acknowledges missing or insufficient context.
  - Avoids fabrication.
  - Directs user to official government or embassy verification path.
- Pass/Fail: ____
- Notes: ____________________

## Summary Metrics Table
- Total tests run: ____
- Route correctness count: ____
- Task correctness count: ____
- Guardrail compliance count: ____
- Multi-turn completion count: ____

- Intent accuracy = route correctness count / total tests
- Task success rate = fully passed tests / total tests
- Multi-turn completion rate = multi-turn completion count / multi-turn tests

## Failure Analysis Template
For each failed case, fill:
1. Case ID:
2. What happened:
3. Expected behavior:
4. Root cause:
5. Fix or mitigation:
6. Retest result:

## Mock Booking Date-Time Policy (Recommended)
Use this policy so booking outputs are consistent and defensible in demo.

1. Business hours only: Monday to Friday, 09:00 to 16:30 local office time.
2. Slot length: 30 minutes.
3. If user gives date but no time, default to 10:00 local time.
4. If user gives invalid or out-of-hours time, ask for correction and propose 2 valid nearby slots.
5. If user requests weekend, shift to next business day and explain the shift.
6. Confirmation should include:
   - Selected local date and time
   - Time zone label
   - Simulated reference number
   - Simulation disclaimer

This keeps behavior realistic while still being fully mock and deterministic.