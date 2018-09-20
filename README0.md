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

2. Transpose genotype data so they have all the SNPs in rows (first column) which can join with the SNP_position.txt  
	`awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt`	 
	`awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt` 

3. Extracting columns of SNP-ID, Chromosome, and Position from the SNP_position.txt  
	`cut -f1,3,4 snp_position.txt > snp_position_f134.txt` 

4. Check line number after transposition:  
	`wc -l snp_position_f134.txt transposed_maize_genotypes.txt transposed_teosinte_genotypes.txt`  
     984 `snp_position_f134`.txt  
     986 `transposed_maize_genotypes`.txt  
     986 `transposed_teosinte_genotypes`.txt  
    2956 total  
 
5. Files needed to be sorted by the common column before conducting join
	`sort -c -k1,1 transposed_maize_genotypes.txt` #verifies if already sorted  
	`sort -k1,1 transposed_maize_genotypes.txt > maize_genotypes_sorted.txt` #sort by the first column
	`sort -c -k1,1 transposed_teosinte_genotypes.txt` #verifies if already sorted  
	`sort -k1,1 transposed_teosinte_genotypes.txt > teosinte_genotypes_sorted.txt` #sort by the first column  
	`sort -c -k1,1 snp_position_f134.txt` #verifies if already sorted  
	`sort -k1,1 snp_position_f134.txt > snp_position_f134_sorted.txt` #sort by the first column  

6. Join the genotype files with SNP data, because we want to have SNP_ID, Chromosome, and Position in the first few columns, we will put SNP_position_f134_sorted.txt first when joining.
	`join -1 1 -2 1 -t $'\t' snp_position_f134_sorted.txt maize_genotypes_sorted.txt > snp_maize_joined.txt`
	`join -1 1 -2 1 -t $'\t' snp_position_f134_sorted.txt teosinte_genotypes_sorted.txt > snp_teosinte_joined.txt`

7. Verify the join results
	`wc -l snp_position_f134_sorted.txt maize_genotypes_sorted.txt snp_maize_joined.txt`
	`wc -l snp_position_f134_sorted.txt teosinte_genotypes_sorted.txt snp_teosinte_joined.txt`

8. Separating joined file by chromosome and sorting by increasing position, missing values represented by "?"
	`awk '$2 ~ 1' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr1_incP.txt`  
	`awk '$2 ~ 2' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr2_incP.txt`     
	`awk '$2 ~ 3' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr3_incP.txt`  
	`awk '$2 ~ 4' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr4_incP.txt`  
	`awk '$2 ~ 5' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr5_incP.txt`  
	`awk '$2 ~ 6' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr6_incP.txt`  
	`awk '$2 ~ 7' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr7_incP.txt`  
	`awk '$2 ~ 8' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr8_incP.txt`  
	`awk '$2 ~ 9' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr9_incP.txt`  
	`awk '$2 ~ 10' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr10_incP.txt` 
9. Use the files from last step and sort by decreasing position, missing values replaced by "-" 
	`sort -k3,3nr snp_maize_joined_chr1_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr1_decP-mi.txt`  
	`sort -k3,3nr snp_maize_joined_chr2_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr2_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr3_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr3_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr4_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr4_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr5_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr5_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr6_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr6_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr7_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr7_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr8_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr8_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr9_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr9_decP-mi.txt`   
	`sort -k3,3nr snp_maize_joined_chr10_incP.txt | sed 's/?/-/g' > snp_maize_joined_chr10_decP-mi.txt`   
10. Same as steps 8&9 for teosinte  
 
11. Extracting SNPs with unknown position
	`grep "unknown" snp_maize_joined.txt > snp_maize_joined_unknownPosition.txt`
	`grep "unknown" snp_teosinte_joined.txt > snp_teosinte_joined_unknownPosition.txt`  
12. Extracting SNPs with multiple positions
	`grep "multiple" snp_maize_joined.txt > snp_maize_joined_MultiPosition.txt`
	`grep "multiple" snp_teosinte_joined.txt > snp_teosinte_joined_MultiPosition.txt`

`awk 'BEGIN {i=0}; {i += 1}; $2 ~ i {print $0 > SNP_maize_chr/i/.txt};' snp_maize_joined.txt`

`awk 'BEGIN { for (i=1; i<=10; ++i) 

`awk '$2 ~ i { feature[$2] +=1


for ((i=1;i<=10; i+=1)); do awk '$2 == $i' snp_maize_joined.txt > snp_maize_joined_chr"$i"_incP_test.txt ; done
for ((i=1;i<=10; i+=1)); do awk '$2 == $i {print $0 > "snp_maize_joined_chr'$i'_incP_test.txt"}' snp_maize_joined.txt; done

for ((i=1;i<=10; i+=1)); do (awk '$2 == $i {print $0}' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr"$i"_incP_test.txt); done











$ awk '{for (i=1; i<=10; i++) if ($2==i && i != 10) print $0 > "teosinte_genotypes_chr0"i".txt"; else if ($2==i && i == 10) print $0 > "teosinte_genotypes_chr"i".txt";}' rmMS_sorted_joined_teosinte_genotypes.txt