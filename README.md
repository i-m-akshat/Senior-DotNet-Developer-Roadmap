# Senior .NET Developer Roadmap & Encyclopedia (2026 Edition)

> **The Complete .NET Developer Encyclopedia + Senior Technical Interview Mastery Guide**  
> Designed for .NET Backend Developers (~3+ Years of Experience) targeting Senior Backend Engineer and Technical Lead roles at **Infosys**, **TCS**, **Wipro**, and top-tier global product organizations.

---

## 📚 Deliverables Overview

This repository serves two distinct purposes:
1. **Lifetime Production Reference Book**: Deep Microsoft documentation rigor + CoreCLR runtime internals + Clean Architecture + mechanical sympathy.
2. **Master Interview Preparation System**: 155+ high-stakes scenario challenges with interviewer traps, expected depth, and follow-up questions.

---

## 📦 What's Inside

### 1. Master Publication PDFs
- **[`./.NET_Developer_Encyclopedia.pdf`](./.NET_Developer_Encyclopedia.pdf)**: **475 Pages (3.36 MB)** — Volumes 0, X, and 2 through 10 covering software engineering fundamentals, .NET platform evolution, C# language internals, GC, concurrency, ASP.NET Core, EF Core, SQL Server, microservices, and system design.
- **[`./Infosys_Interview_Preparation.pdf`](./Infosys_Interview_Preparation.pdf)**: **153 Pages (1.11 MB)** — Volumes 1, 11, and 12 covering DSA thinking patterns, architecture defense scripts, and 155+ master interview questions.

### 2. Interactive Web Portal (`htmls/`)
Open **[`htmls/index.html`](./htmls/index.html)** in any browser for:
- Full syntax highlighting (C#, SQL, JSON, Bash) via Highlight.js.
- Interactive vector flowcharts and sequence diagrams via Mermaid.js.
- Dual navigation sidebars (Module Directory + Floating On-This-Page Table of Contents).
- One-click copy buttons on all production code snippets.
- Real-time client-side module search and filtering.

### 3. Raw Markdown Knowledge Base (`md/`)
Contains all 14 standalone markdown volumes totaling **over 242,000 words** of production engineering content.

---

## 🧭 Curriculum & Volume Directory

| Volume | Module Title | Primary Topics Covered |
|:---:|:---|:---|
| **Vol 0** | [Software Engineering Fundamentals](./md/Volume-0-Software-Engineering-Fundamentals.md) | Problem decomposition, scientific debugging, clean code, SOLID principles, N-Tier vs Clean Architecture. |
| **Vol X** | [.NET Platform & CoreCLR Architecture](./md/Volume-X-DotNet-Platform-Core-Encyclopedia.md) | .NET Framework to .NET 10 evolution, CoreCLR pipeline, RyuJIT Dynamic PGO, Native AOT, Kestrel, DI engine. |
| **Vol 1** | [Logical Thinking & DSA](./md/Volume-1-DSA-Logical-Thinking.md) | 10 thinking patterns, zero-alloc arrays, `Dictionary<TKey,TValue>` bucket hashing, trees, DP, Infosys interview transcript. |
| **Vol 2** | [C# Complete Language Encyclopedia](./md/Volume-2-CSharp-Encyclopedia.md) | Stack vs Heap, Boxing IL, Struct memory layout, Records, Generics variance, LINQ Expression Trees, Pattern Matching. |
| **Vol 3** | [Advanced C# & Memory Management](./md/Volume-3-Advanced-CSharp.md) | Generational GC, LOH/POH, `Span<T>`/`Memory<T>`, `IAsyncStateMachine` lowering, ThreadPool starvation, Channels. |
| **Vol 4** | [OOP & Enterprise Principles](./md/Volume-4-OOP-Design-Principles.md) | Encapsulation vs Hiding, VTable dispatch, Interface vs Abstract Class, Pragmatic SOLID violations, DI lifetimes. |
| **Vol 5** | [Enterprise GoF Design Patterns](./md/Volume-5-Design-Patterns.md) | Singleton thread-safety with `Lazy<T>`, Factory, Strategy, Decorator over inheritance, Mediator with MediatR. |
| **Vol 6** | [ASP.NET Core Complete Architecture](./md/Volume-6-ASPNetCore.md) | Request lifecycle, Middleware Russian-doll pipeline, Captive dependencies, JWT rotation, Output caching, Rate limiting. |
| **Vol 7** | [Entity Framework Core Internals](./md/Volume-7-EFCore.md) | DbContext pooling, Change Tracker states, N+1 detection & generated SQL, Migrations zero-downtime, RowVersion concurrency. |
| **Vol 8** | [Database Engine Internals & SQL Server](./md/Volume-8-Database-SQL.md) | 3-Valued NULL logic, Hash Match vs Nested Loops, B-Tree pages, Seek vs Scan, ACID WAL logging, RCSI, Deadlock graphs. |
| **Vol 9** | [Microservices & Distributed Systems](./md/Volume-9-Microservices.md) | Synchronous gRPC vs Async Kafka/RabbitMQ, CQRS, Event Sourcing, Saga orchestration, Outbox pattern, Polly resilience. |
| **Vol 10** | [Enterprise System Design & Scaling](./md/Volume-10-System-Design.md) | 5-step framework, CAP/PACELC, URL shortener, Notification Hub, 10k TPS Payment system, E-Commerce OMS, Chat system. |
| **Vol 11** | [Project Interview Mastery](./md/Volume-11-Project-Interview-Mastery.md) | Architecture defense scripts ("Why CQRS?", "Why PostgreSQL?"), production outage post-mortems, on-call crisis handling. |
| **Vol 12** | [Master Interview Question Bank](./md/Volume-12-Interview-Question-Bank.md) | 155 Master Questions across 10 categories: Depth, Key Answer Points, The Trap, and Senior Follow-up questions. |

---

## 🏛️ The 12-Section Architecture Rule

Every concept in every volume strictly adheres to a 12-point engineering schema:
1. **Definition** (Precise conceptual framing)
2. **Historical Reason** (Why was this introduced? What problem existed before?)
3. **Problem It Solves** (Concrete production failure modes)
4. **Internal Working** (CLR, RyuJIT, Memory layout, Hardware CPU cache lines)
5. **Real Production Example** (Exclusively Core Banking, Healthcare EHR, SaaS Billing, E-Commerce OMS, Payment Gateways)
6. **Production C# Code Example** (Modern C# 12 / .NET 8, nullable-enabled, zero-allocation idioms)
7. **Execution Flow Diagram** (Detailed state transition maps)
8. **Design Reasoning & Tradeoffs** (Why this approach vs alternative patterns?)
9. **Performance Profile** (CPU, Heap allocation, GC pressure, DB roundtrips)
10. **Common Anti-Patterns** (Beginner and intermediate developer mistakes)
11. **Production Outage Forensics** (Real incident post-mortems with root cause analysis)
12. **Master Interview Preparation** (Basic, 3 YOE expected, Senior follow-up, and Tricky Interviewer Traps)

---

## 🚀 Getting Started

- Open **[`htmls/index.html`](./htmls/index.html)** in Google Chrome, Microsoft Edge, or any modern web browser to navigate the curriculum.
- Refer to **[`Context.md`](./Context.md)** for the 8-week candidate study plan and high-frequency Infosys interview traps.

---
*Authored with Microsoft MVP & Principal .NET Architect rigor.*
