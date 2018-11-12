---
title: "无参转录组二代测序分析流程"
date: 2018-09-05 12:39:06
mathjax: true
tags:
    - Bioinformatics
    - RNA-seq
---
{% centerquote %}
注：本文出现所有脚本仅供参考，若出现任何问题或造成任何损失，本人概不负责。
{% endcenterquote %}

<!-- more -->
<br>

## 0. 输入数据预处理
- 输入数据
  - GZ 格式 Reads 序列
    - 经过去接头处理
    - 去除低质量序列
    - 达到质控标准
    - 双端测序
    - 路径： <code>Cleandata\\</code>

- 输出数据
  - FQ 格式 Reads 序列
    - 路径： <code>0_File_Format\\</code>
  - samples_file.txt
    - FQ 文件路径信息
    - 路径： <code>pbs_RNA-seq_analysis\files\\</code>
  - SOAPdenovo-Trans.config
    - SOAPdenovo-Trans 配置文件
    - 路径： <code>pbs_RNA-seq_analysis\files\\</code>

- 操作流程
  1. 将 <code>Cleandata\\</code> 内数据解压缩并移动至 <code>0_File_Format\\</code> 目录内
  - 扫描 <code>0_File_Format\\</code> 目录结构并生成 <code>samples_file.txt</code> 和 <code>SOAPdenovo-Trans.config</code>

- 其他
  - {% link 参考脚本 https://github.com/ccssooll999/pbs_RNA-seq_analysis/blob/master/0_File_format.pbs %}

<br>

## 1. SOAPdenovo-Trans（从头组装）
- 输入数据
  - Reads 序列

- 输出数据
  - 最优 K-mer 值的 Contig 序列
  - N50 评估
  - BUSCO 评估

- 操作流程
  1. 利用 SOAPdenovo-Trans 进行不同 K-mer 下的组装
  - 利用 quast 计算结果的 N50 快速比较拼接质量
  - 利用 BUSCO 准确评价转录本拼接质量
  - 选出最优 K-mer 下的转录本拼接结果

- 参数详解
```
SOAPdenovo-Trans-127mer all \
          -s ../pbs_RNA-seq_analysis/files/SOAPdenovo-Trans.config \
          -o K37 \
          -p 40 \
          -K 37 \
          -d 1
```
  - SOAPdenovo-Trans-31mer 可处理低于 31 的 K 值，SOAPdenovo-Trans-127mer 可处理低于 127 的 K 值，但内存消耗将会翻倍。
  - s : 配置文件
  - o : 输出文件名前缀
  - p : 调用线程数
  - K : K-mer 值，默认值 23
  - d : 去除频数不大于该值的 K-mer

- 配置文件
  ```
  # reads 的最大长度
  max_rd_len=150
  [LIB]
  # 文库平均插入长度
  avg_ins=350
  # 序列是否需要被反转，转录组设为 0
  reverse_seq=0
  # 该文库中的read序列在组装的哪些过程（contig/scaff/fill）
  asm_flags=3
  # 序列路径
  q1=/**PATH**/fastq_1_R1.fq
  q2=/**PATH**/fastq_1_R2.fq
  q1=/**PATH**/fastq_2_R1.fq
  q2=/**PATH**/fastq_2_R2.fq
  ···
  ```

- 其他
  - {% link 参考脚本 https://github.com/ccssooll999/pbs_RNA-seq_analysis/blob/master/1_SOAPdenovo-Trans.pbs %}
  - {% link 参考文档 https://github.com/aquaskyline/SOAPdenovo-Trans %}

<br>

## 2. Trinotate（转录组注释）
- 输入文件
  - Contig 序列
  - gene_trans_map
  - Trinotate 数据库

- 输出文件
  - 注释结果

- 参数详解
```
perl ${TRINOTATE_HOME}/auto/autoTrinotate.pl \
        --Trinotate_sqlite Trinotate.sqlite \
        --transcripts SOAPdenovo-Trans.fasta \
        --gene_to_trans_map SOAPdenovo-Trans.fasta.gene_trans_map \
        --conf conf.txt \
        --CPU 24
```
  - autoTrinotate.pl 位于 Trinotate 路径下 auto 文件夹，可根据配置文件自动完成 Trinotate 整套操作
  - Trinotate_sqlite : 数据库文件（详情见附）
  - transcripts : Contig 序列
  - gene_to_trans_map : 基因 -> 转录组 信息文件
  - conf : Trinotate 配置文件
  - CPU : 调用线程数

- 配置文件
  - {% link conf.txt https://github.com/ccssooll999/pbs_RNA-seq_analysis/blob/master/files/conf.txt.mould %}

- 其他
  - {% link 参考脚本 https://github.com/ccssooll999/pbs_RNA-seq_analysis/blob/master/2_Trinotate.pbs %}
  - {% link 参考文档 https://github.com/Trinotate/Trinotate.github.io/wiki %}
  - 注 : sqlite 在集群上可能会出现各类不知名错误，可尝试将其移动至 /tmp 下工作

<br>

## 3. Quantification（表达定量）
- 输入文件
  - Contig 序列
  - Reads 序列
  - gene_trans_map

- 输出文件
  - 表达定量矩阵

- align_and_estimate_abundance.pl 参数详解
```
${TRINITY_HOME}/util/align_and_estimate_abundance.pl \
    --transcripts $transcripts \
    --samples_file $samples_file \
    --seqType fq \
    --est_method RSEM \
    --aln_method bowtie2 \
    --thread_count 24 \
    --prep_reference \
    --gene_trans_map $fake_gene_trans_map
```

- abundance_estimates_to_matrix.pl 参数详解
```
${TRINITY_HOME}/util/abundance_estimates_to_matrix.pl \
    --est_method RSEM \
    --cross_sample_norm TMM \
    --name_sample_by_basedir \
    --quant_files genes.quant_files.txt \
    --out_prefix genes \
    --gene_trans_map $fake_gene_trans_map
```
  - transcripts : Contig 组装序列
  - samples_file : Reads 路径信息
  - seqType : 输入文件格式
  - est_method : 指定表达定量的方式 ( 四选一 ) 
    - alignment_based : RSEM | eXpress
    - alignment_free : kallisto | salmon
  - aln_method : 基于 based 模式 ( bowtie | bowtie2 ) 
  - thread_count : 调用线程数
  - prep_reference : 比对前对参考序列构建索引
  - gene_trans_map : 基因 -> 转录组 信息文件
  - cross_sample_norm : 样品间标准化
  - name_sample_by_basedir : 矩阵 ( 列 ) 命名规律
  - quant_files : 表达定量列表 输出文件
  - out_prefix : 输出前缀

- 其他
  - {% link 参考脚本 https://github.com/ccssooll999/pbs_RNA-seq_analysis/blob/master/3_Quantification.pbs %}
  - {% link 参考文档 https://github.com/trinityrnaseq/trinityrnaseq/wiki %}

<br>

## 4. DESeq2（差异基因提取）
- 输入文件
  - Reads 序列
  - 定量表达矩阵

- 输出文件
  - 差异基因列表

- run_DE_analysis.pl 参数详解
```
run_DE_analysis.pl \
        --matrix $isoforms_counts_matrix \
        --method edgeR \
        --samples_file $samples_file
```
  - matrix : 定量表达矩阵
  - method : 计算模型
  - samples_file : Reads 路径信息

- 其他
  - {% link 参考脚本 https://github.com/ccssooll999/pbs_RNA-seq_analysis/blob/master/4_DESeq2.pbs %}
  - {% link 参考文档 http://www.bioconductor.org/packages/release/bioc/html/DESeq2.html %}

<br>

## 5. GOSeq（富集分析）
- 输入文件
  - 注释结果
  - 差异基因列表
  - 定量表达矩阵

- 输出文件
  - 富集分析结果

- extract_GO_assignments_from_Trinotate_xls.pl 参数详解
```
perl ${TRINOTATE_HOME}/util/extract_GO_assignments_from_Trinotate_xls.pl \
        --Trinotate_xls $Trinotate_report \
        --include_ancestral_terms \
        - T > go_annotations.txt
```
    - Trinotate_xls : Trinotate 注释文件
    - T : isoform 层次
    - G : gene 层次

- fasta_seq_length.pl 参数详解
```
perl ${TRINITY_HOME}/util/misc/fasta_seq_length.pl $transcript > trans.lengths.txt
```

- run_GOseq.pl 参数详解
```
perl ${TRINITY_HOME}/Analysis/DifferentialExpression/run_GOseq.pl \
    --genes_single_factor $de_lst \
    --GO_assignments go_annotations.txt \
    --lengths trans.lengths.txt \
    --background $background
```
- genes_single_factor : 差异基因列表
- GO_assignments : 注释结果
- lengths : 序列长度文件
- background : 表达定量矩阵

- 其他
  - {% link 参考脚本 https://github.com/ccssooll999/pbs_RNA-seq_analysis/blob/master/5_GOSeq.pbs %}
  - {% link 参考文档 http://www.bioconductor.org/packages/release/bioc/html/goseq.html %}
  - 深入富集分析工具 : ClusterProfiler