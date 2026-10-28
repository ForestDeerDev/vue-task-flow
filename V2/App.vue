<script setup>
import { computed, reactive, ref } from 'vue'

const categories = ['学习', '工作', '生活', '娱乐']
const categoryMeta = {
  学习: { color: '#7c6cff', icon: 'M4 19.5A2.5 2.5 0 0 1 6.5 17H20M4 19.5A2.5 2.5 0 0 0 6.5 22H20V2H6.5A2.5 2.5 0 0 0 4 4.5v15z' },
  工作: { color: '#38bdf8', icon: 'M20 7h-9m9 0a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V9a2 2 0 0 1 2-2h16zM16 7V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v2' },
  生活: { color: '#34d399', icon: 'M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2zM9 22V12h6v10' },
  娱乐: { color: '#fb7185', icon: 'M12 3l1.9 5.8H20l-4.9 3.6 1.9 5.8-5-3.6-5 3.6 1.9-5.8L4 8.8h6.1z' },
}
const priorities = ['高', '中', '低']
let nextId = 5
const tasks = reactive([
  { id: 1, name: '完成 Vue 任务管理作业', category: '学习', priority: '高', date: '2026-10-27', note: '完成表单、筛选和统计功能。', done: false },
  { id: 2, name: '整理本周学习计划', category: '学习', priority: '中', date: '2026-10-28', note: '为每天安排明确的目标。', done: true },
  { id: 3, name: '清理桌面与下载文件夹', category: '生活', priority: '低', date: '', note: '删除重复文件，整理项目目录。', done: false },
  { id: 4, name: '看一集纪录片放松一下', category: '娱乐', priority: '低', date: '2026-10-30', note: '完成学习后再看。', done: false },
])
const form = reactive({ name: '', category: '学习', priority: '中', date: '', note: '' })
const editingId = ref(null)
const keyword = ref('')
const categoryFilter = ref('全部')
const showPending = ref(true)
const showDone = ref(true)
const confirmClear = ref(false)
const toast = ref('')
let toastTimer

// computed 根据筛选条件自动更新任务列表。
const filteredTasks = computed(() => {
  const search = keyword.value.trim().toLocaleLowerCase()
  return tasks.filter((task) => {
    const matchesName = task.name.toLocaleLowerCase().includes(search)
    const matchesCategory = categoryFilter.value === '全部' || task.category === categoryFilter.value
    const matchesStatus = (task.done && showDone.value) || (!task.done && showPending.value)
    return matchesName && matchesCategory && matchesStatus
  })
})
const completedCount = computed(() => tasks.filter((task) => task.done).length)
const pendingCount = computed(() => tasks.length - completedCount.value)
const progress = computed(() => (tasks.length ? completedCount.value / tasks.length : 0))
const categoryCounts = computed(() => Object.fromEntries(
  categories.map((category) => [category, tasks.filter((task) => task.category === category).length]),
))
const today = new Date().toLocaleDateString('zh-CN', { month: 'long', day: 'numeric', weekday: 'long' })

function notify(message) {
  toast.value = message
  clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { toast.value = '' }, 2200)
}
function resetForm() {
  Object.assign(form, { name: '', category: '学习', priority: '中', date: '', note: '' })
  editingId.value = null
}
// .prevent 阻止表单提交时刷新页面；同一方法处理新建和编辑。
function submitTask() {
  const name = form.name.trim()
  if (!name) return
  const taskData = { name, category: form.category, priority: form.priority, date: form.date, note: form.note.trim() }
  if (editingId.value !== null) {
    const task = tasks.find((item) => item.id === editingId.value)
    if (task) Object.assign(task, taskData)
    notify('任务已更新')
  } else {
    tasks.unshift({ id: nextId++, ...taskData, done: false })
    notify('任务添加成功')
  }
  resetForm()
}
function editTask(task) {
  Object.assign(form, { name: task.name, category: task.category, priority: task.priority, date: task.date, note: task.note })
  editingId.value = task.id
  window.scrollTo({ top: 0, behavior: 'smooth' })
}
function deleteTask(id) {
  const index = tasks.findIndex((task) => task.id === id)
  if (index !== -1) tasks.splice(index, 1)
  if (editingId.value === id) resetForm()
  notify('任务已删除')
}
function clearCompleted() {
  const completedIds = tasks.filter((task) => task.done).map((task) => task.id)
  for (let index = tasks.length - 1; index >= 0; index -= 1) {
    if (tasks[index].done) tasks.splice(index, 1)
  }
  if (completedIds.includes(editingId.value)) resetForm()
  confirmClear.value = false
  notify(`已清除 ${completedIds.length} 项已完成任务`)
}
</script>

<template>
  <main class="app-shell">
    <div class="aurora aurora-a"></div>
    <div class="aurora aurora-b"></div>
    <div class="aurora aurora-c"></div>
    <div class="grain"></div>

    <div class="page-wrap">
      <header class="page-header">
        <div class="brand">
          <div class="logo-mark">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/></svg>
          </div>
          <div>
            <div class="eyebrow">PERSONAL WORKSPACE</div>
            <h1>把每件事，<em>安排妥当。</em></h1>
            <p class="subtitle">清晰的任务列表，让今天更有条理。</p>
          </div>
        </div>
        <div class="date-pill">
          <span class="date-dot"></span>{{ today }}
        </div>
      </header>

      <div class="main-layout">
        <section class="panel form-panel">
          <div class="panel-glow"></div>
          <h2>{{ editingId !== null ? '编辑任务' : '新建任务' }}</h2>
          <p class="panel-hint">补充任务信息，开始规划。</p>
          <form @submit.prevent="submitTask">
            <div class="field">
              <label for="task-name">任务名称 <span class="required">*</span></label>
              <input id="task-name" v-model.trim="form.name" maxlength="100" placeholder="例如：完成 Vue 课程作业" required />
            </div>
            <div class="field">
              <label for="task-category">任务分类</label>
              <div class="category-picker">
                <button
                  v-for="category in categories"
                  :key="category"
                  type="button"
                  class="category-option"
                  :class="{ active: form.category === category }"
                  :style="{ '--cat-color': categoryMeta[category].color }"
                  @click="form.category = category"
                >
                  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path :d="categoryMeta[category].icon"/></svg>
                  {{ category }}
                </button>
              </div>
            </div>
            <fieldset class="field priority-field">
              <legend>任务优先级</legend>
              <div class="priority-options">
                <label v-for="priority in priorities" :key="priority" class="radio-option" :class="[`is-${priority}`, { active: form.priority === priority }]">
                  <input v-model="form.priority" type="radio" name="priority" :value="priority" />
                  <span class="priority-dot"></span>{{ priority }}
                </label>
              </div>
            </fieldset>
            <div class="field">
              <label for="task-date">截止日期</label>
              <input id="task-date" v-model="form.date" type="date" />
            </div>
            <div class="field">
              <label for="task-note">备注说明</label>
              <textarea id="task-note" v-model.trim="form.note" maxlength="500" placeholder="补充任务细节（选填）" rows="3"></textarea>
            </div>
            <button class="primary-button full-button" type="submit">
              <svg v-if="editingId === null" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round"><path d="M12 5v14M5 12h14"/></svg>
              {{ editingId !== null ? '保存修改' : '添加任务' }}
            </button>
            <button v-if="editingId !== null" class="outline-button full-button cancel-button" type="button" @click="resetForm">取消编辑</button>
          </form>
        </section>

        <section class="content-column">
          <div class="panel list-panel">
            <div class="section-heading">
              <div><h2>我的任务</h2><p class="panel-hint no-bottom">所有待办，一目了然。</p></div>
              <!-- .stop 防止点击事件冒泡到外层容器；实际清除前还会二次确认。 -->
              <button class="outline-button" type="button" @click.stop="confirmClear = true">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 6h18M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2m3 0v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6"/></svg>
                清除已完成
              </button>
            </div>
            <div class="filter-grid">
              <div class="search-wrap">
                <svg class="search-symbol" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><circle cx="11" cy="11" r="7"/><path d="M21 21l-4.35-4.35"/></svg>
                <input v-model.trim="keyword" aria-label="关键词搜索" placeholder="搜索任务名称…" />
              </div>
              <select v-model="categoryFilter" aria-label="按分类筛选">
                <option value="全部">全部分类</option>
                <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
              </select>
            </div>
            <div class="status-filters">
              <label class="switch-label">
                <input v-model="showPending" type="checkbox" />
                <span class="switch-track"><span class="switch-thumb"></span></span>
                未完成
              </label>
              <label class="switch-label">
                <input v-model="showDone" type="checkbox" />
                <span class="switch-track"><span class="switch-thumb"></span></span>
                已完成
              </label>
              <span class="list-caption">显示 {{ filteredTasks.length }} / {{ tasks.length }} 项任务</span>
            </div>

            <TransitionGroup v-if="filteredTasks.length" name="list" tag="div" class="task-list">
              <article v-for="task in filteredTasks" :key="task.id" class="task-card" :class="[`priority-${task.priority}`, { completed: task.done }]">
                <label class="task-check" :aria-label="`标记 ${task.name} 为完成`">
                  <input v-model="task.done" type="checkbox" />
                  <span class="check-box">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3.2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6L9 17l-5-5"/></svg>
                  </span>
                </label>
                <div class="task-content">
                  <h3 :class="{ 'task-done': task.done }">{{ task.name }}</h3>
                  <div class="chips">
                    <span class="chip category-chip" :style="{ '--cat-color': categoryMeta[task.category].color }">
                      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path :d="categoryMeta[task.category].icon"/></svg>
                      {{ task.category }}
                    </span>
                    <span class="chip" :class="`priority-chip-${task.priority}`">{{ task.priority }}优先级</span>
                    <span v-if="task.date" class="chip date-chip">
                      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="18" height="18" rx="2"/><path d="M16 2v4M8 2v4M3 10h18"/></svg>
                      {{ task.date }}
                    </span>
                  </div>
                  <p v-if="task.note" class="task-note">{{ task.note }}</p>
                </div>
                <div class="task-actions">
                  <button class="icon-button" type="button" aria-label="编辑任务" @click="editTask(task)">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.12 2.12 0 0 1 3 3L12 15l-4 1 1-4z"/></svg>
                  </button>
                  <button class="icon-button danger" type="button" aria-label="删除任务" @click="deleteTask(task.id)">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9" stroke-linecap="round" stroke-linejoin="round"><path d="M3 6h18M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2m3 0v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6"/></svg>
                  </button>
                </div>
              </article>
            </TransitionGroup>
            <!-- v-if / v-else：没有任务匹配筛选条件时展示空状态。 -->
            <div v-else class="empty-state">
              <div class="empty-icon">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.4" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M8 15h8M9 9h.01M15 9h.01"/></svg>
              </div>
              <strong>暂无匹配任务</strong>
              <p>试试更换关键词或调整筛选条件。</p>
            </div>
          </div>

          <section class="panel stats-panel">
            <div class="section-heading stats-heading">
              <div><h2>任务概览</h2><p class="panel-hint no-bottom">进度与分类分布实时更新。</p></div>
              <span class="live-label"><span class="live-dot"></span>LIVE</span>
            </div>
            <div class="stats-body">
              <div class="progress-ring-wrap">
                <svg class="progress-ring" viewBox="0 0 120 120">
                  <defs>
                    <linearGradient id="ringGradient" x1="0%" y1="0%" x2="100%" y2="100%">
                      <stop offset="0%" stop-color="#7c6cff"/>
                      <stop offset="100%" stop-color="#38bdf8"/>
                    </linearGradient>
                  </defs>
                  <circle class="ring-track" cx="60" cy="60" r="52"/>
                  <circle class="ring-value" cx="60" cy="60" r="52" :stroke-dasharray="326.7" :stroke-dashoffset="326.7 * (1 - progress)"/>
                </svg>
                <div class="ring-text">
                  <strong>{{ Math.round(progress * 100) }}<small>%</small></strong>
                  <span>完成率</span>
                </div>
              </div>
              <div class="stats-grid">
                <div class="stat-card"><div>任务总数</div><strong>{{ tasks.length }}</strong></div>
                <div class="stat-card"><div>已完成</div><strong class="done-color">{{ completedCount }} <small>项</small></strong></div>
                <div class="stat-card"><div>未完成</div><strong class="pending-color">{{ pendingCount }} <small>项</small></strong></div>
              </div>
            </div>
            <div class="category-distribution">
              <span v-for="category in categories" :key="category" class="distribution-chip" :style="{ '--cat-color': categoryMeta[category].color }">
                <span class="distribution-dot"></span>{{ category }} <strong>{{ categoryCounts[category] }}</strong>
              </span>
            </div>
            <p class="footer-note">每完成一项，都是向前一步。</p>
          </section>
        </section>
      </div>
    </div>

    <Transition name="modal">
      <div v-if="confirmClear" class="modal-backdrop" @click.self="confirmClear = false">
        <section class="confirm-dialog" role="dialog" aria-modal="true" aria-labelledby="confirm-title">
          <div class="dialog-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 6h18M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2m3 0v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6"/></svg>
          </div>
          <h2 id="confirm-title">清除已完成任务？</h2>
          <p>此操作会移除所有已勾选完成的任务，且无法撤销。</p>
          <div class="dialog-actions">
            <button class="outline-button" type="button" @click="confirmClear = false">再想想</button>
            <button class="delete-button" type="button" @click="clearCompleted">确认清除</button>
          </div>
        </section>
      </div>
    </Transition>
    <Transition name="toast">
      <div v-if="toast" class="toast-message" role="status">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6L9 17l-5-5"/></svg>
        {{ toast }}
      </div>
    </Transition>
  </main>
</template>
