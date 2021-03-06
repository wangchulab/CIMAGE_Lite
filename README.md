# CIMAGE 2.0
Release version of CIMAGE, scripts for quantification only. (no web server)

Contact: chuwang@pku.edu.cn jinjungao@uchicago.edu or wendao@pku.edu.cn

## Requirements

R (tested on 3.4.1 and 3.4.4)

```
sudo apt install r-base r-cran-ncdf4
```

Python (tested on version 2.7)

Perl (tested on perl5 version 18 and 26)

Other libray

```
sudo apt install libxml2-dev libnetcdf-dev zlib1g-dev
```

XCMS (tested on version 1.51.1 and 1.52.0)

```
source("https://bioconductor.org/biocLite.R")
BiocInstaller::biocLite(c("xcms"))
```

## Installation

Add the path of CIMAGE to environment variable: **CIMAGE_PATH**

Add the path of CIMAGE/Shell to environment variable: **PATH**

## Usage

### Input:
A. For LC-MS/MS data, CIMAGE takes .mzXML files as input, which can be easily converted from raw files by variouty of tools like [ReAdW](http://tools.proteomecenter.org/wiki/index.php?title=Software:ReAdW) and [RawConverter](http://fields.scripps.edu/rawconv/).

B. Before CIMAGE quantification, database search should be performed first and the output file is then used as the input for CIMAGE. CIMAGE can be connected with variety of database search engines (ProLuCID, Mascot, Andromeda, pFind and MSFragger).

### Analysis workflow:

The demo data and results can be downloaded from the following link: https://drive.google.com/drive/folders/1yLNTHNciC9vllmzdA7vuPHmrxjjQ04ce?usp=sharing

A. Make a folder such as "isoTOP-ABPP", upload LC-MS/MS data in mzXML format and create a folder (e.g. dta).

B. Before quantification, you need to edit your cimage.params file to point to the right light/heavy chemical composition files. Template cimage.params file and common light/heavy chemical composition files can be found in the params folder:

```
IA _tev – isoTOP-ABPP with IA-alkyne probe & Tev tag
SILAC – SILAC with Arg10 & Lys8
```

Light/heavy tables are strictly tab-delimited text files, so it is better to use EXCEL or notepad++ to import, edit and then export it.

C. Enter the “dta” folder and upload the database search results. In database search process, the LC-MS/MS data should be searched for light and heavy labelled peptides separately to generate two results containing either light or heavy peptides. By default, ProLuCID is recommended as it is flexible to fit different types of quantitative strategies, but you can also feel free to use your comfortable one.

 - For ProLuCID, the DTASelect-filter.txt file should be renamed as DTASelect-filter_PREFIX_ISO.txt, in which PREFIX represents the prefix of your mzXML file and ISO represent isotope state. For example:
```
DTASelect-filter_20181026_1TO1_heavy.txt
DTASelect-filter_20181026_1TO1_light.txt
```
 - For Andromeda (Maxquant), the evidence.txt file should be renamed as evidence_light.txt or evidence_heavy.txt
 - For Mascot, the database search results in csv format are uploaded directly
 - For pFind, the spectra file should be rename as pFind_light.spectra or pFind_heavy.spectra
 - For MSFragger light and heavy label need to be set as two varible modifications and search only once

D. go into the "dta" folder and run the "cimage" program by typing:
 - For ProLuCID
```
cimage /path/of/cimage.params.IA_tev 20181026_1TO1
```
 - For Mascot
```
SILAC: cimage SILAC your-cimage-params-file PREFIX
ABPP: cimage_mascot 1mod paramfile modify_AA light_number heavy_number PREFIX 
cimage_mascot 2mod parameter_file modify_AA1 light_number1 heavy_number1 modify_AA2 light_number2 heavy_number2 PREFIX
#PS: modify_AA represents the labelled amino acid (e.g., the modify_AA should be “C” when labelling cysteine). light_number and heavy_number are the number allocated to light and heavy modifications.
```
 - For Andromeda
```
cimage_andromeda your-cimage-params-file MOD PREFIX 
#In which MOD is the modification symbol. For example, in PEPTI(ga)DE, the GlcNAc modification is represented by ga, so the MOD should be ‘ga’
```
 - For pFind and MSFragger: comming soon ...

E. If it runs fine, it will generate a "output" folder in which there will be a "to_excel" text file for your editing in excel and a folder of "PNG" containing all individual graphic files. During CIMAGE processing, the screen will also print the progress
```
Total number of pages are 2663
working on pages 1--500
working on pages 501--1000
...
```
After this step, the dta folder should contain:
```
DTASelect-filter_20181026_1TO1_heavy.txt
DTASelect-filter_20181026_1TO1_heavy.txt.tagged
DTASelect-filter_20181026_1TO1_light.txt
DTASelect-filter_20181026_1TO1_light.txt.tagged
Rplots.pdf
all_scan.table
cross_scan.table
findMs1AcrossSetsFromDTASelect.Rout
ipi_name.table
output/
```

F. Move to upper folder and generate a html version of the result by typing:
```
cimage_combine [by_protein] dta
```
It will generate a combine_dta.html and a raw text file combine_dta.txt. Skip the by_protein option if you would like to by default group identified peptides by their sequence instead of their parent proteins.

### Other modules
A. ***add_preview***, to enhance cimage html output with preview script. For detailed usage, please refer to the “README.md” in cimage_preview folder (CIMAGE feature https://github.com/wangchulab/CIMAGE).

B. ***cimage_compare***, to compare ratios obtained by multiple different cimage runs, you can run cimage_compare program by typing:
```
cimage_compare [by_protein] file1 column1 outname1 file2 column2 outname2 ...
```
in which "file1" and "file2" are full names (with paths) of the two combined_dta.txt files to be compared, "column1" and "column2" are names of ratio columns in each combined_dta.txt file, and "outname1" and "outname2" are names of ratios columns when they are output into a tab-delimited text file side by side that you can import to EXCEL for further analysis. for example, if you like to compare protein silac ratios (column "mr.set_1" in combined_dta.txt files) obtained from two experiments "exp1" and "exp2", the command to run would be: 
```
cimage_compare [by_protein] /your-folder/exp1/combined_dta.txt set_1 my_exp1 /your-folder/exp2/combined_dta.txt set_1 my_exp2
```

C. ***pdf_generator***, to generate high quality pdf plot for specific quantified peptide:
```
pdf_generator your-cimage-params-file PREFIX ID
```
the “ID” is entry No. showed in the plotted picture.

D. ***extract_LC_peaks.py***, extract features from MS1 file for peaking paired peaks
```
extract_LC_peaks.py 20181026_1TO1_01.ms1 0.02
python check_pair_by_envelope.py env_[id].dat
```
