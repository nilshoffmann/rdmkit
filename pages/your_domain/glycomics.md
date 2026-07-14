---
title: Glycomics and glycoproteomics
search_exclude: true
description: data management solutions for glycomics and glycoproteomics data.
contributors: []
editors: []
page_id: glycomics
related_pages:
  Your_tasks: [metadata, data_publication, identifiers, existing_data, data_quality]
  Tool_assembly: []
training:
  - name: Glycomics search query in TeSS
    registry: TeSS
    url: https://tess.elixir-europe.org/search?q=glycomics
---

<!--- SEED DRAFT. This page was drafted from desk research as a starting point for a domain
expert to review, correct and expand. Reviewer notes are left inline as HTML comments.
Known gaps are listed at the end of the file. --->

## Introduction

Glycomics is the study of glycans, the carbohydrate structures attached to proteins and lipids or occurring free in a biological system. Glycoproteomics is the study of glycans in the context of the proteins that carry them, including which site on which protein carries which structure.

Glycans are biologically central, in cell recognition, immunity, host-pathogen interaction and the efficacy of biopharmaceuticals. They are also, from a data management point of view, unlike anything else in the omics landscape. Three properties drive that difference:

- **Glycan biosynthesis is not template-driven.** There is no gene that encodes a glycan. You cannot derive the set of possible glycans from a genome the way you can derive possible peptides from a reference proteome, so even constructing the search space for an experiment is a data management problem.
- **Glycans are branched.** A glycan is a tree, not a string. Linear chemical notations such as the Simplified Molecular-Input Line-Entry System (SMILES) or the International Chemical Identifier (InChI) can technically encode one, but they cannot support the structure comparison, substructure search and subsumption reasoning the field actually needs. This forces purpose-built encodings.
- **Ambiguity is the normal state, not an error state.** Isomers that differ only in linkage or branching are frequently indistinguishable by mass spectrometry. A partially determined structure, with an unknown linkage or an unassigned branch, is a routine and legitimate result, and it must be *representable* rather than merely flagged as incomplete.

Glycomics also sits awkwardly between two other domains. Released-glycan glycomics resembles [metabolomics](metabolomics): small molecules, mass spectrometry, chemical structures. Glycoproteomics resembles [proteomics](proteomics): peptide identification, mzIdentML, false discovery rate control. Glycoproteomics is in fact a hybrid of the two, with the added problem of localising a glycan to a site on a peptide. Neither parent domain's data model covers it, which is why the field has built its own.

The good news is that it has built it fairly completely: a reporting standard, a raw data repository, an accession authority for structures and an ontology.

## Describing glycan structures

### Description

Before you can share a glycan dataset, you have to be able to say what a glycan *is* in a way a machine can act on. A name such as "biantennary complex N-glycan" is not sufficient, and a chemical formula is not either, because it does not capture topology.

The field has converged on a layered answer: a canonical machine encoding, a registry that mints an accession for each encoded structure, an ontology that relates those accessions to one another, and a graphical notation for humans. Getting these right is the highest-value thing you can do for the reusability of a glycomics dataset. See also the general [identifiers](identifiers) page.

### Considerations

- How completely did your experiment actually determine each structure: composition only, topology, linkage, anomeric configuration?
- Can your chosen representation express a partially determined structure, or does it force you to over-specify?
- Are you reporting a registry accession, or only a name or a drawing?
- If you are doing glycoproteomics, are you recording the glycosylation site as well as the glycan, and how confident the site assignment is?

### Solutions

- Encode structures in {% tool "wurcs" %}, the Web3 Unique Representation of Carbohydrate Structures. It is the current canonical encoding, it is unique, and it can represent underdetermined structures explicitly. The older GlycoCT connection-table encoding is still supported and still appears in legacy data.
- Register structures in and cite accessions from {% tool "glytoucan" %}, the international glycan structure repository. A GlyTouCan accession is the stable identifier you should use to refer to a glycan in a publication or a dataset, in the same way a UniProt accession identifies a protein.
- Relate structures with {% tool "gnome" %}, the Glycan Naming and Subsumption Ontology. It organises GlyTouCan accessions by **degree of characterisation** and supports subsumption reasoning, so a structure determined only to the composition level can be related formally to the more completely determined structures it subsumes. This is the mechanism that makes glycan ambiguity machine-actionable rather than merely acknowledged.
- Draw structures using the {% tool "snfg" %} (Symbol Nomenclature for Glycans), the community standard graphical notation, which is expected by major journals and databases. {% tool "sugardrawer" %} is a web-based editor that supports ambiguous and underdetermined structures and can produce the corresponding encodings.

## Reporting standards and metadata

### Description

As in every mass spectrometry domain, the experimental context determines the result. Sample preparation, glycan release, derivatisation, separation and instrument settings all shape what you detect, and a glycan dataset without them cannot be interpreted or reused.

Glycomics has a dedicated minimum-information standard for exactly this: MIRAGE, the Minimum Information Required for A Glycomics Experiment, hosted by the Beilstein-Institut since 2011. It is modular, with separate guidelines for different experimental approaches. See also the general [metadata management](metadata) page.

<!--- REVIEWER NOTE: MIRAGE module list below was taken from the Beilstein guidelines page.
Please confirm the current set and version numbers, and whether the two NMR modules have
moved from draft to published since this was written. --->

### Considerations

- Which MIRAGE module applies to your experiment: mass spectrometry, sample preparation, liquid chromatography, capillary electrophoresis, glycan microarray, lectin microarray?
- Are you reporting how the glycans were released and derivatised, not just how they were measured?
- For glycoproteomics, are you reporting the protein database, the search parameters and the glycan search space you used?
- Does your target journal have a glycomics reporting checklist?

### Solutions

- Report against the relevant {% tool "mirage" %} module. Published modules cover sample preparation, mass spectrometry, liquid chromatography, capillary electrophoresis, glycan microarray analysis and lectin microarray analysis; the NMR modules are drafts.
- For glycoproteomics specifically, use the updated MIRAGE mass spectrometry guidelines, which were extended in 2025 to cover glycoproteomics as well as released-glycan glycomics, and which describe an integrated submission route rather than a standalone form.
- Note that MIRAGE now has a joint home with the Human Proteome Organisation Proteomics Standards Initiative as the MIRAGE and Glycosylation working group, which is also where the glycopeptide extension to {% tool "mzidentml" %} is being developed. If you work in glycoproteomics, that extension is the format to watch.
- Be realistic about current tooling. The MIRAGE authors themselves note that the repositories do not yet implement every MIRAGE parameter, so some reporting still has to be done by hand.

## Data processing and analysis

### Description

Glycan and glycopeptide identification from mass spectra is an unsolved problem in a way that peptide identification is not. Because the search space cannot be derived from a genome, each tool constructs it differently, and because glycans are branched and isomeric, each tool scores candidates differently.

The practical consequence is stark: in benchmarking studies, a large fraction of identified glycopeptides are reported by only one tool. **Which software you ran is therefore part of your result**, not an implementation detail, and recording it is a provenance obligation rather than good manners. See also the general [data analysis](data_analysis) page.

<!--- REVIEWER NOTE: the "large fraction unique to a single tool" claim comes from recent
benchmarking work and from the HUPO Human Glycoproteomics Initiative community studies.
Please replace this with a precise figure and a citation. --->

### Considerations

- Could someone reproduce your identifications from your raw data, your search parameters and your glycan database?
- Have you recorded the tool, the version, the glycan search space and the scoring thresholds?
- How are you controlling false discovery, and does your method account for the glycan as well as the peptide?
- Are you reporting site localisation confidence separately from identification confidence?

### Solutions

- Convert vendor raw data to {% tool "mzml" %} using {% tool "msconvert" %}, as for any other mass spectrometry experiment, and keep the vendor raw files as well.
- Choose search software appropriate to your data and record it explicitly. {% tool "pglyco3" %}, {% tool "fragpipe" %} and {% tool "metamorpheus" %} are widely used open options; commercial tools are also common in the field.
- Annotate released-glycan spectra with {% tool "glycoworkbench" %}. Its files are the de facto currency for deposition to UniCarb-DR and for automatic registration of structures in GlyTouCan, so using it has downstream benefits beyond the analysis itself.
- Report the glycan search space you used, not just the tool. Two runs of the same software against different glycan databases are different experiments.
- Given the tool-to-tool variability, consider reporting results from more than one search engine, and be explicit about which identifications are supported by which.

## Sharing, preserving and reusing glycomics data

### Description

Glycomics has its own deposition infrastructure, which is what most clearly distinguishes it from lipidomics and other mass spectrometry subdomains that rely on a parent domain's repositories.

The pieces are designed to work together: a raw data repository, a structure registry that is populated automatically from submissions, a harvester that makes published submissions queryable, and a newer repository for glycoconjugates. In practice, though, glycoproteomics raw data is still often deposited to the proteomics repositories instead, so you may have a genuine choice to make. See also the general [data publication](data_publication) page.

### Considerations

- Is your data released-glycan glycomics, glycoproteomics, or both, and which repository does the community expect for each?
- Do you need an embargo or reviewer access before publication?
- Will your glycan structures be registered and given accessions, and if not, how will anyone refer to them?
- Is your glycomics data part of a larger multi-omics study?

### Solutions

- Deposit raw glycomics and glycoproteomics mass spectrometry data in {% tool "glycopost" %}, the community raw data repository. It is vendor-agnostic, its submission conditions follow MIRAGE, and it supports embargo and reviewer access.
- Deposit annotated glycan spectra in {% tool "unicarb-dr" %}, which harvests published submissions for structured querying, and consult its curated sister resource {% tool "unicarb-db" %}, a reference library of glycan tandem mass spectra.
- For glycopeptide and glycoconjugate data, consider {% tool "glycomb" %}, which assigns identifiers linking the peptide, the protein and the GlyTouCan glycan accession.
- If you are depositing glycoproteomics data alongside a conventional proteomics dataset, {% tool "massive" %} and {% tool "pride" %} remain common destinations. Say clearly which route you took, and cross-reference the deposits. See [data interlinking](data_interlinking).
- Look for existing data and knowledge in the integrative portals: {% tool "glygen" %}, {% tool "glyconnect" %} and {% tool "glycosmos" %}, and, for microbial, plant and fungal glycans, {% tool "csdb" %}. Glyco@Expasy ({% tool "glyco-expasy" %}) is the European entry point.

<!--- REVIEWER NOTES AND KNOWN GAPS — please address before this page is published:

1. CONTRIBUTORS. This page has no domain author yet. It should not be merged without one.
   Suggested communities to approach: the HUPO-PSI MIRAGE and Glycosylation working group,
   SIB (Glyco@Expasy / GlyConnect), and the HUPO Human Glycoproteomics Initiative.

2. There is currently NO ELIXIR Glycomics Community. RDMkit domain pages normally have a
   community behind them. The editorial board should decide whether that is a blocker.

3. QUALITY CONTROL is not covered. Glycomics presumably has reference materials, internal
   standards and interlaboratory comparison practice (the HUPO Human Glycoproteomics
   Initiative has run community challenges). A quality control section modelled on the one
   in the metabolomics page would strengthen this considerably.

4. QUANTIFICATION is not covered at all, and the 2025 MIRAGE update has a quantification
   and statistics section. Worth adding.

5. GLYCAN MICROARRAY and LECTIN ARRAY data management is not covered, though MIRAGE has
   modules for both. Only mass spectrometry is addressed here.

6. The GlySpace Alliance (Glyco@Expasy + GlyCosmos + GlyGen) coordinates glycoinformatics
   internationally. It is deliberately not registered as a tool, because the RDMkit
   contribution guide excludes consortium webpages, and because it is served over HTTP
   only, while RDMkit requires https:// tool URLs. Mention it in prose if useful.

7. Check whether UniCarbKB is still live and distinct from GlyConnect; it appeared to be
   partially folded into GlyConnect and its old URL returns 404.

8. Consider adding a Bibliography. Candidate citations: the 2025 MIRAGE update
   (Mol Cell Proteomics, DOI 10.1016/j.mcpro.2025.101473), the WURCS paper
   (Nucleic Acids Res 44:D1237), the GNOme paper, and the HUPO HGI community
   evaluation of glycoproteomics software (Nat Methods 2021).
--->
