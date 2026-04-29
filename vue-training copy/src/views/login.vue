<template>
  <div class="login-register-page">

    <!-- <header class="site-header">
          <router-link to="/" class="logo">
            HealthGuard🫀
          </router-link>
    </header> -->
        
    <div class="login-register-card">
        <h2 class="form-title">{{ (isRegistering ? 'Sign up' : 'Log In') }}</h2>
        
        <form @submit.prevent="handleSubmit"> <!-- prevent 攔截預設行為（頁面重新載入）執行handleSubmit -->
          <div class="input-group" v-if="isRegistering">
              <input 
                  id="username" 
                  type="text" 
                  v-model="username" 
                  placeholder="" 
                  required 
                  class="field-input"
              />
              <label for="username" class="field-label">Username</label>
          </div>  
          
          <div class="input-group">
                <input 
                    id="email" 
                    type="email" 
                    v-model="email" 
                    placeholder="" 
                    required
                    class="field-input"
                />
                <label for="email" class="field-label">Email</label>
                <!-- <span class="input-icon">👁️</span>  -->
            </div>

            <div class="input-group">
              <input 
                  id="password" 
                  :type="showPassword ? 'text' : 'password'" 	
                  v-model="password" 
                  placeholder="" 
                  required 
                  class="field-input"
              /> 
              
              <!--* 如果 showPassword 是 true → input 的type 會變成'text'（密碼可見）如果 showPassword 是 false → input的type 會變成 password'（密碼隱藏）'password' 是 HTML 的原生屬性值，表示文字被隱藏成點點。-->
              <label for="password" class="field-label">Password</label>
              <span class="input-icon" @click="showPassword = !showPassword"> <!-- 點擊時把 showPassword 的布林值反轉 -->
                {{ showPassword ? '🙈' : '👁️' }}
              </span>
            </div>

            <div v-if="errorMsg" class="message error-message">{{ errorMsg }}</div>
            <div v-if="successMsg" class="message success-message">{{ successMsg }}</div>

            <button type="submit" class="submit-btn">
                {{ (isRegistering ? 'Sign Up' : 'Log In') }}
            </button>
        </form>

        <div class="mode-link-bottom" @click="toggleMode" style="cursor: pointer;">
            {{ isRegistering ? 'Already have an account? Log In' : "Don't have an account? Sign Up" }}
        </div>

        <!-- <div class="predict-button-wrapper" style="margin-top: 20px; text-align: center;">
          <router-link to="/predict">
            <button class="submit-btn" style="background-color: #28a745; border: none;">
              Go to Predict Page
            </button>
          </router-link>
        </div> -->
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { supabase } from '@/supabase' // 引入 Supabase 客戶端

const router = useRouter();

// 響應式狀態
const username = ref('')
const email = ref('');
const isRegistering = ref(false);
const password = ref(''); 
const showPassword = ref(false); // false = 密碼隱藏, true = 顯示密碼

// 錯誤訊息
const errorMsg = ref(null);
const successMsg = ref(null);

// 函式：處理登入
const handleLogin = async () => {
  errorMsg.value = null;
  successMsg.value = null;
  
  try {
    const { error } = await supabase.auth.signInWithPassword({
      email: email.value,
      password: password.value,
    })

    if (error) throw error
    router.push('/predict'); // 登入成功後跳轉到預測頁面

    }catch (error) {
      if (error.message.includes("Invalid login credentials")) {
      errorMsg.value = "Incorrect email or password";
    } else if (error.message.includes("User not confirmed")) {
      errorMsg.value = "Account not confirmed, please check your email";
    } else {
      errorMsg.value = "Login failed, please try again later";
    }
  }
}
  
// 函式：處理註冊 
const handleRegister = async () => {
  errorMsg.value = null;
  successMsg.value = null;

  try {
    const { data, error } = await supabase.auth.signUp({
      email: email.value,
      password: password.value,
      options: {
        data: {
          display_name: username.value, 
        }
      }
    })

    if (error) throw error

    // 🔥 重要：註冊成功後立刻強制登出
    // 這樣可以防止 Supabase 自動在 LocalStorage 建立 Session
    // 確保 App.vue 的監聽器不會把你自動跳轉到 /predict
    await supabase.auth.signOut();

    // 提示使用者檢查信箱
    successMsg.value = 'Registration successful! Please check your email to confirm your account.';
    
    // 清空密碼欄位
    password.value = ''
  
    // 💡 延遲 2 秒後自動切換回登入介面，給使用者時間看綠色訊息
    setTimeout(() => {
      isRegistering.value = false;
      // 注意：切換回登入頁後，我們保留 successMsg 讓使用者知道要去看信
      // 或者你也可以選擇在此時清空它：successMsg.value = null;
    }, 2000);

  } catch (error) {
    errorMsg.value = error.message
  }
}

// 在 template 使用 @click="toggleMode"
// 手動點擊切換連結時，徹底清空所有欄位與訊息
const toggleMode = () => {
  isRegistering.value = !isRegistering.value;
  errorMsg.value = null;   // 清空紅框
  successMsg.value = null; // 清空綠框
  password.value = '';     
  username.value = '';     
  email.value = '';        // 視需求決定是否連 email 都清空
}

// 根據當前模式選擇執行登入或註冊
const handleSubmit = () => {
  if (isRegistering.value) {
    handleRegister();
  } else {
    handleLogin();
  }
}
</script>

<style scoped>
 .site-header {
  width: 100%;                  
  position: fixed;               
  top: 0;                        
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

/* 頁面留出 header 高度 */
.home-page {
  padding-top: 60px
}

/* 外層容器，滿版白色背景 + 置中內容 */
.login-register-page {
  /* width: 100vw;      */         
  /* min-height: 100%; */
  position : fixed;           
  inset: 0;             /* top:0; right:0; bottom:0; left:0; 簡寫 */
  display: flex;              /* 使用 Flex 置中內容 */
  justify-content: center;    /* 水平置中 */
  align-items: flex-start;  /* 垂直置頂 */
  background-color: #f8f0e8; 
  padding: 60px 20px 20px;    /* 預留 header 高度 + 左右內距 */
  box-sizing: border-box;
}

/* 卡片保持原本樣式 */
.login-register-card {
  margin-top: 150px;
  max-width: 330px;               
  width: 100%;
  /* padding: 30px 40px;             
  border-radius: 12px;            
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); 
  background-color: #f8f0e8;         */
  text-align: center;      
  transform: scale(1.2);       /* ⭐ 整體放大 15% */
  transform-origin: top center; /* 從上方中心點放大 */       
}

.form-title {
  font-size: 2.5rem;             
  font-weight: 600;              
  color: #333;                   
  margin-bottom: 30px;        
}

/* 輸入組容器 */
.input-group {
  margin-bottom: 20px;   
  position: relative;          
}

.field-input {
  width: 100%;                    
  padding: 18px 15px 5px 15px;  
  border: 1px solid #ccc;         
  border-radius: 8px;            
  font-size: 1rem;              
  box-sizing: border-box;         
  transition: border-color 0.3s;   
}

/* 聚焦時邊框變色 */
.field-input:focus {
  outline: none;                
  border-color: #2196F3;      
}

/* Label 浮動效果 */
.field-label {
  position: absolute;        
  left: 15px;                  
  top: 5px;                       
  font-size: 0.8rem;               
  color: #999;                  
  pointer-events: none;          
  transition: all 0.2s ease;      
}

/* 輸入框有值或聚焦時 label 浮上 */
.field-input:not(:placeholder-shown) + .field-label,  /* .field-label表示選擇緊接在 input 後面的 label 元素。效果：當 input 有值或聚焦時，label 會浮到上方、變小、變藍色，看起來像「漂浮標籤」的效果。*/
.field-input:focus + .field-label {
  top: 5px;                       
  font-size: 0.7rem;            
  color: #2196F3;                
}

/* 右側圖標（眼睛） */
.input-icon {
  position: absolute;          
  right: 15px;          
  top: 50%;              
  transform: translateY(-50%);    
  color: #aaa;                     
  cursor: pointer;               
  font-size: 1.1rem;             
}

/* 提交按鈕 */
.submit-btn {
  width: 100%;                  
  padding: 15px;                    
  margin-top: 20px;               
  border: none;                    
  border-radius: 8px;              
  background-color: #2196F3;        
  color: white;                   
  font-size: 1.1rem;             
  font-weight: bold;            
  cursor: pointer;                
  transition: background-color 0.3s; 
}

.submit-btn:hover {              
  background-color: #1976D2;       
}

/* 底部切換連結 */
.mode-link-bottom {
  margin-top: 25px;              
  font-size: 0.9rem;              
  color: #666;                   
}

.link-text {
  color: #2196F3;               
  text-decoration: none;          
  font-weight: bold;               
  cursor: pointer;                 
}

.link-text:hover {
  text-decoration: underline;      
}

/* 錯誤/成功訊息 */
.message {
  padding: 10px;                   
  border-radius: 5px;             
  margin-bottom: 15px;           
  text-align: left;                
  font-size: 0.9rem;              
}

.error-message {
  background-color: #fce4e4;      
  color: #cc0033;                 
  border: 1px solid #cc0033;      
}

.success-message {
  background-color: #e6ffe6;       
  color: #008000;                  
  border: 1px solid #008000;       
}

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background: white;   
  overflow: hidden;   /* ✅ 鎖住整個畫面，不能滑 */  
}
</style>

