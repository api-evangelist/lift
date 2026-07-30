---
name: Book a HEXA flight experience
description: Look up LIFT Aircraft's business details, find available HEXA introductory
  flight sessions, and start a booking on a visitor's behalf through the site's public
  MCP endpoint.
api: mcp/lift-mcp.yml
operations:
  - GetBusinessDetails
  - SearchInSite
  - SearchSiteApiDocs
  - GenerateVisitorToken
  - CallWixSiteAPI
---

# Book a HEXA flight experience

LIFT Aircraft sells flight experiences, not aircraft. A visitor books a session at a
base, completes a short training, and then flies HEXA — an 18-motor amphibious powered
ultralight — solo, with no pilot's licence required. There is no developer REST API and
no OpenAPI. The only machine surface is the site's public MCP endpoint.

**Endpoint:** `https://www.liftaircraft.com/_api/mcp` (HTTP, MCP protocol `2025-06-18`)

## Before you start

- No authentication is needed to connect or to read. Only information already published
  on the public site is reachable.
- Anything that acts on a visitor's behalf — querying site data, booking, starting a
  purchase — requires a visitor token first. See step 3.
- This server cannot take payment. Checkout always finishes on the website; hand the
  visitor a link rather than claiming a booking is paid for.

## Steps

1. **Ground yourself in the business.** Call `GetBusinessDetails` to get the timezone,
   contact email, phone and address. Booking availability is published in the site's
   local timezone — read it here rather than assuming the visitor's.

2. **Find what is on offer.** For availability, locations and session content, call
   `SearchInSite`. For anything that needs the underlying booking or store data, call
   `SearchSiteApiDocs` instead — it returns the API documentation for the Wix business
   solutions actually installed on this site, and tells you which methods exist. Do not
   guess method names; the installed solution set is site-specific.

3. **Open a visitor session.** Call `GenerateVisitorToken` to create the session and
   obtain the visitor access token. This must happen before the first `CallWixSiteAPI`
   request, or the call will be rejected.

4. **Read the method schema before calling it.** Take the reference article URL that
   `SearchSiteApiDocs` returned and call `ReadFullDocsMethodSchema` (and
   `ReadFullDocsArticle` for worked examples) so the request body matches the real
   schema. Skipping this is the most common cause of a malformed booking call.

5. **Act.** Call `CallWixSiteAPI` to query availability and to create the booking or
   start the purchase. Confirm the base, date and time back to the visitor in the
   business's timezone before committing.

6. **Hand off checkout.** Direct the visitor to the site to complete payment — the
   booking page is `https://www.liftaircraft.com/book-a-flight`.

## Conventions and cautions

- **Read before write.** `SearchSiteApiDocs` → `ReadFullDocsMethodSchema` →
  `CallWixSiteAPI`. The tool descriptions state this ordering explicitly.
- **No idempotency contract is published.** Do not blindly retry a booking or purchase
  call that times out — re-query availability first and confirm whether the prior call
  landed, or you risk a duplicate reservation.
- **Tool list can change.** The server advertises `tools.listChanged`; re-issue
  `tools/list` when you receive a tool-update notification.
- **Availability is genuinely scarce.** Sessions frequently show no availability and
  the site offers a waitlist instead. Treat "no slots" as the normal case, not an error.
- **Do not quote prices from memory.** Fares are published per location on the booking
  page and change; read them live.
- **Stay inside the public surface.** There is no account login, no user-scoped token
  and no member data reachable here. If a visitor asks for order history or personal
  records, refer them to `support@liftaircraft.com`.
