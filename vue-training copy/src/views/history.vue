<template>
<div class="history-wrapper" style = "margin-top: 45px;">
  <h2 class="page-title">Your Prediction History</h2>

    <div class="history-toolbar">
        <input type="date" v-model="startDate" />
        <input type="date" v-model="endDate" />

        <!-- <button class="search-btn" @click="filterByDate">🔍</button> -->
        <button class="overall-btn" @click="handleOpenOverallDetail">Detail Information</button>
        <button class="heart-view-btn" @click="showHeartModal = true">View Heart</button>
    </div>

    <div v-if="showHeartModal" class="modal"
    @mousedown="startDrag($event, 'heart', 0)"
    :style="{ 
        width: heartModalState.width + 'px', 
        height: heartModalState.height + 'px', 
        left: heartModalState.left + 'px', 
        top: heartModalState.top + 'px', 
        zIndex: heartModalState.zIndex 
    }">
    
      <button class="close-btn" @click="showHeartModal = false">✕</button>
      <h3 class="modal-header">Heart Reference</h3>
      
      <div class="img-container" style="flex: 1; overflow: hidden; display: flex; align-items: center; justify-content: center;">
          <img :src="heartImg" alt="Heart Diagram" class="reference-img" style="max-height: 100%; object-fit: contain;" @mousedown.stop />
      </div>

      <div class="resizer" @mousedown.prevent="startResize($event, 'heart', 0)"></div>
    </div>

    <div v-if="loading" class="loading-overlay">Loading...</div>
    <div v-if="errorMsg" class="message error-message">{{ errorMsg }}</div>

    <div v-if="!loading && (!displayedReports || displayedReports.length === 0)" class="no-history">
    No history yet. Go make your first prediction!
    </div>

    <!-- 單筆報告 -->
    <div v-for="riskReport in displayedReports" :key="riskReport.id" class="report-card">
      <button class="graph-btn" @click="openSingleGraph(riskReport)">Graph</button>

      <p><strong>Date:</strong> {{ formatDate(riskReport.created_at) }}</p>
      <p><strong>Age:</strong> {{ riskReport.input_data.age }}</p>
      <p><strong>Gender:</strong> {{ riskReport.input_data.gender }}</p>
      <p><strong>Medical History:</strong> {{ riskReport.input_data.medical_history }}</p>

      <h3 class="title">
          Possible Diseases 
          <small class="risk-guide">
          (<span class="high-text"> >60% High Risk</span> | 
          <span class="medium-text">30~60% Medium Risk</span> )
          </small>
      </h3>

      <div v-if="riskReport.merged_diseases.filter(d => d?.probability >= 30).length > 0">
          <ul>
          <li v-for="(disease,i) in riskReport.merged_diseases.filter(d => d?.probability >= 30)" 
              :key="i"
              :class="{
                  'high-text': disease.probability >= 60,
                  'medium-text': disease.probability >= 30 && disease.probability < 60
              }">
              {{ disease.name }} — {{ disease.probability }}%
          </li>
          </ul>
      </div>

      <p v-else class="low-text">
          ✅ No significant cardiovascular risk detected at this time.
      </p>

      <h3 class="title">Summary</h3>
      <p class="summary-text">{{ riskReport.llm_report.summary }}</p>

      <h3 class="title">Recommendations</h3>
      <ul>
          <li v-for="(item,i) in formatRecommendations(riskReport.llm_report.recommendations)" :key="i" class="recommendation-text">
          {{ item }}
          </li>
      </ul>
    </div>

    <!-- 單筆 modal -->
    <div v-for="(report,index) in singleModals" :key="index" class="modal"
    :ref="el => singleModalRefs[index] = el"
    @mousedown="startDrag($event,'single',index)"
    :style="{ width: report.width+'px', height: report.height+'px', left: report.left+'px', top: report.top+'px', fontSize: report.fontSize+'px', zIndex: report.zIndex }">
    
      <button class="close-btn" @click="closeSingleGraph(index)">✕</button>
      <h3 style="color: #000000; text-align: center; font-size: 1.2rem; font-weight: 600;">
          Report Graph
      </h3>

      <!-- <div class="category-buttons">
          <button v-for="disease in report.report.merged_diseases" :key="disease.name"
                  :class="{ 'active-btn': report.selectedCategory === disease.name }"
                  @click="selectCategory(disease.name,index)">
          {{ disease.name }}
          </button>
      </div> -->

      <div class="canvas-wrapper">
          <canvas :ref="el => report.canvasRef = el"></canvas>
      </div>

      <p class="risk-legend">
          <span class="high-text">>60% High Risk </span><span style="color:black;">| </span> 
          <span class="medium-text">30~60% Medium Risk </span>
          <!-- <span class="low-text"><30% Low Risk</span> -->
      </p>

      <div class="resizer" @mousedown.prevent="startResize($event,'single',index)"></div>
    </div>

    <!-- 綜合 modal -->
    <div v-for="(detail, index) in overallDetails" :key="'overall-' + index" class="modal"
      :ref="el => overallModalRefs[index] = el"
      @mousedown="startDrag($event, 'overall', index)"
      :style="{ width: detail.width + 'px', height: detail.height + 'px', left: detail.left + 'px', top: detail.top + 'px', zIndex: detail.zIndex }">

      <button class="close-btn" @click="closeOverallDetail(index)">✕</button>
      <h3 class="modal-header">Health Analysis Insight</h3>

      <div class="overall-content" style="overflow:auto; flex:1; padding:15px;">
          <div v-for="(disease, i) in detail.data.diseases" :key="i" class="disease-card">
            <h4 class="disease-name-title">{{ disease.name }}</h4>
            
            <div class="info-group">
                <label>💡 Why it occurs (Cause):</label>
                <p>{{ disease.cause }}</p>
            </div>

            <div class="info-group">
                <label>🚀 Why it matters (Importance):</label>
                <p>{{ disease.importance }}</p>
            </div>
            <hr class="divider" />
          </div>

          <div v-if="detail.data.general_note" class="note-section">
            <strong>Note:</strong> {{ detail.data.general_note }}
          </div>
      </div>

      <div class="resizer" @mousedown.prevent="startResize($event, 'overall', index)"></div>
  </div>
</div>
</template>

<script setup>
import { supabase } from '@/supabase'
import Chart from 'chart.js/auto'
import axios from 'axios'
import { ref, onMounted, nextTick, watch } from 'vue'
import heartImg from '@/assets/heart.jpg'

/* ---------------- Data ---------------- */
const riskReports = ref([])
const displayedReports = ref([])
const loading = ref(false)
const errorMsg = ref(null)

/* ---------------- 單筆 modal ---------------- */
const singleModals = ref([])
const singleModalRefs = ref([])
// const categories = []

const showHeartModal = ref(false)

// 在 script setup 內加入
const heartModalState = ref({
    width: 450,
    height: 400,
    left: 300,
    top: 200,
    zIndex: 1000,
    aspectRatio: 450 / 400 // 鎖定比例，縮放才不會變形
});

/* ---------------- 綜合 modal ---------------- */
const overallDetails = ref([])
const overallModalRefs = ref([])

/* ---------------- 拖拉 & 調整大小 ---------------- */
let dragInfo = null
let resizeInfo = null
let currentZIndex = 1000
const getNextZIndex = () => ++currentZIndex

/* ---------------- 日期篩選 ---------------- */
const startDate = ref(null)
const endDate = ref(null)

const BACKEND_URL = 'http://localhost:8000'

/* ---------------- 合併 LLM + Rule ---------------- */
const mergeReports = (report) => {
const llm = report.llm_report?.possible_diseases || []
const rule = report.rule_report?.possible_diseases || []

const map = {}

llm.forEach(d => {
    map[d.name] = {
    name: d.name,
    probability: d.probability
    }
})

rule.forEach(d => {
    if (map[d.name]) {
    map[d.name].probability = Math.max(map[d.name].probability, d.probability)
    } else {
    map[d.name] = {
        name: d.name,
        probability: d.probability
    }
    }
})

return Object.values(map)
}

/* ---------------- 功能函數 ---------------- */
const fetchHistory = async () => {
loading.value = true
errorMsg.value = null
try {
    const { data: { session }, error: sessionError } = await supabase.auth.getSession()
    if(sessionError || !session) throw new Error('Please log in first')
    const userId = session.user.id
    const { data, error } = await supabase
    .from('risk_reports')
    .select('*')
    .eq('user_id', userId)
    .order('created_at',{ascending:false})
    if(error) throw error

    riskReports.value = (data || []).map(r => ({
    ...r,
    merged_diseases: mergeReports(r)
    }))
    displayedReports.value = riskReports.value
} catch(err) {
    console.error(err)
    errorMsg.value = err.message || 'Failed to fetch history'
} finally {
    loading.value = false
}
}

const formatDate = (timestamp) => {
if(!timestamp) return ''
const date = new Date(timestamp)
return date.toLocaleDateString('en-US',{year:'numeric',month:'short',day:'numeric',hour:'2-digit',minute:'2-digit'})
}

const formatRecommendations = (rec) => {
if(!rec) return []
if(Array.isArray(rec)) return rec
if(typeof rec==='string') return rec.split('\n').filter(r=>r.trim()!=='')
return []
}

watch([startDate, endDate], () => {
console.log("日期變動，自動執行過濾...");
filterByDate();
});

/* ---------------- 日期篩選 (字串比對強效版) ---------------- */
const filterByDate = () => {
if (!startDate.value && !endDate.value) {
    displayedReports.value = riskReports.value;
    return;
}

displayedReports.value = riskReports.value.filter(r => {
    const reportDate = new Date(r.created_at); // 這是資料庫的 UTC 時間
    
    // 設定開始日期的最開端 (00:00:00)
    let startLimit = null;
    if (startDate.value) {
    startLimit = new Date(startDate.value);
    startLimit.setHours(0, 0, 0, 0);
    }

    // 設定結束日期的最末端 (23:59:59)
    let endLimit = null;
    if (endDate.value) {
    endLimit = new Date(endDate.value);
    endLimit.setHours(23, 59, 59, 999);
    }

    // 進行比較
    if (startLimit && reportDate < startLimit) return false;
    if (endLimit && reportDate > endLimit) return false;
    return true;
})
}

/* ---------------- 單筆圖表 ---------------- */
const openSingleGraph = async report => {
const width = 500, height = 350
const left = (window.innerWidth-width)/2
const top = (window.innerHeight-height)/2

singleModals.value.push({
    report,
    width,
    height,
    left,
    top,
    fontSize:14,
    // selectedCategory: report.merged_diseases[0]?.name || '',
    canvasRef:null,
    chartInstance:null,
    zIndex: getNextZIndex(),
    aspectRatio: width/height
})

await nextTick()
drawSingleChart(singleModals.value.length-1)
}

const closeSingleGraph = index=>{
const m = singleModals.value[index]
if(m.chartInstance) m.chartInstance.destroy()
singleModals.value.splice(index,1)
}

// const selectCategory = (cat,index)=>{
//   singleModals.value[index].selectedCategory = cat
//   drawSingleChart(index)
// }

const drawSingleChart = index => {
const modal = singleModals.value[index]
if (!modal.report || !modal.canvasRef) return

const ctx = modal.canvasRef.getContext('2d')

// ✅ 只顯示 >= 30%（中高風險）
const validDiseases = modal.report.merged_diseases
    .filter(d => d.probability >= 30)

if (validDiseases.length === 0) {
    console.warn("No medium/high risk diseases to display.")
    return
}

const labels = validDiseases.map(d => d.name)
const values = validDiseases.map(d => d.probability)
const colors = validDiseases.map(d => getSeverityColor(d.probability))

if (modal.chartInstance) modal.chartInstance.destroy()

modal.chartInstance = new Chart(ctx, {
    type: 'bar',   // ✅ 永遠長條圖
    data: {
    labels,
    datasets: [{
        label: 'Probability (%)',
        data: values,
        backgroundColor: colors,
        borderRadius: 6,
        barThickness: 28
    }]
    },
    options: {
    responsive: true,
    maintainAspectRatio: false,
    scales: {
        y: {
        beginAtZero: true,
        max: 100,
        ticks: {
            stepSize: 10,
            callback: v => `${v}%`
        }
        }
    },
    plugins: {
        legend: { display: false },
        tooltip: {
        callbacks: {
            label: ctx => `${ctx.parsed.y}%`
        }
        }
    }
    }
})
}

/* ---------------- 綜合分析 (Overall Result) ---------------- */
const handleOpenOverallDetail = async () => {
// 這裡已經不需要再呼叫 filterByDate()，因為 watch 已經幫你做好了
if (displayedReports.value.length === 0) {
    errorMsg.value = "Selected range has no reports to analyze.";
    return;
}

loading.value = true;
errorMsg.value = null;

try {
    const { data: { session } } = await supabase.auth.getSession();
    
    // 收集畫面上顯示的報告
    const diseaseMap = {};
    displayedReports.value.forEach(report => {
    report.merged_diseases.forEach(d => {
        if (d.probability >= 30) {
        if (!diseaseMap[d.name] || d.probability > diseaseMap[d.name].probability) {
            diseaseMap[d.name] = d;
        }
        }
    });
    });

    const payload = Object.values(diseaseMap);

    const response = await axios.post(`${BACKEND_URL}/overall-insight`, payload, {
    headers: { Authorization: `Bearer ${session.access_token}` }
    });

    overallDetails.value.push({
    width: 650, height: 550,
    left: (window.innerWidth - 650) / 2,
    top: (window.innerHeight - 550) / 2,
    zIndex: getNextZIndex(),
    aspectRatio: 650 / 550,
    data: response.data 
    });

    await nextTick();
} catch (err) {
    errorMsg.value = "Analysis failed.";
} finally {
    loading.value = false;
}
}

const closeOverallDetail = (index) => {
overallDetails.value.splice(index, 1)
}

/* ---------------- 拖拉 & 縮放 ---------------- */
const startDrag = (e, type, index) => {
    if (e.target.classList.contains('resizer')) return;

    let modalState;
    if (type === 'single') modalState = singleModals.value[index];
    else if (type === 'overall') modalState = overallDetails.value[index];
    else if (type === 'heart') modalState = heartModalState.value; // ✅ 加入這行

    modalState.zIndex = getNextZIndex();

    // 這裡我們直接抓取當前的 DOM 元素
    const modal = e.currentTarget;

    dragInfo = { 
        type, index, 
        startX: e.clientX, startY: e.clientY, 
        modal, 
        origLeft: modalState.left, origTop: modalState.top 
    };
    window.addEventListener('mousemove', drag);
    window.addEventListener('mouseup', stopDrag);
}

const drag = e => {
    if (!dragInfo) return;
    const dx = e.clientX - dragInfo.startX;
    const dy = e.clientY - dragInfo.startY;

    let modalState;
    if (dragInfo.type === 'single') modalState = singleModals.value[dragInfo.index];
    else if (dragInfo.type === 'overall') modalState = overallDetails.value[dragInfo.index];
    else if (dragInfo.type === 'heart') modalState = heartModalState.value; // ✅ 加入這行

    modalState.left = dragInfo.origLeft + dx;
    modalState.top = dragInfo.origTop + dy;
}

const stopDrag = ()=>{
window.removeEventListener('mousemove',drag)
window.removeEventListener('mouseup',stopDrag)
dragInfo=null
}

const startResize = (e,type,index)=>{
    let modal;
    if (type === 'single') modal = singleModals.value[index];
    else if (type === 'overall') modal = overallDetails.value[index];
    else if (type === 'heart') modal = heartModalState.value;
    resizeInfo = { type, index, startX:e.clientX, startY:e.clientY, startWidth:modal.width, startHeight:modal.height, aspectRatio:modal.aspectRatio }
    window.addEventListener('mousemove',resize)
    window.addEventListener('mouseup',stopResize)
}

const resize = async e => {
    if (!resizeInfo) return;

    let modal;
    if (resizeInfo.type === 'single') modal = singleModals.value[resizeInfo.index];
    else if (resizeInfo.type === 'overall') modal = overallDetails.value[resizeInfo.index];
    else if (resizeInfo.type === 'heart') modal = heartModalState.value; // ✅ 加入這行

    const dx = e.clientX - resizeInfo.startX;
    let newWidth = Math.max(300, resizeInfo.startWidth + dx);
    let newHeight = newWidth / resizeInfo.aspectRatio;

    modal.width = newWidth;
    modal.height = newHeight;
    modal.fontSize = Math.max(12,newWidth*0.035)

    await nextTick()
    if(modal.chartInstance && modal.canvasRef){
        modal.canvasRef.style.width = newWidth+'px'
        modal.canvasRef.style.height = newHeight+'px'
        modal.chartInstance.resize()
        modal.chartInstance.update('none')
    }
    }
const stopResize = ()=>{
    window.removeEventListener('mousemove',resize)
    window.removeEventListener('mouseup',stopResize)
    resizeInfo=null
}

/* ---------------- 顏色對應 ---------------- */
const getSeverityColor = (prob)=>{
if(prob >= 60) return '#e53935'
if(prob >= 30) return '#fab70d'
return '#43a047'
}

onMounted(()=>fetchHistory())
</script>

<style scoped>
.history-wrapper { max-width:1000px; margin:0 auto; padding:20px 20px; position:relative; }
.page-title { font-size:2rem; font-weight:bold; text-align:center; margin-bottom:20px; color:#182b86; }
.history-toolbar { display:flex; justify-content:center; gap:10px; margin-bottom:20px; }
.history-toolbar input[type="date"] { 
  /* 增加這行來設定圓角 */
  border-radius: 8px; 
  
  /* 稍微調整內距與邊框，讓它更有質感 */
  padding: 7px 12px; 
  border: 1px solid #ccc;
  font-size: 0.9rem;
  outline: none; /* 點擊時不要出現預設的黑色外框 */
  transition: border-color 0.3s;
}

/* 當滑鼠點擊輸入框時，邊框變色 */
.history-toolbar input[type="date"]:focus {
  border-color: #4fa3ff;
  box-shadow: 0 0 5px rgba(79, 163, 255, 0.2);
}

.overall-btn { 
  /* 增加上下 (10px) 與左右 (20px) 的內距 */
  padding: 7px 20px; 
  border-radius: 8px; 
  border: 2px solid #4fa3ff; 
  background: white; 
  color: #4fa3ff; 
  /* 字體放大到 1.1rem */
  font-size: 0.9rem;
  font-weight: 800; /* 讓字體更粗一點，更有質感 */
  cursor: pointer; 
  transition: all 0.3s ease;
  /* 確保按鈕之間有足夠間距 */
  margin: 0 5px;
}

.overall-btn:hover { 
  background: #4fa3ff; 
  color: white; 
  /* 滑鼠移上去時稍微放大一點點 */
  transform: scale(1.05);
}
.report-card { background:#eef5ff; padding:20px; border-radius:12px; margin-bottom:20px; color:#000; position:relative; }
.graph-btn { position:absolute; top:12px; right:12px; background:#4fa3ff; color:white; border:none; border-radius:6px; padding:6px 10px; font-size:0.9rem; cursor:pointer; }
.graph-btn:hover { background:#2f7bdf; }
.high-text { color:#e53935; }
.medium-text { color:#db9f07; }
.low-text { color:#43a047; }
.source { font-size:0.9rem; color:#555; margin-left:8px; }
.summary-text,.recommendation-text { color:#000; line-height:1.6; }
.title { color:purple; font-weight:bold; margin-top:10px; margin-bottom:5px; }
.loading-overlay {
position: fixed;
top: 0;
left: 0;
width: 100%;
height: 100%;
display: flex;
justify-content: center;
align-items: center;
font-size: 32px;
font-weight: bold;
color: rgb(166, 160, 243);
z-index: 9999;
}

/* Modal */
.modal { position:fixed; background:white; border-radius:12px; padding:16px; cursor:move; display:flex; flex-direction:column; }
.modal h3 { margin-bottom:12px; }
.canvas-wrapper { position:relative; width:100%; height:calc(100% - 80px); display:flex; justify-content:center; align-items:center; overflow:hidden; }
canvas { display:block; width:100% !important; height:100% !important; }
.resizer { width:12px; height:12px; background:#4fa3ff; position:absolute; right:0; bottom:0; cursor:se-resize; border-radius:50%; }
.close-btn { position:absolute; top:10px; right:10px; background:transparent; border:none; font-size:1.5rem; cursor:pointer; }
/* .category-buttons { display:flex; justify-content:center; gap:10px; margin-bottom:10px; }
.category-buttons button { font-size:1em; padding:0.4em 0.8em; }
.category-buttons button.active-btn { background:#4fa3ff; color:white; } */
.risk-legend {
text-align: center;
margin-top: 8px;
font-size: 0.95rem;
font-weight: 500;
}
.modal-header {
text-align: center;
color: #1a237e;
border-bottom: 2px solid #e8eaf6;
padding-bottom: 10px;
margin-bottom: 10px;
}

/* 加入一點卡片感 */
.disease-card {
background: linear-gradient(135deg, #ffffff 0%, #f0f4ff 100%);
border-radius: 15px;
padding: 20px;
margin-bottom: 20px;
border-left: 6px solid #3949ab;
box-shadow: 0 4px 15px rgba(0,0,0,0.05);
}

.disease-name-title {
font-size: 1.25rem;
font-weight: 800;
color: #1a237e;
text-transform: uppercase;
}

.info-group label {
font-size: 1rem;
color: #5c6bc0;
margin-top: 10px;
}

.info-group p {
font-size: 1.05rem;
line-height: 1.7;
color: #2c3e50;
}

.info-group{
margin-bottom: 15px;
}

.divider {
border: 0;
border-top: 1px solid #eee;
margin: 15px 0;
}

.note-section {
background: #fffde7;
padding: 12px;
border-left: 4px solid #fbc02d;
font-size: 0.9rem;
color: #555;
margin-top: 10px;
}
.heart-view-btn {
  padding: 6px 12px;
  border-radius: 6px;
  border: 2px solid #ff4f7b; /* 使用愛心的粉紅色系 */
  background: white;
  color: #ff4f7b;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}
.heart-view-btn:hover {
  background: #ff4f7b;
  color: white;
}

/* 新增：圖片彈窗樣式 */
.image-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7); /* 半透明黑背景 */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000; /* 確保在最上層 */
}

.image-modal-content {
  background: white;
  padding: 20px;
  border-radius: 15px;
  position: relative;
  max-width: 40%;
  max-height: 40%;
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
  text-align: center;
}

/* 移除 .image-modal-overlay */
/* 確保 .modal 樣式能容納圖片 */
.img-container {
    margin-top: 10px;
    height: calc(100% - 60px); /* 扣掉標題高度 */
    display: flex;
    justify-content: center;
    align-items: center;
}

.reference-img {
    max-width: 100%;
    max-height: 100%;
    pointer-events: none; /* 讓圖片不干擾拖拽 */
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 15px;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #333;
}
</style>