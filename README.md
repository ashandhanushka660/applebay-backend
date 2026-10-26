# Applebay Institute - Hardened Academic Portal

Spring Boot 3.3.4 + Next.js 14 (static export) student-management system hardened to the same security posture used for the Ashan Lanka system: JWT auth, role-based access control, security headers/CSP/HSTS, login rate-limiting and account lockout, audit logging, Flyway migrations, actuator hardening, and strict error handling.

## Stack
- Backend: Java 21, Spring Boot, Spring Security, JWT (jjwt), Flyway, PostgreSQL
- Frontend: Next.js 14 App Router, Tailwind CSS (static export served by Spring Boot or any static host)
- Database: PostgreSQL (local or Neon)

## Quick start (local)
### Prerequisites
- Java 21, Maven 3.9+, Node.js 20+, PostgreSQL 15+

### Database
```sql
CREATE DATABASE applebay;
CREATE DATABASE applebay_test;
```

### Backend
```bash
cd backend
export JWT_SECRET="change-me-to-a-32+-byte-random-secret"
export ADMIN_PASSWORD="ChangeMe!2026Strong"
export SPRING_DATASOURCE_URL="jdbc:postgresql://localhost:5432/applebay"
export SPRING_DATASOURCE_USERNAME="postgres"
export SPRING_DATASOURCE_PASSWORD="postgres"
./mvnw spring-boot:run  # or mvn spring-boot:run
```
On first start, Flyway creates schema and `DataInitializer` creates the admin account from `ADMIN_EMAIL`/`ADMIN_PASSWORD`. Default admin email: `admin@applebay.local`.

### Frontend
```bash
cd frontend
npm ci
echo "NEXT_PUBLIC_API_BASE_URL=http://localhost:8080/api" > .env.local
npm run dev
```

## Security features
- JWT bearer authentication; tokens invalidated on password change (tokenVersion)
- All business APIs require `ROLE_ADMIN`
- Security headers: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`, `COOP/CORP`, CSP, and HSTS when behind HTTPS
- Login rate-limiting (IP-based) and account lockout after repeated failures
- Audit events recorded for auth actions
- Generic error messages to avoid information disclosure
- Actuator limited to health probes; JPA open-in-view disabled
- Request ID propagation and structured logging

## Tests
```bash
cd backend
TEST_DATABASE_URL=jdbc:postgresql://localhost:5432/applebay_test mvn test
```
Covers payment validation, attendance eligibility, and security integration (auth, headers, authz).

## Docker
```bash
docker build -t applebay-sms -f backend/Dockerfile .
docker run -p 8080:8080 \
  -e JWT_SECRET="..." \
  -e ADMIN_PASSWORD="..." \
  -e SPRING_DATASOURCE_URL="..." \
  -e SPRING_DATASOURCE_USERNAME="..." \
  -e SPRING_DATASOURCE_PASSWORD="..." \
  applebay-sms
```
Static frontend is embedded in the JAR under `/`.

## Deployment (free)
Recommended: [Neon](https://neon.tech/) for PostgreSQL + [Render](https://render.com/) (Web Service, Docker) or [Railway](https://railway.app/). Set required env vars: `JWT_SECRET`, `ADMIN_PASSWORD`, `SPRING_DATASOURCE_URL/USERNAME/PASSWORD` (use Neon connection string), `FRONTEND_URL` (your frontend origin) or rely on same-origin serving.

## Notes
- Payment records cannot be deleted via API; integrity enforced by balance checks and `@PastOrPresent` dates
- `spring.jpa.hibernate.ddl-auto` is `validate` with Flyway managing schema
- Source of truth schema in `database/` and migrations in `backend/src/main/resources/db/migration/`
