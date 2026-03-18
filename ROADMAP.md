Litforage Master Deployment Plan (React Native & Online-First Architecture)

This document outlines the systematic process for building and deploying the Litforage Android application and Chrome extension entirely within a cloud-based environment. This plan replaces the legacy Python/Colab architecture with a standard React Native (Expo) stack.

Phase 1: Cloud Environment Setup & Project Initialization

The objective of this phase is to establish a browser-based development environment (Cloud IDE) and initialize the React Native and Chrome Extension codebases from scratch.

GitHub Repository Initialization:

Create a new public repository on GitHub (e.g., litforage-monorepo).

Initialize the repository with an MIT License to establish open-source usage terms.

Cloud IDE Provisioning:

Open a cloud-based IDE natively supported in the browser (e.g., Google Project IDX, GitHub Codespaces, or Gitpod).

Clone the empty GitHub repository into the Cloud IDE workspace.

React Native (Expo) Initialization:

In the Cloud IDE terminal, initialize a new React Native app: npx create-expo-app@latest litforage-app --template default

Navigate into the directory: cd litforage-app

Extension Directory Setup:

In the root of the workspace (outside litforage-app), create the extension directory: mkdir litforage-extension

Initialize a standard manifest.json inside this directory.

Phase 2: Application Development & Live Testing (Expo Go)

This phase covers writing the React Native frontend and testing it in real-time on a physical mobile device.

AI-Led Code Generation:

Utilize Gemini Canvas to architect and generate React Native component code, pasting the resulting files into the Cloud IDE litforage-app directory.

Local Testing via Expo Go:

Install the "Expo Go" application on your physical Android device from the Google Play Store.

In the Cloud IDE terminal, start the development server: npx expo start

Scan the QR code displayed in the terminal/browser using your physical Android device. Code changes saved in the Cloud IDE will hot-reload instantly on the phone.

Version Control:

Commit verified working code to GitHub directly from the Cloud IDE terminal: git add ., git commit -m "update", git push.

Phase 3: Backend Services Integration (Firebase)

Firebase serves strictly as the Backend-as-a-Service (BaaS) for database, storage, and authentication.

Project Creation:

Navigate to the Firebase Console and create a new project (litforage-backend).

App Registration:

Register an Android App within the Firebase project using your designated package name (e.g., com.litforage.app).

Register a Web App if the Chrome extension requires database access.

SDK Integration:

In the Cloud IDE (litforage-app directory), install the Firebase JS SDK: npx expo install firebase

Create a firebaseConfig.js file using the configuration block provided by the Firebase Console to enable direct communication between the React Native frontend and the Firebase backend.

Phase 4: Chrome Extension Development & Testing

Develop the browser extension parallel to the mobile application.

Manifest Configuration: Ensure manifest.json inside litforage-extension/ complies with Manifest V3 specifications.

Local Unpacked Testing:

Open Chrome on the Chromebook and navigate to chrome://extensions/.

Enable "Developer mode".

Click "Load unpacked" and point it to the Cloud IDE's exposed file system or download the litforage-extension/ directory locally to test.

Functionality Verification: Test background scripts, content scripts, and cross-origin resource sharing (CORS) configurations if connecting to the Firebase backend.

Phase 5: Mandatory Security Audits

Rigorous security auditing must be performed on both codebases prior to cloud compilation and publication.

5.1 React Native & Firebase Security Checks

Secure Storage: Ensure sensitive data (like authentication tokens) is not stored in plain text. Use expo-secure-store instead of standard AsyncStorage.

Environment Variables: Verify that no private API keys or secrets are hardcoded in the repository. Use .env files and Expo's environment variable configuration.

Firebase Security Rules: Write and deploy strict Firestore and Storage rules ensuring authenticated users can only read/write authorized data.

Dependency Audit: Run npm audit in the litforage-app directory to identify and patch vulnerabilities in React Native packages.

5.2 Chrome Extension Security Checks

Permissions Audit: Review the permissions and host_permissions arrays in manifest.json. Apply the Principle of Least Privilege.

CSP & Remote Code: Verify strict Content Security Policies. Ensure no remote code evaluation occurs, adhering to Manifest V3 restrictions.

Message Passing Audits: Inspect chrome.runtime.sendMessage and chrome.runtime.onMessage listeners to prevent privilege escalation from malicious external webpages.

Phase 6: CI/CD Cloud Compilation (GitHub Actions)

Because the repository is public, GitHub Actions provides unlimited free compute minutes for standard Linux runners. This phase utilizes a GitHub-hosted CI/CD pipeline to compile the Android binary (.aab or .apk) via Expo's local build command, bypassing EAS Free Tier limitations entirely.

Keystore Generation:

In the Cloud IDE terminal, generate a local Android Keystore required for app signing: keytool -genkey -v -keystore litforage.keystore -alias litforage -keyalg RSA -keysize 2048 -validity 10000

Crucial: Do not commit litforage.keystore to the public repository. Add it to .gitignore.

GitHub Secrets Configuration:

Base64 encode the keystore file in the terminal: base64 litforage.keystore > keystore.b64

In the GitHub repository settings, navigate to "Secrets and variables" -> "Actions".

Store the Base64 string, Keystore password, Key alias, and Key password as repository secrets.

Workflow File Creation:

In the repository root, create the directory structure: .github/workflows/

Create an android-build.yml file. Configure the YAML to provision an Ubuntu runner, install Node.js and Java (JDK 17), decode the keystore secret back into a file, and execute npx eas-cli build --platform android --local.

Trigger Build & Download Binary:

Commit and push the code (including the .github folder) to the main branch.

Navigate to the "Actions" tab on GitHub to monitor the pipeline execution.

Once complete, download the resulting .aab or .apk artifact directly from the GitHub Actions run summary.

Phase 7: Developer Accounts & Publishing

Execute the final deployment procedures across both Google platforms.

7.1 Android App Publishing (Google Play)

Google Play Console: Register a Google Play Developer account and pay the $25 one-time fee.

App Creation: Create a new app listing, fill out the store presence, content ratings, and privacy policy URL.

Upload: Upload the .aab file generated by GitHub Actions to the Production or Testing track.

Review: Submit the app for Google Play review.

7.2 Chrome Extension Publishing (Chrome Web Store)

Packaging: Compress the contents of the litforage-extension/ directory into a single .zip file.

Web Store Dashboard: Navigate to the Chrome Web Store Developer Dashboard and pay the $5 one-time registration fee if not already completed.

Upload & Listing: Upload the .zip file, fill out the store listing details, promotional images, and privacy data declarations.

Review: Submit the extension for Chrome Web Store review.