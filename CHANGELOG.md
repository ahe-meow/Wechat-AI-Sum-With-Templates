# 更新日志

## v1.2.5 (2026-07-08)

### 增强日志
- **输出完整请求内容**：记录发送给 API 的完整请求体和请求头
- **便于调试**：可以查看是否成功发送了禁用推理的参数

### 日志内容

**总结请求日志**：
- 请求地址
- 请求模型
- API Key（脱敏）
- 完整请求体 JSON（包含所有参数）
- 请求头

**测试 API 日志**：
- 完整请求体 JSON
- 完整响应 JSON

通过这些日志，可以确认：
1. `reasoning: false` 等参数是否真的发送了
2. API 是否接受了这些参数
3. 模型为什么还在思考

## v1.2.4 (2026-07-08)

### 优化功能
- **强制禁用推理模式**：多重措施防止模型思考导致 token 超限
  - 在提示词中明确要求"直接输出，不要思考"
  - 添加多个 API 参数：`reasoning: false`、`enable_reasoning: false`、`stream_reasoning: false`
  - 增加 `max_tokens` 从 1500 到 4000，即使有思考也能完成输出

### 问题说明

**症状**：模型返回的是思考过程（`reasoning_content`），而不是最终总结

**根本原因**：
- mimo-v2.5 等模型默认启用推理模式
- 模型先进行长篇思考（可能消耗 1000+ tokens）
- 达到 `max_tokens` 限制后被截断
- 只输出了 `reasoning_content`（思考），`content`（结果）为空或不完整

**解决方案**：
1. 系统提示词追加强制要求："直接输出最终总结，不要输出任何思考过程"
2. 请求参数添加三个禁用推理的字段（兼容不同 API 实现）
3. 大幅提升 token 限制，确保即使有思考也能完成输出

## v1.2.3 (2026-07-08)

### 修复问题
- **修复 JSON 解析失败**：支持 `reasoning_content` 字段（适用于 mimo-v2.5 等模型）
- **增强日志功能**：输出完整的 API JSON 响应，便于排查问题

## v1.2.2 (2026-07-08)

### 新增功能
- **自动补全 API 地址**：用户设置 API 地址时，系统会自动补全 `/v1/chat/completions` 路径

### 功能说明

现在用户只需要输入 API 的基础地址，系统会自动补全完整路径：

#### 补全规则

1. **已有完整路径** - 直接使用，不做修改
   - `https://api.openai.com/v1/chat/completions` → 不变
   - `https://api.anthropic.com/v1/messages` → 不变

2. **以 /v1 结尾** - 自动添加 `/chat/completions`
   - `https://api.openai.com/v1` → `https://api.openai.com/v1/chat/completions`
   - `https://api.example.com/v1` → `https://api.example.com/v1/chat/completions`

3. **包含 /v1/ 但路径错误** - 自动修正为 `/v1/chat/completions`
   - `https://api.openai.com/v1/models` → `https://api.openai.com/v1/chat/completions`
   - `https://api.example.com/v1/other` → `https://api.example.com/v1/chat/completions`

4. **没有 /v1** - 自动添加 `/v1/chat/completions`
   - `https://api.openai.com` → `https://api.openai.com/v1/chat/completions`
   - `https://api.example.com` → `https://api.example.com/v1/chat/completions`

5. **自动清理** - 移除尾部斜杠和查询参数
   - `https://api.openai.com/v1/` → `https://api.openai.com/v1/chat/completions`
   - `https://api.openai.com/v1?key=xxx` → `https://api.openai.com/v1/chat/completions`

#### 使用场景

这个功能在以下场景生效：
- ✅ 保存配置时
- ✅ 测试 API 可用性时
- ✅ 获取可用模型列表时
- ✅ 调用 AI 总结接口时

#### 用户体验改进

用户现在可以更简单地配置 API：
- 之前：必须输入完整的 `https://api.xiaomimimo.com/v1/chat/completions`
- 现在：只需输入 `https://api.xiaomimimo.com` 即可

系统会在测试和获取模型时显示规范化后的地址，让用户了解实际使用的完整 URL。

### 技术细节

新增函数：
- `String normalizeApiUrl(String apiUrl)` - API 地址规范化函数

修改函数：
- `String getApiUrl()` - 返回时自动规范化
- `void testApiAvailability(...)` - 测试前规范化地址
- `void fetchAvailableModels(...)` - 获取模型前规范化地址
- 配置保存逻辑 - 保存时规范化地址
