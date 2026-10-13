# Draw the Path: Portfolio Sitemap + Toolkit Setup

## 1. Minimalist Portfolio Sitemap

To ensure the user journey directly drives towards a single action (scheduling a technical interview), the sitemap is kept deliberately lean. Every section earns its place by supporting the core claim: *building high-performance backends and scalable API integrations*.

```
[ Hero Section ]  --->  [ Case Studies ]  --->  [ Engineering Profile ]  --->  [ Direct CTA ]
 (Proof Statement)       (3 Core Proofs)          (Stack & Background)          (Book Interview)
```

### Page Breakdown & Intent

1. **Homepage / Main Landing (Single-Page Layout)**
   * **Hero Section:** States the exact claim immediately ("High-Performance Backend Engineer specializing in Laravel, Rust, and System Integrations"). Displays a primary CTA button: *"Schedule a Technical Interview"*.
   * **Selected Engineering Case Studies (The Proof):**
     1. *Rust Microservice Migration:* Benchmarks & code architecture for rewriting a Laravel backend using Rust (Actix Web).
     2. *Enterprise Data Migration & HRMS:* System migration of 15,000+ records and UAT execution at PT Petro Graha Medika.
     3. *High-Volume Integration Engine:* Multi-platform API integration (Shopee, RajaOngkir, WhatsApp/Telegram automation).
   * **Core Technical Stack & Engineering Philosophy:** Summarizes proficiency in PHP/Laravel + Swoole, Rust, Docker CI/CD, and database optimizations.
   * **About & Contact (Footer CTA):** Concise background highlighting assistant lecturer experience and research background, linked directly to Cal.com / Calendly embed for quick interview booking.

2. **Dedicated Case Study Template (`/case-studies/[project-slug]`)**
   * *Problem Statement & Context:* Business logic challenge and scalability requirements.
   * *Architecture & Code:* System design diagram, concurrency handling, and API endpoints.
   * *Impact & Benchmarks:* Quantitative metrics (response time reduction, system uptime, records processed).
   * *Target Action:* Persistent button at the bottom: *"Discuss this architecture with Faiz"*.

---

## 2. AI Toolkit & Claude Project Setup

### Account Verification
- [x] **Claude (Anthropic)** — Primary Thinking Partner & Portfolio Tutor
- [x] **ChatGPT (OpenAI)** — Auxiliary code helper & alternative critique
- [x] **Gemini (Google)** — Deep context research & architectural review
- [x] **Perplexity AI** — Real-time industry bench-marking & tech stack verification

### Claude Project Configuration
* **Project Name:** `Faiz Portfolio Builder (8-Week Track)`
* **Project Description:** Custom workspace to guide, critique, and pressure-test the design, content, and code of my backend engineer portfolio.

#### Custom Instructions (System Prompt)
```text
You are acting as an elite Senior Technical Recruiter and CTO Tutor guiding Faiz Nurullah through building his backend software engineering portfolio.

Faiz's Proof Statement:
"This portfolio proves my capability to design and build high-performance backend systems and enterprise API integrations (using Laravel, Rust, and Docker), specifically targeted at Engineering Managers, Tech Leads, or CTOs at growth-stage tech startups and enterprise companies, so that they immediately schedule a technical interview using the primary meeting link on this page."

Rules for interaction:
1. Challenge any content, page, or design element that feels decorative or generic.
2. Evaluate all case studies from a Tech Lead / CTO perspective (focus on concurrency, benchmarks, data integrity, and clean architecture).
3. Keep the user focused strictly on the target audience (Engineering Managers/CTOs) and the single target action (scheduling a technical interview).
```

---

## 3. Sitemap Pressure-Test Execution

### The Pressure-Test Prompt
> **User Prompt to Claude:**
> "Here is my proposed portfolio sitemap:
> 1. Hero Section with Proof Statement & Meeting Link
> 2. Case Studies Section (Rust Rewrite, HRMS Migration, API Integration Platform)
> 3. Academic Research & ML Projects Section (TFLite, Hanacaraka SVM, OCR)
> 4. Full Technical Stack List (PHP, Dart, Python, JS, Rust, C#, Laravel, Vue, React, Flutter, .NET, Docker, MySQL, Mongo, etc.)
> 5. About Me & Academic Background
> 6. Blog / Research Notes Page
> 7. Contact Form
> 
> Pressure-test this structure against my Proof Statement and target audience (Engineering Managers / CTOs). Is every section necessary, or am I watering down my primary claim?"

### AI Critique & Output Summary

```text
[Claude Response]:
Your sitemap currently suffers from the "kitchen sink" problem. You are trying to prove you are a High-Performance Backend Engineer, but your sitemap presents you as a generalist CS student who dabbles in frontend, machine learning, mobile, and academic blogging.

1. Cut Section 6 (Blog / Research Notes): A Tech Lead assessing you for a backend role will not read an unfinished blog. It distracts from your case studies.
2. Refine Section 3 (Academic ML): Move research papers into a small accordion or footnote inside the About section. Displaying ML papers prominently confuses your primary positioning as a Backend / Infrastructure Engineer.
3. Condense Section 4 (Tech Stack): Listing 12+ languages and frameworks weakens your specialized claim. Group your stack strictly around Backend, DevOps, and System Design (Laravel, Swoole, Rust, Docker, PostgreSQL, Redis/REST). Hide legacy/secondary tools.
4. Replace Contact Form with Direct Booking: Tech Leads hate generic contact forms. Use a direct calendar widget (e.g., Cal.com) alongside your email and LinkedIn.
```

### Refinements & Key Takeaways

1. **Removed the Blog Page:** Eliminates unnecessary noise and keeps the visitor's focus purely on backend metrics and scheduling an interview.
2. **Re-focused Tech Stack:** Trimmed down generic languages (Flutter, C#, Dart) from the top fold to highlight **PHP/Laravel + Swoole**, **Rust (Actix Web)**, **Docker**, and **Database Performance**.
3. **Streamlined Academic Projects:** Moved ML and paper citations into a secondary tab under "Background" to prevent overshadowing production backend case studies.