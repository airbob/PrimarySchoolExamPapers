<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

// --- SEO Configuration ---
useHead({
  title: 'Free Singapore Primary School Exam Papers | All Levels & Subjects',
  meta: [
    { name: 'description', content: 'Access free Singapore primary school exam papers for all levels. Download quality assessment papers for Math, Science, and English. Perfect for PSLE preparation.' },
    { name: 'keywords', content: 'Singapore primary school exam papers, PSLE practice papers, Primary 1-6 exam papers, Free Singapore school papers, 新加坡小学试卷' },
    // Open Graph
    { property: 'og:title', content: 'Free Singapore Primary School Exam Papers' },
    { property: 'og:description', content: 'Download free primary school exam papers for all levels and subjects.' },
    { property: 'og:type', content: 'website' }
  ],
  script: [
    // Google Analytics 4
    {
      src: 'https://www.googletagmanager.com/gtag/js?id=G-7WKP91PV8C',
      async: true
    },
    {
      innerHTML: `
        window.dataLayer = window.dataLayer || [];
        function gtag(){dataLayer.push(arguments);}
        gtag('js', new Date());
        gtag('config', 'G-7WKP91PV8C');
      `,
      type: 'text/javascript'
    }
  ],
  htmlAttrs: {
    lang: 'en'
  }
})

// --- Interfaces ---
interface DropdownOption {
  code: string
  name: string
}

interface DropdownData {
  Level: DropdownOption[]
  Subject: DropdownOption[]
  Type: DropdownOption[]
  Year: DropdownOption[]
  School: DropdownOption[]
}

interface ParsedPaper {
  filename: string
  levelCode: string
  schoolCode: string
  subjectCode: string
  typeCode: string
  yearCode: string
  // Display names
  levelName: string
  schoolName: string
  subjectName: string
  typeName: string
}

// --- State ---
const loading = ref(true)
const rawFiles = ref<string[]>([])
const options = ref<DropdownData>({
  Level: [],
  Subject: [],
  Type: [],
  Year: [],
  School: []
})

const filters = ref({
  Level: '0',
  School: '0',
  Subject: '0',
  Type: '0',
  Year: '0'
})

// --- Data Fetching ---
onMounted(async () => {
  try {
    const [filesRes, optionsRes] = await Promise.all([
      fetch('/json/files.json'),
      fetch('/json/dropdownOptions.json')
    ])
    
    rawFiles.value = await filesRes.json()
    options.value = await optionsRes.json()
  } catch (e) {
    console.error('Failed to load data', e)
  } finally {
    loading.value = false
  }
})

// --- Helpers ---
const getName = (category: keyof DropdownData, code: string) => {
  const found = options.value[category]?.find(opt => opt.code === code)
  return found ? found.name : code
}

// --- Computed ---
const allPapers = computed<ParsedPaper[]>(() => {
  if (loading.value) return []
  
  return rawFiles.value.map(filename => {
    // Format: Level_School_Subject_Type_Year
    const parts = filename.split('_')
    if (parts.length !== 5) return null // Skip invalid

    const [levelCode, schoolCode, subjectCode, typeCode, yearCode] = parts

    return {
      filename,
      levelCode,
      schoolCode,
      subjectCode,
      typeCode,
      yearCode,
      levelName: getName('Level', levelCode),
      schoolName: getName('School', schoolCode),
      subjectName: getName('Subject', subjectCode),
      typeName: getName('Type', typeCode)
    }
  }).filter((p): p is ParsedPaper => p !== null)
})

const filteredPapers = computed(() => {
  return allPapers.value.filter(paper => {
    const f = filters.value
    if (f.Level !== '0' && paper.levelCode !== f.Level) return false
    if (f.School !== '0' && paper.schoolCode !== f.School) return false
    if (f.Subject !== '0' && paper.subjectCode !== f.Subject) return false
    if (f.Type !== '0' && paper.typeCode !== f.Type) return false
    if (f.Year !== '0' && paper.yearCode !== f.Year) return false
    return true
  })
})

const resultCount = computed(() => filteredPapers.value.length)

const resetFilters = () => {
  filters.value = {
    Level: '0',
    School: '0',
    Subject: '0',
    Type: '0',
    Year: '0'
  }
}

const downloadPaper = (filename: string) => {
  window.open(`/files/${filename}.pdf`, '_blank')
}
</script>

<template>
  <div class="app-container">
    <!-- Header -->
    <header class="hero">
      <div class="content-wrapper">
        <h1 class="title">Singapore Primary School Exam Papers</h1>
        <p class="subtitle">Free resources for PSLE preparation. Download past year papers instantly.</p>
      </div>
    </header>

    <!-- Filters Bar -->
    <div class="filters-bar">
      <div class="content-wrapper filter-grid">
        <div class="filter-group">
          <label>Level</label>
          <select v-model="filters.Level">
            <option v-for="opt in options.Level" :key="opt.code" :value="opt.code">{{ opt.name }}</option>
          </select>
        </div>
        
        <div class="filter-group">
          <label>Subject</label>
          <select v-model="filters.Subject">
            <option v-for="opt in options.Subject" :key="opt.code" :value="opt.code">{{ opt.name }}</option>
          </select>
        </div>

        <div class="filter-group">
          <label>Year</label>
          <select v-model="filters.Year">
            <option v-for="opt in options.Year" :key="opt.code" :value="opt.code">{{ opt.name }}</option>
          </select>
        </div>

        <div class="filter-group">
          <label>Exam Type</label>
          <select v-model="filters.Type">
            <option v-for="opt in options.Type" :key="opt.code" :value="opt.code">{{ opt.name }}</option>
          </select>
        </div>

        <div class="filter-group school-select">
          <label>School</label>
          <select v-model="filters.School">
            <option v-for="opt in options.School" :key="opt.code" :value="opt.code">{{ opt.name }}</option>
          </select>
        </div>

        <button class="reset-btn" @click="resetFilters">Reset Filters</button>
      </div>
    </div>

    <!-- Main Content -->
    <main class="content-wrapper main-content">
      <div v-if="loading" class="loading-state">
        <div class="spinner"></div>
        <p>Loading exam papers...</p>
      </div>

      <div v-else>
        <div class="results-header">
          <h2>Found {{ resultCount }} Papers</h2>
        </div>

        <div v-if="resultCount === 0" class="empty-state">
          <span class="emoji">🔍</span>
          <h3>No papers found</h3>
          <p>Try adjusting your filters to find what you're looking for.</p>
          <button class="primary-btn" @click="resetFilters">Clear Filters</button>
        </div>

        <div v-else class="papers-grid">
          <div v-for="paper in filteredPapers" :key="paper.filename" class="paper-card">
            <div class="card-header">
              <span class="badge year-badge">{{ paper.yearCode }}</span>
              <span class="badge level-badge">{{ paper.levelName }}</span>
            </div>
            
            <h3 class="school-name">{{ paper.schoolName }}</h3>
            
            <div class="card-details">
              <div class="detail-row">
                <span class="label">Subject:</span>
                <span class="value">{{ paper.subjectName }}</span>
              </div>
              <div class="detail-row">
                <span class="label">Exam:</span>
                <span class="value">{{ paper.typeName }}</span>
              </div>
            </div>

            <button class="download-btn" @click="downloadPaper(paper.filename)">
              Download PDF
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>
            </button>
          </div>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer>
      <p>&copy; 2025 Dreamon.im - Singapore Primary School Exam Papers</p>
    </footer>
  </div>
</template>

<style scoped>
/* Reset & Base */
* {
  box-sizing: border-box;
}

.app-container {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  min-height: 100vh;
  background-color: #f8fafc; /* Slate 50 */
  color: #1e293b; /* Slate 800 */
  display: flex;
  flex-direction: column;
}

.content-wrapper {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1.5rem;
  width: 100%;
}

/* Hero Section */
.hero {
  background: linear-gradient(135deg, #4f46e5 0%, #3730a3 100%);
  color: white;
  padding: 4rem 0 3rem;
  text-align: center;
}

.title {
  font-size: 2.5rem;
  font-weight: 800;
  margin-bottom: 0.5rem;
  letter-spacing: -0.025em;
}

.subtitle {
  font-size: 1.125rem;
  color: #c7d2fe;
  max-width: 600px;
  margin: 0 auto;
}

/* Filters Bar */
.filters-bar {
  background: white;
  border-bottom: 1px solid #e2e8f0;
  padding: 1.5rem 0;
  position: sticky;
  top: 0;
  z-index: 10;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
}

.filter-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  align-items: end;
}

.school-select {
  grid-column: span 2;
}

@media (max-width: 768px) {
  .school-select {
    grid-column: span 1;
  }
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
}

.filter-group label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #64748b;
}

select {
  padding: 0.625rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.5rem;
  background-color: white;
  font-size: 0.95rem;
  color: #1e293b;
  width: 100%;
  transition: all 0.2s;
  cursor: pointer;
  appearance: none;
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3e%3cpath stroke='%236b7280' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3e%3c/svg%3e");
  background-position: right 0.5rem center;
  background-repeat: no-repeat;
  background-size: 1.5em 1.5em;
  padding-right: 2.5rem;
}

select:focus {
  outline: none;
  border-color: #4f46e5;
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
}

.reset-btn {
  padding: 0.625rem 1rem;
  background-color: transparent;
  border: 1px solid #cbd5e1;
  color: #64748b;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.reset-btn:hover {
  background-color: #f1f5f9;
  color: #334155;
}

/* Main Content */
.main-content {
  flex: 1;
  padding-top: 2rem;
  padding-bottom: 4rem;
}

.loading-state {
  text-align: center;
  padding: 4rem 0;
  color: #64748b;
}

.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #4f46e5;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 0 auto 1rem;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.results-header {
  margin-bottom: 1.5rem;
}

.results-header h2 {
  font-size: 1.25rem;
  color: #334155;
  font-weight: 600;
}

/* Papers Grid */
.papers-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
}

.paper-card {
  background: white;
  border-radius: 1rem;
  padding: 1.5rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
  display: flex;
  flex-direction: column;
  position: relative;
  overflow: hidden;
  border: 1px solid #f1f5f9;
}

.paper-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  border-color: #e2e8f0;
}

.card-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 1rem;
}

.badge {
  font-size: 0.75rem;
  font-weight: 700;
  padding: 0.25rem 0.6rem;
  border-radius: 9999px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.year-badge {
  background-color: #e0e7ff;
  color: #4338ca;
}

.level-badge {
  background-color: #f1f5f9;
  color: #475569;
}

.school-name {
  font-size: 1.125rem;
  font-weight: 700;
  color: #1e293b;
  margin: 0 0 1rem;
  line-height: 1.4;
  flex-grow: 1; /* Pushes content down */
}

.card-details {
  margin-bottom: 1.5rem;
  font-size: 0.9rem;
  color: #64748b;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.25rem;
}

.detail-row .value {
  font-weight: 500;
  color: #334155;
}

.download-btn {
  width: 100%;
  padding: 0.75rem;
  background-color: #4f46e5;
  color: white;
  border: none;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  transition: background-color 0.2s;
}

.download-btn:hover {
  background-color: #4338ca;
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 4rem 1rem;
  background: white;
  border-radius: 1rem;
  border: 2px dashed #e2e8f0;
}

.emoji {
  font-size: 3rem;
  margin-bottom: 1rem;
  display: block;
}

.empty-state h3 {
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
  font-weight: 600;
}

.empty-state p {
  color: #64748b;
  margin-bottom: 1.5rem;
}

.primary-btn {
  background-color: #4f46e5;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
}

/* Footer */
footer {
  background-color: white;
  border-top: 1px solid #e2e8f0;
  padding: 2rem 0;
  text-align: center;
  color: #94a3b8;
  font-size: 0.875rem;
}

/* Responsive */
@media (max-width: 640px) {
  .title { font-size: 1.875rem; }
  .filter-grid { grid-template-columns: 1fr; }
  .school-select { grid-column: span 1; }
}
</style>
