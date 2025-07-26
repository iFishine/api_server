# 🧪 Vitest 测试脚本集合

这个文件夹包含了用于加班时长统计系统的各种测试脚本。所有脚本都使用 Vitest 测试框架。

## 📁 脚本列表

| 脚本名称 | 功能描述 | 使用场景 |
|---------|---------|---------|
| `test-menu.sh` | 📋 **主菜单脚本** - 交互式选择测试操作 | 🌟 推荐首次使用 |
| `quick-test-overtime.sh` | 🚀 快速运行所有测试并显示覆盖率 | 日常开发验证 |
| `test-watch.sh` | 👀 观察模式 - 文件变化时自动重新运行测试 | 开发过程中持续测试 |
| `test-ui.sh` | 🎨 启动可视化测试界面 | 查看详细测试结果 |
| `test-coverage-detailed.sh` | 📊 生成详细的HTML覆盖率报告 | 代码质量分析 |
| `test-performance.sh` | ⚡ 性能基准测试 - 测量执行时间 | 性能优化分析 |
| `test-cleanup.sh` | 🧹 清理测试缓存和临时文件 | 环境维护 |

## 🚀 快速开始

### 方法一：使用交互式菜单（推荐）
```bash
npm run test:menu
```

### 方法二：直接运行特定测试
```bash
# 快速测试（最常用）
npm run test:overtime

# 观察模式
npm run test:watch

# 可视化界面
npm run test:ui

# 详细覆盖率
npm run test:coverage

# 性能测试
npm run test:performance

# 清理环境
npm run test:cleanup
```

## 📋 详细说明

### 🌟 主菜单脚本 (`test-menu.sh`)
- 提供交互式菜单界面
- 包含所有测试功能的入口
- 带有帮助信息和使用指导
- 适合新用户和偶尔使用者

### 🚀 快速测试脚本 (`quick-test-overtime.sh`)
- 运行所有22个测试用例
- 显示基本覆盖率信息
- 彩色输出，易于阅读
- 执行时间通常 < 1秒

**输出示例：**
```
🚀 开始运行加班时长统计测试...
==================================
✅ 所有测试通过！
📊 覆盖率达到90.2%
🎉 测试完成！
```

### 👀 观察模式脚本 (`test-watch.sh`)
- 监听以下文件变化：
  - `server/services/overtimeService.ts`
  - `server/services/*.test.ts`
  - `server/services/*.integration.test.ts`
- 文件变化时自动重新运行测试
- 适合开发过程中的持续测试

### 🎨 可视化界面脚本 (`test-ui.sh`)
- 启动Web界面：`http://localhost:51204/__vitest__/`
- 功能包括：
  - 实时测试结果查看
  - 交互式测试运行
  - 代码覆盖率可视化
  - 测试文件浏览

### 📊 详细覆盖率脚本 (`test-coverage-detailed.sh`)
- 生成多种格式的覆盖率报告：
  - 终端文本输出
  - JSON格式：`coverage/coverage-final.json`
  - HTML可视化：`coverage/index.html`
- 自动显示HTML报告的访问路径

### ⚡ 性能测试脚本 (`test-performance.sh`)
- 测量测试执行时间
- 计算平均每个测试用例的时间
- 提供性能评级：
  - 🚀 优秀：< 1秒
  - ⚡ 良好：< 3秒
  - 🐌 需要优化：> 3秒
- 包含性能优化建议

### 🧹 清理脚本 (`test-cleanup.sh`)
- 删除测试相关的临时文件：
  - `coverage/` 覆盖率报告目录
  - `node_modules/.vitest/` Vitest缓存
  - `.nyc_output/` NYC输出
  - `test.log` 测试日志
- 适合在测试环境出现问题时使用

## 🔧 自定义配置

所有脚本都可以根据需要进行自定义：

1. **修改颜色主题**：编辑脚本顶部的颜色定义
2. **调整性能阈值**：修改 `test-performance.sh` 中的时间判断条件
3. **添加新的清理项**：在 `test-cleanup.sh` 中添加更多清理逻辑

## 🐛 故障排除

### 常见问题

1. **脚本没有执行权限**
   ```bash
   chmod +x scripts/testing/*.sh
   ```

2. **找不到npm命令**
   - 确保在项目根目录运行
   - 检查Node.js和npm是否正确安装

3. **测试失败**
   - 运行 `npm run test:cleanup` 清理环境
   - 检查依赖是否完整安装：`npm install`

4. **端口占用（UI模式）**
   - 检查端口51204是否被占用
   - 杀死相关进程或使用其他端口

### 调试模式

在脚本中添加 `set -x` 可以启用调试模式，显示每个命令的执行过程：

```bash
#!/bin/bash
set -x  # 启用调试模式
# ... 脚本内容
```

## 🚀 集成到CI/CD

这些脚本也可以集成到持续集成流程中：

```yaml
# GitHub Actions 示例
- name: Run Quick Tests
  run: ./scripts/testing/quick-test-overtime.sh

- name: Generate Coverage Report
  run: ./scripts/testing/test-coverage-detailed.sh

- name: Performance Benchmark
  run: ./scripts/testing/test-performance.sh
```

## 📊 测试统计

当前测试覆盖情况：
- **测试用例总数**: 22个
- **单元测试**: 19个
- **集成测试**: 3个
- **代码覆盖率**: 90.2%
- **平均执行时间**: < 300ms

---

💡 **提示**: 建议新用户从 `npm run test:menu` 开始，熟悉各种测试功能后再直接使用具体的脚本命令。
