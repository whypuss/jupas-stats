<template>
  <div class="container">

    <!-- Header -->
    <header>
      <div class="header-title">JUPAS 2025 <span>智能配對</span></div>
      <div class="header-sub">輸入 HKDSE 成績，計算升學機會</div>
    </header>

    <!-- Calculator -->
    <section>
      <div class="section-title">HKDSE 成績</div>
      <div class="grade-grid">
        <div class="grade-item" v-for="(subj, idx) in subjects" :key="idx">
          <label>{{ subj.label }}</label>
          <select v-model="grades[idx]" class="grade-select" :class="gradeClass(grades[idx])">
            <option value="">—</option>
            <option v-for="g in gradeOptions" :key="g" :value="g">{{ g }}</option>
          </select>
        </div>
      </div>

      <div class="score-card" v-if="best5Score > 0">
        <div class="score-big">{{ best5Score.toFixed(1) }}</div>
        <div class="score-label">最佳5科總分</div>
      </div>

      <div class="match-chips" v-if="best5Score > 0">
        <div class="match-chip safe">
          <div class="chip-count">{{ matchResults.safe.length }}</div>
          <div class="chip-label">穩陣</div>
        </div>
        <div class="match-chip match">
          <div class="chip-count">{{ matchResults.match.length }}</div>
          <div class="chip-label">合理</div>
        </div>
        <div class="match-chip reach">
          <div class="chip-count">{{ matchResults.reach.length }}</div>
          <div class="chip-label">衝刺</div>
        </div>
      </div>

      <div v-if="best5Score > 0">
        <div v-if="matchResults.safe.length" class="match-group">
          <div class="match-title safe">穩陣 — 高於中位分</div>
          <div v-for="p in matchResults.safe.slice(0,6)" :key="p.code+p.uni" class="program-card safe">
            <div class="card-header">
              <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
              <span class="uni-name">{{ getUni(p.uni).name }}</span>
              <span class="prog-code">{{ p.code }}</span>
            </div>
            <div class="prog-name">{{ p.name }}</div>
            <div class="prog-tags">
              <span class="tag">{{ p.category }}</span>
              <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
              <span class="tag interview" v-if="p.interview">面試</span>
            </div>
            <div class="score-pills">
              <div class="score-pill">
                <span class="sp-label">上四分</span>
                <span class="sp-val" :class="scoreClass(p.uq)">{{ p.uq ?? '—' }}</span>
              </div>
              <div class="score-pill highlight">
                <span class="sp-label">中位分</span>
                <span class="sp-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
              </div>
              <div class="score-pill">
                <span class="sp-label">下四分</span>
                <span class="sp-val" :class="scoreClass(p.lq)">{{ p.lq ?? '—' }}</span>
              </div>
            </div>
            <div class="chance-row">
              <span class="chance-label-sm">機會</span>
              <div class="chance-bar-bg">
                <div class="chance-bar" :style="{width: p.chance+'%', background: chanceColor(p.chance)}"></div>
              </div>
              <span class="chance-pct" :style="{color: chanceColor(p.chance)}">{{ p.chance }}%</span>
            </div>
          </div>
        </div>

        <div v-if="matchResults.match.length" class="match-group">
          <div class="match-title match">合理 — 接近中位分</div>
          <div v-for="p in matchResults.match.slice(0,6)" :key="p.code+p.uni" class="program-card match">
            <div class="card-header">
              <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
              <span class="uni-name">{{ getUni(p.uni).name }}</span>
              <span class="prog-code">{{ p.code }}</span>
            </div>
            <div class="prog-name">{{ p.name }}</div>
            <div class="prog-tags">
              <span class="tag">{{ p.category }}</span>
              <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
              <span class="tag interview" v-if="p.interview">面試</span>
            </div>
            <div class="score-pills">
              <div class="score-pill">
                <span class="sp-label">中位分</span>
                <span class="sp-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
              </div>
              <div class="score-pill">
                <span class="sp-label">下四分</span>
                <span class="sp-val" :class="scoreClass(p.lq)">{{ p.lq ?? '—' }}</span>
              </div>
            </div>
            <div class="chance-row">
              <span class="chance-label-sm">機會</span>
              <div class="chance-bar-bg">
                <div class="chance-bar" :style="{width: p.chance+'%', background: chanceColor(p.chance)}"></div>
              </div>
              <span class="chance-pct" :style="{color: chanceColor(p.chance)}">{{ p.chance }}%</span>
            </div>
          </div>
        </div>

        <div v-if="matchResults.reach.length" class="match-group">
          <div class="match-title reach">衝刺 — 低於下四分位</div>
          <div v-for="p in matchResults.reach.slice(0,6)" :key="p.code+p.uni" class="program-card reach">
            <div class="card-header">
              <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
              <span class="uni-name">{{ getUni(p.uni).name }}</span>
              <span class="prog-code">{{ p.code }}</span>
            </div>
            <div class="prog-name">{{ p.name }}</div>
            <div class="prog-tags">
              <span class="tag">{{ p.category }}</span>
              <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
              <span class="tag interview" v-if="p.interview">面試</span>
            </div>
            <div class="score-pills">
              <div class="score-pill">
                <span class="sp-label">中位分</span>
                <span class="sp-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
              </div>
              <div class="score-pill">
                <span class="sp-label">下四分</span>
                <span class="sp-val" :class="scoreClass(p.lq)">{{ p.lq ?? '—' }}</span>
              </div>
            </div>
            <div class="chance-row">
              <span class="chance-label-sm">機會</span>
              <div class="chance-bar-bg">
                <div class="chance-bar" :style="{width: p.chance+'%', background: chanceColor(p.chance)}"></div>
              </div>
              <span class="chance-pct" :style="{color: chanceColor(p.chance)}">{{ p.chance }}%</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <div class="section-divider"><span>全部課程</span></div>

    <!-- Browser -->
    <section>
      <div class="filters">
        <div class="filter-row">
          <div class="filter-group">
            <label>院校</label>
            <select v-model="selectedUni" @change="currentPage=1">
              <option value="all">全部</option>
              <option v-for="u in universities" :key="u.id" :value="u.id">{{ u.name }}</option>
            </select>
          </div>
          <div class="filter-group">
            <label>類別</label>
            <select v-model="selectedCategory" @change="currentPage=1">
              <option value="">全部</option>
              <option v-for="c in categories" :key="c" :value="c">{{ c }}</option>
            </select>
          </div>
        </div>
        <div class="filter-row">
          <div class="filter-group" style="flex:2">
            <label>搜尋</label>
            <input v-model="searchQuery" type="text" placeholder="課程名稱 / 編號..." @input="currentPage=1" />
          </div>
          <div class="filter-group">
            <label>排序</label>
            <select v-model="sortBy" @change="currentPage=1">
              <option value="median-desc">分數 高→低</option>
              <option value="median-asc">分數 低→高</option>
              <option value="name">課程名稱</option>
            </select>
          </div>
        </div>
      </div>

      <div style="font-size:12px;color:var(--text3);padding:0 4px 8px;">
        {{ filteredPrograms.length }} 個課程
      </div>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>院校</th>
              <th>課程</th>
              <th>計分</th>
              <th style="text-align:center">上</th>
              <th style="text-align:center">中</th>
              <th style="text-align:center">下</th>
              <th style="text-align:center">年</th>
              <th style="text-align:center">面</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="p in paginatedPrograms" :key="p.code+p.uni" class="table-row">
              <td>
                <span class="uni-badge" :style="{background: getUni(p.uni).color+'18', color: getUni(p.uni).color}">
                  {{ getUni(p.uni).name }}
                </span>
              </td>
              <td>
                <div class="prog-name-cell">
                  <span class="prog-code-sm">{{ p.code }}</span>
                  <span style="font-size:13px;font-weight:500">{{ p.name }}</span>
                  <span style="font-size:11px;color:var(--text3)">{{ p.category }}</span>
                </div>
              </td>
              <td class="formula-cell">{{ p.formula }}</td>
              <td class="num-cell" :class="p.uq ? 'has-uq' : ''" style="text-align:center">{{ p.uq ?? '—' }}</td>
              <td class="num-cell" style="text-align:center">
                <strong :style="{color: scoreColor(p.median)}">{{ p.median ?? '—' }}</strong>
              </td>
              <td class="num-cell" :class="p.lq ? 'has-lq' : ''" style="text-align:center">{{ p.lq ?? '—' }}</td>
              <td style="text-align:center">
                <span class="tag years" v-if="p.years !== 4" style="font-size:10px;padding:1px 5px">{{ p.years }}</span>
                <span v-else style="color:var(--text3);font-size:12px">4</span>
              </td>
              <td style="text-align:center">
                <span style="color:var(--danger);font-size:12px" v-if="p.interview">要</span>
                <span v-else style="color:var(--text3);font-size:12px">—</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="pagination">
        <button class="page-btn" :disabled="currentPage===1" @click="currentPage--">‹</button>
        <button v-for="n in pageRange" :key="n" class="page-btn" :class="{active: n===currentPage}" @click="currentPage=n">{{ n }}</button>
        <button class="page-btn" :disabled="currentPage>=totalPages" @click="currentPage++">›</button>
      </div>
    </section>

    <footer>
      JUPAS 2025 · 資料僅供參考<br/>
      <small style="color:var(--text3)">升學機會為估算，實際取錄視乎面試、Band選擇等因素</small>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { universities, programs, gradePoints, gradeOptions } from './data.js'

const PAGE_SIZE = 25

const categories = [...new Set(programs.map(p => p.category))].sort()

const subjects = [
  { label: '中文' }, { label: '英文' }, { label: '數學' },
  { label: '通識' }, { label: '選修1' }, { label: '選修2' }, { label: '選修3' },
]
const grades = ref(['', '', '', '', '', '', ''])

function gradeClass(g) {
  if (g === '5**') return 'grade-5starstar'
  if (g === '5*') return 'grade-5star'
  if (g === '5') return 'grade-5'
  if (g === '4') return 'grade-4'
  return ''
}

const best5Score = computed(() => {
  const pts = grades.value.map(g => gradePoints[g] || 0).sort((a, b) => b - a)
  return pts.slice(0, 5).reduce((s, x) => s + x, 0)
})

function calcChance(score, median, lq, uq) {
  if (median == null) return 0
  if (uq != null && score >= uq + 3) return Math.min(95, 75 + (score - median) * 2)
  if (score >= median) {
    const range = (uq ?? median + 5) - median
    return Math.round(65 + ((score - median) / range) * 20)
  }
  if (score >= (lq ?? median - 5)) {
    return Math.round(35 + ((score - (lq ?? median - 5)) / ((median - (lq ?? median - 5)) || 5)) * 30)
  }
  return Math.max(5, Math.round((score / (lq ?? 10)) * 15))
}

const matchResults = computed(() => {
  if (best5Score.value === 0) return { safe: [], match: [], reach: [] }
  const score = best5Score.value
  const safe = [], match = [], reach = []
  for (const p of programs) {
    if (p.median == null) continue
    const chance = calcChance(score, p.median, p.lq, p.uq)
    const progWithChance = { ...p, chance }
    const diff = score - p.median
    if (diff >= 3) safe.push(progWithChance)
    else if (diff >= -5) match.push(progWithChance)
    else reach.push(progWithChance)
  }
  safe.sort((a, b) => b.median - a.median)
  match.sort((a, b) => b.chance - a.chance)
  reach.sort((a, b) => a.median - b.median)
  return { safe, match, reach }
})

function chanceColor(chance) {
  if (chance >= 70) return 'var(--success)'
  if (chance >= 50) return 'var(--warning)'
  return 'var(--danger)'
}

function scoreColor(score) {
  if (score == null) return 'var(--text3)'
  if (score >= 50) return 'var(--danger)'
  if (score >= 30) return 'var(--warning)'
  return 'var(--success)'
}

function scoreClass(score) {
  if (score == null) return ''
  if (score >= 50) return 'low'
  if (score >= 30) return 'mid'
  return 'high'
}

const selectedUni = ref('all')
const selectedCategory = ref('')
const searchQuery = ref('')
const sortBy = ref('median-desc')
const currentPage = ref(1)

watch([selectedUni, selectedCategory, searchQuery, sortBy], () => { currentPage.value = 1 })

const filteredPrograms = computed(() => {
  let list = programs
  if (selectedUni.value !== 'all') list = list.filter(p => p.uni === selectedUni.value)
  if (selectedCategory.value) list = list.filter(p => p.category === selectedCategory.value)
  if (searchQuery.value.trim()) {
    const q = searchQuery.value.toLowerCase()
    list = list.filter(p => p.name.toLowerCase().includes(q) || p.code.toLowerCase().includes(q) || p.formula.toLowerCase().includes(q))
  }
  list = [...list]
  if (sortBy.value === 'median-desc') list.sort((a, b) => (b.median ?? 0) - (a.median ?? 0))
  else if (sortBy.value === 'median-asc') list.sort((a, b) => (a.median ?? 0) - (b.median ?? 0))
  else if (sortBy.value === 'name') list.sort((a, b) => a.name.localeCompare(b.name))
  return list
})

const totalPages = computed(() => Math.max(1, Math.ceil(filteredPrograms.value.length / PAGE_SIZE)))
const paginatedPrograms = computed(() => {
  const start = (currentPage.value - 1) * PAGE_SIZE
  return filteredPrograms.value.slice(start, start + PAGE_SIZE)
})
const pageRange = computed(() => {
  const total = totalPages.value, cur = currentPage.value
  const range = []
  for (let i = Math.max(1, cur - 2); i <= Math.min(total, cur + 2); i++) range.push(i)
  return range
})

function getUni(id) { return universities.find(u => u.id === id) || {} }
</script>
