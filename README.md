# Assignment Documentation

`README.md` == This file; provides a detailed description of the project, workflow, and repository structure.

The repository is organized into specific directories to separate raw data, intermediate processing files, and final results:

Contains the 22 final output files for Maize.

`*_increasing_chr` == 10 files sorted by increasing position.

`*_decreasing_chr` == 10 files sorted by decreasing position.

`*_unknown_multiple_position` == 1 file for unknown positions and 1 file for multiple positions.

* This makes a the 22 final output files for Maize and 22 for teosinte.

metadata == Contains the original raw data files (fang_et_al_genotypes.txt and snp_position.txt) as well as intermediate files created during the join and transpose steps.

backup_files == A safety directory containing copies of the original raw data to prevent accidental data loss during processing.

## Data inspection
 
### listing files 
* After changing directory (`cd`) into the directory containing my assignment files i listed the `ls -lh` to list the files in my current directory with human readable option `-lh`
#command
`ls -lh *.txt`
#standard output
-rw-rw-r--. 1 ameyawp domain users 11M Feb  6 13:13 fang_et_al_genotypes.txt
-rw-rw-r--. 1 ameyawp domain users 81K Feb  6 13:13 snp_position.txt
-rw-rw-r--. 1 ameyawp domain users 11M Feb  6 13:13 transposed_genotypes.txt

* I also further inpected the file size using `du -h` on the assigement .txt files. 
#comand:
`du -h *.txt`
#standard output:
6.7M    fang_et_al_genotypes.txt
49K     snp_position.txt
3.7M    transposed_genotypes.txt

### making backup directory
* i copied the original files into a new directory i created just to have a copy of my files incase my commands mistakenly mess up the files
#commands
`mkdir backup_files`
`cp *.txt backup_files`

* check to confirm i have backup files sucessfully copied and looking at the file size i confirms they were sucessfuy copied
#command
ls -lh backup_files/


### fang_et_al_genotypes.txt inspection
* i first used `head` to look at the first 10 line and it was confusing so i checked on the first line and realised it has a lit of columns
#commands
`head fang_et_al_genotypes.txt`
`head -n1 fang_et_al_genotypes.txt` 

* i used tail to see the last 10 line in the file
#command
`tail fang_et_al_genotypes.txt`

* Now knowing that the file had several columns i checked the number of columns in the file.
#command
`awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt`
#standard output
986

* i used cut to look at the first 15 columns and pipe that into head to see only the first 10 rows to have an idea about what i am dealing with
#command
`cut -f1-15 fang_et_al_genotypes.txt | head`

* i used word count to count the number of rows in the file
#command
`wc -l fang_et_al_genotypes.txt`
#output
2783 fang_et_al_genotypes.txt

### snp_position.txt inspection
* similar commands used above were used to inspect the snp_position.txt file and i found that it also had `15` columns and `984` rows


## data processing 
 
### Splitting files into maize and teosinte
* For spitting the files into maize and teosinte there was the need to include the header which contains necessary information needed when during the joining process
`awk 'NR==1 || $3 ~ /ZMMIL|ZMMLR|ZMMMR/' fang_et_al_genotypes.txt > maize_genotypes.txt`
`awk 'NR==1 || $3 ~ /ZMPBA|ZMPIL|ZMPJA/' fang_et_al_genotypes.txt > teosinte_genotypes.txt`

### transposing maize_genotypes.txt and teosinte_genotypes.txt 
`awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt`
`awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt`

### sorting  and joining 
* to join both `transposed_maize_genotypes.txt` and `transposed_teosinte_genotypes.txt` to `snp_position.txt` the common column `Snp_ID` had to be sorted and the files were redirected in to a new file
* because the common column that need to be sorted had alphabets, numbers uppercase and lowercases i used the options `-f` to ignore case, -K1 to sort on the first fieled `common column` and `-V` for natural sort of numbers within text
* `tail -n+2` and `tail -n+4` were used to remove the headers so that i sort on only the `samples.snp_position.txt` had one header line and `transposed_genotypes.txt` had 3 header lines hence `-n+2` and `-n+4` respectively. 
#command
`tail -n+2 snp_position.txt | sort -f -k1 -V > snp_position_sorted.txt`
`tail -n+4 transposed_maize_genotypes.txt | sort -f -k1 -V > transposed_maize_genotypes_sorted.txt`
`tail -n+4 transposed_teosinte_genotypes.txt | sort -f -k1 -V > transposed_teosinte_genotypes_sorted.txt`

* checked the number of rows in all sorted files that were going to be joined
`wc -l *sorted*`
#standard output
983 snp_position_sorted.txt
983 transposed_sorted_maize_genotypes.txt
983 transposed_sorted_teosinte_genotypes.txt
2949 total

* inspect sorted files with cut, head and to make sure the `SNP_ID` in both files are sorted in the same format
#command
`cut -f1 snp_position_sorted.txt | head`
`cut -f1 transposed_sorted_maize_genotypes.txt | head`
`cut -f1 transposed_sorted_teosinte_genotypes.txt | head`
# Standard Output for all files
abph1.20
abph1.22
ae1.3
ae1.4
ae1.5
an1.4
ba1.6
ba1.9
bt2.5
bt2.7
* similar inpection was done with tail to look at the ends on the sorted files
#commands
`cut -f1 snp_position_sorted.txt | tail`
`cut -f1 transposed_genotypes_sorted.txt | tail`

* check sorting 
#commands
`sort -f -k1 -V -c snp_position_sorted.txt`
`sort -f -k1 -V -c transposed_sorted_maize_genotypes.txt`
`sort -f -k1 -V -c transposed_sorted_teosinte_genotypes.txt`

### subsetting the needed columns from snp_position_sorted.txt to be used for the joining process
* i used `cut` to extract column 1,3 and 4 which is SNP_ID", "Chromosome" and "Position" respectively from `snp_position_sorted.txt` file
`cut -f1,3,4 snp_position_sorted.txt > snp_position_sorted_subset.txt`

### joining snp_position_sorted.txt and transposed_genotypes_sorted.txt
* after sucessfully sorting  and subsetteing the nee columns the files they were joined based on the common columns `SNP_ID` in column 1. i also used option -t $`\t` to introduce a tab delimiter. 
#command
`join -1 1 -2 1 -t $'\t' snp_position_sorted_subset.txt transposed_sorted_maize_genotypes.txt > merged_maize_genotypes.txt`
`join -1 1 -2 1 -t $'\t' snp_position_sorted_subset.txt transposed_sorted_teosinte_genotypes.txt > merged_teosinte_genotypes.txt`

* check joining 
#command
`wc -l snp_position_sorted_subset.txt transposed_sorted_* merged*` 
#standard output
983 snp_position_sorted_subset.txt
983 transposed_sorted_maize_genotypes.txt
983 transposed_sorted_teosinte_genotypes.txt
983 merged_maize_genotypes.txt
983 merged_teosinte_genotypes.txt
4915 total


###subsetting into final 44 files

* i used a `for` loop where the variable `i` takes values from 1 to 10. i used awkto creat a variable named chromosome and assigns it the value of the loop variable `$i`
* `$2 == chromosome` to tell awk to look in the second column which is the chromosome number and piped into `sort -k3,3n` or `sort -k3,3nr` to sort the third column which is the chromosome position in increasing and decreasing order.
* the output was redirected into a new file and named planttype, the he chromosome and number, and if the positions is sorted in increasing or decreasing.
* awk was also used to subset file that had `unknown` and `multiple` positions. because after inspection i saw that both the chromosome and position had multiple positions i used `$2 == "multiple" || $3 == "multiple"` to select all multiple positions

#####maize
### spliting into 10 files, 1 for each chromosome and ordered in increasing position with missing data endoded by `?`
`for i in {1..10}; do awk -v chromosome="$i" '$2 == chromosome' merged_maize_genotypes.txt | sort -k3,3n > maize_chromosome"$i"_increasing.txt; done`

### spliting into 10 files, 1 for each chromosome and ordered in decreasing position with missing data endoded by `-`
`for i in {1..10}; do     awk -v chromosome="$i" '$2 == chromosome' merged_maize_genotypes.txt | sort -k3,3nr | sed 's/?/-/g' > maize_chromosome"$i"_decreasing.txt; done`

### 1 file with all SNPs with unknown position
`awk '$2 == "unknown"' merged_maize_genotypes.txt > maize_unknown_positions.txt`

### 1 file with all SNPs with multiple positions
`awk '$2 == "multiple" || $3 == "multiple"' merged_maize_genotypes.txt > maize_multiple_positions.txt`

#####teosinte
### spliting into 10 files, 1 for each chromosome and ordered in increasing position with missing data endoded by `?`
`for i in {1..10}; do awk -v chromosome="$i" '$2 == chromosome' merged_teosinte_genotypes.txt | sort -k3,3n > teosinte_chromosome"$i"_increasing.txt; done`
 
### spliting into 10 files, 1 for each chromosome and ordered in decreasing position with missing data endoded by `-`
`for i in {1..10}; do     awk -v chromosome="$i" '$2 == chromosome' merged_teosinte_genotypes.txt | sort -k3,3nr | sed 's/?/-/g' > teosinte_chromosome"$i"_decreasing.txt; done`

### 1 file with all SNPs with unknown position
`awk '$2 == "unknown"' merged_teosinte_genotypes.txt > teosinte_unknown_positions.txt`

### 1 file with all SNPs with multiple positions
`awk '$2 == "multiple" || $3 == "multiple"' merged_teosinte_genotypes.txt > teosinte_multiple_positions.txt`
