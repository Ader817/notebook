# eICU-tutorial

## 1 Introduction

[eICU](https://eicu-crd.mit.edu/) 协作研究数据库汇集了来自美国本土众多重症监护室的综合数据，该协作数据库收录了 2014 年至 2015 年期间入住重症监护室的患者的相关数据

- eICU Documents: https://eicu-crd.mit.edu/about/eicu/

- eiCu Data: https://eicu-crd.mit.edu/gettingstarted/access/

  > 获取 eICU 完整数据需要完成 CITI 课程，并在 PhysioNet 上提交结业报告，且需要在申请表单中填写推荐人（即导师的姓名和联系方式）

- **eicu-code**: https://github.com/mit-lcp/eicu-code

  ```shell
  eicu-code/notebooks/
  ├── admissiondrug.ipynb
  ├── admissiondx.ipynb
  ├── allergy.ipynb
  ├── apache.ipynb
  ├── careplan.ipynb
  ├── cross-reference-apache-treatments.ipynb
  ├── demo
  │   ├── 01-explore-patient-table.ipynb
  │   ├── 02-demographics-and-severity-of-illness.ipynb
  │   ├── 03-plot-timeseries.ipynb
  │   └── README.md
  ├── diagnosis.ipynb
  ├── hospital.ipynb
  ├── infusiondrug.ipynb
  ├── intakeoutput.ipynb
  ├── lab.ipynb
  ├── medication.ipynb
  ├── nursecharting.ipynb
  ├── pasthistory.ipynb
  ├── patient.ipynb
  ├── treatment.ipynb
  ├── vitalaperiodic.ipynb
  └── vitalperiodic.ipynb
  ```
  
  `notebooks/` 中包含使用 eicu 数据库的示例，示例中主要是通过 **pandas**, **matplotlib**, **sqlite3**, **psycopg2** 库对数据进行简单的处理和绘图展示
  
  - `notebooks/demo` 中的几个 ipynb 作为简单的学习示例供使用者学习 eicu 数据库的表格及其处理，文件中还会列举出几个问题帮助思考。
  
    使用前需要从 https://physionet.org/content/eicu-crd-demo/2.0.1/ 中下载一个 **eICU Database demo** （完整数据库中有超过 20 万例入院患者的数据，demo 中只有 2500 余例数据，包含所有表格的 .csv 文件以及对 csv 进行整理制作的 .sqlite3 文件）放在 `demo/` 下
  
  - 其他部分的 ipynb 分别是对==完整数据库==中的不同表格的数据处理示例，需要获取完整的数据库（使用可以参考 https://eicu-crd.mit.edu/eicutables/vitalperiodic/）

## 2 Tables

### 2.1 Overview

!!! quote
    eicu-schema-spy: [https://lcp.mit.edu/eicu-schema-spy/](https://lcp.mit.edu/eicu-schema-spy/)

| Table                                                        | Children | Parents | Columns | Rows            |
| :----------------------------------------------------------- | :------- | :------ | :------ | :-------------- |
| [admissiondrug](https://lcp.mit.edu/eicu-schema-spy/tables/admissiondrug.html) |          | 1       | 14      | 874,920         |
| [admissiondx](https://lcp.mit.edu/eicu-schema-spy/tables/admissiondx.html) |          | 1       | 6       | 626,858         |
| [allergy](https://lcp.mit.edu/eicu-schema-spy/tables/allergy.html) |          | 1       | 13      | 251,949         |
| [apacheapsvar](https://lcp.mit.edu/eicu-schema-spy/tables/apacheapsvar.html) |          | 1       | 26      | 171,177         |
| [apachepatientresult](https://lcp.mit.edu/eicu-schema-spy/tables/apachepatientresult.html) |          | 1       | 23      | 297,064         |
| [apachepredvar](https://lcp.mit.edu/eicu-schema-spy/tables/apachepredvar.html) |          | 1       | 51      | 171,177         |
| [careplancareprovider](https://lcp.mit.edu/eicu-schema-spy/tables/careplancareprovider.html) |          | 1       | 8       | 502,765         |
| [careplaneol](https://lcp.mit.edu/eicu-schema-spy/tables/careplaneol.html) |          | 1       | 5       | 1,433           |
| [careplangeneral](https://lcp.mit.edu/eicu-schema-spy/tables/careplangeneral.html) |          | 1       | 6       | 3,115,018       |
| [careplangoal](https://lcp.mit.edu/eicu-schema-spy/tables/careplangoal.html) |          | 1       | 7       | 504,139         |
| [careplaninfectiousdisease](https://lcp.mit.edu/eicu-schema-spy/tables/careplaninfectiousdisease.html) |          | 1       | 8       | 8,056           |
| [customlab](https://lcp.mit.edu/eicu-schema-spy/tables/customlab.html) |          | 1       | 7       | 1,082           |
| [diagnosis](https://lcp.mit.edu/eicu-schema-spy/tables/diagnosis.html) |          | 1       | 7       | 2,710,672       |
| [hospital](https://lcp.mit.edu/eicu-schema-spy/tables/hospital.html) |          |         | 4       | 208             |
| [infusiondrug](https://lcp.mit.edu/eicu-schema-spy/tables/infusiondrug.html) |          | 1       | 9       | 4,803,719       |
| [intakeoutput](https://lcp.mit.edu/eicu-schema-spy/tables/intakeoutput.html) |          | 1       | 12      | 12,030,289      |
| [==lab==](https://lcp.mit.edu/eicu-schema-spy/tables/lab.html) |          | 1       | 10      | 39,132,531      |
| [medication](https://lcp.mit.edu/eicu-schema-spy/tables/medication.html) |          | 1       | 15      | 7,301,853       |
| [microlab](https://lcp.mit.edu/eicu-schema-spy/tables/microlab.html) |          | 1       | 7       | 16,996          |
| [note](https://lcp.mit.edu/eicu-schema-spy/tables/note.html) |          | 1       | 8       | 2,254,179       |
| [nurseassessment](https://lcp.mit.edu/eicu-schema-spy/tables/nurseassessment.html) |          | 1       | 8       | 15,602,498      |
| [nursecare](https://lcp.mit.edu/eicu-schema-spy/tables/nursecare.html) |          | 1       | 8       | 8,311,132       |
| [nursecharting](https://lcp.mit.edu/eicu-schema-spy/tables/nursecharting.html) |          | 1       | 8       | 151,604,232     |
| [pasthistory](https://lcp.mit.edu/eicu-schema-spy/tables/pasthistory.html) |          | 1       | 8       | 1,149,180       |
| [==patient==](https://lcp.mit.edu/eicu-schema-spy/tables/patient.html) | 29       |         | 29      | 200,859         |
| [physicalexam](https://lcp.mit.edu/eicu-schema-spy/tables/physicalexam.html) |          | 1       | 6       | 9,212,316       |
| [respiratorycare](https://lcp.mit.edu/eicu-schema-spy/tables/respiratorycare.html) |          | 1       | 34      | 865,381         |
| [respiratorycharting](https://lcp.mit.edu/eicu-schema-spy/tables/respiratorycharting.html) |          | 1       | 7       | 20,168,176      |
| [treatment](https://lcp.mit.edu/eicu-schema-spy/tables/treatment.html) |          | 1       | 5       | 3,688,745       |
| [==vitalaperiodic==](https://lcp.mit.edu/eicu-schema-spy/tables/vitalaperiodic.html) |          | 1       | 13      | 25,075,074      |
| [==vitalperiodic==](https://lcp.mit.edu/eicu-schema-spy/tables/vitalperiodic.html) |          | 1       | 19      | 146,671,642     |
|                                                              |          |         |         |                 |
| **31 Tables**                                                |          |         | **391** | **457,325,320** |

### 2.1 Identifiers

标识符在整个数据库中被用于识别独特的概念，如患者、医院、ICU 住院记录等。

- `hospitalid` - 用于唯一标识数据库中的每家医院

- `uniquepid` - 用于唯一标识患者（即同一人的该值始终不变）
- `patienthealthsystemsstayid` - 用于唯一标识住院记录
- `patientunitstayid` - 唯一标识单元住院记录（即指医院内的重症监护室）

> 事实上，**patientunitstayid** 是最关键的“标识符”，即 patient 表格的主键以及几乎所有其他表格的外键(https://lcp.mit.edu/eicu-schema-spy/relationships.html)

### 2.2 Key Tables

#### 2.2.1 Patient Table

> https://lcp.mit.edu/eicu-schema-spy/tables/patient.html

包含 patient 的个人信息，`patientunitstayid` 作为主键，连接其他几乎所有 table

#### 2.2.2 Lab Table

> https://eicu-crd.mit.edu/eicutables/lab/
>

主要是一些**代谢和生化的**数据，包含的项目包括：

- 血气：pH、PCO₂、PO₂、Lactate
- 电解质：Na、K、Cl、Mg
- 肝功能：ALT、AST、TBili
- 肾功能：Creatinine、BUN
- 血糖：Glucose
- 炎症标志物：WBC 

示例：

| ==labid== | ==patientunitstayid== | labresultoffset | labtypeid | ==labname==      | ==labresult== | labresulttext | labmeasurenamesystem | labmeasurenameinterface | labresultrevisedoffset |      |
| --------- | --------------------- | --------------- | --------- | ---------------- | ------------- | ------------- | -------------------- | ----------------------- | ---------------------- | ---- |
| 437880563 | 1754323               | -647            | 3         | Hct              | 38.3          | 38.3          | %                    | %                       | -631                   |      |
| 437880572 | 1754323               | -647            | 3         | platelets x 1000 | 181           | 181           | K/mcL                | k/mm cu                 | -631                   |      |
| 437880560 | 1754323               | -647            | 3         | RBC              | 4.86          | 4.86          | M/mcL                | m/mm cu                 | -631                   |      |

#### 2.2.3 Diagnosis Table

> https://eicu-crd.mit.edu/eicutables/diagnosis/

主要记录患者的诊断记录，表中包含有 corresponding International Classification of Diseases (ICD) 即 [ICD](https://en.wikipedia.org/wiki/List_of_ICD-9_codes) 编码

示例：

| ==diagnosisid== | ==patientunitstayid== | activeupondischarge | diagnosisoffset | diagnosisstring                                              | ==icd9code==   | diagnosispriority |
| --------------- | --------------------- | ------------------- | --------------- | ------------------------------------------------------------ | -------------- | ----------------- |
| 7607199         | 346380                | FALSE               | 5028            | cardiovascular\|ventricular disorders\|hypertension          | 401.9, I10     | Other             |
| 7570429         | 346380                | FALSE               | 685             | neurologic\|altered mental status / pain\|change in mental status | 780.09, R41.82 | Major             |
| 7705483         | 346380                | TRUE                | 5035            | cardiovascular\|shock / hypotension\|hypotension             | 458.9, I95.9   | Major             |

#### 2.2.4 VitalPeriodic / VitalAperidoic Table

> https://eicu-crd.mit.edu/eicutables/vitalaperiodic/
>
> https://eicu-crd.mit.edu/eicutables/vitalperiodic/

这些表格记录 ICU 患者**时间序列生理状态**，包含的项目包括：

- 心率 HR
- 血压 SBP/DBP/MAP
- 呼吸率 RR
- 血氧 SpO₂
- 体温 Temp 等等

## 3 Usage Example

### 3.1 HyperEHR

> HyperEHR: https://github.com/LZlab01/HyperEHR#

Code for "Hypergraph Convolutional Networks for Fine-grained ICU Patient Similarity Analysis and Risk Prediction". Paper: https://arxiv.org/abs/2308.12575

构建一个能够利用 ICU 患者电子病历（EHR）数据的患者超图（Hypergraph），并基于这个超图进行患者相似度学习与风险预测，主要用到以下表格：

- patient.csv
- diagnosis.csv (==[icd9Code](https://en.wikipedia.org/wiki/List_of_ICD-9_codes)==)
- vitalPeriodic.csv / vitalAperiodic.csv

**数据流**：

原始数据（raw CSV） -> 清洗合并后的 pandas DataFrame -> 时间序列矩阵、诊断特征矩阵（numpy arrays） -> PyTorch Dataset / Tensor（模型输入）-> 模型输出（logit / 概率）

### 3.2 YAIB

> YAIB: https://github.com/rvandewater/YAIB#">https://github.com/rvandewater/YAIB#

🧪Yet Another ICU Benchmark: a holistic framework for the standardization of clinical prediction model experiments. Provide custom datasets, cohorts, prediction tasks, endpoints, preprocessing, and models. Paper: https://arxiv.org/abs/2306.05109

**TBD**
