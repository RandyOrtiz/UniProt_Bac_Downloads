# Downloading and Merging UniProt Bacterial Data Without Unintended Deduplication

**Date:** 12-10-2024

## Purpose
This document provides a step-by-step procedure for downloading UniProt data for bacterial entries (Taxonomy ID: 2) in a way that avoids unintended deduplication. It ensures that all associated annotations (e.g., multiple GO terms) are retained. The procedure applies to both FASTA and TSV formats.

## Scope
This guide is intended for researchers, data analysts, and bioinformaticians who need to systematically obtain comprehensive bacterial data from UniProt without losing annotation details.

## Prerequisites
- Access to the UniProt website or command-line tools (e.g., `wget`, `curl`) to download files.
- A Bash shell environment for file merging and sorting.
- Familiarity with basic command-line utilities such as `cat`, `tail`, `sort`, and `cut`.
- Sufficient disk space to store all downloaded partial files.

## Responsibility
It is the responsibility of the data analyst or researcher to follow these instructions carefully to maintain data integrity.

---

## Procedure

### 1. Rationale for Partial Downloads
**Do not** attempt to download the entire bacterial dataset in one go. Doing so can cause UniProt’s internal deduplication logic to remove important entries. For instance, a protein entry with four GO terms may be split internally into multiple records. A direct bulk download might retain only one record with fewer GO terms, leading to data loss.

By splitting downloads based on annotation score and sequence length ranges, you ensure that all unique entries and their associated annotations are captured.

### 2. Taxonomic Filter
Select bacteria using Taxonomy ID = 2.

### 3. Annotation Score-Based Downloads
Retrieve subsets of data according to annotation score.

**Examples:**
- **Annotation Score 5:**  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A5)

- **Annotation Score 4:**  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A4)

- **Annotation Score 3:**  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A3)

### 4. Segmenting by Sequence Length for Lower Annotation Scores
For annotation scores 2 and 1, break down downloads by sequence length.

**Annotation Score 2:**
- Length 1-200:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A2%2Clength%3A%5B1+TO+200%5D)
- Length 201-400:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A2%2Clength%3A%5B201+TO+400%5D)
- Length 401-1000000:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A2%2Clength%3A%5B401+TO+1000000%5D)

**Annotation Score 1:**
- Length 1-200:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A1%2Clength%3A%5B1+TO+200%5D)
- Length 201-290:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A1%2Clength%3A%5B201+TO+290%5D)
- Length 291-400:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A1%2Clength%3A%5B291+TO+400%5D)
- Length 401-600:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A1%2Clength%3A%5B401+TO+600%5D)
- Length 601-1000000:  
  [Link](https://www.uniprot.org/uniprotkb?query=%28taxonomy_id%3A2%29+AND+%28go%3A0008150%29&facets=annotation_score%3A1%2Clength%3A%5B601+TO+1000000%5D)

### 5. Downloading the Files
Use `wget`, `curl`, or a browser to download each subset. Name the files descriptively, for example:

uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__01.tsv ... uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__11.tsv

### 6. Merging Downloaded Files
After all files are downloaded, merge them into a single file without duplicating headers:

```bash
files=("uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__01.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__02.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__03.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__04.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__05.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__06.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__07.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__08.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__09.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__10.tsv"
 "uniprotkb_taxonomy_id_2_AND_go_0008150_2024_12_10__11.tsv")

output="bac_bio.tsv"

for i in "${!files[@]}"; do
  if [[ $i -eq 0 ]]; then
    cat "${files[i]}" >> "$output"
  else
    tail -n +2 "${files[i]}" >> "$output"
  fi
done

7. Sorting the Combined File

To sort by the first column (e.g., UniProt accession):

(head -n 1 bac_bio.tsv && tail -n +2 bac_bio.tsv | sort -t$'\t' -k1,1) > sorted_bac_bio.tsv

8. Checking for Duplicate Entries

(Optional) Check how many duplicate entries are present for auditing purposes:

cut -f1,2 sorted_bac_bio.tsv | sort | uniq -d | wc -l

This should give you the count of any duplicate lines based on the first two columns.


---

Records and Documentation

Keep a copy of all downloaded files.

Use the final merged file (sorted_bac_bio.tsv) for downstream analysis.

Document the number of duplicates for internal quality checks.


Version Control

Date: 12-10-2024

Changes: Initial version of the SOP for correct UniProt data download and merging.



---

Recent download of the database: 12-10-2024



