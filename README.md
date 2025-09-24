# Problem & Motivation

Analog design research lacks large, verified datasets mapping schematics → SPICE. Without them, LLMs often hallucinate pins, swap polarities, or output non-simulatable netlists. Manual labeling does not scale. Masala-CHAI removes the human bottleneck by automating extraction, repair, and validation, producing a dataset and pipeline that others can reuse for training, benchmarking, and downstream tools (topology recognition, sizing assistants, etc.).

## What We’re Building (Project Description)
	•	Automated pipeline that ingests schematic PDFs/images + captions, detects symbols/pins/wires, emits SPICE, auto-repairs common issues, and validates each sample.
	•	Topology-first QC: every netlist is re-parsed into a device → module → stage graph. We stamp key motifs (diff-pairs, mirrors, cascodes), enforce symmetry ties, and run DC/AC smoke tests.
	•	Dataset at scale: each item stores (image, caption, SPICE, component list, graph JSON, motif/stage labels)—rich enough for training and programmatic audits.
	•	Models: prompt-tuned “emit/repair” LLMs optimized for structured outputs (strict JSON/SPICE), plus optional self-constrained fine-tuning to reduce format drift.

## System Overview (Pipeline)
	1.	Figure Harvesting → queue of schematic images/captions.
	2.	Vision Parsing → detect symbols (R/C/L, MOS/BJT/diodes, sources), pins, and wires; line-tracing for connectivity.
	3.	LLM Extraction & Repair → guarded “hypothesize → verify → refine” loop over a structured graph (never freestyle on raw text); asks for missing edges if confidence < τ.
	4.	Topology Graph & QC → build device→module→stage; subgraph-match motifs; enforce symmetry; static sanity (body/bias ties, floating nets).
	5.	Simulation Sanity → DC operating point + basic AC checks (e.g., nonzero op-amp gain, sensible pole range).
	6.	Packaging → write SPICE + JSON annotations; accept, auto-repair, or quarantine.
