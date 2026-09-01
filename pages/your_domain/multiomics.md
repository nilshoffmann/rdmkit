---
title: Multi-omics
description: data management solutions for studies that combine several omics modalities.
contributors: [Nils Hoffmann]
editors: []
page_id: multiomics
related_pages:
  Your_tasks: [data_interlinking, metadata, identifiers, data_analysis, existing_data, data_provenance, sensitive]
  Tool_assembly: [galaxy]
training:
  - name: Multi-omics search query in TeSS
    registry: TeSS
    url: https://tess.elixir-europe.org/search?q=multi-omics
---

## Introduction

A multi-omics study measures the same biological material with several different technologies, in the hope that the combination says more than the parts. A cohort might be sequenced, its proteome and metabolome profiled by mass spectrometry, and its tissue imaged. The scientific promise is obvious. The data management problem is that you are not managing one dataset but several, each belonging to a different technical community, with its own formats, its own repositories, its own reporting checklists and its own idea of what a result even is.

It is useful to group omics technologies into three families, because the data management consequences follow from the family rather than from the individual assay:

- **Sequence-based omics**, such as genomics, transcriptomics and epigenomics, together with their community-level counterparts metagenomics and metatranscriptomics. Data is read out as sequence, usually aligned to a reference, and reported as features on coordinates.
- **Mass spectrometry and NMR-based omics**, such as proteomics, metabolomics and lipidomics. Data is read out as spectra, and the molecules present are *inferred* from those spectra.
- **Spatial, optical and microscopy-based omics**, such as bioimaging, spatial transcriptomics and imaging mass spectrometry. Data is read out as images, and the entities of interest are *segmented* out of them.

### Why integration is hard

The usual explanation is that the formats differ, the scales differ and the files are large. All true, and all tractable. The deeper problem is that **the three families do not establish the identity of an entity in the same way**, and this is what makes a naive join across them dangerous.

- In a sequence-based experiment, an entity is **template-derived**. A gene, a transcript or a variant is defined by its position on a reference genome. Two laboratories analysing the same sample will name the same entity the same way. The identifier is stable, and the uncertainty is mostly technical: coverage, mapping quality, and so on.
- In a mass spectrometry or NMR experiment, an entity is **inferred**. A protein or a metabolite identification is a probabilistic claim supported by evidence, and it comes with a confidence level attached. In metabolomics there is no reference to align against at all, and a large fraction of detected features are never identified. The uncertainty is in the identity itself, not merely in its measurement.
- In an imaging or spatial experiment, an entity is **constructed**. A cell, a region or a compartment exists because a segmentation step drew a boundary. Change the segmentation parameters and you change the entities. The identity is an output of your analysis, not an observation.

Metagenomics and metatranscriptomics sit awkwardly in the first family, and it is worth being explicit about why. The material sequenced is a mixture: a soil sample may contain bacteria, archaea, fungi, protozoa and viruses together, and a single read carries no label saying which of them it came from. There is no one reference genome to align against, so reads are assigned to taxa against a reference database, and a metagenome-assembled genome is binned out of an assembly rather than observed directly. Change the database version or the classifier and the taxonomic assignments change; change the binning parameters and the assembled genomes change. These entity identifiers therefore behave much more like the inferred and constructed ones above than like a gene identifier, and they should be treated with the same caution.

So when you build a matrix that puts a gene, a metabolite and a cell in the same table, the row keys have different epistemic status. The gene identifier is close to a fact. The metabolite identifier is a hypothesis with a stated confidence level. The cell identifier is an artefact of a pipeline. Integration methods that treat all three as equally reliable keys will produce results that look more certain than they are.

Two practical consequences run through the rest of this page. First, **the sample is the only thing every modality genuinely shares**, so sample identity and provenance are where multi-omics data management has to invest. Second, **the fragmentation of repositories and standards is not a defect to be fixed but a fact to be managed**: you deposit each modality in its proper home and link the parts, rather than looking for a single repository that takes everything.

### Where to read more

This page covers what is specific to *combining* modalities. For the modalities themselves, and for the research areas that most often combine them, RDMkit has dedicated pages:

- Individual modalities: [proteomics](proteomics), [metabolomics](metabolomics), [bioimaging data](bioimaging_data), [single-cell sequencing](single_cell_sequencing), [epitranscriptome data](epitranscriptome_data) and [structural bioinformatics](structural_bioinformatics).
- Research areas that are routinely multi-omic: [cancer data](cancer_data), [rare disease data](rare_disease_data), [plant sciences](plant_sciences), [microbial biotechnology](microbial_biotechnology), [marine metagenomics](marine_metagenomics), [toxicology data](toxicology_data) and [agroecology](agroecology).
- The generic data management tasks this page depends on: [data interlinking](data_interlinking), [documentation and metadata](metadata_management), [identifiers](identifiers) and [data analysis](data_analysis).

## Sample identity and provenance

### Description

If a multi-omics study has a single point of failure, this is it. The assays were run on aliquots of the same biological material, possibly in different laboratories, months apart, by people who never spoke to each other. The join across modalities is only as good as your ability to say, unambiguously and in a machine-readable way, that this proteomics injection and that sequencing run came from the same specimen.

In practice this is where multi-omics studies break. A sample called `P17-liver-2` in the sequencing facility's spreadsheet and `Pat17_Lv_B` in the mass spectrometry laboratory's notebook is a broken join, and it usually cannot be repaired after the fact. See also the general [identifiers](identifiers) page.

### Considerations

- Does every physical sample have a single, persistent identifier that every modality uses, or does each laboratory maintain its own naming?
- Can you reconstruct the chain from organism, to specimen, to aliquot, to the individual measurement, for each modality?
- Are the aliquots for different modalities recorded as siblings of a common parent sample, or as unrelated samples?
- Is the sample metadata (the organism, the tissue, the treatment, the time point) recorded once, or duplicated and allowed to drift between modalities?

### Solutions

- Assign persistent sample identifiers up front and use them everywhere. {% tool "biosamples" %} at EMBL-EBI provides accessions for biological samples that the modality-specific repositories can all reference, which makes it the natural anchor for a multi-omics study.
- Model the sample hierarchy explicitly, rather than flattening it. The organism, the specimen taken from it, and the aliquots taken from that specimen are different things, and the aliquots sent to different modalities are siblings. {% tool "isa-tools" %} and its Investigation-Study-Assay model are designed for exactly this: one investigation, several studies, many assays, with the sample lineage recorded once.
- Record sample metadata once, in the study-level description, and let each assay reference it. Duplicating the tissue and treatment fields into three modality-specific spreadsheets guarantees they will eventually disagree.
- Describe samples with shared ontologies so that the same tissue means the same thing in every modality. Use {% tool "edam" %} for data and operations, and reach for community ontologies through the {% tool "ontology-lookup-service" %} or {% tool "bioportal" %}.

## Working across the three modality families

### Description

Each family has its own mature ecosystem, and the right approach is to use each one properly rather than to invent a common denominator. What follows is a short orientation to each, with pointers to the RDMkit pages that cover them in depth.

### Considerations

- For each modality in your study, what is the community's open format, its reporting checklist and its deposition database?
- Are you producing open formats from the start, or planning to convert proprietary output later?
- Which modality has the most restrictive constraints, particularly if human data is involved, and does that constrain the whole study?
- Do the modalities in your study have compatible ideas of what a replicate is?

### Solutions

**Sequence-based omics.** Data is read out as sequence and reported against a reference. The formats are stable and near-universal (FASTA, FASTQ, and alignment and variant formats), the reporting checklists are mature, and deposition is well established.

- Deposit raw sequence in the {% tool "european-nucleotide-archive" %} or the {% tool "sequence-read-archive" %}, functional genomics data in {% tool "arrayexpress" %} or the {% tool "gene-expression-omnibus" %}, and human data subject to controlled access in {% tool "the-european-genome-phenome-archive" %}.
- Describe experiments against the Minimum Information about a high-throughput SEQuencing Experiment ({% tool "minseqe" %}), and, for environmental and host-associated samples, {% tool "mixs" %} — see [marine metagenomics](marine_metagenomics).
- For metagenomics and metatranscriptomics, report against {% tool "migs-mims" %}, and record the reference database and the classifier, with their versions, alongside the taxonomic assignments. Without them the assignments cannot be reproduced or compared with anyone else's.
- Plant studies have their own checklist in {% tool "miappe" %}; see [plant sciences](plant_sciences).
- For per-cell resolution, which brings its own container formats and its own integration problems, see [single-cell sequencing](single_cell_sequencing), and for RNA modifications see [epitranscriptome data](epitranscriptome_data).

This is the best-served of the three families; if part of your study is sequence-based, that part is unlikely to be your problem. Metagenomics is the exception to watch, because the taxonomic labels it produces are analytical results rather than stable identifiers.

**Mass spectrometry and NMR-based omics.** Data is read out as spectra and the molecules are inferred from them.

- Convert vendor output to the open standard {% tool "mzml" %} using {% tool "msconvert" %} as soon as it comes off the instrument, and use {% tool "nmrml" %} for NMR.
- Report metabolomics results in {% tool "mztab-m" %}.
- Deposit proteomics data through {% tool "proteomexchange" %} and {% tool "pride" %}, and metabolomics data in {% tool "metabolights" %} or {% tool "metabolomics-workbench" %}.

The critical point for integration is that **every identification carries a confidence level**, and that level must survive into your integrated dataset. See the [proteomics](proteomics) and [metabolomics](metabolomics) pages.

**Spatial, optical and microscopy-based omics.** Data is read out as images and the entities are segmented from them.

- Use the {% tool "ome-data-model-and-file-formats" %} for microscopy data and manage it with {% tool "omero" %}.
- Describe it against {% tool "rembi" %}, the recommended metadata checklist for biological images.
- Deposit in the {% tool "bioimagearchive" %}, or in the {% tool "image-data-resource" %} for datasets with added value.
- Imaging mass spectrometry sits across two families: it produces {% tool "imzml" %} and is annotated and shared through {% tool "metaspace" %}.

The critical point for integration is that **the segmentation is part of the result**, so it must be deposited and versioned alongside the images. See the [bioimaging data](bioimaging_data) page.

## Metadata standards across modalities

### Description

There is no single multi-omics metadata standard, and there is unlikely to be one. Each community has built what it needed: minimum-information checklists, controlled vocabularies and exchange formats, all tuned to its own technology.

The scale of this is worth seeing plainly. The Research Data Alliance's Multi-Omics Metadata Standards Interoperability working group maintains a catalogue of the standards in use across omics. It lists several dozen standards for each individual technology — genomics, proteomics and metabolomics each have their own crowded shelf — and a comparable number that it classifies as *universal*, meaning they are not tied to any one technology.

That last group is the important one. **Interoperability across omics does not happen at the level of the technology-specific standards. It happens at the universal layer**: the identifiers, the ontologies, the packaging formats and the study-level descriptions that all modalities can share. Those are what you should invest in.

### Considerations

- Which minimum-information checklist applies to each modality in your study, and are you reporting against all of them?
- Are you describing the study once at the investigation level, or repeating yourself per modality?
- Are your identifiers resolvable, or are they local accession numbers that mean nothing outside your institute?
- Can a machine traverse from your study description to each of the deposited datasets?

### Solutions

- Report each modality against its own checklist. There is no shortcut here: {% tool "minseqe" %} or {% tool "mixs" %} for sequence, {% tool "migs-mims" %} for metagenomes, the proteomics and metabolomics checklists for mass spectrometry, {% tool "rembi" %} for images. See the [documentation and metadata](metadata_management) page.
- If your study is in a regulatory context, note that the Organisation for Economic Co-operation and Development has begun publishing omics reporting frameworks that are explicitly designed to be consistent across modalities, currently for transcriptomics and metabolomics. This is the closest thing to a genuinely cross-omics reporting standard, and it is the direction of travel. See [toxicology data](toxicology_data).
- Invest in the universal layer, because that is where the modalities actually meet:
  - **Identifiers.** Persistent, resolvable identifiers for samples, people, organisations and datasets. See the [identifiers](identifiers) page.
  - **Ontologies.** Shared vocabularies for organisms, tissues, diseases and experimental factors, so that terms mean the same thing in every modality. Find them through the {% tool "ontology-lookup-service" %} or {% tool "bioportal" %}.
  - **Study-level description.** {% tool "isa-tools" %} models one investigation containing many assays, which is precisely the shape of a multi-omics study.
  - **Packaging.** {% tool "research-object-crate" %} bundles data and its metadata with machine-readable context, which is a practical way to ship a multi-modal dataset with its links intact.
  - **Web-discoverable markup.** {% tool "bioschemas" %} makes your study description findable by machines that do not know your domain.
- Prefer the universal ontologies over inventing project-specific vocabularies. A local term for a tissue type will not survive contact with a second modality, let alone a second laboratory.

## Depositing and interlinking across repositories

### Description

Omics repositories are technology-specific by design, and this is correct: a repository that accepted everything would be able to validate nothing. The consequence is that a multi-omics study is necessarily **fragmented across several repositories**, and the data management task is not to avoid that fragmentation but to make the pieces findable from one another.

This problem is generic enough to have its own RDMkit page. Read [data interlinking](data_interlinking) for the detailed guidance; what follows is only the multi-omics-specific framing.

### Considerations

- Have you recorded, in each deposit, the accessions of the other deposits from the same study?
- Is there a single study-level record that points to all the parts?
- If your study has a controlled-access component, does the public part still reveal that it exists?
- Would a stranger who found one of your datasets be able to discover the others?

### Solutions

- Deposit each modality in its proper technology-specific repository. Do not attempt to force a multi-modal study into a single archive, and do not fall back on a generic repository merely because your study spans modalities.
- Submitting to several repositories by hand is laborious, and it is where cross-references get dropped. The ELIXIR {% tool "mars" %} project addresses exactly this: it takes one ISA-JSON metadata record for a multi-omics study and distributes it to the target repositories through repository-specific adapters, so the parts are submitted together rather than one at a time. It builds on the same Investigation-Study-Assay model as {% tool "isa-tools" %}, which makes it a natural fit if you have already modelled your study that way.
- Create a study-level record that ties the parts together. {% tool "biostudies" %} at EMBL-EBI exists for this: it holds the description of a study and links out to the datasets held in the modality-specific databases, which gives your study one citable entry point.
- Cross-reference the accessions in both directions, so that each deposit names the others. A one-way link is half a link.
- Anchor everything on the sample accessions from {% tool "biosamples" %}, so the join across repositories is explicit rather than inferred from sample names.
- Use the Omics Discovery Index ({% tool "omicsdi" %}) to discover existing multi-omics datasets. It indexes datasets across genomics, transcriptomics, proteomics and metabolomics, and is the most practical starting point when looking for public data to integrate with your own. See [existing data](existing_data).
- If part of your study is human and subject to controlled access, deposit that part accordingly and keep a discoverable public metadata record. See [human data](human_data) and [data sensitivity](data_sensitivity).

## Integrated analysis

### Description

Integration is where the differences between the modality families stop being an abstraction. The modalities have different dynamic ranges, different missingness, different batch structures and different noise models. Sequencing depth, mass spectrometry drift and microscope illumination are not the same kind of nuisance variable, and a batch correction that is right for one is meaningless for another.

There is no single correct integration method, and the field is moving quickly. What is stable, and what belongs on this page, is the data management practice around the analysis. See also the general [data analysis](data_analysis) page.

### Considerations

- Are the modalities batch-corrected separately, before integration, and is each correction appropriate to its technology?
- How are you handling entities measured in one modality but missing from another, and is missingness informative?
- Are you propagating the identification confidence levels from the mass spectrometry modalities into the integrated analysis, or silently discarding them?
- Is the integrated dataset reconstructible from the deposited parts, or does it exist only on someone's laptop?

### Solutions

- Correct batch effects within each modality first, using the method appropriate to that technology, before attempting integration. Do not apply a single correction across the combined matrix.
- Keep the modalities in a container designed to hold them together with their sample mapping intact, rather than in a pile of joined spreadsheets. {% tool "multiassayexperiment" %} in {% tool "bioconductor" %} and {% tool "muon" %} in Python both represent multiple assays over a shared set of samples, and both preserve the link that the sample identifiers established.
- Choose an integration method suited to your question and record it. Multi-Omics Factor Analysis ({% tool "mofa" %}) infers latent factors that explain variation across modalities and is a reasonable default for unsupervised integration; {% tool "mixomics" %} provides a broad set of supervised and unsupervised multivariate methods.
- Carry the uncertainty through. If a metabolite entered the analysis as a level 3 annotation under the Metabolomics Standards Initiative reporting levels {% cite sumner2007Proposed %}, that fact should be visible in the integrated result. Dropping the confidence level at the join is how a tentative annotation becomes a confident conclusion.
- Run the integration in a workflow system so that the parameters and versions are captured. [Galaxy](galaxy_assembly) records every step as executable provenance, and the resulting workflow should be deposited with the data.
- Deposit the integrated dataset and the code that produced it, in addition to the raw modality-specific deposits. The integrated matrix is a derived product, and it is not reproducible from the raw parts unless you say exactly how it was made. See [data provenance](data_provenance).
- If you are applying machine learning to the integrated data, be aware that multi-modal models are unusually easy to fool with leakage across modalities. See the [machine learning](machine_learning) page.
- Look at how neighbouring communities have solved this before inventing your own approach. Multi-omics integration is routine in [cancer data](cancer_data) and [rare disease data](rare_disease_data), which combine genomic, transcriptomic and clinical data for diagnosis and stratification, and in [microbial biotechnology](microbial_biotechnology), where the design-build-test-learn cycle integrates genomic, metabolomic and phenotypic measurements. If your study involves human participants, the constraints of the most restrictive modality govern the whole study; see [human data](human_data) and [health data](health_data).

## Bibliography

{% bibliography --cited %}
