# hse22_hw1
### Создадим символьные ссылки на файлы, чтобы не копировать их

```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

#### Установим seed (919) и выбирем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs

```
seqtk sample -s722 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s722 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s722 oilMP_S4_L001_R1_001.fastq 1500000 > mate_pair1.fastq
seqtk sample -s722 oilMP_S4_L001_R2_001.fastq 1500000 > mate_pair2.fastq
```

### С помощью программы fastQC и multiQC оцениваем качество исходных чтений и выводим по ним общую статистику

```
mkdir fastqc
ls sub* mate_pair* | xargs -tI{} fastqc -o fastqc {}
mkdir multiqc
multiqc -o multiqc fastqc
```

### Подрезаем чтения по качеству

```
platanus_trim sub*
platanus_internal_trim mate_pair*
```

### Удаляем исходные .fastq файлы

```
rm sub1.fastq
rm sub2.fastq
rm mate_pair1.fastq 
rm mate_pair2.fastq
```

### С помощью программ fastQC и multiQC оцениваем качество подрезанных чтений и получаем по ним общую статистику

```
mkdir fastqc_trim
ls sub* mate_pair*| xargs -tI{} fastqc -o fastqc_trim {}
mkdir multiqc_trim
multiqc -o multiqc_trim fastqc_trim
```

### Собираем контиги с помощью platanus assemble

```
time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log
```

### Собираем скаффолды

```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 mate_pair1.fastq.int_trimmed mate_pair2.fastq.int_trimmed 2> scaffold.log
```

### Уменьшаем кол-во гэпов с помощью программы platanus gap_close

```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 mate_pair1.fastq.int_trimmed mate_pair2.fastq.int_trimmed 2> gapclose.log
```

