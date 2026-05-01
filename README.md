# UPV CRS 2.0 — Course Registration and Student Information System

---

##  System Summary

**UPV CRS 2.0** is a modern reimagining of the University of the Philippines Visayas Course Registration and Student Information System (CRSIS). The existing system at [crs.upv.edu.ph](https://crs.upv.edu.ph/) relies on legacy JSP (JavaServer Pages) technology — a server-rendered architecture from the early 2000s — with an outdated UI, table-based HTML layouts, and no mobile responsiveness.

UPV CRS 2.0 replaces this with a secure, fast, and user-friendly web application that serves **students, faculty, and administrators** across all UPV campuses (Miagao, Iloilo City, and Tacloban). Core features include:

- Online enrollment and subject registration
- Real-time class offerings and schedule viewing
- Grade viewing and transcript requests
- Document request portal (Form 137, Honorable Dismissal, etc.)
- Announcements and academic calendar management
- Admin dashboard for registrar staff
- Faculty grade encoding interface
- Scholarship and STS (Socialized Tuition System) integration

### 👥 Team Members

| Name | Role |
|------|------|
| *(Member 1)* | Project Lead / Backend |
| *(Member 2)* | Frontend Developer |
| *(Member 3)* | Database / DevOps |
| *(Member 4)* | UI/UX Designer |

---

##  Tech Stack

### i. Frontend Tools

**Framework: [Next.js](https://nextjs.org/) (React)**

Next.js was chosen as the frontend framework because it supports both **Server-Side Rendering (SSR)** and **Static Site Generation (SSG)**, which is critical for a university portal where some pages (e.g., class offerings, academic calendars) are mostly static while others (e.g., enrollment, grades) require real-time data. Next.js also provides file-based routing, API routes, and excellent performance out of the box.

**Styling: [Tailwind CSS](https://tailwindcss.com/)**

Tailwind CSS provides utility-first styling that allows rapid, consistent UI development without writing large CSS files. It produces a small production bundle and integrates well with Next.js. The existing CRS portal uses raw HTML tables with inline styles — Tailwind helps enforce a clean design system instead.

**UI Component Library: [shadcn/ui](https://ui.shadcn.com/)**

shadcn/ui provides accessible, customizable components (modals, tables, forms, dropdowns) built on Radix UI primitives. This is important for a university portal where **accessibility (a11y)** matters — students with disabilities need a usable interface.

**State Management: [Zustand](https://zustand-demo.pmnd.rs/)**

Zustand is a lightweight client-side state manager. It handles enrollment cart state, session context, and UI state without the boilerplate of Redux.

**Form Handling: [React Hook Form](https://react-hook-form.com/) + [Zod](https://zod.dev/)**

All enrollment and document request forms require strict validation. React Hook Form handles form state efficiently, while Zod enforces schema-level validation on both client and server, preventing malformed submissions.

---

### ii. Backend Tools

**Runtime & Framework: [Node.js](https://nodejs.org/) + [Express.js](https://expressjs.com/)**

The backend is a RESTful API built on Express.js running on Node.js. Express is mature, well-documented, and has a large ecosystem of middleware. It handles all business logic: authentication, enrollment processing, grade encoding, and document requests.

Alternatively, Next.js API routes could handle lightweight endpoints, but a separate Express backend keeps concerns cleanly separated and allows independent scaling.

**Authentication: [NextAuth.js](https://next-auth.js.org/) / [Passport.js](http://www.passportjs.org/) + JWT**

UP Visayas uses a university-wide email system (`@upv.edu.ph`). Authentication is handled via **OAuth 2.0 with Google (UP's G Suite)** so students and faculty log in with their existing UP accounts — no new passwords needed. JSON Web Tokens (JWT) are used for stateless session management between the frontend and backend.

**Email Service: [Nodemailer](https://nodemailer.com/) + SMTP**

For document request confirmations, enrollment notifications, and announcements, Nodemailer connects to SMTP to send transactional emails.

**File Handling: [Multer](https://github.com/expressjs/multer)**

Multer handles multipart file uploads (e.g., proof of payment for online fees, STS supporting documents). Files are validated by type and size before being stored.

---

### iii. Database

**Primary Database: [PostgreSQL](https://www.postgresql.org/)**

PostgreSQL is the best choice for a university registration system because:

- Student records, enrollment data, grades, and course offerings are all **highly relational** — they involve many-to-many relationships (students ↔ subjects, faculty ↔ classes) that relational databases handle natively.
- PostgreSQL has strong **ACID compliance** (Atomicity, Consistency, Isolation, Durability), which is non-negotiable for enrollment transactions where double-enrollment or data loss would be a serious problem.
- It supports **row-level security**, useful for ensuring students can only access their own records.
- It is open-source and free, which aligns with the UP System's mandate to prefer open-source tools.

**ORM: [Prisma](https://www.prisma.io/)**

Prisma provides type-safe database queries, auto-generated migrations, and a clean schema definition language. It prevents raw SQL injection vulnerabilities and makes database schema changes manageable.

**Caching: [Redis](https://redis.io/)**

Redis is used as an in-memory cache for high-read, low-write data such as the class offerings list and academic calendar. During enrollment periods, thousands of students simultaneously check available slots — Redis prevents repeated expensive database queries and keeps the system responsive under load.

---

### iv. Other Tools

| Tool | Purpose |
|------|---------|
| **[Docker](https://www.docker.com/)** | Containerizes the frontend, backend, and database for consistent environments across development, staging, and production |
| **[GitHub Actions](https://github.com/features/actions)** | CI/CD pipeline — automatically runs tests and deploys on push to `main` |
| **[ESLint](https://eslint.org/) + [Prettier](https://prettier.io/)** | Code linting and formatting for consistent code style across the team |
| **[Jest](https://jestjs.io/) + [Supertest](https://github.com/ladjs/supertest)** | Unit and integration testing for API endpoints |
| **[Sentry](https://sentry.io/)** | Error monitoring and performance tracking in production |
| **[Nginx](https://nginx.org/)** | Reverse proxy that sits in front of the Node.js app, handles SSL termination, rate limiting, and static asset serving |

---

##  Hosting

### Platform: [Railway](https://railway.app/) (primary) with [Cloudflare](https://www.cloudflare.com/) (CDN + DNS + DDoS protection)

---

### How the Hosting Works

#### Application Hosting — Railway

Railway is a Platform-as-a-Service (PaaS) that runs Docker containers without requiring manual server management. The system is deployed as three separate services on Railway:

1. **Frontend (Next.js)** — Deployed as a container. Railway automatically assigns an HTTPS endpoint. The Next.js app is served from Railway's edge.
2. **Backend (Express.js API)** — Deployed as a separate container on its own internal Railway service. The frontend communicates with it via environment variable-configured URLs, and it is **not directly exposed to the internet** — only the frontend and Nginx proxy can reach it.
3. **PostgreSQL** — Railway provides a managed PostgreSQL instance with automatic daily backups, connection pooling, and a private network URL. The database is never publicly accessible; only the backend service within the same Railway project can connect to it.
4. **Redis** — Also provisioned as a Railway add-on, accessible only internally.

Railway was chosen because:
- It provides managed infrastructure without requiring a dedicated DevOps engineer, which is appropriate for a student/academic team.
- It supports **automatic deployments from GitHub** via GitHub Actions — push to `main`, and Railway redeploys automatically.
- It offers **private networking** between services, so the database and cache are never exposed to the public internet.
- It includes **built-in SSL/TLS** certificates (HTTPS), critical for protecting student data in transit.

#### CDN and Security — Cloudflare

The domain (`crs2.upv.edu.ph` or equivalent) is proxied through **Cloudflare**, which provides:

- **DDoS protection** — During enrollment periods, the portal is a target for traffic spikes (both legitimate and malicious). Cloudflare absorbs attack traffic before it reaches Railway.
- **Web Application Firewall (WAF)** — Blocks common attack patterns (SQL injection, XSS, bot scraping) at the network edge.
- **CDN caching** — Static assets (JS bundles, CSS, images, PDFs like academic calendars) are cached at Cloudflare's global edge nodes, reducing load on Railway and improving load times for students in Miagao, Tacloban, and beyond.
- **SSL/TLS encryption** — Cloudflare issues and manages the public-facing SSL certificate and enforces HTTPS-only access.

#### Data Security and Reliability

| Concern | Solution |
|---------|----------|
| Student data privacy (RA 10173 / Data Privacy Act) | All data encrypted in transit (TLS 1.3) and at rest (PostgreSQL encryption); no sensitive data in logs |
| Database loss | Railway auto-backups + manual weekly exports to a separate cloud storage bucket |
| Downtime during enrollment | Railway allows horizontal scaling (multiple container replicas); Redis reduces DB pressure |
| Unauthorized access | JWT authentication, rate limiting via Nginx + Cloudflare, row-level security in PostgreSQL |
| Credential exposure | All secrets stored as Railway environment variables, never in source code; `.env` files are in `.gitignore` |

#### Deployment Flow (CI/CD)

```
Developer pushes to GitHub (main branch)
        ↓
GitHub Actions runs: lint → test → build Docker image
        ↓
Image pushed to Railway
        ↓
Railway deploys new container (zero-downtime rolling update)
        ↓
Cloudflare serves traffic to new deployment
```

This means no manual server logins are ever needed, and a bad deployment can be rolled back in Railway's dashboard in under a minute.

---
## Mockup ##
- - # See html files in repository and run locally using live server
- - # All information in mockups are made up and reflect no real people and no real student
- - # Vibe coded with GPT 5.2 Codex