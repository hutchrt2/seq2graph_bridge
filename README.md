# Seq2Graph Bridge

A command-line Python wrapper utility that automates protein sequence alignment searches via MMseqs2 and maps alignment hits back to a relational knowledge graph schema.

## Features

- **Automated Database Setup**: Checks for, initializes, and indexes the reference sequence collection (`input_database/psfd_sequences.fasta`) inside a managed cache directory (`blastdb/`).
- **Dynamic Version Compatibility**: Automatically handles variations in local MMseqs2 version commands (dynamically selecting `convertalignment` or `convertalis` depending on CLI support).
- **Graceful Query Search**: Converts raw input FASTA queries into database format to ensure correct type resolution before searching, and wraps subprocess commands with exit/standard error capture for clean crash debugging.
- **Relational Handshake**: Parses UniProt IDs from matched alignment target strings (e.g. `sp|Q9FL62|WRK30_ARATH` -> `Q9FL62`), joins alignment metrics with knowledge-graph metadata, and preserves 1-to-many context-specific matches (e.g. mapping UniProt IDs to unique `global_node_id`s).

---

## Directory Structure

```text
seq2graph-bridge/
├── input_database/
│   ├── psfd_sequences.fasta        # Reference sequence collection
│   └── sequence_metadata.csv       # KG relational metadata mapping
├── blastdb/                        # Managed MMseqs2 database files (generated)
├── tmp/                            # Temporary run directories and databases (generated)
├── output/                         # Serialized integration outputs (generated)
├── seq2graph_bridge.py             # Modular core python script
├── seq2graph-bridge.py             # Hyphenated entry point wrapper script
├── requirements.txt                # Python package dependencies
└── README.md                       # Documentation
```

---

## Prerequisites

1. **Python 3.8+**
2. **MMseqs2** command-line suite:
   - Ensure the `mmseqs` binary is installed and globally accessible via your system `PATH`.
   - On Linux/macOS, check using:
     ```bash
     which mmseqs
     ```

---

## Installation

Clone this repository and install the Python dependencies:

```bash
pip install -r requirements.txt
```

---

## Usage

### 1. Initialize the Reference Database

Build and index the reference database from the FASTA file:

```bash
python3 seq2graph_bridge.py --init
```

*Note: The script checks if database files are present and will automatically run this step on the first query search if it has not been done manually.*

### 2. Run a Sequence Alignment Query

Align a query sequence file and map the hits back to the relational knowledge graph schema:

```bash
python3 seq2graph_bridge.py --query path/to/query.fasta
```

### 3. Customize Output Location

By default, results are serialized to `output/blast_query_results.csv`. Use the `--output` flag to specify a custom path:

```bash
python3 seq2graph_bridge.py --query path/to/query.fasta --output path/to/my_output.csv
```

---

## Output Format

The output CSV aligns alignment metrics directly next to their biological/relational metadata contexts:

| Column | Description | Source |
|---|---|---|
| `query` | The query sequence identifier | MMseqs Search |
| `uniprot_id` | The clean mapped target UniProt Accession ID | Parsed Target |
| `pident` | Percentage of identical matches | MMseqs Alignment |
| `evalue` | Expected value (statistical significance) | MMseqs Alignment |
| `qcov` | Query coverage fraction | MMseqs Alignment |
| `tcov` | Target coverage fraction | MMseqs Alignment |
| `global_node_id` | Mapped identifier in the relational knowledge graph | metadata CSV |
| `selected_protein_name` | Name of the matched protein | metadata CSV |
| `selected_gene_name` | Name of the matched gene (if available) | metadata CSV |
| `selected_organism` | Species/organism of origin | metadata CSV |
