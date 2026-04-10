# COMPASS Quickstart Guide

Welcome! This guide will help you run COMPASS for the first time on your Mac. Don't worry if you're new to bioinformatics or command-line tools — we'll walk you through each step.

---

## What is COMPASS?

COMPASS analyzes gut microbiome sequencing data to tell you **what microbes are present** and **how abundant** they are in your samples.

Here's what happens:
1. **Check quality** of raw sequencing reads
2. **Remove contamination** (human DNA, adapters)
3. **Identify microbes** using a reference database (Kraken2)
4. **Measure abundance** of each microbe (Bracken)
5. **Generate a report** summarizing everything (MultiQC)

---

## Step 1: Install Required Software

You need to install these tools **once**. Your computer must have at least **48 GB of RAM**.

### 1.1 Install Nextflow

Nextflow is the workflow manager that runs COMPASS.

```bash
brew install nextflow
```

Verify it works:
```bash
nextflow -v
```

You should see version 25.04.0 or higher.

### 1.2 Install Docker Desktop

Docker runs the bioinformatics tools in isolated containers (think of them as pre-configured mini-computers).

1. Go to [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)
2. Download and install the version for your Mac (Intel or Apple Silicon)
3. **Important:** Launch Docker Desktop and let it start. You'll see the whale icon in the menu bar.

To verify Docker is running:
```bash
docker ps
```

You should see an empty list with no error messages.

### 1.3 Install AWS CLI

This tool downloads reference databases from a public cloud server.

```bash
brew install awscli
```

Verify it works:
```bash
aws --version
```

### 1.4 Install Git

Git allows you to clone the COMPASS code.

```bash
brew install git
```

Verify it works:
```bash
git --version
```

---

## Step 2: Get the COMPASS Code

Clone COMPASS to your computer. **Use a local clone, not the GitHub version** — the local version has Mac-specific settings already configured.

```bash
git clone /Users/draminezorgani/Documents/SwipeBiome_CodeBase/compass ~/compass
cd ~/compass
```

Now you have COMPASS ready to use.

---

## Step 3: Prepare Your Input Files

COMPASS needs two things from you:

### 3.1 Add Your Sequencing Data (FASTQ files)

These are your raw sequencing reads. You should have paired-end files (usually ending in `.fastq.gz` or `.fq.gz`).

Example file names:
- `sample1_R1.fastq.gz` (read 1 from sequencer)
- `sample1_R2.fastq.gz` (read 2 from sequencer)

**Note:** You need both files for each sample. If your files are gzipped (compressed), that's fine.

### 3.2 Edit the Sample Sheet

The file `data/samplesheet.csv` tells COMPASS which files to process.

Open it in a text editor and edit it. The format is:
```
sample_id,run_id,fastq_1,fastq_2
```

**Example:**
```
sample_id,run_id,fastq_1,fastq_2
myatient_1,RUN1,/Users/yourname/data/patient1_R1.fastq.gz,/Users/yourname/data/patient1_R2.fastq.gz
myatient_2,RUN1,/Users/yourname/data/patient2_R1.fastq.gz,/Users/yourname/data/patient2_R2.fastq.gz
```

**Important rules:**
- Use **absolute paths** (starting with `/Users/...`), not relative paths
- `sample_id`: A name for your sample (e.g., `patient1`, `control_A`)
- `run_id`: A label for this batch of sequencing (e.g., `RUN1`, `BATCH_2025`)
- `fastq_1` and `fastq_2`: Full paths to your paired-end files

You can have multiple samples. Each sample gets one row.

---

## Step 4: (Optional) Tune Parameters

The file `data/params.json` controls how strict or lenient the analysis is. **Safe defaults are already set.** You only need to edit this if you want to change behavior.

### Common parameters you might adjust:

| Parameter | What It Does | Default |
|-----------|--------------|---------|
| `preprocessing_min_read_length` | Minimum length to keep reads (shorter = faster) | 15 |
| `preprocessing_fastp_qualified_quality_phred` | Minimum quality score (higher = stricter) | 15 |
| `do_host_removal` | Remove human DNA | true |
| `host_removal_bowtie2_very_sensitive` | Use stricter alignment (slower but more accurate) | true |

**Leave these as-is unless you know what you're changing.**

---

## Step 5: Before You Run — Check Everything

Before launching the pipeline, verify:

1. **Docker Desktop is running**  
   Look for the whale icon in the menu bar at the top-right of your screen.

2. **Your samplesheet is edited**  
   Did you add your samples to `data/samplesheet.csv`?

3. **Your FASTQ files exist**  
   Run this command to check:
   ```bash
   ls -lh /Users/yourname/data/*.fastq.gz
   ```
   You should see your files listed.

4. **You're in the right directory**  
   ```bash
   pwd
   ```
   Should show: `/Users/yourname/compass` (or wherever you cloned it)

---

## Step 6: Run COMPASS

Run this **as a single line** (no backslashes). Copy and paste it exactly:

```bash
nextflow run . -profile docker --input data/samplesheet.csv --ref_databases data/ref_databases.csv --outdir results -params-file data/params.json
```

**What's happening:**
- `nextflow run .` — Run Nextflow in the current directory
- `-profile docker` — Use Docker containers
- `--input data/samplesheet.csv` — Use your sample sheet
- `--ref_databases data/ref_databases.csv` — Use the reference databases (already configured for you)
- `--outdir results` — Save output to a `results` folder
- `-params-file data/params.json` — Use your parameters file

You should see output like:
```
N E X T F L O W  ~  version 25.04.0
Launching `./main.nf` [elegant_darwin] - revision: ...
```

The pipeline will start. This takes a long time (1-8 hours depending on your sample size and Mac speed).

---

## Step 7: What's Happening While It Runs

COMPASS will:

1. **Download reference databases** (Kraken2, Bowtie2 indexes) from the cloud
2. **Run FastQC** on raw reads (5-10 min)
3. **Trim adapters** with fastp (10-30 min)
4. **Remove human DNA** with Bowtie2 (20-60 min)
5. **Classify microbes** with Kraken2 (30-90 min)
6. **Estimate abundance** with Bracken (10-20 min)
7. **Generate report** with MultiQC (5 min)

Total runtime depends on:
- Number of samples
- Number of reads per sample
- Your Mac's speed and RAM

---

## Step 8: Resume After Interruption

If the pipeline stops (power loss, network issue, etc.), you can resume from where it left off:

```bash
nextflow run . -profile docker --input data/samplesheet.csv --ref_databases data/ref_databases.csv --outdir results -params-file data/params.json -resume
```

Just add `-resume` at the end. COMPASS skips completed steps and continues.

---

## Step 9: Look at Results

Results go to the `results/` folder. **Start here:**

```bash
open results/multiqc/multiqc_report.html
```

This opens an interactive HTML report in your browser showing:
- Read quality before/after trimming
- How many reads were removed as human DNA
- Taxonomic composition (what microbes are in each sample)
- Summary statistics

Other important output folders:
- `results/kraken2/` — Microbe classifications (raw data)
- `results/bracken/` — Abundance estimates per sample
- `results/qc/` — Quality control reports for each step

---

## Troubleshooting

### "command not found: --input"
You used a backslash across multiple lines. **Run the entire command as one line, no backslashes.**

### "Process requirement exceeds available memory"
Your Mac doesn't have enough RAM. COMPASS needs at least 48 GB for Bowtie2 and Kraken2. This is already configured in `nextflow.config`.

### "Access Denied" error on S3
This shouldn't happen — the reference databases are public. Make sure you're using the `data/ref_databases.csv` file that came with COMPASS (it has `s3://genome-idx/...` paths).

### "Docker not running"
Open Docker Desktop. Look for the whale icon in the top-right menu bar. Wait 30 seconds for it to fully start.

### "FASTQ file not found"
Check:
1. Does the file actually exist? Run `ls -lh /path/to/file.fastq.gz`
2. Is the path absolute (starting with `/Users/...`)? Not relative like `data/myfile.fastq.gz`
3. Can your user read the file? Try `cat /path/to/file.fastq.gz | head`

### Pipeline is very slow
This is normal. Kraken2 with the full reference database can take 1-2 hours per sample. Check Task Manager to ensure Docker is getting enough RAM.

### "Run merging" error
This happens if the same sample has multiple run IDs in the samplesheet. Either:
1. Set `do_run_merging: true` in `data/params.json` to merge them, or
2. Use the same `run_id` for files from the same sample

---

## Getting Help

If you get an error not listed here:

1. Read the error message carefully. Usually it says which file or step failed.
2. Check that all input files exist and paths are correct.
3. Make sure Docker is running.
4. Try running the command again (sometimes network issues are temporary).
5. Contact the COMPASS team at [CMIInfo@ucsd.edu](mailto:CMIInfo@ucsd.edu).

---

## Next Steps

After COMPASS finishes:
1. Open the MultiQC report (`results/multiqc/multiqc_report.html`)
2. Check sample quality and read counts
3. Look at the Bracken abundance tables in `results/bracken/`
4. Use these results in downstream analysis (statistics, visualization, etc.)

For detailed output documentation, see [docs/output.md](docs/output.md).

---

## Key Points to Remember

- **Always run the pipeline command as a single line** (no backslashes)
- **Docker Desktop must be running** before you start
- **Use absolute paths** in your samplesheet (not relative paths)
- **48 GB RAM minimum** for your Mac
- **Reference databases download automatically** (no setup needed)
- **Use `-resume`** if the pipeline gets interrupted

---

## For More Information

- **COMPASS documentation:** See [README.md](README.md)
- **Output documentation:** See [docs/output.md](docs/output.md)
- **Citations:** See [CITATIONS.md](CITATIONS.md)
- **Nextflow basics:** [nf-core documentation](https://nf-co.re/docs/usage/installation)

Good luck! You've got this.
