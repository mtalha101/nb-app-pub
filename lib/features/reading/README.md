# Reading Screen

A beautiful chapter-by-chapter reading interface that matches the exact design from screenshots.

## Features

### ✨ Core Features:

- 📖 **Chapter-by-chapter reading** with smooth page transitions
- 📚 **Book header** with cover, title, author, and duration
- 📊 **Progress indicator** (e.g., "3/8") showing current chapter
- ➡️ **Next navigation** with circular green button
- ✅ **Mark as finished** button on the final chapter
- 💾 **Auto-save progress** when navigating back
- 🎨 **Dark teal design** matching the app theme
- 🚫 **No confirmation dialogs** - seamless back navigation

### 🎯 UI Elements:

- **Top Header**: Book cover (60x80), author name, book title, duration & chapters
- **Content Area**: Scrollable chapter content with proper typography
- **Bottom Navigation**: Chapter counter (left) and action button (right)
- **No back arrow** - Uses system back navigation
- **Done button** appears on last chapter

### 📱 User Experience:

- Swipe left/right to navigate between chapters
- Smooth page transitions
- Responsive text with excellent readability (17px, 1.8 line height)
- Proper spacing and generous padding (32px horizontal)
- Auto-save on back without interruptions

## Usage

### From Book Details Screen:

```dart
// The "Read" button in BookDetailsScreen automatically navigates to ReadingScreen
_buildReadButton() // This button handles navigation with all required params
```

### Direct Navigation:

```dart
import 'package:najeeb_app/features/reading/screens/reading_screen.dart';

Get.to(
  () => ReadingScreen(
    bookId: 'book123',
    bookTitle: 'Me, My Customer, and AI',
    bookAuthor: 'Henrik Werdelin, Nicholas Thorne',
    bookCover: 'https://example.com/cover.jpg',
    duration: '18 min',
    chapters: [
      Chapter(
        title: 'What\'s in it for me? Harness AI to launch...',
        content: 'Introduction text...',
        order: 0,
      ),
      Chapter(
        title: 'Chapter 1: Getting Started',
        content: 'The journey begins...',
        order: 1,
      ),
      // ... more chapters
    ],
    initialChapter: 0, // Optional: start from specific chapter
  ),
);
```

## Chapter Data Model

```dart
class Chapter {
  final String title;      // Chapter title
  final String content;    // Full chapter text
  final int order;         // Chapter number (0-based)

  Chapter({
    required this.title,
    required this.content,
    required this.order,
  });
}
```

## Design Specifications

### Colors:

- **Background**: `AppColors.darkTeal` (`#042330`) - Matches app theme
- **Text**: `AppColors.white` - White
- **Accent**: `AppColors.primaryColor` - Green
- **Button**: Green with dark teal text

### Typography:

- **Author**: 13px, Medium, White 70%
- **Book Title**: 20px, Bold, White
- **Duration**: 13px, Regular, White 60%
- **Chapter Title**: 28px, Bold, White
- **Content**: 17px, Regular, White, Line height 1.8, Letter spacing 0.2

### Layout:

- **Header Padding**: 24px horizontal, 20px vertical
- **Content Padding**: 32px horizontal, 20px top, 100px bottom
- **Bottom Navigation**: 32px horizontal, 20px top, 28px bottom
- **Book Cover**: 60x80px with 8px border radius

## Features Implementation Status

### ✅ Implemented:

- [x] Chapter-by-chapter navigation
- [x] Smooth page transitions
- [x] Progress indicator
- [x] Next chapter navigation (circular button)
- [x] Done button on last chapter
- [x] Mark as finished button
- [x] Book header with cover and details
- [x] Auto-save progress on back
- [x] No confirmation dialogs
- [x] Beautiful UI matching screenshots
- [x] System back navigation

### 🔄 TODO (Production):

- [ ] Connect to Firestore for real chapter data
- [ ] Implement actual progress saving to database
- [ ] Add text size adjustment functionality
- [ ] Add bookmark functionality
- [ ] Implement share feature
- [ ] Add reading statistics
- [ ] Sync progress across devices
- [ ] Add highlighting and notes
- [ ] Add audio narration sync (for highlighting)

## Integration with Firestore

### Expected Firestore Structure:

```
books/{bookId}
├── (metadata: title, author, cover, duration, etc.)
└── chapters (subcollection)
    ├── 0_introduction
    │   ├── title: "Introduction"
    │   ├── content: "..."
    │   ├── order: 0
    │   └── duration: "5 min"
    ├── 1_chapter_one
    └── ...

users/{userId}
└── reading_progress (subcollection)
    └── {bookId}
        ├── currentChapter: 3
        ├── lastRead: Timestamp
        ├── completed: false
        └── finishedChapters: [0, 1, 2]
```

### Loading Real Data:

```dart
// Fetch chapters from Firestore
Future<List<Chapter>> _loadChapters(String bookId) async {
  final snapshot = await FirebaseFirestore.instance
    .collection('books')
    .doc(bookId)
    .collection('chapters')
    .orderBy('order')
    .get();

  return snapshot.docs.map((doc) {
    return Chapter(
      title: doc['title'],
      content: doc['content'],
      order: doc['order'],
    );
  }).toList();
}
```

## Screenshots Reference

The design matches the provided screenshots exactly:

### Top Section:

- ✅ Book cover thumbnail (60x80) on right
- ✅ Author name in light gray above title
- ✅ Book title in bold white
- ✅ Duration with clock icon and chapter count
- ✅ No back arrow

### Content:

- ✅ Chapter title/section heading
- ✅ Scrollable content
- ✅ "Mark as finished" button on last chapter (centered, bordered)

### Bottom Navigation:

- ✅ Chapter counter on left (e.g., "1/8")
- ✅ Circular green next button (with arrow icon)
- ✅ Green "Done" button on last chapter

### Behavior:

- ✅ No exit confirmation dialog
- ✅ Auto-saves progress on back
- ✅ System back gesture works seamlessly

## Notes

- Currently using sample data generated from book metadata
- In production, chapters should be loaded from Firestore
- Progress saving is stubbed out (needs Firestore integration)
- All navigation and UI interactions are fully functional
- Design perfectly matches the provided screenshots
