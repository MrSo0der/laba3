# hse24_hw1

## Список всех команд

Создание символических ссылок

```shell
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

Выбор случайных чтений

```shell
seqtk sample -s208 oil_R1.fastq 5000000 > pe1.fq
seqtk sample -s208 oil_R2.fastq 5000000 > pe2.fq
seqtk sample -s208 oilMP_S4_L001_R1_001.fastq 1500000 > me1.fq
seqtk sample -s208 oilMP_S4_L001_R2_001.fastq 1500000 > me2.fq
```

Оценка качества чтений с помощью FastQC и MultiQC

```shell
fastqc pe1.fq
fastqc pe2.fq
fastqc me1.fq
fastqc me2.fq
multiqc .
```

Подрезание чтений и удаление адаптеров

```shell
printf "pe1.fq\npe2.fq" > pe.fofn
printf "me1.fq\nme2.fq" > me.fofn
platanus_trim -i pe.fofn -q 20
platanus_internal_trim -i me.fofn -a 2 -q 20
```

Оценка качества подрезанных чтений с помощью FastQC и MultiQC

```shell
fastqc pe1.fq.trimmed
fastqc pe2.fq.trimmed
fastqc me1.fq.int_trimmed
fastqc me2.fq.int_trimmed
multiqc .
```

Сборка контигов из подрезанных чтений

```shell
platanus assemble -f pe1.fq.trimmed pe2.fq.trimmed
```

Сборка скаффолдов из контигов и подрезанных чтений

```shell
platanus scaffold \
    -c ./out_contig.fa \
    -b ./out_contigBubble.fa \
    -IP1 pe1.fq.trimmed pe2.fq.trimmed \
    -OP2 me1.fq.int_trimmed me2.fq.int_trimmed
```

Уменьшение количества гэпов

```shell
platanus gap_close \
    -c ./out_scaffold.fa \
    -IP1 pe1.fq.trimmed pe2.fq.trimmed \
    -OP2 me1.fq.int_trimmed me2.fq.int_trimmed
```

## Скриншоты и статистика из файлов MultiQC

### Исходные чтения

![MultiQC initial reads quality](/multiqc/initial/image.png "Качество исходных чтений")

[Ссылка](https://mrso0der.github.io/hse24_hw1/multiqc/initial/multiqc_report.html) на полный отчет в формате HTML

[Ссылка](https://github.com/MrSo0der/hse24_hw1/tree/main/multiqc/initial) на все output-файлы MultiQC

### Подрезанные чтения

![MultiQC initial reads quality](/multiqc/trimmed/image.png "Качество подрезанных чтений")

[Ссылка](https://mrso0der.github.io/hse24_hw1/multiqc/trimmed/multiqc_report.html) на полный отчет в формате HTML

[Ссылка](https://github.com/MrSo0der/hse24_hw1/tree/main/multiqc/trimmed) на все output-файлы MultiQC

## [Ссылка на jupiter ноутбуки](https://github.com/MrSo0der/hse24_hw1/tree/main/src)

## Результаты бонусной части задания

[Модифицируем](https://github.com/MrSo0der/hse24_hw1/tree/main/src/for_9_10.ipynb) скрипты из предыдущей части задания (смерджим и добавим подсчёт статистик L50 и N90).
Получим контиги и скаффолды командами, описанными выше, из 500000 чтений типа paired-end и 150000 чтений типа mate-pairs, прогоним через скрипт, получим следующие результаты:

| 5M pe/1,5M mp                                                                                                                                                                                                                                                                                                                                                                                                                                                      | 500k pe/150k mp                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Общее количество контигов: 641<br>Общая длина: 3926058<br>Длина самого длинного контига: 158295<br>N50: 52797 <br>L50: 24<br>N90: 11787<br><br>Общее количество скаффолдов: 71<br>Общая длина: 3875920<br>Длина самого длинного скаффолда: 3834838<br>N50: 3834838 <br>L50: 1<br>N90: 3834838<br><br>Исходный самый длинный скаффолд:<br>Гепы: 71 <br>Общая длина: 7207<br><br>Самый длинный скаффолд после gap_close:<br>Гепы: 12 <br>Общая длина: 2244<br> | Общее количество контигов: 3963<br>Общая длина: 3919282<br>Длина самого длинного контига: 19949<br>N50: 3110 <br>L50: 340<br>N90: 641<br><br>Общее количество скаффолдов: 606<br>Общая длина: 3865199<br>Длина самого длинного скаффолда: 740658<br>N50: 439412 <br>L50: 4<br>N90: 106975<br><br>Исходный самый длинный скаффолд:<br>Гепы: 378 <br>Общая длина: 20152<br><br>Самый длинный скаффолд после gap_close:<br>Гепы: 24 <br>Общая длина: 9257<br> |

На основе этих данных можно сделать вывод о том, что увеличение объема чтений значительно улучшает сборку как контигов (меньше количество, но длиннее и качественнее), так и скаффолдов
(более длинные с меньшим количеством пробелов, следовательно, более успешная интеграция контигов в скаффолды).
Уменьшение же объема чтений приводит к фрагментации сборки и снижению её качества по основным статистикам.