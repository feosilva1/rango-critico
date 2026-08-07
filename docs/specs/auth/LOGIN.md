# Description & Motivation

The login feature allows users to authenticate and access protected features. This is required to deliver a personalized experience by tying user data and preferences directly to their profile. Ultimately, it gives users a dedicated space to manage their reviews and build their profile over time.

# Functional Requirements

## FR1: Magic link login where users enter their email and receive an authentication link. Clicking the link validates access and redirects them to the app, fully authenticated.

## FR2: The login endpoint must return a generic success message regardless of whether the email exists in the database to prevent user enumeration attacks.

# Technical Requirements

## TR1: Rate limiting to prevent abuse. The user shall have a cooldown of 1 minute, and a limit per email of 3 requests per 15 minutes, and a limit per IP of 10 requests per hour.

## TR2: Email addresses must be trimmed and converted to lowercase before validation, rate-limiting, and storage.

## TR3: The magic link token shall be cryptographically secure (CSPRNG), one-time use (burned upon consumption), and have a 10-minute TTL.

## TR4: Access token shall have a TTL of 15 minutes and a refresh token shall have a TTL of 7 days.

# Critical User Journeys

## CUJ1: Successful magic link authentication flow

step 1: The user enters their email address and submits the login form.

step 2: The user opens their email provider, receives the magic link message, and clicks the authentication button.

step 3: The application validates the token, burns it, issues JWT tokens (access + refresh), and redirects the user fully authenticated.

## CUJ2: Attempt login with an expired or already used magic link

step 1: The user clicks an expired or already used magic link in their email.

step 2: The application rejects the token and displays an error message informing the user that the link is invalid or expired, offering an option to request a new link.

# Spec(optional):

```json
{
    // User's primary email address used to identify the account and dispatch the magic link authentication email.
    // Must follow local-part@domain format with total length between 5 and 254 characters (referencing RFC 5321 and RFC 5322). Local-part (max 64 chars) accepts letters, numbers, '.', '-', '_', and '+' without leading, trailing, or consecutive dots (RFC 5322 dot-atom format subset). Domain must consist of dot-separated labels (supporting subdomains and TLDs) using letters, numbers, and hyphens.
    "email": string // required
}
```
