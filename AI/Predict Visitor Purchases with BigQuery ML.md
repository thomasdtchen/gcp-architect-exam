# Overview
BigQuery Machine Learning (BigQuery ML) enables users to create and execute machine learning models in BigQuery using SQL queries. The goal is to democratise machine learning by enabling SQL practitioners to build models using their existing tools and to increase development speed by eliminating the need for data movement.

There is an ecommerce dataset that has millions of Google Analytics records for the Google Merchandise Store loaded into BigQuery. In this lab you will use this data to create a model that predicts whether a visitor will make a transaction.

-- Create a model name sample_model, type logistic_reg to predict whether a visitor will make a transaction using the training_data view table.
CREATE MODEL
  `qwiklabs-gcp-03-8a21ed9a0102.bqml_lab.sample_model` OPTIONS ( MODEL_TYPE = 'LOGISTIC_REG',
    INPUT_LABEL_COLS = ['label']) AS
SELECT
  label,
  os,
  is_mobile,
  country,
  pageviews
FROM
  `qwiklabs-gcp-03-8a21ed9a0102`.`bqml_lab`.`training_data`;


  -- Write a query to evaluate the performance of the model `bqml_lab.sample_model` using the `ml.EVALUATE` function.
SELECT
  *
FROM
  ML.EVALUATE( MODEL `qwiklabs-gcp-03-8a21ed9a0102`.`bqml_lab`.`sample_model`,
    TABLE `qwiklabs-gcp-03-8a21ed9a0102`.`bqml_lab`.`training_data`);


SELECT
  country,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(MODEL `bqml_lab.sample_model`, (
SELECT * FROM `bqml_lab.july_data`))
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;

# Explain 
1. 你创建的模型名字就叫：
plaintext
sample_model
后面预测用的也是：
plaintext
MODEL `bqml_lab.sample_model`
这就是在调用你刚刚训练好的模型。
2. ml. 是 BigQuery 关键字吗？
是的！ml. 是 BigQuery ML 内置的固定前缀，不是你定义的，是系统自带的。
BigQuery ML 固定函数：
ML.PREDICT() → 做预测
ML.EVALUATE() → 评估模型
ML.CREATE_MODEL() → 创建模型（简写 CREATE MODEL）
你不需要定义 ml，直接用就行。

## create model 
一、CREATE MODEL 到底在做什么？（超级易懂版）
一句话总结
用历史数据，训练一个 “会不会购买” 的二分类预测模型。
逐行拆解你这段代码
sql
CREATE MODEL
  `qwiklabs-gcp-03-8a21ed9a0102.bqml_lab.sample_model`
OPTIONS ( 
  MODEL_TYPE = 'LOGISTIC_REG',   -- 模型类型：逻辑回归（二分类）
  INPUT_LABEL_COLS = ['label']   -- 要预测的目标列
) AS
SELECT
  label,        -- 标签：1=买了，0=没买（你要预测的东西）
  os,           -- 特征：手机系统
  is_mobile,    -- 特征：是否手机访问
  country,      -- 特征：国家
  pageviews     -- 特征：浏览页数
FROM
  `qwiklabs-gcp-03-8a21ed9a0102.bqml_lab.training_data`
训练逻辑（人话）
label = 你要预测的结果
1 = 访客最终购买了
0 = 访客没购买
os /is_mobile/country /pageviews = 输入特征
模型靠这些信息来判断：
从什么国家来？
用手机还是电脑？
看了几页？
→ 然后预测会不会买。
MODEL_TYPE = LOGISTIC_REG
逻辑回归 = 二分类模型
专门用来预测：
是 / 否
买 / 不买
会 / 不会
正好适合这个业务场景。

三、预测语句到底在干嘛？（你这段）
sql
SELECT
  country,
  SUM(predicted_label) as total_predicted_purchases
FROM
  ml.PREDICT(
    MODEL `bqml_lab.sample_model`,  -- 调用你创建的模型
    (SELECT * FROM `bqml_lab.july_data`)
  )
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;
人话翻译：
把 7 月份的新数据 丢进模型
模型自动给每一行输出：
predicted_label = 1 → 预测会购买
predicted_label = 0 → 预测不会购买
按国家分组
把每个国家预测会购买的人数加总
取 TOP10 购买国家

四、整个实验的完整逻辑（你一看就懂）
1. 准备训练数据
plaintext
label (买/不买) + 特征 (系统、手机、国家、页数)
2. 训练模型
plaintext
CREATE MODEL sample_model
让模型学习：
什么特征的人更容易购买
3. 用模型预测新数据
plaintext
ML.PREDICT(MODEL sample_model, 新数据)
4. 统计每个国家预测购买人数
plaintext
SUM(predicted_label)

## note
bqml_lab 是你当前实验里，BigQuery 里的一个「数据集（Dataset）」，专门用来存放这个 BigQuery ML 实验用到的所有表、视图和模型。



