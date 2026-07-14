# Metabolomics domain page — research findings and editorial decisions

Working document supporting the drafting of `pages/your_domain/metabolomics.md`.
Not site content — do not include in the PR (or add to `.gitignore`).

Date of research: 2026-07-14.

---

## 1. Sources used

| Source | What it gave us | Reliability |
|---|---|---|
| The RDMkit repo itself (21 domain pages, `pages/contribute/*`, `_data/*`, `var/tools_validator.py`, `_config.yml`) | Structural conventions, frontmatter schema, tool-table mechanics, style rules, page sizing | Authoritative |
| [RDA-MOMSI Dashboard database](https://github.com/RDA-MOMSI/Dashboard/raw/refs/heads/main/src/data/database.json) (267 records: 54 metabolomics, 47 proteomics, 97 genomics, 69 universal) | Standards, reporting guidelines, ontologies, formats, identifier schemes, governance bodies | Good for standards; **has no repository records at all** |
| Web research on lipidomics + glycomics RDM | Subdomain scoping decision | Verified with URLs |
| Direct URL verification (`curl`) of every resource cited on the page | Link liveness | Authoritative |

**Gap:** a planned agent covering "current status of individual repositories/tools + human-metabolomics re-identifiability" failed with an API error. Consequently:

- Every URL on the page was instead verified by hand (see §6).
- **No claim about re-identifiability of human metabolic profiles is made on the page.** There is literature suggesting metabolic profiles can be individually identifying, but it was not verified in this round. The page instead makes the uncontroversial point that human metabolomics data derives from human samples and is often linked to health data, and defers to the existing `sensitive` / `gdpr_compliance` / `human_data` pages. **If a contributor can verify the re-identifiability literature, that claim is worth adding.**

---

## 2. RDMkit conventions the page must obey

Derived from a survey of all 21 existing domain pages.

- **Section structure:** `## Section` → `### Description` / `### Considerations` (optional) / `### Solutions`. 18 of 21 pages follow this without deviation.
- **Sizing:** median domain page = **2,172 words**, **5 `##` sections**, **~25 `{% tool %}` mentions**. Range 616–4,191 words. Oversized starts at 7–8 sections; undersized at 2–3.
- **No markdown tables.** Zero of 21 domain pages contain one.
- **The "Tools and resources on this page" table is auto-generated** by the remote theme (`ELIXIR-Belgium/elixir-toolkit-theme`) from `{% tool "id" %}` mentions. It is never hand-written. A hand-written tools table is the most common newcomer error.
- **Frontmatter — human-writable keys only:** `title`, `description` (lowercase first letter), `contributors`, `page_id`, `related_pages` (only `Your_tasks` and `Tool_assembly` are permitted on a domain page), `training`, `affiliations`.
  - `editors:` is filled in **by an editor**, not the contributor.
  - `search_exclude: true` must be **deleted** before merge.
  - `fairsharing:`, `dsw:`, `faircookbook:` are **bot-maintained** — never hand-write them.
- **Style guide:** British spelling, `-ise` not `-ize`; "data" is **singular**; headings in sentence case; spell out acronyms on first use; spell out numbers one to ten; no ampersands; descriptive link text.
- **Cross-page links:** internal links use the bare page_id, e.g. `[proteomics](proteomics)`.
- **Tool registry** (`_data/tool_and_resource_list.yml`): `id` (kebab-case), `name`, `url` (must start with `https://`), `description` (avoid `"` and `'`), optional `registry` (`biotools` / `fairsharing` / `tess`). Validated in CI by `var/tools_validator.py`. A weekly bot injects registry IDs.
- **Not tools/resources:** publications, policies, and "webpages of groups or consortia" (`pages/contribute/tool_resource_update.md`). Publications go in the bibliography instead.
- **Editor checklist extras:** contributors added to `_data/CONTRIBUTORS.yaml`; page linked in `_data/sidebars/data_management.yml`; news item added to `_data/news.yml`.

---

## 3. Structural decision

The originally proposed structure was sound and correctly sized, but had three problems:

1. A hand-written tools table (must be deleted — auto-generated).
2. Sharing placed **before** analysis, breaking the data life cycle order used by every comparable page.
3. "Metadata and standards" and "Data formats and interoperability" overlapped heavily (proteomics merges these into one section).
4. **No coverage of quality control** — the single most metabolomics-distinctive RDM topic.

### Adopted structure

| # | Section | Rationale |
|---|---|---|
| 1 | Introduction | Scope, challenges, and what is deliberately out of scope |
| 2 | Metadata, standards and data formats | Merge of the original sections 2 and 3 |
| 3 | Study design and quality control | **New.** The biggest gap in the original draft |
| 4 | Metabolite identification and annotation | Kept, but must add confidence levels |
| 5 | Data processing and analysis | Moved **before** sharing |
| 6 | Sharing, preserving and reusing data | Moved **after** analysis |
| 7 | Lipidomics | **New.** See §4 |

Seven `##` sections, ~2,600 words — comparable to `enzymology_and_biocatalysis` (6 sections, 3,014 words) and `cancer_data` (6, 3,683).

### Why "study design and quality control" is essential

Metabolomics is unusual among the omics in that **data quality is a designed property of the acquisition sequence**, not something recoverable afterwards. Pooled QC samples, blanks, internal standards, randomised injection order, batch structure and drift correction all have to be planned *before* acquisition — and, crucially for RDM, **recorded as metadata** so a reuser can tell which injections are QCs and which batch each sample belongs to. A deposited study that omits this is effectively not reusable. No other RDMkit page covers it, and it is where `mzQC` belongs.

---

## 4. Subdomain scoping decisions

### Lipidomics → **section inside the metabolomics page**

Decisive fact: **lipidomics has no repository of its own.** It deposits to MetaboLights and Metabolomics Workbench, uses mzML in and mzTab-M out (mzTab-M is explicitly scoped to "metabolomics *and lipidomics*"), and runs on the same tool stack. Splitting it would create two half-pages that each repeat the metabolomics life cycle.

But it has four things a generic metabolomics page cannot say, and those form the section:

1. **The structural-detail hierarchy.** `PC 34:1` (species) → `PC 16:0_18:1` (molecular species, *sn* unassigned) → `PC 16:0/18:1` (*sn* assigned) → `PC 16:0/18:1(9Z)` (double-bond position). A lipid name **encodes how much the assay actually resolved** — it is a provenance problem dressed as a naming problem, with no metabolomics analogue. Anchor: Liebisch et al. 2020, *J Lipid Res* 61:1539–1555, DOI `10.1194/jlr.S120001025`.
2. **Goslin** — parses and normalises lipid names across grammars (Shorthand2020, LIPID MAPS, SwissLipids, HMDB) and returns the structural level. Unusually mature tooling. Maintained by lifs-tools.
3. **The Lipidomics Minimal Reporting Checklist** — lipid-specific, not MSI-derived; web wizard with Zenodo DOI minting. Kopczynski et al. 2024, *J Lipid Res* 65:100621.
4. **LSI good practice and QC** — living guidelines, plus NIST SRM 1950 (31-lab ring trial, consensus values for 339 lipids) and SPLASH LIPIDOMIX internal standards.

### Glycomics → **its own page**

A **seed draft now exists** at `pages/your_domain/glycomics.md`. It was written from desk research, has **no domain author**, and carries inline reviewer notes plus a list of known gaps at the foot of the file (quality control, quantification, and glycan/lectin microarray data management are all uncovered). It must not be merged without a domain expert.

Glycomics is the only MS sibling with a **complete independent RDM stack**:

| Criterion | Glycomics | Lipidomics |
|---|---|---|
| Own minimum-information standard | **MIRAGE** (9 modules; MS module updated Nov 2025 to cover glycoproteomics) | Lipidomics Minimal Reporting Checklist |
| Own **raw-data repository** | **GlycoPOST** (~3.5 TB, 357 projects) + UniCarb-DR + GlyComb | **None** — uses metabolomics repos |
| Own accession authority | **GlyTouCan** | — |
| Own data encoding | **WURCS / GlycoCT** (glycans are *trees*; SMILES/InChI are useless for search) | shares SMILES/InChI |
| Own OBO ontology | **GNOme** (OBO Foundry) | LION (not Foundry-core) |
| Natural parent page? | **No** — straddles proteomics and metabolomics | Yes (metabolomics) |

Caveats before committing:
- **There is no ELIXIR Glycomics Community.** RDMkit pages depend on community ownership, so **recruit a glyco author** — SIB (Lisacek) or the HUPO-PSI MIRAGE/Glycosylation working group — rather than writing it from the metabolomics side.
- The mzIdentML glycopeptide extension is **drafted, not released**.
- MIRAGE's own 2025 paper admits GlycoPOST and UniCarb-DR have not yet fully implemented all MIRAGE parameters.
- **Do not link `mirage-project.org`** — that domain now hosts an unrelated music-informatics project. Canonical MIRAGE URL: `https://www.beilstein-institut.de/en/projects/mirage/`.
- **The GlySpace Alliance is served over HTTP only.** `http://www.glyspace.org/` returns 200; the server does not listen on port 443 at all, so any HTTPS check fails. Since RDMkit requires tool URLs to begin with `https://`, and the contribution guide excludes "webpages of groups or consortia" from the registry anyway, GlySpace is referenced in prose rather than registered as a tool.
- **UniCarbKB** appears to be partially folded into GlyConnect; its old URL returns 404. Verify before citing.

### Other siblings

| Subdomain | Infrastructure | Decision |
|---|---|---|
| **Mass spectrometry imaging / spatial metabolomics** | imzML + **METASPACE** (>10,000 public datasets) | Brief mention in the introduction; plausible future page/section. Cross-link `bioimaging_data`. |
| **Exposomics** | NORMAN-SLE (>120,000 structures, versioned on Zenodo) + MassBank Europe + EIRENE RI (ESFRI) | Brief mention; cross-link `toxicology_data`. Revisit as a page when EIRENE RI is operational. |
| **Fluxomics** | FluxML is a *model* exchange format. No repository, no reporting standard, dominant tools closed-source | Passing mention only |
| **Volatilomics / breathomics** | No community repository, no reporting checklist, no format | Passing mention only |

---

## 5. Cross-cutting with proteomics (a key message for the page)

From RDA-MOMSI: **41 of the 101 metabolomics + proteomics standards records are governed by HUPO-PSI.** By comparison the Metabolomics Standards Initiative has 3 records and the Lipidomics Standards Initiative 2. HUPO-PSI is by far the dominant standards body in the mass spectrometry space.

Directly shared between the metabolomics and `proteomics` pages:

- **mzML** — raw spectra, same format in both domains
- **PSI-MS CV** — the controlled vocabulary underpinning mzML, mzTab-M and imzML
- **msconvert / ProteoWizard** — vendor format conversion
- **mzTab family** — `mzTab` (proteomics) and `mzTab-M` (metabolomics) are the same specification family
- **mzQC** — quality control, explicitly multi-omics
- **USI** (Universal Spectrum Identifier) — point to an individual spectrum in a public repository
- **ISA** and **SDRF** — sample-to-data relationships

The page says this explicitly and links `[proteomics](proteomics)`. It matters because readers otherwise assume metabolomics standards are a separate universe.

### Notable structural asymmetry worth stating

**There is no ProteomeXchange equivalent for metabolomics.** Proteomics has a single coordinated submission and dissemination consortium; metabolomics does not. MetabolomeXchange attempted this and is dead (see §7). Readers must therefore choose a repository directly, and MetaboLights and Metabolomics Workbench operate a data-sharing arrangement rather than a formal consortium.

---

## 6. Resource inventory (all URLs verified 2026-07-14)

### Repositories — study/raw data

| Resource | URL | Notes |
|---|---|---|
| MetaboLights | https://www.ebi.ac.uk/metabolights/ | EMBL-EBI; ISA-Tab based; ELIXIR Core Data Resource; `MTBLS` accessions |
| Metabolomics Workbench | https://www.metabolomicsworkbench.org/ | NIH; `mwTab` format; `ST` study IDs; mints DOIs; de facto lipidomics archive |
| GNPS | https://gnps.ucsd.edu/ | Molecular networking + spectral libraries |
| MassIVE | https://massive.ucsd.edu/ | Already in the RDMkit registry (`massive`) |
| MetaboBank | https://www.ddbj.nig.ac.jp/metabobank/index-e.html | DDBJ. **Note:** `https://mb2.ddbj.nig.ac.jp/` returned 502 at time of check; the DDBJ landing page above is stable |

### Spectral libraries and compound databases

| Resource | URL | Notes |
|---|---|---|
| MassBank Europe | https://massbank.eu/MassBank/ | ~120k spectra; integrated with NORMAN |
| MassBank of North America (MoNA) | https://mona.fiehnlab.ucdavis.edu/ | |
| HMDB | https://hmdb.ca/ | **403 to `curl`** (Cloudflare bot protection) — site is alive in a browser |
| LIPID MAPS | https://www.lipidmaps.org/ | **403 to `curl`** (bot protection) — alive. LMSD ~50k structures |
| ChEBI | https://www.ebi.ac.uk/chebi/ | Already in registry (`chebi`) |
| PubChem | https://pubchem.ncbi.nlm.nih.gov/ | |
| SwissLipids | https://www.expasy.org/resources/swisslipids | SIB |
| NORMAN Suspect List Exchange | https://www.norman-network.com/nds/SLE | Exposomics; >120k structures; versioned on Zenodo |

### Standards, formats and vocabularies

| Resource | URL | Notes |
|---|---|---|
| mzML | (in registry: `mzml`) | HUPO-PSI. Shared with proteomics |
| mzTab-M | https://hupo-psi.github.io/mzTab/ | **The** metabolomics results format. HUPO-PSI + MSI |
| mzQC | https://www.psidev.info/mzqc | Quality control. Multi-omics |
| nmrML | https://nmrml.org/ | NMR. COSMOS + HUPO-PSI + MSI |
| imzML | https://github.com/imzML | Imaging MS. **`ms-imaging.org` returns HTTP 410 Gone and fails TLS** — the GitHub org is the working home |
| USI | https://www.psidev.info/usi | Universal Spectrum Identifier |
| SPLASH | https://splash.fiehnlab.ucdavis.edu/ | Spectral hash identifier |
| RefMet | https://www.metabolomicsworkbench.org/databases/refmet/index.php | Metabolite nomenclature harmonisation |
| CIMR | https://github.com/MSI-Metabolomics-Standards-Initiative/CIMR | Core Information for Metabolomics Reporting (MSI checklist) |
| MSIO | https://github.com/MSI-Metabolomics-Standards-Initiative/MSIO | Metabolomics Standards Initiative Ontology |
| ISA-Tools | https://isa-tools.org/ | Already in registry (`isa-tools`). MetaboLights is ISA-Tab based |
| ProteoWizard / msconvert | https://proteowizard.sourceforge.io/ | Already in registry (`msconvert`) |

Other standards found in RDA-MOMSI, deliberately **not** put on the page (to keep it focused): ANDI-MS (ASTM E1947), ThermoML, MDL Molfile, SDF, CDX, MGF, MSP, ITA/ITM (ToF-SIMS), PSI-MI XML, MI-TAB, MIMIx, MIABE, ISA-TAB-Nano, CHEMINF, CHMO, XLMOD, ChAMP, MFIF, SMART, mzTab-L (deprecated), LION (deprecated), nmrCV (deprecated).

### Identification confidence

Two scales, both essential and both absent from the original draft:

- **MSI identification levels 1–4** (Sumner et al. 2007) — level 1 identified, 2 putatively annotated, 3 putatively characterised compound class, 4 unknown.
- **Schymanski levels 1–5** (2014) for high-resolution MS — level 1 confirmed structure (reference standard), 2 probable structure, 3 tentative candidate, 4 unequivocal molecular formula, 5 exact mass only.

Reporting *which level* an annotation was made at is the core FAIR act in metabolomics. An identification without a stated confidence level is not reusable.

### Analysis tools

| Tool | URL |
|---|---|
| XCMS | https://bioconductor.org/packages/release/bioc/html/xcms.html |
| MZmine | https://mzmine.github.io/ |
| MS-DIAL | https://systemsomicslab.github.io/compms/msdial/main.html |
| MetaboAnalyst | https://www.metaboanalyst.ca/ |
| Workflow4Metabolomics | https://workflow4metabolomics.usegalaxy.fr/ |
| SIRIUS (incl. CSI:FingerID, CANOPUS) | https://bio.informatik.uni-jena.de/software/sirius/ |
| MetFrag | https://msbi.ipb-halle.de/MetFrag/ |
| METASPACE | https://metaspace2020.org/ |
| Goslin | https://apps.lifs-tools.org/goslin/ |
| Lipidomics Standards Initiative | https://lipidomicstandards.org/ |
| OpenMS | (in registry: `openms`) |

---

## 7. Bugs found in the existing RDMkit content

### MetabolomeXchange is dead

`http://www.metabolomexchange.org/site/` returns **HTTP 404** (verified 2026-07-14).

It is still:
- registered at `_data/tool_and_resource_list.yml:1320` (`id: metabolomexchange`)
- actively recommended to readers at `pages/your_domain/microbial_biotechnology.md:241`: *"Metabolomic studies can be shared through the {% tool "metabolomexchange" %}, which provides a resource for sharing data from metabolic studies and guidance for the submission of metabolome data."*

**Recommended fix (same PR):** remove the `metabolomexchange` entry from the registry and rewrite that sentence in `microbial_biotechnology.md` to point at MetaboLights and Metabolomics Workbench instead.

### The tool registry has almost no metabolomics

Of **632** registered tools, essentially every metabolomics-specific resource is missing. Reusable existing IDs: `mzml`, `mztab`, `mzidentml`, `msconvert`, `chebi`, `isa-tools`, `sdrf`, `openms`, `skyline`, `galaxy`, `massive`, `omicsdi`, `psi-ms`, `proteomics-standards-initiative`, `pride`, `proteomexchange`.

Roughly **25 new entries** are needed. This is in line with the median tool density of a domain page (~25 `{% tool %}` mentions).

### `ms-imaging.org` is down

The imzML homepage returns **410 Gone** over HTTP and fails TLS over HTTPS. Worth reporting to the Mass Spectrometry Imaging Society. Use `https://github.com/imzML` meanwhile.

---

## 8. Cross-links used on the page

**Task pages:** `metadata`, `data_quality`, `data_analysis`, `data_publication`, `identifiers`, `existing_data`, `data_interlinking`, `sensitive`, `storage`, `machine_actionability`.

**Domain pages:** `proteomics` (shared standards), `toxicology_data` (exposomics, OECD omics reporting framework), `plant_sciences`, `human_data` (human cohort metabolomics), `machine_learning`, `bioimaging_data` (imaging MS).

**Tool assemblies:** `galaxy` (Workflow4Metabolomics runs there; `usegalaxy.eu` has a metabolomics instance).

---

## 9. Open items for the contributors

1. **Verify the re-identifiability literature** for human metabolic profiles and, if it holds, strengthen the sensitive-data guidance.
2. **Decide on the glycomics page** and recruit an author if going ahead.
3. **Confirm MSI/CIMR is the right anchor** for reporting standards — CIMR comes from the RDA-MOMSI database and its current adoption level was not independently verified.
4. **Check whether the Metabolomics Standards Initiative qualifies as a "tool/resource"** under `tool_resource_update.md` (which excludes "webpages of groups or consortia"). Precedent exists: `proteomics-standards-initiative` is already registered, and MSI ships CVs and checklists rather than just a homepage.
5. **Add a bibliography** if the identification-levels and lipid-nomenclature papers should be cited formally (Sumner 2007, Schymanski 2014, Liebisch 2020, Kopczynski 2024). RDMkit uses BibTeX via `_bibliography/references.bib` and `{% cite %}`.
