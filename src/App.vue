<template>
  <div class="container">

    <!-- Header -->
    <header>
      <div class="header-title">JUPAS 2025 <span>智能配對</span></div>
      <div class="header-sub">輸入 HKDSE 成績，計算升學機會</div>
    </header>

    <!-- ── Calculator ── -->
    <section class="calc-section">
      <div class="section-title">&#x1F4DA; 選擇大學計分制</div>

      <div class="uni-selector-row">
        <select v-model="selectedUni" class="uni-selector">
          <option value="">— 選擇大學 —</option>
          <option value="hku">香港大學（5**=8.5分）</option>
          <option value="polyu">香港理工大學（5**=8.5分）</option>
          <option value="cityu">香港城市大學（5**=7分）</option>
          <option value="hkbu">香港浸會大學（5**=7分）</option>
          <option value="hkmud">香港都會大學（5**=7分）</option>
        </select>
        <div class="scoring-hint" v-if="selectedUni">
          計分制：{{ currentScoringSystem }}
        </div>
      </div>

      <div class="section-title" style="margin-top:16px">&#x1F4DA; HKDSE 成績輸入</div>

      <div class="grade-grid">
        <div class="grade-item" v-for="(subj, idx) in subjects" :key="idx">
          <label>{{ subj.label }}</label>
          <div class="grade-select-wrap">
            <select v-model="grades[idx]" class="grade-select" :class="gradeClass(grades[idx])">
              <option value="">—</option>
              <option v-for="g in gradeOptions" :key="g" :value="g">{{ g }}</option>
            </select>
            <span class="grade-arrow">&#x276F;</span>
          </div>
          <div class="grade-pts" v-if="gradePointsForUni[grades[idx]]">
            {{ gradePointsForUni[grades[idx]] }}分
          </div>
          <div class="grade-pts dim" v-else>—</div>
        </div>
      </div>

      <!-- Bonus Input -->
      <div class="bonus-row" v-if="best5Score > 0 || bonusScore > 0">
        <div class="bonus-label">⬡ M1/M2 額外加分</div>
        <div class="bonus-input-wrap">
          <input type="number" v-model.number="bonusScore" min="0" max="20" step="0.5" class="bonus-input" placeholder="0">
          <span class="bonus-unit">分</span>
        </div>
      </div>

      <!-- Score -->
      <div class="score-card" v-if="best5Score > 0">
        <div class="score-big">{{ best5Score.toFixed(1) }}</div>
        <div class="score-label">最佳5科 + M1/M2（{{ currentScoringSystem }}）</div>
        <div class="score-breakdown" v-if="bonusScore > 0">
          <span class="score-part">Best5: {{ (best5Score - bonusScore).toFixed(1) }}</span>
          <span class="score-plus">+</span>
          <span class="score-bonus">{{ bonusScore }}</span>
        </div>
      </div>

      <!-- Match Chips -->
      <div class="match-chips" v-if="best5Score > 0">
        <div class="match-chip safe">
          <div class="chip-count">{{ matchResults.safe.length }}</div>
          <div class="chip-label">&#x2705; 穩陣</div>
        </div>
        <div class="match-chip match">
          <div class="chip-count">{{ matchResults.match.length }}</div>
          <div class="chip-label">&#x1F3AF; 合理</div>
        </div>
        <div class="match-chip reach">
          <div class="chip-count">{{ matchResults.reach.length }}</div>
          <div class="chip-label">&#x1F525; 衝刺</div>
        </div>
      </div>

      <!-- Safe Cards -->
      <div v-if="best5Score > 0 && matchResults.safe.length">
        <div class="match-group-title safe">&#x2705; 穩陣 — 高於中位分</div>
        <div v-for="p in matchResults.safe.slice(0,12)" :key="p.code+p.uni" class="prog-card safe">
          <div class="prog-card-top">
            <div class="prog-card-left">
              <span class="uni-badge" :style="{background: getUni(p.uni).color+'18', color: getUni(p.uni).color}">
                {{ getUni(p.uni).name }}
              </span>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-tags">
                <span class="tag">{{ p.category }}</span>
                <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="tag incompatible" v-if="p.incompatible">⚠ {{ p.formula }}（計分制與HKDSE不同）</span>
                <span class="tag interview" v-if="p.interview && !p.incompatible">&#x1F4AC; 面試</span>
              </div>
            </div>
            <div class="prog-card-right">
              <div class="chance-big" :style="{color: p.incompatible ? 'var(--text3)' : chanceColor(p.chance)}">{{ p.incompatible ? '—' : p.chance + '%' }}</div>
              <div class="chance-label-sm">機會</div>
            </div>
          </div>
          <div class="prog-card-scores">
            <div class="sc-item">
              <span class="sc-lbl">&#x25B2; 上四分</span>
              <span class="sc-val" :class="scoreClass(p.uq)">{{ p.uq ?? '—' }}</span>
            </div>
            <div class="sc-item highlight">
              <span class="sc-lbl">&#x25CF; 中位</span>
              <span class="sc-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
            </div>
            <div class="sc-item">
              <span class="sc-lbl">&#x25BC; 下四分</span>
              <span class="sc-val" :class="scoreClass(p.lq)">{{ p.lq ?? '—' }}</span>
            </div>
            <div class="sc-item your">
              <span class="sc-lbl">你的</span>
              <span class="sc-val" :class="scoreClass(best5Score)">{{ best5Score.toFixed(1) }}</span>
            </div>
          </div>
          <div class="chance-bar-wrap">
            <div class="chance-bar-bg">
              <div class="chance-bar" :style="{width: p.chance+'%', background: chanceColor(p.chance)}"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Match Cards -->
      <div v-if="best5Score > 0 && matchResults.match.length">
        <div class="match-group-title match">&#x1F3AF; 合理 — 接近中位分</div>
        <div v-for="p in matchResults.match.slice(0,12)" :key="p.code+p.uni" class="prog-card match">
          <div class="prog-card-top">
            <div class="prog-card-left">
              <span class="uni-badge" :style="{background: getUni(p.uni).color+'18', color: getUni(p.uni).color}">
                {{ getUni(p.uni).name }}
              </span>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-tags">
                <span class="tag">{{ p.category }}</span>
                <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="tag incompatible" v-if="p.incompatible">⚠ {{ p.formula }}（計分制與HKDSE不同）</span>
                <span class="tag interview" v-if="p.interview && !p.incompatible">&#x1F4AC; 面試</span>
              </div>
            </div>
            <div class="prog-card-right">
              <div class="chance-big" :style="{color: p.incompatible ? 'var(--text3)' : chanceColor(p.chance)}">{{ p.incompatible ? '—' : p.chance + '%' }}</div>
              <div class="chance-label-sm">機會</div>
            </div>
          </div>
          <div class="prog-card-scores">
            <div class="sc-item highlight">
              <span class="sc-lbl">&#x25CF; 中位</span>
              <span class="sc-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
            </div>
            <div class="sc-item">
              <span class="sc-lbl">&#x25BC; 下四分</span>
              <span class="sc-val" :class="scoreClass(p.lq)">{{ p.lq ?? '—' }}</span>
            </div>
            <div class="sc-item your">
              <span class="sc-lbl">你的</span>
              <span class="sc-val" :class="scoreClass(best5Score)">{{ best5Score.toFixed(1) }}</span>
            </div>
          </div>
          <div class="chance-bar-wrap">
            <div class="chance-bar-bg">
              <div class="chance-bar" :style="{width: p.chance+'%', background: chanceColor(p.chance)}"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Reach Cards -->
      <div v-if="best5Score > 0 && matchResults.reach.length">
        <div class="match-group-title reach">&#x1F525; 衝刺 — 低於下四分</div>
        <div v-for="p in matchResults.reach.slice(0,12)" :key="p.code+p.uni" class="prog-card reach">
          <div class="prog-card-top">
            <div class="prog-card-left">
              <span class="uni-badge" :style="{background: getUni(p.uni).color+'18', color: getUni(p.uni).color}">
                {{ getUni(p.uni).name }}
              </span>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-tags">
                <span class="tag">{{ p.category }}</span>
                <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="tag incompatible" v-if="p.incompatible">⚠ {{ p.formula }}（計分制與HKDSE不同）</span>
                <span class="tag interview" v-if="p.interview && !p.incompatible">&#x1F4AC; 面試</span>
              </div>
            </div>
            <div class="prog-card-right">
              <div class="chance-big" :style="{color: p.incompatible ? 'var(--text3)' : chanceColor(p.chance)}">{{ p.incompatible ? '—' : p.chance + '%' }}</div>
              <div class="chance-label-sm">機會</div>
            </div>
          </div>
          <div class="prog-card-scores">
            <div class="sc-item highlight">
              <span class="sc-lbl">&#x25CF; 中位</span>
              <span class="sc-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
            </div>
            <div class="sc-item">
              <span class="sc-lbl">&#x25BC; 下四分</span>
              <span class="sc-val" :class="scoreClass(p.lq)">{{ p.lq ?? '—' }}</span>
            </div>
            <div class="sc-item your">
              <span class="sc-lbl">你的</span>
              <span class="sc-val" :class="scoreClass(best5Score)">{{ best5Score.toFixed(1) }}</span>
            </div>
          </div>
          <div class="chance-bar-wrap">
            <div class="chance-bar-bg">
              <div class="chance-bar" :style="{width: p.chance+'%', background: chanceColor(p.chance)}"></div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <div class="section-divider"><span>&#x1F4CB; 全部課程</span></div>

    <!-- ── Program Browser ── -->
    <section class="browser-section">
      <div class="filters">
        <div class="filter-row">
          <div class="filter-group">
            <label>院校</label>
            <div class="select-wrap">
              <select v-model="selectedUni" @change="currentPage=1">
                <option value="all">全部</option>
                <option v-for="u in universities" :key="u.id" :value="u.id">{{ u.name }}</option>
              </select>
              <span class="sel-arrow">&#x276F;</span>
            </div>
          </div>
          <div class="filter-group">
            <label>類別</label>
            <div class="select-wrap">
              <select v-model="selectedCategory" @change="currentPage=1">
                <option value="">全部</option>
                <option v-for="c in categories" :key="c" :value="c">{{ c }}</option>
              </select>
              <span class="sel-arrow">&#x276F;</span>
            </div>
          </div>
        </div>
        <div class="filter-row">
          <div class="filter-group" style="flex:2">
            <label>&#x1F50D; 搜尋</label>
            <input v-model="searchQuery" type="text" placeholder="課程名稱..." @input="currentPage=1" />
          </div>
          <div class="filter-group">
            <label>排序</label>
            <div class="select-wrap">
              <select v-model="sortBy" @change="currentPage=1">
                <option value="chance-desc">成功機率</option>
                <option value="median-desc">分數 高→低</option>
                <option value="median-asc">分數 低→高</option>
                <option value="name">名稱</option>
              </select>
              <span class="sel-arrow">&#x276F;</span>
            </div>
          </div>
        </div>
      </div>

      <div class="result-count">{{ filteredPrograms.length }} 個課程</div>

      <!-- Program Card List -->
      <div class="prog-list">
        <div v-for="p in paginatedPrograms" :key="p.code+p.uni" class="prog-card browse">
          <div class="prog-card-top">
            <div class="prog-card-left">
              <span class="uni-badge" :style="{background: getUni(p.uni).color+'18', color: getUni(p.uni).color}">
                {{ getUni(p.uni).name }}
              </span>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-tags">
                <span class="tag">{{ p.category }}</span>
                <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="tag incompatible" v-if="p.incompatible">⚠ {{ p.formula }}（計分制與HKDSE不同）</span>
                <span class="tag interview" v-if="p.interview && !p.incompatible">&#x1F4AC; 面試</span>
              </div>
            </div>
          </div>
          <div class="prog-card-scores">
            <div class="sc-item" v-if="p.uq">
              <span class="sc-lbl">&#x25B2; 上四分</span>
              <span class="sc-val warn">{{ p.uq }}</span>
            </div>
            <div class="sc-item highlight">
              <span class="sc-lbl">&#x25CF; 中位</span>
              <span class="sc-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
            </div>
            <div class="sc-item" v-if="p.lq">
              <span class="sc-lbl">&#x25BC; 下四分</span>
              <span class="sc-val ok">{{ p.lq }}</span>
            </div>
            <div class="sc-item">
              <span class="sc-lbl">計分</span>
              <span class="sc-val dim">{{ p.formula }}</span>
            </div>
          </div>
        </div>
      </div>

      <div class="pagination" v-if="totalPages > 1">
        <button class="page-btn" :disabled="currentPage===1" @click="currentPage--">&#x276E;</button>
        <button v-for="n in pageRange" :key="n" class="page-btn" :class="{active: n===currentPage}" @click="currentPage=n">{{ n }}</button>
        <button class="page-btn" :disabled="currentPage>=totalPages" @click="currentPage++">&#x276F;</button>
      </div>
    </section>

    <footer>
      JUPAS 2025 · 資料僅供參考<br/>
      <small>升學機會為估算，實際取錄視乎面試、Band選擇等因素</small>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { universities, programs, gradePoints, gradeOptions } from './data.js'

const PAGE_SIZE = 20

const categories = [...new Set(programs.map(p => p.category))].sort()

const subjects = [
  { label: '中文' }, { label: '英文' }, { label: '數學' },
  { label: '選修1' }, { label: '選修2' }, { label: '選修3' },
]
const grades = ref(['', '', '', '', '', ''])
const bonusScore = ref(0)

// 根據所選大學返回對應計分制
// 8.5分制：港大/理工
const gradePointsHigh = { '5**': 8.5, '5*': 7, '5': 6, '4': 5, '3': 4, '2': 3, '1': 2 }
// 7分制：城大/浸大/都大
const gradePointsStandard = { '5**': 7, '5*': 6, '5': 5, '4': 4, '3': 3, '2': 2, '1': 1 }

const gradePointsForUni = computed(() => {
  const highUni = ['hku', 'polyu']
  return highUni.includes(selectedUni.value) ? gradePointsHigh : gradePointsStandard
})

const currentScoringSystem = computed(() => {
  const highUni = ['hku', 'polyu']
  return highUni.includes(selectedUni.value) ? '8.5分制' : '7分制'
})

function gradeClass(g) {
  if (g === '5**') return 'grade-5starstar'
  if (g === '5*') return 'grade-5star'
  if (g === '5') return 'grade-5'
  if (g === '4') return 'grade-4'
  return ''
}

const best5Score = computed(() => {
  const gp = gradePointsForUni.value
  const pts = grades.value.map(g => gp[g] || 0).sort((a, b) => b - a)
  return pts.slice(0, 5).reduce((s, x) => s + x, 0) + (bonusScore.value || 0)
})

// Detect non-DSE scoring (pure numeric formula like "20.68", "16")
function isNonDSE(formula) {
  if (!formula) return false
  return /^[\d.]+$/.test(formula.trim())
}

function calcChance(score, median, lq, uq) {
  if (median == null) return 0
  const cap = (v) => Math.min(100, Math.round(v))
  if (uq != null && score >= uq + 3) return cap(75 + (score - median) * 2)
  if (score >= median) {
    const range = (uq ?? median * 1.15) - median
    return cap(65 + ((score - median) / range) * 20)
  }
  if (score >= (lq ?? median - 5)) {
    const lo = lq ?? median - 5
    const hi = median
    return cap(35 + ((score - lo) / (hi - lo || 5)) * 30)
  }
  return Math.max(5, cap((score / (lq || median)) * 15))
}

const matchResults = computed(() => {
  if (best5Score.value === 0) return { safe: [], match: [], reach: [], incompatible: [] }
  const score = best5Score.value
  const safe = [], match = [], reach = [], incompatible = []
  for (const p of programs) {
    if (p.median == null) continue
    if (p.median > 100) { incompatible.push({ ...p, chance: null, incompatible: true }); continue }
    const chance = calcChance(score, p.median, p.lq, p.uq)
    const progWithChance = { ...p, chance, incompatible: false }
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

const selectedUni = ref('')
const selectedCategory = ref('')
const searchQuery = ref('')
const sortBy = ref('median-desc')
const currentPage = ref(1)

watch([selectedUni, selectedCategory, searchQuery, sortBy], () => { currentPage.value = 1 })

const filteredPrograms = computed(() => {
  let list = programs.map(p => {
    const score = best5Score.value
    const incompatible = (p.median != null && p.median > 100) || isNonDSE(p.formula)
    const chance = score > 0 && !incompatible ? calcChance(score, p.median, p.lq, p.uq) : 0
    return { ...p, incompatible, chance }
  })
  if (selectedUni.value) list = list.filter(p => p.uni === selectedUni.value)
  if (selectedCategory.value) list = list.filter(p => p.category === selectedCategory.value)
  if (searchQuery.value.trim()) {
    const q = searchQuery.value.toLowerCase()
    list = list.filter(p => p.name.toLowerCase().includes(q) || p.code.toLowerCase().includes(q) || p.formula.toLowerCase().includes(q))
  }
  list = [...list]
  if (sortBy.value === 'chance-desc') list.sort((a, b) => (b.chance ?? 0) - (a.chance ?? 0))
  else if (sortBy.value === 'median-desc') list.sort((a, b) => (b.median ?? 0) - (a.median ?? 0))
  else if (sortBy.value === 'median-asc') list.sort((a, b) => (a.median ?? 0) - (b.median ?? 0))
  else if (sortBy.value === 'name') list.sort((a, b) => a.name.localeCompare(b.name))
  // Move non-DSE to end
  list.sort((a, b) => (a.incompatible === b.incompatible) ? 0 : a.incompatible ? 1 : -1)
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
