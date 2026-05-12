<template>
  <div class="container">

    <!-- Header -->
    <header>
      <div class="header-title">JUPAS 2025 <span>智能配對</span></div>
      <div class="header-sub">輸入 HKDSE 成績，計算升學機會</div>

      <!-- Tab Bar -->
      <div class="tab-bar">
        <button class="tab-btn" :class="{ active: activeTab === 'match' }" @click="activeTab = 'match'">
          &#x1F3AF; 配對
        </button>
        <button class="tab-btn" :class="{ active: activeTab === 'interview' }" @click="activeTab = 'interview'">
          &#x1F4AC; 面試
        </button>
      </div>
    </header>

    <!-- ════ MATCH TAB ════ -->
    <template v-if="activeTab === 'match'">

      <!-- Calculator -->
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
                  <span class="tag incompatible" v-if="p.incompatible">⚠ {{ p.formula }}</span>
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

        <!-- Reach Cards -->
        <div v-if="best5Score > 0 && matchResults.reach.length">
          <div class="match-group-title reach">&#x1F525; 衝刺 — 低於中位分</div>
          <div v-for="p in matchResults.reach.slice(0,6)" :key="p.code+p.uni" class="prog-card reach">
            <div class="prog-card-top">
              <div class="prog-card-left">
                <span class="uni-badge" :style="{background: getUni(p.uni).color+'18', color: getUni(p.uni).color}">
                  {{ getUni(p.uni).name }}
                </span>
                <div class="prog-name">{{ p.name }}</div>
                <div class="prog-tags">
                  <span class="tag">{{ p.category }}</span>
                  <span class="tag interview" v-if="p.interview">&#x1F4AC; 面試</span>
                </div>
              </div>
              <div class="prog-card-right">
                <div class="chance-big" :style="{color: chanceColor(p.chance)}">{{ p.chance }}%</div>
                <div class="chance-label-sm">機會</div>
              </div>
            </div>
            <div class="prog-card-scores">
              <div class="sc-item">
                <span class="sc-lbl">&#x25CF; 中位</span>
                <span class="sc-val" :class="scoreClass(p.median)">{{ p.median ?? '—' }}</span>
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

      <!-- ── Browser ── -->
      <section class="browser-section">
        <div class="section-title">&#x1F50D; 瀏覽所有課程</div>

        <div class="filter-row">
          <select v-model="selectedUni" class="uni-selector">
            <option value="">全部大學</option>
            <option v-for="u in universities" :key="u.id" :value="u.id">{{ u.name }}</option>
          </select>
          <select v-model="selectedCategory" class="uni-selector">
            <option value="">全部類別</option>
            <option v-for="c in categories" :key="c" :value="c">{{ c }}</option>
          </select>
        </div>

        <div class="filter-row">
          <input v-model="searchQuery" class="search-input" placeholder="搜尋課程名稱或代碼...">
          <select v-model="sortBy" class="sort-select">
            <option value="median-desc">分數 高→低</option>
            <option value="median-asc">分數 低→高</option>
            <option value="chance-desc">機會 高→低</option>
            <option value="name">名稱 A→Z</option>
          </select>
        </div>

        <div class="browser-stats" v-if="filteredPrograms.length > 0">
          共 {{ filteredPrograms.length }} 個課程
          <span v-if="best5Score > 0" class="your-score-badge">你的分數：{{ best5Score.toFixed(1) }}</span>
        </div>

        <div v-for="p in paginatedPrograms" :key="p.code+p.uni" class="prog-card" :class="{ 'incompatible-card': p.incompatible }">
          <div class="prog-card-top">
            <div class="prog-card-left">
              <span class="uni-badge" :style="{background: getUni(p.uni).color+'18', color: getUni(p.uni).color}">
                {{ getUni(p.uni).name }}
              </span>
              <div class="prog-name">{{ p.name }}</div>
              <div class="prog-tags">
                <span class="tag">{{ p.category }}</span>
                <span class="tag years" v-if="p.years !== 4">{{ p.years }}年</span>
                <span class="tag incompatible" v-if="p.incompatible">⚠ {{ p.formula }}</span>
                <span class="tag interview" v-if="p.interview && !p.incompatible">&#x1F4AC; 面試</span>
              </div>
            </div>
            <div class="prog-card-right" v-if="!p.incompatible && best5Score > 0">
              <div class="chance-big" :style="{color: chanceColor(p.chance)}">{{ p.chance }}%</div>
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
            <div class="sc-item" v-if="p.lq">
              <span class="sc-lbl">&#x25BC; 下四分</span>
              <span class="sc-val ok">{{ p.lq }}</span>
            </div>
            <div class="sc-item">
              <span class="sc-lbl">計分</span>
              <span class="sc-val dim">{{ p.formula }}</span>
            </div>
          </div>
          <div class="chance-bar-wrap" v-if="!p.incompatible && best5Score > 0">
            <div class="chance-bar-bg">
              <div class="chance-bar" :style="{width: p.chance+'%', background: chanceColor(p.chance)}"></div>
            </div>
          </div>
        </div>

        <div v-if="filteredPrograms.length === 0" class="empty-state">
          <div class="empty-icon">&#x1F50D;</div>
          <div class="empty-title">未找到課程</div>
          <div class="empty-sub">嘗試調整篩選條件</div>
        </div>

        <div class="pagination" v-if="totalPages > 1">
          <button class="page-btn" :disabled="currentPage===1" @click="currentPage--">&#x276E;</button>
          <button v-for="n in pageRange" :key="n" class="page-btn" :class="{active: n===currentPage}" @click="currentPage=n">{{ n }}</button>
          <button class="page-btn" :disabled="currentPage>=totalPages" @click="currentPage++">&#x276F;</button>
        </div>
      </section>

    </template>

    <!-- ════ INTERVIEW TAB ════ -->
    <template v-if="activeTab === 'interview'">

      <section class="interview-hero">
        <div class="interview-hero-title">&#x1F4AC; 面試題目庫</div>
        <div class="interview-hero-sub">共 {{ totalInterviewQuestions }} 條題目 · 涵蓋 DSE / 大學 / Marketing</div>
        <div class="iq-filter-row">
          <button v-for="cat in interviewCategories" :key="cat.key"
            class="iq-cat-btn" :class="{ active: selectedInterviewCat === cat.key }"
            @click="selectedInterviewCat = cat.key">
            {{ cat.emoji }} {{ cat.label }}
          </button>
        </div>
      </section>

      <section class="iq-section" v-for="group in filteredInterviewGroups" :key="group.key">
        <div class="iq-group-title">{{ group.emoji }} {{ group.label }}</div>
        <div v-for="q in group.questions" :key="q.id" class="iq-card">
          <div class="iq-card-q">{{ q.id }}. {{ q.text }}</div>
          <div class="iq-card-tip" v-if="q.tip">&#x1F4A1 {{ q.tip }}</div>
          <div class="iq-card-example" v-if="q.example">&#x270F 例：{{ q.example }}</div>
        </div>
      </section>

      <section class="iq-star-section">
        <div class="iq-star-title">&#x1F3AF; STAR 答題格式</div>
        <div class="iq-star-grid">
          <div class="iq-star-item">
            <div class="iq-star-letter">S</div>
            <div class="iq-star-desc">Situation<br><small>設定場景</small></div>
          </div>
          <div class="iq-star-item">
            <div class="iq-star-letter">T</div>
            <div class="iq-star-desc">Task<br><small>你的目標</small></div>
          </div>
          <div class="iq-star-item">
            <div class="iq-star-letter">A</div>
            <div class="iq-star-desc">Action<br><small>具體行動</small></div>
          </div>
          <div class="iq-star-item">
            <div class="iq-star-letter">R</div>
            <div class="iq-star-desc">Result<br><small>結果（最好有數字）</small></div>
          </div>
        </div>
      </section>

      <section class="iq-checklist-section">
        <div class="iq-check-title">&#x2705; 面試準備檢查清單</div>
        <div class="iq-check-item" v-for="item in checklist" :key="item">
          <span class="iq-check-box">&#x25FB</span> {{ item }}
        </div>
      </section>

    </template>

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

// ── Tab System ──
const activeTab = ref('match')

// ── Interview Questions ──
const interviewCategories = [
  { key: 'all', label: '全部', emoji: '📋' },
  { key: 'dse', label: 'DSE/大學', emoji: '🎓' },
  { key: 'marketing', label: 'Marketing', emoji: '📊' },
  { key: 'generic', label: '通用技能', emoji: '💼' },
  { key: 'medicine', label: '健康醫學', emoji: '🏥' },
  { key: 'law', label: '法學', emoji: '⚖️' },
  { key: 'arch_eng', label: '建築/工程', emoji: '🏗️' },
  { key: 'social', label: '社科', emoji: '🧑‍⚖️' },
  { key: 'speech', label: '演說題', emoji: '🎤' },
]

const selectedInterviewCat = ref('all')

const checklist = [
  '1 分鐘自我介紹（背熟）',
  '3 分鐘自我介紹（靈活運用）',
  '為每個學系準備「點解報讀」答案',
  '研究公司 / 院校背景',
  '準備 2-3 個具體事例（STAR 格式）',
  '準備問題問面試官（至少 2 條）',
  '穿著正式服裝',
  '練習普通話自我介紹（如需要）',
]

const interviewData = [
  {
    key: 'dse', label: 'DSE / 大學面試', emoji: '🎓',
    questions: [
      { id: 1, text: '請用 1 分鐘自我介紹', tip: '重點：興趣、成就、與學科的連結' },
      { id: 2, text: '請用 3 分鐘深度自我介紹', tip: '包含學業、課外活動、未來志向三部分' },
      { id: 3, text: '你嘅朋友通常點形容你？', tip: '用實際例子證明，唔好只係形容詞' },
      { id: 4, text: '你有咩課外活動？邊個對你影響最深？', tip: '盡量具體，講你學到咩' },
      { id: 5, text: '你高中最深刻嘅學習經歷係咩？', tip: '可以係一個 project、比賽或克服困難的經歷' },
      { id: 6, text: '點解你報讀呢個學系/科目？', tip: '必須具體，唔好只係「有興趣」' },
      { id: 7, text: '你對呢個學科有咩理解？', tip: '準備具體例子，顯示你做過功課' },
      { id: 8, text: '你嘅志願/職業志向係咩？', tip: '盡量與學科相關聯' },
      { id: 9, text: '你點解覺得自己適合呢個課程？', tip: '結合你嘅強項同課程特色' },
      { id: 10, text: '你 5 年後 / 10 年後會喺咩位置？', tip: '顯示你有規劃，但唔好太具體以免限制' },
      { id: 11, text: '你對香港教育制度有咩睇法？', tip: '批評要建設性，提出改善建議' },
      { id: 12, text: '你最近關注咩時事議題？（最好與學科相關）', tip: '準備 2-3 個可持續談論的話題' },
      { id: 13, text: '你讀過咩書或文章影響到你嘅想法？', tip: '準備 1-2 本書，識得討論內容' },
      { id: 14, text: '你認為香港面臨最大嘅挑戰係咩？', tip: '顯示批判思考，但避免負面抱怨' },
      { id: 15, text: '如果你可以改變香港教育制度嘅一樣嘢，你會改咩？', tip: '要有理有據，唔好只係抱怨' },
      { id: 16, text: '你遇到最大嘅挫折係咩？你點面對？', tip: '重點係你點成長，唔係挫折有幾慘' },
      { id: 17, text: '你嘅強項和弱項係咩？', tip: '弱項要話你點改善，唔好話「我好完美」' },
      { id: 18, text: '你面試失敗過嗎？點處理？', tip: '顯示你從失敗中學習' },
      { id: 19, text: '你點樣學習新技能？举个例子', tip: '具體描述學習過程和方法' },
      { id: 20, text: '你最近做過咩有意義嘅決定？', tip: '顯示你嘅價值觀和思考方式' },
    ]
  },
  {
    key: 'marketing', label: 'Marketing / Digital Marketing', emoji: '📊',
    questions: [
      { id: 21, text: '你點解想讀 / 做 Marketing？', tip: '展示你對行業的理解同熱情' },
      { id: 22, text: '你對 Digital Marketing 有咩理解？', tip: '涵蓋 SEO/SEM、社交媒體、內容營銷等' },
      { id: 23, text: '傳統 Marketing 同 Digital Marketing 有咩分別？', tip: '從渠道、測量、精準度等角度分析' },
      { id: 24, text: '你關注咩 marketing trends？', tip: '2024-2025 重點：AI營銷、短影片、私域流量' },
      { id: 25, text: '你知唔知咩係 SEO / SEM？', tip: '準備基本定義 + 一個實際例子' },
      { id: 26, text: '你用過 Google Ads 或 Facebook Ads 嗎？', tip: '最好有實際操作經驗，哪怕係個人project' },
      { id: 27, text: '你點樣量度一個 campaign 成功與否？（KPI 設定）', tip: 'CTR、CPC、Conversion Rate、ROAS 等' },
      { id: 28, text: '你用過咩 social media platforms？分析下佢哋嘅用家特徵', tip: '每個平台優缺點都要識比較' },
      { id: 29, text: '你有冇做過任何 Marketing 相關嘅 project 或實習？', tip: '用 STAR 格式描述，側重你的具體角色' },
      { id: 30, text: '如果比你規劃一個產品嘅推廣策略，你會點做？', tip: '先確定目標受眾，再選擇渠道' },
      { id: 31, text: '介紹一個你認為好成功嘅 marketing campaign', tip: '解釋點解成功，適用於邊個品牌/產品' },
      { id: 32, text: '如果每月預算得 HK$10,000，你會點分配？', tip: '展示預算分配嘅思考過程' },
      { id: 33, text: '你點解想加入我哋公司？', tip: '必須研究過公司準備，唔好只係答「因為你哋係大公司」' },
      { id: 34, text: '你認為我哋品牌可以點樣改善？', tip: '提出具體建議，顯示你有用戶思維' },
      { id: 35, text: '假設你要為一間新開嘅咖啡店做推廣，你會點做？', tip: '從品牌定位、目標受眾、渠道選擇、預算分配角度' },
      { id: 36, text: '如果一個廣告嘅 CTR 很低，你會點分析同改善？', tip: '從受眾定向、創意設計、着陸頁、競價策略角度' },
      { id: 37, text: 'KOL/KOC 推廣同傳統廣告邊個更有效？', tip: '視乎產品、受眾、預算，兩者各有優勢' },
    ]
  },
  {
    key: 'generic', label: '通用求職 / 生存技能', emoji: '💼',
    questions: [
      { id: 38, text: '你嘅優點係咩？舉例說明', tip: '用實際事例證明，唔好只係形容詞' },
      { id: 39, text: '你嘅缺點係咩？你點克服？', tip: '選擇唔影響核心能力的缺點，側重改善方法' },
      { id: 40, text: '你係團隊入面通常扮演咩角色？', tip: 'Leader / Facilitator / Executor，提前準備例子' },
      { id: 41, text: '你創新嘅經歷係點嘅？', tip: '具體描述問題、你的想法、行動、結果' },
      { id: 42, text: '你遇到壓力時點處理？', tip: '展示你的情緒管理能力' },
      { id: 43, text: '你點樣同一個好難相處嘅同事合作？', tip: '顯示你的人際技巧和成熟度' },
      { id: 44, text: '你點解離開上一份工 / 上一間學校？', tip: '千祈唔好負面批評，側重你追求的新方向' },
      { id: 45, text: '你收過最難聽嘅 feedback 係咩？你點反應？', tip: '顯示你能接受批評並成長' },
      { id: 46, text: '你有一個什麼樣的領導經驗？', tip: '用 STAR 格式，最好有量化結果' },
      { id: 47, text: '你點樣激勵一個動力低嘅團隊？', tip: '展示你的領導風格和實際做法' },
      { id: 48, text: '你點樣做決定？傾一個你做過嘅艱難決定', tip: '展示你的決策框架同價值觀' },
    ]
  },
  {
    key: 'speech', label: '演說面試題 (SINGKING)', emoji: '🎤',
    questions: [
      { id: 49, text: '請就以下議題作 2 分鐘演說：「香港學生需要培養國際視野」', tip: '開場立場鮮明，中段有例子，結尾總結' },
      { id: 50, text: '如果你可以帶一樣嘢去荒島，你會帶咩？', tip: '考創意同價值觀，解釋原因' },
      { id: 51, text: '你認為金錢係唔係快樂嘅來源？', tip: '展示批判思考，唔好只答係或唔係' },
      { id: 52, text: '香港應唔應該繼續發展旅遊業？', tip: '包含經濟、環境、民生多角度分析' },
      { id: 53, text: '你認為 AI 會取代幾多人類工作？', tip: '展示你對科技趨勢的理解同個人觀點' },
    ]
  },


  {
    key: 'medicine', label: '健康醫學面試題目', emoji: '🏥',
    questions: [
      { id: 54, text: '點解你想讀醫科/護理/牙科？呢個決定係幾時做嘅？', tip: '顯示你對呢個專業有深刻理解，而唔係一時興起' },
      { id: 55, text: '你點睇香港公立醫院嘅問題？你有咩改善建議？', tip: '顯示你關注醫療系統，但避免過度負面批評' },
      { id: 56, text: '你有冇喺醫院或診所做過志願者或實習？你學到咩？', tip: '具體描述經歷，唔好只係列出工作內容' },
      { id: 57, text: '你點樣平衡工作同生活？醫療工作壓力好大，你點應付？', tip: '顯示你知道行業挑戰，並有實際應對方法' },
      { id: 58, text: '你認為醫療科技（AI、遠程醫療）會點改變醫療行業？', tip: '顯示你對醫療趨勢嘅認識同個人觀點' },
      { id: 59, text: '你身邊有人係醫護人員嗎？佢哋嘅經歷有冇影響到你？', tip: '側面展示你對呢個行業嘅接觸同了解' },
      { id: 60, text: '如果病人拒絕治療，你會點做？', tip: '測試你嘅溝通能力同倫理判斷' },
      { id: 61, text: '中醫同西醫，你點睇兩者嘅關係？你點解報讀中醫？', tip: '特別針對中醫學士，面試官好重視你對中西醫結合嘅理解' },
      { id: 62, text: '你有冇讀過任何關於醫學或健康嘅書或文章？', tip: '準備 1-2 本具體書名，顯示你有持續關注' },
      { id: 63, text: '你認為一個好醫生/護士需要具備咩素質？', tip: '聯繫到自己嘅強項，用具體例子說明' },
      { id: 64, text: '物理治療師 / 職業治療師嘅工作係咩？你點解想讀呢個專科？', tip: '針對物理治療、職業治療面試，顯示你知道日常工作' },
    ]
  },
  {
    key: 'law', label: '法學面試題目', emoji: '⚖️',
    questions: [
      { id: 65, text: '點解你想讀法律？你對法律有咩理解？', tip: '顯示你唔只係因為高薪或父母期望而讀' },
      { id: 66, text: '你最近關注咩法律議題或法庭案件？', tip: '準備 1-2 個近期案件，顯示你關心時事同法律發展' },
      { id: 67, text: '你認為香港嘅法治面臨咩挑戰？', tip: '批評要有建設性，唔好過度政治化' },
      { id: 68, text: '你讀過咩法律相關嘅書？（建議：《洞穴奇案》、《正義的價值》）', tip: '顯示你對法律哲學有興趣，而唔只係 technical knowledge' },
      { id: 69, text: '如果法律同道德衝突，你會點取捨？', tip: '經典法律哲學問題，測試你嘅批判思考' },
      { id: 70, text: '你點解報讀雙學位？你點規劃你嘅學習？', tip: '針對雙學位申請者，顯示你有清晰規劃' },
      { id: 71, text: '你有冇參加過模擬法庭、辯論賽或法律相關活動？', tip: '具體描述你嘅角色同學習到嘅技能' },
      { id: 72, text: '你認為律師嘅社會責任係咩？', tip: '顯示你對法律職業嘅全面理解' },
      { id: 73, text: '普通法同大陸法有咩分別？你點解想讀普通法系？', tip: '特別針對香港法律教育，顯示你知道香港法律制度基礎' },
      { id: 74, text: '如果有一個你認為有罪嘅人喺法庭被判無罪，你點諗？', tip: '測試你對法律原則（無罪推定、程序正義）嘅理解' },
    ]
  },
  {
    key: 'arch_eng', label: '建築 / 工程面試題目', emoji: '🏗️',
    questions: [
      { id: 75, text: '你最近關注咩建築設計或城市規劃議題？', tip: '準備 1-2 個具體項目或爭議，顯示你對行業嘅認識' },
      { id: 76, text: '你認為香港城市規劃面臨咩最大挑戰？', tip: '可以提及房屋問題、棕地、綠化等' },
      { id: 77, text: '你有冇做過任何建築或設計相關嘅 project？', tip: 'Portfolio 準備好，具體描述你嘅貢獻' },
      { id: 78, text: '你點解想讀建築？你認為建築師嘅角色係咩？', tip: '顯示你唔只係因為興趣，仲理解專業責任' },
      { id: 79, text: '你點睇「建築係藝術定實用」呢個議題？', tip: '展示你對美學同功能性平衡嘅理解' },
      { id: 80, text: '你有冇留意過香港邊個建築項目你最深刻？點解？', tip: '準備 1-2 個具體例子，解釋原因' },
      { id: 81, text: '可持續發展建築係咩？你點睇綠色建築喺香港嘅前景？', tip: '顯示你關注最新行業趨勢' },
      { id: 82, text: '你會用咩軟件或工具做設計？（如 SketchUp、AutoCAD、Rhino）', tip: '具備基本技術能力係優勢，但誠實回答' },
    ]
  },
  {
    key: 'social', label: '社科 / 犯罪學面試題目', emoji: '🧑‍⚖️',
    questions: [
      { id: 83, text: '你點解想讀犯罪學？你對犯罪學有咩理解？', tip: '顯示你唔只係因為電視劇而感興趣' },
      { id: 84, text: '你認為香港犯罪問題有咩趨勢？你點睇青少年犯罪？', tip: '顯示你對本地社會問題嘅關注' },
      { id: 85, text: '你讀過咩關於犯罪學或社會學嘅書？', tip: '準備具體書單，顯示你有持續閱讀' },
      { id: 86, text: '罪犯係天生定後天？你點睇遺傳同環境對犯罪行為嘅影響？', tip: '經典犯罪學辯論題，顯示你嘅分析能力' },
      { id: 87, text: '你認為刑罰嘅目的係咩？懲罰，定預防，定康復？', tip: '顯示你對刑事司法制度嘅思考' },
    ]
  },]

const filteredInterviewGroups = computed(() => {
  if (selectedInterviewCat.value === 'all') return interviewData
  return interviewData.filter(g => g.key === selectedInterviewCat.value)
})

const totalInterviewQuestions = computed(() => {
  return interviewData.reduce((sum, g) => sum + g.questions.length, 0)
})

// ── Existing JUPAS Logic ──

const categories = [...new Set(programs.map(p => p.category))].sort()

const subjects = [
  { label: '中文' }, { label: '英文' }, { label: '數學' },
  { label: '選修1' }, { label: '選修2' }, { label: '選修3' }, { label: '選修4' },
]
const grades = ref(['', '', '', '', '', '', ''])
const bonusScore = ref(0)

const gradePointsHigh = { '5**': 8.5, '5*': 7, '5': 6, '4': 4, '3': 3, '2': 2, '1': 1 }
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
