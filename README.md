<div align="center">

# ⚡ Zero Coder

### **Zero Friction. Zero Token Waste. Infinite Context.**
*The open-source AI coding agent built for lightning-fast desktop performance, dynamic multi-provider routing, and zero context loss.*

[![License: MIT](https://img.shields.io/badge/License-MIT-purple.svg)](https://opensource.org/licenses/MIT)
[![Tauri 2.0](https://img.shields.io/badge/Tauri-2.0-blue.svg)](https://tauri.app/)
[![TypeScript](https://img.shields.io/badge/TypeScript-React-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Rust Backend](https://img.shields.io/badge/Rust-Backend-orange?logo=rust)](https://www.rust-lang.org/)

</div>

---

## 🚀 What is Zero Coder?

**Zero Coder** is an elite, open-source desktop AI software engineering agent. Inspired by open-source agent tools like OpenCode, Zero Coder is engineered from the ground up to eliminate common developer pain points: lagging terminal UIs, sudden context-limit crashes, wasted token budgets, and rigid provider lock-in.

Powered by a **Tauri 2.0 (Rust) backend** and a **React + Tailwind CSS dark-minimal UI**, Zero Coder runs buttery-smooth at 60fps while keeping your CPU and RAM usage minimal.

---

## 🔥 Core Architecture & Features

* **⚡ Zero UI Lag:** Built on a native Rust backend that handles all file I/O, AST parsing, and process spawning asynchronously on background threads. Zero blocking, zero frame drops.
* **🧠 Auto-Anchor Context Engine:** Never lose your session to context limits again. When your prompt budget hits **85% capacity**, Zero Coder automatically compresses past chat history into a smart `.zero/anchor.json` state, clears chat bloat, and lets you resume instantly with infinite memory.
* **🔌 Universal Provider Gateway:** Not locked into any single cloud API. Plug in local Ollama instances (e.g., running on an RTX 3060), OpenRouter, Anthropic, or any custom LAN endpoint. Dynamically fetches `/v1/models` on the fly.
* **🌐 Built-In Live Preview & Self-Healing:** Spawns your local dev server natively, captures browser preview and terminal stderr errors in real time, and feeds stack traces right back into the agent loop to self-heal its own code.
* **🎯 AST-First Token Efficiency:** Uses **Tree-sitter** for high-speed symbol mapping and compact **Search/Replace diff patches** instead of rewriting entire files. Saves massive amounts of tokens and money.

---

## 📂 Repository Structure

```text
zero-coder/
├── src-tauri/             # Rust high-performance backend
│   ├── src/
│   │   ├── main.rs        # Tauri entry point & command dispatch
│   │   ├── provider_gateway.rs # Dynamic model fetcher & proxy
│   │   ├── diff_engine.rs # Fast Search/Replace patch applier
│   │   └── context_manager.rs # Token accountant & Auto-Anchor state
└── src/                   # React + TypeScript UI
    ├── store/             # Zustand global state
    ├── components/        # Provider dropdown, Token gauge, Diff viewer, Live Preview
    └── services/          # Strongly typed Tauri IPC bridge
