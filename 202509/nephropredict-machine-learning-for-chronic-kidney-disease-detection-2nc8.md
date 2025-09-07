---
Title: 🩺 NephroPredict: Machine Learning for Chronic Kidney Disease Detection
Description: 
Author: AbuBakar Shabbir
Date: 2025-09-07T20:31:58.000Z
Robots: noindex,nofollow
Template: index
---
<p>Chronic Kidney Disease (CKD) is a global health concern affecting millions of people worldwide. Early detection is crucial, as timely intervention can significantly reduce the need for dialysis and kidney transplants.  </p>

<p><strong>NephroPredict</strong> is a machine learning-based project designed to provide an efficient solution for early CKD detection using clinical and laboratory data.</p>




<h2>
  
  
  Project Overview
</h2>

<ul>
<li>
<strong>Goal</strong>: Early detection of CKD using predictive machine learning models
</li>
<li>
<strong>Dataset</strong>: Chronic Kidney Disease dataset (UCI Repository)
</li>
<li>
<strong>Approach</strong>: Implemented multiple ML models such as Logistic Regression, KNN, SVM, Decision Tree, and Random Forest
</li>
<li>
<strong>Tuning</strong>: Applied hyperparameter tuning (GridSearchCV) to improve model performance
</li>
<li>
<strong>Evaluation Metrics</strong>: Accuracy, Precision, Recall, F1-score, and Cross-validation scores
</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx3hrgu8s93zmkjxnet8z.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx3hrgu8s93zmkjxnet8z.png" alt=" " width="800" height="372"></a></p>




<h2>
  
  
  Attributes Used
</h2>

<p>The dataset consists of important medical attributes, including:  </p>

<p><code>Blood Pressure (Bp), Specific Gravity (Sg), Albumin (Al), Sugar (Su), Red Blood Cells (Rbc), Blood Urea (Bu), Serum Creatinine (Sc), Sodium (Sod), Potassium (Pot), Hemoglobin (Hemo), White Blood Cell Count (Wbcc), Red Blood Cell Count (Rbcc), Hypertension (Htn), Class (Target Variable)</code>  </p>




<h2>
  
  
  Models &amp; Results
</h2>

<p>The following models were implemented and compared <strong>before and after hyperparameter tuning</strong>:</p>

<ul>
<li>
<strong>Logistic Regression</strong>
</li>
<li>
<strong>K-Nearest Neighbors (KNN)</strong>
</li>
<li>
<strong>Support Vector Machine (SVM)</strong>
</li>
<li>
<strong>Decision Tree</strong>
</li>
<li>
<strong>Random Forest</strong>
</li>
</ul>

<h3>
  
  
  Key Findings
</h3>

<ul>
<li>Logistic Regression and KNN showed slight improvement after tuning, but also signs of overfitting
</li>
<li>SVM significantly improved after tuning, highlighting the importance of kernel and regularization
</li>
<li>Decision Tree accuracy dropped after tuning, showing sensitivity to depth and splitting criteria
</li>
<li>Random Forest consistently achieved the best results with <strong>100% training accuracy</strong> and <strong>96.55% test accuracy</strong>, making it the most robust model
</li>
</ul>

<p><strong>Conclusion</strong>: Random Forest is the most reliable model for this dataset and is recommended for CKD prediction.  </p>




<h2>
  
  
  📈 Visualization
</h2>

<p>The project includes professional visualizations such as styled tables and bar charts for comparing model performance.  </p>




<h2>
  
  
  Repository
</h2>

<p>The complete source code, dataset, and notebook are available on GitHub:  </p>

<p>🔗 <a href="https://github.com/Abubakar-Shabbir/Machine-Learning-for-Chronic-Kidney-Disease-Detection" rel="noopener noreferrer">Machine Learning for Chronic Kidney Disease Detection</a></p>




<h2>
  
  
  👨‍💻 Author
</h2>

<p>Developed by <strong>Abubakar Shabbir</strong>  </p>




<h2>
  
  
  📜 License
</h2>

<p>This project is licensed under the <strong>MIT License</strong> – feel free to use, modify, and share.  </p>




<p>✨ <em>If you found this project helpful, don’t forget to leave a ⭐ on GitHub!</em>  </p>

