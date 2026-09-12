<template>
  <div class="settings-section">
    <h3 class="section-title">Device Information</h3>

    <div class="setting-group">
      <h4 class="group-title">Current Status</h4>
      <div class="status-box">
        <div class="status-row">
          <span class="status-key">Login Status</span>
          <span :class="['status-value', deviceInfo?.loggedIn ? 'status-on' : 'status-off']">
            {{ deviceInfo?.loggedIn ? 'Logged in (fixed device code)' : 'Not logged in (random device code)' }}
          </span>
        </div>
        <div class="status-row">
          <span class="status-key">Device Code (X-App-Device)</span>
          <code class="status-code" :title="deviceInfo?.deviceCode">{{ deviceInfo?.deviceCode || 'Loading...' }}</code>
        </div>
        <p class="tray-tip">
          <i class="fas fa-info-circle"></i>
          When not logged in, the device code is randomly generated (fixed after the first generation on each computer); after logging in, the fixed device code bound to the account is used (by default consistent with the official SDK and can pass write-operation validation). Do not modify it manually.
        </p>
      </div>
    </div>

    <div class="setting-group">
      <h4 class="group-title">Custom Device Fingerprint</h4>
      <div class="setting-row">
        <div class="row-info">
          <span class="row-label">Enable Custom Device Information</span>
          <span class="row-sub">Customize the device model, version, and system information in request headers (when disabled, the client default values are used)</span>
        </div>
        <AppSwitch v-model="settingsStore.settings.deviceFingerprint.customFingerprint" />
      </div>
    </div>

    <template v-if="settingsStore.settings.deviceFingerprint.customFingerprint">
      <div class="setting-group">
        <h4 class="group-title">Device Model Template</h4>
        <div class="setting-row">
          <div class="row-info">
            <span class="row-label">Preset Model</span>
            <span class="row-sub">Quickly apply a common device model template, or select "Custom" to enter it manually</span>
          </div>
          <select v-model="presetModel" class="text-input select-input">
            <option value="">Custom Model</option>
            <option v-for="p in DEVICE_PRESETS" :key="p.model" :value="p.model">
              {{ p.label }}（{{ p.model }}）
            </option>
          </select>
        </div>

        <div class="setting-row">
          <div class="row-info">
            <span class="row-label">Device Model</span>
            <span class="row-sub">Embedded in the User-Agent, e.g. 23113RKC6C (Xiaomi 14)</span>
          </div>
          <input
            v-model="settingsStore.settings.deviceFingerprint.model"
            type="text"
            class="text-input"
            placeholder="e.g. 23113RKC6C"
            maxlength="40"
          />
        </div>

        <div class="field-row">
          <div class="row-info">
            <span class="row-label">Android Version</span>
            <span class="row-sub">Android version number in the UA</span>
          </div>
          <input
            v-model="settingsStore.settings.deviceFingerprint.androidVersion"
            type="text"
            class="text-input small-input"
            placeholder="16"
            maxlength="8"
          />
          <div class="row-info">
            <span class="row-label">Build Number</span>
            <span class="row-sub">Build version in the UA</span>
          </div>
          <input
            v-model="settingsStore.settings.deviceFingerprint.build"
            type="text"
            class="text-input"
            placeholder="AQ3A.250226.002"
            maxlength="40"
          />
        </div>
      </div>

      <div class="setting-group">
        <h4 class="group-title">Application & System Information</h4>
        <div class="field-row">
          <div class="row-info">
            <span class="row-label">App Version (X-App-Version)</span>
            <span class="row-sub">Must not be lower than the minimum version officially supported by CoolApk</span>
          </div>
          <input
            v-model="settingsStore.settings.deviceFingerprint.appVersion"
            type="text"
            class="text-input small-input"
            placeholder="16.2.0"
            maxlength="20"
          />
          <div class="row-info">
            <span class="row-label">Version Code (X-App-Code)</span>
            <span class="row-sub">Also applied to X-App-Supported</span>
          </div>
          <input
            v-model="settingsStore.settings.deviceFingerprint.appCode"
            type="text"
            class="text-input small-input"
            placeholder="2604201"
            maxlength="12"
          />
        </div>

        <div class="field-row">
          <div class="row-info">
            <span class="row-label">SDK Int (X-Sdk-Int)</span>
            <span class="row-sub">Android SDK version number</span>
          </div>
          <input
            v-model="settingsStore.settings.deviceFingerprint.sdkInt"
            type="text"
            class="text-input small-input"
            placeholder="35"
            maxlength="4"
          />
          <div class="row-info">
            <span class="row-label">Language (X-Sdk-Locale)</span>
            <span class="row-sub">e.g. zh-CN / en-US</span>
          </div>
          <input
            v-model="settingsStore.settings.deviceFingerprint.locale"
            type="text"
            class="text-input small-input"
            placeholder="zh-CN"
            maxlength="16"
          />
        </div>

        <div class="setting-row">
          <div class="row-info">
            <span class="row-label">Dark Mode (X-Dark-Mode)</span>
            <span class="row-sub">Simulates the client's light/dark mode state independently of the interface theme</span>
          </div>
          <AppSwitch
            :model-value="settingsStore.settings.deviceFingerprint.darkMode === '1'"
            @update:model-value="(v: boolean) => (settingsStore.settings.deviceFingerprint.darkMode = v ? '1' : '0')"
          />
        </div>
      </div>

      <div class="setting-group">
        <h4 class="group-title">Preview</h4>
        <div class="preview-box">
          <div class="preview-row">
            <span class="preview-key">User-Agent</span>
            <code class="preview-value">{{ previewUserAgent }}</code>
          </div>
          <div class="preview-row">
            <span class="preview-key">X-App-Version</span>
            <code class="preview-value">{{ fingerprint.appVersion || '16.2.0' }}</code>
            <span class="preview-key">X-App-Code</span>
            <code class="preview-value">{{ fingerprint.appCode || '2604201' }}</code>
          </div>
          <div class="preview-row">
            <span class="preview-key">X-Sdk-Int</span>
            <code class="preview-value">{{ fingerprint.sdkInt || '35' }}</code>
            <span class="preview-key">X-Sdk-Locale</span>
            <code class="preview-value">{{ fingerprint.locale || 'zh-CN' }}</code>
            <span class="preview-key">X-Dark-Mode</span>
            <code class="preview-value">{{ fingerprint.darkMode }}</code>
          </div>
          <p v-if="versionWarning" class="version-warning">
            <i class="fas fa-exclamation-triangle"></i>
            {{ versionWarning }}
          </p>
        </div>
      </div>

      <div class="setting-group">
        <button class="reset-button" @click="resetToDefault">
          <i class="fas fa-undo"></i>
          Restore Default Settings
        </button>
      </div>
    </template>

    <div class="setting-group">
      <h4 class="group-title">Important Notes</h4>
      <p class="tray-tip">
        <i class="fas fa-info-circle"></i>
        The device code (X-App-Device) and request token (X-App-Token) are bound to the account and cannot be customized. After modifying the device model, version, or other fields, if CoolApk returns "Abnormal network environment" or "Please upgrade the client", it means the combination was rejected by the server. Restore the defaults or switch to another device model template.
      </p>
      <p class="tray-tip">
        <i class="fas fa-info-circle"></i>
        Changes take effect immediately without restarting the client and apply to all requests (including posting feeds, comments, likes, etc.).
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';
import { useSettingsStore, buildDeviceUserAgent } from '../../stores/settings';
import AppSwitch from '../../components/common/AppSwitch.vue';
import { DEVICE_PRESETS } from '../../utils/devicePresets';
import { invoke } from '@tauri-apps/api/core';
import { useAuthStore } from '../../stores/auth';
import type { DeviceFingerprintSettings } from '../../types/settings';

/** 当前生效设备信息（Rust 端查询）：登录态 + 设备码 */
const deviceInfo = ref<{ loggedIn: boolean; deviceCode: string } | null>(null);

const authStore = useAuthStore();

async function loadDeviceInfo() {
  try {
    const res = await invoke<any>('get_device_info');
    if (res && res.code === 200) {
      deviceInfo.value = res.data;
    }
  } catch (err) {
    console.warn('获取设备信息失败:', err);
  }
}
onMounted(loadDeviceInfo);
// 登录/登出/切换账号后刷新设备码状态
watch(
  () => authStore.user?.uid,
  () => loadDeviceInfo()
);

const settingsStore = useSettingsStore();

const fingerprint = computed(() => settingsStore.settings.deviceFingerprint);
const previewUserAgent = computed(() => buildDeviceUserAgent(fingerprint.value));

const presetModel = computed({
  get: () => {
    const f = fingerprint.value;
    return DEVICE_PRESETS.some((p) => p.model === f.model.trim()) ? f.model.trim() : '';
  },
  set: (model: string) => {
    const preset = DEVICE_PRESETS.find((item) => item.model === model);
    if (!preset) return;
    Object.assign(fingerprint.value, {
      model: preset.model,
      androidVersion: preset.androidVersion,
      build: preset.build,
    });
  },
});

const versionWarning = computed(() => {
  const f = fingerprint.value;
  const code = Number(f.appCode);
  if (!Number.isNaN(code) && code > 0 && code < 2604201) {
    return `版本号 ${f.appCode} 低于当前官方版本 2604201，服务端可能拒绝请求（err_request_need_upgrade_new_version）。`;
  }
  const version = f.appVersion.trim();
  if (version) {
    const major = Number(version.split('.')[0]);
    if (!Number.isNaN(major) && major > 0 && major < 16) {
      return `App 版本 ${version} 低于当前官方主版本 16，服务端可能拒绝请求。`;
    }
  }
  return '';
});

function resetToDefault() {
  const defaults: DeviceFingerprintSettings = {
    customFingerprint: true,
    model: '23113RKC6C',
    androidVersion: '16',
    build: 'AQ3A.250226.002',
    appVersion: '16.2.0',
    appCode: '2604201',
    sdkInt: '35',
    locale: 'zh-CN',
    darkMode: '0',
  };
  Object.assign(settingsStore.settings.deviceFingerprint, defaults);
}
</script>

<style scoped>
.settings-section {
  display: flex;
  flex-direction: column;
  gap: var(--space-6);
  max-width: 760px;
}

.section-title {
  font-size: var(--font-size-title-md);
  font-weight: var(--font-weight-bold);
  color: var(--text-primary);
  border-bottom: 1px solid var(--border);
  padding-bottom: var(--space-3);
}

.setting-group {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}

.group-title {
  font-size: var(--font-size-title-sm);
  font-weight: var(--font-weight-semibold);
  color: var(--text-primary);
}

.setting-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);
  padding: var(--space-3) 0;
  border-bottom: 1px solid var(--border-light);
}

.field-row {
  display: flex;
  align-items: center;
  gap: var(--space-4);
  padding: var(--space-3) 0;
  border-bottom: 1px solid var(--border-light);
}

.row-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex: 1;
  min-width: 120px;
}

.row-label {
  font-size: var(--font-size-sub);
  font-weight: var(--font-weight-medium);
  color: var(--text-primary);
}

.row-sub {
  font-size: var(--font-size-caption);
  color: var(--text-tertiary);
}

.text-input {
  background-color: var(--background);
  border: 1px solid var(--border);
  border-radius: var(--radius-control);
  padding: 6px 12px;
  font-size: var(--font-size-sub);
  color: var(--text-primary);
  outline: none;
  width: 220px;
  transition: border-color var(--duration-fast) var(--ease-default);
}

.small-input {
  width: 130px;
}

.select-input {
  width: 230px;
  cursor: pointer;
}

.text-input:hover,
.text-input:focus {
  border-color: var(--brand-primary);
}

.preview-box {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  background-color: var(--background);
  border: 1px solid var(--border);
  border-radius: var(--radius-card);
  padding: var(--space-4);
}

.preview-row {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  flex-wrap: wrap;
}

.preview-key {
  font-size: var(--font-size-caption);
  color: var(--text-tertiary);
  white-space: nowrap;
}

.preview-value {
  font-family: var(--font-mono, Consolas, monospace);
  font-size: var(--font-size-caption);
  color: var(--text-primary);
  background-color: var(--surface);
  border-radius: var(--radius-control);
  padding: 2px 8px;
  word-break: break-all;
}

.version-warning {
  font-size: var(--font-size-caption);
  color: #e0533d;
  margin: var(--space-2) 0 0;
}

.status-box {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  background-color: var(--background);
  border: 1px solid var(--border);
  border-radius: var(--radius-card);
  padding: var(--space-4);
}

.status-row {
  display: flex;
  align-items: center;
  gap: var(--space-3);
}

.status-key {
  font-size: var(--font-size-caption);
  color: var(--text-tertiary);
  white-space: nowrap;
  width: 130px;
}

.status-value {
  font-size: var(--font-size-sub);
  font-weight: var(--font-weight-medium);
}

.status-on {
  color: var(--brand-primary);
}

.status-off {
  color: #e0533d;
}

.status-code {
  font-family: var(--font-mono, Consolas, monospace);
  font-size: var(--font-size-caption);
  color: var(--text-primary);
  background-color: var(--surface);
  border-radius: var(--radius-control);
  padding: 2px 8px;
  word-break: break-all;
  max-width: 420px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.reset-button {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  align-self: flex-start;
  background-color: var(--background);
  border: 1px solid var(--border);
  border-radius: var(--radius-control);
  padding: 8px 16px;
  font-size: var(--font-size-sub);
  color: var(--text-secondary);
  cursor: pointer;
  transition: all var(--duration-fast) var(--ease-default);
}

.reset-button:hover {
  border-color: var(--brand-primary);
  color: var(--brand-primary);
}

.tray-tip {
  font-size: var(--font-size-caption);
  color: var(--text-tertiary);
  display: flex;
  gap: var(--space-2);
  align-items: flex-start;
  margin: 0;
}

.tray-tip i {
  margin-top: 2px;
}
</style>
