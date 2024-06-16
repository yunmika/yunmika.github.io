---
layout:     post
title:      "「mitotools」从细胞器genbank文件中快速提取序列"
subtitle:   "Fast sequence extraction from organelle genbank files"
date:       2024-06-16 22:00:00
author:     "云伴风行"
header-img: "img/blog-life4.jpg"
header-mask: 50%
catalog: true
tags:
    - 细胞器基因组
    - get_seq
    - mitotools
    - C

---

细胞器基因组分析相关脚本添加新的功能[get_seq](https://github.com/yunmika/mitotools)，该脚本由C编写，可以快速根据细胞器基因组的gb文件提取注释序列。我觉得这是一个比较实用的功能，当我们做进化分析时，批量下载gb文件，然后使用`get_seq`批量获取cds和pep序列，拒绝点点点 :)

# 使用说明

- get_seq

  get_seq can be used to quickly extract annotated sequences from genbank files of mitochondrial genomes such as faa, cds, pep, trn, rrn.

  Run `get_seq --help` to show the program's usage guide.

  ```
  Usage:./get_seq -g <genbank_file> -a
  Required options:
     -g, --genbank  Intput genbank file
  Optional options:
     -pre, --prefix  Prefix of the output
     -a, --all    Flag to output all annotations
     -f, --faa    Flag to output fasta
     -p, --pep    Flag to output pep
     -c, --cds    Flag to output cds
     -t, --trn    Flag to output trn
     -r, --rrn    Flag to output rrn
     -o, --output The output path
     -h, --help      Display this help message
  
  ```

# 输出结果

```bash
$ ./get_seq -g PPtest.gb -a -o ./
[2024-06-16 22:08:45] INFO: The genbank file: PPtest.gb
[2024-06-16 22:08:45] INFO: The prefix: PPtest
[2024-06-16 22:08:45] INFO: The output path: ./
[2024-06-16 22:08:45] INFO: CDS sequences saved to ./PPtest.cds
[2024-06-16 22:08:45] INFO: rRNA sequences saved to ./PPtest.rrn
[2024-06-16 22:08:45] INFO: tRNA sequences saved to ./PPtest.trn
[2024-06-16 22:08:45] INFO: Pep sequences saved to ./PPtest.pep
[2024-06-16 22:08:45] INFO: Faa sequences saved to ./PPtest.faa
```

直接下载编译后的文件`get_seq`就可以使用，如果有其他想要实现的功能，可以在[mitotools](https://github.com/yunmika/mitotools)告诉我们。