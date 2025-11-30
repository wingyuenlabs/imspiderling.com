---
Title: My Model Cheated: How Grad-CAM Exposed a 95% Accuracy Lie
Description: 
Author: Adarsh Sriuma
Date: 2025-11-30T21:40:05.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  The Project
</h2>

<p>This week I was trying to work on a simple Deep Learning project to get familiar with PyTorch APIs. Since I had just returned from a staycation where I drove a rental car for over 600 miles, I decided to build a "Car Damage Classifier." I kept the goal simple: the model classifies if an uploaded image of a car is "DAMAGED" or "UNDAMAGED".</p>

<p>I couldn't find a ready-to-use dataset, so I created one by merging a damaged car dataset (source: [lplenka/coco-car-damage-detection-dataset]) and a new car dataset (source: [yamaerenay/100-images-of-top-50-car-brands]). I made sure I roughly had the same number of images in my training set to avoid class imbalance.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhmkjguwd6uzrmlxmglh9.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhmkjguwd6uzrmlxmglh9.png" alt="Image showing number of images in dataset" width="501" height="165"></a></p>

<p>I began training using ResNet-18 as the backbone. Just after 10 epochs, my code reported a validation accuracy of 95.08%. I was thrilled. I tested the model with random images from the internet, and it looked like it had truly learned to classify the cars with high confidence.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs3jpm0ztudi2kluvcmz0.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fs3jpm0ztudi2kluvcmz0.png" alt="Image showing the model correctly classifying a damaged car" width="608" height="603"></a></p>

<h2>
  
  
  The Discovery
</h2>

<p>Instead of calling it a day, out of curiosity, I wanted to visualize where the model was looking to make its decision. I implemented Grad-CAM (Gradient-weighted Class Activation Mapping) to draw a heatmap over the image, highlighting the pixels that triggered the classification.</p>

<p>I was expecting a red "hot spot" on the dents, scratches, or broken bumpers.</p>

<p>But the results were the direct opposite.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fezulofsz4d6yfk9hotnv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fezulofsz4d6yfk9hotnv.png" alt="Image of a heatmap overlayed on the picture of a damaged car showing red hotspots in the background and blue spots in the damaged parts of the car" width="794" height="240"></a></p>

<p>I observed the background of the image was red, while the car itself was often blue (ignored).</p>

<p>This was a big "Aha!" moment. The model was cheating, and it was very good at it. Taking a closer look at my data, I realized the bias:</p>

<p>Damaged Cars: Most images were taken in junkyards or accident sites, meaning the backgrounds were cluttered with debris, dirt, and fences.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgoclxpurlpooii1lk81s.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgoclxpurlpooii1lk81s.png" alt="Row of 4 images showing a sample batch from dataset" width="552" height="173"></a></p>

<p>Undamaged Cars: Most images were professional stock photos with clean, smooth backgrounds (showrooms, open roads, nature).</p>

<p>The model wasn't looking at the car at all. It was just checking: "Is the background messy? If yes, then Damaged."</p>

<h2>
  
  
  The "Showroom" Test
</h2>

<p>To verify this, I took a photo of a damaged car that the model correctly classified as "DAMAGED." I used Photoshop to remove the messy background and replaced it with a smooth white gradient (mimicking a showroom).</p>

<p>Sure enough, the model instantly changed its prediction to "UNDAMAGED."</p>

<p>How do I fix this? I am brainstorming ways to force the model to look at the car, not the scenery:</p>

<p>Architecture Change: Employing an Object Detection model (like YOLO) to force the model to look only inside a bounding box around the car.</p>

<p>Preprocessing: Removing the background of all images before training (though this is computationally expensive).</p>

<p>I am curious to learn from other Deep Learning practitioners -how do you usually solve this "background bias" problem in your projects?</p>

