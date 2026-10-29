# K6 lab results
## Part 0
- Step 0.1, http_req_duration: 361.11 ms.
- Step 0.2, requests, rate and p(95), and why the rate stays below 5:
  65 requests, 3.084139 req/s, p(95) = 1.09 s.
  Each user waits for the response and sleeps for 1 second, so each iteration takes longer than 1 second.
- Step 0.6, exit code of the normal run and of the run with P95=300:
  Normal run: 0. P95=300 run: 99.
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
  All three rules passed: p(95)<1000, p(99)<1500, and failure rate<1%.
  p(95) = 667.02 ms; p(99) = 833.55 ms.
  After changing the p(95) limit to 1 ms, the exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it:
  The exclusions used lowercase names instead of 'Pepperoni' and 'Knife'.
  Before fixing, the checks failed 6 and 14 times respectively.
  After fixing, all 186 checks passed.
  Repeated requests exposed incorrect random recommendations that one manual request might miss.
## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.465942 | 631.35 ms | 776.34 ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.089683 | 1.6 s | 3.38 s | 0.00% | 100.00% |
| A3 | Stress | 5 | 1.72372 | 1.83 s | 5.39 s | 0.00% | 100.00% |
| A4 | Spike | 5 | 0.799205 | 2.13 s | 6.9 s | 0.00% | 100.00% |
| A5 | Soak | 2 | 0.834602 | 1.56 s | 6.12 s | 0.00% | 100.00% |
## Part B
1. Results are shown in the Part A table.
2. Each user waits for the response and sleeps for 1 second, so each iteration takes longer than 1 second.
3. p(95) increased from 1.60 s to 1.83 s. No requests failed, but response-time thresholds were exceeded. These small loads cannot identify the system's full operational envelope.
4. Response time first returned near its pre-spike level around 80 seconds, approximately 10 seconds after ramp-down, but fluctuated again afterward.
5. Response times fluctuated without sustained deterioration, and errors remained at 0%. A longer soak test could reveal memory leaks.
6. The average can hide slow requests. p(95) better represents response times experienced by most users.
7. Internet latency varies, and other users create uncontrolled load on the public site.
8. Large tests could disrupt the service for real users and exceed the owner's permission.
9. Vague: “Product searches should be fast.”
   Testable: With 3 users for 30 seconds, search API p(95) must be below 1,000 ms, p(99) below 1,500 ms, and failures below 1%.

   ```javascript
   export const options = {
     vus: 3,
     duration: '30s',
     thresholds: {
       http_req_duration: ['p(95)<1000', 'p(99)<1500'],
       http_req_failed: ['rate<0.01'],
     },
   };
   ```