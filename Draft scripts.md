for ((i=1;i<=10; i+=1)); do awk -v awki="$i" '$2 == awki {print $0}' snp_maize_joined.txt | sort -k3,3n > snp_maize_joined_chr"$i"_incP.txt; done

for ((i=1;i<=10; i+=1)); do wc -l snp_maize_joined_chr"$i"_incP.txt; done


for ((i=1;i<=10; i+=1)); do awk -v awki="$i" '$2 == awki {print $0}' snp_maize_joined.txt | sort -k3,3nr | sed 's/?/-/g' > snp_maize_joined_chr"$i"_decP-mi.txt; done

for ((i=1;i<=10; i+=1)); do wc -l snp_maize_joined_chr"$i"_decP-mi.txt; done


for ((i=1;i<=10; i+=1)); do awk -v awki="$i" '$2 == awki {print $0}' snp_teosinte_joined.txt | sort -k3,3n > snp_teosinte_joined_chr"$i"_incP.txt; done

for ((i=1;i<=10; i+=1)); do wc -l snp_teosinte_joined_chr"$i"_incP.txt; done


for ((i=1;i<=10; i+=1)); do awk -v awki="$i" '$2 == awki {print $0}' snp_teosinte_joined.txt | sort -k3,3nr | sed 's/?/-/g' > snp_teosinte_joined_chr"$i"_decP-mi.txt; done

for ((i=1;i<=10; i+=1)); do wc -l snp_teosinte_joined_chr"$i"_decP-mi.txt; done


----------

for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do awk -v awki="$i" '$2 == awki {print $0}' snp_"$j"_joined.txt | sort -k3,3n > snp_"$j"_joined_chr"$i"_incP_test.txt; done; done 

for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do awk -v awki="$i" '$2 == awki {print $0}' snp_"$j"_joined.txt | sort -k3,3nr | sed 's/?/-/g' > snp_"$j"_joined_chr"$i"_decP-mi_test.txt; done; done 

for j in "maize" "teosinte"; do for ((i=1;i<=10; i+=1)); do wc -l snp_"$j"_joined_chr"$i"_*_test.txt; done; done | sort -k1,1n