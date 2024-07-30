
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
```r

This is a simple r code

setwd("/path/to/downloadfile/GSE135497")

data <- read.table("SraRunTable.txt",sep=",",header = T,check.names = F)
t1 <- read.table('filereport_read_run_PRJNA559094_tsv.txt',sep = "\t",check.names = F,header = T)
table <- t1
colnames(data)
colnames(table)
data_select <- data[,c("Run","Assay Type","Organism","source_name" ,"perturbations","cell_line"  )]

merge<-merge(data_select,table,by.x="Run",by.y = "run_accession")
colnames(merge)
# Custom rename the fastq file names.
merge$Named1<-paste0("scRNAseq","_GSE135497_",merge$perturbations,"_",merge$Organism,"_",merge$cell_line,"_",merge$Run,"_pairend_1.fastq.gz")
merge$Named2<-paste0("scRNAseq","_GSE135497_",merge$perturbations,"_",merge$Organism,"_",merge$cell_line,"_",merge$Run,"_pairend_2.fastq.gz")
# Ensure that the file names do not contain symbols such as '\, /, +, -, :'
library(stringi)
#here, i replace those to "_".
patterns <- c(" ", "\\(", "\\)","\\\\",'/',":","\\+",'\\-')
replacements <- c(rep("_", 8))
merge$Named1 <- stri_replace_all_regex(merge$Named1, patterns, replacements, vectorize_all = FALSE)
merge$Named2 <- stri_replace_all_regex(merge$Named2, patterns, replacements, vectorize_all = FALSE)

colnames(merge)
merge_s<-merge[,c('Run','fastq_aspera','Named1','Named2')]
write.csv(merge,"metadata_all.csv",row.names = F)
write.csv(merge_s,"metadata.csv",row.names = F)


```
<br><br>

| Run         | fastq_aspera                                                 | Named1                                                       | Named2                                                       |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SRR10885102 | fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/002/SRR10885102/SRR10885102_1.fastq.gz;fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/002/SRR10885102/SRR10885102_2.fastq.gz | scRNAseq_GSE135497_None_Homo_sapiens_K562_SRR10885102_pairend_1.fastq.gz | scRNAseq_GSE135497_None_Homo_sapiens_K562_SRR10885102_pairend_2.fastq.gz |
| SRR10885103 | fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/003/SRR10885103/SRR10885103_1.fastq.gz;fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/003/SRR10885103/SRR10885103_2.fastq.gz | scRNAseq_GSE135497_None_Homo_sapiens_K562_SRR10885103_pairend_1.fastq.gz | scRNAseq_GSE135497_None_Homo_sapiens_K562_SRR10885103_pairend_2.fastq.gz |
| SRR10885104 | fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/004/SRR10885104/SRR10885104_1.fastq.gz;fasp.sra.ebi.ac.uk:/vol1/fastq/SRR108/004/SRR10885104/SRR10885104_2.fastq.gz | scRNAseq_GSE135497_None_Mus_musculus_1_1_1_Mix_of_3T3_cells__Embryonic_Stem_Cells_and_Neutrophils__all_from_mouse__SRR10885104_pairend_1.fastq.gz | scRNAseq_GSE135497_None_Mus_musculus_1_1_1_Mix_of_3T3_cells__Embryonic_Stem_Cells_and_Neutrophils__all_from_mouse__SRR10885104_pairend_2.fastq.gz |
<br>

- **Step 3** <br>
Then, vim aspera_download.py  change the .openssh to your path **`ASPERA_KEY = '/path/to/your/asperaweb_id_dsa.openssh'`**, save it and do "`python aspera_download.py`" in the shell：<br><br>
```python
# This is aspera_download.py
import csv
import os
import subprocess

ASPERA_KEY = '/home/wuj/anaconda3/etc/asperaweb_id_dsa.openssh'
# path to your Aspera Connect'key

script_dir = os.path.dirname(os.path.abspath(__file__))
data_dir = os.path.join(script_dir, 'data')#you need to mkdir *data* folder
metadata_file = os.path.join(script_dir, 'metadata.csv')

# re download
MAX_RETRIES = 5

def download_with_retries(ascp_cmd, temp_file):
    for attempt in range(MAX_RETRIES):
        try:
            # Execute ascp command
            result = subprocess.run(ascp_cmd, check=True)
            if os.path.exists(temp_file):
                print(f"Downloaded {temp_file} successfully.")
                return True
            else:
                print(f"Error: {temp_file} does not exist after download attempt {attempt + 1}.")
        except subprocess.CalledProcessError as e:
            print(f"Error downloading file on attempt {attempt + 1}: {e}")
    return False

with open(metadata_file, 'r', encoding='utf-8-sig') as csvfile:
    reader = csv.DictReader(csvfile)
    print("CSV Columns:", reader.fieldnames)
    for row in reader:
        run_id = row['Run']
        fastq_aspera = row['fastq_aspera'].split(';')
        newname1 = os.path.join(data_dir, row['Named1'])
        newname2 = os.path.join(data_dir, row['Named2'])
        link1 = f"era-fasp@{fastq_aspera[0]}"
        link2 = f"era-fasp@{fastq_aspera[1]}"
        temp_file1 = os.path.join(data_dir, run_id + "_1.fastq.gz")
        temp_file2 = os.path.join(data_dir, run_id + "_2.fastq.gz")

        # Prepare ascp commands
        ascp_cmd1 = ['ascp', '-i', ASPERA_KEY, '-QT', '-k', '1', '-v', '-l', '300m', '-P', '33001', link1, temp_file1]
        ascp_cmd2 = ['ascp', '-i', ASPERA_KEY, '-QT', '-k', '1', '-v', '-l', '300m', '-P', '33001', link2, temp_file2]

        # Print the commands for debugging
        print("Executing:", ' '.join(ascp_cmd1))
        print("Executing:", ' '.join(ascp_cmd2))

        # Attempt to download the files with retries
        if download_with_retries(ascp_cmd1, temp_file1):
            os.rename(temp_file1, newname1)
            print(f"Renamed {temp_file1} to {newname1}")
        else:
            print(f"Failed to download {temp_file1} after multiple attempts.")

        if download_with_retries(ascp_cmd2, temp_file2):
            os.rename(temp_file2, newname2)
            print(f"Renamed {temp_file2} to {newname2}")
        else:
            print(f"Failed to download {temp_file2} after multiple attempts.")

```
<br>

**All rights reserved by Kurejacky.**
