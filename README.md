#Qi Mu - BCB546X Unix Assignment
---------
##File Inspection

1. Looking at the top of the files:  
	`head -n 2 fang_et_al_genotypes.txt`  
	`head -n 2 snp_position.txt`

2. Looking at the bottom of the files:  
	`tail -n 2 fang_et_al_genotypes.txt`  
	`tail -n 2 snp_position.txt`

3. Looking at the file sizes:  
	`ls -lh fang_et_al_genotypes.txt snp_position.txt`   
or  
	`du -h fang_et_al_genotypes.txt snp_position.txt`  
 - 11M     fang_et_al_genotypes.txt  
 - 84K     snp_position.txt  

4. Count the number of lines, words, and characters of the files:  
	`wc fang_et_al_genotypes.txt snp_position.txt`   
or just looking at the number of lines of each file:  
	`wc -l fang_et_al_genotypes.txt snp_position.txt`  	 
 - 2783 fang_et_al_genotypes.txt  
 - 984 snp_position.txt  

5. Count the number of columns in each file:  
	`awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt`   
	`awk -F "\t" '{print NF; exit}' snp_position.txt`  
 - 986 fang_et_al_genotypes.txt  
 - 15 snp_position.txt  

6. Checking if there are non-ASCII character in both files:  
	`file fang_et_al_genotypes.txt snp_position.txt`  
 - fang_et_al_genotypes.txt: ASCII text, with very long lines, with CRLF line erminators  
 - snp_position.txt: ASCII text, with CRLF line terminators  

7. Summary  
	1) fang_et_al_genotype.txt  
 	- Size: 11M      
	- Row (samples) : 2783    
	- Col (SNPs) : 984  

	2) snp_position.txt  
	- Size: 84K  
	- Row (SNPs) : 984  
	- Col (Other info) : 15    
 
----------

      
##Date Processiing

1. Extracting teosinte and maize data from fang_et_al_genotype.txt    
	`awk '$3 ~ /Group|ZMMIL|ZMMLR|ZMMMR/' fang_et_al_genotypes.txt > maize_genotypes.txt`  
	`awk '$3 ~ /Group|ZMPBA|ZMPIL|ZMPJA/' fang_et_al_genotypes.txt > teosinte_genotypes.txt`  
  
	- Group was included to extract header as well  
  
2. Transpose genotype data so they have all the SNPs in rows (first column) which can join with the SNP_position.txt  
	`awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt`  
	`awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt`  
  
3. Extracting columns of SNP-ID, Chromosome, and Position from the SNP_position.txt  
	`cut -f1,3,4 snp_position.txt > snp_position_f134.txt`  
  
4. Check line number after transposition to make sure transposition was success    
	`wc -l snp_position_f134.txt transposed_maize_genotypes.txt transposed_teosinte_genotypes.txt`  
     984 `snp_position_f134`.txt  
     986 `transposed_maize_genotypes`.txt  
     986 `transposed_teosinte_genotypes`.txt  
    2956 total  
  
5. Files needed to be sorted by the common column before conducting join  
	- verifies if already sorted or not:  
	`sort -c -k1,1 transposed_maize_genotypes.txt`  
	- keepng the header (awk) and sort by the first column:   
	`awk 'NR<2 {print $0;next}{print $0| "sort -k1,1"}' transposed_maize_genotypes.txt > maize_genotypes_sorted_WH.txt`  
	- verifies if already sorted or not:  
	`sort -c -k1,1 transposed_teosinte_genotypes.txt`  
	- keepng the header (awk) and sort by the first column:  
	`awk 'NR<2 {print $0;next}{print $0| "sort -k1,1"}' transposed_teosinte_genotypes.txt > teosinte_genotypes_sorted_WH.txt`   
	- verifies if already sorted or not:  
	`sort -c -k1,1 snp_position_f134.txt` #verifies if already sorted   
	- keepng the header (awk) and sort by the first column:  
	`awk 'NR<2 {print $0;next}{print $0| "sort -k1,1"}' snp_position_f134.txt > snp_position_f134_sorted_WH.txt`   
  
6. Join the genotype files with SNP data, because we want to have SNP-ID, Chromosome, and Position in the first few columns, we will put SNP_position_f134_sorted_WH.txt first when joining  
	`sed 's/Sample_ID/SNP_ID/' maize_genotypes_sorted_WH.txt > maize_genotypes_sorted_WH1.txt`  
	`sed 's/Sample_ID/SNP_ID/' teosinte_genotypes_sorted_WH.txt > teosinte_genotypes_sorted_WH1.txt`  
	- sed was used to make sure the first cell are the same for two joining files so headers can be joined  
	`join -1 1 -2 1 -t $'\t' snp_position_f134_sorted_WH.txt maize_genotypes_sorted_WH1.txt > snp_maize_joined.txt`  
	`join -1 1 -2 1 -t $'\t' snp_position_f134_sorted_WH.txt teosinte_genotypes_sorted_WH1.txt > snp_teosinte_joined.txt`  
	- Headers were kept in the joined files (WH: With Header)   
  
7. Verify the join results, and get an idea how many lines are for each chromosome so we can make sure the future steps are correct  
	`wc -l snp_position_f134_sorted_WH.txt maize_genotypes_sorted_WH.txt snp_maize_joined.txt`  
	`wc -l snp_position_f134_sorted_WH.txt teosinte_genotypes_sorted_WH.txt snp_teosinte_joined.txt`  
  
	`cut -f2 snp_maize_joined.txt |sort -k2,2n | uniq -c | column -t`  
155  1  
53   10  
127  2  
107  3  
91   4  
122  5  
76   6  
97   7  
62   8  
60   9  
1    Chromosome  
6    multiple  
27   unknown  
	- `uniq -c` was used to count the number of specific chromosome.  
	- `column -t` was used to align the out put by columns.    

8. Separating joined file by chromosome and sorting by increasing position, missing values represented by "?" on both maize and teosinte files (need to put the header in)  
	`for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do (head -n1 snp_"$j"_joined.txt; awk -v awki="$i" '$2 == awki {print $0}' snp_"$j"_joined.txt | sort -k3,3n) > snp_"$j"_joined_chr"$i"_incP.txt; done; done`  
	- The inside loop is to separate chromosome 1 to 10, outside loop is to do both maize and teosinte.  
	- The `head` function is to extract the header line from joined file to new files.  
	- In `awk` commend, `$` and variables are represented by different rules compared to bash commend. So in order for `awk` to recognize the variable `i`, we need to create a new variable by `-v`, and I named it `awki`. So the whole table will be filtered by matching to 1-10 in the second column.  
	- `Sort` was used following `awk` to sort the filtered file by the third column (position).  
	- In `do` fuction, `awk` and `sort` are seperated by `|` , this is because the result from `awk` is piped to `sort`. While `;` was used to seperate `head` and `awk`, this is because they are parallel. `()` was used to indicate everything in it will be conducted together, in this case it to save them together in the new file.   
  
9. Separating joined file by chromosome and sorting by decreasing position, missing values replaced by "-" on both maize and teosinte files  
	`for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do (head -n1 snp_"$j"_joined.txt; awk -v awki="$i" '$2 == awki {print $0}' snp_"$j"_joined.txt | sort -k3,3nr | sed 's/?/-/g') > snp_"$j"_joined_chr"$i"_decP-mi.txt; done; done`  
	- Similar structure to #8.  
	- `Sort` was conducted in reverse way by adding `r`.  
	- `Sed` was added via pipe to replace the missing value by `-`.  

10. check if the files are correct by word counts  
	`for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do wc snp_"$j"_joined_chr"$i"_*.txt; done; done | sort -k4,4 | column -t`  
	- Inside loop is to seperate chromosome 1 to 10, outside loop is to do both maize and teosinte.  
	- Wild card * was used to select both increasing and decreasing sorted files  
	- `column -t` was used to align the out put by columns.  
   
11. Extracting SNPs with unknown position  
	`grep -E "(Chromosome|unknown)" snp_maize_joined.txt > snp_maize_joined_unknownPosition.txt`  
	`grep -E "(Chromosome|unknown)" snp_teosinte_joined.txt > snp_teosinte_joined_unknownPosition.txt`  
	- `-E` was used for `grep` to recognize extended regular expression: Chromosome or unknow. So header line containing Chromosome can be saved.  

12. Extracting SNPs with multiple positions  
	`grep -E "(Chromosome|multiple)" snp_maize_joined.txt > snp_maize_joined_MultiPosition.txt`  
	`grep -E "(Chromosome|multiple)" snp_teosinte_joined.txt > snp_teosinte_joined_MultiPosition.txt`  

13. Check if the files with unknown and multiple positions are correct by line counts and also checking column 2&3 make up  
	`wc -l snp_maize_joined_unknownPosition.txt snp_maize_joined_MultiPosition.txt snp_teosinte_joined_unknownPosition.txt snp_teosinte_joined_MultiPosition.txt`  
  
	`cut -f2 snp_maize_joined_unknownPosition.txt`  
	`cut -f2,3 snp_maize_joined_MultiPosition.txt`  

	`cut -f2 snp_teosinte_joined_unknownPosition.txt`  
	`cut -f2,3 snp_teosinte_joined_MultiPosition.txt`  
	- Headers were included, results are correct by looking at the how many lines for each in original joined file: (multiple was counted as multiple chromosomes and multiple positions in the same chromosomes)   
    28 `snp_maize_joined_unknownPosition.txt`  
    18 `snp_maize_joined_MultiPosition.txt `  
    28 `snp_teosinte_joined_unknownPosition.txt`  
    18 `snp_teosinte_joined_MultiPosition.txt`  
    92 `total`  

	`cut -f2,3 maize_results/snp_maize_joined_MultiPosition.txt | sort -k1,1n | column -t`  
	- To check the details of multiple SNPs
Chromosome  Position  
multiple  
multiple  
multiple  
multiple  
multiple  
multiple  
2 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
4 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
4 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
4 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
6 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
6 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
6 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
7 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
9 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
9 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
9 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; multiple  
  
14. Orginize files by creating results folder for maize and teosinte separately.  
	`mkdir maize_results teosinte_results source_files`  
	`mv snp_maize_joined_*.txt maize_results`  
	`mv snp_teosinte_joined_*.txt teosinte_results`  
	`mv fang_et_al_genotypes.txt snp_position.txt source_files`  
  
15. Make up: to remove SNPs that matches multiple positions in a fixed chromosome from all Chr1-Chr10 files. (SNPs that matching multiple position in for example chr1 was included in chr1 files, so this step is a make-up to remove those using `for loop`)  
	`for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do sed -i '/multiple/d' *_results/snp_"$j"_joined_chr"$i"_*.txt; done; done `  

	- Check by word counts for each file:  
	`for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do wc *_results/snp_"$j"_joined_chr"$i"_*.txt; done; done | sort -k4,4 | column -t  

	- The results show that Chr2 and 7 reduced one line, and Chr4,6,and 9 reduced 3 lines. So the multiple position SNPs in there chromosomes are removed.  

