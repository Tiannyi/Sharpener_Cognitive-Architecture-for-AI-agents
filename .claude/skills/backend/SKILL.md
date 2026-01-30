---
description: Load when choosing or building backend infrastructure — Firebase vs Supabase, cloud functions, databases, or APIs.
---

# Backend Services

> **Domain skill — load when the task involves backend infrastructure, cloud functions, databases, or APIs.**

## Core Philosophy

**Do not build backend infrastructure from scratch.** Use managed services. Your backend should be a thin layer of cloud functions connecting managed services — not a custom server.

## Decision: Firebase vs. Supabase

| Criteria | Firebase | Supabase |
|----------|----------|----------|
| Data model | Document (NoSQL) | Relational (Postgres) |
| Best for | Mobile-first apps, real-time sync, rapid prototyping | Complex queries, joins, relational data |
| Auth | Firebase Auth (mature, many providers) | Supabase Auth (Postgres-based, growing) |
| Real-time | Built-in listeners on documents/collections | Postgres LISTEN/NOTIFY, real-time subscriptions |
| Functions | Cloud Functions (Node.js, Python) | Edge Functions (Deno/TypeScript) |
| Pricing | Pay-per-read/write (can spike) | Predictable (compute + storage based) |
| Vendor lock-in | High (proprietary) | Lower (Postgres underneath) |
| iOS SDK | Excellent, first-class | Good, improving |

**Default recommendation:** Firebase for iOS-first apps. Supabase if you need relational data or want to avoid lock-in.

## What NOT to Build

| Need | Use This | Never Build |
|------|----------|-------------|
| REST API for CRUD | Firestore SDK / Supabase client | Custom Express/FastAPI server |
| Auth server | Firebase Auth / Supabase Auth | Custom auth with JWT |
| File storage | Firebase Storage / Supabase Storage | Custom S3 wrapper |
| Background jobs | Cloud Functions triggers | Custom job queue |
| Real-time updates | Firestore listeners / Supabase realtime | Custom WebSocket server |
| Email sending | SendGrid / Resend (via cloud function) | Custom SMTP |

## What TO Build

- Cloud functions for **business logic** that can't live on the client
- Security rules / row-level security policies
- Data validation beyond what the client enforces
- Third-party API integrations (as cloud functions)
- Scheduled tasks specific to your domain

## Sub-Skill Routing

| Task | Load Sub-Skill |
|------|---------------|
| Cloud Functions, Firestore triggers, Firebase config | `firebase/SKILL.md` |
| Postgres, Edge Functions, RLS policies | `supabase/SKILL.md` |
