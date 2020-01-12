> First attempt to use HiC-Pro to finish a 3D genome analysis
> HiCPro document: http://nservant.github.io/HiC-Pro/

# Content
1. Install HiC-Pro and dependencies	
2. Download data and preprocess
3. Setting the configuration file
4. About the output
5. Visualization with HiCPlotter

# 1. Install HiC-Pro and dependencies
## 1.1 Install dependencies
#### What we need for hicpro
Dependency| function
--|--
bowtie2 | index
Python (>2.7 & <3) |with 4 packages following
pysam (>=0.8.3)|
bx-python(>=0.5.0)| 
numpy(>=1.8.2)| 
scipy(>=0.15.1) |
R|with packages RColorBrewer and ggplot2 (>2.2.1) 
g++ compiler | use "which" to check if we have
samtools (>1.1) |

#### how to install
1. python pip
2. **conda**: the most convenient

## 1.2 Download HiC-Pro
```
mkdir -p ~/biosoft/hicpro
cd ~/biosoft/hicpro
git clone https://github.com/nservant/HiC-Pro.git
```
## 1.3 System configuration
- to edit the config-install.txt file and manually defined the paths to dependencies
```
cat config-install.txt

# edit 
PREFIX =/home/zengjianming/biosoft/hicpro/bin
BOWTIE2_PATH =/home/zengjianming/miniconda3/envs/hic/bin/bowtie2
SAMTOOLS_PATH =/home/zengjianming/miniconda3/envs/hic/bin/samtools
R_PATH =/home/zengjianming/miniconda3/envs/hic/bin/R
PYTHON_PATH =/home/zengjianming/miniconda3/envs/hic/bin/python
CLUSTER_SYS =TORQUE # Resource management system of our server

# install
make configure
make install

# add hicpro to $PATH
vim .bashrc
export HICPRO_PATH=/public/home/liuj626/tools/biosoft/hicpro/bin/HiC-Pro_2.11.1/bin
export PATH=$HICPRO_PATH:$PATH

# check if install successfully 
HiC-Pro -h
```

# 2. Download data and preprocess
- Two parts: Experiment data and Reference genome
## 2.1 Experiment data
> Article:**3D genome of multiple myeloma reveals spatial genome disorganization associated with copy number variations**.*Nature Communications*
> SRX: SRX2208970
> SRR: SRR4341901-SRR4341904
- ==download sra toolkit==
```
# use sratool
prefetch SRR4341901

# transform to fa.gz
# sample must be placed in the same folder
fastq-dump --gzip --split-3 SRR4341901
```

## 2.2 Reference genome
> hg19
- to prepare **.bed** and **genome size** files
#### .bed file
```
$ cd /your/path/of/reference/
$ wget http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz
$ tar zvfx chromFa.tar.gz

# Notice that the download file is seperated in different chr
$ cat *.fa > hg19.fa
$ rm chr*.fa

# bowtie2 index：add bowtie2 to path before
$ bowtie2-build hg19.fa hg19.fa
# first "hg19.fa":input file; Second "hg19.fa": prefix of output file
# generate 6 ".bt2"

# .bed 
mkdir -p ~/data/project/hic/digest  # create new folder
cd ~/data/project/hic/digest

bin=/public/.../HiC-Pro_2.10.0/HiC-Pro/bin/utils/digest_genome.py  # use digest_genome.py to generate .bed file
$bin -r C^CATGG  -o  hg19.bed ../ref/hg19.fa 
```
#### genome size 
- If you use hg19 as reference, hicpro provides some config files(like ```chrom_hg19.sizes```) 

# 3. Setting the configuration file and Run
> **every time you run the pipeline you should create a configuration file.**See the manual for details about the configuration file

#### Step1: edit your config file
1. Copy and edit the configuration file "config-hicpro.txt" in your local folder. 
2. Put all input files in a rawdata folder. The input files have to be organized with a folder per sample.
3. Use ```qsub``` to submit your pbs script
> I have provide my file as example (see)
```
# the basic context of config file
HiC-Pro -i FULL_PATH_TO_RAW_DATA -o FULL_PATH_TO_OUTPUTS -c MY_LOCAL_CONFIG_FILE -p
```
#### Step2: qsub the files generated
- You will get message like this and do as the guidance.
```
Please run HiC-Pro in two steps :
1- The following command will launch the parallel workflow through 12 torque jobs:
qsub HiCPro_step1.sh
2- The second command will merge all outputs to generate the contact maps:
qsub HiCPro_step2.sh
```

# 4. Results
- The results provide:
1. mapping results
2. contact map(matrix form)

# 5. Visualization with HiCPlotter
#### Install HiCPlotter
> kcakdemir/HiCPlotter: https://github.com/kcakdemir/HiCPlotter/releases

#### Visualization
- Notice that **the output(.png) will be placed in home directory instead of the working directory**(Still don't find solution)
- It has lots of optional parameters
- I have uploaded my file as example (see:)
```
# basic grammar
python HiCPlotter.py -f file -n name -chr chrX -o output
```

