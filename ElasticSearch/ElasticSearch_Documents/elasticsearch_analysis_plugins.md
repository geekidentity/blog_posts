---
categories: elasticsearch

tags: 
  - Elasticsearch
  - Elasticsearch插件

title: Elasticsearch插件和集成[5.5] -- 分析插件

date: 2017-07-13
---

分析插件通过向Elasticsearch添加新的分析器，标记器，令牌过滤器或字符过滤器来扩展Elasticsearch。

### 核心分析插件包括：

* [ICU](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/analysis-icu.html)：使用[ICU库](http://site.icu-project.org/)增加扩展的Unicode支持，包括更好地分析亚洲语言，Unicode规范化，Unicode感知案例折叠，排序规则支持和音译。
* [Kuromoji](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/analysis-kuromoji.html)：使用[Kuromoji分析器](http://www.atilika.org/)对日本语进行高级分析。
* [Phonetic](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/analysis-phonetic.html)：使用Soundex，Metaphone，Caverphone和其他编解码器将tokens 分析成其语音等效。
* [SmartCN](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/analysis-smartcn.html)：中文或混合中文 - 英文文本分析器。 该分析仪使用概率知识找到简体中文文本的最佳分词。 文本首先被分成句子，然后将每个句子分割成单词。
* [Stempel](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/analysis-stempel.html)：为波兰语提供高品质的词干。
* [Ukrainian](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/analysis-ukrainian.html)：乌克兰语

### 社区贡献分析插件

* [Hebrew Analysis Plugin](https://github.com/synhershko/elasticsearch-analysis-hebrew) (by Itamar Syn-Hershko)
* [IK Analysis Plugin](https://github.com/medcl/elasticsearch-analysis-ik) (by Medcl)
* [Mmseg Analysis Plugin](https://github.com/medcl/elasticsearch-analysis-mmseg) (by Medcl)
* [Russian and English Morphological Analysis Plugin](https://github.com/medcl/elasticsearch-analysis-pinyin) (by Igor Motov)
* [Pinyin Analysis Plugin](https://github.com/medcl/elasticsearch-analysis-pinyin) (by Medcl)
* [Vietnamese Analysis Plugin](https://github.com/duydo/elasticsearch-analysis-vietnamese) (by Duy Do)
* [Network Addresses Analysis Plugin](https://github.com/ofir123/elasticsearch-network-analysis) (by Ofir123)
* [String2Integer Analysis Plugin](https://github.com/medcl/elasticsearch-analysis-string2int) (by Medcl)

# ICU分析插件

ICU Analysis插件将Lucene ICU模块集成到弹性搜索中，使用[ICU库](http://site.icu-project.org/)添加扩展的Unicode支持，包括对亚洲语言的更好分析，Unicode标准化，Unicode感知案例折叠，排序规则支持和音译。

ICU分析器和向后兼容性

