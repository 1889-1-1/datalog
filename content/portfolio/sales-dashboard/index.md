---
title: "销售数据自动化看板"
date: 2025-12-01
summary: "搭建从数据抽取到可视化呈现的全自动Pipeline，每日自动更新KPI仪表盘"
key_insight: "自动化Pipeline将日报制作时间从2小时压缩到5分钟，覆盖GMV、客单价、复购率等12个核心指标，实现数据驱动决策。"
tags: ["Python", "Airflow", "SQL", "Power BI", "自动化"]
methods: "Airflow DAG调度 → SQL数据抽取 → Python清洗 → Power BI可视化 → 企业微信推送"
findings: "自动化后报表产出效率提升24倍，异常检测从T+2提前到T+0实时告警。"
code_link: "https://github.com/yourusername/sales-dashboard"
chart:
  description: "自动化Pipeline架构图"
metrics:
  - label: "监控指标数"
    value: "12个"
  - label: "效率提升"
    value: "24x"
  - label: "异常响应"
    value: "T+0"
  - label: "日报耗时"
    value: "5分钟"
---

## 项目背景

团队每天需要手动从多个数据源拉取数据、Excel 加工、制作 Power BI 报表，重复性工作耗时约 2 小时/天。本项目将全流程自动化。

## 技术架构

### 数据层
- **数据源**：MySQL（订单）、PostgreSQL（用户）、API（广告投放）
- **抽取策略**：增量抽取 + 全量兜底

### 调度层
- **Apache Airflow**：每日 7:00 触发 DAG
- **依赖管理**：数据抽取 → 清洗 → 指标计算 → 看板刷新 → 告警推送

### 展示层
- **Power BI**：通过 API 自动刷新数据集
- **企业微信机器人**：推送核心指标摘要 + 异常告警

## 核心功能

1. 自动生成日报/周报/月报
2. 异常检测（环比/同比波动超过阈值自动告警）
3. 数据质量监控（缺失值、重复订单检测）
4. 邮件+企微双通道推送
