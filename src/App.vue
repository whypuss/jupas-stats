<template>
  <div class="container">

    <!-- Header -->
    <header>
      <div class="header-badge">📚 JUPAS 2025</div>
      <h1>香港大學收生分數<br><span class="gradient-text">智能配對系統</span></h1>
      <p class="subtitle">輸入你嘅 HKDSE 成績，計算升學機會</p>
    </header>

    <!-- ===== Calculator ===== -->
    <section class="calc-section">
      <h2 class="section-title">🎯 HKDSE 分數計算機</h2>
      <p class="calc-hint">選擇你預計 / 實際拿到的等級，系統會計算「最佳5科」總分及升學機會</p>

      <div class="grade-grid">
        <div class="grade-item" v-for="(subj, idx) in subjects" :key="idx">
          <label>{{ subj.label }}</label>
          <select v-model="grades[idx]" class="grade-select" :class="gradeClass(grades[idx])">
            <option value="">—</option>
            <option v-for="g in gradeOptions" :key="g" :value="g" v-show="g">{{ g }}</option>
          </select>
        </div>
      </div>

      <div class="score-summary" v-if="best5Score > 0">
        <div class="score-pill highlight">
          <span class="score-pill-label">最佳5科總分</span>
          <span class="score-pill-value">{{ best5Score.toFixed(1) }}</span>
        </div>
        <div class="score-pill">
          <span class="score-pill-label">公民與社會發展</span>
          <span class="score-pill-value dim">不計分</span>
        </div>
      </div>

      <!-- Match Results -->
      <div v-if="best5Score > 0" class="match-results">
        <div class="match-summary-row">
          <div class="match-chip safe">
            <span class="match-count">{{ matchResults.safe.length }}</span>
            <span>穩陣</span>
          </div>
          <div class="match-chip reach">
            <span class="match-count">{{ matchResults.reach.length }}</span>
            <span>衝刺</span>
          </div>
          <div class="match-chip match">
            <span class="match-count">{{ matchResults.match.length }}</span>
            <span>合理</span>
          </div>
        </div>

        <div v-if="matchResults.safe.length" class="match-group">
          <h3 class="match-title safe">✅ 穩陣（高於中位分）</h3>
          <div class="match-cards">
            <div v-for="p in matchResults.safe.slice(0,9)" :key="p.code + p.uni" class="program-card safe">
              <div class="card-header">
                <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
                <span class="uni-name">{{ getUni(p.uni).name_en }}</span>
                <code class="prog-code">{{ p.code }}</code>
              </div>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-meta">
                <span class="cat-tag">{{ p.category }}</span>
                <span class="years-tag" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="interview-tag" v-if="p.interview">面試</span>
              </div>
              <div class="prog-scores">
                <div class="score-row">
                  <span class="score-label">上四分位</span>
                  <span class="score-val">{{ p.uq ?? '—' }}</span>
                </div>
                <div class="score-row">
                  <span class="score-label">中位分</span>
                  <span class="score-val median">{{ p.median }}</span>
                </div>
                <div class="score-row">
                  <span class="score-label">下四分位</span>
                  <span class="score-val">{{ p.lq ?? '—' }}</span>
                </div>
                <div class="score-row your">
                  <span class="score-label">你的分數</span>
                  <span class="score-val" :style="{color: scoreColor(p.median)}">{{ best5Score.toFixed(1) }}</span>
                </div>
              </div>
              <div class="chance-bar-wrap">
                <div class="chance-label">
                  <span>升學機會</span>
                  <strong :style="{color: chanceColor(p.chance)}">{{ chanceLabel(p.chance) }}</strong>
                </div>
                <div class="chance-bar-bg">
                  <div class="chance-bar" :style="{width: p.chance + '%', background: chanceColor(p.chance)}"></div>
                </div>
                <div class="chance-pct">{{ p.chance }}%</div>
              </div>
            </div>
          </div>
        </div>

        <div v-if="matchResults.match.length" class="match-group">
          <h3 class="match-title match">🎯 合理（接近中位分）</h3>
          <div class="match-cards">
            <div v-for="p in matchResults.match.slice(0,9)" :key="p.code + p.uni" class="program-card match">
              <div class="card-header">
                <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
                <span class="uni-name">{{ getUni(p.uni).name_en }}</span>
                <code class="prog-code">{{ p.code }}</code>
              </div>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-meta">
                <span class="cat-tag">{{ p.category }}</span>
                <span class="years-tag" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="interview-tag" v-if="p.interview">面試</span>
              </div>
              <div class="prog-scores">
                <div class="score-row" v-if="p.uq">
                  <span class="score-label">上四分位</span>
                  <span class="score-val">{{ p.uq }}</span>
                </div>
                <div class="score-row">
                  <span class="score-label">中位分</span>
                  <span class="score-val median">{{ p.median }}</span>
                </div>
                <div class="score-row">
                  <span class="score-label">下四分位</span>
                  <span class="score-val">{{ p.lq ?? '—' }}</span>
                </div>
                <div class="score-row your">
                  <span class="score-label">你的分數</span>
                  <span class="score-val" :style="{color: scoreColor(p.median)}">{{ best5Score.toFixed(1) }}</span>
                </div>
              </div>
              <div class="chance-bar-wrap">
                <div class="chance-label">
                  <span>升學機會</span>
                  <strong :style="{color: chanceColor(p.chance)}">{{ chanceLabel(p.chance) }}</strong>
                </div>
                <div class="chance-bar-bg">
                  <div class="chance-bar" :style="{width: p.chance + '%', background: chanceColor(p.chance)}"></div>
                </div>
                <div class="chance-pct">{{ p.chance }}%</div>
              </div>
            </div>
          </div>
        </div>

        <div v-if="matchResults.reach.length" class="match-group">
          <h3 class="match-title reach">🔥 衝刺（低於下四分位）</h3>
          <div class="match-cards">
            <div v-for="p in matchResults.reach.slice(0,9)" :key="p.code + p.uni" class="program-card reach">
              <div class="card-header">
                <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
                <span class="uni-name">{{ getUni(p.uni).name_en }}</span>
                <code class="prog-code">{{ p.code }}</code>
              </div>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-meta">
                <span class="cat-tag">{{ p.category }}</span>
                <span class="years-tag" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="interview-tag" v-if="p.interview">面試</span>
              </div>
              <div class="prog-scores">
                <div class="score-row" v-if="p.uq">
                  <span class="score-label">上四分位</span>
                  <span class="score-val">{{ p.uq }}</span>
                </div>
                <div class="score-row">
                  <span class="score-label">中位分</span>
                  <span class="score-val median">{{ p.median }}</span>
                </div>
                <div class="score-row">
                  <span class="score-label">下四分位</span>
                  <span class="score-val">{{ p.lq ?? '—' }}</span>
                </div>
                <div class="score-row your">
                  <span class="score-label">你的分數</span>
                  <span class="score-val" :style="{color: scoreColor(p.median)}">{{ best5Score.toFixed(1) }}</span>
                </div>
              </div>
              <div class="chance-bar-wrap">
                <div class="chance-label">
                  <span>升學機會</span>
                  <strong :style="{color: chanceColor(p.chance)}">{{ chanceLabel(p.chance) }}</strong>
                </div>
                <div class="chance-bar-bg">
                  <div class="chance-bar" :style="{width: p.chance + '%', background: chanceColor(p.chance)}"></div>
                </div>
                <div class="chance-pct">{{ p.chance }}%</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <div class="section-divider"><span>全部課程庫</span></div>

    <!-- ===== Program Browser ===== -->
    <section class="browser-section">
      <h2 class="section-title">📋 全部課程 ({{ filteredPrograms.length }})</h2>

      <div class="filters">
        <div class="filter-group">
          <label>院校</label>
          <select v-model="selectedUni">
            <option value="all">全部院校</option>
            <option v-for="u in universities" :key="u.id" :value="u.id">{{ u.name }}</option>
          </select>
        </div>
        <div class="filter-group">
          <label>類別</label>
          <select v-model="selectedCategory">
            <option value="">全部類別</option>
            <option v-for="c in categories" :key="c" :value="c">{{ c }}</option>
          </select>
        </div>
        <div class="filter-group">
          <label>搜尋</label>
          <input v-model="searchQuery" type="text" placeholder="課程名稱 / 編號..." @input="currentPage = 1" />
        </div>
        <div class="filter-group">
          <label>排序</label>
          <select v-model="sortBy">
            <option value="median-desc">分數 高→低</option>
            <option value="median-asc">分數 低→高</option>
            <option value="name">課程名稱</option>
          </select>
        </div>
      </div>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>院校</th>
              <th>課程名稱</th>
              <th>類別</th>
              <th>計分</th>
              <th>上四分</th>
              <th>中位分</th>
              <th>下四分</th>
              <th>年</th>
              <th>面試</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="p in paginatedPrograms" :key="p.code + p.uni" class="table-row">
              <td>
                <span class="uni-badge" :style="{background: getUni(p.uni).color + '22', color: getUni(p.uni).color}">
                  {{ getUni(p.uni).name_en }}
                </span>
              </td>
              <td>
                <div class="prog-name-cell">
                  <code class="prog-code-sm">{{ p.code }}</code>
                  {{ p.name }}
                </div>
              </td>
              <td><span class="cat-tag">{{ p.category }}</span></td>
              <td class="formula-cell">{{ p.formula }}</td>
              <td class="num-cell" :class="p.uq ? 'has-uq' : ''">{{ p.uq ?? '—' }}</td>
              <td class="num-cell">
                <strong :style="{color: scoreColor(p.median)}">{{ p.median ?? '—' }}</strong>
              </td>
              <td class="num-cell" :class="p.lq ? 'has-lq' : ''">{{ p.lq ?? '—' }}</td>
              <td class="num-cell">
                <span class="years-tag" v-if="p.years !== 4">{{ p.years }}年</span>
                <span v-else class="text-dim">4</span>
              </td>
              <td class="num-cell">
                <span class="interview-tag" v-if="p.interview">要</span>
                <span v-else class="text-dim">—</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="pagination">
        <button class="page-btn" :disabled="currentPage === 1" @click="currentPage--">←</button>
        <button v-for="n in pageRange" :key="n" class="page-btn" :class="{ active: n === currentPage }" @click="currentPage = n">{{ n }}</button>
        <button class="page-btn" :disabled="currentPage >= totalPages" @click="currentPage++">→</button>
      </div>
    </section>

    <footer>
      資料來源：JUPAS 官方網站 · 2025 entry · 僅供參考<br/>
      <small>升學機會僅作估算，實際取錄視乎面試、Band選擇、整體申請者表現等因素。個別院校計分制不同，不可跨校比較。</small>
    </footer>
  </div>
</template>


<script setup>
import { ref, computed, watch } from 'vue'
import { universities, programs, gradePoints, gradeOptions } from './data.js'

const PAGE_SIZE = 30

// ---- Categories ----
const categories = [...new Set(programs.map(p => p.category))].sort()

// ---- HKDSE Calculator ----
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

// Calculate admission chance based on score vs quartiles
function calcChance(score, median, lq, uq) {
  if (median == null) return 0
  if (score >= (uq ?? median + 5)) return Math.min(95, 75 + (score - median) * 2)
  if (score >= median) {
    const range = (uq ?? median + 5) - median
    const pos = score - median
    return Math.round(65 + (pos / range) * 20)
  }
  if (score >= (lq ?? median - 5)) {
    const top = median
    const bot = lq ?? median - 5
    const pos = score - bot
    return Math.round(35 + (pos / (top - bot)) * 30)
  }
  if (score >= lq - 5) return Math.round(15 + (score - (lq - 5)) * 4)
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

function chanceLabel(chance) {
  if (chance >= 80) return '極高'
  if (chance >= 65) return '高'
  if (chance >= 45) return '中'
  if (chance >= 25) return '低'
  return '極低'
}

function chanceColor(chance) {
  if (chance >= 70) return '#4ade80'
  if (chance >= 50) return '#fbbf24'
  if (chance >= 30) return '#f97316'
  return '#f87171'
}

// ---- Browser ----
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

function scoreColor(score) {
  if (score == null) return '#7a8099'
  if (score >= 50) return '#f87171'
  if (score >= 30) return '#fbbf24'
  if (score >= 200) return '#f87171'
  if (score >= 180) return '#fbbf24'
  return '#4ade80'
}
</script>
