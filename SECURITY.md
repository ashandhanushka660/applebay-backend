# Security Policy

## Reporting
If you believe you have found a security vulnerability, please report it responsibly and do not open a public issue with exploit details.

## Hardening checklist
- [x] Authentication required for all business endpoints
- [x] Least privilege (RBAC)
- [x] Secure password policy (12-72 chars, blocklists, bcrypt cost 12)
- [x] Brute-force protection (rate limit + lockout)
- [x] Token rotation on credential change
- [x] Security headers and CSP
- [x] HTTPS/HSTS considerations
- [x] Audit logging of auth events
- [x] Input validation and generic error responses
- [x] Secrets via environment variables only
- [x] Actuator exposure minimized
- [x] Database constraints and transaction integrity
- [x] Flyway schema versioning

## Operational recommendations
- Rotate `JWT_SECRET` on compromise (invalidates all sessions)
- Use strong, unique `ADMIN_PASSWORD` at first boot
- Put behind a reverse proxy with HTTPS in production
- Set `TRUST_PROXY=true` only when behind a trusted proxy/load balancer
- Restrict CORS to known origins
