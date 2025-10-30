Perfecto, Jose ⚙️
Aquí tienes el **`README.md` inicial** para el nuevo repositorio `SciTag`.
Está diseñado para que puedas usarlo como *readme de lanzamiento* (MVP con visión, propósito, comandos y hoja de ruta).
Incluye badges, tagline y estructura base adaptada a una PWA o CLI, según evolucione.

---

````markdown
# 🧠 SciTag — The Smart Journal Formatter

**Version:** 0.1.0  
**Author:** José L. Sanz Bretón  
**License:** MIT  
**Status:** 🚧 MVP in development  

---

## 🌍 Overview

**SciTag** is an intelligent formatting assistant designed for scientific authors.  
It automatically **adapts, structures and validates manuscripts** according to the editorial standards of each target journal — starting with **Geogaceta**, a leading Spanish publication in Earth Sciences.

Unlike standard formatters, **SciTag** goes beyond LaTeX or Word templates.  
It analyzes the scientific adequacy, semantic structure, figures, tables, references and tone — guiding the author to meet *review-ready quality*.

---

## ✨ Key Features

- **AI-powered manuscript analysis:**  
  Detects missing sections, incorrect ordering, and misaligned styles.
  
- **Journal-specific templates:**  
  Automatically applies Geogaceta’s official layout (2-column structure, figure captions, references, etc.).

- **Scientific Fit Score (SFS):**  
  Quantifies how close the article is to the journal’s editorial expectations (0–100).

- **Smart corrections & feedback:**  
  Suggests wording, structural and figure improvements based on recent Geogaceta papers.

- **One-click export:**  
  Generates the final PDF and submission package (article, figures, and cover letter).

---

## 🧩 Tech Stack (provisional)

| Layer | Tool |
|-------|------|
| Frontend / UI | React + Vite |
| Backend | Supabase / Node |
| AI Layer | OpenAI / Gemini API (fine-tuned corpus Geogaceta) |
| Formatter | Pandoc + LaTeX templates |
| Storage | Supabase Buckets / Local Dexie (offline-first mode) |

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/jlsanzbreton/SciTag.git
cd SciTag

# Install dependencies
npm install

# Run the app (development mode)
npm run dev

# Build for production
npm run build
````

---

## 🧱 Project Structure

```
SciTag/
 ├── src/
 │   ├── components/
 │   ├── pages/
 │   ├── lib/
 │   ├── styles/
 │   ├── assets/
 │   └── main.tsx
 ├── public/
 │   └── templates/
 │       └── geogaceta.latex
 ├── docs/
 │   ├── ROADMAP.md
 │   ├── FORMAT_RULES.md
 │   └── DATASET_PLAN.md
 ├── package.json
 ├── vite.config.ts
 └── README.md
```

---

## 🧭 Roadmap (v0 → v1.0)

| Milestone | Description                                                   | Status         |
| --------- | ------------------------------------------------------------- | -------------- |
| **v0.1**  | Basic LaTeX → PDF conversion with Geogaceta template          | 🟡 in progress |
| **v0.2**  | Section detection + initial AI prompts (structure & abstract) | 🔜             |
| **v0.3**  | Scientific Fit Score + feedback dashboard                     | 🔜             |
| **v0.4**  | Image handler + figure layout verification                    | 🔜             |
| **v0.5**  | Reference parser (BibTeX, EndNote, plain text)                | 🔜             |
| **v0.6**  | Offline mode + local draft autosave                           | 🔜             |
| **v1.0**  | Full submission-ready workflow for Geogaceta                  | ⏳              |

---

## 🧠 Vision

> *“Science should be judged by clarity and structure, not by formatting struggles.”*

**SciTag** aims to democratize the scientific publication process, giving every author —experienced or novice— a clear path to journal readiness.

---

## 🤝 Contributing

Pull requests, issue reports, and journal integration proposals are welcome.
If you want to help train SciTag for your favorite journal, open a discussion or submit a `.pdf` corpus to `data/training/`.

---

## 🧾 License

This project is licensed under the [MIT License](./LICENSE).

---

## 🛰️ Related Projects

* [SciForm](https://github.com/jlsanzbreton/sciform) — multi-journal formatter prototype
* [ClaimBuddy](https://github.com/jlsanzbreton/claimbuddy) — AI-assisted claims inspection app
* [Gray Zone Mapper](https://grayzonemapper.com) — cognitive blind-spot mapping tool

---

> © 2025 José L. Sanz Bretón — Madrid, Spain
> Built with 🧠, curiosity, and perseverance.

```
