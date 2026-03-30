# Cartfy Architecture (MVP)

## Recommended stack
- Backend: Node.js + NestJS (or Express) / alternatively Laravel.
- Database: PostgreSQL.
- Cache & rate limit: Redis.
- Storage: S3-compatible bucket for avatars.
- Reverse proxy: Nginx / Caddy (for custom domain routing).

## Request flow
1. User scans NFC card.
2. Device opens direct URL (slug or custom domain).
3. Server resolves profile by:
   - `domains.hostname == request.host` OR
   - `profiles.slug == path slug`.
4. Server renders active profile template with allowed dynamic fields.
5. If user clicks Save Contact, request goes to `/vcard/:slug.vcf` and receives generated vCard.

## Custom domain strategy
- Store requested hostname in `domains`.
- Verify ownership by DNS TXT or CNAME check.
- On verified domains, route traffic by Host header.
- Force HTTPS certificates via wildcard/automatic ACME flow.

## HTML/CSS injection guardrails
- Allow admin-provided template region only (e.g., inside a sandboxed container).
- Sanitize HTML with allowlist tags.
- Strip scripts and event handlers.
- Sanitize CSS to prevent dangerous expressions/imports.
- Consider CSP headers for stricter runtime protection.

## Minimal API contract
- `POST /admin/customers`
- `PATCH /admin/profiles/:id/template`
- `PATCH /customer/profile`
- `GET /profiles/:slug`
- `GET /vcard/:slug.vcf`
- `POST /domains/verify`

## vCard fields (minimum)
- `FN` Full Name
- `TITLE` Job Title
- `TEL` Phone
- `EMAIL`
- `URL` Main profile link
- `PHOTO` (optional, URL)
