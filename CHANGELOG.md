# 更新日志

## v1.2.3 (2026-07-08)

### 修复问题
- **修复 JSON 解析失败**：支持 `reasoning_content` 字段（适用于 mimo-v2.5 等模型）
- **增强日志功能**：输出完整的 API JSON 响应，便于排查问题

### 技术细节

**问题原因**：
某些 AI 模型（如 mimo-v2.5）返回的 JSON 中，实际内容不在 `content` 字段，而在 `reasoning_content` 字段。之前的代码只检查 `content` 字段，导致解析失败。

**解决方案**：
1. 修改 `parseSummaryResponse()` 函数，当 `content` 为空时，尝试读取 `reasoning_content`
2. 增强日志输出，完整记录 API 返回的 JSON（不再截断为 200 字符）
3. 添加内容提取成功的日志，显示提取到的字符数

**日志改进**：
- 测试 API 时输出完整 JSON 响应
- 调用总结接口时输出完整 JSON 响应
- 成功提取内容时显示字符数统计

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
