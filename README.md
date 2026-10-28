# Senior .NET Developer Roadmap & Encyclopedia (2026 Edition)

> **The Complete .NET Developer Encyclopedia + Senior Technical Interview Mastery Guide**  
> Designed for .NET Backend Developers (~3+ Years of Experience) targeting Senior Backend Engineer and Technical Lead roles at **Infosys**, **TCS**, **Wipro**, and top-tier global product organizations.

---

## 🌟 What's Inside

This repository provides an exhaustive, production-grade engineering curriculum across **14 comprehensive volumes** totaling **over 242,000 words** of deep technical content.

### 1. Interactive Web Documentation Portal (`htmls/`)
Open **[`htmls/index.html`](./htmls/index.html)** in Google Chrome, Microsoft Edge, or any modern web browser for:
- 🌓 **Instant Dark Mode & Light Mode**: Persistent theme toggle across all 14 chapters with smooth transitions.
- 🎨 **Syntax Highlighting**: C#, SQL, JSON, and Bash blocks styled with [Highlight.js](https://highlightjs.org/).
- 📋 **One-Click Code Copy**: Built-in clipboard integration on every production code block.
- 📊 **Interactive Flowcharts & Diagrams**: Rendered via [Mermaid.js](https://mermaid.js.org/) for execution lifecycles and architectures.
- 🧭 **Dual-Sidebar Navigation**: Left sidebar for curriculum modules + Right floating Table of Contents for instant section jumping.
- 🔍 **Real-Time Module Search**: Instant keyword filtering directly from the homepage.

### 2. Raw Markdown Knowledge Base (`md/`)
Contains all 14 standalone, portable markdown volumes in `md/` for offline study, Obsidian, or local markdown readers.

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

## 🚀 Quick Start

1. Clone the repository:
   ```bash
   git clone https://github.com/i-m-akshat/Senior-DotNet-Developer-Roadmap.git
   ```
2. Open `htmls/index.html` in your browser to start reading with dark/light mode toggle.
3. Review `Context.md` for the 8-week candidate study schedule and high-frequency Infosys interview traps.

---
*Authored with Microsoft MVP & Principal .NET Architect rigor.*
