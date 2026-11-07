# RushDesign: AI-assisted design for specialty coffee shops

> **Showcase repository.** RushDesign is a commercial product, so its source code stays private. This page explains what it does and how it is built. A live demo and a code walkthrough are available on request.

RushDesign turns an empty commercial space into a **turnkey coffee-shop project**. A franchisee or café owner lays out the bar in the browser and gets the full build package: floor plans, plumbing and electrical plans, an equipment list priced from a real catalog, a budget, a schedule and photoreal renders.

I built it to solve my own problem. I open coffee shops for **Rush Island Coffee Roasters**, a 10-location specialty coffee brand in Mexico. Every new location used to need weeks of back-and-forth between me, an architect, an MEP engineer and suppliers. RushDesign compresses that into a single working session.

---

## What it does

| Area | Capability |
|---|---|
| **2D editor** | A to-scale floor-plan canvas (1 unit = 1 cm) built on Konva. You drag in equipment, furniture and fixtures from the catalog, and it snaps, rotates and locks them. You can trace over a background plan. |
| **3D view** | A real-time 3D view of the same layout, with post-processing, plus a "build" animation module that choreographs how the space comes together, used for client presentations and video. |
| **AI spatial commands** | Natural-language instructions ("move the grinder next to the espresso machine and leave room for the queue") go to an LLM. It returns **typed layout mutations** (`move / rotate / add / swap / delete`) that are validated before they are applied. The model receives hard rules: locked items stay put, the health code requires ≥100 cm of clearance in front of espresso machines, and grinder → espresso → sink must follow a logical bar workflow. |
| **AI copilot** | A conversational "barista copilot" inside the editor for design questions. It runs through a server-side function, so API keys never reach the browser. |
| **Photoreal renders** | A capture of the 3D scene acts as the **depth control image** for a diffusion model (flux-depth-pro). Geometry is locked to the real plan, and the prompt only controls materials, light and mood. The render can't invent a layout that doesn't exist. |
| **Catalog** | 500+ priced SKUs (espresso machines, grinders, refrigeration, furniture, finishes) across three tiers: entry, pro and premium. It includes validation for adding new SKUs. |
| **Brand standard engine** | A franchisor defines, per equipment role, the minimum model for each tier, the approved alternatives, vetoed items and locked elements. The editor enforces the standard and the dossier documents compliance, with sign-off. |
| **Dossier generator** | A single click produces the full package as **PDF and Excel**: layout, hydraulic and electrical plans, bill of materials, budget, WBS and schedule. Entitlements are tiered by subscription. |
| **Menu module** | Menu engineering for the new location (recipes, costing, pricing), tied to the same project. |
| **Billing** | Subscriptions through **Stripe and MercadoPago**, with signed webhooks (fail-closed, constant-time comparison) and Supabase row-level security that prevents self-upgrades from the browser. |

---

## Architecture

```
Browser (React 18 + TypeScript + Vite)
 ├─ Konva 2D editor ──┐
 ├─ 3D view + build animation
 ├─ Zustand stores (project, catalog, brand standard, dossier, menu, billing)
 └─ jsPDF / SheetJS ── dossier export (PDF, XLSX)
          │
          ▼
Vercel Functions (server-side; keys live only here)
 ├─ /api/ai/spatialCommand   LLM → validated layout mutations
 ├─ /api/ai/copilot          conversational design assistant
 ├─ /api/render/photoreal    3D capture → depth-conditioned photoreal render
 ├─ /api/stripe/*            checkout + signed webhook
 ├─ /api/mercadopago/*       preference + signed webhook
 └─ /api/email/welcome
          │
          ▼
Supabase: Postgres + Auth + RLS (projects, brand standards, subscriptions)
```

**Stack:** React 18 · TypeScript · Vite · Tailwind · Konva · Zustand · Supabase (Postgres, Auth, RLS) · Vercel Functions · Gemini / DeepSeek (OpenAI-compatible) · Replicate (flux-depth-pro) · Stripe · MercadoPago · jsPDF · SheetJS · Vitest

**Size:** about 28k lines of TypeScript and 58 test files.

---

## How it was built

I built RushDesign as a solo developer working with AI coding agents. For larger sprints I ran a **multi-agent workflow**: an orchestrator splits the work into disjoint zones (for example 3 code tracks and 4 research tracks), and parallel sub-agents work on them at the same time. The orchestrator then integrates the results behind a hard gate: type-check and production build must pass before anything merges. One sprint went from a broken build to a green, monetizable release, with billing, security hardening, a catalog audit, a competitive benchmark and ad campaigns, without breaking the build despite six areas changing in parallel.

What I design and decide myself:
- the domain model: how a coffee bar actually works, from workflow and clearances to utilities and brand standards
- the safety rails around AI: typed mutations, locked items, server-side keys, validation before apply
- what ships and in which order, driven by what a franchise actually needs to open a location

---

## Status

The MVP is complete and is being piloted for new Rush locations. Payments and the public subdomain are in final configuration.

**Contact:** Roy Camacho · [linkedin.com/in/roycjacome](https://www.linkedin.com/in/roycjacome)

---

© 2026 Roy Camacho Jácome. All rights reserved. This repository contains documentation only.
