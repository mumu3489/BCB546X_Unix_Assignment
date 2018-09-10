#Qi Mu - BCB546X Unix Assignment
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

      
##Date Processiing
