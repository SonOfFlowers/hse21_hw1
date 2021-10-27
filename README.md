# hse21_hw1

1. В первую очередь, создаём папку под названием "hw", которая будет содержать все рабочие файлы, и подключаем к ней линки:
```
mkdir hw
cd hw
mkdir 1
cd 1
ls -1 /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```

Создаём случайные чтения и удаляем ненужные файлы:
```
seqtk sample -s1108 oil_R1.fastq 5000000 > pe_R1.fastq
seqtk sample -s1108 oil_R2.fastq 5000000 > pe_R2.fastq
seqtk sample -s1108 oilMP_S4_L001_R1_001.fastq 1500000 > mp_R1.fastq
seqtk sample -s1108 oilMP_S4_L001_R2_001.fastq 1500000 > mp_R2.fastq

rm -r oil_R1.fastq
rm -r oil_R2.fastq
rm -r oilMP_S4_L001_R1_001.fastq
rm -r oilMP_S4_L001_R2_001.fastq
```

3. С помощью fastQC и multiQC оцениваем качество данных чтений:
```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}

mkdir multiqc
multiqc -o multiqc fastqc
```

Файлы с сервера скачивал с помощью WinSCP

4. Обрезаем чтение по качеству и убираем праймеры: 
```
platanus_trim pe_R1.fastq pe_R2.fastq 
platanus_internal_trim mp_R1.fastq mp_R2.fastq  
```
Удаляем ненужные файлы
```
ls -1 *.fastq | xargs -tI{} rm -r {}
```
5. С помощью fastQC и multiQC оцениваем качество новых обрезанных чтений:
```
mkdir trimmed_fastq
mv -v *trimmed trimmed_fastq/
mkdir trimmed_fastqc
ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```

6. Полученные результаты сравниваем:

До:
![image](https://user-images.githubusercontent.com/93160309/139125278-a02db6ba-b972-451f-8fff-235c14b419e6.png)

После:
![image](https://user-images.githubusercontent.com/93160309/139125570-4b7184f7-d58e-470c-a2d2-235e80239f26.png)

До:
![image](https://user-images.githubusercontent.com/93160309/139125948-a97e3ae3-8dcf-4b78-bd51-567631dd13e3.png)

После:
![image](https://user-images.githubusercontent.com/93160309/139125799-9450811d-847d-4027-a0ba-c031f16ca98e.png)

До:
![image](https://user-images.githubusercontent.com/93160309/139126135-244f0443-a2b4-4d90-b795-fefbe334fa00.png)

После:
![image](https://user-images.githubusercontent.com/93160309/139126216-92d6c882-dc99-425f-a7ba-6d73a0213611.png)

7. Cобираем контиги из подрезанных чтений и анализируем (результаты анализа см. в блокноте Jupiter Notebook):
```
time platanus assemble -o Poil -f trimmed_fastq/pe_R1.fastq.trimmed trimmed_fastq/pe_R2.fastq.trimmed 2> assemble.log
```
![image](https://user-images.githubusercontent.com/93160309/139126752-6245102c-d29a-42a9-bee4-9d109b0b4b75.png)

8. Из контигов и подрезанных чтений собираем скаффолды, анализ результатов проводим в блокноте: 
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> scaffold.log
```
![image](https://user-images.githubusercontent.com/93160309/139127188-623ca23d-7be9-4b25-9727-6be6f32923b4.png)

9. Создаем файл с одним самым большим размером:
```
echo scaffold1_len3834575_cov231 > name_scaff.txt
seqtk subseq Poil_scaffold.fa name_scaff.txt > BigScaff.fna
rm -r name_scaff.txt
```
10. Уменьшаем количество гэпов с помощью подрезанных чтений:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> gapclose.log
```
11. Создаем файл с одним самым большим размером:
```
echo scaffold1_cov231 > name_scaff.txt
seqtk subseq Poil_gapClosed.fa name_scaff.txt > longest.fna
rm -r name_scaff.txt
```
Расчёты можно найти в приложенном блокноте

Поскольку longest.fna у меня весит 0 КВ, я не могу его прикрепить в репозиторий

![image](https://user-images.githubusercontent.com/93160309/139137020-a272baa6-8586-4609-a6f5-e79b010657da.png)


