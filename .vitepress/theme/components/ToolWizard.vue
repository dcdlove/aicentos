<script setup lang="ts">
import { ref, computed } from 'vue'
import { useData } from 'vitepress'

type Scenario = 'cli' | 'vscode' | 'both'
type Model = 'claude' | 'gpt' | 'any'
type Trait = 'ootb' | 'customizable' | 'lightweight'

interface I18nText {
  title: string
  steps: string[]
  scenarioLabel: string
  scenarios: Record<Scenario, string>
  modelLabel: string
  models: Record<Model, string>
  traitLabel: string
  traits: Record<Trait, string>
  resultPrefix: string
  viewGuide: string
  restart: string
}

const I18N: Record<string, I18nText> = {
  'zh-CN': {
    title: '找到最适合你的 AI Coding 工具',
    steps: ['使用场景', '偏好模型', '使用特点'],
    scenarioLabel: '你主要在哪里写代码？',
    scenarios: { cli: 'CLI 终端', vscode: 'VS Code 编辑器', both: '两者都要' },
    modelLabel: '你偏好哪个 AI 模型？',
    models: { claude: 'Claude', gpt: 'GPT', any: '不限' },
    traitLabel: '你最看重什么？',
    traits: { ootb: '开箱即用', customizable: '高度可定制', lightweight: '轻量快速' },
    resultPrefix: '推荐工具',
    viewGuide: '查看配置指南',
    restart: '重新选择',
  },
  'en-US': {
    title: 'Find the Best AI Coding Tool for You',
    steps: ['Scenario', 'Model', 'Preference'],
    scenarioLabel: 'Where do you mainly write code?',
    scenarios: { cli: 'CLI Terminal', vscode: 'VS Code Editor', both: 'Both' },
    modelLabel: 'Which AI model do you prefer?',
    models: { claude: 'Claude', gpt: 'GPT', any: 'Any' },
    traitLabel: 'What matters most to you?',
    traits: { ootb: 'Out of the box', customizable: 'Highly customizable', lightweight: 'Lightweight & fast' },
    resultPrefix: 'Recommended Tool',
    viewGuide: 'View Setup Guide',
    restart: 'Start Over',
  },
  'fr-FR': {
    title: 'Trouvez l\'outil AI Coding idéal',
    steps: ['Scénario', 'Modèle', 'Préférence'],
    scenarioLabel: 'Où écrivez-vous principalement du code ?',
    scenarios: { cli: 'Terminal CLI', vscode: 'Éditeur VS Code', both: 'Les deux' },
    modelLabel: 'Quel modèle IA préférez-vous ?',
    models: { claude: 'Claude', gpt: 'GPT', any: 'Peu importe' },
    traitLabel: 'Qu\'est-ce qui compte le plus ?',
    traits: { ootb: 'Prêt à l\'emploi', customizable: 'Personnalisable', lightweight: 'Léger et rapide' },
    resultPrefix: 'Outil recommandé',
    viewGuide: 'Voir le guide',
    restart: 'Recommencer',
  },
  'es-ES': {
    title: 'Encuentra la herramienta AI Coding ideal',
    steps: ['Escenario', 'Modelo', 'Preferencia'],
    scenarioLabel: '¿Dónde escribes código principalmente?',
    scenarios: { cli: 'Terminal CLI', vscode: 'Editor VS Code', both: 'Ambos' },
    modelLabel: '¿Qué modelo de IA prefieres?',
    models: { claude: 'Claude', gpt: 'GPT', any: 'Cualquiera' },
    traitLabel: '¿Qué es lo más importante para ti?',
    traits: { ootb: 'Listo para usar', customizable: 'Altamente personalizable', lightweight: 'Ligero y rápido' },
    resultPrefix: 'Herramienta recomendada',
    viewGuide: 'Ver guía',
    restart: 'Reiniciar',
  },
  'pt-BR': {
    title: 'Encontre a ferramenta AI Coding ideal',
    steps: ['Cenário', 'Modelo', 'Preferência'],
    scenarioLabel: 'Onde você escreve código principalmente?',
    scenarios: { cli: 'Terminal CLI', vscode: 'Editor VS Code', both: 'Ambos' },
    modelLabel: 'Qual modelo de IA você prefere?',
    models: { claude: 'Claude', gpt: 'GPT', any: 'Qualquer um' },
    traitLabel: 'O que é mais importante para você?',
    traits: { ootb: 'Pronto para usar', customizable: 'Altamente personalizável', lightweight: 'Leve e rápido' },
    resultPrefix: 'Ferramenta recomendada',
    viewGuide: 'Ver guia',
    restart: 'Recomeçar',
  },
}

const LANG_PREFIX: Record<string, string> = {
  'zh-CN': '', 'en-US': '/en', 'fr-FR': '/fr', 'es-ES': '/es', 'pt-BR': '/pt',
}

const RECOMMENDATIONS: Record<string, { name: string; path: string }> = {
  'cli|claude':   { name: 'Claude Code',   path: '/start' },
  'cli|gpt':      { name: 'OpenAI Codex',  path: '/codex' },
  'cli|any':      { name: 'Claude Code',   path: '/start' },
  'both|claude':  { name: 'Claude Code',   path: '/start' },
  'both|gpt':     { name: 'OpenAI Codex',  path: '/codex' },
  'both|any':     { name: 'Claude Code',   path: '/start' },
  'vscode':       { name: 'RooCode',       path: '/roocode' },
  'lightweight':  { name: 'Droid CLI',     path: '/droid' },
  'customizable': { name: 'OpenCode',      path: '/opencode' },
}

const { lang } = useData()
const t = computed(() => I18N[lang.value] ?? I18N['zh-CN'])
const prefix = computed(() => LANG_PREFIX[lang.value] ?? '')

const step = ref(0)
const scenario = ref<Scenario | ''>('')
const model = ref<Model | ''>('')
const trait = ref<Trait | ''>('')

const result = computed(() => {
  let rec: { name: string; path: string }
  if (scenario.value === 'vscode')
    rec = RECOMMENDATIONS['vscode']
  else if (trait.value === 'lightweight')
    rec = RECOMMENDATIONS['lightweight']
  else if (trait.value === 'customizable')
    rec = RECOMMENDATIONS['customizable']
  else {
    const key = `${scenario.value}|${model.value}`
    rec = RECOMMENDATIONS[key] ?? RECOMMENDATIONS['cli|claude']
  }
  return { name: rec.name, link: `${prefix.value}${rec.path}` }
})

function selectScenario(val: Scenario) {
  scenario.value = val
  if (val === 'vscode') { step.value = 3; return }
  step.value = 1
}

function selectModel(val: Model) {
  model.value = val
  step.value = 2
}

function selectTrait(val: Trait) {
  trait.value = val
  step.value = 3
}

function restart() {
  step.value = 0
  scenario.value = ''
  model.value = ''
  trait.value = ''
}
</script>

<template>
  <div class="wizard">
    <h3 class="wizard-title">{{ t.title }}</h3>

    <div class="steps-indicator">
      <template v-for="(label, i) in t.steps" :key="i">
        <div class="step-dot" :class="{ active: step >= i, done: step > i }">
          <span class="dot">{{ step > i ? '&#10003;' : i + 1 }}</span>
          <span class="step-label">{{ label }}</span>
        </div>
        <div v-if="i < t.steps.length - 1" class="step-line" :class="{ active: step > i }" />
      </template>
    </div>

    <Transition name="fade" mode="out-in">
      <div v-if="step === 0" key="s0" class="options-panel">
        <p class="options-label">{{ t.scenarioLabel }}</p>
        <div class="options-grid">
          <button
            v-for="(label, key) in t.scenarios" :key="key"
            class="option-card" :class="{ selected: scenario === key }"
            @click="selectScenario(key as Scenario)"
          >{{ label }}</button>
        </div>
      </div>

      <div v-else-if="step === 1" key="s1" class="options-panel">
        <p class="options-label">{{ t.modelLabel }}</p>
        <div class="options-grid">
          <button
            v-for="(label, key) in t.models" :key="key"
            class="option-card" :class="{ selected: model === key }"
            @click="selectModel(key as Model)"
          >{{ label }}</button>
        </div>
      </div>

      <div v-else-if="step === 2" key="s2" class="options-panel">
        <p class="options-label">{{ t.traitLabel }}</p>
        <div class="options-grid">
          <button
            v-for="(label, key) in t.traits" :key="key"
            class="option-card" :class="{ selected: trait === key }"
            @click="selectTrait(key as Trait)"
          >{{ label }}</button>
        </div>
      </div>

      <div v-else key="result" class="result-panel">
        <p class="result-label">{{ t.resultPrefix }}</p>
        <p class="result-name">{{ result.name }}</p>
        <div class="result-actions">
          <a class="btn-primary" :href="result.link">{{ t.viewGuide }}</a>
          <button class="btn-secondary" @click="restart">{{ t.restart }}</button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.wizard {
  max-width: 560px;
  margin: 2rem auto;
  padding: 2rem;
  border-radius: 12px;
  background: var(--vp-c-bg);
  border: 1px solid var(--vp-c-divider);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
}

.wizard-title {
  margin: 0 0 1.5rem;
  text-align: center;
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--vp-c-text-1);
}

.steps-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
  margin-bottom: 1.75rem;
}

.step-dot {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
}

.dot {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  font-weight: 600;
  border: 2px solid var(--vp-c-divider);
  color: var(--vp-c-text-3);
  background: var(--vp-c-bg);
  transition: all 0.3s;
}

.step-dot.active .dot {
  border-color: var(--vp-c-brand-1);
  color: var(--vp-c-brand-1);
}

.step-dot.done .dot {
  background: var(--vp-c-brand-1);
  border-color: var(--vp-c-brand-1);
  color: #fff;
}

.step-label {
  font-size: 0.75rem;
  color: var(--vp-c-text-3);
  white-space: nowrap;
}

.step-dot.active .step-label {
  color: var(--vp-c-text-1);
}

.step-line {
  width: 48px;
  height: 2px;
  background: var(--vp-c-divider);
  margin: 0 8px;
  margin-bottom: 22px;
  transition: background 0.3s;
}

.step-line.active {
  background: var(--vp-c-brand-1);
}

.options-panel {
  text-align: center;
}

.options-label {
  font-size: 1rem;
  font-weight: 500;
  color: var(--vp-c-text-1);
  margin: 0 0 1rem;
}

.options-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 10px;
}

.option-card {
  padding: 0.75rem 1rem;
  border-radius: 8px;
  border: 1px solid var(--vp-c-divider);
  background: var(--vp-c-bg-soft);
  color: var(--vp-c-text-1);
  font-size: 0.9rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.option-card:hover {
  border-color: var(--vp-c-brand-2);
  background: var(--vp-c-brand-soft);
}

.option-card.selected {
  border-color: var(--vp-c-brand-1);
  background: var(--vp-c-brand-soft);
  color: var(--vp-c-brand-1);
}

.result-panel {
  text-align: center;
}

.result-label {
  font-size: 0.9rem;
  color: var(--vp-c-text-2);
  margin: 0 0 0.5rem;
}

.result-name {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--vp-c-brand-1);
  margin: 0 0 1.5rem;
}

.result-actions {
  display: flex;
  justify-content: center;
  gap: 12px;
  flex-wrap: wrap;
}

.btn-primary {
  display: inline-block;
  padding: 0.5rem 1.25rem;
  border-radius: 8px;
  font-weight: 500;
  font-size: 0.9rem;
  color: #fff;
  background: var(--vp-c-brand-1);
  text-decoration: none;
  transition: background 0.25s;
}

.btn-primary:hover {
  background: var(--vp-c-brand-2);
}

.btn-secondary {
  padding: 0.5rem 1.25rem;
  border-radius: 8px;
  font-weight: 500;
  font-size: 0.9rem;
  color: var(--vp-c-text-1);
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-divider);
  cursor: pointer;
  transition: border-color 0.25s;
}

.btn-secondary:hover {
  border-color: var(--vp-c-brand-2);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@media (max-width: 480px) {
  .wizard { padding: 1.25rem; }
  .options-grid { grid-template-columns: 1fr; }
  .step-line { width: 24px; }
}
</style>
