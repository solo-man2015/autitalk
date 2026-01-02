

### **Autitalk: Application Architecture Document v1.1 (Approved)**

#### **1. Executive Summary**

Autitalk is a mobile/tablet-based assistive communication and learning application designed for children with autism spectrum disorder (ASD). The app's core mission is to bridge the communication gap by interpreting a child's spoken attempts, offering visual choices to confirm intent, and articulating a well-formed sentence. This process not only facilitates immediate communication but also serves as a therapeutic tool to help children learn sentence structure and express their needs more clearly.

This document details a scalable, cross-platform, AI-driven architecture to support Autitalk's vision.

#### **2. Architectural Goals & Principles**

*   **User-Centric & Accessible:** The UI/UX must be extremely simple, predictable, and calming. It should avoid overwhelming stimuli and be highly customizable to cater to individual sensory sensitivities and preferences.
*   **AI-Powered Core:** The application's success hinges on its ability to accurately understand varied and non-typical speech patterns. Our AI models must be robust, adaptable, and continuously improvable.
*   **Safety & Privacy First:** As the application will handle children's data, including voice recordings, we must adhere to the strictest privacy standards (e.g., COPPA, HIPAA considerations). All data must be encrypted in transit and at rest.
*   **Cross-Platform:** The application must provide a consistent experience on both iOS and Android devices (phones and tablets).
*   **Scalable & Maintainable:** The backend will be built on a microservices architecture to ensure that individual components can be scaled and updated independently.
*   **Personalization:** The app must be deeply personalizable. A guardian should be able to add custom items (e.g., favorite foods, toys, family members' names) with corresponding images and vocabulary.

#### **3. Overall Architecture Diagram**

The system is designed with a client-server model. The mobile app serves as the client, and a set of cloud-based microservices provides the backend processing power, especially for the AI components.

#### **4. Software Modules & Components**

##### **4.1. Mobile Application (Client-Side)**

The mobile application is the primary interface for the child and their guardian.

*   **UI/UX Layer:**
    *   **Description:** Manages all visual elements, animations, and user interactions. Built with large, clear, high-contrast visual components. Includes a "Guardian Mode" for configuration and a "Child Mode" for daily use.
    *   **Components:** Home Screen, Listening/Interaction Screen, Visual Choice Display, Sentence Display, Guardian Dashboard.
*   **Audio Processing Module:**
    *   **Description:** Responsible for capturing high-quality audio from the device's microphone, performing basic noise reduction, and compressing it for efficient transmission to the backend.
*   **State Management Module:**
    *   **Description:** Manages the application's state, ensuring the UI is always in sync with the underlying data (e.g., user profile, interaction state).
*   **API Client Module:**
    *   **Description:** Handles all secure communication (RESTful APIs over HTTPS) with the backend services. Manages request/response cycles, authentication tokens, and error handling.
*   **Local Storage (Persistence) Module:**
    *   **Description:** Uses an embedded database (like SQLite) to store the user profile, custom vocabulary, cached images, and common phrases for offline access and faster performance.
*   **Text-to-Speech (TTS) Module:**
    *   **Description:** Utilizes the native device's TTS engine for speaking sentences. This allows for offline functionality and reduces latency. Can be augmented with higher-quality cloud-based voices when online.

##### **4.2. Backend Services (Cloud-Side)**

The backend is designed as a set of containerized microservices managed by an orchestrator.

*   **API Gateway:**
    *   **Description:** The single entry point for all client requests. It handles routing, authentication, rate limiting, and request logging.
*   **Authentication Service:**
    *   **Description:** Manages guardian user accounts, including registration, login, and password management using secure protocols like OAuth 2.0.
*   **User Profile Service:**
    *   **Description:** Stores and manages all user-specific data, including the child's profile, guardian information, and personalized vocabulary/image library (e.g., "Mommy," "Daddy," favorite cereal brand).
*   **AI Core Service:**
    *   **Description:** This is the brain of the application. It's a pipeline of machine learning models.
        1.  **Speech-to-Text (STT):** Transcribes the child's audio into text. This model will be fine-tuned on datasets of atypical speech for higher accuracy.
        2.  **Natural Language Understanding (NLU):** Analyzes the transcribed text to extract the core *intent* (e.g., `request_food`) and *entities* (e.g., `cereal`, `juice`).
        3.  **Intent Disambiguation & Recommendation:** Based on the extracted intent/entities, this component queries the User Profile Service for context (e.g., child's favorite foods) and generates a set of probable visual options.
        4.  **Sentence Generation:** Constructs grammatically correct sentences based on the user's final visual selection (e.g., `[Parent_Name]`, `I want to have some [Food_Item] now`).
*   **Cloud Text-to-Speech (TTS) Service:**
    *   **Description:** An optional, higher-quality voice generation service for generating audio clips for new or custom sentences.

#### **5. Technology Stack & Tools**

| Category                  | Technology / Tool                                                              | Justification                                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| **Mobile Development**    | **Flutter & Dart**                                                             | High-performance, cross-platform framework for a consistent UI/UX on both iOS and Android from a single codebase. |
| **State Management**        | **BLoC / Provider**                                                            | Robust and scalable state management patterns for Flutter.                                                    |
| **Backend Framework**     | **Python (FastAPI)**                                                           | Python is the industry standard for AI/ML. FastAPI is modern, fast, and easy to use for building APIs.        |
| **AI / Machine Learning** | **PyTorch / TensorFlow**                                                       | For building and training custom NLU models.                                                                  |
|                           | **Cloud AI Services (AWS Transcribe, AWS Polly)**                              | To bootstrap STT and TTS capabilities. We will fine-tune these models with our own data for better accuracy. |
| **Database**              | **Amazon DynamoDB or Firestore (NoSQL)**                                       | Flexible schema is ideal for storing user profiles and custom data. Scalable and fully managed.               |
|                           | **SQLite (on-device)**                                                         | For local caching and offline functionality on the mobile app.                                                |
| **Cloud Platform**        | **Amazon Web Services (AWS)**                                                  | Mature ecosystem with excellent AI/ML services (SageMaker, Transcribe), container support (EKS), and database options. |
| **Containerization**      | **Docker & Kubernetes (Amazon EKS)**                                           | For packaging, deploying, and managing our backend microservices reliably and at scale.                       |
| **Source Control**        | **Git (GitHub / GitLab)**                                                      | Industry standard for version control and collaboration.                                                      |
| **CI/CD**                 | **GitHub Actions**                                                             | Tightly integrated with GitHub for automating build, test, and deployment pipelines for both mobile and backend. |

#### **6. Core User Workflow**

1.  **Initiation:** The child taps a large, friendly "Talk to me" button on the app's main screen.
2.  **Audio Capture:** The app starts recording audio. The UI provides simple visual feedback (e.g., a pulsing circle) to indicate it's listening.
3.  **API Request:** The captured audio file is sent securely to the backend API Gateway.
4.  **AI Processing:**
    *   The request is routed to the **AI Core Service**.
    *   **STT** converts the audio to text (e.g., "wan cewal").
    *   **NLU** identifies `intent: request_food` and `entity: cereal`.
    *   The service fetches the user's profile to find preferred cereals (e.g., "Cheerios," "Fruit Loops") and associated images.
5.  **Visual Options:** The backend returns a list of ranked options (e.g., `{item: "Cheerios", image_url: "...", sentence: "I want to have some Cheerios now"}`).
6.  **User Selection:** The mobile app displays these options as large, clear picture cards. The child taps the image of what they want (e.g., the Cheerios box).
7.  **Sentence Articulation:** Upon selection, the app displays the full sentence: "Mommy, I want to have some Cheerios now." It then presents two options:
    *   **"Say it with me":** Prompts the child to repeat the sentence, potentially using the device's microphone to provide positive feedback.
    *   **"Say it for me":** The app uses its TTS module to speak the sentence aloud in a clear, friendly voice.

#### **7. Test Execution Strategy**

*   **Unit Testing:** Each function/widget in the Flutter app and each endpoint/function in the backend services will have dedicated unit tests (using `pytest` for Python, `flutter_test` for Dart).
*   **Integration Testing:** We will test the interaction between microservices (e.g., can the AI Core service correctly fetch data from the Profile service?).
*   **End-to-End (E2E) Testing:** Using a framework like Appium or Flutter's `integration_test` package, we will automate full user workflows from tapping the listen button to the final sentence output.
*   **AI Model Validation:** A "golden dataset" of audio clips with known intents will be used to continuously measure the accuracy, precision, and recall of our STT and NLU models.
*   **Usability & User Acceptance Testing (UAT):** This is the most critical phase. We will work closely with child development specialists, therapists, and families with autistic children to test the app in real-world scenarios and gather qualitative feedback.

#### **8. Deployment Mechanism (CI/CD)**

We will use GitHub Actions to create separate, automated pipelines for the mobile app and the backend.

**Backend Deployment:**
1.  **On push to `main` branch:**
2.  Run unit tests and linting.
3.  Build Docker images for each microservice.
4.  Push images to a container registry (Amazon ECR).
5.  Deploy the new images to the Kubernetes cluster (EKS) using Helm charts for blue/green or canary deployments to ensure zero downtime.

**Mobile App Deployment:**
1.  **On push to `release` branch:**
2.  **iOS:**
    *   Job runs on a macOS runner.
    *   Build the Flutter app for iOS (`.ipa` file).
    *   Code-sign the application using secure credentials.
    *   Upload the build to **TestFlight** for internal testing and UAT.
    *   On approval, promote the build for App Store review and release.
3.  **Android:**
    *   Job runs on a Linux runner.
    *   Build the Flutter app for Android (`.aab` file).
    *   Sign the app bundle using secure credentials.
    *   Upload the build to the **Google Play Console** internal testing track.
    *   On approval, promote the build to production.

#### **9. Running on Simulators for Testing**

**iOS (on macOS):**
1.  **Prerequisites:** A Mac computer with Xcode and CocoaPods installed.
2.  **Setup:**
    *   Clone the Git repository.
    *   Navigate to the `ios` directory within the Flutter project (`cd ios`).
    *   Run `pod install` to install iOS dependencies.
    *   Open the `.xcworkspace` file in Xcode.
3.  **Execution:**
    *   In your IDE (VS Code or Android Studio), select an iOS Simulator from the device dropdown (e.g., "iPhone 14 Pro").
    *   Run the Flutter application (`flutter run` or click the "Run" button). The app will be built and launched on the selected simulator.

**Android (on macOS, Windows, or Linux):**
1.  **Prerequisites:** Android Studio installed with the Android SDK and emulator set up.
2.  **Setup:**
    *   Open Android Studio.
    *   Go to the Device Manager (formerly AVD Manager) and create a new Android Virtual Device (AVD) if one doesn't exist.
3.  **Execution:**
    *   In your IDE, select the desired Android Emulator from the device dropdown.
    *   Start the emulator.
    *   Run the Flutter application (`flutter run` or click "Run"). The app will be installed and launched on the running emulator.

#### **10. Security, Privacy, and Compliance**

Given the sensitive nature of the data being handled (children's voice and personal information), security and privacy are paramount.

*   **Data Encryption:** All communication between the mobile client and backend services will be encrypted using TLS 1.2+. All user data at rest in the database and object storage will be encrypted using industry-standard AES-256 encryption.
*   **Parental Consent:** The application will feature a clear, easy-to-understand consent form during onboarding. Guardians must provide explicit consent for data collection, especially for voice data that may be used for model training. Users will have the right to access and delete their data.
*   **Data Anonymization:** Any voice data used for training our custom AI models will be fully anonymized and disassociated from any personally identifiable information (PII).
*   **Compliance:** The system will be architected with **COPPA (Children's Online Privacy Protection Act)** and **HIPAA (Health Insurance Portability and Accountability Act)** guidelines in mind. This includes secure data handling, access controls, and audit trails. We will engage with legal counsel specializing in this area to ensure full compliance.
*   **Authentication & Authorization:** The Guardian account will be protected by robust authentication. Access to the backend services will be strictly controlled via role-based access control (RBAC) and short-lived authentication tokens.

#### **11. Phased Rollout & MVP Strategy**

To mitigate risk and accelerate time-to-market, we will adopt a phased development and release strategy.

*   **Phase 1: Minimum Viable Product (MVP)**
    *   **Goal:** Validate the core user workflow and gather initial user feedback.
    *   **Features:**
        *   Core workflow with a limited, predefined set of intents (e.g., want food, want drink, want toy, need help).
        *   Utilize standard AWS Transcribe for STT and AWS Polly for TTS.
        *   Sentence generation will be based on simple, hardcoded templates.
        *   No guardian-led personalization.
        *   Online-only functionality.
*   **Phase 2: Personalization & Data Foundation**
    *   **Goal:** Enhance user engagement through personalization and begin building our unique dataset.
    *   **Features:**
        *   Introduce the Guardian Dashboard for adding custom nouns (people, foods, places) with uploaded images.
        *   Implement the explicit consent flow for collecting anonymized voice data for model improvement.
        *   Introduce basic offline capability by caching the user's personalized library on the device.
*   **Phase 3: Advanced AI & Offline Mode**
    *   **Goal:** Dramatically improve recognition accuracy for our target users and provide robust offline support.
    *   **Features:**
        *   Deploy our own fine-tuned STT and NLU models trained on the data collected in Phase 2.
        *   Integrate on-device, lightweight STT/TTS models to provide a functional offline mode.
        *   Explore more dynamic, context-aware sentence generation beyond simple templates.
