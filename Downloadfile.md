#### This example demonstrates how to download raw data, such as fastq.gz files, from the ENA database and rename the filenames

**For example, as GSE135497**<br>

- **Step 1**
First，download metadata from [Run Selector :: NCBI (nih.gov)](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA559094&o=acc_s%3Aa), and the ENA [ENA Browser (ebi.ac.uk)](https://www.ebi.ac.uk/ena/browser/view/PRJNA559094)<br>
(here, i selected aspera to download, so please download report TSV: include column **"fastq_aspera"**)<br>

- **Step 2**
Then, based on *SraRunTable.txt* and filereport_read_run_PRJNA559094_tsv.txt from ENA, import them into Excel to create a metadata.csv.
(For paired-end sequencing, the table includes four columns: **Run, fastq_aspera, Named1, and Named2**.<br>
Run represents the sample name, such as *SRR10885102*.<br>
fastq_aspera includes entries like: *fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/002/SRR10885102/SRR10885102_1.fastq.gz; fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/002/SRR10885102/SRR10885102_2.fastq.gz*.<br>
Named1 entries look like *RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885102_pairend_1.fastq.gz*,<br>
Named2 entries look like *RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885102_pairend_2.fastq.gz*.<br>



| Run         | fastq_aspera                                                 | Named1                                                       | Named2                                                       |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SRR10885102 | fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/002/SRR10885102/SRR10885102_1.fastq.gz;fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/002/SRR10885102/SRR10885102_2.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885102_pairend_1.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885102_pairend_2.fastq.gz |
| SRR10885103 | fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/003/SRR10885103/SRR10885103_1.fastq.gz;fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/003/SRR10885103/SRR10885103_2.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885103_pairend_1.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885103_pairend_2.fastq.gz |
| SRR10885106 | fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/006/SRR10885106/SRR10885106_1.fastq.gz;fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/006/SRR10885106/SRR10885106_2.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_Control perturbations reduced  set_SRR10885106_pairend_1.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_Control perturbations reduced  set_SRR10885106_pairend_2.fastq.gz |
| SRR10885107 | fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/007/SRR10885107/SRR10885107_1.fastq.gz;fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/007/SRR10885107/SRR10885107_2.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885107_pairend_1.fastq.gz | RNA-Seq_GSE135497_Homo sapiens_K562_None_SRR10885107_pairend_2.fastq.gz |
<br>

- **Step 3** <br>
Then, vim aspera_download.py  change the .openssh to your path **`ASPERA_KEY = '/path/to/your/asperaweb_id_dsa.openssh'`**, save it and do "`python aspera_download.py`" in the shell：<br><br>

**All rights reserved by Kurejacky.**
