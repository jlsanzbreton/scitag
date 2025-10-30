Walker aquí. Te dejo el entregable en **Markdown** listo para pegar en tu repo (`docs/Geogaceta-Formatter.md`). Incluye flujo funcional, diagrama de arquitectura, esquema de datos, validadores “duros” para Geogaceta (según lo que definiste: **Resumen/Abstract a 2 columnas** y resto **en 3 columnas**), rúbrica de *Scientific Fit Assistant*, endpoints mínimos, prompts de IA, y plan de sprints.

---

# Geogaceta Formatter — Functional & Technical Blueprint

**Autor:** Jose (product owner) — *Walker* (arquitectura & UX)
**Versión:** 0.9 (MVP orientado a envío real)
**Objetivo:** Reducir el tiempo y la fricción para entregar un manuscrito **lista-para-enviar** a **Geogaceta**, combinando **formateo automático** + **Asistente de Adecuación Científica** (*Scientific Fit Assistant*, SFA).

---

## 1) Premisas clave (Geogaceta)

* **Maquetación:**

  * **Resumen** (ES) y **Abstract** (EN) en **2 columnas** (una por columna).
  * A partir de ahí, **todo el artículo en 3 columnas**.
* **Contenidos básicos:** Título, autores + afiliaciones, palabras clave/keywords, secciones (Introducción, Área de estudio/Material y métodos, Resultados, Discusión, Conclusiones, Agradecimientos, Referencias), figuras, tablas, material suplementario.
* **Figuras:** con **escala** y **norte** (si aplica), **pie de figura** y **citadas en el texto**.
* **Referencias:** estilo bibliográfico consistente; se exige que **todas las citas en texto** existan en bibliografía y viceversa.

> *Nota:* Estas reglas se toman de los requisitos que nos has indicado y de prácticas habituales de revistas geológicas de ámbito hispano. Ajustaremos tras validar contra las guías editoriales públicas de Geogaceta en la siguiente iteración.

---

## 2) Propuesta de valor — Elemento diferenciador

**Scientific Fit Assistant (SFA):** un motor de evaluación y coaching editorial que mide el **SGA (Score de Adecuación Geogaceta)** y ofrece **recomendaciones accionables** por sección (estructura, tono, figuras, referencias), antes de formatear.

* **Salida clave del SFA:**

  * SGA de 0-100
  * Lista priorizada de *issues* (bloqueantes vs. recomendables)
  * Botón “*Arreglar automáticamente*” para micro-ediciones (con diffs visibles)
  * Indicadores de “Listo para enviar” (≥90)

---

## 3) Flujo funcional (end-to-end)

```mermaid
flowchart LR
  A[Inicio] --> B[Crear proyecto / Importar .docx/.md/.tex]
  B --> C[Ingesta & Parsing]
  C --> D[Detección de estructura (secciones, citas, figuras)]
  D --> E[SFA: Análisis semántico + Reglas Geogaceta]
  E --> F{Bloqueantes?}
  F -- Sí --> G[Checklist guiada + Fix aut. opcional]
  G --> E
  F -- No --> H[Formateo plantilla Geogaceta]
  H --> I[Previsualización (2+3 col.)]
  I --> J{OK autor?}
  J -- Ajustes --> E
  J -- Sí --> K[Export: PDF + ZIP (fuentes+figs) + Carta de envío]
  K --> L[Resumen de cumplimiento + SGA final]
  L --> M[Fin]
```

**Eventos destacables:**

* *Autosave* y versionado por cambios.
* Validación dura de correspondencia **citas↔bibliografía**.
* Verificación de **figuras citadas** y **resolución mínima**.

---

## 4) SGA — Rúbrica de Adecuación Geogaceta

| Dimensión                        | Peso | Criterios                                                    | Bloqueante               |
| -------------------------------- | ---: | ------------------------------------------------------------ | ------------------------ |
| Estructura y secciones           |  20% | Presencia y orden de secciones mínimas; títulos normalizados | Sí                       |
| Resumen/Abstract (2 col.)        |  10% | Longitud esperada, claridad, keywords                        | Sí si ausente            |
| Maquetación 3 columnas           |  10% | Cuerpo tras resúmenes en 3 columnas                          | Sí                       |
| Figuras y tablas                 |  15% | Citadas en texto; pies completos; escala/norte; resolución   | Sí si falta citación/pie |
| Referencias (texto↔Biblio)       |  20% | Citas existentes; estilo consistente; DOIs si aplica         | Sí                       |
| Lenguaje/tono                    |  10% | Claridad, formalidad, terminología geológica                 | No (recomendable)        |
| Metadatos (autores/afiliaciones) |   5% | Estructura y correspondencia                                 | Sí si incompleto         |
| Length & ratios                  |   5% | Rango de palabras/figuras razonable                          | No                       |
| Gráficos/mapas                   |   5% | Legibilidad, simbología                                      | No                       |

**Umbrales recomendados:**

* 90–100 = *Ready to Submit*
* 75–89 = *Near Ready* (arreglar recomendados)
* <75 = *Work Needed* (arreglar bloqueantes)

---

## 5) Validadores “duros” (MVP)

* **V-1. Secciones mínimas:** Regex/AST para `Resumen`, `Abstract`, `Introducción`, `...`, `Referencias`.
* **V-2. Doble columna Resumen/Abstract**: estructura obligatoria antes del cuerpo.
* **V-3. Tres columnas en cuerpo**: comprobación de grid en motor de render.
* **V-4. Citas en texto ↔ entradas bibliográficas**: 100% de correspondencia.
* **V-5. Figuras/tablas citadas**: cada `Figura n` debe estar citada y con `pie`.
* **V-6. Resolución de imágenes**: ≥300 dpi para impresión; tamaños consistentes.
* **V-7. Autores/afiliaciones**: todos presentes y con índice/superscripts coherentes.

---

## 6) Modelo de datos (resumen)

```ts
type Manuscript = {
  id: string;
  title: string;
  authors: Author[];
  affiliations: Affiliation[];
  keywordsES: string[];
  keywordsEN: string[];
  abstractES: string;
  abstractEN: string;
  sections: Section[];      // orden lógico
  figures: Figure[];
  tables: Table[];
  citations: InTextCitation[];
  references: ReferenceEntry[]; // CSL-JSON recomendado
  assets: Asset[];          // imgs, zip, supl.
  journalTarget: "Geogaceta";
  sga?: SGAReport;
  versions: VersionRef[];
  createdAt: string;
  updatedAt: string;
};

type SGAReport = {
  score: number;
  blocks: Issue[];      // bloqueantes
  recs: Issue[];        // recomendaciones
  metrics: Record<string, number>;
};
```

**Referencias en formato CSL-JSON** (ideal para compatibilidad con *citeproc-js* y con estilos de **Zotero CSL**).

* Repositorio de estilos CSL: [https://www.zotero.org/styles](https://www.zotero.org/styles)
* Especificación CSL: [https://docs.citationstyles.org/en/stable/specification.html](https://docs.citationstyles.org/en/stable/specification.html)

---

## 7) Arquitectura técnica (MVP)

```mermaid
graph TD
  UI[Web App (PWA)] --> API
  subgraph Frontend
    UI --> Preview[Previsualizador (PDF/HTML 2+3 col.)]
  end

  subgraph Backend
    API[API BFF] --> Parse[Ingesta & Parsing (.docx/.md/.tex)]
    API --> SFA[Scientific Fit Assistant (LLM + reglas)]
    API --> TPL[Motor de Plantillas Geogaceta]
    API --> BIB[Normalizador de referencias (CSL/citeproc)]
    API --> IMG[Validador de imágenes (dpi/size)]
    API --> EXP[Exportador (PDF/ZIP)]
    API --> Store[Storage (DB + objetos)]
  end

  subgraph Infra
    Store --> CDN[(CDN/Obj Storage)]
  end
```

**Stack sugerido (rápido y mantenible):**

* **Frontend:** React + Vite + Tailwind, PWA offline-first, visor PDF (pdf.js).
* **Parsing:** `docx` → `pandoc` pipeline o `docx-parser` + heurística; `md/tex` con `remark`/`latex.js`.
* **Bibliografía:** `citeproc-js` + estilos CSL.
* **Plantillas & Export:**

  * Opción A (HTML→PDF): `paged.js` o `Playwright`/`Chromium` headless con CSS multicol (`column-count`).
  * Opción B (LaTeX): clase `.cls` Geogaceta y `xelatex` en contenedor.
* **IA (SFA):** LLM con *prompts* especializados + reglas deterministas (validadores).
* **Storage:** Supabase (Postgres + Storage) o equivalente.

**Referencias técnicas útiles:**

* Pandoc: [https://pandoc.org/](https://pandoc.org/)
* Paged.js (CSS→PDF): [https://pagedjs.org/](https://pagedjs.org/)
* pdf.js visor: [https://mozilla.github.io/pdf.js/](https://mozilla.github.io/pdf.js/)
* Playwright: [https://playwright.dev/](https://playwright.dev/)

---

## 8) Endpoints mínimos (BFF)

```
POST   /ingest           // sube .docx/.md/.tex + assets
POST   /parse            // devuelve Manuscript base
POST   /sfa/analyze      // SGA + issues + fixes sugeridos
POST   /sfa/apply-fixes  // aplica micro-ediciones (con diff)
POST   /format/geogaceta // compone HTML/LaTeX con 2+3 col.
GET    /preview/:id      // HTML/PDF preliminar
POST   /export           // PDF final + ZIP fuentes + carta
GET    /versions/:id     // historial
```

---

## 9) Reglas de maquetación (CSS/HTML) — MVP

```css
/* Bloque doble columna solo para Resumen/Abstract */
.section--abstracts {
  column-count: 2;
  column-gap: 18pt;
}

/* Cuerpo principal en tres columnas */
.article-body {
  column-count: 3;
  column-gap: 16pt;
}

/* Pies de figura y tablas */
.figcaption, .table-caption {
  font-size: 9pt;
  line-height: 1.2;
}

/* Forzar que pies no queden solos en otra columna */
.figcaption, .table-caption { break-inside: avoid; }

/* Evitar huérfanas/viudas (si el motor lo soporta) */
p { widows: 2; orphans: 2; }
```

---

## 10) Validaciones semánticas (pseudo-código)

```ts
// V-4: Correspondencia citas ↔ bibliografía
for (const c of manuscript.citations) {
  assert(exists(manuscript.references, r => r.key === c.key),
         `Cita ${c.key} no tiene entrada en bibliografía`);
}
for (const r of manuscript.references) {
  assert(exists(manuscript.citations, c => c.key === r.key),
         `Referencia ${r.key} no está citada en el texto`);
}

// V-5: Figuras citadas & pies
for (const fig of manuscript.figures) {
  assert(has(fig.caption), `Figura ${fig.number} sin pie`);
  assert(textIncludes(manuscript.sections, `Figura ${fig.number}`),
         `Figura ${fig.number} no está citada en el texto`);
}

// V-6: DPI imágenes
for (const img of manuscript.assets) {
  if (isRaster(img)) {
    assert(img.dpi >= 300, `${img.name}: resolución < 300 dpi`);
  }
}
```

---

## 11) Prompts IA (SFA) — extractos

**A) Detección de estructura (ES/EN)**

> “Analiza este manuscrito y devuelve JSON con: secciones (título, start-end), presencia de Resumen y Abstract, palabras clave, correspondencia de citas y referencias. Si faltan secciones mínimas para Geogaceta, márcalas con `blocking:true`.”

**B) Reescritura guiada de Resumen/Abstract**

> “Reescribe el Resumen/Abstract para geociencias: 150–250 palabras, claridad, objetivo-métodos-resultados-conclusión; conserva términos técnicos y citas numéricas si existen. Devuelve *diff* markdown.”

**C) Terminología y tono**

> “Evalúa el tono científico y terminología (estratigrafía/litología/tectónica). Propón 5 mejoras máximas y no cambies el significado. Devuelve *suggestions[]* con justificación breve.”

---

## 12) Carta de envío (auto-generada)

* Metadatos del artículo + declaración de originalidad + ajuste a normas.
* Check de co-autores y conflictos de interés.
* Texto editable por el autor.
* Export en PDF junto con el manuscrito.

---

## 13) Métricas de producto

* Tasa de *Ready to Submit* en ≤2 iteraciones.
* Nº de bloqueantes por manuscrito (descendente con uso).
* Tiempo medio desde ingesta → exportación.
* % de errores de referencias y figuras (tender a 0).
* NPS del autor.

---

## 14) Seguridad, privacidad y trazabilidad

* Procesamiento de archivos en contenedores aislados.
* Logs de validación (no contenido sensible) y *checksums* de assets.
* Opción **100% local** (CLI/desktop) para instituciones con privacidad estricta (futuro).

---

## 15) Roadmap por sprints (4–6 semanas MVP)

**Sprint 0 — Infra & parsing**

* Estructura repo (frontend, BFF, workers).
* Ingesta + parsing .docx/.md/.tex (baseline).
* Esquema `Manuscript` + persistencia.

**Sprint 1 — SFA (núcleo)**

* Validadores V-1..V-7.
* SGA v1 + panel de issues.
* Diff editor para fixes.

**Sprint 2 — Maquetación & Export**

* Plantilla Geogaceta (HTML→PDF o LaTeX).
* Implementar doble/tri columna + pies.
* Exportador PDF + ZIP + carta.

**Sprint 3 — UX finas y pruebas**

* Previsualización fluida, autosave, versionado.
* Tests e2e (manuscrito de ejemplo) + *golden PDF*.
* Documentación de envío.

---

## 16) Pruebas de aceptación (MVP)

* [ ] Subo `.docx` con Resumen/Abstract → *detecta 2 col.*
* [ ] Cuerpo en 3 col. con saltos correctos de figuras/tablas.
* [ ] 100% citas↔bibliografía corresponden; reporte si no.
* [ ] Exporta PDF + ZIP + carta sin warnings bloqueantes.
* [ ] SGA ≥ 90 tras aplicar fixes sugeridos.

---

## 17) Riesgos y mitigación

* **Variabilidad DOCX** → *Mitigación:* pipeline `pandoc` + reglas propias y tests con corpus.
* **Cambios de guía editorial** → *Mitigación:* “Plantillas versionadas” + flags de caducidad.
* **Imágenes heterogéneas** → *Mitigación:* normalizador (DPI, dimensiones) con reportes claros.
* **Sesgos IA** → *Mitigación:* SFA combina reglas deterministas + IA “no intrusiva” (siempre con diff).

---

## 18) Extensibilidad (multi-revista)

* Abstraer reglas en `journalProfiles/geogaceta.json`.
* Añadir `journalProfiles/<otra_revista>.json` con: columnas, límites, secciones, estilo CSL y plantillas.

---

## 19) Estructura de carpetas (propuesta)

```
/app
  /frontend
  /server
  /workers
  /templates
    /geogaceta
      layout.html
      layout.css
      letter.md
  /journalProfiles
    geogaceta.json
  /docs
    Geogaceta-Formatter.md   <-- este archivo
  /tests
    fixtures/
    e2e/
```

---

## 20) Licencias y dependencias (referencias)

* **CSL styles (Zotero):** [https://www.zotero.org/styles](https://www.zotero.org/styles)
* **Pandoc (GPL):** [https://pandoc.org/](https://pandoc.org/)
* **Paged.js (MPL-2.0):** [https://pagedjs.org/](https://pagedjs.org/)
* **Playwright (Apache-2.0):** [https://playwright.dev/](https://playwright.dev/)
* **pdf.js (Apache-2.0):** [https://mozilla.github.io/pdf.js/](https://mozilla.github.io/pdf.js/)

---

### Conclusión

El *moat* real es el **SFA**: validar “adecuación editorial” con **bloqueantes objetivos**, ofrecer **fixes con diff**, y exportar **ya listo para enviar** (2+3 columnas, referencias perfectas y figuras conformes). Eso ahorra horas y rechazos.

---

### Nivel de confianza

**Alto.** El diseño es sólido y técnicamente factible para un MVP en 4–6 semanas con foco. Riesgos principales: parsing de `.docx` complejos y variabilidad de figuras — mitigables con pipelines probados y validadores estrictos.

**Fuentes (tecnológicas) citadas:**

* CSL Styles (Zotero): [https://www.zotero.org/styles](https://www.zotero.org/styles)
* CSL Spec: [https://docs.citationstyles.org/en/stable/specification.html](https://docs.citationstyles.org/en/stable/specification.html)
* Pandoc: [https://pandoc.org/](https://pandoc.org/)
* Paged.js: [https://pagedjs.org/](https://pagedjs.org/)
* Playwright: [https://playwright.dev/](https://playwright.dev/)
* pdf.js: [https://mozilla.github.io/pdf.js/](https://mozilla.github.io/pdf.js/)

> Si quieres, preparo también el **perfil `journalProfiles/geogaceta.json`** inicial con límites de palabras, nombres de secciones y flags de validación, y un **layout.css** base con los estilos de 2 y 3 columnas.
