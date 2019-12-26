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
## Genotyping and call-rate check
Should any sample have failed the quality check at the previous step, it would have been removed for this genotyping step. The genotying step performs the crucial task to convert intensity values into nucleotide calls. The apt command depends on the model and version of the chip. Most manuals of apt-tools focus con apt-probeset-genotype. This command's output include the calls.txt file along with a report.txt. The latter file has several lines beginning with # (comments) at the header and then is a row-per-sample file. Columns hold several values. The third one is the call-rate. Samples with a call_rate below 0.97 should be filtered out and Genotyping rerun on the remaining samples.
```bash
apt-genotype-axiom \
    --analysis-files-path  Axiom_SpainAnalysisAll/\
    --dual-channel-normalization true \
    --analysis-name All \
    --chip-type Axiom_SpainBA \
    --enable true \
    --artifact-reduction-clip 0.4 \
    --artifact-reduction-close 2 \
    --artifact-reduction-open 2 \
    --artifact-reduction-fringe 4 \
    --artifact-reduction-cc 2 \
    --sketch-target-scale-value 1000 \
    --sketch-size 50000 \
    --brlmmp-CM 1 \
    --brlmmp-bins 100 \
    --brlmmp-mix 1 \
    --brlmmp-bic 2 \
    --brlmmp-lambda 1.0 \
    --brlmmp-HARD 3 \
    --brlmmp-SB 0.75 \
    --brlmmp-transform MVA \
    --brlmmp-copyqc 0.00000 \
    --brlmmp-wobble 0.05 \
    --brlmmp-MS 0.15 \
    --brlmmp-copytype -1 \
    --brlmmp-clustertype 2 \
    --brlmmp-ocean 0.00001 \
    --brlmmp-CSepPen 0.1 \
    --brlmmp-CSepThr 4 \
    --snp-priors-input-file Axiom_SpainBA.r2.generic_prior.txt \
    --probeset-ids Axiom_SpainBA.r2.step1.ps \
    --cdf-file Axiom_SpainBA.r2.cdf \
    --special-snps Axiom_SpainBA.r2.specialSNPs \
    --x-probes-file Axiom_SpainBA.r2.chrXprobes \
    --y-probes-file Axiom_SpainBA.r2.chrYprobes \
    --sketch-target-input-file Axiom_SpainBA.r2.AxiomGT1.sketch \
    --igender-female-threshold 0.54 \
    --igender-male-threshold 1.0 \
    --allele-summaries true \
    --snp-posteriors-output true \
    --out-dir /media/egarmo/Extreme\ SSD/outAll \
    --cel-files listacelAll.txt \
    --log-file ./apt-genotype-axiomAll.log

grep call_rate All.report.txt
awk -F '\t' '$3<0.' All.report.txt 
```
The grep command checks whether call_rate is actually the third column in the report.txt file. The last command produces only lines beginning with #, it means that no sample has a call_rate below 0.97, so genotyping should not be repeated. 
The output of the apt-genotype-axiom command includes the actual genotyping in the calls.file. Further processing will be explained in the following section. Other output files like summary.txt  snp-posteriors will be used later for targeted quality check of selected variants, after association study in plink. 
## Exporting genotype results for plink input
The outpuf from the previous step that holds genotypes needs to be converted to the formats of association analysis software. The conversion for plink export takes the following code
```bash
apt-format-result \
--calls-file /media/egarmo/Extreme\ SSD/outAll/All.calls.txt \
--annotation-file Axiom_SpainBA/Axiom_SpainBA.na35.r2.a2.annot.db/Axiom_SpainBA.na35.r2.a2.annot.db \
--export-plink-file /media/egarmo/Extreme\ SSD/outAll/All.plink.ped
```
