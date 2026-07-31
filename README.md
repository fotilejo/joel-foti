# Joel Foti — Portfolio

**Information Systems Specialist | Semantic Data Architecture | Buenos Aires, Argentina**

🌐 [fotilejo.github.io/Joel-Foti](https://fotilejo.github.io/Joel-Foti) · 🚀 Live production system: [biblioteca.inti.gob.ar](https://biblioteca.inti.gob.ar/)

---

## About

I'm a librarian and information systems specialist with a strongly technical profile. I design and build the data infrastructure of the library at **INTI** (Argentina's National Institute of Industrial Technology): four inherited ISIS databases now published as an RDF graph of **145,177 entities and 2.36 million triples**, queryable over SPARQL and exposed as Linked Open Data — plus the tooling that keeps that data clean.

My background is unusual for the tech space: two degrees from the **Universidad de Buenos Aires** — Library & Information Science, and History. That gives me the judgement to decide how a large body of complex information should be structured, classified and retrieved *before* writing the first line of code — information architecture, controlled vocabularies, authority control and semantic modelling as a discipline, not as an afterthought.

I use AI as a delivery multiplier, but my contribution isn't prompting: it's **steering the architecture, auditing what gets generated and catching the errors the model doesn't see** — a Wikidata QID already in use elsewhere, a builder silently discarding data without failing, a URI scheme that looks correct but structurally prevents dereferencing. When I correct 59,605 fields of a live catalogue with AI assistance, what I contribute isn't the speed: it's knowing the database has gaps and must therefore never be reloaded wholesale, and leaving untouched the 377 cases where two different people share a surname.

---

## Featured Projects

### 1. INTI Library Web Platform
**Live:** [biblioteca.inti.gob.ar](https://biblioteca.inti.gob.ar/) | **Stack:** `SPARQL` · `Apache Jena Fuseki` · `Lucene` · `JavaScript` · `nginx`

Complete re-engineering of the institutional library platform, from an undocumented 2000s-era ISIS/WXIS system to a public semantic portal. 30 iterated versions, 186 commits.

* Reverse-engineered the `.xis` scripts and `.fdt` files of each legacy database to map the full search and display logic.
* Rebuilt the frontend as a modular static application: semantic catalogue, browsable thesaurus, technical dictionaries, source guide, saved searches and an interactive guided tour.
* Catalogue queries run on **SPARQL over Apache Jena Fuseki**, with a **Lucene** index and Spanish analyzer for full-text search; the three smaller databases are served from static JSON mirrors for O(1) lookup without hitting the triplestore.
* Federated search hitting catalogue, dictionaries, thesaurus and source guide simultaneously; real-time semantic suggestions drawn from actual graph entities.
* Bibliographic export in **RIS, BibTeX, CSV, APA and ISO**; persistent URLs that reproduce a query with every filter applied.
* Public, documented **SPARQL endpoint** for third-party consumption.

### 2. Data Pipeline — ISIS to RDF and JSON
**Repo:** [Herramienta-Exportacion-Bases](https://github.com/fotilejo/Herramienta-Exportacion-Bases) | **Stack:** `Python` · `rdflib` · `CISIS` · `OWL` · `TDB2`

A single command regenerates the library's **four databases** towards two targets: Turtle for the triplestore and JSON mirrors for the browser — both written **in the same pass from the same source**, so the mirror cannot drift from what the catalogue queries.

| Entity | Count |
|---|---|
| Bibliographic resources | 68,914 |
| — Documents | 49,592 |
| — Serials | 14,023 |
| — Technical standards | 5,299 |
| People / authors | 21,672 |
| Series & collections | 15,786 |
| Organisations | 14,900 |
| Subject concepts | 12,638 |
| Events & conferences | 3,721 |
| **Total entities** | **145,177** |
| **RDF triples (4 databases)** | **2,361,110** |

* Designed a purpose-built **OWL ontology** as an application profile unifying the source databases, reusing **BIBFRAME**, **Dublin Core Terms**, **SKOS**, **schema.org** and **PROV-O** rather than inventing vocabulary.
* **Automated authority control:** deduplication and normalisation of person and institution names so each entity is a unique node instead of a string repeated thousands of times.
* Technical dictionaries enriched with `skos:exactMatch` to **Wikidata and PubChem** — 4,492 equivalences.
* The thesaurus publishes only the QIDs **approved in manual audit**: the pipeline filters by review status rather than dumping everything.
* Documented why hash URIs structurally prevent entity dereferencing, and designed the migration to a hashless instance namespace with a resolver following the **Cool URIs** pattern (303 + content negotiation).

**Bugs found and fixed while auditing the pipeline:**

* 🔴 **Mass mojibake** — the orchestrator wrote the temp CSV as UTF-8 while the builder read it as `windows-1252`: **253,371 corrupted characters** across the catalogue. After the fix, 24 remained (the ones that were like that in the source).
* 🟠 **Silent data loss** — the source-guide builder was still looking for ISIS subfields the extractor no longer emitted, discarding everything without failing: **Zero related links where there should have been 86**. What makes this dangerous is that nothing breaks; the file is produced all the same, just incomplete.
* 🔴 **Crash from `dict.get` semantics** — `.get(key, [])` returns `None`, not the default, when the key exists with a null value. One record with `narrower: null` aborted the whole thesaurus build.
* **Verification, not trust:** TTL subject counts checked against CSV record counts, plus a 690-record comparison between old and new output — 689 identical, the single difference being a real edit made in ISIS.

### 3. INTI Thesaurus — Restructuring & LOD Enrichment
**Stack:** `SKOS` · `Wikidata API` · `Python` · `TemaTres` · `Git` — 323 documented commits

A thesaurus of industrial technology and applied sciences that had grown bottom-up for decades: 1,986 of its terms were orphans, with no broader term placing them in the hierarchy.

| Metric | Value |
|---|---|
| Published concepts | 5,762 |
| Matched to Wikidata (`skos:exactMatch`) | **4,626** |
| Orphans placed in hierarchy | 1,986 |
| Hierarchical relations | 5,759 |
| `skos:altLabel` synonyms recovered | 513 |

* Worked under five rules fixed in advance: endogenous priority, no spurious specificity, no unilateral modification of existing relations, recursive ascent, mandatory semantic mapping.
* Multi-source LOD phase querying **8 external vocabularies in parallel**, with Wikidata as priority source.
* Final phase over 1,266 never-processed terms using 4 search strategies each (ES/EN Wikipedia, entity search in both languages with heuristic singularisation) — 98% returned at least one candidate.
* **Automated anti-duplicate validation:** no term may take a QID already used by another concept or by its own broader/narrower/related terms. Caught and auto-downgraded 52 violations in the final phase.
* Built a **traceable visual audit tool** listing every mapping with its review status. The publishing pipeline emits only the confirmed ones — anything doubtful stays in the working file, not in the public data.
* The unmapped terms are characterised, not ignored: mostly Argentine national bodies absent from Wikidata, plus highly specific or archaic technical terminology.

### 4. Mass Cataloguing Correction over ISIS
**Stack:** `Python` · `CDS/ISIS` · `ABCD` · `ISO 2709` · `openpyxl`

A closed-loop **ISIS → Excel → AI-assisted editing → ISIS** tool, and 18 correction rounds applied to the live catalogue of 49,592 records.

| Metric | Value |
|---|---|
| Corrections applied | **59,605** |
| Distinct records corrected | **47,150** (95%) |
| Verified rounds | 18 |
| Author mentions normalised | 4,201 → 1,088 authorised forms |
| Editable ISIS fields | 53 |

**What was corrected:** document type and subject descriptors (52,245 changes in the largest round); author authority control plus a follow-up sweep for accent and abbreviation variants; propagation of each authorised form into affiliation, contributor and notes fields; language codes normalised to **ISO 639-2**; reference level derived from document type; corrupted role subfields; embedded line breaks; undecoded HTML entities; institutional authors filed under personal author.

**Why it's safe to run on production:**

* **The round trip is proven before anything is edited** — exporting and re-importing with no changes yields zero differences, verified over the full database.
* **Selective update, never a full reload.** The database has 1,876 gaps from deleted records; reloading it whole would shift the internal identifiers and break loan references.
* **Clone first, production second**, with field-by-field and cell-by-cell verification against the live database, plus an automatic backup per round.
* **When the database moves, revalidate:** on finding a round had already been applied, I re-applied the following 460 changes against a fresh export, checking every "before" value prior to writing — all 460 matched without exception.
* **Ambiguity is left alone:** 377 probable homonyms and 50 indistinguishable surnames left untouched and documented in a 16-sheet review workbook. Authority control that merges two different people does more damage than it repairs.
* **Odd engine behaviour is documented, not papered over:** two fields outside the database's field definition table accumulate instead of being replaced. Four strategies tested, none avoids it — recorded as cosmetic and explicitly flagged as *not* to be cleaned in bulk, because in some records that field holds real data.

### 5. BiblioData — 7 Bibliometric Analysis Tools
**Live:** [biblioteca.inti.gob.ar/herramientas.html](https://biblioteca.inti.gob.ar/herramientas.html) | **Stack:** `SPARQL` · `JavaScript` · `Node` · `ECharts` · `ApexCharts`

Once the catalogue is a graph, it stops being a search box and becomes an object of study. Collection Profile (64,224 indexed documents, 46,696 authors, 9,711 of them the institute's own output) · Subject Evolution · Descriptor Co-occurrence · Standards Map · Author Profile · Subject Cloud · Standards-by-Object Finder.

**Architectural decision — snapshot vs. live query:** each tool was firing several aggregation queries against the full catalogue on every load, to compute figures that change once a week. After analysing each tool I found the user's parameter space was 3–10 fixed combinations, not free text — enumerable. The rule I set: **aggregates are precomputed into JSON snapshots, documents are always queried live.** Five of the seven tools read their aggregates from snapshots; the other two are point lookups rather than aggregates and stay fully live. The snapshots are generated with Node reading the catalogue's Turtle directly, with no need for the triplestore to be running, parsing the file once for all five.

Every tool carries an explicit scope notice: BiblioData reflects what has been catalogued, not the institute's total output. A chart without that caveat reads as if it were a census.

### 6. Infrastructure, Security & Observability
**Stack:** `Linux` · `nginx` · `Apache Jena Fuseki` · `Bash` · `Python` · `SSH/SCP`

* Full service migration from a Windows/Apache environment to **Linux + nginx**, with a migration manual so the systems team could reproduce it.
* Complete security headers (explicit CSP, `X-Frame-Options`, `nosniff`, `Referrer-Policy`, `Permissions-Policy`), sensitive-path blocking, and **rate limiting on the SPARQL endpoint**.
* Tiered caching: immutable for versioned static assets, forced revalidation for CSS/JS.
* Automated one-way deployment over SSH, documented for information-security review.
* **In-house usage analytics without third-party trackers:** a scheduled pipeline consolidating rotated server logs, producing a technical report plus a custom Python executive report (HTML, locally served charts) and a lightweight monthly JSON for year-over-year trends. Bot/human and mobile/desktop separation; static assets and technical endpoints excluded so page rankings reflect real content. All log-derived values HTML-escaped; temp files randomised with guaranteed cleanup.

**Security audit of my own code** — I audited the seven BiblioData tools and their integration points. Real findings, all fixed: a **reflected XSS exploitable via a shared link** (the search term was inserted unescaped into the "no results" message, and that term travels inside the URL the Share button generates); **insufficient attribute escaping** (HTML-entity escaping isn't enough inside an `onclick`, since the browser decodes the attribute before handing it to the JS parser — this needed a separate escape function); **wrong escaping order in SPARQL literals** (the quote escaped without first escaping the backslash, allowing early literal termination); and **formula injection in CSV exports**. Two findings were marked *deliberately not fixed*, with the reasoning written down: removing `'unsafe-inline'` from the CSP would require rewriting every inline handler or generating per-request nonces, impossible on a static site; and hardening the query endpoint requires production server access. Plus **SEO and accessibility** audits of the whole site.

---

## Skills & Knowledge Areas

| Area | Details |
|---|---|
| Semantic Web & LOD | RDF, OWL, SKOS, SPARQL, BIBFRAME, Dublin Core, schema.org, PROV-O, Wikidata & PubChem mapping, knowledge graphs |
| Data Engineering | Python, rdflib, Pandas, Node, ETL pipelines, automated authority control, entity deduplication, integrity auditing |
| Data Quality | Mass correction of legacy catalogues, ISO 2709 round-trip, verification methodology, ambiguity handling |
| Databases & Backend | Apache Jena Fuseki, TDB2, Lucene, CDS/ISIS & CISIS, SQL/MySQL |
| Web Development | HTML5, CSS3, JavaScript, JSON APIs, responsive & accessible UI, technical SEO, web security |
| Infrastructure | Linux, nginx administration, hardening, caching strategy, deployment automation, log analytics |
| Library Systems | ISIS/WXIS, ABCD, Koha, ExLibris Alma, Greenstone 3, DSpace, TemaTres |
| Metadata & Cataloguing | RDA, MARC21, AACR2, FOCAD, UDC classification, controlled vocabularies, thesaurus construction |
| AI-assisted Engineering | Architecture steering, code auditing, human-in-the-loop verification at scale |
| Languages | Spanish (native) · English B2 (Laboratorio de Idiomas, UBA) |

---

## Education

- **Licenciatura en Ciencia de la Información** — UBA *(in progress)*
- **Profesorado de Enseñanza Media y Superior en Bibliotecología y Ciencia de la Información** — UBA
- **Profesorado de Enseñanza Media y Superior en Historia** — UBA

**Certifications:** English B2 (Laboratorio de Idiomas, UBA) · SQL & MySQL Databases · Introduction to Python

---

## Other Repositories

- [Biblioteca-INTI-ISIS](https://github.com/fotilejo/Biblioteca-INTI-ISIS) — new website development for the INTI Library; database modernisation.
- [Herramienta-Exportacion-Bases](https://github.com/fotilejo/Herramienta-Exportacion-Bases) — extracts the source databases and transforms them into TTL and JSON ready for publication.
- [Semantic-Entity-Extractor-Graph-Modeler](https://github.com/fotilejo/Semantic-Entity-Extractor-Graph-Modeler) — Python engine turning flat bibliographic data into knowledge-graph topologies with automated authority control.
- [Data-Normalizer-ETL](https://github.com/fotilejo/Data-Normalizer-ETL) — ETL pipeline converting legacy bibliographic CSV exports into structured JSON.
- [Interfaz-INTI-Greenstone3](https://github.com/fotilejo/Interfaz-INTI-Greenstone3) — interface work for the institutional repository.

---

## Contact

- 📧 fotilejo@gmail.com
- 💼 [LinkedIn](https://www.linkedin.com/in/joel-foti)
- 🐙 [GitHub](https://github.com/fotilejo)
