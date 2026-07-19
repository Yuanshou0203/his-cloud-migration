# HIS 业务系统云上迁移项目

## 📋 项目概述
模拟医院信息系统(HIS)从本地数据中心迁移至阿里云的完整流程，涵盖 MySQL 全量+增量数据迁移、数据库分级权限管控、Nginx WAF 边界安全防护、多节点高可用架构，围绕医疗数据安全合规要求建立完整安全运维体系。

## 🏗️ 迁移架构

```
  本地数据中心 (.20)                       阿里云环境 (.40)
  ┌──────────────────┐                  ┌──────────────────┐
  │  HIS 业务系统     │    全量+增量     │  HIS 业务系统     │
  │                  │  ─────────────►  │                  │
  │  MySQL 5.5       │    mysqldump     │  MySQL 5.5       │
  │  ├ patients(21)  │    + binlog      │  ├ patients(21)  │
  │  ├ drugs(15)     │                  │  ├ drugs(15)     │
  │  ├ prescriptions │                  │  ├ prescriptions │
  │  └ items         │                  │  └ items         │
  │                  │                  │                  │
  │  Apache (源站)   │                  │  Nginx WAF       │
  │  端口 80/8080    │                  │  SQL注入拦截     │
  │                  │                  │  XSS 拦截        │
  │  分级账号:       │                  │  文件上传控制    │
  │  DBA/只读/应用   │                  │  速率限制        │
  └──────────────────┘                  └──────────────────┘
```

## 🛠️ 技术栈
| 类别 | 技术 |
|------|------|
| 数据库 | MariaDB 5.5, MySQL binlog 增量同步 |
| Web服务器 | Nginx 1.20, Apache 2.4 (源站) |
| 安全防护 | Nginx WAF (SQL注入/XSS/目录遍历) |
| 高可用 | Nginx upstream 负载均衡 + 故障剔除 |
| 备份 | mysqldump 全量 + binlog 增量, 定时 crontab |
| 系统 | CentOS 7.9, VMware 虚拟化 |

## 📁 目录结构

```
├── mysql/
│   └── his_migration.sh          # 数据迁移完整脚本
├── nginx/
│   └── his_waf.conf              # Nginx WAF 防护配置
├── screenshots/                  # 截图目录
└── README.md
```

## ✨ 项目亮点

### 1. MySQL 全量+增量迁移
```bash
# 全量：一致性快照导出
mysqldump --single-transaction --master-data=2 his_hospital > full_backup.sql

# 增量：binlog 实时同步
mysqlbinlog --start-position=<pos> mysql-bin.000001 > incremental.sql
```

### 2. 分级权限管控
| 账号 | 权限 | 用途 |
|------|------|------|
| his_dba | ALL | 数据库管理员 |
| his_readonly | SELECT | 报表/查询 |
| his_app | CRUD | 应用程序 |
| his_audit | SELECT(患者/处方) | 合规审计 |

### 3. Nginx WAF 边界防护
- **SQL 注入拦截**：`union select`, `drop table`, `--` 等模式
- **XSS 拦截**：`<script>`, `javascript:`, `onerror=` 等
- **文件上传限制**：阻断 `.php`, `.jsp`, `.asp` 等可执行文件
- **CC 攻击防护**：`limit_req` 速率限制（30r/s）
- **目录遍历防护**：`../` 路径拦截

### 4. 安全复测验证
```bash
# SQL注入测试 → 预期 403
curl 'http://192.168.133.40/?id=1 UNION SELECT * FROM patients'

# XSS测试 → 预期 403
curl 'http://192.168.133.40/?q=<script>alert(1)</script>'

# 正常请求 → 预期 200
curl 'http://192.168.133.40/'
```

## 🚀 快速开始

```bash
# 在 docker-master (.10) 上执行完整迁移
bash mysql/his_migration.sh
```

自动完成：测试数据生成 → 分级账号 → 全量导出 → 增量同步 → WAF 部署 → 安全复测

## 📊 迁移数据规模
| 表 | 行数 |
|----|------|
| patients (患者) | 21 |
| drugs (药品) | 15 |
| prescriptions (处方) | 15 |
| prescription_items | 30+ |

## 📝 相关项目
- [阿里云容器集群管控](../aliyun-container-security) - 本项目的基础设施层

---

> 🎓 应届生实训项目 | 独立完成 | 2025.09-2025.10
