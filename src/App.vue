<script setup>
import { computed, reactive, ref } from 'vue'

const categories = ['学习', '工作', '生活', '娱乐']
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
const categoryCounts = computed(() => Object.fromEntries(
  categories.map((category) => [category, tasks.filter((task) => task.category === category).length]),
))

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
    <div class="page-wrap">
      <header class="page-header">
        <div>
          <div class="eyebrow">PERSONAL WORKSPACE · 01</div>
          <h1>把每件事，安排妥当。</h1>
          <p class="subtitle">清晰的任务列表，让今天更有条理。</p>
        </div>
        <div class="date-pill">{{ new Date().toLocaleDateString('zh-CN', { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' }) }}</div>
      </header>

      <div class="main-layout">
        <section class="panel form-panel">
          <h2>{{ editingId !== null ? '编辑任务' : '新建任务' }}</h2>
          <p class="panel-hint">补充任务信息，开始规划。</p>
          <form @submit.prevent="submitTask">
            <div class="field">
              <label for="task-name">任务名称 <span class="required">*</span></label>
              <input id="task-name" v-model.trim="form.name" maxlength="100" placeholder="例如：完成 Vue 课程作业" required />
            </div>
            <div class="field">
              <label for="task-category">任务分类</label>
              <select id="task-category" v-model="form.category">
                <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
              </select>
            </div>
            <fieldset class="field priority-field">
              <legend>任务优先级</legend>
              <div class="priority-options">
                <label v-for="priority in priorities" :key="priority" class="radio-option">
                  <input v-model="form.priority" type="radio" name="priority" :value="priority" />
                  <span>{{ priority === '高' ? '🔴' : priority === '中' ? '🟠' : '🟢' }} {{ priority }}</span>
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
            <button class="primary-button full-button" type="submit">{{ editingId !== null ? '保存修改' : '＋ 添加任务' }}</button>
            <button v-if="editingId !== null" class="outline-button full-button cancel-button" type="button" @click="resetForm">取消编辑</button>
          </form>
        </section>

        <section class="content-column">
          <div class="panel">
            <div class="section-heading">
              <div><h2>我的任务</h2><p class="panel-hint no-bottom">所有待办，一目了然。</p></div>
              <!-- .stop 防止点击事件冒泡到外层容器；实际清除前还会二次确认。 -->
              <button class="outline-button" type="button" @click.stop="confirmClear = true">清除已完成</button>
            </div>
            <div class="filter-grid">
              <div class="search-wrap"><span class="search-symbol">⌕</span><input v-model.trim="keyword" aria-label="关键词搜索" placeholder="搜索任务名称…" /></div>
              <select v-model="categoryFilter" aria-label="按分类筛选">
                <option value="全部">全部分类</option>
                <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
              </select>
            </div>
            <div class="status-filters">
              <label><input v-model="showPending" type="checkbox" /> 未完成</label>
              <label><input v-model="showDone" type="checkbox" /> 已完成</label>
            </div>
            <div class="list-caption"><span>显示 {{ filteredTasks.length }} / {{ tasks.length }} 项任务</span><span>按最近添加排序</span></div>

            <div v-if="filteredTasks.length" class="task-list">
              <article v-for="task in filteredTasks" :key="task.id" class="task-card" :class="[`priority-${task.priority}`, { completed: task.done }]">
                <input v-model="task.done" class="task-checkbox" type="checkbox" :aria-label="`标记 ${task.name} 为完成`" />
                <div class="task-content">
                  <h3 :class="{ 'task-done': task.done }">{{ task.name }}</h3>
                  <div class="chips">
                    <span class="chip category-chip">{{ task.category }}</span>
                    <span class="chip" :class="`priority-chip-${task.priority}`">{{ task.priority }}优先级</span>
                    <span class="chip category-chip">{{ task.done ? '✓ 已完成' : '◷ 未完成' }}</span>
                  </div>
                  <div class="task-details">
                    <div><span>截止日期</span>{{ task.date || '未设置' }}</div>
                    <div><span>备注</span>{{ task.note || '暂无备注' }}</div>
                  </div>
                </div>
                <div class="task-actions">
                  <button class="outline-button small-button" type="button" @click="editTask(task)">编辑</button>
                  <button class="delete-button small-button" type="button" @click="deleteTask(task.id)">删除</button>
                </div>
              </article>
            </div>
            <!-- v-if / v-else：没有任务匹配筛选条件时展示空状态。 -->
            <div v-else class="empty-state"><div class="empty-icon">✳</div><strong>暂无匹配任务</strong><p>试试更换关键词或调整筛选条件。</p></div>
          </div>

          <section class="panel stats-panel">
            <div class="section-heading stats-heading"><div><h2>任务概览</h2><p class="panel-hint no-bottom">进度与分类分布实时更新。</p></div><span class="live-label">LIVE STATS</span></div>
            <div class="stats-grid">
              <div class="stat-card"><div>任务总数</div><strong>{{ tasks.length }}</strong></div>
              <div class="stat-card"><div>已完成</div><strong>{{ completedCount }} <small>项</small></strong></div>
              <div class="stat-card"><div>未完成</div><strong>{{ pendingCount }} <small>项</small></strong></div>
            </div>
            <div class="category-distribution"><span v-for="category in categories" :key="category" class="distribution-chip">{{ category }} <strong>{{ categoryCounts[category] }}</strong></span></div>
            <p class="footer-note">每完成一项，都是向前一步。</p>
          </section>
        </section>
      </div>
    </div>

    <div v-if="confirmClear" class="modal-backdrop" @click.self="confirmClear = false">
      <section class="confirm-dialog" role="dialog" aria-modal="true" aria-labelledby="confirm-title">
        <h2 id="confirm-title">清除已完成任务？</h2>
        <p>此操作会移除所有已勾选完成的任务，且无法撤销。</p>
        <div class="dialog-actions"><button class="outline-button" type="button" @click="confirmClear = false">再想想</button><button class="delete-button" type="button" @click="clearCompleted">确认清除</button></div>
      </section>
    </div>
    <Transition name="toast"><div v-if="toast" class="toast-message" role="status">{{ toast }}</div></Transition>
  </main>
</template>
