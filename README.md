# bi-lstm-crf

## 简介

采用序列标注的方式实现中文分词，模型架构为Embedding + Bi-LSTM + CRF，参考论文：https://arxiv.org/abs/1508.01991。

## 目录说明

1. model

在`model`文件夹中，包含使用人民日报80万语料和使用[百度百科训练得到的词向量](https://pan.baidu.com/s/1eeCS7uD3e_qVN8rPwmXhAw)迭代**15**次后获得的模型文件：
* model.final.h5 模型权重文件
* model.dict 处理语料后生产的词库
* model.cfg 模型的配置文件

2. corups

在`corups`文件夹中，包含人民日报80万语料的部分示例

3. score

在`score`文件夹中，包含用于评估分词效果的示例文件

## 分词

使用命令：
```py
predict.py -s <语句>
```
进行分词，单独一句话不可超过150词，否则会截断。多句话可用符号（|）分隔，具体使用方式可使用`predict.py -h`查看。

### 分词效果展示

1. 科技类

```py
python predict.py -s "目前多任务学习方法大致可以总结为两类，一是不同任务之间共享相同的参数，二是挖掘不同任务之间隐藏的共有数据特征。"
```
>  '目前', '多', '任务', '学习', '方法', '大致', '可以', '总结', '为', '两类', '，', '一', '是', '不同', '任务', '之间', '共享', '相同', '的', '参数', '，', '二', '是', '挖掘', '不同', '任务', '之间', '隐藏', '的', '共有', '数据', '特征', '。'

2. 新闻类

```py
python predict.py -s "美国司法部副部长罗森·施泰因（Rod Rosenstein）指，这些俄罗斯情报人员涉嫌利用电脑病毒或“钓鱼电邮”，成功入侵民主党的电脑系统，偷取民主党高层成员之间的电邮，另外也从美国一个州的电脑系统偷取了50万名美国选民的资料。"
```

 > '美国司法部', '罗森·施泰因', '（', 'Rod', ' ', 'Rosenstein', '）', '指', '，', '这些', '俄罗斯', '情报', '人员', '涉嫌', '利用', '电脑病毒', '或', '“', '钓鱼', '电邮', '”', '，', '成功', '入侵', '民主党', '的', '电脑系统', '，', '偷取', '民主党', '高层', '成员', '之间', '的', '电邮', '，', '另外', '也', '从', '美国', '一个', '州', '的', '电脑系统', '偷取', '了', '50万名', '美国', '选民', '的', '资料', '。'

## 数据预处理

原始的语料为人民日报的 80 万语料（语料库很大，可在附录获得）。
这些语料已经对语句进行了切分和词性标注，为了转换为模型可用的语料，使用命令:
```python
python preprocess_data.py <语料目录> train.data -a
```
将原始的有词性标注的文档转换为使BIS（B:表示语句块的开始，I:表示非语句块的开始，S:表示单独成词）标注的文件。

## 训练

可使用`train.py`进行模型的训练，详细使用方式见`train.py -h`。

### 使用字向量

1. 使用预训练的字向量：
    使用`--embedding_file_path <字向量文件>`利用预训练的字向量进行训练。
2. 字向量格式
    字向量文件中每一行格式为一个字与其对应的300维向量。可从[https://github.com/Embedding/Chinese-Word-Vectors](https://github.com/Embedding/Chinese-Word-Vectors)获得字向量。

### 训练效果

1. 精度

    在迭代**15**次后，训练集精度达到了**99.3%**，验证集精度达到了**99.4%**。
2. 耗时
   
   训练时在**1070 ti** 上耗费**约2小时**左右。
   
3. 在人民日报2014年750篇新闻上的分词评估结果如下
    * 标准词数：196658个，正确词数：190436个，错误词数：5986个
    * 标准行数：5505，正确行数：3200，错误行数：2305
    * Recall: 0.968361
    * Precision: 0.969525
    * F MEASURE: 0.968943
    * ERR RATE: 0.030439

## 评估

使用与黄金标准文件进行对比的方式，进行评估。

1. 数据预处理

为了生成黄金标准文件和无分词标记的原始文件，可用下列命令：
```python
python score_preprocess.py --corups_dir <语料文件夹> \
--gold_file_path <生成的黄金标准文件路径> \
--restore_file_path <生成无标记的原始文件路径>
```

2. 读取无标记的原始文件，并进行分词，输出到文件：

```python
python predict.py -tf <要分割的文本文件的路径> -pf <保存分词结果的文件路径>
```

3. 生成评估结果：

执行`score.py`可生成评估文件，默认使用黄金分割文件`./score/gold.utf8`，使用模型分词后的文件`./score/pred_text.utf8`，评估结果保存到`prf_tmp.txt`中。

```py
def main():
    F = prf_score('./score/gold.utf8', './score/pred_text.utf8', 'prf_tmp.txt', 15)
```



## 附录

1. 分词语料库： https://pan.baidu.com/s/1-LLzKOJglP5W0VCVsI0efg 密码: krhr 
