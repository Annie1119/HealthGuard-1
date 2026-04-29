<template>
    <div class="source-wrapper" style = "margin-top: 30px">
  
      <h2 class="page-title">Medical Sources & Data References</h2>
  
      <p class="intro">
        The following medical sources and datasets support the disease explanations 
        and risk analysis used in this heart risk prediction system.
      </p>
  
      <div class="source-grid">
  
        <!-- Machine Learning Datasets -->
        <div class="source-card">
          <h3>Machine Learning Datasets</h3>
  
          <ul style="color: black;">
            <li><strong>Cardiovascular Disease Dataset:</strong> Used to train the cardiovascular risk prediction model based on clinical features such as age, blood pressure, cholesterol, and lifestyle factors.</li>
            <li><strong>Stroke Prediction Dataset:</strong> Used to estimate stroke risk based on glucose level, BMI, smoking status, and medical history.</li>
          </ul>
  
          <p class="source-title">Sources</p>
  
          <a href="https://www.kaggle.com/datasets/sulianova/cardiovascular-disease-dataset" target="_blank">
            Cardiovascular Disease Dataset (Kaggle)
          </a>
  
          <a href="https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset" target="_blank">
            Stroke Prediction Dataset (Kaggle)
          </a>
        </div>
  
        <!-- Hypertension -->
        <div class="source-card">
          <h3>Hypertension (High Blood Pressure)</h3>
  
          <ul style="color: black;">
            <li>Normal: 120/80 mmHg (systolic 120, diastolic 80)</li>
            <li>Prehypertension: 120/80 – 140/90 mmHg; may develop into hypertension, lifestyle changes recommended</li>
            <li>Hypertension: above 140/90 mmHg; consult a doctor for follow-up and treatment</li>
          </ul>
  
          <p class="source-title">Sources</p>
  
          <a href="https://www.hpa.gov.tw/Pages/Detail.aspx?nodeid=127&pid=8159" target="_blank">
            Taiwan Health Promotion Administration
          </a>
        </div>
  
        <!-- Hyperlipidemia -->
        <div class="source-card">
          <h3>Hyperlipidemia (High Blood Lipids)</h3>
  
          <ul style="color: black;">
            <li><strong>Normal levels:</strong> Total cholesterol &lt; 200 mg/dL, Triglycerides &lt; 150 mg/dL</li>
            <li><strong>Definition:</strong> Elevated cholesterol or triglycerides beyond normal range</li>
            <li><strong>Common causes:</strong> High-fat diet, lack of exercise, genetics, alcohol, and other diseases</li>
          </ul>
  
          <p class="source-title">Sources</p>
  
          <a href="https://wwwv.tsgh.ndmutsgh.edu.tw/unit/10012/12867" target="_blank">
            Tri-Service General Hospital
          </a>
  
          <a href="https://www.liver.org.tw/journalView.php?cat=73&sid=1067&page=1" target="_blank">
            Taiwan Liver Foundation
          </a>
        </div>
  
        <!-- Arrhythmia -->
        <div class="source-card">
          <h3>Arrhythmia / Palpitations</h3>
  
          <ul style="color: black;">
            <li><strong>Seek medical help if:</strong> chest pain, shortness of breath, dizziness, or cold sweats occur</li>
            <li><strong>High-risk groups:</strong> smokers, hypertension, high cholesterol, obesity, diabetes, sedentary lifestyle</li>
            <li><strong>Symptoms:</strong> rapid heartbeat, skipped beats, irregular rhythm</li>
          </ul>
  
          <p class="source-title">Source</p>
  
          <a href="https://helloyishi.com.tw/heart-health/arrhythmias/what-are-heart-palpitations/" target="_blank">
            Hello Health
          </a>
        </div>
  
        <!-- Coronary Artery Disease -->
        <div class="source-card">
          <h3>Coronary Artery Disease</h3>
  
          <p style="color: black;"><strong>High-risk factors:</strong></p>
  
          <ul style="color: black;">
            <li><strong>Demographic:</strong> Male, Age &gt; 40</li>
            <li><strong>Medical:</strong> Hypertension, High cholesterol, Diabetes</li>
            <li><strong>Lifestyle:</strong> Smoking, Obesity, High stress</li>
            <li><strong>Genetic:</strong> Family history of heart disease or stroke</li>
          </ul>
  
          <p class="source-title">Sources</p>
  
          <a href="https://www.tsmh.org.tw/sites/nursing_department/int_13.html" target="_blank">
            Tainan Municipal Hospital
          </a>
  
          <a href="https://wd.vghtpe.gov.tw/cvs/Fpage.action?muid=11018&fid=10428" target="_blank">
            Taipei Veterans General Hospital
          </a>
        </div>
  
      </div>
  
      <div class="disclaimer" style="color: black;">
        <strong>Disclaimer:</strong>
        This system is intended for educational and research purposes only. 
        It does not provide medical diagnosis or treatment recommendations. 
        Please consult a qualified healthcare professional for medical advice.
      </div>
  
    </div>
  </template>
  
  <style scoped>
  
  .source-wrapper{
    max-width:1000px;
    margin:auto;
    padding:40px 20px;
  }
  
  .page-title{
    font-size:2rem;
    font-weight:800;
    text-align:center;
    color:#182b86;
    margin-bottom:15px;
  }
  
  .intro{
    text-align:center;
    margin-bottom:30px;
    color:#555;
  }
  
  .source-grid{
    display:grid;
    grid-template-columns:1fr 1fr;
    gap:20px;
  }
  
  .source-card{
    background:#eef5ff;
    padding:20px;
    border-radius:12px;
    box-shadow:0 5px 12px rgba(0,0,0,0.1);
  }
  
  .source-card h3{
    color:#1a237e;
    margin-bottom:8px;
  }
  
  .source-title{
    font-weight:bold;
    margin-top:10px;
    color:#1a237e;
  }
  
  .source-card a{
    display:block;
    color:#1a73e8;
    margin-top:4px;
  }
  
  .source-card a:hover{
    text-decoration:underline;
  }
  
  .disclaimer{
    margin-top:40px;
    padding:15px;
    background:#fff8e1;
    border-left:5px solid #fbc02d;
    border-radius:6px;
  }
  
  .styled-photo {
    width: 100%;      
    max-width: 600px; 
    height: auto;    
    border-radius: 12px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  }
  
  </style>