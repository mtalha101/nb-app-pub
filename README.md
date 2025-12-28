<p align="center">
  <img src="assets/app_icon.jpg" alt="Najeeb Logo" width="120" height="120" style="border-radius: 20px;"/>
</p>

<h1 align="center">Najeeb Books</h1>

<p align="center">
  <strong>Premium Arabic Audiobook & E-Reader Platform</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Flutter-3.7.0+-02569B?logo=flutter" alt="Flutter Version"/>
  <img src="https://img.shields.io/badge/Platform-iOS%20%7C%20Android-lightgrey" alt="Platform"/>
  <img src="https://img.shields.io/badge/Language-Arabic-green" alt="Language"/>
  <img src="https://img.shields.io/badge/RTL-Supported-blue" alt="RTL Support"/>
</p>

---

## 📖 Overview

**Najeeb Books** is a premium Arabic audiobook and e-reader mobile application designed to make knowledge accessible through immersive audio-visual reading experiences. The app combines synchronized audio playback with beautifully formatted text, allowing users to read and listen simultaneously.

### 🎯 The Problem

Arabic readers face challenges finding quality audiobook platforms that:

- Provide synchronized text highlighting with audio
- Support proper RTL (Right-to-Left) text rendering
- Offer a curated library of valuable content
- Track reading progress across devices
- Provide a premium, distraction-free reading experience

### 💡 The Solution

Najeeb Books delivers:

- **Synchronized Reading**: Audio playback with real-time text highlighting
- **Native RTL Support**: Proper Arabic text rendering throughout the app
- **Smart Progress Tracking**: Resume where you left off across sessions
- **Premium Subscriptions**: Access to exclusive content via RevenueCat
- **Beautiful UI**: Clean, modern interface optimized for reading

---

## 👨‍💻 My Role & Contributions

As the **Lead Flutter Developer**, I was responsible for:

### Core Development

- 🏗️ **Architected** the entire Flutter application using feature-based modular architecture
- 📱 **Built** 10+ feature modules from scratch (Reading, Audio, Library, Subscriptions, etc.)
- 🎧 **Implemented** advanced audio playback with background support and system controls
- 📖 **Developed** HTML content parser for rich text rendering with RTL support

### Technical Challenges Solved

- ⚡ **Audio Sync**: Built synchronized text highlighting with audio timestamps
- 🔤 **Arabic Support**: Implemented comprehensive RTL and Arabic text normalization
- 📊 **Progress Tracking**: Designed offline-first progress saving with cloud sync
- 🎨 **Theming**: Created adaptive dark/light themes with custom typography

### Backend & Infrastructure

- ☁️ **Firebase**: Designed Firestore data models for books, chapters, and user progress
- 🔐 **Authentication**: Integrated Email, Google, and Apple Sign-In
- 📊 **Analytics**: Set up comprehensive tracking with Firebase Analytics + Adjust SDK
- 💳 **Monetization**: Integrated RevenueCat for subscription management

---

## 📊 Impact & Results

<p align="center">
  <table>
    <tr>
      <td align="center">
        <h3>10+</h3>
        <p>Feature Modules</p>
      </td>
      <td align="center">
        <h3>Full RTL</h3>
        <p>Arabic Support</p>
      </td>
      <td align="center">
        <h3>Offline</h3>
        <p>Progress Sync</p>
      </td>
      <td align="center">
        <h3>Premium</h3>
        <p>Subscriptions</p>
      </td>
    </tr>
  </table>
</p>

### Key Achievements

- 📈 **Seamless** audio-text synchronization for immersive reading
- ⚡ **Instant** progress restoration across app sessions
- 🌙 **Beautiful** dark/light theme implementation
- 🔄 **Reliable** background audio playback with system controls

---

## ✨ Key Features

### 📱 For Readers

| Feature                  | Description                                         |
| ------------------------ | --------------------------------------------------- |
| **Audiobook Playback**   | High-quality audio with background playback support |
| **Synchronized Reading** | Text highlighting synced with audio timestamps      |
| **Smart Progress**       | Automatic bookmark saving and restoration           |
| **Reading History**      | Track all books you've started or completed         |
| **Saved Library**        | Bookmark favorite books for quick access            |
| **Category Browse**      | Explore books by categories                         |
| **Search**               | Find books quickly with fuzzy search                |

### 🎨 User Experience

| Feature                | Description                                 |
| ---------------------- | ------------------------------------------- |
| **RTL Support**        | Full Arabic Right-to-Left text rendering    |
| **Dark/Light Themes**  | Adaptive theming for comfortable reading    |
| **Mini Player**        | Compact audio controls while browsing       |
| **Full Player**        | Immersive playback experience with controls |
| **Push Notifications** | Stay updated on new releases                |

### 💎 Premium Features

| Feature                | Description                   |
| ---------------------- | ----------------------------- |
| **Subscription Plans** | Multiple tiers via RevenueCat |
| **Exclusive Content**  | Premium books for subscribers |
| **Offline Access**     | Download for offline reading  |

---

## 🏗️ Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                      NAJEEB BOOKS ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐     ┌──────────────┐                              │
│  │   Flutter    │     │   Flutter    │                              │
│  │     iOS      │     │   Android    │                              │
│  └──────┬───────┘     └──────┬───────┘                              │
│         │                    │                                      │
│         └─────────┬──────────┘                                      │
│                   │                                                 │
│         ┌─────────▼─────────┐                                       │
│         │   GetX State      │                                       │
│         │   Management      │                                       │
│         └─────────┬─────────┘                                       │
│                   │                                                 │
│    ┌──────────────┼──────────────┐                                  │
│    │              │              │                                  │
│  ┌─▼──────┐  ┌────▼─────┐    ┌───▼──────┐                           │
│  │Firebase│  │RevenueCat│    │Just Audio│                           │
│  │Backend │  │  Subs    │    │ Player   │                           │
│  └─┬──────┘  └──────────┘    └──────────┘                           │
│    │                                                                │
│  ┌─┴────────────────────────────────────────┐                       │
│  │ • Firestore (Books, Chapters, Progress)  │                       │
│  │ • Firebase Auth (Email, Google, Apple)   │                       │
│  │ • Firebase Storage (Audio, Covers)       │                       │
│  │ • Firebase Messaging (Notifications)     │                       │
│  │ • Firebase Analytics (User Tracking)     │                       │
│  └──────────────────────────────────────────┘                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Project Structure

```
lib/
├── core/                    # Core utilities and shared functionality
│   ├── constants/          # App constants, icons, images
│   ├── localization/       # Arabic/English translations
│   ├── navigation/         # Navigation helpers
│   ├── routes/             # Route definitions
│   └── theme/              # Colors, text styles, theming
│
├── features/               # Feature modules
│   ├── auth/              # 🔐 Authentication (Email, Google, Apple)
│   ├── book_details/      # 📖 Book detail view
│   ├── explore/           # 🔍 Browse & search books
│   ├── home/              # 🏠 Home screen & navigation
│   ├── my_library/        # 📚 Saved books & history
│   ├── notifications/     # 🔔 Push notifications
│   ├── onboarding/        # 👋 Welcome & onboarding
│   ├── reading/           # 📖 Reading screen with audio sync
│   ├── settings/          # ⚙️ User settings & profile
│   └── subscription/      # 💎 Premium subscriptions
│
├── shared/                 # Shared components & services
│   ├── controllers/       # Audio player, theme controllers
│   ├── services/          # Analytics, storage, progress tracking
│   ├── utils/             # HTML parser, helpers
│   └── widgets/           # Reusable UI components
│
└── main.dart              # App entry point
```

### Feature Module Pattern

Each feature follows a consistent, scalable structure:

```
feature/
├── bindings/       # GetX dependency injection
├── controllers/    # Business logic (GetX controllers)
├── models/         # Data models & entities
├── screens/        # UI screens
├── services/       # API/Firestore services
└── widgets/        # Feature-specific widgets
```

---

## 🧩 Technical Challenges & Solutions

### 1. Audio-Text Synchronization

**Challenge**: Synchronize text highlighting with audio playback in real-time.

**Solution**:

- Implemented highlight data model with `startTime`, `endTime`, and `text`
- Built word-level parsing from HTML content
- Created Arabic text normalization to handle diacritics
- Real-time position tracking with debounced UI updates

**Result**: Smooth, accurate text highlighting synchronized with audio.

### 2. Arabic Text & RTL Support

**Challenge**: Proper rendering of Arabic text with diacritics (tashkeel) and RTL layout.

**Solution**:

- Custom Arabic text normalization function
- Comprehensive RTL support throughout UI
- HTML content parser preserving Arabic formatting
- Custom font integration (Hurme Geometric Sans)

**Result**: Native-quality Arabic reading experience.

### 3. Offline-First Progress Tracking

**Challenge**: Track reading progress reliably across sessions, online and offline.

**Solution**:

- Local storage with SharedPreferences for immediate saves
- Background sync to Firestore when online
- Conflict resolution for multi-device usage
- Automatic progress restoration on app launch

**Result**: Users never lose their reading position.

### 4. Background Audio Playback

**Challenge**: Maintain audio playback when app is backgrounded with system controls.

**Solution**:

- Integrated `just_audio_background` for system media controls
- Custom audio handler for notification controls
- State preservation across app lifecycle
- Mini player for in-app navigation

**Result**: Seamless audio experience matching native apps.

---

## 🛠️ Tech Stack

### Frontend

| Technology           | Purpose                       |
| -------------------- | ----------------------------- |
| Flutter 3.7.0+       | Cross-platform UI framework   |
| GetX                 | State management, DI, routing |
| Google Fonts         | Typography                    |
| Cached Network Image | Image caching                 |

### Backend & Database

| Technology         | Purpose                       |
| ------------------ | ----------------------------- |
| Firebase Firestore | Real-time NoSQL database      |
| Firebase Auth      | Authentication                |
| Firebase Storage   | Media storage (audio, covers) |
| Firebase Messaging | Push notifications            |

### Audio & Media

| Technology               | Purpose                        |
| ------------------------ | ------------------------------ |
| Just Audio               | Audio playback engine          |
| Just Audio Background    | Background playback & controls |
| Audio Video Progress Bar | Playback progress UI           |

### Payments & Analytics

| Technology         | Purpose                 |
| ------------------ | ----------------------- |
| RevenueCat         | Subscription management |
| Firebase Analytics | User behavior tracking  |
| Adjust SDK         | Attribution analytics   |

### Localization

| Technology            | Purpose                |
| --------------------- | ---------------------- |
| GetX Translations     | Multi-language support |
| Flutter Localizations | Date/time formatting   |

---

## 🚀 Getting Started

### Prerequisites

- Flutter SDK 3.7.0+
- Xcode 15+ (iOS)
- Android Studio (Android)
- Firebase CLI
- CocoaPods (iOS)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-repo/najeeb-books.git

# 2. Install dependencies
flutter pub get

# 3. iOS setup
cd ios && pod install && cd ..

# 4. Run the app
flutter run
```

### Build Commands

```bash
# Development
flutter run

# Production (Android)
flutter build appbundle --release

# Production (iOS)
flutter build ipa --release
```

---

## 🔐 Security & Best Practices

- ✅ **Firebase Security Rules**: Granular access control on Firestore
- ✅ **Secure Authentication**: Multi-provider auth with Firebase
- ✅ **Input Validation**: Comprehensive form validation
- ✅ **Secure Payments**: RevenueCat handles subscription security
- ✅ **ProGuard**: Android code obfuscation enabled

---

## 🌍 Localization

- **Arabic** (ar) - Primary language with full RTL support
- **English** (en) - Secondary language support

---

## 🔮 Future Roadmap

- [ ] Offline audio downloads
- [ ] Reading statistics dashboard
- [ ] Social sharing features
- [ ] Bookmarks within chapters
- [ ] Audio playback speed control
- [ ] Sleep timer functionality

---

## 📬 Contact

**Muhammad Talha**

- 📧 Email: m.talhaarshad98@gmail.com
- 💼 LinkedIn: [linkedin.com/in/tvlhv](https://linkedin.com/in/tvlhv)
- 🐙 GitHub: [github.com/mtalha101](https://github.com/mtalha101)

---

<p align="center">
  <sub>Built with ❤️ using Flutter</sub>
</p>
