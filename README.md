#报名信息即将公布，请大家持续关注！

# 我们创建了一个钉钉交流群，欢迎加入！

第三届汉语框架语义解析评测交流群-钉钉群号: 101450015383

二维码：

<div align=center>
<img src="./钉钉群码.jpg" width="30%" height="30%" />
</div>

加群时请提供您的机构、姓名、邮箱、电话信息！

# 总体概述

框架语义解析（Frame Semantic Parsing，FSP）是一项基于框架语义学的细粒度语义分析任务，其目标是从句子中提取框架语义结构，实现对句子中事件或情境的深层理解。框架语义解析对阅读理解、文本摘要、关系抽取等下游任务具有重要意义。

然而，句子中的语言成分普遍存在语义角色嵌套的现象。例如，在句子“我的眼睛什么也看不见了。”中，“我的眼睛”充当了【自由感知】框架中的“身体部位”角色，同时“我”又充当了“自主感知者”的角色。传统的语义角色标注方法通常优先标注粒度较粗的“身体部位”角色，而忽视了粒度较细的“自主感知者”角色，导致部分语义信息的遗漏。此外，由于大模型的快速发展，粗粒度的语义分析（如：Semantic Role Labeling）几乎已经被解决，并且大模型已经可以较好地理解这些语义，并用于实际的任务中，但对于更加细粒度与复杂的语义场景，大模型仍有欠缺。

为了进一步的评估和提升模型在细粒度语言理解上的能力，我们推出了基于CFN2.1的评测任务。不同于基于CFN1.0和CFN2.0的前两届评测任务，本次评测更多地关注模型在面向语义嵌套现象时的分析能力，并改进现有分析工具在面临框架元素的嵌套和融合（例如在“雇佣秘书”中，“秘书”既是雇员也是职位）等语言现象时容易遗漏部分角色信息的问题。


# 任务介绍

汉语框架语义解析（Chinese FSP，CFSP）是基于汉语框架网(Chinese FrameNet, CFN)的语义解析任务，本次标注数据格式如下：

1. 标注数据的字段信息如下：  
   + sentence_id：例句id
   + cfn_spans：框架元素标注信息
   + frame：例句所激活的框架名称
   + target：目标词的相关信息
      + start：目标词在句中的起始位置
      + end：目标词在句中的结束位置
      + pos：目标词的词性
   + text：标注例句
   + word：例句的分词结果及其词性信息
 
   数据样例：
   ```json
   [{
      "sentence_id": 2611,
      "cfn_spans": [
         { "start": 0, "end": 2, "fe_abbr": "ent_1", "fe_name": "实体1" },
         { "start": 4, "end": 17, "fe_abbr": "ent_2", "fe_name": "实体2" }
      ],
      "frame": "等同",
      "target": { "start": 3, "end": 3, "pos": "v" },
      "text": "餐饮业是天津市在海外投资的重点之一。",
      "word": [
         { "start": 0, "end": 2, "pos": "n" },
         { "start": 3, "end": 3, "pos": "v" },
         { "start": 4, "end": 6, "pos": "nz" },
         { "start": 7, "end": 7, "pos": "p" },
         { "start": 8, "end": 9, "pos": "n" },
         { "start": 10, "end": 11, "pos": "v" },
         { "start": 12, "end": 12, "pos": "u" },
         { "start": 13, "end": 14, "pos": "n" },
         { "start": 15, "end": 16, "pos": "n" },
         { "start": 17, "end": 17, "pos": "wp" }
      ]
   }]
   ```
   
2. 框架信息在`frame_info.json`中，框架数据的字段信息如下：
   + frame_name：框架名称
   + frame_ename：框架英文名称
   + frame_def：框架定义  
   + fes：框架元素信息
      + fe_name：框架元素名称  
      + fe_abbr：框架元素缩写  
      + fe_ename：框架元素英文名称  
      + fe_def：框架元素定义  
   
   数据样例：
   ```json
   [{
   "frame_name": "等同",
   "frame_ename": "Equating",
   "frame_def": "表示两个实体具有相等、相同、同等看待等的关系。",
   "fes": [
      { "fe_name": "实体集", "fe_abbr": "ents", "fe_ename": "Entities", "fe_def": "具有同等关系的两个或多个实体" },
      { "fe_name": "实体1", "fe_abbr": "ent_1", "fe_ename": "Entity_1", "fe_def": "与实体2具有等同关系的实体" },
      { "fe_name": "实体2", "fe_abbr": "ent_2", "fe_ename": "Entity_2", "fe_def": "与实体1具有等同关系的实体" },
      { "fe_name": "施动者", "fe_abbr": "agt", "fe_ename": "Agent", "fe_def": "判断实体集具有同等关系的人。" },
      { "fe_name": "方式", "fe_abbr": "manr", "fe_ename": "Manner", "fe_def": "修饰用来概括无法归入其他更具体的框架元素的任何语义成分，包括认知的修饰（如很可能，大概，神秘地），辅助描述（安静地，大声地），和与事件相比较的一般描述（同样的方式）。" },
      { "fe_name": "时间", "fe_abbr": "time", "fe_ename": "Time", "fe_def": "实体之间具有等同关系的时间" }
   ]
   }]
   ```

本次评测共分为以下三个子任务：

* 子任务1: 框架识别（Frame Identification），识别句子中给定目标词激活的框架。  
* 子任务2: 论元范围识别（Argument Identification），识别句子中给定目标词所支配论元的边界范围。  
* 子任务3: 论元角色识别（Role Identification），预测子任务2所识别论元的语义角色标签。  


## 子任务1: 框架识别（Frame Identification）
### 1. 任务描述

框架识别任务是框架语义学研究中的核心任务，其要求根据给定句子中目标词的上下文语境，为其寻找一个可以激活的框架。框架识别任务是自然语言处理中非常重要的任务之一，它可以帮助计算机更好地理解人类语言，并进一步实现语言处理的自动化和智能化。具体来说，框架识别任务可以帮助计算机识别出句子中的关键信息和语义框架，从而更好地理解句子的含义。这对于自然语言处理中的许多任务都是至关重要的。

### 2. 任务说明

该任务给定一个包含目标词的句子，需要根据目标词语境识别出激活的框架，并给出识别出的框架名称。  

   1. 输入：句子相关信息（id和文本内容）及目标词。
   2. 输出：句子id及目标词所激活框架的识别结果，数据为json格式，<font color="red">所有例句的识别结果需要放在同一list中</font>,样例如下：  

      ```json
      [
         [2611, "事件发生场所停业"],
         [2612, "等同"],
         ...
      ]
      ```

### 3. 评测指标

   &emsp;&emsp;框架识别采用正确率作为评价指标：

   $$task1\_acc = 正确识别的个数 / 总数$$


## 子任务2: 论元范围识别（Argument Identification）

### 1. 任务描述

给定一句汉语句子及目标词，在目标词已知的条件下，从句子中自动识别出目标词所搭配的语义角色的边界。该任务的主要目的是确定句子中目标词所涉及的每个论元在句子中的位置。论元范围识别任务对于框架语义解析任务来说非常重要，因为正确识别谓词和论元的范围可以帮助系统更准确地识别论元的语义角色，并进一步分析句子的语义结构。

### 2. 任务说明

论元范围识别任务是指，在给定包含目标词的句子中，识别出目标词所支配的语义角色的边界。
   
   1. 输入：句子相关信息（id和文本内容）及目标词。
   2. 输出：句子id，及所识别出所有论元角色的范围，每组结果包含例句id：`task_id`, `span`起始位置, `span`结束位置，<font color="red">每句包含的论元数量不定，识别出多个论元需要添加多个元组，所有例句识别出的结果共同放存在一个list中</font>，样例如下：
      ```json
      [
         [ 2611, 0, 2 ],
         [ 2611, 4, 17],
         ...
         [ 2612, 5, 7],
         ...
      ]
      ```

### 3. 评测指标

   论元范围识别采用P、R、F1作为评价指标：
   
   
   $${\rm{precision}} = \frac{{{\rm{InterSec(gold,pred)}}}}{{{\rm{Len(pred)}}}}$$
   
   $${\rm{recall}} = \frac{{{\rm{InterSec(gold,pred)}}}}{{{\rm{Len(gold)}}}}$$
   
   $${\rm{task2\\_f1}} = \frac{{{\rm{2\*precision\*recall}}}}{{{\rm{precision}} + {\rm{recall}}}}$$    
   
   其中：gold 和 pred 分别表示真实结果与预测结果，InterSec(\*)表示计算二者共有的token数量， Len(\*)表示计算token数量。

## 子任务3: 论元角色识别（Role Identification）

### 1. 任务描述

框架语义解析任务中，论元角色识别任务是非常重要的一部分。该任务旨在确定句子中每个论元对应的框架元素，即每个论元在所属框架中的语义角色。例如，在“我昨天买了一本书”这个句子中，“我”是“商业购买”框架中的“买方”框架元素，“一本书”是“商品”框架元素。论元角色识别任务对于许多自然语言处理任务都是至关重要的，例如信息提取、关系抽取和机器翻译等。它可以帮助计算机更好地理解句子的含义，从而更准确地提取句子中的信息，进而帮助人们更好地理解文本。

### 2. 任务说明

论元角色识别任务是指，在给定包含目标词的句子中，识别出目标词所支配语义角色的角色名称，该任务需要论元角色的边界信息以及目标词所激活框架的信息（即子任务1和子任务2的结果）。  
框架及其框架元素的所属关系在`frame_info.json`文件中。  


   1. 输入：句子相关信息（id和文本内容）、目标词、框架信息以及论元角色范围。
   2. 输出：句子id，及论元角色识别的结果，示例中“实体集”和“施动者”是“等同”框架中的框架元素。<font color="red">注意所有例句识别出的结果应共同放存在一个list中</font>，样例如下：  
   
   ```json
   [
      [ 2611, 0, 2, "实体集" ],
      [ 2611, 4, 17, "施动者" ],
      ...
      [ 2612, 5, 7, "时间" ],
      ...
   ]
   ```

### 3. 评测指标
论元角色识别采用P、R、F1作为评价指标：

   $${\rm{precision}} = \frac{{{\rm{Count(gold \cap pred)}}}} {{{\rm{Count(pred)}}}}$$  
   
   $${\rm{recall}} = \frac{{{\rm{Count(gold \cap pred)}}}} {{{\rm{Count(gold)}}}}$$  
   
   $${\rm{task3\\_f1}} = \frac{{{\rm{2\*precision\*recall}}}}{{{\rm{precision}} + {\rm{recall}}}}$$  

   其中，gold 和 pred 分别表示真实结果与预测结果， Count(\*) 表示计算集合元素的数量。
  

# 赛程安排 

具体赛程安排和要求如下：  
1.  <font  color="red">报名时间：2月1日-5月19日</font>
2.  训练集、验证集发布：4月10日
3.  测试集发布：5月13日
4.  <font  color="red">测试集结果提交截止：5月19日 </font>
5.  <font  color="red">参赛队伍成绩及排名公示：5月31日 </font>
    

# 奖项信息
   本次评测将评选出如下奖项。
   由中国中文信息学会计算语言学专委会（CIPS-CL）为获奖队伍提供荣誉证书。
   
   |奖项|一等奖|二等奖|三等奖|
   |----|----|----|----|
   |数量|1名|1名| 1名 |
   |奖励|荣誉证书|荣誉证书|荣誉证书|

   一等奖1名，奖金合计2500元；
   二等奖1名，奖金合计1500元；
   三等奖1名，奖金合计1000元。

   本次评测奖金由北京并行科技股份有限公司赞助。

# 论文格式 
会议投稿需统一使用LaTeX模板。提交的论文最多包含 6 页正文，参考文献页数不限。由于本次会议采用双盲审稿，作者姓名和单位不能出现在投稿的论文中。因此，作者的自引不可采用“作者名字提出…”的方式，而是用“我们提出…”。不符合这些要求的论文将不经过完整的审稿流程而直接被拒稿。

# 组织者和联系人
评测组织者：李茹、谭红叶（山西大学）

任务联系人：许豪（山西大学硕士生，202322407052@email.sxu.edu.cn）
