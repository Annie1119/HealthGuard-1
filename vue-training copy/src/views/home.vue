<template>
  <div class="home-page">
  <!-- <header class="site-header">
    <router-link to="/" class="logo">
        HealthGuard🫀
    </router-link>
  </header> -->
    <section class="hero-section">
      <div class="hero-content">
        <h1 class="main-tagline">
          Predict today, prevent decay.
          <br>
          Live secure, be sure.
        </h1>
        <p class="sub-tagline">
          HealthGuard — a professional **HEART DISEASE ANALYSIS** boilerplate with secure auth and clean UI.
        </p>
        
        <div class="action-buttons">
          <router-link to="/login" class="btn get-started">
            GET STARTED 🚀
          </router-link>
          
          <router-link to="/login" class="btn log-in">
            LOG IN
          </router-link>
        </div>

        <div class="privacy-disclaimer">
          <span class="lock-icon">🔒</span>
          Your data is safe. All records are <strong>strictly encrypted</strong> and stored 
          in a professional cloud infrastructure for maximum security.
        </div>

        <div class="explore-features" @click="scrollToFeatures">
          EXPLORE FEATURES
        </div>
      </div>
    </section>

    <section class="features-section" ref="featuresSection">
      <h2>🎯 Features</h2>
      <div class="feature-list">
        <div class="feature-card">
          <h3>Accurate Risk Prediction</h3>
          <p>Combining advanced machine learning models, we provide personalized disease risk analysis reports based on your health data.</p>
        </div>
        <div class="feature-card">
          <h3>Secure Data Management</h3>
          <p>We use industry-standard encryption technologies to ensure the security of all your personal health data.</p>
          </div>
          <div class="feature-card">
          <h3>User-Friendly Interface</h3>
          <p>A clean and intuitive interface design that allows you to easily view complex analysis results and take action.</p>
          </div>
          </div>
          <div style="height: 300px; text-align: center; padding-top: 50px;">
          <p>More features are waiting for you to explore...</p>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const featuresSection = ref(null);

function scrollToFeatures() {
  featuresSection.value.scrollIntoView({ behavior: 'smooth' });
}
</script>

<style scoped>
/* 頁面內容容器 (將 Header 偏移和居中樣式合併) */
.home-page {
padding-top: 60px;  /* 確保內容不會被 fixed header 遮擋 */
/* 讓整個 Home 頁面內容居中 */
/* max-width: 1200x; 
margin: 0 auto; */
width: 100%;
/* 將 padding-top 以外的 padding 設為 20px，並加上 border-box */
padding: 0 20px 20px; 
box-sizing: border-box;
/* 保持 column 佈局 */
display: flex;
flex-direction: column;
}

/* --- Header --- */
.site-header {
width: 100%;
position: fixed;      
top: 0;
/* 讓 header 寬度永遠滿版，不被 home-page 的 max-width 限制 */
left: 0;
padding: 15px 20px;
background-color: #ffffffcc; 
display: flex;
align-items: center;
z-index: 1000;        
box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

.logo {
font-size: 1.5rem;
font-weight: 800;
color: #4f8898;
text-decoration: none;
}

/* --- Hero Section --- */
.hero-section {
  width: 100vw;      /* 滿版背景 */
  /* margin-left: calc(50% - 50vw); */
  min-height: calc(100vh - 60px); 
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  /* background-color: #f8f0e8; */
  box-sizing: border-box;
}

.hero-content {
max-width: 1000px;
width: 90%;
/* 給 Hero 內容添加一個垂直間距 */
padding: 40px 0; 
padding: 120px 0 40px;
}

/* 標語 & 副標語 */
.main-tagline {
font-size: 4rem; 
font-weight: 800;
line-height: 1.2;
color: #9162a0;
margin-bottom: 20px;
}

.sub-tagline {
font-size: 1.5rem;
color: #150101;
margin-bottom: 40px;
}

/* 按鈕群組 */
.action-buttons {
display: flex;
justify-content: center;
gap: 20px;
margin-bottom: 20px;
}

.btn {
padding: 12px 30px;
border-radius: 8px;
font-weight: bold;
text-decoration: none;
transition: background-color 0.3s, transform 0.2s;
}

/* 按鈕顏色 */
.get-started {
background-color: #1a5a89;
color: white;
border: 2px solid #1a5a89;
}

.get-started:hover {
background-color: #13456b;
transform: translateY(-2px);
}

.log-in {
background-color: white;
color: #1a5a89;
border: 2px solid #1a5a89;
}

.log-in:hover {
background-color: #e9ecef;
transform: translateY(-2px);
}

.explore-features {
/* 新增樣式，讓它看起來像個連結或提示 */
margin-top: 15px;
font-size: 0.9rem;
color: #233a7a;
cursor: pointer;
opacity: 0.7;
}

/* --- Features Section --- */
.features-section {
  width: 100vw;                        /* 滿版背景 */
  /* margin-left: calc(50% - 50vw);        */
  padding: 60px 0;
  /* background-color: #f8f0e8; */
  text-align: center;
  box-sizing: border-box;
}

.features-section h2 {
font-size: 2.5rem;
color: #4274ad;
margin-bottom: 40px;
}

.feature-list {
display: flex;
justify-content: center;
gap: 30px;
flex-wrap: wrap;
}

.feature-card {
max-width: 350px;
padding: 20px;
border-radius: 10px;
box-shadow: 0 4px 12px rgba(55, 82, 103, 0.08);
text-align: left;
transition: transform 0.3s;
background-color: white; /* 讓卡片內部背景與區塊背景區隔 */
}

.feature-card:hover {
transform: translateY(-5px);
box-shadow: 0 6px 15px rgba(55, 82, 103, 0.15); /* hover 時加強陰影 */
}

.feature-card h3 {
color: #592289;
margin-bottom: 10px;
font-size: 1.5rem;
}

.feature-card p {
color: #283374;
font-size: 1rem;
line-height: 1.5;
}


/* --- 響應式調整 (手機/小螢幕) --- */
@media (max-width: 768px) {
/* 調整 Header 偏移，以應對更小的手機邊距 */
.home-page {
  padding-top: 55px;
}

/* 調整 Hero 區塊高度 */
.hero-section {
  min-height: calc(100vh - 55px);
}

.main-tagline {
  font-size: 2.5rem;
}
.sub-tagline {
  font-size: 1rem;
}
.hero-content {
  width: 95%;
}
.features-section h2 {
  font-size: 2rem;
}
.feature-card {
  max-width: 90%;
  padding: 15px;
}
.feature-card h3 {
  font-size: 1.2rem;
}
.feature-card p {
  font-size: 0.9rem;
}

/* 響應式調整按鈕 (維持您原有的優化) */
.btn {
  padding: 10px 20px;
  font-size: 0.9rem;
}

.action-buttons {
  flex-direction: column;
  gap: 15px;
}
}

/* 手機響應式 */
@media (max-width: 768px) {
  .footer-links {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
}

/* --- Privacy Disclaimer --- */
.privacy-disclaimer {
  margin-top: 8px;
  padding: 15px 25px;
  background-color: rgba(255, 255, 255, 0.5); /* 半透明背景 */
  border-radius: 50px; /* 膠囊形狀 */
  display: inline-block;
  font-size: 0.9rem;
  color: #000000;
  line-height: 1.4;
  max-width: 600px;
  border: 1px border;
  border: 1px solid rgba(79, 136, 152, 0.2);
}

.lock-icon {
  margin-right: 8px;
}

.privacy-disclaimer strong {
  color: #4f8898; /* 強調第三方字眼 */
  font-weight: 700;
}

/* 針對手機版的調整 */
@media (max-width: 768px) {
  .privacy-disclaimer {
    font-size: 0.8rem;
    padding: 12px 20px;
    border-radius: 15px; /* 手機版改為圓角矩形比較省空間 */
    margin: 20px 10px 0;
  }
}
</style>
