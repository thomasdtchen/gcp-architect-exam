GSP079【Cloud KMS 入门实验】极简核心总结（贴合汇丰金融场景、好记）
一、实验核心目的
用 Google Cloud KMS 密钥管理服务，对金融敏感数据做手动加密 / 解密，搭配云存储、IAM 权限、审计日志，完成云上敏感文件安全存储全流程，对应企业数据合规、加密管控需求。
二、核心知识点（按任务拆解）
Cloud Storage 基础
创建独立存储桶，用来存放加密后的机密文件
原始明文不直接上云，满足金融数据防泄露
Cloud KMS 核心概念
Keyring（密钥环）：密钥分组容器（按环境 / 业务分类，如财务、风控）
CryptoKey（加密密钥）：实际用来加解密的密钥
KMS 是谷歌全托管密钥服务，企业统一管密钥，不把密钥放代码 / 本地
手动加解密实操
文本先 Base64 编码，再调用 KMS API 加密
加密生成 ciphertext 密文，解密还原原文
批量脚本：批量加密文件夹所有文件，批量上传云端
KMS 精细化 IAM 权限
cloudkms.admin：密钥环 / 密钥 管理权限（创建、修改、禁用密钥）
cloudkms.cryptoKeyEncrypterDecrypter：业务核心权限，允许用密钥加解密
权限绑定在密钥环层级，下级密钥自动继承，符合企业权限分级
安全合规能力
加密文件上传存储桶，明文不外泄
Cloud Audit Logs 审计日志：记录密钥创建、加解密操作、操作人员
满足金融行业「谁操作、何时操作、密钥使用记录可追溯」
三、关键业务价值（对应 HSBC 这类银行）
敏感金融资料不能裸存云端，客户资料 / 财报 / 内部单据必须加密
集中托管密钥，避免员工私自保存、泄露密钥
权限分离：管密钥的人 ≠ 随便解密数据的人
全程审计留痕，满足监管合规
四、一句话极简背诵
这个实验就教三件事：
用 KMS 创建密钥环 + 加密密钥；
对敏感文件手动 / 批量加密再存到云存储；
通过 IAM 控密钥权限 + 审计日志，实现云上机密数据安全管控。



一、实验里的 加密流程 + 命令
1. 先准备明文文件（实验给的金融文档）
bash
运行
gsutil cp gs://${GOOGLE_CLOUD_PROJECT}-kms-lab-data/finance-dept/inbox/1.txt .
2. 把文件转成 Base64（KMS 要求）
bash
运行
PLAINTEXT=$(cat 1.txt | base64 -w0)
3. 调用 KMS 加密（核心命令）
bash
运行
curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \
  -d "{\"plaintext\":\"$PLAINTEXT\"}" \
  -H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \
  -H "Content-Type: application/json"
4. 把加密结果保存成文件
bash
运行
curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \
  -d "{\"plaintext\":\"$PLAINTEXT\"}" \
  -H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \
  -H "Content-Type:application/json" \
| jq .ciphertext -r > 1.encrypted
二、实验里的 解密流程 + 命令
直接用 KMS 解密，并还原成原文
bash
运行
curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:decrypt" \
  -d "{\"ciphertext\":\"$(cat 1.encrypted)\"}" \
  -H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \
  -H "Content-Type:application/json" \
| jq .plaintext -r | base64 -d
执行后你会看到：
plaintext
Content: This is a sample financial document for encryption (File 1).
三、实验加密 / 解密 最精简总结（背这个）
加密
读取文件 → base64 编码
调用 KMS :encrypt API
得到密文（ciphertext）
保存到文件
解密
读取密文文件
调用 KMS :decrypt API
base64 解码 → 得到原文
四、实验里 批量加密命令（也会考）
bash
运行
MYDIR=finance-dept
FILES=$(find $MYDIR -type f -not -name "*.encrypted")
for file in $FILES; do
  PLAINTEXT=$(cat $file | base64 -w0)
  curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \
    -d "{\"plaintext\":\"$PLAINTEXT\"}" \
    -H "Authorization:Bearer $(gcloud auth application-default print-access-token)" \
    -H "Content-Type:application/json" \
  | jq .ciphertext -r > $file.encrypted
done
gsutil -m cp finance-dept/inbox/*.encrypted gs://${BUCKET_NAME}/finance-dept/inbox


