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
```python
# This is aspera_download.py
import csv
import os

ASPERA_KEY = '/home/wuj/anaconda3/etc/asperaweb_id_dsa.openssh'
#path to your Aspera Connect'key

script_dir = os.path.dirname(os.path.abspath(__file__))
data_dir = os.path.join(script_dir, 'data')
metadata_file = os.path.join(script_dir, 'metadata.csv')

# download retry
MAX_RETRIES = 10

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

with open('metadata.csv', 'r',encoding='utf-8-sig') as csvfile:
    reader = csv.DictReader(csvfile)
    print("CSV Columns:", reader.fieldnames)
	for row in reader:
    number = row['Run']
    fastq_aspera = row['fastq_aspera'].split(';')
    newname1 = os.path.join(data_dir, row['Named1'])
    newname2 = os.path.join(data_dir, row['Named2'])
    #
    link1 = f"era-fasp@{fastq_aspera[0]}"
    link2 = f"era-fasp@{fastq_aspera[1]}"
    print (link1)
    print (link2)
    #download filename
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
