<template>
  <div class="overtime-calculator">
    <div class="tool-layout">
      <!-- 左侧输入区域 -->
      <div class="input-section">
        <div class="section-header">
          <h3>输入加班数据</h3>
          <div class="actions">
            <button @click="clearInput" class="btn btn-secondary">
              <i class="fas fa-trash"></i> 清空
            </button>
            <button @click="resetToExample" class="btn btn-secondary">
              <i class="fas fa-file-import"></i> 示例
            </button>
            <button @click="calculateOvertime" class="btn btn-primary">
              <i class="fas fa-calculator"></i> 计算
            </button>
          </div>
        </div>
        <div class="config-section">
          <h4>计算配置</h4>
          <div class="config-row">
            <div class="config-item">
              <label for="hourlyRate">时薪基数 (元)</label>
              <input type="number" id="hourlyRate" v-model="hourlyRate" min="1" step="1" class="config-input">
            </div>
            <div class="config-item">
              <label for="overtimeStart">加班开始时间</label>
              <select id="overtimeStart" v-model="overtimeStartTime" class="config-select">
                <option value="18:30">18:30</option>
                <option value="19:00">19:00</option>
                <option value="19:30">19:30</option>
              </select>
            </div>
          </div>
        </div>
        <div class="json-input-area">
          <label>加班数据 (JSON格式)</label>
          <div class="json-info">
            <i class="fas fa-info-circle"></i>
            <span>请输入JSON格式的加班数据，数据可从<a
                href="https://hr.quectel.com/portal/index" target="_blank">https://hr.quectel.com/portal/index</a>中获取，该页面只负责解析处理，不会保留用户数据
            </span>
          </div>
          <textarea v-model="jsonInput" class="json-input" @input="validateJson"></textarea>
          <div v-if="validationError" class="error-message">
            <i class="fas fa-exclamation-triangle"></i>
            {{ validationError }}
          </div>
        </div>
      </div>

      <!-- 右侧输出区域 -->
      <div class="output-section">
        <div class="section-header">
          <h3>加班时间统计</h3>
          <div class="actions">
            <button @click="copyResults" class="btn btn-secondary">
              <i class="fas fa-copy"></i> 复制
            </button>
            <button @click="downloadResults" class="btn btn-secondary">
              <i class="fas fa-download"></i> 下载
            </button>
          </div>
        </div>
        <!-- 计算状态提示 -->
        <div v-if="isLoading" class="status-indicator loading">
          <i class="fas fa-spinner fa-spin"></i>
          <span>计算中...</span>
        </div>
        <div v-else-if="error" class="status-indicator error">
          <i class="fas fa-exclamation-circle"></i>
          <span>{{ error }}</span>
        </div>
        <div v-else-if="!results" class="status-indicator empty">
          <i class="fas fa-info-circle"></i>
          <span>请输入加班数据并点击"计算"按钮</span>
        </div>

        <!-- 详细统计摘要 -->
        <div v-if="results" class="summary-section">
          <div class="summary-header-section">
            <h4>{{ results.header?.title || '加班统计详情' }}</h4>
            <div v-if="results.header?.hasUnderwork" class="warning-indicator">
              <i class="fas fa-exclamation-triangle"></i>
              <span>检测到矿工时间</span>
            </div>
          </div>

          <!-- 核心数据概览 -->
          <div class="core-overview">
            <div class="overview-card primary">
              <div class="card-icon">
                <i class="fas fa-coins"></i>
              </div>
              <div class="card-content">
                <div class="card-label">加班收入</div>
                <div class="card-value">{{ results.income?.total?.formatted || '0元' }}</div>
                <div class="card-note" v-if="results.income?.total?.note">{{ results.income.total.note }}</div>
              </div>
            </div>
            <div class="overview-card">
              <div class="card-icon">
                <i class="fas fa-utensils"></i>
              </div>
              <div class="card-content">
                <div class="card-label">餐费补贴</div>
                <div class="card-value">{{ results.income?.mealAllowance?.formatted || '0元' }}</div>
                <div class="card-note" v-if="results.income?.mealAllowance?.note">{{ results.income.mealAllowance.note }}</div>
              </div>
            </div>
            <div class="overview-card">
              <div class="card-icon">
                <i class="fas fa-clock"></i>
              </div>
              <div class="card-content">
                <div class="card-label">总加班时长</div>
                <div class="card-value">{{ results.hours?.total?.formatted || '0小时' }}</div>
                <div class="card-note" v-if="results.hours?.total?.note">{{ results.hours.total.note }}</div>
              </div>
            </div>
            <div class="overview-card">
              <div class="card-icon">
                <i class="fas fa-trophy"></i>
              </div>
              <div class="card-content">
                <div class="card-label">加班等级</div>
                <div class="card-value rank-display" :class="'rank-' + (results.rank?.level || 'unknown')">
                  {{ results.rank?.level || 'Unknown' }}
                </div>
                <div class="card-note">{{ results.rank?.message || '未知评级' }}</div>
              </div>
            </div>
          </div>

          <!-- 详细数据分类 -->
          <div class="detail-sections">
            <!-- 收入明细 -->
            <div class="detail-section">
              <h5><i class="fas fa-money-bill-wave"></i> 收入明细</h5>
              <div class="detail-grid">
                <div class="detail-item">
                  <span class="detail-label">工作日加班:</span>
                  <span class="detail-value">{{ results.income?.breakdown?.workday?.formatted || '0元' }}</span>
                  <span class="detail-extra" v-if="results.income?.breakdown?.workday?.note">({{ results.income.breakdown.workday.note }})</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">周末加班:</span>
                  <span class="detail-value">{{ results.income?.breakdown?.weekend?.formatted || '0元' }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">节假日加班:</span>
                  <span class="detail-value">{{ results.income?.breakdown?.holiday?.formatted || '0元' }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">加班收入:</span>
                  <span class="detail-value">{{ results.income?.total?.formatted || '0元' }}</span>
                  <span class="detail-extra" v-if="results.income?.total?.note">({{ results.income.total.note }})</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">餐费补贴:</span>
                  <span class="detail-value">{{ results.income?.mealAllowance?.formatted || '0元' }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">总加班收入:</span>
                  <span class="detail-value">{{ results.income?.totalIncome?.formatted || '0元' }}</span>
                  <span class="detail-extra">(含餐费补贴)</span>
                </div>
                <div class="detail-item total" v-if="results.income?.actualTotalIncome">
                  <span class="detail-label">扣减后收入:</span>
                  <span class="detail-value highlight">{{ results.income.actualTotalIncome.formatted }}</span>
                  <span class="detail-extra" v-if="results.income.actualTotalIncome.note">({{ results.income.actualTotalIncome.note }})</span>
                </div>
              </div>
            </div>

            <!-- 工时统计 -->
            <div class="detail-section">
              <h5><i class="fas fa-stopwatch"></i> 工时统计</h5>
              <div class="detail-grid">
                <div class="detail-item">
                  <span class="detail-label">工作日加班:</span>
                  <span class="detail-value">{{ results.hours?.workday?.formatted || '0小时' }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">周末加班:</span>
                  <span class="detail-value">{{ results.hours?.weekend?.formatted || '0小时' }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">节假日加班:</span>
                  <span class="detail-value">{{ results.hours?.holiday?.formatted || '0小时' }}</span>
                </div>
                <div class="detail-item total" v-if="results.hours?.actualTotal">
                  <span class="detail-label">扣减后时长:</span>
                  <span class="detail-value highlight">{{ results.hours.actualTotal.formatted }}</span>
                  <span class="detail-extra" v-if="results.hours.actualTotal.note">({{ results.hours.actualTotal.note }})</span>
                </div>
              </div>
            </div>

            <!-- 考勤情况 -->
            <div class="detail-section" v-if="results.attendance">
              <h5><i class="fas fa-calendar-check"></i> 考勤情况</h5>
              <div class="detail-grid">
                <div class="detail-item" v-if="results.attendance.workdays">
                  <span class="detail-label">出勤天数:</span>
                  <span class="detail-value">{{ results.attendance.workdays.ratio || '0/0' }}天</span>
                  <span class="detail-extra" v-if="results.attendance.workdays.note">({{ results.attendance.workdays.note }})</span>
                </div>
                <div class="detail-item" v-if="results.attendance.late">
                  <span class="detail-label">迟到情况:</span>
                  <span class="detail-value">{{ results.attendance.late.message || '无迟到记录' }}</span>
                </div>
                <div class="detail-item" v-if="results.attendance.early">
                  <span class="detail-label">早退情况:</span>
                  <span class="detail-value">{{ results.attendance.early.message || '无早退记录' }}</span>
                </div>
              </div>
            </div>

            <!-- 智能建议 -->
            <div class="detail-section" v-if="results.tips && results.tips.length > 0">
              <h5><i class="fas fa-lightbulb"></i> 智能建议</h5>
              <div class="tips-container">
                <div v-for="(tip, index) in results.tips" :key="index" class="tip-item">
                  <i class="fas fa-arrow-right"></i>
                  <span>{{ tip }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, computed } from 'vue';
import api from '@/utils/api';

// 表单数据
const jsonInput = ref(getDefaultJsonData());
const hourlyRate = ref(20); // 默认时薪20元
const overtimeStartTime = ref('19:00'); // 默认19:00开始计算加班

// 结果数据
const results = ref<any>(null);
const isLoading = ref(false);
const error = ref('');
const validationError = ref('');

// 获取示例JSON数据
function getDefaultJsonData(): string {
  const now = new Date();
  const year = now.getFullYear();
  const month = String(now.getMonth() + 1).padStart(2, '0');
  return `
  [
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_1",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_1",
      "ID": "10000001",
      "CARDTIME": "2025-02-17 09:00:00",
      "SHIFTTERM": "2025-02-17",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_2",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_2",
      "ID": "10000002",
      "CARDTIME": "2025-02-17 18:00:00",
      "SHIFTTERM": "2025-02-17",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_3",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_3",
      "ID": "10000003",
      "CARDTIME": "2025-02-18 09:00:00",
      "SHIFTTERM": "2025-02-18",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_4",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_4",
      "ID": "10000004",
      "CARDTIME": "2025-02-18 20:00:00",
      "SHIFTTERM": "2025-02-18",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_5",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_5",
      "ID": "10000005",
      "CARDTIME": "2025-02-19 09:40:00",
      "SHIFTTERM": "2025-02-19",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_6",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_6",
      "ID": "10000006",
      "CARDTIME": "2025-02-19 20:00:00",
      "SHIFTTERM": "2025-02-19",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_7",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_7",
      "ID": "10000007",
      "CARDTIME": "2025-02-20 10:15:00",
      "SHIFTTERM": "2025-02-20",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_8",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_8",
      "ID": "10000008",
      "CARDTIME": "2025-02-20 21:00:00",
      "SHIFTTERM": "2025-02-20",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_9",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_9",
      "ID": "10000009",
      "CARDTIME": "2025-02-21 11:00:00",
      "SHIFTTERM": "2025-02-21",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_10",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_10",
      "ID": "10000010",
      "CARDTIME": "2025-02-21 19:30:00",
      "SHIFTTERM": "2025-02-21",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_11",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_11",
      "ID": "10000011",
      "CARDTIME": "2025-02-22 09:00:00",
      "SHIFTTERM": "2025-02-22",
      "K_LOCKED": ""
    },
    {
      "K_EXTRAS": "EXAMPLE_EXTRAS_12",
      "EMPID": "10001",
      "K_PKEYS": "EXAMPLE_PKEYS_12",
      "ID": "10000012",
      "CARDTIME": "2025-02-22 21:00:00",
      "SHIFTTERM": "2025-02-22",
      "K_LOCKED": ""
    }
  ]
  `
}

// 验证JSON输入
function validateJson(): void {
  if (!jsonInput.value.trim()) {
    validationError.value = '';
    return;
  }

  try {
    JSON.parse(jsonInput.value);
    validationError.value = '';
  } catch (err: any) {
    validationError.value = `JSON格式错误: ${err.message}`;
  }
}

// 清空输入
function clearInput(): void {
  jsonInput.value = '';
  validationError.value = '';
  results.value = null;
  error.value = '';
}

// 重置为示例数据
function resetToExample(): void {
  jsonInput.value = getDefaultJsonData();
  validationError.value = '';
  results.value = null;
  error.value = '';
}

// 计算加班时间
async function calculateOvertime(): Promise<void> {
  // 验证输入
  if (!jsonInput.value.trim()) {
    error.value = '请输入加班数据';
    return;
  }

  validateJson();
  if (validationError.value) {
    error.value = validationError.value;
    return;
  }

  // 清空错误和结果
  error.value = '';
  results.value = null;
  isLoading.value = true;

  try {
    // 直接使用JSON输入作为请求数据
    const requestData = JSON.parse(jsonInput.value);

    // 构造请求URL，包含查询参数
    const queryParams = new URLSearchParams({
      hourlyRate: hourlyRate.value.toString(),
      overtimeStartTime: overtimeStartTime.value
    });
    
    // 发送请求到API
    const response = await api.post(`/api/http/overtime/calculate?${queryParams}`, requestData);

    if (response.status === 200 && response.data && response.data.success) {
      results.value = response.data.data;
    } else {
      error.value = response.data?.error || '计算失败: 服务器未返回有效数据';
    }
  } catch (err: any) {
    error.value = `计算失败: ${err.response?.data?.error || err.message || '未知错误'}`;
  } finally {
    isLoading.value = false;
  }
}

// 复制结果
function copyResults(): void {
  if (!results.value) return;

  // 格式化JSON数据为可读的文本
  const formattedResult = formatResultForCopy(results.value);

  navigator.clipboard.writeText(formattedResult)
    .then(() => {
      // 可以添加一个复制成功的提示
      const originalText = '复制';
      const button = document.querySelector('.output-section .actions button:first-child');
      if (button) {
        const icon = button.querySelector('i');
        if (icon) icon.className = 'fas fa-check';
        button.textContent = ' 已复制';

        setTimeout(() => {
          if (icon) {
            icon.className = 'fas fa-copy';
            button.textContent = '';
            button.appendChild(icon);
            button.appendChild(document.createTextNode(' 复制'));
          }
        }, 2000);
      }
    })
    .catch(err => {
      console.error('复制失败:', err);
    });
}

// 下载结果
function downloadResults(): void {
  if (!results.value) return;

  const formattedResult = formatResultForCopy(results.value);
  const blob = new Blob([formattedResult], { type: 'text/plain;charset=utf-8' });
  const link = document.createElement('a');
  const currentDate = new Date().toISOString().split('T')[0];

  link.href = URL.createObjectURL(blob);
  link.download = `加班统计_${currentDate}.txt`;
  link.click();

  URL.revokeObjectURL(link.href);
}

// 格式化结果用于复制和下载
function formatResultForCopy(data: any): string {
  if (!data) return '';
  
  const lines = [];
  
  // 标题
  lines.push('═══════════════════════════════════════');
  lines.push(data.header?.title || '加班统计报告');
  lines.push('═══════════════════════════════════════');
  
  // 警告信息
  if (data.header?.hasUnderwork && data.header?.underworkWarning) {
    lines.push('');
    lines.push(`⚠️  ${data.header.underworkWarning}`);
    lines.push('');
  }
  
  // 核心收入信息
  lines.push('💰 核心收入信息');
  lines.push('───────────────────────────────────────');
  lines.push(`   总加班收入：${data.income?.total?.formatted || '0元'}${data.income?.total?.note ? ` (${data.income.total.note})` : ''}`);
  lines.push(`   餐费补贴：  ${data.income?.mealAllowance?.formatted || '0元'}`);
  lines.push(`   合计收入：  ${data.income?.totalIncome?.formatted || '0元'}`);
  if (data.income?.actualTotalIncome) {
    lines.push(`   扣减后收入：${data.income.actualTotalIncome.formatted}`);
  }
  
  // 收入明细
  lines.push('');
  lines.push('📈 收入明细');
  lines.push('───────────────────────────────────────');
  lines.push(`   工作日加班：${data.income?.breakdown?.workday?.formatted || '0元'}${data.income?.breakdown?.workday?.note ? ` (${data.income.breakdown.workday.note})` : ''}`);
  lines.push(`   周末加班：  ${data.income?.breakdown?.weekend?.formatted || '0元'}`);
  lines.push(`   节假日加班：${data.income?.breakdown?.holiday?.formatted || '0元'}`);
  
  // 工时统计
  lines.push('');
  lines.push('⏱️  工时统计');
  lines.push('───────────────────────────────────────');
  lines.push(`   工作日加班：${data.hours?.workday?.formatted || '0小时'}`);
  lines.push(`   周末加班：  ${data.hours?.weekend?.formatted || '0小时'}`);
  lines.push(`   节假日加班：${data.hours?.holiday?.formatted || '0小时'}`);
  lines.push(`   总计时长：  ${data.hours?.total?.formatted || '0小时'}`);
  if (data.hours?.actualTotal) {
    lines.push(`   扣减后时长：${data.hours.actualTotal.formatted}`);
  }
  
  // 考勤情况
  lines.push('');
  lines.push('📅 考勤情况');
  lines.push('───────────────────────────────────────');
  lines.push(`   出勤天数：  ${data.attendance?.workdays?.ratio || '0/0'}天`);
  lines.push(`   迟到情况：  ${data.attendance?.late?.message || '无迟到记录'}`);
  
  // 智能建议
  if (data.tips && data.tips.length > 0) {
    lines.push('');
    lines.push('💭 智能建议');
    lines.push('───────────────────────────────────────');
    data.tips.forEach((tip: string) => {
      lines.push(`   ${tip}`);
    });
  }
  
  // 等级评价
  lines.push('');
  lines.push('🏆 加班等级评定');
  lines.push('═══════════════════════════════════════');
  lines.push(`   ${data.rank?.message || '未知评级'}`);
  lines.push('═══════════════════════════════════════');
  
  return lines.join('\n');
}

// 初始化
onMounted(() => {
  // 可以在这里添加初始化逻辑，例如加载保存的设置等
});
</script>

<style scoped>
.overtime-calculator {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.tool-layout {
  display: flex;
  gap: 1.5rem;
  min-height: 600px;
  max-height: 800px;
}

.input-section,
.output-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: var(--vscode-editor-background, #ffffff);
  border-radius: 8px;
  padding: 1rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.08);
  overflow: hidden; /* 防止溢出 */
  height: 100%; /* 确保占满容器高度 */
}

.section-header {
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--vscode-panel-border, #e0e0e0);
}

.section-header h3 {
  margin: 0;
  font-size: 1.2rem;
  color: var(--vscode-foreground, #333333);
}

.actions {
  display: flex;
  gap: 0.5rem;
}

.btn {
  padding: 0.5rem 1rem;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 5px;
  transition: all 0.2s ease;
}

.btn-primary {
  background-color: var(--vscode-button-background, #0e639c);
  color: var(--vscode-button-foreground, white);
}

.btn-primary:hover {
  background-color: var(--vscode-button-hoverBackground, #1177bb);
}

.btn-secondary {
  background-color: var(--vscode-button-secondaryBackground, #3a3d41);
  color: var(--vscode-button-secondaryForeground, white);
}

.btn-secondary:hover {
  background-color: var(--vscode-button-secondaryHoverBackground, #45494e);
}

.input-fields {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

label {
  font-size: 0.9rem;
  color: var(--vscode-input-foreground, #333333);
  font-weight: 500;
}

input[type="month"],
input[type="number"],
textarea {
  padding: 0.5rem;
  background-color: var(--vscode-input-background, #3c3c3c);
  color: var(--vscode-input-foreground, #cccccc);
  border: 1px solid var(--vscode-input-border, #3c3c3c);
  border-radius: 4px;
  font-family: var(--vscode-font-family, 'Consolas, "Courier New", monospace');
}

textarea {
  resize: vertical;
  min-height: 3rem;
}

.json-input-area {
  display: flex;
  flex-direction: column;
  flex: 1;
  overflow: hidden;
  /* 防止溢出 */
}

.json-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin: 0.5rem 0;
  padding: 0.5rem;
  background-color: #f0f7ff;
  border: 1px solid #d0e0f0;
  border-radius: 4px;
  font-size: 0.85rem;
  color: #0066cc;
}

.json-input {
  flex: 1;
  /* 固定高度 */
  font-family: var(--vscode-editor-font-family, 'Consolas, "Courier New", monospace');
  font-size: 0.9rem;
  padding: 0.5rem;
  background-color: var(--vscode-input-background, #f5f5f5);
  color: var(--vscode-input-foreground, #333333);
  border: 1px solid var(--vscode-input-border, #d0d0d0);
  border-radius: 4px;
  margin-top: 0.5rem;
  resize: none;
  /* 禁止调整大小，保持布局稳定 */
}

.error-message {
  margin-top: 0.5rem;
  padding: 0.5rem;
  background-color: var(--vscode-inputValidation-errorBackground, #5a1d1d);
  color: var(--vscode-inputValidation-errorForeground, #ffffff);
  border: 1px solid var(--vscode-inputValidation-errorBorder, #be1100);
  border-radius: 4px;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.status-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.8rem;
  padding: 1rem;
  margin-bottom: 1rem;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: 500;
}

.status-indicator.loading {
  background-color: var(--vscode-editor-inactiveSelectionBackground, #f0f7ff);
  color: var(--vscode-foreground, #666);
  border: 1px solid #d0e0f0;
}

.status-indicator.loading i {
  color: var(--vscode-button-background, #0e639c);
}

.status-indicator.error {
  background-color: var(--vscode-inputValidation-errorBackground, #5a1d1d);
  color: var(--vscode-inputValidation-errorForeground, #ffffff);
  border: 1px solid var(--vscode-inputValidation-errorBorder, #be1100);
}

.status-indicator.error i {
  color: #f48771;
}

.status-indicator.empty {
  background-color: var(--vscode-sideBar-background, #f7f7f7);
  color: var(--vscode-descriptionForeground, #8a8a8a);
  border: 1px solid var(--vscode-panel-border, #e0e0e0);
  font-weight: 400;
}

.status-indicator.empty i {
  color: var(--vscode-descriptionForeground, #999);
}

.summary-text {
  font-size: 0.95rem;
  color: var(--vscode-descriptionForeground, #666);
  line-height: 1.4;
}

/* 配置区域样式 - 紧凑版本 */
.config-section {
  margin-bottom: 0.8rem;
  padding: 0.6rem 0.8rem;
  background-color: var(--vscode-sideBar-background, #f8f8f8);
  border: 1px solid var(--vscode-panel-border, #e0e0e0);
  border-radius: 4px;
}

.config-section h4 {
  margin: 0 0 0.6rem 0;
  font-size: 0.9rem;
  color: var(--vscode-foreground, #333);
  font-weight: 600;
}

.config-row {
  display: flex;
  gap: 0.8rem;
  flex-wrap: wrap;
  align-items: end;
}

.config-item {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  min-width: 120px;
  flex: 1;
}

.config-item label {
  font-size: 0.8rem;
  color: var(--vscode-input-foreground, #333);
  font-weight: 500;
}

.config-input,
.config-select {
  padding: 0.3rem 0.5rem;
  background-color: var(--vscode-input-background, #ffffff);
  color: var(--vscode-input-foreground, #333);
  border: 1px solid var(--vscode-input-border, #d0d0d0);
  border-radius: 3px;
  font-size: 0.85rem;
  outline: none;
  height: 32px;
}

.config-input:focus,
.config-select:focus {
  border-color: var(--vscode-focusBorder, #0e639c);
  box-shadow: 0 0 0 1px var(--vscode-focusBorder, #0e639c);
}

.config-select {
  cursor: pointer;
}



.status-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.8rem;
  padding: 1rem;
  margin-bottom: 1rem;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: 500;
}

.status-indicator.loading {
  background-color: var(--vscode-editor-inactiveSelectionBackground, #f0f7ff);
  color: var(--vscode-foreground, #666);
  border: 1px solid #d0e0f0;
}

.status-indicator.loading i {
  color: var(--vscode-button-background, #0e639c);
}

.status-indicator.error {
  background-color: var(--vscode-inputValidation-errorBackground, #5a1d1d);
  color: var(--vscode-inputValidation-errorForeground, #ffffff);
  border: 1px solid var(--vscode-inputValidation-errorBorder, #be1100);
}

.status-indicator.error i {
  color: #f48771;
}

.status-indicator.empty {
  background-color: var(--vscode-sideBar-background, #f7f7f7);
  color: var(--vscode-descriptionForeground, #8a8a8a);
  border: 1px solid var(--vscode-panel-border, #e0e0e0);
  font-weight: 400;
}

.status-indicator.empty i {
  color: var(--vscode-descriptionForeground, #999);
}

.summary-section {
  padding: 1.5rem;
  background-color: var(--vscode-editor-background, #fff);
  border-radius: 8px;
  margin-top: 1rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
  border: 1px solid var(--vscode-panel-border, #e0e0e0);
}

.summary-header-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 2px solid var(--vscode-panel-border, #e0e0e0);
}

.summary-header-section h4 {
  margin: 0;
  font-size: 1.3rem;
  color: var(--vscode-foreground, #333);
  font-weight: 600;
}

.warning-indicator {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  background-color: #fff3cd;
  color: #856404;
  border: 1px solid #ffeaa7;
  border-radius: 6px;
  font-size: 0.9rem;
  font-weight: 500;
}

.core-overview {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.overview-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.5rem;
  background-color: var(--vscode-sideBar-background, #f7f7f7);
  border: 1px solid var(--vscode-panel-border, #e0e0e0);
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.04);
  transition: all 0.2s ease;
}

.overview-card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  transform: translateY(-1px);
}

.overview-card.primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
}

.overview-card.primary .card-label,
.overview-card.primary .card-note {
  color: rgba(255, 255, 255, 0.9);
}

.card-icon {
  font-size: 2rem;
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 50%;
}

.overview-card:not(.primary) .card-icon {
  background-color: var(--vscode-button-background, #0e639c);
  color: white;
}

.card-content {
  flex: 1;
}

.card-label {
  font-size: 0.9rem;
  color: var(--vscode-descriptionForeground, #666);
  margin-bottom: 0.3rem;
  font-weight: 500;
}

.card-value {
  font-size: 1.4rem;
  font-weight: bold;
  color: var(--vscode-foreground, #222);
  margin-bottom: 0.2rem;
}

.card-value.rank-display {
  text-transform: uppercase;
  letter-spacing: 1px;
  font-size: 1.2rem;
}

.card-note {
  font-size: 0.8rem;
  color: var(--vscode-descriptionForeground, #888);
  font-style: italic;
}

.detail-sections {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.detail-section {
  background-color: var(--vscode-editor-background, #fff);
  border: 1px solid var(--vscode-panel-border, #e0e0e0);
  border-radius: 8px;
  padding: 1.2rem;
  box-shadow: 0 1px 3px rgba(0,0,0,0.02);
}

.detail-section h5 {
  margin: 0 0 1rem 0;
  font-size: 1.1rem;
  color: var(--vscode-foreground, #333);
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid var(--vscode-panel-border, #e0e0e0);
}

.detail-section h5 i {
  color: var(--vscode-button-background, #0e639c);
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.detail-item {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
  padding: 1rem;
  background-color: var(--vscode-sideBar-background, #f7f7f7);
  border-radius: 6px;
  border: 1px solid var(--vscode-panel-border, #e0e0e0);
  transition: all 0.2s ease;
}

.detail-item:hover {
  background-color: var(--vscode-editor-inactiveSelectionBackground, #f0f7ff);
}

.detail-item.total {
  grid-column: 1 / -1;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
  border: none;
}

.detail-item.total .detail-label,
.detail-item.total .detail-extra {
  color: rgba(255, 255, 255, 0.9);
}

.detail-label {
  font-size: 0.9rem;
  color: var(--vscode-descriptionForeground, #666);
  font-weight: 500;
}

.detail-value {
  font-size: 1.1rem;
  font-weight: bold;
  color: var(--vscode-foreground, #222);
}

.detail-value.highlight {
  color: white;
  font-size: 1.2rem;
}

.detail-extra {
  font-size: 0.8rem;
  color: var(--vscode-descriptionForeground, #888);
  font-style: italic;
}

.tips-container {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.tip-item {
  display: flex;
  align-items: flex-start;
  gap: 0.8rem;
  padding: 0.8rem;
  background-color: var(--vscode-editor-inactiveSelectionBackground, #f0f7ff);
  border: 1px solid #d0e0f0;
  border-radius: 6px;
  font-size: 0.9rem;
  line-height: 1.4;
}

.tip-item i {
  color: var(--vscode-button-background, #0e639c);
  margin-top: 0.1rem;
  flex-shrink: 0;
}

.rank-very-low {
  color: #f48771;
}

.rank-low {
  color: #e58833;
}

.rank-medium {
  color: #ddb100;
}

.rank-high {
  color: #89d185;
}

.rank-very-high {
  color: #4ec9b0;
}

.rank-expert {
  color: #9cdcfe;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .tool-layout {
    flex-direction: column;
  }

  .core-overview {
    grid-template-columns: 1fr;
  }

  .detail-grid {
    grid-template-columns: 1fr;
  }
}
</style>