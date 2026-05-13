<script setup lang="ts">
import {
  NAlert,
  NButton,
  NCard,
  NForm,
  NFormItem,
  NInput,
  NSelect,
  NSpin,
  NSwitch,
  NTag,
  useMessage,
} from 'naive-ui'
import { computed, onMounted, reactive, ref, watch } from 'vue'

import { listWebsiteInfo, updateWebsiteInfo } from '@/services/website-info'

const emit = defineEmits<{ 'dirty-change': [dirty: boolean] }>()

type FirstViewportVariant = 'grtblog' | 'yohaku'
type JsonRecord = Record<string, unknown>

interface FirstViewportForm {
  variant: FirstViewportVariant
  hideGlobalSidebarOnHome: boolean
  hideGlobalMobileNavOnHome: boolean
  showTopNav: boolean
  showScrollHint: boolean
  avatarUrl: string
  titleLine1Prefix: string
  titleLine1Highlight: string
  titleLine1Suffix: string
  titleLine2Prefix: string
  titleCode: string
  titleLine2Suffix: string
  description: string
  mottoText: string
}

const defaultForm: FirstViewportForm = {
  variant: 'grtblog',
  hideGlobalSidebarOnHome: false,
  hideGlobalMobileNavOnHome: false,
  showTopNav: true,
  showScrollHint: true,
  avatarUrl: '',
  titleLine1Prefix: "Hi, I'm ",
  titleLine1Highlight: 'grtsinry43',
  titleLine1Suffix: ' 👋',
  titleLine2Prefix: 'A Full Stack ',
  titleCode: '<Developer />',
  titleLine2Suffix: ' |',
  description: 'Java & JavaScript full-stack developer committed to crafting excellent software.',
  mottoText:
    '热衷于在逻辑与感性的缝隙中构建数字花园。\n也许，代码是现代的诗歌，而文字是思想的快照。',
}

const variantOptions: Array<{ label: string; value: FirstViewportVariant }> = [
  { label: 'GrtBlog 默认首屏', value: 'grtblog' },
  { label: 'Yohaku 风格首屏', value: 'yohaku' },
]

const message = useMessage()
const loading = ref(false)
const saving = ref(false)
const themeExtendInfo = ref<JsonRecord>({})
const form = reactive<FirstViewportForm>({ ...defaultForm })
const originalSnapshot = ref(serializeForm(defaultForm))

const isDirty = computed(() => serializeForm(form) !== originalSnapshot.value)
const isYohaku = computed(() => form.variant === 'yohaku')

watch(isDirty, (dirty) => emit('dirty-change', dirty), { immediate: true })

function isRecord(value: unknown): value is JsonRecord {
  return typeof value === 'object' && value !== null && !Array.isArray(value)
}

function cloneRecord(value: JsonRecord): JsonRecord {
  return JSON.parse(JSON.stringify(value)) as JsonRecord
}

function serializeForm(value: FirstViewportForm): string {
  return JSON.stringify({
    variant: value.variant,
    hideGlobalSidebarOnHome: value.hideGlobalSidebarOnHome,
    hideGlobalMobileNavOnHome: value.hideGlobalMobileNavOnHome,
    showTopNav: value.showTopNav,
    showScrollHint: value.showScrollHint,
    avatarUrl: value.avatarUrl,
    titleLine1Prefix: value.titleLine1Prefix,
    titleLine1Highlight: value.titleLine1Highlight,
    titleLine1Suffix: value.titleLine1Suffix,
    titleLine2Prefix: value.titleLine2Prefix,
    titleCode: value.titleCode,
    titleLine2Suffix: value.titleLine2Suffix,
    description: value.description,
    mottoText: value.mottoText,
  })
}

function toStringValue(value: unknown, fallback = ''): string {
  return typeof value === 'string' ? value : fallback
}

function toBoolean(value: unknown, fallback: boolean): boolean {
  if (typeof value === 'boolean') return value
  if (typeof value === 'string') {
    if (value.toLowerCase() === 'true') return true
    if (value.toLowerCase() === 'false') return false
  }
  return fallback
}

function normalizeFirstViewport(value: unknown): FirstViewportForm {
  if (!isRecord(value)) {
    return { ...defaultForm }
  }
  return {
    ...defaultForm,
    variant: value.variant === 'yohaku' ? 'yohaku' : 'grtblog',
    hideGlobalSidebarOnHome: toBoolean(
      value.hideGlobalSidebarOnHome,
      defaultForm.hideGlobalSidebarOnHome,
    ),
    hideGlobalMobileNavOnHome: toBoolean(
      value.hideGlobalMobileNavOnHome,
      defaultForm.hideGlobalMobileNavOnHome,
    ),
    showTopNav: toBoolean(value.showTopNav, defaultForm.showTopNav),
    showScrollHint: toBoolean(value.showScrollHint, defaultForm.showScrollHint),
  }
}

function readTitleLines(template: unknown) {
  if (!Array.isArray(template)) {
    return {
      titleLine1Prefix: defaultForm.titleLine1Prefix,
      titleLine1Highlight: defaultForm.titleLine1Highlight,
      titleLine1Suffix: defaultForm.titleLine1Suffix,
      titleLine2Prefix: defaultForm.titleLine2Prefix,
      titleCode: defaultForm.titleCode,
      titleLine2Suffix: defaultForm.titleLine2Suffix,
    }
  }

  const firstLine: JsonRecord[] = []
  const secondLine: JsonRecord[] = []
  let currentLine = firstLine
  for (const node of template) {
    if (!isRecord(node)) continue
    if (node.type === 'br') {
      currentLine = secondLine
      continue
    }
    currentLine.push(node)
  }

  const readLineWithHighlight = (nodes: JsonRecord[]) => {
    const idx = nodes.findIndex((node) => node.variant === 'hero_h1_highlight')
    if (idx < 0) {
      return {
        prefix: nodes.map((node) => toStringValue(node.text)).join(''),
        highlight: '',
        suffix: '',
      }
    }
    return {
      prefix: nodes
        .slice(0, idx)
        .map((node) => toStringValue(node.text))
        .join(''),
      highlight: toStringValue(nodes[idx]?.text),
      suffix: nodes
        .slice(idx + 1)
        .map((node) => toStringValue(node.text))
        .join(''),
    }
  }

  const readLineWithCode = (nodes: JsonRecord[]) => {
    const idx = nodes.findIndex(
      (node) => node.type === 'code' || node.variant === 'hero_code_inline',
    )
    if (idx < 0) {
      return {
        prefix: nodes.map((node) => toStringValue(node.text)).join(''),
        code: '',
        suffix: '',
      }
    }
    return {
      prefix: nodes
        .slice(0, idx)
        .map((node) => toStringValue(node.text))
        .join(''),
      code: toStringValue(nodes[idx]?.text),
      suffix: nodes
        .slice(idx + 1)
        .map((node) => toStringValue(node.text))
        .join(''),
    }
  }

  const line1 = readLineWithHighlight(firstLine)
  const line2 = readLineWithCode(secondLine)
  return {
    titleLine1Prefix: line1.prefix || defaultForm.titleLine1Prefix,
    titleLine1Highlight: line1.highlight || defaultForm.titleLine1Highlight,
    titleLine1Suffix: line1.suffix || defaultForm.titleLine1Suffix,
    titleLine2Prefix: line2.prefix || defaultForm.titleLine2Prefix,
    titleCode: line2.code || defaultForm.titleCode,
    titleLine2Suffix: line2.suffix || defaultForm.titleLine2Suffix,
  }
}

function normalizeHero(value: unknown): Partial<FirstViewportForm> {
  if (!isRecord(value)) return {}
  return {
    avatarUrl: toStringValue(value.avatarUrl, defaultForm.avatarUrl),
    description: toStringValue(value.description, defaultForm.description),
    mottoText: Array.isArray(value.mottoLines)
      ? value.mottoLines
          .map((item) => toStringValue(item))
          .filter(Boolean)
          .join('\n')
      : defaultForm.mottoText,
    ...readTitleLines(isRecord(value.title) ? value.title.template : value.titleTemplate),
  }
}

function assignForm(value: FirstViewportForm) {
  form.variant = value.variant
  form.hideGlobalSidebarOnHome = value.hideGlobalSidebarOnHome
  form.hideGlobalMobileNavOnHome = value.hideGlobalMobileNavOnHome
  form.showTopNav = value.showTopNav
  form.showScrollHint = value.showScrollHint
  form.avatarUrl = value.avatarUrl
  form.titleLine1Prefix = value.titleLine1Prefix
  form.titleLine1Highlight = value.titleLine1Highlight
  form.titleLine1Suffix = value.titleLine1Suffix
  form.titleLine2Prefix = value.titleLine2Prefix
  form.titleCode = value.titleCode
  form.titleLine2Suffix = value.titleLine2Suffix
  form.description = value.description
  form.mottoText = value.mottoText
}

async function fetchData() {
  loading.value = true
  try {
    const list = await listWebsiteInfo()
    const item = (list || []).find((i) => i.key === 'theme_extend_info')
    const raw = isRecord(item?.infoJson) ? cloneRecord(item.infoJson) : {}
    const homeRoot = isRecord(raw.home) ? raw.home : raw
    themeExtendInfo.value = raw
    assignForm({
      ...defaultForm,
      ...normalizeFirstViewport(homeRoot.firstViewport),
      ...normalizeHero(homeRoot.hero),
    })
    originalSnapshot.value = serializeForm(form)
  } catch (err) {
    message.error(err instanceof Error ? err.message : '加载失败')
  } finally {
    loading.value = false
  }
}

function buildTitleTemplate(): JsonRecord[] {
  return [
    { type: 'span', text: form.titleLine1Prefix },
    { type: 'span', text: form.titleLine1Highlight, variant: 'hero_h1_highlight' },
    { type: 'span', text: form.titleLine1Suffix },
    { type: 'br' },
    { type: 'span', text: form.titleLine2Prefix },
    { type: 'code', text: form.titleCode, variant: 'hero_code_inline' },
    { type: 'span', text: form.titleLine2Suffix, variant: 'hero_cursor' },
  ]
}

function buildMottoLines(): string[] {
  return form.mottoText
    .split('\n')
    .map((item) => item.trim())
    .filter(Boolean)
}

function buildNextThemeExtendInfo(): JsonRecord {
  const next = cloneRecord(themeExtendInfo.value)
  const home = isRecord(next.home) ? { ...next.home } : {}
  const hero = isRecord(home.hero) ? { ...home.hero } : {}

  home.firstViewport = {
    variant: form.variant,
    hideGlobalSidebarOnHome: form.hideGlobalSidebarOnHome,
    hideGlobalMobileNavOnHome: form.hideGlobalMobileNavOnHome,
    showTopNav: form.showTopNav,
    showScrollHint: form.showScrollHint,
  }
  hero.avatarUrl = form.avatarUrl.trim()
  hero.description = form.description.trim()
  hero.titleTemplate = buildTitleTemplate()
  hero.mottoLines = buildMottoLines()
  home.hero = hero
  next.home = home

  return next
}

async function handleSave() {
  if (saving.value) return
  if (!isDirty.value) {
    message.warning('没有检测到更改')
    return
  }

  saving.value = true
  try {
    const next = buildNextThemeExtendInfo()
    await updateWebsiteInfo('theme_extend_info', { infoJson: next })
    themeExtendInfo.value = next
    originalSnapshot.value = serializeForm(form)
    message.success('保存成功')
  } catch (err) {
    message.error(err instanceof Error ? err.message : '保存失败')
  } finally {
    saving.value = false
  }
}

function applyYohakuPreset() {
  assignForm({
    variant: 'yohaku',
    hideGlobalSidebarOnHome: true,
    hideGlobalMobileNavOnHome: true,
    showTopNav: true,
    showScrollHint: true,
    avatarUrl: form.avatarUrl,
    titleLine1Prefix: form.titleLine1Prefix,
    titleLine1Highlight: form.titleLine1Highlight,
    titleLine1Suffix: form.titleLine1Suffix,
    titleLine2Prefix: form.titleLine2Prefix,
    titleCode: form.titleCode,
    titleLine2Suffix: form.titleLine2Suffix,
    description: form.description,
    mottoText: form.mottoText,
  })
}

function applyDefaultPreset() {
  assignForm({ ...defaultForm })
}

onMounted(fetchData)
</script>

<template>
  <NCard>
    <template #header>
      <div class="flex flex-wrap items-center justify-between gap-3">
        <div>
          <div class="text-base font-semibold">首页首屏</div>
          <div class="text-xs text-neutral-500">写入 theme_extend_info.home.firstViewport</div>
        </div>
        <div class="flex items-center gap-2">
          <NTag
            v-if="isDirty"
            type="warning"
          >
            未保存
          </NTag>
          <NButton
            size="small"
            secondary
            :loading="loading"
            @click="fetchData"
          >
            刷新
          </NButton>
          <NButton
            size="small"
            type="primary"
            :loading="saving"
            :disabled="!isDirty"
            @click="handleSave"
          >
            保存
          </NButton>
        </div>
      </div>
    </template>

    <NSpin :show="loading">
      <div class="space-y-5">
        <NAlert
          type="info"
          :show-icon="false"
        >
          这里只控制首页首屏样式；头像、标题、描述、motto 和社交链接继续共用主题扩展里的 home.hero
          配置。
        </NAlert>

        <div class="flex flex-wrap gap-2">
          <NButton
            size="small"
            tertiary
            @click="applyYohakuPreset"
          >
            使用 Yohaku 首屏
          </NButton>
          <NButton
            size="small"
            tertiary
            @click="applyDefaultPreset"
          >
            恢复 GrtBlog 首屏
          </NButton>
        </div>

        <NForm
          label-placement="left"
          :label-width="180"
          class="max-w-2xl"
        >
          <NFormItem label="首屏样式">
            <NSelect
              v-model:value="form.variant"
              :options="variantOptions"
            />
          </NFormItem>

          <NFormItem label="隐藏桌面侧栏">
            <NSwitch
              v-model:value="form.hideGlobalSidebarOnHome"
              :disabled="!isYohaku"
            />
          </NFormItem>

          <NFormItem label="隐藏移动端导航">
            <NSwitch
              v-model:value="form.hideGlobalMobileNavOnHome"
              :disabled="!isYohaku"
            />
          </NFormItem>

          <NFormItem label="显示首屏顶部导航">
            <NSwitch
              v-model:value="form.showTopNav"
              :disabled="!isYohaku"
            />
          </NFormItem>

          <NFormItem label="显示滚动提示">
            <NSwitch
              v-model:value="form.showScrollHint"
              :disabled="!isYohaku"
            />
          </NFormItem>

          <div
            class="mt-6 mb-3 border-t border-neutral-200 pt-5 text-sm font-semibold dark:border-neutral-700"
          >
            首屏内容
          </div>

          <NFormItem label="头像 URL">
            <NInput
              v-model:value="form.avatarUrl"
              placeholder="https://..."
            />
          </NFormItem>

          <NFormItem label="第一行前缀">
            <NInput
              v-model:value="form.titleLine1Prefix"
              placeholder="Hi, I'm "
            />
          </NFormItem>

          <NFormItem label="第一行高亮">
            <NInput
              v-model:value="form.titleLine1Highlight"
              placeholder="你的名字"
            />
          </NFormItem>

          <NFormItem label="第一行后缀">
            <NInput
              v-model:value="form.titleLine1Suffix"
              placeholder=" 👋"
            />
          </NFormItem>

          <NFormItem label="第二行前缀">
            <NInput
              v-model:value="form.titleLine2Prefix"
              placeholder="A Full Stack "
            />
          </NFormItem>

          <NFormItem label="代码标签">
            <NInput
              v-model:value="form.titleCode"
              placeholder="<Developer />"
            />
          </NFormItem>

          <NFormItem label="第二行后缀">
            <NInput
              v-model:value="form.titleLine2Suffix"
              placeholder=" |"
            />
          </NFormItem>

          <NFormItem label="副标题">
            <NInput
              v-model:value="form.description"
              placeholder="An independent developer coding with love."
            />
          </NFormItem>

          <NFormItem label="底部小字">
            <NInput
              v-model:value="form.mottoText"
              type="textarea"
              :autosize="{ minRows: 2, maxRows: 5 }"
              placeholder="每行一条，留空会回退默认配置"
            />
          </NFormItem>
        </NForm>
      </div>
    </NSpin>
  </NCard>
</template>
