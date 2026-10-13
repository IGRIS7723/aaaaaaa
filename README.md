# OAN Portal — Product Master Blueprint & Architecture Guide (A to Z)

> **Document Classification:** Product & Technical Specification Master Reference  
> **Target Audience:** Engineering, Product Management, Leadership, Security & Operations  
> **Application Target:** Oracle APEX 26.x (APEXLang), Oracle Autonomous Database (Serverless), OCI  
> **Application Identifier:** App 103 (`OAN-APP-PORTAL`)  
> **Status:** Authoring Complete (Offline Validated via SQLcl 26.2.2; Gate G0 Ready)  

---

## 1. Executive Summary: What is OAN Portal?

The **Oracle Alumni Network (OAN) Portal** is a dedicated, enterprise-grade digital collaboration, networking, and skills platform built specifically for the global community of former Oracle employees (alumni).

### 1.1 Independence Notice & Brand Boundaries
> **CRITICAL LEGAL NOTICE:**  
> OAN is an **independent alumni community** and is **not affiliated with, sponsored by, or endorsed by Oracle Corporation**.  
> In accordance with strict governance guidelines (Decision Q30):
> - The application carries **no Oracle corporate logos, proprietary red branding, or official Oracle marks**.
> - It is branded cleanly as **"OAN Portal"** using neutral, accessible Redwood design tokens.
> - The platform operates under an independent privacy and data governance framework owned by OAN leadership.

### 1.2 Core Mission & Value Proposition
> **"Connect the right people, skills, knowledge, opportunities, and ideas within the OAN ecosystem."**

The portal replaces fragmented, unvetted external channels (like disconnected WhatsApp chats, generic LinkedIn groups, and email lists) with a single, private, curated hub. The primary measure of platform success is **"find the right person for the right requirement at the right time."**

### 1.3 The 5 Core Capabilities
The entire platform is architected around five interconnected lifecycle stages:
1. **Consolidate:** Unify alumni records, historical domain expertise, curated documentation, and regional chapters into a single source of truth.
2. **Discover:** Enable multi-dimensional search across skills, industries, geographic chapters, and verified Oracle experience.
3. **Connect:** Facilitate peer-to-peer networking, connection requests, and direct messaging without exposing personal contact details to spam.
4. **Collaborate:** Foster community groups, discussion forums, knowledge-sharing libraries, collaborative initiatives, and startup projects.
5. **Grow:** Provide mentorship, skill gap analysis, peer endorsements, partner career opportunities, and structured learning pathways.

---

## 2. Why Does It Exist? (The Problem & Business Drivers)

### 2.1 The Problem
Historically, alumni networks suffer from several critical failure modes:
- **Scattered Communication:** Important discussions, job postings, and technical knowledge are lost across disparate WhatsApp and LinkedIn groups with zero indexing.
- **Privacy vs. Discoverability Dilemma:** Members hesitate to share phone numbers or personal emails publicly, yet desire professional visibility.
- **Spam & Cold Outreach:** Unfiltered partner recruitment and unsolicited solicitations quickly degrade community trust.
- **Verification Gaps:** Public social networks make it difficult to verify authentic alumni tenure and specific enterprise product experience.
- **Lost Knowledge:** Decades of collective technical, consulting, and managerial experience exit the ecosystem when individuals transition careers.

### 2.2 The 6 Strategic Business Goals
The portal delivers on six functional specification goals (FSD §2.3):

| Goal | Strategic Intent | Delivered By |
|---|---|---|
| **1. Reconnect** | Enable alumni to rediscover former colleagues, managers, and peers across teams, business units, and global regions. | Alumni Directory, Hierarchical Regional Chapters, Connection Graph. |
| **2. Re-engage** | Activate community participation through knowledge contribution, discussions, and local events. | Knowledge Library, Community Groups, Discussion Threads, Events. |
| **3. Re-hire** | Provide verified pathways for alumni re-entry, peer job referrals, and career advancement. | Opportunities Feed, Job Search, Peer Referral Requests (J-08). |
| **4. Self-serve** | Give alumni direct tools for credential verification, helpdesk support, and profile management. | Account Settings, Support Tickets, Help-Me-Connect Requests. |
| **5. Employability** | Help members remain at the cutting edge of tech through skill benchmarking, coaching, and training. | Skill Gap Insights (J-02), Domain Experts Corner, Mentoring Programs. |
| **6. Productive Engagement** | Channel entrepreneurial and business energy into vetted projects, consulting, and startup ventures. | Partner Talent Requests (J-09), Project Initiatives, Startup Ideas. |

---

## 3. Platform Personas & Ecosystem Roles

The portal supports six core user roles, managed through Oracle APEX Access Control Lists (ACL):

```
                        ┌─────────────────────────────────────────┐
                        │              Platform Admin             │
                        │ (Global Ops, Security, Taxonomies, ACL) │
                        └────────────────────┬────────────────────┘
                                             │
               ┌─────────────────────────────┼─────────────────────────────┐
               ▼                             ▼                             ▼
  ┌─────────────────────────┐   ┌─────────────────────────┐   ┌─────────────────────────┐
  │     Chapter Admin       │   │        Moderator        │   │      Partner User       │
  │ (Regional Ops & Events) │   │ (Content & Safety SLA)  │   │ (Opportunities & Talent)│
  └────────────┬────────────┘   └────────────┬────────────┘   └────────────┬────────────┘
               │                             │                             │
               └──────────────────────┬──────┴─────────────────────────────┘
                                      ▼
                        ┌─────────────────────────┐
                        │       OAN Member        │
                        │ (Verified Oracle Alumni)│
                        └─────────────┬───────────┘
                                      ▼
                        ┌─────────────────────────┐
                        │       OAN Expert        │
                        │ (Verified Subject Lead) │
                        └─────────────────────────┘
```

1. **OAN Member:** Verified alumnus with `ACTIVE` status. Can build profiles, search directory, connect, post in groups, access library, and apply for opportunities.
2. **OAN Expert:** Member granted the `EXPERT` designation via admin review. Holds a verified expert badge, publishes authoritative articles, hosts office hours, and offers mentoring.
3. **Partner User:** Representative of an approved partner organization. Can post requirements and review candidates who opt in. Bound by the **Partner Contact Rule** (cannot cold-outreach private profiles).
4. **Chapter Administrator:** Manages a regional node (e.g., EMEA › UK › London). Approves local applicants, hosts regional events, and coordinates local networking.
5. **Moderator:** Enforces community guidelines, reviews flagged content, triages pre-moderation queues (library submissions, opportunities), and processes appeals.
6. **Platform Administrator:** Maintains system configurations, taxonomy master data, user roles, security attestations, and audit logs.

---

## 4. Architectural Blueprint: How It Works

### 4.1 High-Level Architecture Diagram

```
       USER INTERFACE LAYER (Browser / Mobile Web - Responsive Universal Theme)
  ────────────────────────────────────────────────────────────────────────────────
        Pages 0-9999 authored offline in APEXLang 26.1/26.2 (mmdVersion 26.1)
  ────────────────────────────────────────────────────────────────────────────────
                                        ▲
                                        │  Synonyms Only (SELECT Views / EXEC Packages)
                                        ▼
   ┌────────────────────────────────────────────────────────────────────────────┐
   │                        PARSING SCHEMA: OAN_APP                             │
   │  • Contains NO Tables and NO Base Data                                     │
   │  • APEX Authentication & Authorization Schemes                             │
   │  • Session State Management & UI Event Handling                            │
   └─────────────────────────────────────┬──────────────────────────────────────┘
                                         │
                                         ▼ Database Proxy Grants
   ┌────────────────────────────────────────────────────────────────────────────┐
   │                         DATA SCHEMA: OAN_DATA                              │
   │                                                                            │
   │   ┌────────────────────────────────────────────────────────────────────┐   │
   │   │ SECURE VIEWS LAYER                                                 │   │
   │   │ • Filters data using oan_vis.can_see(viewer, target, section)      │   │
   │   │ • Respects Viewer Context (OAN_CTX derived from APEX session)      │   │
   │   └─────────────────────────────────┬──────────────────────────────────┘   │
   │                                     │                                      │
   │   ┌─────────────────────────────────┴──────────────────────────────────┐   │
   │   │ PL/SQL PACKAGE ENGINE (23 Packages)                                │   │
   │   │ • oan_auth_pkg, oan_registration_pkg, oan_profile_pkg, etc.       │   │
   │   │ • Lost update detection via ROW_VERSION                            │   │
   │   │ • PBKDF2-HMAC-SHA512 Password hashing (≥ 210,000 rounds)           │   │
   │   └─────────────────────────────────┬──────────────────────────────────┘   │
   │                                     │                                      │
   │   ┌─────────────────────────────────┴──────────────────────────────────┐   │
   │   │ RELATIONAL DATA & TEXT ENGINE (9 DDL Modules, ~50 Tables)           │   │
   │   │ • Oracle Text Indexes (CONTAINS full-text indexing)                │   │
   │   │ • Tamper-evident Audit Traces & Versioned Consents                 │   │
   │   │ • Reserved VECTOR Embeddings columns for AI semantic search        │   │
   │   └────────────────────────────────────────────────────────────────────┘   │
   └────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Two-Schema Security Model
To prevent SQL injection or accidental data leakage through APEX reports, the database is strictly bifurcated:
- **`OAN_DATA` (Owner Schema):** Holds all base tables, constraints, indexes, triggers, and package bodies. It is non-login / proxy-only.
- **`OAN_APP` (APEX Parsing Schema):** The runtime schema for APEX. It owns **zero tables**. It is granted only:
  - `SELECT` on Secure Views (e.g., `oan_profiles_v`, `oan_library_items_v`).
  - `EXECUTE` on API packages (e.g., `oan_registration_pkg`, `oan_moderation_pkg`).
  - An automated database assertion script (`30_assert_no_table_grants.sql`) fails the deployment if any direct table grant is detected.

### 4.3 The Privacy Engine (`oan_vis.can_see` and `OAN_CTX`)
Every query executed in the portal is contextualized by the viewing user:
1. When an APEX request arrives, `oan_ctx_pkg.set_viewer` dynamically resolves the authenticated user ID and roles from the APEX session.
2. In secure views, field visibility is evaluated using the high-performance scalar SQL macro `oan_vis.can_see`:
   ```sql
   oan_vis.can_see(
       p_viewer_id  => sys_context('OAN_CTX', 'USER_ID'),
       p_owner_id   => profile.user_id,
       p_visibility => profile.contact_visibility
   ) = 1
   ```
3. **Visibility Hierarchy:**
   - `PRIVATE`: Visible strictly to the owner and Platform Admins.
   - `CONNECTIONS`: Visible only to approved, 1st-degree mutual connections.
   - `CHAPTER`: Visible to members belonging to the same regional chapter.
   - `MEMBERS`: Visible to any active, authenticated OAN member.
   - `PARTNERS`: Visible to both active members and verified partner organizations.
4. If a viewer is blocked or unapproved, sensitive data resolves to `NULL` before ever leaving the database kernel.

### 4.4 Authentication & Identity Security
The platform implements a multi-tier identity architecture meeting standard enterprise controls:
- **Custom Local Accounts (`OAN_LOCAL`):** Password hashing uses `DBMS_CRYPTO` PBKDF2-HMAC-SHA512 with a strict minimum of **210,000 iterations** and a 32-byte cryptographically secure salt.
- **Social Sign-In (OIDC):** Native Google OAuth 2.0 (`OAN_GOOGLE`) and LinkedIn (`OAN_LINKEDIN`).
- **Privileged Access Enforcement:** Any user holding `ADMIN`, `MODERATOR`, or `CHAPTER_ADMIN` roles must authenticate through an enterprise IdP with multi-factor authentication (MFA) enabled. Local password logins for admins are blocked.
- **Re-Authentication:** Sensitive admin actions (granting roles, deleting accounts) require password/IdP re-challenge.

---

## 5. Detailed Database Model & DDL Architecture

The database schema is organized into 9 ordered DDL modules in `db/ddl/`:

| Script | Module | Key Tables & Responsibilities |
|---|---|---|
| `010_taxonomy.sql` | Taxonomy & Vocabularies | `oan_taxonomies`, `oan_taxonomy_terms`, `oan_term_synonyms`. Controlled lists for skills, industries, Oracle product suites, and member-suggested terms. |
| `020_identity.sql` | Users, Identity & Auth | `oan_users`, `oan_user_credentials`, `oan_consents`, `oan_user_roles`, `oan_sessions`, `oan_blocks`. Versioned GDPR/DPDP consent logs and account lockouts. |
| `030_chapters_registration.sql` | Chapters & Applications | `oan_chapters` (Region › Country › City tree), `oan_chapter_admins`, `oan_registrations` (Pending queue with tenure claims), `oan_registration_history`. |
| `040_profile_network.sql` | Profiles & Networking | `oan_profiles`, `oan_profile_experiences`, `oan_profile_skills`, `oan_connections`, `oan_endorsements`. Section-level visibility settings. |
| `050_content_community.sql` | Library, Groups & Safety | `oan_library_items`, `oan_groups`, `oan_posts`, `oan_comments`, `oan_moderation_items`, `oan_appeals`. Full community discussions and moderation pipeline. |
| `060_ops.sql` | Operational Services | `oan_notifications`, `oan_files` (secure BLOB storage up to 20 MB with MIME verification), `oan_throttles`, `oan_audit_log`. |
| `065_lookup_chat.sql` | Lookups & Chat Engine | `oan_lu_codes`, `oan_chat_conversations`, `oan_chat_messages`. Deterministic 8-step conversational onboarding state machine and journey routes. |
| `070_pd_registry.sql` | Opportunities & Registry | `oan_organizations`, `oan_opportunities` (Jobs, Projects, RFPs), `oan_initiatives` (Startups, Mentoring), `oan_matches` (Weighted scores). |
| `080_text_indexes.sql` | Search Infrastructure | Oracle Text `CONTEXT` indexes across profiles, library documents, and posts with automated sync schedules. |

---

## 6. End-to-End User Experience & The 21 Core Journeys

The platform maps all user activities to 21 structured functional journeys (J-01 through J-21):

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                             THE 21 USER JOURNEYS                                │
├───────────────────────────────┬──────────────────────────────────────────────────┤
│ Category                      │ Journeys Included                                │
├───────────────────────────────┼──────────────────────────────────────────────────┤
│ Career & Talent               │ • J-01: Job Search & Application                 │
│                               │ • J-02: Skill Gap Analysis                       │
│                               │ • J-08: Peer Job Referral Request                │
│                               │ • J-20: Maintain Live CV / Work Profile          │
├───────────────────────────────┼──────────────────────────────────────────────────┤
│ Mentorship & Learning         │ • J-03: Find Expert / Skill Demand Insights      │
│                               │ • J-05: 1-on-1 Coaching & Mentorship             │
│                               │ • J-06: Training Course Enrolment & Labs         │
│                               │ • J-13: Team / Enterprise Training Requests      │
│                               │ • J-14: Domain Expert Advice & Booking           │
├───────────────────────────────┼──────────────────────────────────────────────────┤
│ Community & Ideation          │ • J-04: Experience Sharing in Groups             │
│                               │ • J-07: Propose & Build Business / Startup Ideas │
├───────────────────────────────┼──────────────────────────────────────────────────┤
│ Partner & Commercial          │ • J-09: Partner Posts Talent Need                │
│ Collaboration                 │ • J-10: Review Candidate Submissions             │
│                               │ • J-11: Submit RFP / Partner Bid                 │
│                               │ • J-12: Post Technical Problem for Solving       │
│                               │ • J-15: Regional Product Rollout Partnering      │
│                               │ • J-16: International Expansion Collaboration    │
│                               │ • J-17: Sponsored / Funded Work Matching         │
│                               │ • J-18: Non-Oracle Partner Opportunities         │
│                               │ • J-19: Solution Demonstration of Interest       │
├───────────────────────────────┼──────────────────────────────────────────────────┤
│ Integrations & Omnichannel    │ • J-21: Access OAN Notifications on Slack/WA     │
└───────────────────────────────┴──────────────────────────────────────────────────┘
```

### Detailed Walkthrough of Critical Workflows:

#### Workflow A: Member Registration & Conversational Onboarding (P0 Flow)
1. **Public Signup (Page 101):** User enters name, personal email, and creates credentials (or signs in with Google).
2. **Email Verification (Page 102):** One-time cryptographic token is emailed. User confirms email.
3. **Interactive Onboarding Chat (Page 115):** The user enters a deterministic, guided 8-step conversation powered by `oan_chat_pkg`:
   - Step 1: Welcome & verification of alumni intent.
   - Step 2: Primary alumni location & suggested chapter assignment.
   - Step 3: Former Oracle business unit, role, and exit year.
   - Step 4: Primary skills and domains of expertise.
   - Step 5: Primary goal on OAN (Job search, networking, consulting, mentoring).
   - Step 6: Initial privacy preferences (Public to members vs Connections only).
   - Step 7: Terms of service and DPDP/GDPR consent confirmation.
   - Step 8: Completion summary. Answers automatically populate the draft profile.
4. **Formal Submission (Page 110):** The draft application is finalized and queued.
5. **Admin Approval (Page 1010/1011):** Chapter or Platform Admin inspects tenure details, verifies legitimacy, and approves.
6. **Smart Routing:** On next login, the user is greeted by Page 1 (Home) with customized route cards reflecting their stated journey intent.

#### Workflow B: Find & Reconnect (Networking & Directory)
1. **Directory Search (Page 1):** Member searches by name, former product group, skills, or chapter.
2. **Profile Inspection (Page 11):** System displays profile fields subject to `oan_vis.can_see`. Contact details remain hidden if set to `CONNECTIONS`.
3. **Connection Request:** Member sends a connection request with an optional note.
4. **Approval & Endorsement:** Once accepted, contact details become mutually visible. The peer can now leave **Positive-Only Skill Endorsements** (Page 11).

#### Workflow C: Partner Requirement & Matching
1. **Requirement Publishing:** An approved partner posts a talent or consulting requirement with required skills, industry tags, and experience level.
2. **Deterministic Match Scoring:** The database executes a weighted scoring algorithm:
   $$\text{Score} = (\text{Skills} \times 0.50) + (\text{Industry} \times 0.20) + (\text{Region} \times 0.15) + (\text{Experience} \times 0.15)$$
3. **Candidate Opt-In:** Qualified members receive an alert in their Opportunities Feed. **Partners cannot cold-message members.** A connection is established only when the member explicitly indicates interest.

---

## 7. APEX Application Map (App 103 — 37 Authored Pages)

The frontend is authored in declarative APEXLang files under `oan-app-portal/pages/`:

| Page | Title / File | Functionality |
|---|---|---|
| **0** | Global Page (`p00000-global-page.apx`) | Navigation bar, user menu, global alert messages, session state indicators. |
| **1** | Home (`p00001-home.apx`) | Personalized dashboard, intent cards, upcoming events, recent discussions, global search. |
| **2** | Application Status (`p00002-application-status.apx`) | Displays status for pending/under-review applicant accounts. |
| **10** | My Profile (`p00010-my-profile.apx`) | Comprehensive CV builder: experience, education, skills, certifications, and privacy switches. |
| **11** | Member Profile (`p00011-member-profile.apx`) | Public view of another member, filtered through `oan_vis.can_see`. Connect and Endorse buttons. |
| **12** | Account Settings (`p00012-account-settings.apx`) | Password reset, notification preferences, GDPR data export, account deactivation. |
| **20** | Knowledge Library (`p00020-knowledge-library.apx`) | Filterable repository of technical articles, case studies, videos, and historical knowledge. |
| **21** | Library Item (`p00021-library-item.apx`) | Detailed document viewer, rating/bookmarking, and secure download handler (`beforeHeader`). |
| **22** | Submit Library Item (`p00022-submit-library-item.apx`) | Form for members/experts to contribute content into the pre-moderation queue. |
| **30** | Discussions / Groups (`p00030-discussions.apx`) | Community hub listing groups (General, Cloud, ERP, Founders, Regional). |
| **32** | New Post (`p00032-new-post.apx`) | Authoring modal for initiating discussions or asking technical questions. |
| **37** | Post Detail (`p00037-post-detail.apx`) | Thread conversation view with nested comments, like actions, and report flags. |
| **80** | Report an Item (`p00080-report-item.apx`) | Confidential flagging interface for inappropriate content, policy violations, or IP concerns. |
| **81** | My Decisions (`p00081-my-decisions.apx`) | Transparent list of moderator actions taken against user's content, with appeal submission. |
| **100**| Welcome (`p00100-welcome.apx`) | Public landing page explaining OAN's mission, values, and onboarding options. |
| **101**| Create Account (`p00101-create-account.apx`) | Email/password sign-up and Google/LinkedIn social authentication buttons. |
| **102**| Confirm Email (`p00102-confirm-email.apx`) | Verification token intake and confirmation page. |
| **103**| Reset Password (`p00103-reset-password.apx`) | Self-service secure password recovery flow. |
| **104**| Legal & Privacy (`p00104-legal.apx`) | Full text of Terms of Service, Privacy Policy, and Community Guidelines. |
| **105**| Report a Concern (`p00105-report-concern.apx`) | Public/unauthenticated reporting mechanism for external copyright or identity issues. |
| **106**| Accept Updated Terms (`p00106-accept-updated-terms.apx`) | Interstitial gate requiring acceptance when terms of service are revved. |
| **107**| Link Account (`p00107-link-existing-account.apx`) | Merges social OAuth accounts with existing verified local profiles. |
| **108**| Confirm Identity (`p00108-confirm-identity.apx`) | Re-authentication modal for high-privilege administrative sessions. |
| **109**| Access Denied (`p00109-access-denied.apx`) | Friendly 403 authorization boundary page. |
| **110**| Membership Application (`p00110-membership-application.apx`)| Detailed form capturing Oracle tenure, employee badge number (optional), and chapter preference. |
| **115**| Onboarding Chat (`p00115-onboarding-conversation.apx`)| Interactive 8-step conversational intake UI using the APEX Comments template. |
| **1010**| Approval Queue (`p01010-approval-queue.apx`)| Admin dashboard of pending member applications sorted by submission age. |
| **1011**| Application Review (`p01011-application-review.apx`)| Admin dossier view of applicant answers with Approve, Reject, and Request More Info actions. |
| **1020**| Member Directory Admin (`p01020-members.apx`)| Platform user management, search by status, role assignment, and suspension tools. |
| **1021**| Member Detail Admin (`p01021-member-detail.apx`)| Deep admin inspection of user security state, audit trails, and role management. |
| **1030**| Chapters Admin (`p01030-chapters-admin.apx`)| Hierarchical tree view of global chapters with chapter coordinator assignments. |
| **1031**| Chapter Requests (`p01031-chapter-requests.apx`)| Intake for member petitions requesting the establishment of new city/regional chapters. |
| **1040**| Taxonomy Admin (`p01040-taxonomy.apx`)| Controlled vocabulary editor for skills, industries, synonym merging, and term curation. |
| **1050**| Moderation Queue (`p01050-moderation-queue.apx`)| Triage center for reported comments, pending library submissions, and flagged profiles. |
| **1051**| Appeals Admin (`p01051-appeals.apx`)| High-level review panel for hearing member appeals on moderated content. |
| **1080**| Audit Log (`p01080-audit-log.apx`)| Read-only, un-exportable audit trail of administrative actions, logins, and permission changes. |
| **9999**| Login (`p09999-login.apx`) | Primary authentication gate with credential validation and Social Sign-In redirects. |

---

## 8. Governance, Privacy & Compliance (GDPR & DPDP)

The platform was built from the ground up to comply with the European Union **General Data Protection Regulation (GDPR)** and the **India Digital Personal Data Protection (DPDP) Act 2023**:

1. **Explicit, Versioned Consent:** Sign-up records the exact legal text version accepted (`oan_consents`). Any terms update forces an interstitial re-acceptance gate (Page 106).
2. **Right to Access & Data Portability:** Members can trigger a one-click automated export of their entire dossier (profile, posts, comments, connections) formatted in machine-readable JSON/CSV (Page 12).
3. **Right to Erasure (Forgotten):** Account deletion initiates a 30-day grace period. After 30 days:
   - Personal profile attributes and credentials are hard-deleted.
   - Authored discussion posts are pseudonymized and attributed to `"Former Member"`.
   - Security audit logs are scrubbed of PII but retained for 2 years for fraud prevention.
4. **Partner Anti-Harvesting Controls:** Partners are prohibited from running bulk scrapers or downloading raw member directories. Profiles can only be viewed individually through the secure web UI.

---

## 9. Key Technical & Design Decisions (Decision Log Highlights)

The project history includes 30 formal consensus decisions (Q1 through Q30) that resolved all BRD/FSD ambiguities:

| Decision # | Core Issue | Resolution & Rationale |
|---|---|---|
| **Q2** | Ratings vs. Endorsements | **Endorsements only.** Star ratings on human professionals foster toxic dynamics. Endorsements are strictly positive and tied to concrete skills, protected by anti-abuse rate limits (max 20/day). |
| **Q4** | Historical Conversations | Raw chat transcripts from WhatsApp/Slack are **never imported raw**. They are curated into anonymized, high-value Knowledge Articles or structured FAQ posts. |
| **Q7** | Expert Financials | **Zero on-platform payments or commissions.** Experts may label offerings as "Free", "Paid", or "On Request", but commercial transactions occur off-platform with zero OAN liability. |
| **Q14** | Technical Performance | Target scale: 10,000 active members, 500 concurrent users. 95th percentile page load under 2.0s, search response under 3.0s. Monthly availability target: 99.5%. |
| **Q16** | Authentication Model | Pluggable OIDC with Google and LinkedIn, accompanied by hardened local accounts. Privileged accounts (admins) strictly require IdP sign-in with MFA. |
| **Q22–Q28**| FSD v1.1 Integration | Consolidated functional scope across 8 phased delivery waves, aligning FSD user journeys directly with PRD privacy macros. |
| **Q29** | Oracle HR Dependency | **Independent execution.** Direct automated brokering with Oracle internal HR systems is excluded from v1. The helpdesk provides public signposts only. |
| **Q30** | Brand Independence | Absolute removal of Oracle corporate logos and trademarked assets. Enforced via automated CI brand linting (`ci/check-brand.ps1`). |

---

## 10. Phased Roadmap & Implementation Waves

The platform implementation is structured into 8 cohesive waves (W0 through W7) scheduled across 16 sprints (S0 to S15):

```
W0: Foundation & Access (Identity, Roles, Onboarding Chat, Admin Queues)
 │
 ├──► W1: Front Door & Navigation (Home, Search, Directory Discovery)
 │
 ├──► W2: Profiles & Privacy (Dynamic CV, Section Toggles, Connections)
 │
 ├──► W3: Knowledge & Learning (Curated Library, Document Viewer, Submissions)
 │
 ├──► W4: Community & Discussions (Groups, Post Threads, Moderation Queue)
 │
 ├──► W5: Careers & Opportunities (Job Listings, Referrals, Helpdesk)
 │
 ├──► W6: Ecosystem & Growth (Partners, Domain Experts, Mentorship, Chapters)
 │
 └──► W7: Hardening, Load Testing & Production Launch (Gate 8.0 Go-Live)
```

### Sprint & Gate Schedule
- **Sprint Cycle:** 16 two-week sprints (S0 to S15).
- **Quality Gates:** 11 formal review gates (e.g., Gate G0 Database Deployment, Gate G3 Registration SLA, Gate G-RB Rollback Verification, Gate G8.0 Production Cutover).
- **Current Milestone:** Wave 0 authoring is complete. All 37 APEXLang pages and 23 PL/SQL packages validate cleanly offline against SQLcl 26.2.2.

---

## 11. Developer & Operations Quick Start

### 11.1 Repository Structure
```
OAN_APP_Portal/
├── ci/                             # CI/CD and validation automation
│   ├── check-brand.ps1             # Enforces zero Oracle logo/brand violations
│   ├── deploy-apex-app.ps1         # SQLcl automated deployment script
│   └── validate-apex-app.ps1       # Offline APEXLang syntax and grammar validator
├── db/                             # Complete database source code
│   ├── ddl/                        # 9 ordered DDL scripts (~50 tables)
│   ├── grants/                     # Two-schema proxy grants and assertions
│   ├── packages/                   # 23 PL/SQL business logic packages
│   ├── seed/                       # Taxonomies, lookup codes, and system seeds
│   ├── tests/                      # utPLSQL automated unit test suites
│   ├── views/                      # Context-aware secure views
│   └── install.sql                 # Master deployment orchestration script
├── docs/                           # Master project documentation
│   ├── 01-requirements-review.md   # BRD decisions log (Q1–Q30)
│   ├── 02-PRD.md                   # Formal Product Requirements Document
│   ├── 03-fsd-v1.1-gap-analysis.md # FSD vs. PRD alignment analysis
│   ├── 05-flow-priority.md         # Ranked implementation flow matrix (P0–P4)
│   └── delivery-board.html         # Interactive delivery Kanban board
├── oan-app-portal/                 # Oracle APEX application source (App 103)
│   ├── application.apx             # App definition and Redwood theme config
│   ├── pages/                      # 37 declarative APEXLang page files
│   └── shared-components/          # Auth schemes, LOVs, lists, automations
├── findings.md                     # Engineering scratchpad & toolchain notes
├── progress.md                     # Chronological milestone execution log
├── task_plan.md                    # Detailed sprint and task execution plan
└── product.md                      # This comprehensive master product guide
```

### 11.2 Offline Validation
Before pushing any code or deploying to a live Oracle APEX environment, run the offline validator:
```powershell
# Validates all APEXLang declarative pages and shared components
.\ci\validate-apex-app.ps1 -AppDirectory .\oan-app-portal

# Validates compliance with brand independence rules
.\ci\check-brand.ps1
```

### 11.3 Database Deployment Sequence
To deploy to an Oracle Autonomous Database via SQLcl:
```sql
-- 1. Connect as deployment user into the data schema:
sql -cloudconfig Wallet_OAN.zip oan_deploy[oan_data]/***@oan_high

-- 2. Execute master install (DDL, packages, views, seed, utPLSQL tests):
@db/install.sql

-- 3. Switch to the APEX parsing schema to generate synonyms:
conn oan_app/***@oan_high
@db/grants/20_app_synonyms.sql

-- 4. Verify two-schema isolation (must return zero table grants):
conn oan_data/***@oan_high
@db/grants/30_assert_no_table_grants.sql
```

---

## 12. Summary Checklist: The Project at a Glance

| Feature / Dimension | Specification / Architecture Implementation |
|---|---|
| **What is it?** | An independent alumni collaboration, networking, and opportunities platform for former Oracle staff. |
| **Core Technology** | Oracle APEX 26.x (Universal Theme Redwood) on Oracle AI Autonomous Database (Serverless). |
| **Separation of Concerns** | Strict Two-Schema Architecture: `OAN_DATA` (base storage & logic) and `OAN_APP` (APEX parsing schema with synonyms only). |
| **Data Privacy Engine** | Dynamic session context (`OAN_CTX`) + scalar SQL macro `oan_vis.can_see` embedded directly into database views. |
| **User Authentication** | Enterprise OIDC (Google, LinkedIn) + PBKDF2-HMAC-SHA512 (≥ 210,000 rounds). Privileged roles require IdP + MFA. |
| **Reputation System** | Positive-only, skill-linked member endorsements with anti-abuse rate limits (no star ratings on people). |
| **Partner Matching** | Multi-factor weighted scoring (Skills 50%, Industry 20%, Region 15%, Experience 15%) with strict candidate opt-in privacy. |
| **Current Build State** | 100% authored offline: 37 APEX pages, 23 PL/SQL packages, ~50 tables, passing offline SQLcl 26.2 validation. |

---
*Created as part of the OAN Portal Master Engineering Series. For full technical appendices, consult [02-PRD.md](file:///d:/DEV%20WORK/OAN%20APP/OAN_APP_Portal/docs/02-PRD.md) and [task_plan.md](file:///d:/DEV%20WORK/OAN%20APP/OAN_APP_Portal/task_plan.md).*
