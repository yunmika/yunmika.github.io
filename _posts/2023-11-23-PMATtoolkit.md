---
layout:     post
title:      "「PMAT」线粒体基因组组装工具包"
subtitle:   "An efficient assembly toolkit for plant mitochondrial genome"
date:       2023-11-23 16:00:00
author:     "云伴风行"
header-img: "img/blog-PMAT2.png"
header-mask: 50%
catalog: true
tags:
    - PMAT
    - 线粒体基因组
    - python
    - shell
---


目前已经开发出很多软件应用在植物线粒体基因组的组装，大部分都是针对二代测序数据和三代测序数据ONT及CLR进行组装。受限于二代数据的长度及ONT和CLR测序数据的准确性，对复杂的植物线粒体基因组组装效果并不理想。PMAT可以对WGS的三代测序数据ONT、CLR及HiFi进行组装，并且自动选择线粒体基因组中的序列。该软件目前只能在linux系统运行，[github地址](https://github.com/bichangwei/PMAT) 。

### PMAT 安装
通过git安装
```sh
git clone https://github.com/bichangwei/PMAT.git
cd PMAT/bin
chmod a+x PMAT
PMAT --help
```
源代码安装
```sh
wget https://github.com/bichangwei/PMAT/archive/refs/tags/v1.5.2.tar.gz
tar -zxvf v1.5.2.tar.gz
cd PMAT-1.5.2/bin
chmod a+x PMAT
PMAT --help
```

### Apptainer 安装
需要提前安装Apptainer，具体安装方法可以参考[链接](https://github.com/apptainer/apptainer/blob/main/INSTALL.md)


### 依赖软件
###### 当使用HiFi数据进行组装时，有以下依赖:
- [**blastn**](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=Download)  需要添加到环境变量中.

###### 对于ONT和crl测序数据，有以下依赖:
- [**blastn**](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=Download)     需要添加到环境变量中.
- 如果数据纠错软件指定使用[**canu**](https://github.com/marbl/canu)，则需要安装[**canu**](https://github.com/marbl/canu)，否则[**canu**](https://github.com/marbl/canu)和[**NextDenovo**](https://github.com/Nextomics/NextDenovo)都需要安装(建议添加到环境变量)。



### PMAT 使用
Run `PMAT --help` to view the program's usage guide.
```shell
usage: PMAT <command> <arguments>

 ______     ___           __        ____       _____________ 
|   __  \  |   \        /   |      / __ \     |_____   _____|
|  |__)  | | |\ \      / /| |     / /  \ \          | |      
|   ____/  | | \ \    / / | |    / /____\ \         | |      
|  |       | |  \ \  / /  | |   / /______\ \        | |      
|  |       | |   \ \/ /   | |  / /        \ \       | |      
|__|       |_|    \__/    |_| /_/          \_\      |_|      

PMAT            an efficient assembly toolkit for plant mitochondrial genome
Version         1.5.2
Contributors    Bi,C. and Han,F.
Email           bichwei@njfu.edu.cn, hanfc@caf.ac.cn

For more information about PMAT, see https://github.com/bichangwei/PMAT

optional arguments:
-h, --help     show this help message and exit
-v, --version  show program's version number and exit

Commands:

    autoMito    One-step de novo assembly of the mitochondrial genome. 
                This command corrects the raw ONT/CLR data or uses 
                the corrected data or HiFi for assembly directly. 
                Based on the assembly result, automatically select 
                seeds for extension and filter false positives to 
                obtain an assembly map of the mitochondrial genome.

    graphBuild  If PMAT fails to generate the mitochondrial genome 
                assembly map in one-step assembly, you can use this 
                command by manually select seeds for assembly.
```

**autoMito** 对WGS测序数据（ONT, CLR 和 HiFi）进行从头组装分析  
`PMAT autoMito -h` 查看程序autoMito的帮助文档
```sh
# 必选参数
-i --input      #WGS测序数据文件，可以是CLR，ONT 和 HiFi，必须是fasta或者fastq格式或者压缩后的文件
-o --output     #输出结果路径
-st --seqtype   #输入数据类型 CLR/clr、ONT/ont、HiFi/hifi
-g --genomesize #物种基因组大小，例如1G 200M等

# 可选参数
-tk --task        #可选任务all/p1，默认为all。如果输入数据为高错误率的原始测序数据需要使用all，如果输入数据为纠错后的数据可以选择p1
-tp --type        #可选择all/mt/pt，默认为mt。选择进行组装细胞器基因组的类型
-cp --canu        #canu软件安装路径，如果已经添加到环境变量中则不需要提供该参数，用于前期的测序数据纠错和修剪过程
-np --nextDenovo  #nextDenovo软件的安装路径，如果已经添加到环境变量中则不需要提供该参数，用于数据的纠错
-cs --correctsoft #测序数据纠错使用的软件，默认使用nextDenovo，可以选择canu或者NextDenovo
-cfg --correctcfg #当使用NextDenovo为纠错软件时，需要提供该参数
-fc --factor      #选择一定比例的子集序列作为组装线粒体基因组的数据集，默认使用全部的测序数据进行组装
-u, --unloop      #用于自动解环的参数，默认关闭状态
-cpu              #选择使用线程数
```

**graphBuild** 对组装结果自动检查线粒体基因组序列，并生成gfa文件  
`PMAT graphBuild -h` 查看程序graphBuild的帮助文档
```sh
# 必选参数
-c --ContigGraph    #组装得到的 PMATContigGraph.txt
-a --AllContigGraph #组装得到的 PMATAllContigs.fna
-o --output         #输出文件路径
-gs --genomesize    #物种基因组大小
-rs --readsize      #组装使用数据量大小

# 可选参数
-cpu #线程数
-u, --unloop      #用于自动解环的参数，默认关闭状态
-s --seeds #选择指定的seeds作为候选种子进行延伸,不提供该参数则进行自动选择
```

###### 应用实例

1. 下载拟南芥HiFi数据:
```sh
wget https://github.com/bichangwei/PMAT/releases/download/v1.1.0/Arabidopsis_thaliana_550Mb.fa.gz
```
2. 运行autoMito进行一步组装:
```sh
PMAT autoMito -i Arabidopsis_thaliana_550Mb.fa.gz -o ./test1 -st hifi -g 120m -m -tp all
```
3. 使用graphBuild手动设置候选contig进行组装:
```sh
# Based on the PMATContigGraph.txt file, manually select 3 or more contigs that match the depth of mitochondrial genome sequencing
PMAT graphBuild -c ./test1/assembly_result/PMATContigGraph.txt -a ./test1/assembly_result/PMATAllContigs.fna -gs 125m -rs ./test1/subsample/assembly_seq_subset.0.1.fasta -o ./test1_gfa -s 343 345 905 513 1344 -tp mt
```
4. 运行时间

```
8 CPUs: 13m25.342s; 16 CPUs: 9m29.853s; 32 CPUs: 8m42.429s; 64 CPUs: 7m57.279s
```

输出结果：
- subsample/assembly_seq_subset.0.1.fasta *用于组装的数据*
- subsample/assembly_seq.cut20K.fasta     *截断为长度为20kb的reads*
- assembly_result/PMATAllContigs.fna      *组装结果contig序列文件*
- assembly_result/PMATContigGraph.txt     *组装结果contig连接关系*
- assembly_result/PMAT_mt_raw.gfa         *用于可视化的线粒体基因组组装初始结果*
- assembly_result/PMAT_mt_master.gfa      *用于可视化的线粒体基因组组装优化结果*
- assembly_result/PMAT_pt_raw.gfa         *用于可视化的叶绿体基因组组装初始结果*
- assembly_result/PMAT_pt_master.gfa      *用于可视化的叶绿体基因组组装优化结果*

###### 更新日志
PMAT version 1.5.0 (23/11/14)
Updates:
- PMAT添加自动解环功能（还在测试中）

PMAT version 1.4.0 (23/11/12)
Updates:
- PMAT添加可选参数`-tp`用于组装指定类型的细胞器基因组

PMAT version 1.3.0 (23/9/25)
Updates:
- 1.3.0之前的版本使用singularity
- 1.3.0之后的版本使用apptainer，并实现多个任务并行处理

<p style="color: red"><b>软件已经公布在github(https://github.com/bichangwei/PMAT)，期待大家宝贵的建议！</b></p>