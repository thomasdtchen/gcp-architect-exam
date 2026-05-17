
-- Datastream: PostgreSQL Replication to BigQuery
-- PostgreSQL（Cloud SQL） → Datastream → BigQuery
-- 本质是：把操作型数据库（OLTP）里的数据，近乎实时地同步到数据仓库（OLAP）里，做后续分析。
-- PostgreSQL：业务系统的 “生产库”，用户下单、交易、读写都在这里，要求低延迟、高并发。
-- Datastream：GCP 的 CDC（Change Data Capture，变更数据捕获）工具，实时捕捉 PostgreSQL 的增 / 删 / 改操作。
-- BigQuery：数据仓库，用来跑大规模查询、报表、机器学习、BI 分析，不适合直接对业务库做。

gcloud sql connect postgres-db --user=postgres

CREATE SCHEMA IF NOT EXISTS test;

CREATE TABLE IF NOT EXISTS test.example_table (
id  SERIAL PRIMARY KEY,
text_col VARCHAR(50),
int_col INT,
date_col TIMESTAMP
);

ALTER TABLE test.example_table REPLICA IDENTITY DEFAULT; 

INSERT INTO test.example_table (text_col, int_col, date_col) VALUES
('hello', 0, '2020-01-01 00:00:00'),
('goodbye', 1, NULL),
('name', -987, NOW()),
('other', 2786, '2021-01-01 00:00:00');

CREATE PUBLICATION test_publication FOR ALL TABLES;
ALTER USER POSTGRES WITH REPLICATION;
SELECT PG_CREATE_LOGICAL_REPLICATION_SLOT('test_replication', 'pgoutput');




