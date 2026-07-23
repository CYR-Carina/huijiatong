# 惠家通 API 接口文档

## 一、接口概述

本文档描述惠家通 Agent 的 RESTful API 接口，用于与外部系统集成。

## 二、基础配置

### 2.1 服务地址
```
开发环境: http://localhost:8080
生产环境: https://api.huijiatong.com
```

### 2.2 认证方式
- API Key 认证
- 请求头: `Authorization: Bearer ${API_KEY}`

### 2.3 响应格式
```json
{
  "code": 200,
  "message": "success",
  "data": {...},
  "timestamp": 1699999999
}
```

## 三、接口列表

### 3.1 对话接口

#### POST /api/chat
发送消息给 Agent，获取回复

**请求体:**
```json
{
  "user_id": "string (用户唯一标识)",
  "message": "string (用户输入内容)",
  "mode": "string (可选: funding/elderly/auto)",
  "session_id": "string (可选: 会话ID)",
  "enable_speech": "boolean (可选: 是否启用语音, 默认true)"
}
```

**成功响应 (200):**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "session_id": "string",
    "response": "string (Agent回答内容)",
    "mode": "string (当前模式)",
    "speech_url": "string (语音文件URL, 可选)",
    "safety_warning": "string (安全提示, 可选)",
    "sources": ["string (知识源列表, 可选)"]
  }
}
```

**失败响应 (400):**
```json
{
  "code": 400,
  "message": "参数错误",
  "data": null
}
```

### 3.2 条件自测接口

#### POST /api/funding/self-assessment
资助条件自测

**请求体:**
```json
{
  "user_id": "string",
  "annual_income": "number (家庭年收入, 万元)",
  "student_count": "integer (在读学生人数)",
  "education_level": "string (最高学历: 初中/高中/大学/大专)",
  "special_difficulty": "boolean (是否有特殊困难)"
}
```

**成功响应 (200):**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "recommendations": [
      {
        "name": "生源地信用助学贷款",
        "priority": 1,
        "description": "最高16000元/年",
        "link": "string (详细链接)"
      },
      {
        "name": "国家助学金",
        "priority": 2,
        "description": "平均3300元/年",
        "link": "string"
      }
    ],
    "application_tips": "string (申请提示)"
  }
}
```

### 3.3 诈骗识别接口

#### POST /api/security/fraud-detect
分析信息是否为诈骗

**请求体:**
```json
{
  "user_id": "string",
  "description": "string (用户描述的电话/信息内容)"
}
```

**成功响应 (200):**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "result": "string (fraud/suspicious/normal)",
    "confidence": "number (置信度 0-1)",
    "features": [
      {
        "text": "要求转账",
        "is_fraud": true
      }
    ],
    "advice": "string (建议内容)",
    "emergency_contacts": [
      {
        "name": "110",
        "type": "police"
      }
    ]
  }
}
```

### 3.4 模式切换接口

#### POST /api/chat/switch-mode
手动切换对话模式

**请求体:**
```json
{
  "session_id": "string (会话ID)",
  "mode": "string (funding/elderly)"
}
```

**成功响应 (200):**
```json
{
  "code": 200,
  "message": "模式切换成功",
  "data": {
    "mode": "string (新模式)",
    "greeting": "string (模式问候语)"
  }
}
```

### 3.5 会话管理接口

#### GET /api/chat/sessions
获取用户会话列表

**请求参数:**
- `user_id`: string (必填)
- `limit`: integer (可选, 默认10)

**成功响应 (200):**
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "session_id": "string",
      "last_message": "string",
      "mode": "string",
      "created_at": "timestamp",
      "updated_at": "timestamp"
    }
  ]
}
```

#### DELETE /api/chat/sessions/{session_id}
删除指定会话

**成功响应 (200):**
```json
{
  "code": 200,
  "message": "会话删除成功",
  "data": null
}
```

### 3.6 知识库接口

#### GET /api/knowledge/categories
获取知识库分类

**成功响应 (200):**
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "id": "string",
      "name": "资助政策库",
      "document_count": 10,
      "description": "存储资助政策相关文档"
    },
    {
      "id": "string",
      "name": "银龄关怀库",
      "document_count": 10,
      "description": "存储防诈骗、健康、养老相关文档"
    }
  ]
}
```

## 四、错误码

| 错误码 | 含义 |
|--------|------|
| 200 | 成功 |
| 400 | 参数错误 |
| 401 | 未授权 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 500 | 服务器错误 |

## 五、使用示例

### 5.1 cURL 示例

```bash
# 发送消息
curl -X POST http://localhost:8080/api/chat \
  -H "Authorization: Bearer your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user001",
    "message": "我家孩子读高三，能申请什么资助？",
    "mode": "auto"
  }'

# 条件自测
curl -X POST http://localhost:8080/api/funding/self-assessment \
  -H "Authorization: Bearer your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user001",
    "annual_income": 3,
    "student_count": 2,
    "education_level": "高中",
    "special_difficulty": false
  }'

# 诈骗识别
curl -X POST http://localhost:8080/api/security/fraud-detect \
  -H "Authorization: Bearer your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user001",
    "description": "有人打电话说我中奖了，让我交500元手续费才能领奖"
  }'
```

## 六、Webhook 接口

### 6.1 会话结束通知

**触发条件:** 会话超时或用户主动结束

**请求地址:** 配置的 webhook_url

**请求体:**
```json
{
  "event_type": "session_ended",
  "session_id": "string",
  "user_id": "string",
  "duration": "integer (会话时长秒)",
  "message_count": "integer (消息数)",
  "mode": "string (最后模式)",
  "summary": "string (会话摘要)"
}
```

## 七、更新日志

### v1.0.0 (2026-07-22)
- 初始版本
- 支持对话接口、条件自测、诈骗识别
- 支持双模式切换
- 支持知识库查询