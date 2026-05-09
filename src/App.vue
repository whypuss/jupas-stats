<template>
  <div class="container">

    <!-- Header -->
    <header>
      <div class="header-badge">📚 JUPAS 2025</div>
      <h1>香港大學收生分數<br><span class="gradient-text">智能配對系統</span></h1>
      <p class="subtitle">輸入你嘅 HKDSE 成績，自動計算符合嘅課程</p>
    </header>

    <!-- ===== HKDSE Calculator ===== -->
    <section class="calc-section">
      <h2 class="section-title">🎯 HKDSE 分數計算機</h2>
      <p class="calc-hint">選擇你預計 / 實際拿到的等級，系統會自動計算你的「最佳5科」總分</p>

      <!-- Grade inputs -->
      <div class="grade-grid">
        <div class="grade-item" v-for="(subj, idx) in subjects" :key="idx">
          <label>{{ subj.label }}</label>
          <select v-model="grades[idx]" class="grade-select" :class="gradeClass(grades[idx])">
            <option value="">—</option>
            <option v-for="g in gradeOptions" :key="g" :value="g" v-show="g">{{ g }}</option>
          </select>
        </div>
      </div>

      <!-- Score summary -->
      <div class="score-summary" v-if="best5Score > 0">
        <div class="score-pill">
          <span class="score-pill-label">最佳5科總分</span>
          <span class="score-pill-value">{{ best5Score.toFixed(1) }}</span>
        </div>
        <div class="score-pill">
          <span class="score-pill-label">公民與社會發展</span>
          <span class="score-pill-value dim">不計分</span>
        </div>
      </div>

      <!-- Match results -->
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
          <h3 class="match-title safe">✅ 穩陣（低於中位分）</h3>
          <div class="match-cards">
            <div v-for="p in matchResults.safe.slice(0,6)" :key="p.code + p.uni" class="program-card safe">
              <div class="card-header">
                <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
                <span class="uni-name">{{ getUni(p.uni).name_en }}</span>
                <code class="prog-code">{{ p.code }}</code>
              </div>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-meta">
                <span class="cat-tag">{{ p.category }}</span>
                <span class="prog-score">
                  中位 {{ p.median }} / 你的 {{ best5Score.toFixed(1) }}
                  <span class="diff plus">+{{ (best5Score - p.median).toFixed(1) }}</span>
                </span>
              </div>
            </div>
          </div>
        </div>

        <div v-if="matchResults.match.length" class="match-group">
          <h3 class="match-title match">🎯 合理（在中位分附近）</h3>
          <div class="match-cards">
            <div v-for="p in matchResults.match.slice(0,6)" :key="p.code + p.uni" class="program-card match">
              <div class="card-header">
                <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
                <span class="uni-name">{{ getUni(p.uni).name_en }}</span>
                <code class="prog-code">{{ p.code }}</code>
              </div>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-meta">
                <span class="cat-tag">{{ p.category }}</span>
                <span class="prog-score">
                  中位 {{ p.median }} / 你的 {{ best5Score.toFixed(1) }}
                  <span class="diff" :class="best5Score >= p.median ? 'plus' : 'minus'">
                    {{ best5Score >= p.median ? '+' : '' }}{{ (best5Score - p.median).toFixed(1) }}
                  </span>
                </span>
              </div>
            </div>
          </div>
        </div>

        <div v-if="matchResults.reach.length" class="match-group">
          <h3 class="match-title reach">🔥 衝刺（高於中位分）</h3>
          <div class="match-cards">
            <div v-for="p in matchResults.reach.slice(0,6)" :key="p.code + p.uni" class="program-card reach">
              <div class="card-header">
                <span class="uni-dot" :style="{background: getUni(p.uni).color}"></span>
                <span class="uni-name">{{ getUni(p.uni).name_en }}</span>
                <code class="prog-code">{{ p.code }}</code>
              </div>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-meta">
                <span class="cat-tag">{{ p.category }}</span>
                <span class="prog-score">
                  中位 {{ p.median }} / 你的 {{ best5Score.toFixed(1) }}
                  <span class="diff minus">{{ (best5Score - p.median).toFixed(1) }}</span>
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ===== Divider ===== -->
    <div class="section-divider"><span>或者 Browse 所有課程</span></div>

    <!-- ===== Program Browser ===== -->
    <section class="browser-section">
      <h2 class="section-title">📋 全部課程庫</h2>

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
            <option value="全部">全部類別</option>
            <option v-for="c in categories.slice(1)" :key="c" :value="c">{{ c }}</option>
          </select>
        </div>
        <div class="filter-group">
          <label>搜尋</label>
          <input v-model="searchQuery" type="text" placeholder="課程名稱..." @input="currentPage = 1" />
        </div>
        <div class="filter-stats"><strong>{{ filteredPrograms.length }}</strong> 個課程</div>
      </div>

      <div class="stats-row">
        <div class="stat-card">
          <div class="stat-num">{{ filteredPrograms.length }}</div>
          <div class="stat-label">課程總數</div>
        </div>
        <div class="stat-card">
          <div class="stat-num green">{{ highestScore }}</div>
          <div class="stat-label">最高中位分</div>
        </div>
        <div class="stat-card">
          <div class="stat-num yellow">{{ avgMedian }}</div>
          <div class="stat-label">平均中位分</div>
        </div>
        <div class="stat-card">
          <div class="stat-num">{{ universities.length }}</div>
          <div class="stat-label">院校數</div>
        </div>
      </div>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>院校</th>
              <th>課程名稱</th>
              <th>類別</th>
              <th>計分公式</th>
              <th>中位分</th>
              <th>難度</th>
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
              <td>
                <strong class="median-score" :style="{color: scoreColor(p.median)}">{{ p.median }}</strong>
                <span class="lq-hint" v-if="p.lq">/ {{ p.lq }}</span>
              </td>
              <td>
                <div class="difficulty-bar-wrap">
                  <div class="diff-bar-bg">
                    <div class="diff-bar" :style="{width: barWidth(p.median), background: scoreColor(p.median)}"></div>
                  </div>
                  <span class="diff-label" :style="{color: scoreColor(p.median)}">{{ difficulty(p.median) }}</span>
                </div>
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
      <small>計分制各院校不同，跨校比較僅供參考。實際取錄分數每年因申請人整體表現而異。</small>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { universities, programs, categories, gradePoints, gradeOptions } from './data.js'

const PAGE_SIZE = 25

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

const matchResults = computed(() => {
  if (best5Score.value === 0) return { safe: [], match: [], reach: [] }
  const score = best5Score.value
  const safe = [], match = [], reach = []
  for (const p of programs) {
    if (p.median == null) continue
    const diff = score - p.median
    if (diff >= 5) safe.push(p)
    else if (diff >= -5) match.push(p)
    else reach.push(p)
  }
  safe.sort((a, b) => b.median - a.median)
  match.sort((a, b) => b.median - a.median)
  reach.sort((a, b) => a.median - b.median)
  return { safe, match, reach }
})

// ---- Browser ----
const selectedUni = ref('all')
const selectedCategory = ref('全部')
const searchQuery = ref('')
const currentPage = ref(1)

watch([selectedUni, selectedCategory, searchQuery], () => { currentPage.value = 1 })

const filteredPrograms = computed(() => {
  let list = programs
  if (selectedUni.value !== 'all') list = list.filter(p => p.uni === selectedUni.value)
  if (selectedCategory.value !== '全部') list = list.filter(p => p.category === selectedCategory.value)
  if (searchQuery.value.trim()) {
    const q = searchQuery.value.toLowerCase()
    list = list.filter(p => p.name.toLowerCase().includes(q) || p.code.toLowerCase().includes(q))
  }
  return [...list].sort((a, b) => b.median - a.median)
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

const highestScore = computed(() => {
  if (!filteredPrograms.value.length) return 0
  return Math.max(...filteredPrograms.value.map(p => p.median))
})
const avgMedian = computed(() => {
  if (!filteredPrograms.value.length) return 0
  return (filteredPrograms.value.reduce((s, p) => s + p.median, 0) / filteredPrograms.value.length).toFixed(1)
})

function getUni(id) { return universities.find(u => u.id === id) || {} }

function scoreColor(score) {
  if (score >= 200) return '#f56c6c'
  if (score >= 50) return '#f56c6c'
  if (score >= 30) return '#f5c842'
  if (score >= 20) return '#5ce6a1'
  return '#6c8ef5'
}

function difficulty(score) {
  if (score >= 200) return '極難'
  if (score >= 50) return '極難'
  if (score >= 30) return '困難'
  if (score >= 20) return '中等'
  return '一般'
}

function barWidth(score) {
  return Math.min(100, (score / 340 * 100)).toFixed(1) + '%'
}
</script>
