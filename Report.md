# Metagenomik

# Graphical Abstract
<img src="https://github.com/fhwnmatt/Metagenomics/blob/master/figures/flowchart_abstract.png">

# 1. General Introduction

Metagenomik ist ein Forschungsgebiet, das mit molekularbiologischen Methoden die Gesamtheit der Mikroorganismen eines Biotops erfasst. Dabei wird nicht nur die gesamte genomische Information der Mikroorganismen eines Lebensraumes untersucht, sondern auch die Umwelt-DNA dieses Lebensraumes zum Zeitpunkt der Untersuchung. Dies ist wichtig, um auch die metabolischen und enzymatischen Aktivitäten in einer Probe (z.B. einer Darmprobe oder ein Moospolster) zu verstehen. Das Verständnis der Abläufe der Vorgänge kann für medizinische oder technische Amplifikationen hilfreich sein (Brader et al. 2019). 

Eine der größten Motivationen Metagenomik zu betreiben, ist die Tatsache, dass 99 % aller Mikroorganismen nicht oder nur sehr schwer kultivierbar sind. Mithilfe metagenomischer Methoden ist es möglich Mikroorganismen zu identifizieren, auch wenn sie nicht kultivierbar sind (Brader et al. 2019).

Derzeit gibt es jedoch noch Limitierungen in der Metagenomik. Zum Beispiel werden Archaeen bei metagenomischen Untersuchungen leider oft übersehen. Der Grund dafür sind die Primer, die für metagenomische Analysen verwendet werden. Diese funktionieren nur bei einer kleinen Gruppe von Archaeen. Auch bei grampositiven und gramnegativen Bakterien gibt es hier Einschränkungen (Brader et al. 2019).

Die großen Datenmengen, die bei üblichen Sequenziermethoden, wie z.B. Illumina entstehen und die dadurch immer größer werdenden Datenbanken, sind eine Herausforderung für die Forscher. Inzwischen wurden eine Vielzahl von Algorithmen und rechnerischen Tools entwickelt, die die Analyse von Metagenom-Daten unterstützen. Zwei der heute gängigsten Methoden das Mikrobiom zu analysieren, sind Klassifizierungsmethoden und Assembly-Methoden. Bei ersteren wird das Gemisch von Spezies in einer Probe, entweder mithilfe von Markergenen zur Abschätzung ihrer Häufigkeit oder durch taxonomische Zuordnung einzelner Reads bestimmt. Bei Assembly-Methoden werden die Reads der gleichen Spezies zu größeren Contigs assembliert, wodurch taxonomische Zuordnung erfolgen kann. Jedoch sind Inkonsistenzen in der mikrobiellen Taxonomie oder Fehler im Genmodell ebenso große Herausforderungen, weil dies die meisten Methoden nicht berücksichtigen (Breitwieser et al. 2017).


# 2. Data generation

## 2.1 Introduction

Für Metagenom-Analysen wie diese, sind Reads einer Sequenzierung nötig. Da das Tool InSilicoSeq realistische Illumina-Reads erzeugen kann, wurden simulierte Daten verwendet, um die Analyse effizient durchzuführen zu können.
 
Ein großer Vorteil bei simulierten Daten ist die Tatsache, dass sie kostenlos erzeugt werden können und so eine teure Sequenzierung vermieden werden kann. Ebenso ein Vorteil gegenüber real-life Daten ist die Zeitersparnis. Eine weitere Motivation simulierte Daten zu verwenden, wäre experimentelles Design, neue Projekte zu entwerfen und auch zur Beurteilung und Validierung eines biologischen Modells.

InSilicoSeq bietet drei Modelle um Reads zu erzeugen: MiSeq, HiSeq und NovaSeq. Weitere Tools um simulierte Illumina-Reads für die Metagenom-Analyse zu erzeugen sind MetaSim, BEAR, FASTQSim, GemSim, Grinder, Mason, NeSSM und pIRS (Escalona et al. 2016).


## 2.2 Methods

Im ersten Schritt werden mit dem Sequencing Simulator Tool iss (InSilicoSeq - https://github.com/HadrienG/InSilicoSeq) die benötigten paired-end Illumina MySeq Reads erzeugt. 

Mit dem folgenden Befehl wurden 2 000 000 MiSeq reads von 2 bakteriellen und 1 Archae Genom - von RefSeq zufällig ausgewählt - simuliert.

```sh
iss generate --ncbi bacteria archaea --n_genomes_ncbi 2 1 --n_reads 2000000 --model MiSeq --output ncbi_lc --cpus 8
```

## 2.3 Results and discussion

Die 2.000.000 Reads wurden in einem .fasta File und die zugehörige Abundance in einem .txt File gespeichert.
Das Abundance File enthält 2 Spalten - in der ersten den Sequence Identifier und in der zweiten die Abundance.

In real-life Proben, z.B. in 1 mg Bodenprobe sind mehr als 1000 Mikroorganismen enthalten. Wenn davon ausgegangen wird, dass das Genom eines Bakteriums eine durchschnittliche Größe von 6 Millionen Basenpaare hat und bei der Sequenzierung eine Coverage von 100 erzielt wird, dann wären bei einer Readlänge von ca. 300 bp 2 Milliarden Reads das Resultat.

In Anbetracht dessen kann bei 3 Genomen, die in dieser Analyse verwendet wurden, nicht von Komplexität gesprochen werden.

# 3. Quality Control

## 3.1 Introduction

Um die Qualität der Reads zu kontrollieren, wurden die generierten .html Files im Browser begutachtet. Kontaminationen der Reads, wie zum Beispiel die Adaptersequenzen, die bei der Illumina-Sequenzierung angehängt werden, wurden mit dem Trimming entfernt, damit die Simulation nicht gestört wird.
Auch zu kurze Reads und Reads mit schlechter Qualität wurden entfernt.
 
Zusätzlich wurden statistische Methoden angewandt, um Informationen wie Anzahl der Reads, Anzahl der Sequenzen, durchschnittliche, minimale und maximale Readlänge usw. zu erhalten.


## 3.2 Methods

Die Qualität der Reads wurde mit dem Tool FastQC (https://github.com/s-andrews/FastQC) überprüft und anschließend wurde mit dem Tool bbduk (https://github.com/BioInfoTools/BBMap) ein Quality Trimming durchgeführt.

Mit FastQC wurde die Qualität beurteilt.

```sh
fastqc --outdir 01_fastqc_results $IN $IN2
```

Das Quality Trimming erfolgte mit bbduk.

```sh
bbduk.sh in1=$IN in2=$IN2 out1=$IN.bbduk.fq out2=$IN2.bbduk.fq qtrim=r trimq=10 maq=10 minlen=100
```

Die statistischen Informationen wurden mit seqstats aufgelistet.

```sh
seqstats $IN.bbduk.fq
seqstats $IN2.bbduk.fq
```

## 3.3 Results and discussion

Da die Qualität der Reads am 3' Ende abnimmt wurden sämtliche Reads mit einer Quality < 10 entfernt.
Ebenso wurden nur Reads berücksichtigt mit einer Mindestlänge von 100 Basen.

# 4. Profiling of the community

## 4.1 Introduction

Ziel dieses Schritts war es aufzuzeigen, welche Organismen sich in der Probe (der simulierten Daten) befinden. Dazu wurden die Read-Sequenzen mit den Sequenzen der Datenbanken verglichen um Ähnlichkeiten vom gleichen Organismus oder ähnlichen Organismen zu finden.

Universell für alle Spezies können hier entweder conserved taxonomic Markers, wie z.B. 16S rRNA verwendet werden oder single copy universal conserved taxonomic Proteins, auch Housekeeping-Genes genannt. 
Ein anderer Ansatz Sequenzen zu vergleichen, sind clade spezific Markers, die die Reads zu Gruppen von Organismen zuordnen, z.B. stickstofffixierende Organismen.

Limitierung bei diesem Schritt wäre, wenn die Sequenz des Genoms (noch) nicht in der Datenbank vorhanden ist.

Es wurden einige Tools herangezogen um ein taxonomisches Profiling des Datensatzes durchzuführen.
Folgende Tools wurden verwendet:
* Krakenuniq - https://github.com/fbreitwieser/krakenuniq
* Metaxa2    - https://microbiology.se/software/metaxa2/
* Motus      - https://motu-tool.org/
* Metaphlan  - http://huttenhower.sph.harvard.edu/metaphlan

Visualisierung der Ergebnisse:
* Krona      - https://github.com/marbl/Krona/wiki

## 4.2 Methods

### Krakenuniq

Das Profiling basierend auf der whole genome sequence erfolgte mit Krakenuniq.

```sh
krakenuniq --db $DBDIR --threads 10 --report-file kraken_taxonomy_profile.txt --paired read1.fq read2.fq  > kraken_read_classification.tsv
```

### Metaxa

Das Profiling basierend auf den rRNA Genen erfolgte mit Metaxa.

```sh
metaxa2 -1 read1.fq -2 read2.fq -g ssu --mode metagenome --plus T --cpu 8 --megablast T -o $OUT
```

###  Motus

Das Profiling basierend auf universal single-copy marker genes erfolgte mit Motus.

```sh
motus profile -f read1.fq -r read2.fq -t 30 > $OUT
```

### Metaphlan

Das Profiling basierend auf clade-specific marker genes erfolgte mit Metaphlan.

```sh
metaphlan2.py read1.fq,read2.fq --bowtie2out metagenome.bowtie2.bz2 --nproc 8 --input_type fastq > $OUT
```

### Krona

Die Darstellung der Ergebnisse erfolgte mit Krona.

```sh
ktImportText -o $OUT.krona.html metaphlan_krona.out
```

## 4.3 Results and discussion
Die Ergebnisse der einzelnen Tools sind nachfolgend angeführt:

* Krakenuniq
<img src="https://github.com/fhwnmatt/Metagenomics/blob/master/figures/Kraken.PNG" title="Kraken">

* Metaxa
<img src="https://github.com/fhwnmatt/Metagenomics/blob/master/figures/Metaxa.png" title="Metaxa">

* Motus
<img src="https://github.com/fhwnmatt/Metagenomics/blob/master/figures/Motus.PNG" title="Motus">

* Metaphlan
<img src="https://github.com/fhwnmatt/Metagenomics/blob/master/figures/Metaphlan.png" title="Metaphlan">

# 5 Assembly

## 5.1 Introduction

Genome assembly ist eine Herausforderung für einzelne Genome. Für ein Sample mit verschiedenen Spezies ist das noch komplizierter. Das wahrscheinlich größte Problem ist die ungleiche sequencing Depth der verschiedenen Organismen in einer metagenomischen Probe. Standard Assemblers nehmen an, dass die Depth of Coverage annähernd gleich über das gesamte Genom ist. Mögliche Tools dafür sind Megahit, SPAdes, MetaSPAdes und MetAMOS (Breitwieser et al. 2017).

## 5.2 Methods

Das Metagenom wurde mit dem Tool Megahit assembled.

```sh
megahit -1 read1.fq -2 read2.fq -o megahit_out
```
Das Assembly wurde mit Quast evaluiert.

```sh
quast -1 read1.fq -2 read2.fq $IN
```

Die Contig Coverage wurde mit bbmap/bbwrap durchgeführt.

```sh
bbwrap.sh ref=$IN in=read1.fq in2=read2.fq out=aln.sam.gz kfilter=22 subfilter=15 maxindel=80
pileup.sh in=aln.sam.gz out=cov.txt
```

## 5.3 Results and discussion

In der Datei assembly_stats.filtered.txt sind die Spalten #ID, Avg_fold, Length und	Read_GC angeführt.

# 6 Binning

## 6.1 Introduction

Binning ist der Prozess in dem Reads oder Contigs gruppiert werden und diese zu Operational Taxonomic Units (OTUs) assigned werden. Mögliche Tools sind Maxbin, CONCOCT, COCACOLA, MetaBAT und MetaCluster (Breitwieser et al. 2017).

## 6.2 Methods

Das Binning erfolgte mit MaxBin2 und MetaBAT.

```sh
run_MaxBin.pl -contig $IN -out $OUT -reads read1.fq -reads2 read2.fq -thread 8 -markerset 40
metabat -i $IN -a depth.txt -o $OUT -v --saveTNF saved.tnf --saveDistance saved.dist
```

Mit Checkm wurden die Ergebnisse evaluiert.

```sh
checkm lineage_wf -t 20 -x fa $IN $OUT > checkm_summary.txt
```

## 6.3 Results and discussion

Wie in checkm_summary.txt ersichtlich beträgt die Completeness über 90 % (außer bei bin.4: k__Bacteria (UID203)) und die Contamination weniger als 1 %. Die strain heterogeneity beträgt 0 % (außer bei bin.2: g__Mycobacterium (UID1816)).

# 7 References

Breitwieser, F. P., Lu, J., & Salzberg, S. L. (2017). A review of methods and databases for metagenomic classification and assembly. Briefings in Bioinformatics, (June), 1–15. https://doi.org/10.1093/bib/bbx120

Brader, G., Turaev, D. (2019). Metagenomik ILV

Escalona, M., Rocha, S., Posada, D. (2016). A comparison of tools for the simulation od genomic next-generation sequencing data. Nature Reviews Genetics, 20 June 2016 https://doi.org/10.1038/nrg.2016.57
