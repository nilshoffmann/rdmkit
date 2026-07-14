---
title: Metabolomics
description: data management solutions for metabolomics and lipidomics data.
contributors: [Nils Hoffmann]
editors: []
page_id: metabolomics
related_pages:
  Your_tasks: [metadata, data_quality, data_analysis, data_publication, identifiers, existing_data]
  Tool_assembly: [galaxy]
training:
  - name: Metabolomics search query in TeSS
    registry: TeSS
    url: https://tess.elixir-europe.org/search?q=metabolomics
---

## Introduction

Metabolomics is the large-scale study of metabolites, the small molecules present in cells, biofluids, tissues or whole organisms. Because metabolite levels respond quickly to genetic and environmental change, a metabolome is a snapshot of what a biological system is actually doing at a given moment, rather than what it is capable of doing.

That closeness to phenotype is also what makes metabolomics data hard to manage. Unlike a genome, a metabolome has no reference. There is no template you can align against, no finite list of entities to match, and no equivalent of a reference proteome from which the measurable set can be derived. What you can detect depends on the extraction protocol, the analytical platform, the chromatography and the instrument settings, so two laboratories studying the same samples can legitimately report different metabolites. This has four consequences for research data management:

- The experimental context is part of the result. Sample preparation, separation method, instrument and acquisition parameters all shape what is measured, so they have to be captured as metadata rather than left in a laboratory notebook.
- Measurements are only comparable if quality control is designed into the acquisition. Signal drift and batch effects are the norm, not the exception, and they cannot be corrected afterwards unless the run structure was recorded.
- Metabolite identification is uncertain, and that uncertainty is itself data. An annotation is only reusable if you also report how confident it is and what evidence supports it.
- Data is heterogeneous. Nuclear magnetic resonance (NMR), liquid chromatography-mass spectrometry (LC-MS), gas chromatography-mass spectrometry (GC-MS) and direct infusion mass spectrometry all produce different raw data, most of it in proprietary vendor formats.

This page covers the core data management practices for metabolomics and, in the final section, for lipidomics. Several neighbouring areas are only touched on briefly: mass spectrometry imaging and spatial metabolomics, where {% tool "imzml" %} and {% tool "metaspace" %} are the main entry points; exposomics, which shares its infrastructure with [toxicology data](toxicology_data); and fluxomics and volatilomics, which do not yet have mature community repositories or reporting standards.

Much of what follows is shared with mass spectrometry-based [proteomics](proteomics). If you work across both, read the two pages together.

## Metadata, standards and data formats

### Description

Metabolomics data without metadata is close to worthless. A peak list on its own tells a reuser nothing about which organism was sampled, how the sample was extracted, which column and gradient were used, or how the instrument was configured, and every one of those affects the result.

Fortunately, you do not have to invent a way to record this. The {% tool "metabolomics-standards-initiative" %} and the Human Proteome Organisation Proteomics Standards Initiative ({% tool "proteomics-standards-initiative" %}) between them provide reporting checklists, controlled vocabularies and open file formats that cover the whole path from raw spectra to reported results. Notably, most of the formats here are governed by the Proteomics Standards Initiative and are shared with proteomics, so adopting them buys you interoperability across both domains. See also the general [metadata management](metadata) page.

### Considerations

- Which analytical platform did you use, and does an open format exist for it?
- Can your instrument's raw output be converted to an open format, and are you keeping the vendor raw files as well?
- Are you describing your samples and your assays with a controlled vocabulary, or with free text?
- Will your chosen repository accept the format you are producing, and does it require a particular metadata structure?

### Solutions

- Report the experiment against a community checklist. The Core Information for Metabolomics Reporting ({% tool "cimr" %}) checklist from the Metabolomics Standards Initiative sets out the minimum you should describe. For studies in a regulatory toxicology context, the Organisation for Economic Co-operation and Development (OECD) omics reporting framework applies instead.
- Structure your study metadata with {% tool "isa-tools" %}. The Investigation-Study-Assay model separates the experimental design from the individual assays, and it is what {% tool "metabolights" %} expects on submission, so using it early saves work later.
- Convert raw data to an open format as soon as it comes off the instrument. {% tool "mzml" %} is the open standard for raw mass spectra, and {% tool "msconvert" %} converts the major vendor formats to it. For NMR, use {% tool "nmrml" %}. For imaging, use {% tool "imzml" %}.
- Report your results in {% tool "mztab-m" %}. This is the metabolomics-specific results format, and it is the one to use for identifications and quantifications. It is the counterpart of mzTab in proteomics and, unusually, is explicitly designed to cover lipidomics too.
- Annotate with controlled vocabularies rather than free text. The Metabolomics Standards Initiative Ontology ({% tool "msio" %}) covers metabolomics study design, and the mass spectrometry controlled vocabulary ({% tool "psi-ms" %}) covers instruments and acquisition. Use {% tool "chebi" %} for chemical entities.
- Keep the vendor raw files. Conversion is lossy in practice, tools improve, and repositories generally accept both.

## Study design and quality control

### Description

This is where metabolomics differs most sharply from other omics, and it is the part most often left out of a data management plan.

In metabolomics, data quality is a property of how the acquisition sequence was designed, and it cannot be recovered afterwards. Instrument response drifts over the course of a run, and it differs between batches. If you acquire your control samples first and your treated samples second, the batch effect and the biological effect are confounded and no amount of downstream statistics will separate them.

The remedy is well established: randomise the injection order, interleave pooled quality control samples throughout the run, include blanks and internal standards, and record the batch structure. The data management point is that **all of this has to be captured as metadata**. A reuser downloading your study needs to know which injections were quality control samples, which were blanks, which batch each sample belonged to, and in what order they were run. A deposited dataset that omits this is, for practical purposes, not reusable. See also the general [data quality](data_quality) page.

### Considerations

- Can someone reading your deposited metadata tell which injections were pooled quality control samples, which were blanks and which were study samples?
- Is the batch structure and the injection order recorded, or only the sample list?
- Did you include internal standards, and are they described well enough for someone else to use them for normalisation?
- Are you reporting quality metrics alongside the data, or expecting reusers to recompute them?
- For a clinical or multi-site study, did you use a common reference material so that batches can be aligned across sites?

### Solutions

- Design the run before you acquire anything. Randomise sample order, and interleave pooled quality control samples (an aliquoted mixture of all study samples) at regular intervals so that drift can be modelled and corrected.
- Encode the sample role in your metadata explicitly. In the {% tool "isa-tools" %} model, a pooled quality control sample, a blank and a study sample are different sample types, and saying so is what makes the distinction machine-readable.
- Report quality metrics in {% tool "mzqc" %}, the Proteomics Standards Initiative format for exchanging quality control information from mass spectrometry runs. It is deliberately multi-omics and applies directly to metabolomics.
- Use a reference material where one exists, so that your batches can be compared with other laboratories' rather than only with each other.
- Correct for drift and batch effects during processing, and record what you did. The correction is part of the provenance of the final data, not a private preprocessing detail. See [data provenance](data_provenance).

## Metabolite identification and annotation

### Description

Identifying a metabolite is not a lookup. In a typical untargeted experiment you detect thousands of features, most of which you will never identify, and the ones you do identify you identify with varying degrees of certainty. A mass alone might narrow a feature to a molecular formula. A fragmentation spectrum matching a library entry makes a structure probable. Only a comparison against an authentic reference standard, run on your own instrument under your own conditions, makes it confirmed.

Treating these as if they were the same thing is the most common way metabolomics results become unreusable. The single most important thing you can do is **report the confidence level of every annotation** and the evidence behind it.

### Considerations

- For each reported metabolite, what evidence do you actually have: an exact mass, a formula, a spectral match, or a reference standard?
- Are you reporting a confidence level explicitly, using a recognised scale?
- Are you using structure-based identifiers, or only compound names?
- What are you doing with the features you could not identify?

### Solutions

- Report an identification confidence level for every annotation, using a recognised scale. The Metabolomics Standards Initiative levels (1 to 4, from identified compound down to unknown) and the Schymanski levels (1 to 5, designed for high-resolution mass spectrometry) are both widely accepted. State which scale you used.
- Use structure-based identifiers, not names. Metabolite names are ambiguous and inconsistent; the International Chemical Identifier (InChI) and its hashed form, the InChIKey, are not. Record database accessions alongside them: {% tool "chebi" %}, {% tool "pubchem" %} and {% tool "hmdb" %}. {% tool "refmet" %} harmonises metabolite nomenclature across studies and is useful when you need to compare datasets. See also the general [identifiers](identifiers) page.
- Search spectral libraries for annotation. {% tool "massbank" %}, {% tool "mona" %} and {% tool "gnps" %} are the main open collections. {% tool "splash" %} gives a spectrum a hash-based identifier, which lets you refer to a specific spectrum unambiguously, and the Universal Spectrum Identifier ({% tool "usi" %}) lets you point at an individual spectrum inside a public repository.
- Where no library entry exists, use computational annotation. {% tool "sirius" %} predicts molecular formulae and structures from fragmentation spectra, and {% tool "metfrag" %} performs in-silico fragmentation against candidate structures. Both produce ranked candidates, not answers, so report them at the appropriate confidence level.
- Deposit the unidentified features too. The large fraction of unannotated signals, sometimes called metabolomic dark matter, is not noise. Deposited raw data lets these features be annotated later, as libraries and tools improve, by you or by anyone else.

## Data processing and analysis

### Description

Getting from raw spectra to a table of metabolite abundances involves a long chain of choices: peak picking, alignment, deconvolution, gap filling, normalisation, drift correction, statistics. Each has parameters, and the parameters materially change the result. Two analysts processing the same raw files with different settings will produce different tables, both defensible.

This makes analysis provenance a first-order data management problem rather than an afterthought. See also the general [data analysis](data_analysis) page.

### Considerations

- Could someone else reproduce your processed data from your raw data and your recorded parameters?
- Are your processing parameters recorded in a machine-readable way, or only in the methods section of a paper?
- Are you using a workflow system, and is the workflow itself deposited alongside the data?
- Does your tool export an open results format?

### Solutions

- Use established, open processing tools that read and write open formats. {% tool "xcms" %}, {% tool "mzmine" %}, {% tool "ms-dial" %} and {% tool "openms" %} are the most widely used, and all consume {% tool "mzml" %}.
- Run the analysis in a workflow system so that the parameters and the tool versions are captured automatically. {% tool "workflow4metabolomics" %} provides a curated metabolomics environment on [Galaxy](galaxy), which records every step, parameter and version as executable provenance.
- Use {% tool "metaboanalyst" %} for statistical analysis and pathway interpretation, but export and record what you did rather than relying on an interactive session that leaves no trace.
- Export results as {% tool "mztab-m" %} so that the identifications, the quantifications and their supporting evidence travel together in a single documented format.
- Deposit the workflow, the parameters and the tool versions with the data. A processed table without them is a conclusion, not a result.

## Sharing, preserving and reusing metabolomics data

### Description

Metabolomics has good public repositories, and most journals and funders now expect deposition. What it does not have is a single coordinated submission consortium: there is no direct equivalent of ProteomeXchange in proteomics, so you choose a repository yourself rather than submitting through a common front door.

Deposit the raw data, not just the processed table. Raw spectra can be reprocessed and re-annotated as tools and libraries improve; a summary table cannot. See also the general [data publication](data_publication) and [existing data](existing_data) pages.

### Considerations

- Which repository fits your data type, your funder's requirements and your community's expectations?
- Are you depositing raw data, processed data and metadata, or only the final table?
- Do you need an embargo or private reviewer access before publication?
- Does your study involve human samples, and if so what constraints apply?
- Is your metabolomics data part of a multi-omics study whose parts will end up in different repositories?

### Solutions

- Deposit study data in a dedicated repository. {% tool "metabolights" %} at EMBL-EBI is the ELIXIR-recommended deposition database; it is built on the {% tool "isa-tools" %} model and accepts data from all major platforms including NMR. {% tool "metabolomics-workbench" %} is the main alternative, uses its own mwTab format, and mints a digital object identifier for each study. {% tool "metabobank" %} at DDBJ serves the same role in Japan, and {% tool "gnps" %} is the natural home for data you intend to analyse by molecular networking.
- Contribute your reference spectra back to a library. Depositing to {% tool "massbank" %} or {% tool "mona" %} turns a private measurement into a public annotation resource, and it is one of the highest-leverage things a metabolomics laboratory can do.
- If your samples are of human origin, treat the data as potentially personal. Metabolic profiles are derived from human material and are usually linked to clinical or lifestyle data, so consent, access control and legal basis all apply. See [human data](human_data), [data sensitivity](sensitive) and [GDPR compliance](gdpr_compliance).
- For a multi-omics study, deposit each data type in its appropriate repository and then interlink them, so the parts can be found together. See [data interlinking](data_interlinking). {% tool "omicsdi" %} indexes datasets across genomics, proteomics, transcriptomics and metabolomics, and is a good place to look for existing data to reuse.

## Lipidomics

### Description

Lipidomics is the branch of metabolomics concerned with lipids. It shares its repositories, its file formats and most of its tooling with the rest of metabolomics, which is why it is covered here rather than on a page of its own: you deposit to the same places, in the same formats, and process with many of the same tools.

What is genuinely different is naming. A lipid name is not a stable identifier but a statement about **how much structural detail the measurement actually resolved**. `PC 34:1` says only that a phosphatidylcholine with 34 carbons and one double bond was detected. `PC 16:0_18:1` additionally names the two fatty acyl chains but does not say which is at which position. `PC 16:0/18:1` assigns those positions. `PC 16:0/18:1(9Z)` further locates the double bond and its geometry. These are four different levels of structural detail, and reporting at a level higher than your assay supports is a data integrity problem, not a stylistic one.

### Considerations

- At what level of structural detail did your assay actually resolve each lipid, and are you reporting at exactly that level?
- Are your lipid names valid under the current shorthand nomenclature, or are they house style?
- Are you reporting the internal standards and the quantification approach you used?

### Solutions

- Follow the shorthand nomenclature maintained by {% tool "lipidmaps" %}, and report each lipid at the level of structural detail your data supports, no higher.
- Normalise and validate your lipid names with {% tool "goslin" %}. It parses the major lipid naming grammars, converts between them, and tells you which level of structural detail a given name encodes, which makes it the practical way to make lipid nomenclature machine-actionable.
- Use the {% tool "lipidomics-standards-initiative" %} guidelines, which cover extraction, acquisition, identification, quantification and quality control. They also publish a lipidomics-specific minimal reporting checklist, which is separate from the general metabolomics checklists and is what you should report against.
- Look lipids up in {% tool "lipidmaps" %} for structures and classification, or {% tool "swisslipids" %} for links to biological knowledge.
- Report results in {% tool "mztab-m" %}, which supports lipids explicitly, and deposit to {% tool "metabolights" %} or {% tool "metabolomics-workbench" %} as for any other metabolomics study.
