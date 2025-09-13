---
Title: 🚀 Upgrading & Utilising My Model (ML/AI Integration Series)
Description: 
Author: Hassam Fathe Muhammad
Date: 2025-09-13T21:33:23.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  Continuing from: “🚀 My First Step Towards AI/ML Model Integration | Inspire Sphere”
</h2>

<p>Back then, the ML/AI model I deployed to get categories of the quotes written by users was trained on data using the MultinomialNB algorithm from naive_bayes. The accuracy of my model was not more than <strong>17%</strong>, mainly due to several incompatibilities and the absence of supportive parameters in the <strong>TFIDF Vectorizer</strong>, as well as an inappropriate use of MultinomialNB.</p>

<p>This algorithm works well <strong>only when the features (words)</strong> in the document are <strong>independent</strong> of each other, as it calculates probabilities by taking the <strong>product of individual probabilities</strong>. It's a good fit where text can be classified based on <strong>specific independent flagged words</strong>, like in detecting spam or fraud in an email.</p>

<p>However, my previous model/system lacked important TfidfVectorizer parameters like:</p>

<ul>
<li>stop_words</li>
<li>ngram_range</li>
<li>max_df</li>
<li>min_df</li>
</ul>

<p>These parameters help in converting the text into a <strong>cleaner and more structured numeric form</strong>, making it more ready for classification.</p>




<h2>
  
  
  What’s New Now?
</h2>

<p>So, the next algorithm to take the place of MultinomialNB is <strong>Logistic Regression</strong> from linear_model. Unlike the former, this doesn’t assume that features are independent — it <strong>considers relationships between word appearances</strong> in the document.</p>

<p>The results were notably better. When my <strong>first model</strong> categorized this quote on Inspire Sphere as:</p>

<p><em>Category: Love “A wolf saw me when I was alone… I was that wolf”</em></p>

<p>Then the <strong>improved model</strong> categorized it as:</p>

<p><em>Category: Humour</em></p>

<p>That was quite a significant and meaningful difference.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx2h05pxu3stghh00k3kp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx2h05pxu3stghh00k3kp.png" alt=" " width="800" height="450"></a></p>




<h2>
  
  
  Why This Upgrade Mattered
</h2>

<p>After upgrading my model, I realized that <strong>the real value of an AI model also depends on the data processing and how it interacts with real users.</strong></p>

<p>I made the model predict the <strong>category of quotes written by users on Inspire Sphere to auto-fill the title input field.</strong> This made the platform look more professional and supportive — helping users feel guided and saving time.</p>

