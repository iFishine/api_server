<template>
  <div class="uuid-generator">
    <div class="generator-layout">
      <!-- UUID生成器 -->
      <div class="generation-section">
        <div class="section-header">
          <h3>UUID Generator</h3>
          <div class="actions">
            <button @click="generateUuid" class="btn btn-primary">
              <i class="fas fa-plus"></i> Generate New
            </button>
            <button @click="clearAll" class="btn btn-secondary">Clear All</button>
          </div>
        </div>

        <div class="uuid-types">
          <div class="type-selector">
            <label>UUID Version:</label>
            <select v-model="selectedVersion" @change="generateUuid">
              <option value="v4">Version 4 (Random)</option>
              <option value="v1">Version 1 (Timestamp + MAC)</option>
              <option value="nil">Nil UUID</option>
            </select>
          </div>

          <div class="format-options">
            <label>Output Format:</label>
            <div class="format-checkboxes">
              <label>
                <input type="checkbox" v-model="includeBraces" @change="updateFormats">
                Braces { }
              </label>
              <label>
                <input type="checkbox" v-model="includeHyphens" @change="updateFormats">
                Hyphens -
              </label>
              <label>
                <input type="checkbox" v-model="upperCase" @change="updateFormats">
                Uppercase
              </label>
            </div>
          </div>
        </div>

        <div class="current-uuid">
          <label>Generated UUID:</label>
          <div class="uuid-display">
            <input v-model="currentUuid" readonly class="uuid-input" />
            <button @click="copyUuid(currentUuid)" class="copy-btn">
              <i class="fas fa-copy"></i>
            </button>
          </div>
        </div>

        <!-- 不同格式显示 -->
        <div class="format-variants">
          <h4>Format Variants:</h4>
          <div class="variant-list">
            <div v-for="(variant, index) in formatVariants" :key="index" class="variant-item">
              <label>{{ variant.name }}:</label>
              <div class="variant-display">
                <input :value="variant.value" readonly class="variant-input" />
                <button @click="copyUuid(variant.value)" class="copy-btn-small">
                  <i class="fas fa-copy"></i>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 批量生成 -->
      <div class="batch-section">
        <div class="section-header">
          <h3>Bulk Generation</h3>
          <div class="actions">
            <button @click="generateBulk" class="btn btn-primary">Generate Bulk</button>
            <button @click="clearBulk" class="btn btn-secondary">Clear</button>
          </div>
        </div>

        <div class="bulk-options">
          <div class="option-group">
            <label>Count:</label>
            <input v-model.number="bulkCount" type="number" min="1" max="1000" class="count-input" />
          </div>
          <div class="option-group">
            <label>Format:</label>
            <select v-model="bulkFormat">
              <option value="standard">Standard (with hyphens)</option>
              <option value="compact">Compact (no hyphens)</option>
              <option value="braces">With braces</option>
              <option value="uppercase">Uppercase</option>
            </select>
          </div>
        </div>

        <div class="bulk-output">
          <label>Generated UUIDs:</label>
          <textarea
            v-model="bulkOutput"
            readonly
            class="bulk-textarea"
            placeholder="Bulk generated UUIDs will appear here..."
          ></textarea>
          <div class="bulk-actions">
            <button @click="copyBulkUuids" class="copy-btn">
              <i class="fas fa-copy"></i> Copy All
            </button>
            <button @click="downloadUuids" class="download-btn">
              <i class="fas fa-download"></i> Download
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- UUID验证器 -->
    <div class="validation-section">
      <div class="section-header">
        <h3>UUID Validator</h3>
        <button @click="validateUuid" class="btn btn-primary">Validate</button>
      </div>
      <div class="validator-content">
        <div class="input-area">
          <label>UUID to validate:</label>
          <input
            v-model="uuidToValidate"
            placeholder="Enter UUID to validate..."
            class="validate-input"
            @input="validateUuid"
          />
        </div>
        <div v-if="validationResult" class="validation-result">
          <div :class="['result-status', validationResult.isValid ? 'valid' : 'invalid']">
            <i :class="validationResult.isValid ? 'fas fa-check-circle' : 'fas fa-times-circle'"></i>
            {{ validationResult.isValid ? 'Valid UUID' : 'Invalid UUID' }}
          </div>
          <div v-if="validationResult.isValid" class="uuid-info">
            <div class="info-item">
              <strong>Version:</strong> {{ validationResult.version }}
            </div>
            <div class="info-item">
              <strong>Variant:</strong> {{ validationResult.variant }}
            </div>
            <div v-if="validationResult.timestamp" class="info-item">
              <strong>Timestamp:</strong> {{ validationResult.timestamp }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- UUID统计 -->
    <div class="stats-section">
      <h3>Generation Statistics</h3>
      <div class="stats-grid">
        <div class="stat-item">
          <span class="stat-label">Total Generated:</span>
          <span class="stat-value">{{ totalGenerated }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">Session Count:</span>
          <span class="stat-value">{{ sessionCount }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">Last Generated:</span>
          <span class="stat-value">{{ lastGeneratedTime }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

const selectedVersion = ref('v4');
const includeBraces = ref(false);
const includeHyphens = ref(true);
const upperCase = ref(false);
const currentUuid = ref('');
const bulkCount = ref(10);
const bulkFormat = ref('standard');
const bulkOutput = ref('');
const uuidToValidate = ref('');
const validationResult = ref<any>(null);
const totalGenerated = ref(0);
const sessionCount = ref(0);
const lastGeneratedTime = ref('');

// 格式变体计算属性
const formatVariants = computed(() => {
  if (!currentUuid.value) return [];
  
  const baseUuid = currentUuid.value.replace(/[{}-]/g, '').toLowerCase();
  
  return [
    {
      name: 'Standard',
      value: formatUuid(baseUuid, true, false, false)
    },
    {
      name: 'Compact',
      value: formatUuid(baseUuid, false, false, false)
    },
    {
      name: 'With Braces',
      value: formatUuid(baseUuid, true, true, false)
    },
    {
      name: 'Uppercase',
      value: formatUuid(baseUuid, true, false, true)
    },
    {
      name: 'Uppercase + Braces',
      value: formatUuid(baseUuid, true, true, true)
    }
  ];
});

const generateUuid = () => {
  let uuid = '';
  
  switch (selectedVersion.value) {
    case 'v4':
      uuid = generateV4();
      break;
    case 'v1':
      uuid = generateV1();
      break;
    case 'nil':
      uuid = '00000000-0000-0000-0000-000000000000';
      break;
  }
  
  currentUuid.value = formatUuid(uuid, includeHyphens.value, includeBraces.value, upperCase.value);
  updateStats();
};

const generateV4 = (): string => {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, (c) => {
    const r = Math.random() * 16 | 0;
    const v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
};

const generateV1 = (): string => {
  // 简化的 V1 UUID 生成（实际实现会更复杂）
  const timestamp = Date.now();
  const timeHex = timestamp.toString(16).padStart(12, '0');
  const clockSeq = Math.random().toString(16).substr(2, 4);
  const node = Math.random().toString(16).substr(2, 12);
  
  return `${timeHex.substr(0, 8)}-${timeHex.substr(8, 4)}-1${timeHex.substr(12, 3)}-${clockSeq}-${node}`;
};

const formatUuid = (uuid: string, hyphens: boolean, braces: boolean, upper: boolean): string => {
  let formatted = uuid.replace(/[{}-]/g, '');
  
  if (hyphens) {
    formatted = `${formatted.substr(0, 8)}-${formatted.substr(8, 4)}-${formatted.substr(12, 4)}-${formatted.substr(16, 4)}-${formatted.substr(20, 12)}`;
  }
  
  if (braces) {
    formatted = `{${formatted}}`;
  }
  
  if (upper) {
    formatted = formatted.toUpperCase();
  }
  
  return formatted;
};

const updateFormats = () => {
  if (currentUuid.value) {
    const baseUuid = currentUuid.value.replace(/[{}-]/g, '').toLowerCase();
    currentUuid.value = formatUuid(baseUuid, includeHyphens.value, includeBraces.value, upperCase.value);
  }
};

const generateBulk = () => {
  const uuids: string[] = [];
  
  for (let i = 0; i < bulkCount.value; i++) {
    let uuid = generateV4();
    
    switch (bulkFormat.value) {
      case 'compact':
        uuid = uuid.replace(/-/g, '');
        break;
      case 'braces':
        uuid = `{${uuid}}`;
        break;
      case 'uppercase':
        uuid = uuid.toUpperCase();
        break;
    }
    
    uuids.push(uuid);
  }
  
  bulkOutput.value = uuids.join('\n');
  totalGenerated.value += bulkCount.value;
  sessionCount.value += bulkCount.value;
  lastGeneratedTime.value = new Date().toLocaleTimeString();
};

const validateUuid = () => {
  if (!uuidToValidate.value.trim()) {
    validationResult.value = null;
    return;
  }
  
  const uuid = uuidToValidate.value.trim();
  const uuidRegex = /^[{]?[0-9a-fA-F]{8}-?[0-9a-fA-F]{4}-?[0-9a-fA-F]{4}-?[0-9a-fA-F]{4}-?[0-9a-fA-F]{12}[}]?$/;
  
  if (!uuidRegex.test(uuid)) {
    validationResult.value = { isValid: false };
    return;
  }
  
  const cleanUuid = uuid.replace(/[{}-]/g, '');
  const version = parseInt(cleanUuid.charAt(12), 16);
  const variant = parseInt(cleanUuid.charAt(16), 16);
  
  let variantName = 'Unknown';
  if ((variant & 0x8) === 0) variantName = 'NCS';
  else if ((variant & 0xC) === 0x8) variantName = 'RFC 4122';
  else if ((variant & 0xE) === 0xC) variantName = 'Microsoft';
  
  validationResult.value = {
    isValid: true,
    version: `Version ${version}`,
    variant: variantName,
    timestamp: version === 1 ? 'Available (V1)' : 'Not available'
  };
};

const updateStats = () => {
  totalGenerated.value++;
  sessionCount.value++;
  lastGeneratedTime.value = new Date().toLocaleTimeString();
};

const copyUuid = async (uuid: string) => {
  try {
    await navigator.clipboard.writeText(uuid);
  } catch (err) {
    console.error('Failed to copy:', err);
  }
};

const copyBulkUuids = async () => {
  if (bulkOutput.value) {
    try {
      await navigator.clipboard.writeText(bulkOutput.value);
    } catch (err) {
      console.error('Failed to copy:', err);
    }
  }
};

const downloadUuids = () => {
  if (bulkOutput.value) {
    const blob = new Blob([bulkOutput.value], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'uuids.txt';
    a.click();
    URL.revokeObjectURL(url);
  }
};

const clearAll = () => {
  currentUuid.value = '';
};

const clearBulk = () => {
  bulkOutput.value = '';
};

onMounted(() => {
  generateUuid();
  // 从 localStorage 加载统计数据
  const savedStats = localStorage.getItem('uuid-generator-stats');
  if (savedStats) {
    const stats = JSON.parse(savedStats);
    totalGenerated.value = stats.total || 0;
  }
});

// 保存统计数据
const saveStats = () => {
  localStorage.setItem('uuid-generator-stats', JSON.stringify({
    total: totalGenerated.value
  }));
};

// 监听统计变化并保存
setInterval(saveStats, 5000);
</script>

