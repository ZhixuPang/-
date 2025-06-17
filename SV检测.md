## 结构变异检测软件简介及安装使用指南

### 1. Delly2

**简介**
Delly2 是一款广泛使用的结构变异检测软件，能检测包括缺失（DEL）、插入（INS）、倒位（INV）、易位（TRA）等多种类型SV。它基于短序列比对的配对末端信息和分裂读取进行鉴定，支持单样本和多样本联合调用。

**Conda 安装**

```bash
conda install -c bioconda delly
```

**基本使用**

* 单样本SV检测：

  ```bash
  delly call -g reference.fa -o sample.bcf sample.bam
  ```
* 多样本合并SV位点：

  ```bash
  delly merge -o all.sites.bcf sample1.bcf sample2.bcf sample3.bcf
  ```
* 对合并的位点进行基因型调用：

  ```bash
  delly call -g reference.fa -v all.sites.bcf -o sample1.geno.bcf sample1.bam
  ```
* 合并多个样本的基因型结果：

  ```bash
  bcftools merge -m id -O b -o merged.bcf sample1.geno.bcf sample2.geno.bcf sample3.geno.bcf
  ```

---

### 2. Smoove （Lumpy的集成封装）

**简介**
Smoove 是对 Lumpy 的封装与流程简化工具，适合大规模样本SV检测，集成预处理、调用、合并和过滤等步骤，方便批量分析。

**Conda 安装**

```bash
conda install -c bioconda smoove
```

**基本使用**

* 单样本调用SV：

  ```bash
  smoove call --outdir results/ --name sample --fasta reference.fa -p 4 --genotype sample.bam
  ```
* 合并多个样本的SV位点：

  ```bash
  smoove merge --name merged -f reference.fa --outdir ./ results/*.genotyped.vcf.gz
  ```
* 对合并的位点进行基因型调用：

  ```bash
  smoove genotype -d -x -p 4 --name sample-joint --outdir results-genotyped/ --fasta reference.fa --vcf merged.sites.vcf.gz sample.bam
  ```
* 过滤QUAL=0的低质量变异：

  ```bash
  bcftools filter -e 'QUAL==0' -o filtered_sample.vcf results-genotyped/sample-joint.genotyped.vcf.gz
  ```

---

### 3. Manta

**简介**
Manta 是一款快速灵敏的结构变异和融合基因检测软件，支持单样本或多样本联合分析，能准确检测多种SV类型。

**Conda 安装**

```bash
conda install -c bioconda manta
```

**基本使用**

* 配置分析目录（多样本联合调用）：

  ```bash
  configManta.py --bam sample1.bam --bam sample2.bam --bam sample3.bam --referenceFasta reference.fa --runDir manta_run
  ```
* 运行SV检测：

  ```bash
  manta_run/runWorkflow.py -m local -j 8
  ```
* 过滤低质量SV（QUAL < 20）：

  ```bash
  bcftools filter -i 'QUAL>=20' -o manta_filtered.vcf manta_run/results/variants/diploidSV.vcf.gz
  ```

---

### 多软件SV结果合并指南

为了提高SV检测的准确性和全面性，常将不同软件的结果合并。推荐使用 **SURVIVOR** 工具，支持多软件SV合并，考虑位点距离和SV类型。

**安装SURVIVOR**

```bash
conda install -c bioconda survivor
```

**合并流程示例**

1. 准备待合并VCF文件列表（均经过格式转换和过滤）：

```bash
echo "delly.filter.vcf" > sv_list.txt
echo "smoove.filter.vcf" >> sv_list.txt
echo "manta.filter.vcf" >> sv_list.txt
```

2. 运行合并（最大距离1kb，至少2个软件支持，考虑类型和方向）：

```bash
SURVIVOR merge sv_list.txt 1000 2 1 1 0 50 merged_SVs.vcf
```

---

### 软件比较汇总

| 软件名    | 主要特点                 | Conda安装命令                          | 适用场景            |
| ------ | -------------------- | ---------------------------------- | --------------- |
| Delly2 | 多样本联合调用，支持多种SV类型     | `conda install -c bioconda delly`  | 精准SV调用，中小规模样本   |
| Smoove | Lumpy封装，自动化流程，批量样本友好 | `conda install -c bioconda smoove` | 大规模样本结构变异检测     |
| Manta  | 快速灵敏，支持融合基因检测        | `conda install -c bioconda manta`  | 灵敏高效，适合快速联合SV检测 |

---

如需完整示例脚本或更多细节，欢迎告诉我！
