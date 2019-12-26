# GWAS
Code and data to perform GWAS on Affymetrix Axiom Genotype chip
## Preprocessing
The aim of this part of code is to process raw intensity files, as direct output from the genotyping machine, into files to be used as input for general open source genetic association software as PLINK. 
```bash
apt-geno-qc \
--analysis-files-path Axiom_SpainAnalysis \
--xml-file Axiom_SpainAnalysis/Axiom_SpainBA.r2.apt-geno-qc.AxiomQC1.xml \
--cdf-file Axiom_SpainBA.r2.cdf \
--qca-file Axiom_SpainBA.r2.qca \
--qcc-file Axiom_SpainBA.r2.qcc \
--chrX-probes Axiom_SpainAnalysis/Axiom_SpainBA.r2.chrXprobes \
--chrY-probes Axiom_SpainAnalysis/Axiom_SpainBA.r2.chrYprobes \
--cel-files listacelAll.txt \
--out-dir data \
--out-file aptGenoQcAll.txt \
--log-file aptGenoQcAll.log
#Segun el manual de Affymetrix el control de calidad requiere que el campo 'axiom-dishqc-DQC' tenga valor superior a 0.82 para cada muestra. Es columna decimooctava (18). Comprobación en la línea siguiente
cut -f18 aptGenoQcAll.txt|head
#Filtrado a un fichero de los valores 
awk -F '\t' '$18<0.82' aptGenoQcAll.txt >aptGenoQcAll_Filter0.82.txt
cat aptGenoQcAll_Filter0.82.txt
```
