<template>
  <v-container>
    <!-- API设置 -->
    <v-card>
      <v-card-title class="mb-2">OpenAI 接口设置</v-card-title>
      <v-card-text>
        <v-text-field
          v-model="data.apiUrl"
          label="API 地址"
          variant="outlined"
          placeholder="https://api.openai.com/v1"
          class="mb-4"
        ></v-text-field>

        <v-text-field
          v-model="data.apiKey"
          label="API Key"
          variant="outlined"
          placeholder="sk-..."
          :type="showKey ? 'text' : 'password'"
          :append-inner-icon="showKey ? 'mdi-eye-off' : 'mdi-eye'"
          @click:append-inner="showKey = !showKey"
        ></v-text-field>

        <!-- 添加测试按钮 -->
        <div class="d-flex mt-2 mb-6">
          <v-btn
            color="info"
            size="small"
            :loading="testing"
            @click="testConnection"
          >
            <v-icon v-if="data.isApiValid" left color="success" class="mr-2">
              mdi-check-circle
            </v-icon>
            测试连接
            <v-icon right class="ml-2">mdi-connection</v-icon>
          </v-btn>
          <v-chip
            v-if="data.lastTestTime"
            size="small"
            class="ml-6"
            :color="data.isApiValid ? 'success' : 'error'"
          >
            上次检测: {{ formatDate(data.lastTestTime) }}
          </v-chip>
        </div>
        <v-textarea
          v-model="data.summaryPrompt"
          label="文章总结提示词"
          variant="outlined"
          rows="4"
          placeholder="请用简洁的语言总结这篇文章的主要内容..."
        ></v-textarea>

        <!-- 模型选择部分 -->
        <div v-if="data.isApiValid">
          <v-divider class="my-4"></v-divider>
          <div class="d-flex align-center justify-space-between mb-4">
            <v-select
              v-model="data.selectedModel"
              :items="availableModels"
              label="选择模型"
              variant="outlined"
              :loading="loadingModels"
              :disabled="loadingModels"
              class="flex-grow-1"
              hide-details
            >
              <template v-slot:prepend>
                <v-icon>mdi-robot-outline</v-icon>
              </template>
            </v-select>
            <v-btn
              color="primary"
              variant="text"
              height="50"
              class="ml-2"
              :loading="loadingModels"
              @click="fetchModels"
            >
              <v-icon>mdi-refresh</v-icon>
            </v-btn>
          </div>
        </div>
      </v-card-text>
    </v-card>

    <!-- 保存按钮 -->
    <v-row class="mt-4">
      <v-col>
        <v-btn color="primary" @click="saveSettings">
          保存设置
          <v-icon right class="ml-2">mdi-content-save</v-icon>
        </v-btn>
        <v-btn
          color="error"
          variant="outlined"
          class="ml-2"
          @click="resetSettings"
        >
          重置默认
          <v-icon right class="ml-2">mdi-refresh</v-icon>
        </v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import { useSettingsStore } from "@/store";
import { storeToRefs } from "pinia";
import { formatDate } from "@/utils/dateFormat";

const settingsStore = useSettingsStore();
const { integrated: data } = storeToRefs(settingsStore);
const showKey = ref(false);
const testing = ref(false);
const loadingModels = ref(false);
const availableModels = ref<string[]>([]);

// 获取可用模型列表
const fetchModels = async () => {
  loadingModels.value = true;
  try {
    const response = await fetch(`${data.value.apiUrl}/models`, {
      headers: {
        Authorization: `Bearer ${data.value.apiKey}`,
      },
    });

    if (response.ok) {
      const result = await response.json();
      availableModels.value = result.data.map((model: any) => model.id).sort();
    } else {
      throw new Error(response.statusText);
    }
  } catch (error: any) {
    alert(`获取模型列表失败：${error.message}`);
  } finally {
    loadingModels.value = false;
  }
};

// 修改测试连接方法，成功后自动获取模型列表
const testConnection = async () => {
  testing.value = true;
  try {
    const response = await fetch(`${data.value.apiUrl}/models`, {
      headers: {
        Authorization: `Bearer ${data.value.apiKey}`,
      },
    });

    data.value.isApiValid = response.ok;
    data.value.lastTestTime = new Date().toISOString();

    if (response.ok) {
      alert("连接成功！API 配置有效。");
      // 连接成功后自动获取模型列表
      await fetchModels();
    } else {
      alert(`连接失败：${response.statusText}`);
    }
  } catch (error: any) {
    data.value.isApiValid = false;
    data.value.lastTestTime = new Date().toISOString();
    alert(`连接错误：${error.message}`);
  } finally {
    testing.value = false;
  }
};

// 方法定义
const saveSettings = () => {
  settingsStore.saveToLocalStorage();
};

const resetSettings = () => {
  settingsStore.resetIntegratedSettings();
};
</script>
