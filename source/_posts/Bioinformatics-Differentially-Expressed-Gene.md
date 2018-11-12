---
title: "差异表达基因分析流程_笔记"
date: 2018-06-25 20:16:30
mathjax: true
tags:
    - Bioinformatics
---

## 测序
- RNA 提取
- 文库建立
- 测序

## 数据处理
- FastQC : 分析数据
- Cutadapt : 切除 adapter 序列
- FASTX - Toolkit :
    - fastx_trimmer ( Trimmomatic ) : 切除 duplicate 序列、不良序列
    - fastq to fasta : 格式转换
    - fastx_barcodesplitter : 单链测序的 index

## 数据组装
- Trinity : 构建 denovo 转录组
- SOAP denovo : reads -> Contig -> Unigene


- Bowtie2 : 建立参考基因组的索引
- Tophat2 : reads mapping 到参考基因组
- Cufflinks : 拼接转录本，定量表达量


- Unigene 表达量 : 
   $$FPKM=\frac{10^6C}{NL/10^3}$$
    - C : 比对到 Unigene A 的 fragments 数
    - N : 比对到所有 Unigene 总 fragments 数
    - L : Unigene A 的碱基数

## 差异基因筛选 ( DEGs )
- FC ( Fold - Change ) : 表达差异倍数
- P - value : 差异显著性分析
- FDR 校验 ( False Discovery Ratio ) : P 值假阳性检验


- Cuffdiff : 筛选差异表达基因
- Cuffcompare : 转录本与参考基因组注释文件比较

## 差异表达分析
- Unigene 功能注释
    - Blastx : 数据库 ( Nr ) 比对 -> 功能注释信息
    - Blast 2 GO : 富集分析 -> GO 功能注释
    - WEGO : 数据库 ( GO ) 比对 -> 功能分类统计
        - 生物过程 ( Biological Process, BP )
        - 分子功能 ( Molecular Function, MF )
        - 细胞组成 ( Cellular Component, CC )
- 映射 
    - GO 数据库 -> GO term
    - KEGG 数据库 -> Pathway
    - 计算基因数目

## 后期处理
- 校验
    - 超几何检验
        $$P=\sum_{i=0}^{m-1}\frac{\binom{M}{i}\binom{N-M}{n-i}}{\binom{N}{n}}$$
        - N : 具有 GO 或 Pathway 注释的基因数目
        - n : N 中差异表达基因的数目
        - M : 注释为某特定 GO term 或 Pathway 基因数目
        - m : 注释为某特定 GO term 或 Pathway 差异表达基因数目
    - 实验检验
        - RT - q PCR <sup><a href="#r.1">[2]</a></sup>
- 整理排序
    - EXCEL
    - Python
    - R

## 备注
- RNA-Seq fastq -> BAM ( tophat2、hisat、star …… ) 
- Cufflinks BAM
- Cuffdiff BAM GTF


- Enrichment Analysis :
    - GO -> KEGG -> DO

## 参考文献
{% blockquote %}
[1] {% link 张春香, et al. "基于高通量转录组测序的山羊睾丸和附睾头差异表达基因分析." 畜牧兽医学报 45.3 (2014): 391-401. http://118.145.16.233/Jweb_xmsy/CN/article/downloadArticleFile.do?attachType=PDF&id=13190 %}
<br/>
[2] <a name="r.1">{% link 朱帅旗, et al. "葡萄糖诱导绿色杜氏藻转录组及相关通路差异分析." 中国生物化学与分子生物学报 31.8 (2015): 857-865. http://www.cqvip.com/qk/92593x/201508/665671113.html %}</a>
<br/>
[3] {% link 李小金, et al. "基于 RNA-seq 技术对不同品种猪背最长肌差异表达基因的筛选与注释." 西北农林科技大学学报: 自然科学版 44.6 (2016): 1-8. http://www.cqvip.com/qk/90760x/201606/668932837.html%}
<br/>
[4] {% link 刘洪博, et al. "干旱胁迫下割手密根系转录组差异表达分析." 中国农业科学 50.6 (2017): 1167-1178. http://111.203.21.2/Jwk_zgnykx/CN/article/downloadArticleFile.do?attachType=PDF&id=19620 %}
<br/>
[5] {% link 李建平, et al. "不同脂肪源饲粮育成猪肝脏转录组差异分析." 动物营养学报 27.7 (2015): 2128-2139. http://manu17.magtech.com.cn/dwyy/CN/article/downloadArticleFile.do?attachType=PDF&id=11996 %}
<br/>
[6] {% link 卢坤, et al. "利用 RNA-Seq 鉴定甘蓝型油菜叶片干旱胁迫应答基因." 中国农业科学 48.4 (2015): 630-645. http://111.203.21.2/Jwk_zgnykx/CN/article/downloadArticleFile.do?attachType=PDF&id=18513 %}
<br/>
[7] {% link RNA-seq :TopHat2 + Cufflinks分析流程 http://blog.sina.com.cn/s/blog_711ea0600102vn5p.html %}
{% endblockquote %}