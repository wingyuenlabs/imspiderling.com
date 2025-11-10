---
Title: Fundamental Rewrite of the WebFormsJS Library in WebForms Core 2
Description: 
Author: Elanat Framework
Date: 2025-11-10T21:55:20.000Z
Robots: noindex,nofollow
Template: index
---
<p>In version 2 of <a href="https://elanat.net/page_content/web_forms_core" rel="noopener noreferrer">WebForms Core</a> technology, the client-side <a href="https://github.com/webforms-core/Web_forms" rel="noopener noreferrer">WebFormsJS</a> library has been fundamentally rewritten. This is in addition to adding new features and capabilities.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2mk0l6pdzztiiqoiwq8j.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2mk0l6pdzztiiqoiwq8j.jpg" alt="Rewrite WebFormsJS Library" width="800" height="533"></a></p>

<h2>
  
  
  Improved browser support checking for modern JavaScript features.
</h2>

<p>In this release, we have improved browser support checking and created a list to log APIs and features that are not supported by the browser. This structure is designed to check browser support for a set of modern web features. First, a list of important features is defined, including DOM and form APIs such as <code>FormData</code>, ES6 JavaScript language features such as <code>let/const</code>, <code>Promise</code>, <code>Map</code>, <code>Set</code> and <code>BigInt</code>, web APIs such as <code>fetch</code>, <code>WebSocket</code>, <code>ServiceWorker</code> and <code>WebRTC</code>, UI observers such as <code>IntersectionObserver</code> and <code>MutationObserver</code>, media and sharing features such as <code>Clipboard API</code> and <code>Web Share API</code>, local storage (<code>localStorage</code>) and internationalization support (<code>Intl API</code>). The user's browser is then checked to see which of these features are not supported, and if there are any incompatibilities, a warning is given that the browser is outdated and may have poor performance when running WebForms Core-based applications.</p>

<h2>
  
  
  Improved data encoding for GET (and similar) method submissions.
</h2>

<p>In this release, we fixed a bug in sending data via the GET method by implementing the <code>encodeURIComponent</code> function. The <code>encodeURIComponent</code> function is used to encode a portion of a URL to ensure that the data is correctly and without errors in the URL. This function converts characters such as spaces (<code></code>), equals signs (<code>=</code>), question marks (<code>?</code>), ampersands (<code>&amp;</code>), and other special characters into a safe form so that the browser treats them as data, not as commands or delimiters. For example, if you want to send a form value or parameter in a URL, using <code>encodeURIComponent</code> ensures that the value is transmitted correctly and the URL structure is not broken. This function is usually used in combination with <code>encodeURI</code> or when constructing a query string for HTTP requests.</p>

<h2>
  
  
  Added error display section in the Message section of WebFormsJS.
</h2>

<p>In this version, it is possible to customize errors by integrating the connection error display in the WebFormsJS settings section. For error display, the <code>cb_ShowConnectionError</code> function has been added to handle and display network connection errors, especially in <code>xmlHttpRequest</code> requests. This function generates an error message titled "Connection Error" and, if there is an error code, it is added to the message and displayed to the user via the <code>cb_ShowMessage</code> function. This message usually appears as a warning or notification in the user interface to inform the user of the connection problem. This method is fully editable; you can change the way the message is displayed, the error text, the message type (e.g. warning, error, notification), the display duration, or even side behaviors such as logging or sending to the server. This flexibility allows you to tailor the user experience to suit the needs of your project or audience.</p>

<h2>
  
  
  Improved WebSocket data sending
</h2>

<p>In this update, sending data via WebSocket has been improved to better recognize the type of data being sent. By adding the string ``form=true&amp;'' to the beginning of the serialized data, it is clear that this data is the result of a form submission. This helps the server or receiver process the received data as form information, and not as regular data or other messages. This method increases the accuracy of handling different data types in WebSocket communications and allows for appropriate responses to form requests.</p>

<blockquote>
<p>Note: The <a href="https://elanat.net/page_content/code_behind" rel="noopener noreferrer">CodeBehind</a> 4.4 update will also coincide with the release of <a href="https://github.com/elanatframework/Code_behind" rel="noopener noreferrer">WebForms Core</a> 2. On the server side, we also support this structure in CodeBehind and initialize the form data by recognizing the string ``form=true&amp;''. Please note that for WebSocket sends to different backends, you need to manage this structure yourself.</p>
</blockquote>

<h2>
  
  
  Retry Mechanism
</h2>

<p>The Retry Mechanism in the WebFormsJS library allows the system to automatically retry a network request (such as <code>xmlHttpRequest</code>) if it fails. This feature is controlled by three settings:</p>

<ul>
<li><p><code>UseRetryRequest</code>: whether retry is enabled or disabled;</p></li>
<li><p><code>MaxRetryCount</code>: the maximum number of times we are allowed to retry;</p></li>
<li><p><code>RetryRequestInterval</code>: the time interval between each retry (in milliseconds).</p></li>
</ul>

<p>If this feature is enabled and the maximum number of retries is not reached, the request is retried with the specified delay. If the attempts fail, a connection error message is displayed and UI elements such as the progress bar and button type are reset.</p>

<h2>
  
  
  New Logs
</h2>

<p>In this release, more logs are provided for better error analysis, which helps developers identify and fix the source of problems faster. The effective use of the <code>try-catch</code> structure has led to errors not only being correctly identified, but also being recorded and managed in more detail. This approach allows for more detailed analysis of system behavior in critical situations, which increases the stability and reliability of the application. Also, by examining the logs, developers can identify error patterns and provide more optimal solutions to prevent their recurrence.</p>

<p>In this release, a set of warning messages (<code>console.warn</code>) and error notifications in the WebFormsJS library have been designed to more accurately report execution problems. The messages specifically refer to errors that occur when executing commands related to initialization, storage, condition checking, method calls, or module loading. Also, restrictions such as the <code>eval</code> method being disabled or attempts to access methods and modules that are not allowed are detected and warned. This reporting structure helps developers to detect, analyze, and fix errors faster.</p>

<h2>
  
  
  New settings for displaying messages
</h2>

<p>In the new version of WebFormsJS, the settings for displaying messages have been improved directly in the user interface (not in the console) to inform about various system states in a clear and understandable way. These settings include the duration of displaying messages (<code>MessageDuration</code>) and enabling or disabling specific messages for events such as connection error (<code>ConnectionErrorMessage</code>), WebSocket connection start or stop, WebSocket errors, and various SSE (Server-Sent Events) related states. For each event, if the corresponding option is enabled, the <code>cb_ShowMessage</code> function displays an appropriate message of a specific type (such as <code>success'',</code>problem'', <code>help'' or</code>none'') on the screen.</p>

<p>In the new version of WebFormsJS, displaying user-friendly messages has been widely implemented in key parts of the system. These parts include specifying form elements, setting inputs, storing data, checking logical conditions, and retrieving values. For each of these operations, specific settings are defined in <code>WebFormsOptions</code> that, if enabled, display specific messages in the user interface via the <code>cb_ShowMessage</code> function when an error or incomprehensible execution occurs. These messages can include warnings, help, or information and help the user to be aware of the system status. This approach not only increases transparency in the internal processes of the application, but also improves the user experience and makes troubleshooting easier for developers. Also, by being able to enable or disable these messages, developers can have precise control over the amount and type of feedback displayed and adjust them according to the needs of the project or the level of access of users.</p>

<p>In the new version of WebFormsJS, the display settings of user-friendly messages can be controlled separately for different parts of the system. Below is the complete list of these sections:</p>

<h3>
  
  
  ✅ List of sections that have message settings:
</h3>

<h4>
  
  
  📦 Basic Form Operations
</h4>

<ul>
<li>Element Detection</li>
<li>Set Value to Input</li>
<li>Fetch Value</li>
<li>Save Value</li>
<li>Check Condition</li>
</ul>

<h4>
  
  
  🌐 Network Communications
</h4>

<ul>
<li>Connection Error</li>
<li>WebSocket:</li>
<li>Initializing</li>
<li>Open</li>
<li>Close</li>
<li>Error</li>
<li>SSE (Server-Sent Events):</li>
<li>Initializing</li>
<li>Connect</li>
<li>Disconnect</li>
<li>Reconnect</li>
<li>Close</li>
<li>CloseAll</li>
</ul>

<h4>
  
  
  ⚠️ Incomprehensible Errors
</h4>

<ul>
<li>Incomprehensible WebForms Initialization</li>
<li>Incomprehensible Inputs Initialization</li>
<li>Get Incomprehensible value</li>
<li>Incomprehensible storage</li>
<li>Incomprehensible condition check</li>
</ul>

<h4>
  
  
  ❌ Execution errors
</h4>

<ul>
<li>Error in initializing WebForms</li>
<li>Error in initializing inputs</li>
<li>Error in receiving value</li>
<li>Error in storage</li>
<li>Error in condition check</li>
</ul>

<h2>
  
  
  Support for submit buttons
</h2>

<p>In this version, submit buttons will also receive the PostBack method like submit inputs, which means that when these buttons are clicked, the request sent to the server is made via the PostBack method and the form submission and data processing process is done correctly and seamlessly. This structure allows developers to easily and without the need for special settings, use all form elements uniformly to send data to the server, which increases the efficiency and capabilities of web application development.</p>

<h2>
  
  
  Rewrite the server connection section to record errors and network problems.
</h2>

<p>In this version of WebFormsJS, the server connection section has been rewritten to handle HTTP responses with more accuracy and flexibility. One of the important changes is <strong>Adding support for the range of success statuses between 200 and 299</strong>; In previous versions, only status 200 could be checked, but now all successful responses (such as 201 Created or 206 Partial Content) are correctly recognized and processed. Also, special statuses such as 202 (Accepted) and 204 (No Content) are handled separately to avoid unnecessary processing.</p>

<p>In addition, the error handling structure has been improved:</p>

<ul>
<li>
<strong>Client errors (400–499)</strong> are accompanied by a connection error message to the user.</li>
<li>
<strong>Server errors (500–599)</strong> first trigger a retry (if set), and if unsuccessful, an error message is displayed.</li>
<li>In both cases, form elements are reset and loading is stopped to prevent UI hangs.</li>
</ul>

<p>This rewrite has increased stability, accuracy in detecting response status, and improved user experience in the face of network errors.</p>

<h2>
  
  
  WebFormsJS library reload warning.
</h2>

<p>In this version of WebFormsJS, the warning mechanism for reloading the library is intelligently designed to prevent serious performance errors. If the library has already been loaded and is called again, the system automatically warns in the console that this action can cause code execution disruption and instability in application behavior. This warning helps developers to more quickly identify and fix problems caused by incorrect configuration or repeated loading. Such an approach plays an important role in maintaining system integrity and preventing unwanted collisions in complex environments.</p>

<h2>
  
  
  Rewrite of the serialization method (for submitting data)
</h2>

<p>In the new version of WebFormsJS, the serialization method of forms for submitting data has been completely rewritten to bring its behavior as close as possible to the default HTML behavior. This rewrite includes several important improvements:</p>

<p>✅ Ignoring disabled fields: Now fields with the disabled attribute are excluded from the serialization process, even if they are inside a disabled fieldset. This behavior is in line with the HTML standard and prevents invalid data from being submitted.</p>

<p>✅ Output field support: For the first time, output fields are also supported in serialization. This feature allows developers to include calculated or representational values ​​in the submitted data.</p>

<p>This rewrite has made the serialization method in WebFormsJS not only more accurate and standardized, but also provides more flexibility to manage complex forms and optimize data submission.</p>

<h2>
  
  
  Improved Settings Section Structure
</h2>

<p>In the new version of WebFormsJS, the settings section structure has been significantly improved and organized into categories. This change has made managing settings much easier, more readable, and more maintainable. Instead of placing all the options scattered around, settings are now grouped into specific groups such as <strong>Initialization</strong>, <strong>Send</strong>, <strong>Response</strong>, <strong>State</strong>, <strong>Message</strong>, <strong>Style</strong>, <strong>Console Message</strong>, <strong>Non-Response Management</strong>, <strong>Animation</strong>, <strong>Compress</strong>, and <strong>Security</strong>.</p>

<p>This new structure not only helps developers find and change the settings they need faster, but also allows for extensibility and the addition of new options. Each category clearly indicates the purpose and use of the settings in that section; For example, message settings are now under the <code>Message</code> section, and security settings are now under the <code>Security</code> section.</p>

<p>This redesign is a significant step towards making the library more professional and more usable in large, complex projects. Developers can now fine-tune the behavior of the system to their specific needs with more precision and control, providing a more consistent and predictable experience for end users.</p>

<h2>
  
  
  WebFormsJS rewrite progress at a glance
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Aspect</th>
<th>Level of Progress</th>
<th>Explanation</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Performance</strong></td>
<td>⭐⭐⭐⭐☆</td>
<td>Network improvements, Retry, WebSocket</td>
</tr>
<tr>
<td><strong>Architecture</strong></td>
<td>⭐⭐⭐⭐⭐</td>
<td>Complete rewrite and new settings structure</td>
</tr>
<tr>
<td><strong>Maintainability</strong></td>
<td>⭐⭐⭐⭐⭐</td>
<td>Cleaner code and grouped settings</td>
</tr>
<tr>
<td><strong>User Experience (UX)</strong></td>
<td>⭐⭐⭐⭐☆</td>
<td>Clear messages, user-friendly errors</td>
</tr>
<tr>
<td><strong>Extensibility</strong></td>
<td>⭐⭐⭐⭐☆</td>
<td>Solid foundation for future plugins and updates</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  Conclusion
</h2>

<p>The WebForms Core 2 update with a fundamental rewrite of the WebFormsJS library is not a regular update, but a <strong>structural transformation</strong> that brings this framework to a new level of technical maturity. What we see in this version is a smart focus on the four fundamental foundations of modern web software development:</p>

<ol>
<li><p><strong>Stability and reliability</strong> - with a request retry mechanism, intelligent management of server responses and accurate error logging</p></li>
<li><p><strong>Developer experience</strong> - with a well-structured configuration structure, intelligent alerts and rich internal documentation</p></li>
<li><p><strong>User experience</strong> - with a transparent messaging system and standard behavior in form interactions</p></li>
<li><p><strong>Performance and standards</strong> - with a core rewrite, communication optimization and compliance with modern web standards</p></li>
</ol>

<p>This update shows that WebForms Core is no longer just a simple tool for managing forms, but a <strong>comprehensive platform for developing enterprise web applications</strong> that can work well in large and complex projects. Attention to technical details, along with consideration of end-user needs, makes this release a logical and forward-thinking choice for development teams.</p>

<p>By migrating to this release, developers will not only benefit from immediate stability and performance benefits, but will also have a solid foundation for future scalability and development of their product.</p>

