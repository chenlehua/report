# TARA报告系统 - 问题修复说明

## 修复的问题

### 1. 图片与JSON关联问题

**问题描述**: 上传了图片，但生成的Excel中图片未渲染

**修复方案**:
- 新增 `/api/upload/batch` 端点，支持同时上传JSON文件和图片文件
- 图片上传后自动保存到 `uploads/images/` 目录
- 图片路径自动关联到JSON数据的对应字段（definitions.item_boundary_image 等）
- Excel生成器使用绝对路径读取图片并嵌入Excel

**图片类型映射**:
| 前端字段 | JSON路径 | 用途 |
|---------|---------|------|
| item_boundary_image | definitions.item_boundary_image | 项目边界图 |
| system_architecture_image | definitions.system_architecture_image | 系统架构图 |
| software_architecture_image | definitions.software_architecture_image | 软件架构图 |
| dataflow_image | assets.dataflow_image | 数据流图 |

### 2. 前端预览缺少攻击树展示

**问题描述**: 报告预览中没有攻击树的展示，且相关定义中没有显示对应的图片

**修复方案**:
- 更新 `ReportPreview.vue` 组件，添加攻击树展示区域
- 添加图片预览组件，支持放大查看
- 修复图片URL生成逻辑（支持API路径和文件路径两种格式）
- 添加功能描述、假设条目、术语表等展示

**新增展示内容**:
- 架构图预览（项目边界图、系统架构图、软件架构图、数据流图）
- 功能描述
- 相关项假设列表
- 术语表
- 攻击树列表（带图片）

### 3. TARA分析结果显示不完整

**问题描述**: 前端页面TARA分析结果显示的信息不完整

**修复方案**:
更新TARA分析结果表格，现在显示以下字段：
- 资产ID、资产名称
- STRIDE模型（带颜色标签）
- 威胁ID、威胁场景、损害场景
- 攻击路径
- 攻击可行性（AV、AC、PR、UI）
- 影响等级（安全、财务、运营、隐私）
- 安全措施
- 有效性、安全目标
- 残余风险（带颜色标签）

## 文件修改清单

### 后端 (backend/tara_api/)

1. **main.py**
   - 新增 `POST /api/upload/batch` 端点
   - 新增 `GET /api/reports/{report_id}/preview` 端点
   - 图片保存和路径关联逻辑

2. **models.py**
   - 更新 `AttackTree` 模型，添加 asset_id, asset_name, image_url 字段

### 前端 (frontend/src/)

1. **components/ReportPreview.vue**
   - 完全重写，添加完整的预览功能
   - 攻击树展示
   - 完整的TARA分析结果表格
   - 图片预览支持

2. **api/index.ts**
   - 更新 `uploadAndGenerate` 返回类型
   - 更新 `getReports` 适配后端响应格式

3. **stores/report.ts**
   - 更新 `generateReport` 方法签名和错误处理

4. **types/index.ts**
   - 更新 `PreviewData` 类型定义

## 使用说明

### 启动后端
```bash
cd backend
pip install -e .
python -m uvicorn tara_api.main:app --host 0.0.0.0 --port 8000
```

### 启动前端
```bash
cd frontend
npm install
npm run dev
```

### API端点说明

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/upload/batch` | POST | 批量上传JSON和图片 |
| `/api/reports` | GET | 获取报告列表 |
| `/api/reports/{id}/preview` | GET | 获取预览数据 |
| `/api/reports/{id}/download` | GET | 下载报告 |
| `/api/images/{id}` | GET | 获取图片 |

### 图片上传字段

批量上传时使用的 FormData 字段：
- `json_file`: JSON数据文件（必需）
- `item_boundary_image`: 项目边界图（可选）
- `system_architecture_image`: 系统架构图（可选）
- `software_architecture_image`: 软件架构图（可选）
- `dataflow_image`: 数据流图（可选）
