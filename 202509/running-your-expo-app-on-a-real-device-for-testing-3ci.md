---
Title: Running Your Expo App on a Real Device for Testing
Description: 
Author: Youngho Andrew Chaa
Date: 2025-09-06T22:02:37.000Z
Robots: noindex,nofollow
Template: index
---
<p>As an Expo developer, you’ve likely spent countless hours running your app on a simulator or emulator. While that's great for most development, nothing beats the feel of your app on a real device. It's the only way to truly test performance, native features, and user experience.</p>

<p>There are two primary paths for running your Expo app on a physical device: using the <strong>Expo Go</strong> app or building a <strong>custom development client</strong>. The right choice depends on your project's dependencies.</p>

<h3>
  
  
  Method 1: The Fast Path with Expo Go
</h3>

<p>If your project relies only on the standard APIs that come with the Expo SDK—meaning no custom native modules or libraries outside of what Expo Go supports—this is your go-to method. It’s quick and requires minimal setup.</p>

<h4>
  
  
  Prerequisites
</h4>

<ul>
<li>Your Device: An iPhone or Android phone.</li>
<li>Expo Go: Make sure the free <strong>Expo Go</strong> app is installed on your device from the App Store or Google Play Store.</li>
<li>Shared Network: This is the most critical step. Your computer and your mobile device must be connected to the exact same Wi-Fi network. Issues with corporate or university networks that have client isolation are the most common cause of connection failures.</li>
</ul>

<h4>
  
  
  Step-by-Step Guide
</h4>

<ol>
<li> Start Your Development Server: In your project's root directory, open your terminal and run the command <code>npx expo start</code>.</li>
<li> Scan the QR Code: The terminal will display a QR code. Use your device to scan it.

<ul>
<li>On iOS: Open the built-in Camera app and point it at the QR code. A notification will pop up. Tap it to open your app in Expo Go.</li>
<li>On Android: Open the Expo Go app and tap the "Scan QR Code" button to use the in-app scanner.</li>
</ul>
</li>
<li> Run the App: Expo Go will connect to your computer, download the JavaScript bundle, and run your app. Now, whenever you save changes in your code, they'll instantly update on your device.</li>
</ol>

<h4>
  
  
  Accessing the Developer Menu
</h4>

<p>Once the app is running, you can access the developer menu for debugging tools by shaking your device firmly. This menu allows you to open a remote debugger, reload the app, or view a performance monitor.</p>

<h3>
  
  
  Method 2: The Professional Path with a Development Build
</h3>

<p>Once you add custom native modules—such as a specific video player, a payment library, or Firebase—the Expo Go app won't work anymore. Its core is immutable and doesn't contain the native code your new library needs. The solution is to create a custom development build, which is a personalized version of Expo Go with your project's unique native code compiled in.</p>

<h4>
  
  
  Prerequisites
</h4>

<ul>
<li>Expo Account: You'll need a free Expo account.</li>
<li>EAS CLI: Install the Expo Application Services (EAS) command-line interface by running <code>npm install -g eas-cli</code> and then log in with <code>eas login</code>.</li>
</ul>

<h4>
  
  
  Step-by-Step Guide
</h4>

<ol>
<li> Add the Development Client Library: This library allows your custom-built app to communicate with your development server. Run <code>npx expo install expo-dev-client</code>.</li>
<li> Configure the Build Profile: Make sure your <code>eas.json</code> file is configured with a <code>development</code> build profile that has <code>"developmentClient": true</code>. If you don't have this file, you can generate it by running <code>eas build:configure</code>.</li>
<li> Create the Development Build: This is the process of building the native <code>.apk</code> (Android) or <code>.ipa</code> (iOS) file. It's done on Expo's cloud servers and can take 10-20 minutes.

<ul>
<li>For Android: <code>eas build --profile development --platform android</code>
</li>
<li>For iOS: <code>eas build --profile development --platform ios</code> (Note: This requires a paid Apple Developer account.)</li>
</ul>
</li>
<li> Install the Build: Once the build is complete, EAS will provide a link and a QR code. Scan the QR code to download and install the app on your device. On iOS, you'll likely use TestFlight for this.</li>
<li> Run Your App:

<ul>
<li>Start your server with <code>npx expo start --dev-client</code>.</li>
<li>Open the new custom app you just installed on your phone. It will automatically connect to your development server.</li>
<li>Just like with Expo Go, you can shake your device to access the developer menu and debugging tools.</li>
</ul>
</li>
</ol>

<h3>
  
  
  When to Rebuild Your App
</h3>

<p>With a development build, you don't need to rebuild every time you make a change. Your JavaScript code is loaded dynamically from the development server, providing a fast and iterative experience.</p>

<p>You must create a new build only when you change the native code. This includes:</p>

<ul>
<li>Adding or removing a native module npm package.</li>
<li>Upgrading or downgrading an existing native module.</li>
<li>Changing native configuration files like <code>app.json</code>, <code>Info.plist</code>, or <code>AndroidManifest.xml</code> (e.g., updating your app icon or package name).</li>
<li>Upgrading your project to a new Expo SDK version.</li>
</ul>

<p>For everything else—like editing your JavaScript/TypeScript files or changing CSS styles—the Metro server will instantly push the updates to your running app.</p>

<p>This two-path approach makes Expo incredibly flexible, allowing you to choose the right workflow for your project at every stage of development.</p>

