
<script setup>
import { computed, reactive, ref } from 'vue'

const categories = ['学习', '工作', '生活', '娱乐']
const priorities = ['高', '中', '低']

let nextId = 5

const tasks = reactive([
  {
    id: 1,
    name: '完成 Vue 任务管理作业',
    category: '学习',
    priority: '高',
    date: '2026-10-27',
    note: '完成表单、筛选和统计功能。',
    done: false,
  },
  {
    id: 2,
    name: '整理本周学习计划',
    category: '学习',
    priority: '中',
    date: '2026-10-28',
    note: '为每天安排明确的目标。',
    done: true,
  },
  {
    id: 3,
    name: '清理桌面与下载文件夹',
    category: '生活',
    priority: '低',
    date: '',
    note: '删除重复文件，整理项目目录。',
    done: false,
  },
  {
    id: 4,
    name: '看一集纪录片放松一下',
    category: '娱乐',
    priority: '低',
    date: '2026-10-30',
    note: '完成学习后再看。',
    done: false,
  },
])

const form = reactive({
  name: '',
  category: '学习',
  priority: '中',
  date: '',
  note: '',
})

const editingId = ref(null)
const keyword = ref('')
const categoryFilter = ref('全部')
const showPending = ref(true)
const showDone = ref(true)
const confirmClear = ref(false)
const toast = ref('')
let toastTimer

const today = new Date()

const dayNumber = computed(() =>
  String(today.getDate()).padStart(2, '0'),
)

const monthYear = computed(() =>
  today
    .toLocaleDateString('en-US', {
      month: 'short',
      year: 'numeric',
    })
    .toUpperCase()
    .replace(' ', ' · '),
)

const filteredTasks = computed(() => {
  const search = keyword.value.trim().toLocaleLowerCase()

  return tasks.filter((task) => {
    const matchesKeyword = task.name
      .toLocaleLowerCase()
      .includes(search)

    const matchesCategory =
      categoryFilter.value === '全部' ||
      task.category === categoryFilter.value

    const matchesStatus = task.done
      ? showDone.value
      : showPending.value

    return matchesKeyword && matchesCategory && matchesStatus
  })
})

const completedCount = computed(
  () => tasks.filter((task) => task.done).length,
)

const pendingCount = computed(
  () => tasks.filter((task) => !task.done).length,
)

const progressPercent = computed(() =>
  tasks.length
    ? Math.round((completedCount.value / tasks.length) * 100)
    : 0,
)

const categoryCounts = computed(() =>
  categories.map((category) => {
    const count = tasks.filter(
      (task) => task.category === category,
    ).length

    return {
      name: category,
      count,
      percent: tasks.length
        ? Math.round((count / tasks.length) * 100)
        : 0,
    }
  }),
)

function notify(message) {
  toast.value = message
  clearTimeout(toastTimer)

  toastTimer = setTimeout(() => {
    toast.value = ''
  }, 2200)
}

function resetForm() {
  Object.assign(form, {
    name: '',
    category: '学习',
    priority: '中',
    date: '',
    note: '',
  })

  editingId.value = null
}

function submitTask() {
  const name = form.name.trim()

  if (!name) {
    notify('请输入任务名称')
    return
  }

  const data = {
    name,
    category: form.category,
    priority: form.priority,
    date: form.date,
    note: form.note.trim(),
  }

  if (editingId.value !== null) {
    const task = tasks.find(
      (item) => item.id === editingId.value,
    )

    if (task) {
      Object.assign(task, data)
      notify('任务已更新')
    }
  } else {
    tasks.unshift({
      id: nextId++,
      ...data,
      done: false,
    })

    notify('任务添加成功')
  }

  resetForm()
}

function editTask(task) {
  editingId.value = task.id

  Object.assign(form, {
    name: task.name,
    category: task.category,
    priority: task.priority,
    date: task.date,
    note: task.note,
  })

  window.scrollTo({
    top: 0,
    behavior: 'smooth',
  })
}

function cancelEdit() {
  resetForm()
}

function deleteTask(id) {
  const index = tasks.findIndex((task) => task.id === id)

  if (index === -1) return

  tasks.splice(index, 1)

  if (editingId.value === id) {
    resetForm()
  }

  notify('任务已删除')
}

function toggleTask(task) {
  task.done = !task.done
}

function clearCompleted() {
  const editingTaskIsCompleted = tasks.some(
    (task) => task.id === editingId.value && task.done,
  )

  for (let i = tasks.length - 1; i >= 0; i--) {
    if (tasks[i].done) {
      tasks.splice(i, 1)
    }
  }

  if (editingTaskIsCompleted) {
    resetForm()
  }

  confirmClear.value = false
  notify('已清除已完成任务')
}
</script>

<template>
  <main class="app">
    <header class="topbar">
      <div class="brand">
        <div class="brand-mark">✳</div>
        <span>DAYFORM / WORKSPACE</span>
      </div>

      <div class="top-meta">
        <span>MAKE ROOM FOR WHAT MATTERS</span>
        <div class="avatar">D</div>
      </div>
    </header>

    <div class="layout">
      <section class="main-col">
        <header class="hero">
          <div>
            <div class="eyebrow">MONDAY, YOUR WAY · 01</div>
            <h1>把重要的事，<br />慢慢做好。</h1>
            <p>不用一次完成所有事，只要开始下一步。</p>
          </div>

          <div class="date-stamp">
            <strong>{{ dayNumber }}</strong>
            <span>{{ monthYear }}</span>
          </div>
        </header>

        <section class="progress-strip">
          <div>
            <div class="progress-title">今天的进度</div>
            <div class="progress-sub">
              {{
                tasks.length
                  ? `${completedCount} / ${tasks.length} 项已完成，继续保持自己的节奏。`
                  : '先添加一项任务，开启今天。'
              }}
            </div>
          </div>

          <div class="progress-number">
            {{ progressPercent }}<small>%</small>
          </div>

          <div class="progress-track">
            <div
              class="progress-fill"
              :style="{ width: `${progressPercent}%` }"
            />
          </div>
        </section>

        <div class="section-head">
          <h2>
            任务清单
            <span class="task-total">
              {{ String(tasks.length).padStart(2, '0') }}
            </span>
          </h2>
          <small>按最近添加排序 ↘</small>
        </div>

        <div class="filters">
          <input
            v-model="keyword"
            type="search"
            placeholder="⌕ 搜索任务名称…"
            aria-label="搜索任务名称"
          />

          <select
            v-model="categoryFilter"
            aria-label="按分类筛选"
          >
            <option value="全部">全部分类</option>
            <option
              v-for="category in categories"
              :key="category"
              :value="category"
            >
              {{ category }}
            </option>
          </select>
        </div>

        <div class="status">
          <label>
            <input v-model="showPending" type="checkbox" />
            未完成
          </label>

          <label>
            <input v-model="showDone" type="checkbox" />
            已完成
          </label>

          <button
            class="mini clear-button"
            type="button"
            @click="confirmClear = true"
          >
            清除已完成
          </button>
        </div>

        <div v-if="filteredTasks.length" class="task-list">
          <article
            v-for="task in filteredTasks"
            :key="task.id"
            class="task"
            :class="[
              `priority-${task.priority}`,
              { done: task.done },
            ]"
          >
            <input
              type="checkbox"
              :checked="task.done"
              :aria-label="`标记 ${task.name} 为完成`"
              @change="toggleTask(task)"
            />

            <div class="task-content">
              <h3>{{ task.name }}</h3>

              <div class="chips">
                <span class="chip">{{ task.category }}</span>

                <span
                  class="chip"
                  :class="{
                    high: task.priority === '高',
                    medium: task.priority === '中',
                    low: task.priority === '低',
                  }"
                >
                  {{ task.priority }}优先级
                </span>

                <span class="chip">
                  {{ task.done ? '✓ 已完成' : '◷ 未完成' }}
                </span>
              </div>

              <div class="task-meta">
                <span>日期 · {{ task.date || '未设置' }}</span>
                <span>备注 · {{ task.note || '暂无备注' }}</span>
              </div>
            </div>

            <div class="task-actions">
              <button
                class="mini"
                type="button"
                @click="editTask(task)"
              >
                编辑
              </button>

              <button
                class="mini delete"
                type="button"
                @click="deleteTask(task.id)"
              >
                删除
              </button>
            </div>
          </article>
        </div>

        <div v-else class="empty">
          没有找到匹配的任务。
          <span>试试更换关键词或调整筛选条件。</span>
        </div>
      </section>

      <aside class="side">
        <section class="side-card form-card">
          <div class="eyebrow">CAPTURE A THOUGHT</div>

          <h2>{{ editingId !== null ? '编辑任务' : '新建任务' }}</h2>

          <p class="side-desc">
            把脑海中的待办放下来，专注眼前。
          </p>

          <form @submit.prevent="submitTask">
            <div class="field">
              <label for="task-name">
                任务名称 <span class="required">*</span>
              </label>

              <input
                id="task-name"
                v-model="form.name"
                maxlength="100"
                placeholder="例如：完成 Vue 课程作业"
                required
              />
            </div>

            <div class="field">
              <label for="task-category">任务分类</label>

              <select id="task-category" v-model="form.category">
                <option
                  v-for="category in categories"
                  :key="category"
                  :value="category"
                >
                  {{ category }}
                </option>
              </select>
            </div>

            <fieldset class="field">
              <legend>任务优先级</legend>

              <div class="priority-options">
                <label
                  v-for="priority in priorities"
                  :key="priority"
                  :class="{ selected: form.priority === priority }"
                >
                  <input
                    v-model="form.priority"
                    type="radio"
                    name="priority"
                    :value="priority"
                  />
                  {{ priority }}
                </label>
              </div>
            </fieldset>

            <div class="field">
              <label for="task-date">截止日期</label>
              <input id="task-date" v-model="form.date" type="date" />
            </div>

            <div class="field">
              <label for="task-note">备注说明</label>

              <textarea
                id="task-note"
                v-model="form.note"
                maxlength="500"
                rows="3"
                placeholder="补充任务细节（选填）"
              />
            </div>

            <button class="primary full" type="submit">
              {{ editingId !== null ? '保存修改' : '＋ 添加任务' }}
            </button>

            <button
              v-if="editingId !== null"
              class="secondary full cancel"
              type="button"
              @click="cancelEdit"
            >
              取消编辑
            </button>
          </form>
        </section>

        <section class="side-card">
          <div class="eyebrow">A LITTLE OVERVIEW</div>
          <h2>任务概览</h2>

          <p class="side-desc">
            清楚地看见，自己走了多远。
          </p>

          <div class="stats">
            <div class="stat">
              <span>全部任务</span>
              <strong>{{ tasks.length }}</strong>
            </div>

            <div class="stat">
              <span>已完成</span>
              <strong>{{ completedCount }}</strong>
            </div>

            <div class="stat">
              <span>未完成</span>
              <strong>{{ pendingCount }}</strong>
            </div>
          </div>

          <div class="categories">
            <div
              v-for="category in categoryCounts"
              :key="category.name"
              class="cat-row"
            >
              <span>{{ category.name }}</span>

              <div class="cat-track">
                <div
                  class="cat-fill"
                  :style="{ width: `${category.percent}%` }"
                />
              </div>

              <strong>{{ category.count }}</strong>
            </div>
          </div>

          <div class="footnote">
            “进步不必很大，只要每天发生。”
          </div>
        </section>
      </aside>
    </div>

    <div
      v-if="confirmClear"
      class="modal"
      @click.self="confirmClear = false"
    >
      <section
        class="dialog"
        role="dialog"
        aria-modal="true"
        aria-labelledby="dialog-title"
      >
        <h2 id="dialog-title">清除已完成任务？</h2>

        <p>这会移除所有已标记完成的任务，且无法撤销。</p>

        <div class="dialog-actions">
          <button
            class="secondary"
            type="button"
            @click="confirmClear = false"
          >
            再想想
          </button>

          <button
            class="danger"
            type="button"
            @click="clearCompleted"
          >
            确认清除
          </button>
        </div>
      </section>
    </div>

    <Transition name="toast">
      <div v-if="toast" class="toast" role="status">
        {{ toast }}
      </div>
    </Transition>
  </main>
</template>
