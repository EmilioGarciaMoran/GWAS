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
--cel-files listacelAlvarez1.txt \
--out-dir data \
--out-file aptGenoQcAlvarez1.txt \
--log-file aptGenoQcAlvarez1.log
#Segun el manual de Affymetrix el control de calidad requiere que el campo 'axiom-dishqc-DQC' tenga valor superior a 0.82 para cada muestra. Es columna R en Excel, orden16

#awk -F '\t' '$2<0.86 && $8<0.4' aptGenoQcAlvarez1.txt >aptGenoQcAlvarez1_Filtered.txt  OLD VERSION
awk -F '\t' '$16<0.82' aptGenoQcAlvarez2.txt >aptGenoQcAlvarez2_Filtered.txt
```
