---
layout: post
title: 2021年04月06日 ElasticSearch 分词器（插件）安装
date: 2021-04-06
description: "ElasticSearch 分词器安装"
tags: ElasticSearch 插件 IK 分词器 kuromoji
---   
### ES 分词器（elasticsearch analysis）

#### 索引
##### 正排索引，倒排索引（https://www.it610.com/article/1297280663585628160.htm）

#### 分词器

Analysis(分词）： 文本分析是把全文本转换一系列单词(term/token)的过程，也叫分词。
Analysis是通过Analyzer来实现的。

Analysis组成（分词流程）
- Character Filters：针对原始文本进行处理，比如去除html标签；
- Tokenizer：将原始文本按照一定规则切分为单词；
- Token Filters：针对Tokenizer处理的单词进行再加工，比如转小写、删除或增新等处理；

![流程](https://gitee.com/dbin0123/picgo/raw/master/image/20210406222453.png)


>ES内置分词器

- Standard - 默认分词器，按词切分，小写处理
``` JSON
# 分词
GET /_analyze
{
  "analyzer":"standard",
  "text":"He gave his wife a ring as a token of his love."
}
# 响应
{
  "tokens": [
    {
      "token": "he",
      "start_offset": 0,
      "end_offset": 2,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "gave",
      "start_offset": 3,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "his",
      "start_offset": 8,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "wife",
      "start_offset": 12,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "a",
      "start_offset": 17,
      "end_offset": 18,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "ring",
      "start_offset": 19,
      "end_offset": 23,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "as",
      "start_offset": 24,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "a",
      "start_offset": 27,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "token",
      "start_offset": 29,
      "end_offset": 34,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "of",
      "start_offset": 35,
      "end_offset": 37,
      "type": "<ALPHANUM>",
      "position": 9
    },
    {
      "token": "his",
      "start_offset": 38,
      "end_offset": 41,
      "type": "<ALPHANUM>",
      "position": 10
    },
    {
      "token": "love",
      "start_offset": 42,
      "end_offset": 46,
      "type": "<ALPHANUM>",
      "position": 11
    }
  ]
}
```

- Simple - 按照非字母切分(符号被过滤), 小写处理
```JSON
# simple分词
GET /_analyze
{
  "analyzer":"simple",
  "text":"He gave his wife a ring as a token of his love."
}
# 响应
{
  "tokens": [
    {
      "token": "he",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 0
    },
    {
      "token": "gave",
      "start_offset": 3,
      "end_offset": 7,
      "type": "word",
      "position": 1
    },
    {
      "token": "his",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 2
    },
    {
      "token": "wife",
      "start_offset": 12,
      "end_offset": 16,
      "type": "word",
      "position": 3
    },
    {
      "token": "ring",
      "start_offset": 19,
      "end_offset": 23,
      "type": "word",
      "position": 5
    },
    {
      "token": "token",
      "start_offset": 29,
      "end_offset": 34,
      "type": "word",
      "position": 8
    },
    {
      "token": "his",
      "start_offset": 38,
      "end_offset": 41,
      "type": "word",
      "position": 10
    },
    {
      "token": "love",
      "start_offset": 42,
      "end_offset": 46,
      "type": "word",
      "position": 11
    }
  ]
}
```


- Stop - 小写处理，停用词过滤(the,a,is)
  > [英文停用词](http://www.ranks.nl/stopwords) [中文停用词](http://www.ranks.nl/stopwords/chinese-stopwords)
```JSON
# stop分词
GET /_analyze
{
  "analyzer":"stop",
  "text":"He gave his wife a ring as a token of his love."
}
# 响应
{
  "tokens": [
    {
      "token": "he",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 0
    },
    {
      "token": "gave",
      "start_offset": 3,
      "end_offset": 7,
      "type": "word",
      "position": 1
    },
    {
      "token": "his",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 2
    },
    {
      "token": "wife",
      "start_offset": 12,
      "end_offset": 16,
      "type": "word",
      "position": 3
    },
    {
      "token": "ring",
      "start_offset": 19,
      "end_offset": 23,
      "type": "word",
      "position": 5
    },
    {
      "token": "token",
      "start_offset": 29,
      "end_offset": 34,
      "type": "word",
      "position": 8
    },
    {
      "token": "his",
      "start_offset": 38,
      "end_offset": 41,
      "type": "word",
      "position": 10
    },
    {
      "token": "love",
      "start_offset": 42,
      "end_offset": 46,
      "type": "word",
      "position": 11
    }
  ]
}
```
- Whitespace - 按照空格切分，不转小写
- Keyword - 不分词，直接将输入当作输出
```JSON
# keyword分词
GET /_analyze
{
  "analyzer":"keyword",
  "text":"He gave his wife a ring as a token of his love."
}
# 输出
{
  "tokens": [
    {
      "token": "He gave his wife a ring as a token of his love.",
      "start_offset": 0,
      "end_offset": 47,
      "type": "word",
      "position": 0
    }
  ]
}
```
- Patter - 正则表达式，默认\W+(非字符分割)
- Language - 提供了30多种常见语言的分词器
- Customer 自定义分词器
  
#### IK分词器安装

1. 下载与ES对于的版本 https://github.com/medcl/elasticsearch-analysis-ik/releases
2. 解压安装包至ES config目录
3. 重启ES就可以使用 ik_max_word,ik_smart

##### 示例代码
>ik_max_word（细粒度拆分）

```JSON

# ik_max_word分词
GET /_analyze
{
  "analyzer":"ik_max_word",
  "text":"我是中国人"
}
# 结果
{
  "tokens": [
    {
      "token": "我",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "是",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "中国人",
      "start_offset": 2,
      "end_offset": 5,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "中国",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "国人",
      "start_offset": 3,
      "end_offset": 5,
      "type": "CN_WORD",
      "position": 4
    }
  ]
}
```

>ik_smart(粗粒度拆分)

```JSON
# ik_smart分词
GET /_analyze
{
  "analyzer":"ik_smart",
  "text":"我是中国人"
}
# 结果
{
  "tokens": [
    {
      "token": "我",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "是",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "中国人",
      "start_offset": 2,
      "end_offset": 5,
      "type": "CN_WORD",
      "position": 2
    }
  ]
}
```


##### 在线安装analysis-kuromoji分词器

- ./bin/elasticsearch-plugin install --help
![列举可以在线安装的插件](https://gitee.com/dbin0123/picgo/raw/master/image/20210406230253.png)

- 安装

>./es_master/bin/elasticsearch-plugin install analysis-kuromoji

![在线安装的插件](https://gitee.com/dbin0123/picgo/raw/master/image/20210406230852.png)

![测试](https://gitee.com/dbin0123/picgo/raw/master/image/20210406231541.png)