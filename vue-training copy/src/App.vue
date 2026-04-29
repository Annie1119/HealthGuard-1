<template>
  <div id="app-container">
    <header 
      v-if="route.path === '/predict' || route.path === '/history' || route.path === '/source' || route.path === '/' || route.path === '/login'" 
      class="site-header"
      :class="headerClass"
    >
      
      <router-link to="/" class="logo" @click="dropdownOpen = false">HealthGuard🫀</router-link>

      <nav v-if="dropdownOpen && isFunctionalPage" class="nav-center-menu">
        <RouterLink to="/predict" class="nav-item" @click="dropdownOpen = false">predict</RouterLink>
        <RouterLink to="/history" class="nav-item" @click="dropdownOpen = false">history</RouterLink>
        <RouterLink to="/source" class="nav-item" @click="dropdownOpen = false">source</RouterLink>
      </nav>

      <div v-if="isLoginedIn && route.path !== '/' && route.path !== '/login'" class="header-right">
        <div class="dropdown">
          <button class="dropdown-btn" @click.stop="toggleDropdown">
            Menu ▼
          </button>
        </div>
        <Profile />
      </div>
    </header>

    <main class="page-content">
      <RouterView />
    </main>

    <div 
      v-if="isFunctionalPage" 
      class="floating-widget"
      :style="{ transform: `translate(${position.x}px, ${position.y}px)` }"
      @mousedown="startDrag"
    >
      <div class="widget-content">
        <div class="widget-header">
          <span class="icon">📝</span>
          <span class="title">Feedback</span>
        </div>
        <p style= "font-size: 1rem" class="description">Help us improve!</p>
        <a 
          href="https://docs.google.com/forms/d/e/1FAIpQLSdcalrjCHNJk6ZWN_Z9P3uatvIcEB98ukdHKzKz1rnJe92SHA/viewform?usp=header" 
          target="_blank" 
          class="feedback-btn"
          style= "font-size: 1rem"
        >
          Open Form
        </a>
      </div>
    </div>

    <footer v-if="route.path === '/' || route.path === '/history' || route.path === '/source'" class="site-footer">
      <div class="footer-content">
        <p>© 2025 HealthGuard. All rights reserved.</p>
        
        <div class="attribution">
          <a href="https://www.flaticon.com/free-icons/heart" title="heart icons" target="_blank" rel="noopener" style="color: white">
            Heart icons created by Futuer - Flaticon
          </a>
        </div>

        <div class="footer-links">
          <a href="#">Privacy Policy</a>
          <a href="#">Terms of Service</a>
          <a href="mailto:foreverannie.1119@gmail.com">Contact</a>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { supabase } from '@/supabase';
import Profile from '@/views/profile.vue';

const router = useRouter();
const route = useRoute();
const isLoginedIn = ref(false);
const dropdownOpen = ref(false);

const toggleDropdown = () => { dropdownOpen.value = !dropdownOpen.value; };

const isFunctionalPage = computed(() => {
  return ['/predict', '/history', '/source'].includes(route.path);
});

const headerClass = computed(() => {
  // 首頁與登入頁：保持窄版白色
  if (route.path === '/' || route.path === '/login') {
    return 'header-home-login';
  }
  // 功能頁面展開
  if (isFunctionalPage.value && dropdownOpen.value) {
    return 'header-expanded-white';
  }
  // 功能頁面平時透明
  return 'header-functional-transparent';
});

const handleClickOutside = (event) => {
  if (!event.target.closest('.dropdown') && !event.target.closest('.nav-center-menu')) {
    dropdownOpen.value = false;
  }
};

onMounted(() => {
  window.addEventListener('click', handleClickOutside);

  // 1. 檢查初始 Session
  supabase.auth.getSession().then(({ data: { session } }) => {
    isLoginedIn.value = !!session;
    // 只有當使用者「手動輸入 /login」且已登入時，才把他踢走
    if (session && route.path === '/login') {
      router.push('/predict');
    }
  });

  // 2. 監聽狀態變更
  supabase.auth.onAuthStateChange((event, session) => {
    isLoginedIn.value = !!session;

    if (event === 'SIGNED_IN') {
      // 💡 關鍵修正：只有當目前確實在 '/login' 頁面時，登入成功才跳轉
      // 這樣平常在 History / Source 之間切換時，就不會被抓回 Predict
      if (route.path === '/login') {
        router.push('/predict');
      }
    } else if (event === 'SIGNED_OUT') {
      router.push('/login');
    }
  });
});

onUnmounted(() => {
  window.removeEventListener('click', handleClickOutside);
});

const position = ref({ x: 20, y: 100 }); // 初始位置
let isDragging = false;

const startDrag = (e) => {
  isDragging = true;
  const startX = e.clientX - position.value.x;
  const startY = e.clientY - position.value.y;

  const onMouseMove = (e) => {
    if (!isDragging) return;
    position.value = {
      x: e.clientX - startX,
      y: e.clientY - startY
    };
  };

  const stopDrag = () => {
    isDragging = false;
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mouseup', stopDrag);
  };

  window.addEventListener('mousemove', onMouseMove);
  window.addEventListener('mouseup', stopDrag);
};
</script>

<style>
body {
  margin: 0 !important; /* 🔥 加上 !important 確保強制置頂 */
  padding: 0;
  background-color: #f8f0e8;
}
#app-container { min-height: 100vh; display: flex; flex-direction: column; }
.page-content { flex: 1;}

.site-header {
  width: 100%;
  padding: 15px 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: fixed;
  top: 0; left: 0; z-index: 1000;
  box-sizing: border-box;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.header-home-login {
  background-color: white !important;
  height: 70px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
}

.header-functional-transparent {
  background-color: transparent !important;
  height: 70px;
  box-shadow: none;
}

.header-expanded-white {
  background-color: white !important;
  height: 110px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.08);
}

.logo { font-size: 1.5rem; font-weight: 800; color: #4f8898; text-decoration: none; }

.nav-center-menu {
  display: flex;
  gap: 220px; 
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  animation: slideDown 0.4s ease;
}

.nav-item {
  color: #333;
  text-decoration: none;
  font-size: 1.3rem;
  font-weight: 700;
  text-transform: lowercase;
}

.router-link-active.nav-item {
  color: #4f8898;
  border-bottom: 3px solid #4f8898;
}

@keyframes slideDown {
  from { opacity: 0; transform: translate(-50%, -15px); }
  to { opacity: 1; transform: translate(-50%, 0); }
}

.dropdown { position: relative; margin-right: 80px; }
.dropdown-btn {
  background-color: #8ebfe6; 
  color: white;
  padding: 10px 20px; 
  font-size: 1.1rem;
  font-weight: 700;
  border: none;
  border-radius: 10px;
  cursor: pointer;
}
.header-right { display: flex; align-items: center; gap: 15px; }
.site-footer {
  width: 100vw;
  margin-left: calc(50% - 50vw);
  background-color: #253e69;
  color: white;
  padding: 20px 20px;
  text-align: center;
  margin-top: auto;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  line-height: 1.5;
}

.footer-links a {
  color: white;
  text-decoration: none;
  margin: 0 15px;
  transition: color 0.3s;
}
.footer-links a:hover { color: #eda6c1; }

.floating-widget {
  position: fixed;
  width: 180px;
  height: 100px;
  padding: 8px;
  background: rgba(255, 255, 255, 0.6); /* 半透明 */
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 15px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  z-index: 9999;
  cursor: grab; /* 讓使用者知道可以抓取 */
  user-select: none; /* 防止拖曳時選取到文字 */
}

.floating-widget:active {
  cursor: grabbing;
}

.widget-content span {
  font-size: 1.2rem;
  font-weight: 800;
  color: #4f8898;
}

.widget-content p {
  margin: 5px 0 0;
  font-size: 0.9rem;
  color: #333;
}
</style>