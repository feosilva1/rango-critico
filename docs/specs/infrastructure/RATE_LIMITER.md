# Description & Motivation

The rate limiter will define the rules that the application shall enforce to prevent abuses, like brute force attacks or user spam. This is extremely important to prevent unnecessary costs and hardware overloads that can cause regular users to be affected.

# Functional Requirements

## FR1: Service consumers of any kind (human or machine) while spamming any action in the application must be restricted from using the application for a period of time.

## FR2: Rate limit rules must be configured via .env files, without support for dynamic reconfiguration.

## FR3: The rate limiter must automatically identify consumers by User ID when authenticated, and fall back to IP address for unauthenticated (public) requests.

## FR4: A generic error must be returned when the rate limit exceeds. No details regarding the restriction can be exposed to the user to avoid understanding about the system rules.

## FR5: To be able to differentiate between heavy and light resources, rate limits per resource must be configurable in the application. This will allow, e.g., higher rate limits for popular and light-weight resources and lower limits for heavy and expensive resources.

## FR6: Additionally to specific resource limits, a global rate limiter will be enforced across all endpoints using the Token Bucket algorithm.

## FR7: Shall implement a Token Bucket algorithm.

# Non-functional Requirements

## NFR1: The limiter shall have low latency, this is only a piece of a whole request and definetly is not the main goal of any end user. So this shouldn't be a bottleneck since we need to assume we likely have a much more heavy process after the limiter check.

## NFR2: Must be Resilient and Fail-Open. If the rate limiter storage or checking mechanism fails for any unexpected reason, the request must pass to prevent the rate limiter from bringing down the application.

## NFR3: Idependently of where the limiter will live, we need to assume that the components that compound the system can be replicated to deal heavy workloads. Therefore, we need to have a centralized storage.

# Critical User Journeys

## CUJ1: Anonymous User Rate Limited on Public Endpoint (IP Identification)

- **Step 1:** An unauthenticated user (or bot) sends multiple requests in rapid succession to a public endpoint (e.g., `/auth/login`).
- **Step 2:** Rate limiter middleware detects no active session, resolves the client IP address, and checks token availability in Redis for the resource.
- **Step 3:** Available tokens drop below request cost. The rate limiter rejects the request, returning an HTTP 429 status code with a generic error payload and a `Retry-After` header.

## CUJ2: Authenticated User Rate Limited on Heavy Resource (User ID Identification)

- **Step 1:** An authenticated user initiates multiple heavy resource requests.
- **Step 2:** Rate limiter middleware identifies the authenticated user and evaluates the bucket.
- **Step 3:** The user exhausts their token bucket for that resource and receives an HTTP 429 error. Other users sharing the same public IP remain completely unaffected.

## CUJ3: Centralized Storage Failure (Fail-Open Behavior)

- **Step 1:** An incoming request arrives at the application while centralized storage (Redis) is experiencing network downtime or latency.
- **Step 2:** Rate limiter middleware attempts to evaluate token availability, but the connection times out or throws an error.
- **Step 3:** The rate limiter catches the storage exception, logs a warning metric to internal monitoring, and passes the request through (Fail-Open), preventing application downtime.

# Alternatives Considered

## Fixed Window Counter

- **Pros:** Simple `INCR` + `EXPIRE` implementation.
- **Cons:** Vulnerable to traffic spikes at window boundaries (2x limit burst in short window).

## Sliding Window Log

- **Pros:** Precise window tracking.
- **Cons:** High memory footprint storing every timestamp in Redis sets.

## Sliding Window Counter

- **Pros:** Good accuracy and low memory usage.
- **Cons:** Assumes requests are evenly distributed across the time window (which is not always true in real scenarios), leading to potential inaccuracies during concentrated traffic bursts.

## Leaky Bucket

- **Pros:** Enforces smooth constant output flow.
- **Cons:** Can introduce artificial latency or reject legitimate user bursts on web SPAs.

## Token Bucket (Selected)

- **Pros:** Allows burst capacity up to bucket size while enforcing a stable refill rate.
- **Cons:** Requires more memory than fixed window counters.
