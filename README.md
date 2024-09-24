# hse24_hw1

## Список всех команд

Создание символических ссылок

```bash
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

Выбор случайных чтений

```bash
seqtk sample -s208 oil_R1.fastq 5000000 > pe1.fq
seqtk sample -s208 oil_R2.fastq 5000000 > pe2.fq
seqtk sample -s208 oilMP_S4_L001_R1_001.fastq 1500000 > me1.fq
seqtk sample -s208 oilMP_S4_L001_R2_001.fastq 1500000 > me2.fq
```

Оценка качества чтений с помощью FastQC и MultiQC

```bash
fastqc pe1.fq
fastqc pe2.fq
fastqc me1.fq
fastqc me2.fq
multiqc .
```

Подрезание чтений и удаление адаптеров

```bash
printf "pe1.fq\npe2.fq" > pe.fofn
printf "me1.fq\nme2.fq" > me.fofn
platanus_trim -i pe.fofn -q 20
platanus_internal_trim -i me.fofn -a 2 -q 20
```

Оценка качества подрезанных чтений с помощью FastQC и MultiQC

```bash
fastqc pe1.fq.int_trimmed
fastqc pe2.fq.int_trimmed
fastqc me1.fq.int_trimmed
fastqc me2.fq.int_trimmed
multiqc .
```

Сборка контигов из подрезанных чтений

```bash
platanus assemble -f pe1.fq.int_trimmed pe2.fq.int_trimmed
```

Сборка скаффолдов из контигов и подрезанных чтений

```bash
platanus scaffold \
    -c ./out_contig.fa \
    -b ./out_contigBubble.fa \
    -IP1 pe1.fq.trimmed pe2.fq.trimmed \
    -OP2 me1.fq.int_trimmed me2.fq.int_trimmed
```

Уменьшение количества гэпов

```bash
platanus gap_close \
    -c ./out_scaffold.fa \
    -IP1 pe1.fq.trimmed pe2.fq.trimmed \
    -OP2 me1.fq.int_trimmed me2.fq.int_trimmed
```

## Скриншоты и статистика из файлов multiQC

### Исходные чтения

![alt text](/multiqc/initial/image.png)

### Подрезанные чтения

