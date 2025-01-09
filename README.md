## Degradome Analysis

Degradome sequencing is used to identify and quantify RNA degradation products, or fragments, present in any given biological sample. This approach allows for the systematic identification of targets of RNA decay and provides insight into the dynamics of transcriptional and post-transcriptional gene regulation.

**[CleaveLand4](https://github.com/MikeAxtell/CleaveLand4)**: Finding evidence of sliced targets of small RNAs from degradome data. **CleaveLand4.pl** is a Perl program created by Michael J. Axtell. When running CleaveLand4, the required inputs are:

1. **Degradome (-e)**: A FASTA file containing the reads obtained from degradome sequencing.
2. **Query (-u)**: A FASTA file containing the sequences you wish to analyze as potential targets (e.g., miRNAs).
3. **Reference (-n)**: A FASTA file containing the transcriptome or reference sequences for cleavage site mapping.

<br>

### **1.   Degradome (-e)**
<br>

**Step 1: Download Degradome Library**

Use the fastq-dump from the [*fastx_toolkit*](https://github.com/agordon/fastx_toolkit) package to download the degradome library from the NCBI Sequence Read Archive [SRA](https://www.ncbi.nlm.nih.gov/sra).

```bash
fastq-dump SRR000000  # (version 2.8.2)
# Replace SRR000000 with the correct identification number for each library.
```
<br>

**Step 2: Quality control**

Perform a quality analysis using [*FastQC*](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

```bash
fastqc SRR000000.fastq  # (version v0.10.1)
```

Based on the results, decide whether to apply quality filtering.

The criteria to be worked on were:
- Remove present adapters. (fastx_clipper from the [*fastx_toolkit*](https://github.com/agordon/fastx_toolkit) package)
- Remove sequences that do not meet the defined minimum Phred quality score and the percentage of acceptable bases. (fastq_quality_flter from the [*fastx_toolkit*](https://github.com/agordon/fastx_toolkit) package)

<br>

**Step 3.1: Adapter Removal**

📄 Some libraries had adapter information available in the article describing the library construction. In other cases, we used the FastQC report to identify adapters, for example, the Illumina TrueSeq 3' adapter sequence is: Adapter 3': 5' AGATCCGAAGAGCACACGTCT. Finally, in one library we did a manual search for adapters with "head", "grep". To remove the adapter (using just 8 nucleotides is sufficient):

```bash
fastx_clipper -a AGATCGGA -l 18 -c -v -M 5 -i SRR000000.fastq -o SRR000000_clipped.fastq  # (version 0.0.14)
```
Parameters:
*   -a: Sequence to search (adapter sequence)
*   -l: Discard sequences shorter than this length
*   -c: Discard sequences that were not clipped
*   -v: Generate a report
*   -M: Clip only if the first n bases of the adapter match (e.g., 5 bases)
*   -i: Input file
*   -o: Output file

<br>

**Step 3.2: Quality Filtering**

```bash
fastq_quality_filter -q 25 -p 85 -v -i SRR000000_clipped.fastq -o SRR000000_clipped_filter.fastq  # (version 0.0.14)
```
Parameters:
*   -q: Minimum quality score to retain
*   -p: Minimum percentage of bases meeting the quality threshold
*   -v: Generate a report
*   -i: Input file
*   -o: Output file

<br>

**Step 3.3: Quality control**

Re-evaluate quality with [*FastQC*](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

```bash
fastqc SRR000000_clipped_filter.fastq  # (version v0.10.1)
```

<br>

**Step 4: Convert to FASTA Format**

Convert the processed FASTQ files to FASTA format:

```bash
fastq_to_fasta -i SRR000000_clipped_filter.fastq -o output.fasta
```

Optional:
You can concatenate multiple FASTA files. This is useful if libraries are from different tissues, growth stages, or conditions. But, concatenating may mask specific results, so analyze carefully.

```bash
cat file1.fasta file2.fasta > combined.fasta
```
<br>

### **2.   Query (-u)**

### **3.   Reference (-n)**

<br>

**Step 5: Prepare Query and reference Sequences**

Ensure the identifiers in the fasta files used as queries or reference have no spaces or uncommon characters. Only underscores (_) and dashes (-) are safe. Replace problematic characters with sed:

```bash
sed 's/-/_/g' input.fasta > output.fasta
```
For slashes (/ or \), use an alternative delimiter:
```bash
sed 's@/@_@g' input.fasta > output.fasta
```
Important: Verify the characters to replace are not part of important data.

<br>

**Step 6: Run CleaveLand**

To identify cleavage sites using the degradome library, use CleaveLand (version 4.4). By default, mapping does not allow mismatches, but for genomes of different cultivars, allow up to 2 mismatches (bowtie -v 2).

Command to execute CleaveLand:

```bash
./CleaveLand4.pl -e /path/to/degradome.fasta -u /path/to/query.fasta -n /path/to/reference.fasta -a -t -o /path/to/t_plots > output_cleaveland.tab
```
*   -e: Path to degradome file
*   -u: Path to query file
*   -n: Path to reference file
*   -a: Sort results by Allen score
*   -t: Generate tabular output (important for Excel analysis)
*   -o: Directory for T-plots (requires R installation)

Note: This step can be time-intensive, depending on the number of query sequences.

<br>

**Step 7: Evaluate Output**

Analyze the tabular output using tools like Excel. Recommended criteria for evaluable alignments:

- P-value ≤ 0.05
- Allen score ≤ 7.5–8
- Degradome peak ≤ 1

Even with excellent values, always review the generated T-plots before making final decisions.
