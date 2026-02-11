---
Title: How to Implement Pause and Resume for Large File Uploads in React Using Filestack
Description: 
Author: IderaDevTools
Date: 2026-02-11T20:59:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>Large file uploads can be frustrating for users. Problems like slow internet or a lost connection make it even worse.</p>

<p>Imagine uploading a 2GB video, reaching almost the end, and then the upload fails. You have to start again from the beginning. This is why pause and resume upload features are very important for a good user experience.</p>

<p><em>For a deeper understanding of the challenges with large files, see our</em> <a href="https://blog.filestack.com/complete-guide-handling-large-file-uploads/" rel="noopener noreferrer"><em>complete guide to large file uploads</em></a><em>.</em></p>

<p>In this tutorial, you will learn how to add pause and resume for large file uploads in a React app using Filestack’s SDK.</p>

<p>We will build a simple and complete solution that shows upload progress, handles errors, and manages upload state, everything you need to upload large files smoothly in your React application.</p>

<p>Before we begin building the upload component, let’s quickly look at what you’ll learn from this tutorial.</p>

<h2>
  
  
  <strong>Key Takeaways</strong>
</h2>

<ul>
<li><p>Learn how to implement pause, resume, and cancel uploads using Filestack’s SDK in React.</p></li>
<li><p>Understand how chunked uploads help prevent failed large file uploads.</p></li>
<li><p>Track upload progress using Filestack’s progress callbacks.</p></li>
<li><p>Improve reliability by handling unstable network connections.</p></li>
<li><p>Build a user-friendly upload UI with clear status and controls.</p></li>
</ul>

<p>Now that you know what you’ll build, let’s make sure you have everything ready to follow along.</p>

<h2>
  
  
  <strong>Prerequisites</strong>
</h2>

<p>Before getting into the implementation, make sure you have:</p>

<ul>
<li><p>Basic knowledge of React and React Hooks.</p></li>
<li><p>A Filestack account and an API key (<a href="https://www.filestack.com/signup/" rel="noopener noreferrer">create a Filestack account</a>).</p></li>
<li><p>Node.js and npm are installed on your system.</p></li>
<li><p>A React project already set up (or you can <a href="https://vite.dev/guide/" rel="noopener noreferrer">create a React project using Vite</a>).</p></li>
</ul>

<p><strong><em>💡Tip:</em></strong> <em>If you need to set up basic file uploads in React first, check out our</em> <a href="https://blog.filestack.com/react-file-upload-tutorial-filestack/" rel="noopener noreferrer"><em>basic React file upload tutorial</em></a><em>.</em></p>

<p>Once your environment is ready, it’s important to understand how Filestack makes pause and resume uploads possible.</p>

<h2>
  
  
  <strong>Understanding Chunked Uploads in Filestack</strong>
</h2>

<p>Filestack uses chunked uploads to support pause and resume. This means large files are broken into small parts, and each part is uploaded one by one.</p>

<p>As announced in our <a href="https://blog.filestack.com/filestack-file-picker-4-0-release/" rel="noopener noreferrer">Filestack File Picker 4.0 release</a>, pause and resume functionality was introduced to improve upload reliability and user control. This makes uploads more reliable and user-friendly.</p>

<p>Here’s why this approach is useful:</p>

<ul>
<li><p><strong>Resilience</strong>: If one small part fails, only that part needs to upload again.</p></li>
<li><p><strong>Pause &amp; Resume</strong>: Uploading can stop and continue from the last uploaded part.</p></li>
<li><p><strong>Progress Tracking</strong>: You can easily show upload progress as each part finishes.</p></li>
<li><p><strong>Better for Poor Internet</strong>: Small parts upload more easily on slow or unstable networks.</p></li>
</ul>

<p>When an upload is paused, Filestack remembers which parts are already uploaded. When you resume the upload, it continues from the same point instead of starting from zero.</p>

<p>Now that you understand how chunked uploads work behind the scenes, let’s start implementing this functionality in your React project.</p>

<h2>
  
  
  <strong>Implementing Pause and Resume for Large File Uploads in React</strong>
</h2>

<p>Let’s start adding pause and resume functionality to make uploads more reliable and user-friendly.</p>

<h2>
  
  
  <strong>Step 1: Install and Set Up Filestack SDK</strong>
</h2>

<p>First, install the Filestack JavaScript SDK in your project:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>npm install filestack-js
</code></pre>

</div>



<p>Next, create a configuration file to store your Filestack settings:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// config/filestackConfig.js

export const FILESTACK_API_KEY = 'YOUR_API_KEY_HERE';export const UPLOAD_OPTIONS = {
intelligent: true, // Improves upload performance
intelligentChunkSize: 8 * 1024 * 1024, // Uploads file in 8MB parts
timeout: 60000, // Stops upload if it takes more than 60 seconds
};
</code></pre>

</div>



<p>Here, replace ‘YOUR_API_KEY_HERE’ with your actual Filestack API key.</p>

<p><a href="https://github.com/WebdevShefali/react-filestack-resumable-upload" rel="noopener noreferrer"><em>The complete source code used in this tutorial is available here.</em></a></p>

<p>With the SDK installed and configured, the next step is to build the React component that will manage our upload logic.</p>

<h2>
  
  
  <strong>Step 2: Create the Basic Component Structure</strong>
</h2>

<p>Now, let’s create a basic React component that will handle the file upload and keep track of its state.</p>

<p>Create a new file for the upload component:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// components/ResumableFileUpload.jsx
import React, { useState, useRef } from "react";
import * as filestack from "filestack-js";
import {
FILESTACK_API_KEY,
UPLOAD_OPTIONS,
} from "../../config/filestackConfig";const ResumableFileUpload = () =&gt; {
// State to track file and upload status
const [selectedFile, setSelectedFile] = useState(null);
const [uploadProgress, setUploadProgress] = useState(0);
const [uploadStatus, setUploadStatus] = useState("idle");
// idle, uploading, paused, completed, error
const [uploadedFileUrl, setUploadedFileUrl] = useState(null);
const [errorMessage, setErrorMessage] = useState(null);// Refs to store values between re-renders
const uploadInstanceRef = useRef(null);
const filestackClientRef = useRef(null);

// Create Filestack client when the component loads
React.useEffect(() =&gt; {
filestackClientRef.current = filestack.init(FILESTACK_API_KEY);
console.log("Filestack client initialized");
}, []);
return (
&lt;div className="upload-container"&gt;
&lt;h2&gt;Resumable File Upload&lt;/h2&gt;
&lt;p&gt;Status: {uploadStatus}&lt;/p&gt;
&lt;p&gt;Progress: {uploadProgress}%&lt;/p&gt;
{selectedFile &amp;&amp; &lt;p&gt;File: {selectedFile.name}&lt;/p&gt;}
&lt;/div&gt;
);
};
export default ResumableFileUpload;
</code></pre>

</div>



<p><strong>What this code does:</strong></p>

<ul>
<li><p>Stores the selected file and upload progress.</p></li>
<li><p>Keeps track of the upload state (idle, uploading, paused, etc.).</p></li>
<li><p>Creates a Filestack client once when the component loads.</p></li>
<li><p>Shows basic upload information on the screen.</p></li>
</ul>

<p><strong>How to test step 2:</strong></p>

<p>Import and use the component in App.js:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// App.jsx
import ResumableFileUpload from "./components/ResumableFileUpload";function App() {
return (
&lt;div className="App"&gt;
&lt;ResumableFileUpload /&gt;
&lt;/div&gt;
);
}
export default App;
</code></pre>

</div>



<ul>
<li><p>Start your app using npm run dev.</p></li>
<li><p>Open the browser console and check for the message: <strong>“Filestack client initialised”.</strong></p></li>
<li><p>On the page, you should see:</p></li>
<li><p>Status: idle</p></li>
<li><p>Progress: 0%</p></li>
</ul>

<p>At this point, your basic upload component is ready.</p>

<p>Now that the component structure is ready, let’s allow users to select files for upload.</p>

<h2>
  
  
  <strong>Step 3: Implement File Selection</strong>
</h2>

<p>Now, let’s allow users to choose a file and store it in the component state.</p>

<p>Add this function inside the ResumableFileUpload component (after the useEffect):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const handleFileSelect = (event) =&gt; {
const file = event.target.files[0];if (file) {
setSelectedFile(file);
setUploadProgress(0);
setUploadStatus('idle');
setErrorMessage(null);
setUploadedFileUrl(null);console.log(
'File selected:',
file.name,
'Size:',
(file.size / 1024 / 1024).toFixed(2),
'MB'
);
}
};
</code></pre>

</div>



<p>Next, update the JSX to show a file input and file details:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>return (
&lt;div className="upload-container"&gt;
&lt;h2&gt;Resumable File Upload&lt;/h2&gt;&lt;div className="file-input-section"&gt;
&lt;input
type="file"
onChange={handleFileSelect}
disabled={uploadStatus === 'uploading' || uploadStatus === 'paused'}
accept="*/*"
/&gt;{selectedFile &amp;&amp; (
&lt;p className="file-info"&gt;
Selected: {selectedFile.name} (
{(selectedFile.size / 1024 / 1024).toFixed(2)} MB)
&lt;/p&gt;
)}
&lt;/div&gt;
&lt;p&gt;Status: {uploadStatus}&lt;/p&gt;
&lt;p&gt;Progress: {uploadProgress}%&lt;/p&gt;
&lt;/div&gt;
);
</code></pre>

</div>



<p><strong>What this code does:</strong></p>

<ul>
<li><p>The user selects a file using the file input.</p></li>
<li><p>The selected file is saved in the state.</p></li>
<li><p>Upload progress and errors are reset.</p></li>
<li><p>The file name and size are shown on the screen.</p></li>
<li><p>File selection is disabled while an upload is running or paused.</p></li>
</ul>

<p><strong>How to test step 3:</strong></p>

<ol>
<li><p>Refresh your browser.</p></li>
<li><p>Click the file input and select a file (try a larger file, 50MB+).</p></li>
<li><p>Check the browser console for the “File selected” log with file details.</p></li>
<li><p>Verify the file name and size display correctly below the input.</p></li>
<li><p>Select a different file and confirm the display updates.</p></li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fer5mischnuubpfucoxhq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fer5mischnuubpfucoxhq.png" alt=" " width="800" height="439"></a></p>

<p>After selecting a file, the next step is to actually upload it using Filestack.</p>

<h2>
  
  
  <strong>Step 4: Start the Upload</strong>
</h2>

<p>Now we’ll start uploading the selected file using Filestack’s SDK and create an upload control token.</p>

<p>This token is required to pause, resume, or cancel the upload later.</p>

<h3>
  
  
  <strong>Why Do We Need a Control Token</strong>
</h3>

<p>When you call client.upload(), Filestack allows pause, resume, and cancel only if you pass a control object into the upload call.</p>

<p>This object is then enhanced by the SDK with .pause(), .resume(), and .cancel() methods.</p>

<p><em>If you want to explore all available upload options and methods, you can check the official</em> <a href="https://filestack.github.io/filestack-js/classes/Client.html" rel="noopener noreferrer"><em>Filestack Client API documentation</em></a><em>.</em></p>

<h3>
  
  
  <strong>Add the Upload Function</strong>
</h3>

<p>Add this function after handleFileSelect inside your component:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Add this ref at the top of the component
const uploadControlRef = useRef(null);const startUpload = async () =&gt; {
if (!selectedFile || !filestackClientRef.current) {
console.error("No file selected or Filestack client not initialized");
return;
}setUploadStatus("uploading");
setErrorMessage(null);

// Create a control token object
const uploadControl = {};
uploadControlRef.current = uploadControl;
const uploadOptions = {
...UPLOAD_OPTIONS,
onProgress: (event) =&gt; {
const percent = Math.round(event.totalPercent || 0);
setUploadProgress(percent);
},
};
try {
const result = await filestackClientRef.current.upload(
selectedFile,
uploadOptions,
{ filename: selectedFile.name },
uploadControl // 👈 IMPORTANT: pass the control token
);
setUploadedFileUrl(
`https://cdn.filestackcontent.com/${result.handle}`
);
setUploadStatus("completed");
setUploadProgress(100);
} catch (error) {
setUploadStatus("error");
setErrorMessage(error.message || "Upload failed");
console.error("Upload error:", error);
}
};
</code></pre>

</div>



<h3>
  
  
  <strong>Add the Start Upload Button</strong>
</h3>

<p>Update your JSX to include the button:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{selectedFile &amp;&amp; uploadStatus === 'idle' &amp;&amp; (
&lt;button
onClick={startUpload}
style={{ padding: '10px 20px', fontSize: '16px' }}
&gt;
Start Upload
&lt;/button&gt;
)}
</code></pre>

</div>



<p>Also, show the uploaded file link after completion:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{uploadedFileUrl &amp;&amp; (
&lt;p&gt;
&lt;a href={uploadedFileUrl} target="_blank" rel="noopener noreferrer"&gt;
View uploaded file
&lt;/a&gt;
&lt;/p&gt;
)}
</code></pre>

</div>



<p><strong>What this code does:</strong></p>

<ul>
<li><p>User selects a file.</p></li>
<li><p>User clicks <strong>Start Upload.</strong></p></li>
<li><p>Filestack uploads the file in chunks.</p></li>
<li><p>Tracks upload progress in real time.</p></li>
<li><p>Creates a <strong>control token</strong> that enables pause, resume, and cancel.</p></li>
<li><p>Stores the uploaded file URL on success.</p></li>
<li><p>Handles errors safely using try/catch.</p></li>
</ul>

<p><strong>How to test step 4:</strong></p>

<ol>
<li><p>Select a small file (5–10MB).</p></li>
<li><p>Click <strong>Start Upload.</strong></p></li>
<li><p>Watch the progress percentage increase.</p></li>
<li><p>Wait for the upload to finish.</p></li>
<li><p>Status should be <strong>completed.</strong></p></li>
<li><p>Progress should show <strong>100%.</strong></p></li>
<li><p>Click <strong>View uploaded file</strong> and confirm it opens.</p></li>
</ol>

<p>At this point, the upload is working and ready to be controlled.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkj8jkcwkcnhfyb9zf6mr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkj8jkcwkcnhfyb9zf6mr.png" alt=" " width="800" height="404"></a></p>

<p>The upload is now working, but users still cannot control it. Let’s add pause, resume, and cancel functionality next.</p>

<h2>
  
  
  <strong>Step 5: Add Pause, Resume, and Cancel Functionality</strong>
</h2>

<p>Now that we have a control token, we can pause, resume, or cancel the upload at any time.</p>

<p>These actions work because Filestack uploads files in chunks and keeps track of which chunks are already uploaded.</p>

<h3>
  
  
  <strong>Add Pause, Resume, and Cancel Functions</strong>
</h3>

<p>Add these functions after startUpload inside your component:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>// Add this ref near the top of your component
const isCancelledRef = useRef(false);const pauseUpload = () =&gt; {
if (uploadControlRef.current &amp;&amp; uploadStatus === "uploading") {
uploadControlRef.current.pause();
setUploadStatus("paused");
console.log("Upload paused at", uploadProgress + "%");
}
};

const resumeUpload = () =&gt; {
if (uploadControlRef.current &amp;&amp; uploadStatus === "paused") {
uploadControlRef.current.resume();
setUploadStatus("uploading");
console.log("Upload resumed from", uploadProgress + "%");
}
};
const cancelUpload = () =&gt; {
if (uploadControlRef.current) {
// Mark this as a user-initiated cancellation
isCancelledRef.current = true;
uploadControlRef.current.cancel();
uploadControlRef.current = null;
setUploadStatus("idle");
setUploadProgress(0);
setErrorMessage(null);
console.log("Upload cancelled by user");
}
};
</code></pre>

</div>



<p><strong>What these functions do:</strong></p>

<ul>
<li><p><strong>pauseUpload:</strong> Stops uploading new chunks without losing progress.</p></li>
<li><p><strong>resumeUpload:</strong> Continues uploading from the last completed chunk.</p></li>
<li><p><strong>cancelUpload:</strong> Stops the upload completely and resets the state.</p></li>
</ul>

<h3>
  
  
  <strong>Update the startUpload Error Handling</strong>
</h3>

<p>Update the catch block inside startUpload:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>try {
const result = await filestackClientRef.current.upload(
selectedFile,
uploadOptions,
{ filename: selectedFile.name },
uploadControl
);

setUploadedFileUrl(
`https://cdn.filestackcontent.com/${result.handle}`
);
setUploadStatus("completed");
setUploadProgress(100);
} catch (error) {
// If the user cancelled, do not treat it as an error
if (isCancelledRef.current) {
console.log("Upload cancelled, skipping error state");
isCancelledRef.current = false;
return;
}
setUploadStatus("error");
setErrorMessage(error.message || "Upload failed");
console.error("Upload error:", error);
}
</code></pre>

</div>



<h3>
  
  
  <strong>Update the UI with Control Buttons</strong>
</h3>

<p>Now update your JSX to show the correct buttons based on upload status:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{selectedFile &amp;&amp; uploadStatus === "idle" &amp;&amp; (
&lt;button
onClick={startUpload}
style={{ padding: "10px 20px", margin: "5px" }}
&gt;
Start Upload
&lt;/button&gt;
)}

{uploadStatus === "uploading" &amp;&amp; (
&lt;div&gt;
&lt;button
onClick={pauseUpload}
style={{ padding: "10px 20px", margin: "5px" }}
&gt;
Pause
&lt;/button&gt;
&lt;button
onClick={cancelUpload}
style={{ padding: "10px 20px", margin: "5px" }}
&gt;
Cancel
&lt;/button&gt;
&lt;/div&gt;
)}
{uploadStatus === "paused" &amp;&amp; (
&lt;div&gt;
&lt;button
onClick={resumeUpload}
style={{ padding: "10px 20px", margin: "5px" }}
&gt;
Resume
&lt;/button&gt;
&lt;button
onClick={cancelUpload}
style={{ padding: "10px 20px", margin: "5px" }}
&gt;
Cancel
&lt;/button&gt;
&lt;/div&gt;
)}
</code></pre>

</div>



<p>Keep the status and progress display:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>&lt;p&gt;Status: {uploadStatus}&lt;/p&gt;
&lt;p&gt;Progress: {uploadProgress}%&lt;/p&gt;{errorMessage &amp;&amp; &lt;p style={{ color: "red" }}&gt;Error: {errorMessage}&lt;/p&gt;}{uploadedFileUrl &amp;&amp; (
&lt;p&gt;
&lt;a href={uploadedFileUrl} target="_blank" rel="noopener noreferrer"&gt;
View uploaded file
&lt;/a&gt;
&lt;/p&gt;
)}
</code></pre>

</div>



<p><strong>What’s happening here:</strong></p>

<ul>
<li><p>Filestack uploads the file in small chunks.</p></li>
<li><p>When you pause, it stops uploading new chunks.</p></li>
<li><p>When you resume, it continues from the last uploaded chunk.</p></li>
<li><p>Already uploaded data is not lost.</p></li>
<li><p>Cancelling stops the upload entirely.</p></li>
</ul>

<p><strong>How to Test Step 5:</strong></p>

<ul>
<li><p>Select a large file (100MB+ recommended for this test).</p></li>
<li><p>Click “Start Upload”.</p></li>
<li><p><strong>Pause test:</strong></p></li>
<li><p>Pause around 30–40%.</p></li>
<li><p>Status changes to paused.</p></li>
<li><p>Progress stops updating.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhorbw2n0klxrmo2akv3j.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhorbw2n0klxrmo2akv3j.png" alt=" " width="800" height="404"></a></p>

<ul>
<li><p><strong>Resume test:</strong></p></li>
<li><p>Click <strong>Resume.</strong></p></li>
<li><p>Status changes to uploading.</p></li>
<li><p>Progress continues at the same percentage.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnd7mw70edtvf78o1sawv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnd7mw70edtvf78o1sawv.png" alt=" " width="800" height="404"></a></p>

<ul>
<li><p><strong>Multiple pauses:</strong></p></li>
<li><p>Pause and resume several times.</p></li>
<li><p>Progress should never reset.</p></li>
<li><p><strong>Cancel test:</strong></p></li>
<li><p>Start a new upload.</p></li>
<li><p>Click <strong>Cancel.</strong></p></li>
<li><p>Status returns to idle.</p></li>
<li><p>Progress resets to 0%.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe0vh3qj3gb40xg1a3o84.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe0vh3qj3gb40xg1a3o84.png" alt=" " width="800" height="404"></a></p>

<p>Now that users can control uploads, let’s improve the user experience by showing a visual progress bar.</p>

<h2>
  
  
  <strong>Step 6: Add Progress Bar Visualisation</strong>
</h2>

<p>Now let’s improve the user experience by adding a visual progress bar.</p>

<p>This makes it easy for users to see how much of the file has been uploaded and what’s happening during pause and resume.</p>

<p>The progress bar will be driven by Filestack’s onProgress callback, which updates as each upload chunk completes.</p>

<h3>
  
  
  <strong>Update the UI to Show a Progress Bar</strong>
</h3>

<p>Replace the return statement in your component with the following JSX:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>return (
&lt;div
className="upload-container"
style={{
maxWidth: "600px",
margin: "40px auto",
padding: "30px",
border: "2px solid #e0e0e0",
borderRadius: "8px",
}}
&gt;
&lt;h2 style={{ textAlign: "center" }}&gt;
Resumable File Upload
&lt;/h2&gt;&lt;div className="file-input-section"&gt;
&lt;input
type="file"
onChange={handleFileSelect}
disabled={
uploadStatus === "uploading" ||
uploadStatus === "paused"
}
accept="*/*"
style={{
display: "block",
width: "100%",
padding: "10px",
marginBottom: "10px",
}}
/&gt;{selectedFile &amp;&amp; (
&lt;p style={{ color: "#666", fontSize: "14px" }}&gt;
Selected: {selectedFile.name} (
{(selectedFile.size / 1024 / 1024).toFixed(2)} MB)
&lt;/p&gt;
)}
&lt;/div&gt;

{selectedFile &amp;&amp; uploadStatus === "idle" &amp;&amp; (
&lt;button
onClick={startUpload}
style={{
padding: "10px 20px",
margin: "5px",
backgroundColor: "#007bff",
color: "white",
border: "none",
borderRadius: "4px",
cursor: "pointer",
}}
&gt;
Start Upload
&lt;/button&gt;
)}
{uploadStatus === "uploading" &amp;&amp; (
&lt;div style={{ marginTop: "20px" }}&gt;
&lt;button
onClick={pauseUpload}
style={{
padding: "10px 20px",
margin: "5px",
backgroundColor: "#ffc107",
border: "none",
borderRadius: "4px",
cursor: "pointer",
}}
&gt;
Pause
&lt;/button&gt;
&lt;button
onClick={cancelUpload}
style={{
padding: "10px 20px",
margin: "5px",
backgroundColor: "#dc3545",
color: "white",
border: "none",
borderRadius: "4px",
cursor: "pointer",
}}
&gt;
Cancel
&lt;/button&gt;
&lt;/div&gt;
)}
{uploadStatus === "paused" &amp;&amp; (
&lt;div style={{ marginTop: "20px" }}&gt;
&lt;button
onClick={resumeUpload}
style={{
padding: "10px 20px",
margin: "5px",
backgroundColor: "#28a745",
color: "white",
border: "none",
borderRadius: "4px",
cursor: "pointer",
}}
&gt;
Resume
&lt;/button&gt;
&lt;button
onClick={cancelUpload}
style={{
padding: "10px 20px",
margin: "5px",
backgroundColor: "#dc3545",
color: "white",
border: "none",
borderRadius: "4px",
cursor: "pointer",
}}
&gt;
Cancel
&lt;/button&gt;
&lt;/div&gt;
)}
{uploadStatus !== "idle" &amp;&amp; uploadStatus !== "error" &amp;&amp; (
&lt;div style={{ marginTop: "20px" }}&gt;
&lt;div
style={{
width: "100%",
height: "30px",
backgroundColor: "#e9ecef",
borderRadius: "4px",
overflow: "hidden",
}}
&gt;
&lt;div
style={{
width: `${uploadProgress}%`,
height: "100%",
backgroundColor: "#007bff",
transition:
uploadStatus === "paused"
? "none"
: "width 0.3s ease",
display: "flex",
alignItems: "center",
justifyContent: "center",
color: "white",
fontWeight: "bold",
}}
&gt;
{uploadProgress}%
&lt;/div&gt;
&lt;/div&gt;
&lt;p
style={{
textAlign: "center",
marginTop: "10px",
color: "#666",
}}
&gt;
Status:{" "}
{uploadStatus.charAt(0).toUpperCase() +
uploadStatus.slice(1)}
&lt;/p&gt;
&lt;/div&gt;
)}
{uploadStatus === "error" &amp;&amp; errorMessage &amp;&amp; (
&lt;div
style={{
padding: "15px",
backgroundColor: "#f8d7da",
border: "1px solid #f5c6cb",
borderRadius: "4px",
color: "#721c24",
marginTop: "20px",
}}
&gt;
&lt;p&gt;Error: {errorMessage}&lt;/p&gt;
&lt;/div&gt;
)}
{uploadStatus === "completed" &amp;&amp; uploadedFileUrl &amp;&amp; (
&lt;div
style={{
padding: "15px",
backgroundColor: "#d4edda",
border: "1px solid #c3e6cb",
borderRadius: "4px",
color: "#155724",
marginTop: "20px",
}}
&gt;
&lt;p&gt;Upload completed successfully!&lt;/p&gt;
&lt;a
href={uploadedFileUrl}
target="_blank"
rel="noopener noreferrer"
style={{
color: "#0056b3",
textDecoration: "underline",
}}
&gt;
View uploaded file
&lt;/a&gt;
&lt;/div&gt;
)}
&lt;/div&gt;
);
</code></pre>

</div>



<p><strong>What this code does:</strong></p>

<ul>
<li><p>A visual progress bar that fills as the upload progresses.</p></li>
<li><p>Smooth animation while uploading.</p></li>
<li><p>Progress stops visually when paused.</p></li>
<li><p>Progress resumes smoothly after the resume.</p></li>
<li><p>Clear success and error feedback.</p></li>
<li><p>Consistent UI state after cancel or retry.</p></li>
</ul>

<p><strong>How to Test Step 6:</strong></p>

<ul>
<li><p>Refresh the page and verify the new layout looks clean.</p></li>
<li><p>Select a file and confirm that the file name and size are shown.</p></li>
<li><p>Click <strong>Start Upload.</strong></p></li>
<li><p>Watch the progress bar:</p></li>
<li><p>It should fill from left to right.</p></li>
<li><p>Percentage text should stay centered.</p></li>
<li><p>Click <strong>Pause:</strong></p></li>
<li><p>The progress bar should stop moving.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fizhb6e7uad3u9nrvg5r2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fizhb6e7uad3u9nrvg5r2.png" alt=" " width="800" height="404"></a></p>

<ul>
<li><p>Click <strong>Resume:</strong></p></li>
<li><p>Progress should continue from the same point.</p></li>
<li><p>Complete the upload:</p></li>
<li><p>The success message should appear in green.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F78idf75apo2cjmlnam18.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F78idf75apo2cjmlnam18.png" alt=" " width="800" height="404"></a></p>

<p>The upload UI now looks much better. Next, we’ll make uploads more reliable by handling network interruptions automatically.</p>

<h2>
  
  
  <strong>Step 7: Handle Network Interruptions Automatically</strong>
</h2>

<p>Large file uploads can fail when the internet connection is slow or unstable.</p>

<p>To make uploads more reliable, we’ll check when the internet goes offline or comes back online and automatically pause the upload or resume it at the right time.</p>

<h3>
  
  
  <strong>Add Network-related State and Refs</strong>
</h3>

<p>Add these near the top of the component:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>const [isOnline, setIsOnline] = useState(navigator.onLine);
const wasPausedByNetworkRef = useRef(false);
</code></pre>

</div>



<h3>
  
  
  <strong>Monitor Network Status</strong>
</h3>

<p>Add this useEffect after Filestack initialisation:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>React.useEffect(() =&gt; {
const handleOnline = () =&gt; {
setIsOnline(true);
console.log("Network connection restored");// Resume only if pause was caused by network loss
if (
wasPausedByNetworkRef.current &amp;&amp;
uploadStatus === "paused"
) {
console.log("Auto-resuming upload after network recovery");
wasPausedByNetworkRef.current = false;
resumeUpload();
}
};

const handleOffline = () =&gt; {
setIsOnline(false);
console.log("Network connection lost");
// Pause only if currently uploading
if (uploadStatus === "uploading") {
console.log("Auto-pausing upload due to network loss");
wasPausedByNetworkRef.current = true;
pauseUpload();
}
};
window.addEventListener("online", handleOnline);
window.addEventListener("offline", handleOffline);
return () =&gt; {
window.removeEventListener("online", handleOnline);
window.removeEventListener("offline", handleOffline);
};
}, [uploadStatus]);
</code></pre>

</div>



<h3>
  
  
  <strong>Reset Network Flags When Upload Ends</strong>
</h3>

<p>Update your startUpload success and cancel flows:</p>

<p><strong>On successful upload:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>setUploadStatus("completed");
setUploadProgress(100);
wasPausedByNetworkRef.current = false;
</code></pre>

</div>



<p><strong>On cancel:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>wasPausedByNetworkRef.current = false;
</code></pre>

</div>



<p>This prevents stale network state from affecting future uploads.</p>

<h3>
  
  
  <strong>Show Network Warning in the UI</strong>
</h3>

<p>Add this near the top of your return JSX:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>{!isOnline &amp;&amp; (
&lt;div
style={{
padding: "10px",
backgroundColor: "#fff3cd",
border: "1px solid #ffc107",
borderRadius: "4px",
color: "#856404",
marginBottom: "20px",
textAlign: "center",
fontSize: "14px",
}}
&gt;
⚠️ No internet connection. Upload will resume automatically when
the connection is restored.
&lt;/div&gt;
)}
</code></pre>

</div>



<p><strong>What this code does:</strong></p>

<ul>
<li><p>The browser tells us when the internet goes offline or comes back online.</p></li>
<li><p>If the internet disconnects while a file is uploading:</p></li>
<li><p>We pause the upload.</p></li>
<li><p>We remember that the pause happened because of the network.</p></li>
<li><p>When the internet is back:</p></li>
<li><p>We resume the upload only if it was paused because of the network.</p></li>
<li><p>If the user paused the upload themselves, we do nothing.</p></li>
</ul>

<p>This way, the app respects what the user wants and only auto-resumes when it makes sense.</p>

<p><strong>How to test step 7:</strong></p>

<ul>
<li><p><strong>Offline detection:</strong></p></li>
<li><p>Open DevTools → Network tab.</p></li>
<li><p>Switch to <strong>Offline.</strong></p></li>
<li><p>Verify the warning banner appears.</p></li>
<li><p>The console shows “Network connection lost”.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxygor5rycuc77e3ehlwc.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxygor5rycuc77e3ehlwc.png" alt=" " width="800" height="404"></a></p>

<ul>
<li><p><strong>Auto-pause:</strong></p></li>
<li><p>Start uploading a large file (200MB+).</p></li>
<li><p>Go offline at ~30%.</p></li>
<li><p>Upload pauses automatically.</p></li>
<li><p>Status changes to paused.</p></li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2kwhgbpiovuakr237hip.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2kwhgbpiovuakr237hip.png" alt=" " width="800" height="404"></a></p>

<ul>
<li><p><strong>Auto-resume:</strong></p></li>
<li><p>Switch back to <strong>No throttling.</strong></p></li>
<li><p>Upload resumes automatically.</p></li>
<li><p>Progress continues from the same point.</p></li>
<li><p><strong>Manual pause safety:</strong></p></li>
<li><p>Start upload.</p></li>
<li><p>Click <strong>Pause.</strong></p></li>
<li><p>Go offline → online.</p></li>
<li><p>Upload does <strong>NOT</strong> resume automatically.</p></li>
<li><p>Click <strong>Resume</strong> manually to continue.</p></li>
</ul>

<p>Before finishing, let’s review some important best practices to keep your upload feature reliable and user-friendly.</p>

<h2>
  
  
  <strong>Best Practices for Resumable Large File Upload</strong>
</h2>

<p>Following a few simple best practices can make large file uploads faster, more reliable, and easier for users to manage, especially when working with pause and resume functionality.</p>

<ul>
<li><p><strong>Use chunked uploads:</strong> Break large files into smaller parts, so uploads are more reliable and can resume if they stop.</p></li>
<li><p><strong>Store upload controls properly:</strong> Save the upload control object using useRef so pause, resume, and cancel keep working even when the component re-renders.</p></li>
<li><p><strong>Track upload status clearly:</strong> Keep track of whether the upload is idle, uploading, paused, completed, or failed so the UI behaves correctly.</p></li>
<li><p><strong>Show upload progress:</strong> Add a progress bar and status text so users can easily see how much of the file is uploaded.</p></li>
<li><p><strong>Handle internet issues smartly:</strong> Pause the upload if the internet disconnects and resume it when the connection returns, without interrupting user actions.</p></li>
</ul>

<h2>
  
  
  <strong>Conclusion</strong>
</h2>

<p>Uploading large files can be difficult, especially when the internet is slow or keeps disconnecting. By adding pause and resume using Filestack’s SDK, you can make uploads smoother and more reliable.</p>

<p>In this tutorial, you built a React upload component that supports chunked uploads, shows real-time progress, allows users to pause and resume uploads, and handles network interruptions. These features help reduce upload failures and give users better control over their uploads.</p>

<p>Using these techniques, you can build upload features that work well with large files and provide a better experience for users.</p>

<p><strong><em>Originally published on the</em></strong> <a href="https://blog.filestack.com/pause-resume-large-file-uploads-react-filestack/" rel="noopener noreferrer"><strong><em>Filestack blog</em></strong></a><strong><em>.</em></strong></p>

