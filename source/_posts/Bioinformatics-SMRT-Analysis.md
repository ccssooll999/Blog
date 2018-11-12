---
title: "无参转录组三代测序分析流程"
date: 2018-10-20 13:26:52
mathjax: false
tags:
    - Bioinformatics
    - RNA-seq
---
{% centerquote %}
注：本文出现所有脚本仅供参考，若出现任何问题或造成任何损失，本人概不负责。
{% endcenterquote %}

<!-- more -->
<br>

## 1. Canu / wtdbg（从头组装）
- （略）

<br>

## 2. Polin（校对）
简介 : 

- Pilon 可对初步组装的基因组进行 polish，以提高组装结果。

预处理 : 

- 生成 bam 文件

运行 : 
```
pilon \
  --genome h1.contigs.fasta \
  --frags h1.contigs.fasta.sorted.bam \
  --output h1_pilon \
  --outdir h1_pilon \
  --threads 24 \
  --diploid
```
- genome : 待校对基因
- frags : 待校对基因生成的 bam 文件（需排序）
- output : 输出文件前缀
- outdir : 输出文件夹
- threads : 调用线程数
- diploid : 待分析物种为二倍体
- fix bases : 只有二代数据时使用
- vcf : 只有二代数据时使用
- changes : 只有二代数据时使用

注 : 

- 若计算节点内存不足，可将待校对基因分割后置于不同节点并行计算。
- 分割后 frags 依然可使用原 bam 文件

<br>

## 3. Maker（基因组结构注释）
简介 : 

- Maker 是一套完善且强大的基因组结构注释流程，使用简单快捷。它可根据序列结构、已有的转录本序列、蛋白序列，寻找基因结构的证据，然后对这些证据进行整合，最终给出判断。

输入 : 
```
A. genome.fasta (必须)
   基因组序列文件。
B. inchworm.fasta
   转录组组装结果文件。此文件内容为转录子序列或 EST 序列。
C. proteins.fasta
   临近物种的蛋白质序列。可从 NCBI 的 Taxonomy 数据库中获取。
D. consensi.fa.classified
   适合本物种的重复序列数据库。此文件为 fasta 格式，序列代表着本物种的高重复序列，可使用 RepeatModeler 制作。
E. Augustus hmm files [species]
   适合本物种的 Augustus 的 HMM 文件。此文件示例位于 augustus/config/species/ 目录下。亦可自行训练模型，但生成速度一般较为缓慢。
F. Snap hmm file [species.hmm]
   适合本物种的 SNAP 的 HMM 文件。将 genome.fasta 和 inchworm.fasta 输入 PASA，得到用于 trainning 的基因，然后使用 SNAP 进行 training，可得到 HMM 文件。
G. Genemark_es hmm file [es.mod]
   适合本物种的 Genemark_es 的 HMM 文件。将 genome.fasta 输入 genemark_es 软件进行自我训练，可得到 HMM 文件。
H. rRNA.fa
   rRNA 的序列文件。此文件可通过 RNAmmer 对基因组进行分析得到。
```

运行 :
```
cd ${workplace}
maker -CTL
maker
```
1. 进入工作目录
2. 初始化 maker，可得到三个配置文件 :
    - maker_exe.ctl : 配置 Maker 需要调用的程序的路径
    - maker_bopts.ctl : 配置 blast 的各种阈值
    - maker_opts.ctl : 配置输入文件的路径，以及使用流程（主要配置文件）
3. 运行 maker

配置参考 :

- {% link maker_opts.ctl https://github.com/ccssooll999/Script/blob/master/config/maker_opts.ctl %}

注 : 

- 若基因组序列较大，可将其分割后并行计算。
- JBrowse 可完成数据可视化。
- RepeatMasker 可能会调用 {% link Repbase https://www.girinst.org/repbase/ %} 数据库，需自行下载并初始化。
- 部分集群无法在`\home`目录直接运行，可尝试将工作目录转移到`\tmp`，完成后再将结果文件回收到`\home`目录。

<br>

## 4. Interproscan（基因组功能注释）
简介 : 

- Interpro 是一个包含蛋白功能、蛋白家族等信息的数据库，而 Interproscan 可以将待测蛋白序列与其比对，从而得到完整的序列功能注释。

运行 : 
```
interproscan.sh \
  -dp -pa \
  -cpu 24 \
  -t p \
  -f tsv \
  -i ${file_name}.fasta \
  -o ${file_name}.tsv
```
- cpu : 调用线程数
- dp : 关闭 InterProScan 5 lookup service 功能
- iprlookup : 开启 interpro 注释
- goterms : 开启 GO 注释，需 iprlookup 参数作为前置
- pa : 开启可能的代谢注释
- ms : 最小核酸 ORF 的大小
- t : 输入序列的类型
    - p : 蛋白（默认）
    - n : 核酸
- f : 输出文件的格式
    - TSV、XML、GFF3、HTML、SVG
- i : 输入文件
- o : 输出文件

注 : 

- 若服务器无法联网，务必使用 -dp 参数，关闭 InterProScan 5 lookup service 功能。

<br>

## 5. Nucmer（基因组序列比对）
简介 : 

- nucmer 是一个快速比对两个基因组序列的软件，用于联配相近 (closely related) 的核酸序列。

运行 : 
```
nucmer \
  --prefix=A-B \
    ${A.fasta} \
    ${B.fasta}
```

- prefix : 输出文件前缀
- A.fasta : 输入文件 A
- B.fasta : 输入文件 B

注 : 

- 为提高nucmer的精确性，可将输入序列进行遮盖 (mask) 处理，避免不感兴趣的序列的联配，或修改单一性限制以降低重复导致的联配数。
- nucmer 属于 mummer 软件包

<br>

## 6. OrthoFinder（提取单拷贝基因）
简介 : 

- OrthoFinder 是一款可以寻找不同物种之间的单拷贝基因的软件，为后期构建物种树做准备。

预处理 : 

- 新建文件夹并导入序列文件

运行 : 
```
orthofinder \
  -t 24 \
  -a 24 \
  -S diamond \
  -f ${work_folder}
```
- t : 序列搜索调用线程数
- a : 序列分析调用线程数
- S : 指定比对软件（默认 : blast）
- f : 工作目录
    - 需将待分析序列提前放入此目录

注 : 

- 若基因组较大，使用 diamond 代替 blast 作为比对软件，可大幅提高执行速度。

<br>

## 7. Muscle（蛋白质多序列比对）
简介 : 

- MUSCLE 是一款蛋白质水平多序列比对的软件，在速度和精度上都明显优于 ClustalW 。

预处理 : 

- 蛋白序列名称处理
    - `awk '{print $1}' ${X.fa}`
- {% link extract.pl https://github.com/ccssooll999/Script/blob/master/SMRT-Analysis/extract.pl %}&nbsp;&nbsp;<所有蛋白序列> <单拷贝家族 ID> <家族文件>

运行 : 
```
muscle \
  -in  ${A.fasta} \
  -out ${B.fasta}
```
- in : 输入序列
- out : 输出序列

<br>

## 8. Gblock（比对位点过滤）
简介 : 

- Gblocks 是一个用于消除 核酸序列 / 蛋白序列 较差的比对位点及分散区域的软件。

运行 : 
```
Gblock ${B.fasta} -t=p
```
- t : 序列种类

<br>

## 9. RAxML / MEGA（绘制系统发育树）
简介 : 

- RAxML 是一款能够利用最大似然法构建进化树的软件。

预处理 : 

- 合并所有 muscle 结果为一个文件
- {% link merge.py https://github.com/ccssooll999/Script/blob/master/SMRT-Analysis/merge.py %}&nbsp;&nbsp;<输入文件> <序列名称 - 首两位字符> <输出序列名称>
    - 序列名称首两位字符不能重复
    - 包含两种序列名称时需预处理
- 格式处理
    - 去除空格
    - 去除换行符

运行 : 
```
raxmlHPC-PTHREADS-SSE3 \
  -s ${A.phy} \
  -T 24 \
  -n tre \
  -m PROTGAMMALGX \
  -p 12345
```

- s : 输入文件 (phy / fasta)
- T : 调用线程数
- n : 指定输出文件后缀
- m : 指定核苷酸或氨基酸替代模型
- p : 指定一个随机数作为种子

参考 : 

- {% link 使用 RAxML 构建进化树 http://www.chenlianfu.com/?p=2225 %}

注 : 

- 按 CPU 的计算速度划分为 3 种版本：标准版、SSE3 和 AVX 版本。
- 按并行化方式划分为 4 种版本：Sequential、Pthreads、MPI、Hybrid Pthreads/MPI。
- MEGA 使用方法：略

<br>

## 10. CAFE（基因家族分析）
简介 : 

- CAFE 是一款基因家族分析的软件，可以为进化推论提供统计基础。

预处理 : 

- `awk 'OFS="\t" {$NF="" ;print $0}' Orthogroups.GeneCount.csv > cafe.data`

运行 :
```
load -i ${cafe.data} -t 10 -l logfile.txt -p 0.05 
tree (((chimp:6,human:6):81,(mouse:17,rat:17):70):6,dog:93) 
lambda -s -t (((1,1)1,(2,2)2)2,2) 
report resultfile
```
- λ : 表示在物种进化过程中，每个单位时间内基因获得与丢失的概率，可由软件自行计算。
- t : 默认下所有分支的 λ 值是相同的，若需要不同的分支有不同的 λ 值，则用该参数进行设置。该参数的值和 tree 命令中的树的内容一致，只是去除了分歧时间，并将物种名换成了表示 λ 值的编号。其中，相同的编号表示有相同的 λ 值。

参考 : 

- {% link CAFE 官方文档 https://hahnlab.github.io/CAFE/src_docs/html/Usage.html %}
- {% link The Newick tree format http://evolution.genetics.washington.edu/phylip/newicktree.html %}