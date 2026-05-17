In this task, you deploy the new Cloud Run function and trigger it so that the data is loaded into BigQuery.

In Cloud Shell, run the following command to install the two javascript libraries to read from Cloud Storage and store the output in BigQuery:

npm install @google-cloud/storage @google-cloud/bigquery

Copied!
Run the following command to deploy the function:

gcloud functions deploy loadBigQueryFromAvro \
    --gen2 \
    --runtime nodejs24 \
    --source . \
    --region $REGION \
    --trigger-resource gs://$PROJECT_ID \
    --trigger-event google.storage.object.finalize \
    --memory=512Mi \
    --timeout=540s \
    --service-account=$PROJECT_NUMBER-compute@developer.gserviceaccount.com 


Note: If you see an error message relating to eventarc service agent propagation, wait a few minutes and try the command again.
Run the following command to confirm that the trigger was successfully created. The output will be similar to the following:

gcloud eventarc triggers list --location=$REGION


NAME: loadbigqueryfromavro-177311
TYPE: google.cloud.storage.object.v1.finalized
DESTINATION: Cloud Functions: loadBigQueryFromAvro
ACTIVE: Yes
LOCATION: europe-west1
Run the following command to download the Avro file that will be processed by the Cloud Run function for storage in BigQuery:

wget https://storage.googleapis.com/cloud-training/dataengineering/lab_assets/idegc/campaigns.avro

Run the following command to move the Avro file to the staging Cloud Storage bucket you created earlier. This action will trigger the Cloud Run function:

gcloud storage cp campaigns.avro gs://qwiklabs-gcp-00-30ce5a325be9

bq query \
 --use_legacy_sql=false \
 'SELECT * FROM `loadavro.campaigns`;'

gcloud logging read "resource.labels.service_name=loadBigQueryFromAvro"

该实验核心说明内容（精简总结）
核心技术场景
演示GCP 事件驱动架构：基于 Cloud Run Functions（第二代云函数），实现云端文件上传自动触发数据入库的自动化数据流转。
完整业务流程
当Cloud Storage（GCS） 上传 Avro 格式数据文件后，自动触发云函数，自动读取存储文件、自动在 BigQuery 创建数据表、批量加载结构化数据，完成离线数据同步入库。
关键产品联动
串联 GCP 多款核心服务：
Cloud Storage（文件存储）+ Cloud Run Function（无服务器事件计算）+ BigQuery（数仓分析）+ Eventarc（事件触发器）+ IAM 权限管控，展示云上数据工程的基础组合方案。
技术价值与设计思想
采用Serverless 无服务器架构，函数仅在事件触发时运行，按需计费、节约资源；
支持 Avro 列式数据格式自动识别、表自动创建、覆盖写入，适配批量数据接入；
配套日志审计、权限绑定、API 启用等企业级配置，贴合金融 / 企业数据工程落地规范（适配 HSBC 这类企业云上数据场景）。
学习目标
掌握 GCP 无服务器函数开发部署、事件触发器配置、跨服务权限配置、云端日志排查，理解自动化 ETL 轻量化落地的实现方式。
一句话极简概括
本实验演示如何借助 GCP 无服务器云函数，实现云存储文件上传 → 自动触发 → 自动入库 BigQuery的轻量化、自动化数据 ETL 流程。
