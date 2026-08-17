# Shoryan — Blood Donation Matching App

> Shoryan is a mobile-first platform that connects patients in need of blood with compatible, eligible, nearby donors — quickly, and without the guesswork.

---

## Overview

Shoryan lets users register as **donors**, **requesters (patients)**, or both. A requester can post an **"ask"** specifying blood type, hospital location, and urgency. Nearby, eligible donors see these asks in a location-based feed — much like a social media feed — and can accept or reject the request directly from the app.

Beyond simple matching, Shoryan applies data science to predict which donors are most likely to respond to a given ask, based on blood-type compatibility, location, and eligibility. The app also sends automated reminders to donors who have become eligible again (3+ months since their last donation), and includes an AI chatbot that answers common donation-related questions.

---

## Problem Statement

Finding a compatible blood donor during an emergency is often slow, manual, and inefficient:

- Hospitals and patients typically rely on word-of-mouth, social media posts, or manual phone calls to find donors — wasting critical time in emergencies.
- Existing donors aren't notified in a targeted way; requests are broadcast broadly instead of reaching donors who are actually eligible, nearby, and likely to respond.
- Willing, eligible donors often don't donate simply because they forget when they become eligible again, or never hear about nearby requests.
- There's no reliable way to prioritize urgent/critical cases over routine requests, so patients in genuine emergencies compete for attention with non-urgent asks.
- Donors have common, repetitive questions (eligibility, pre/post-donation care, compatibility) with no immediate, trustworthy source of answers.

**Shoryan closes these gaps** by combining location-based matching, predictive prioritization, and automated engagement (reminders + chatbot) into a single platform.

---

## Target Audience

| Segment                                                | Description                                                                                                                                             |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Blood Donors**                                       | Individuals who are willing and medically eligible to donate blood, and want to be notified of nearby, relevant requests instead of generic broadcasts. |
| **Requesters / Patients & their families**             | People who need to find a compatible blood donor urgently, often during a medical emergency.                                                            |
| **Hospitals & Blood Banks** _(indirect beneficiaries)_ | Benefit from faster donor turnaround and reduced reliance on manual, ad-hoc donor searches.                                                             |

---

## Tech Stack

| Layer              | Technology                   |
| ------------------ | ---------------------------- |
| Mobile App         | Flutter / Dart               |
| Backend / API      | Laravel (PHP)                |
| Database           | MySQL                        |
| API Authentication | Laravel Sanctum & Breeze     |
| Notifications      | Firebase (FCM)               |
| AI Chatbot         | RAG Knowledge-based          |
| Data Science       | Python, pandas, scikit-learn |
| Model Serving      | FastAPI                      |
| Deployment         | Vercel, Aiven                |

### AI & Data Science — Donor Response Prediction

A trained model predicts whether a donor is likely to be available for a given ask, based on blood-type compatibility, location,weight, hemoglobin, gender and age. Its output is a ranked list of best-matching donors for a request.

- **Dataset:** Blood Donor Dataset (Kaggle)

### AI Chatbot Knowledge Base

The chatbot is grounded in a fixed knowledge base covering:

- **Eligibility guidelines** — age 18–65, minimum weight 50kg, donation interval of every 3 months, general good health required
- **Before donation** — eat a healthy meal, stay hydrated, get enough sleep, bring a national ID
- **After donation** — drink extra fluids, avoid heavy exercise for the rest of the day, keep the bandage on for a few hours, eat iron-rich foods
- **Blood type compatibility**
- **Permanent restrictions** — HIV, Hepatitis B/C, blood cancers (leukemia, lymphoma, myeloma)
- **Temporary restrictions** — pregnancy/recent childbirth, anemia, active fever/infection, recent major surgery, recent transfusion, uncontrolled high blood pressure, uncontrolled heart disease, recent travel to malaria-risk areas, recent live vaccines, temporarily disqualifying medications, recent tattoos/piercings, recent exposure to hepatitis or other infectious disease

---

## User Flow

### 1. Onboarding & Authentication

```mermaid
flowchart TD
    Start([Start]) --> Onboarding[Onboarding]
    Onboarding -->|continue| GetStarted[Get Started]
    GetStarted -->|proceed| ExistingUser{Existing user?}

    ExistingUser -->|No| SignUp[Sign Up]
    ExistingUser -->|Yes| SignIn[Sign In]

    SignUp -->|verify| VerifyOTP[Verify Email OTP]
    VerifyOTP --> OTPValid{OTP valid?}
    OTPValid -->|No| ReEnterOTP[Re-enter OTP] -->|retry| VerifyOTP
    OTPValid -->|Yes| ProfileSetup[Profile Setup]
    ProfileSetup -->|finish| HomeScreen[Home Screen]

    SignIn -->|forgot?| ForgotPassword[Forgot Password]
    ForgotPassword -->|enter| EnterEmail[Enter Email]
    EnterEmail -->|send OTP| RecoveryOTP[Recovery OTP]
    RecoveryOTP -->|reset| ResetPassword[Reset Password]
    ResetPassword -->|back to sign in| SignIn

    SignIn --> HomeScreen
```

### 2. Home Navigation

```mermaid
flowchart TD
    HomeScreen[Home Screen] -->|home tab| HomeTab[Home Tab]
    HomeScreen -->|map| MapTab[Map Tab]
    HomeScreen -->|requests| RequestsTab[Requests Tab]
    HomeScreen -->|assistant| AIAssistantTab[AI Assistant Tab]
    HomeScreen -->|profile| ProfileTab[Profile Tab]

    MapTab -->|need location| LocationEnabled{Location enabled?}
    LocationEnabled -->|No| EnableLocation[Enable Location]
    LocationEnabled -->|Yes| NotificationsEnabled{Notifications enabled?}
    EnableLocation -->|enable| NotificationsEnabled
    NotificationsEnabled -->|No| EnableAlerts[Enable Alerts]
    NotificationsEnabled -->|Yes| HomeTab
    EnableAlerts -->|enable| HomeTab
```

### 3. Donor Flow — Browsing & Accepting Requests

```mermaid
flowchart TD
    HomeTab[Home Tab] -->|browse| RequestsTab[Requests Tab]
    RequestsTab -->|view| RequestDetails[Request Details]
    RequestDetails -->|accept| AcceptRequest[Accept Request]
    AcceptRequest -->|check eligibility| EligibleToDonate{Eligible to donate?}
    EligibleToDonate -->|No| RecoveryPeriod[Recovery Period] -->|countdown| HomeTab
    EligibleToDonate -->|Yes| DonationJourney[Donation Journey]
    DonationJourney -->|donate| CompleteDonation[Complete Donation]
    CompleteDonation -->|update| UpdateStats[Update Stats]
```

### 4. Emergency / Critical Requests Flow

```mermaid
flowchart TD
    HomeTab[Home Tab] -->|emergency| EmergencyBanner[Emergency Banner]
    EmergencyBanner -->|view nearby| CriticalRequests[Critical Requests]
    CriticalRequests -->|choose| EmergencyAction{Emergency action?}
    EmergencyAction -->|Accept| DonationJourney[Donation Journey]
    EmergencyAction -->|Call| CallHotline[Call Hotline]
    CallHotline -->|dial| PhoneCall((Phone Call))
    DonationJourney -->|donate| CompleteDonation[Complete Donation]
```

### 5. Requester Flow — Creating & Matching a Request

```mermaid
flowchart TD
    HomeTab[Home Tab] -->|create| CreateRequest[Create Request]
    CreateRequest -->|fill details| RequestDetailsForm[Request Details]
    RequestDetailsForm -->|submit| SubmitRequest[Submit Request]
    SubmitRequest -->|match| SmartMatching[Smart Matching]
    SmartMatching -->|show donors| MatchedDonors[Matched Donors]
    MatchedDonors -->|notify| NotifyTopDonors[Notify Top Donors]
    NotifyTopDonors -->|activate| RequestActive[Request Active]
    RequestActive -->|visible in list| RequestsTab[Requests Tab]
```

### 6. End-to-End Flow (High Level)

```mermaid
flowchart LR
    A[Onboarding & Auth] --> B[Home Screen]
    B --> C[Donor: Browse & Accept Requests]
    B --> D[Requester: Create Request]
    B --> E[Emergency: Critical Requests]
    D --> F[Smart Matching + Notify Donors]
    F --> C
    C --> G[Complete Donation]
    E --> G
    G --> H[Update Stats / Donation History]
```

---
