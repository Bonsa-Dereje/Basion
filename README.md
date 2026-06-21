# Basion

> Internal tool for managing recurring client projects — with automated follow-up texting and in-depth social media analytics.

Built with **Tauri** + **SvelteKit** for a fast, native-feeling desktop app.

<p>
  <img src="https://img.shields.io/badge/Tauri-24C8DB?style=for-the-badge&logo=tauri&logoColor=white" />
  <img src="https://img.shields.io/badge/SvelteKit-FF3E00?style=for-the-badge&logo=svelte&logoColor=white" />
  <img src="https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white" />
  <img src="https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white" />
  <img src="https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white" />
</p>

---

## Features

- **Client project tracking** — manage recurring projects in one place
- **Automated follow-up texting** — never let a client go cold
- **Social media analytics** — in-depth insights on client pages
- **Cross-platform desktop app** — powered by Tauri (Rust + Web)
- **Fast & lightweight** — SvelteKit frontend, Vite build pipeline

---

## Tech Stack

| Layer | Tech |
|---|---|
| Frontend | SvelteKit |
| Desktop Shell | Tauri (Rust) |
| Build Tool | Vite |
| Styling | CSS |

---

## Getting Started

### Prerequisites
- [Node.js](https://nodejs.org/)
- [Rust](https://www.rust-lang.org/tools/install)
- [Tauri prerequisites](https://tauri.app/start/prerequisites/) for your OS

### Install

```bash
git clone https://github.com/Bonsa-Dereje/Basion.git
cd Basion
npm install
```

### Run in dev mode

```bash
npm run tauri dev
```

### Build for production

```bash
npm run tauri build
```

---

## Project Structure

```
Basion/
├── src/           # SvelteKit frontend
├── src-tauri/     # Tauri (Rust) backend
├── designLang/    # Design system / language
├── forAgents/      # Agent-related resources
└── static/        # Static assets
```

---

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Svelte](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode) + [Tauri](https://marketplace.visualstudio.com/items?itemName=tauri-apps.tauri-vscode) + [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer)

---

## Status

This is an **internal tool** — built for managing client work, not for public/general use.
