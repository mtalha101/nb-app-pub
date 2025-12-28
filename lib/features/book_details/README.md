# Book Details Screen

A comprehensive book details screen that supports both subscribed and non-subscribed users.

## Features

### For All Users:

- ✅ Book cover with semicircular background design
- ✅ Book title, author, subtitle, and duration
- ✅ Category tags
- ✅ Bookmark and share functionality
- ✅ Two tabs: "About" and "Outline"
- ✅ "About" tab with sections:
  - What's it about?
  - Who is it for?
  - Meet the author
  - You might also like (recommended books)
- ✅ "Outline" tab with chapter list and timestamps

### For Subscribed Users (Unlocked):

- ✅ "Read" button to start reading
- ✅ "Play" button to start audio playback
- ✅ Access to all chapters

### For Non-Subscribed Users (Locked):

- ✅ "Unlock" button prominently displayed
- ✅ Lock icon on book cover
- ✅ Chapters visible but not accessible

## Usage

To navigate to the book details screen, use the `BookNavigation` helper:

```dart
import 'package:your_app/core/navigation/book_navigation.dart';

// Example: Navigate to book details
BookNavigation.toBookDetails(
  bookId: 'book123',
  title: 'Coming Up Short',
  author: 'Robert B. Reich',
  subtitle: 'A Memoir of My America',
  coverUrl: 'https://example.com/cover.jpg',
  duration: '20 min',
  chapters: 8,
  isLocked: false, // true for non-subscribed users
  backgroundColor: Colors.orange.shade300,
  categories: ['Biography & Memoir', 'Society & Culture'],
  aboutDescription: 'Coming Up Short (2025) shares politician and academic Robert Reich\'s insider perspective...',
  whoIsItFor: 'Political activists for a primer on real-world progressive policy...',
  authorBio: 'Robert Reich is a former U.S. Labor Secretary under Bill Clinton...',
);
```

## Example: From Home Screen

```dart
// In your book card's onTap:
GestureDetector(
  onTap: () {
    BookNavigation.toBookDetails(
      bookId: book.id,
      title: book.title,
      author: book.author,
      subtitle: book.subtitle,
      coverUrl: book.coverUrl,
      duration: book.duration,
      chapters: book.chapters,
      isLocked: !userIsSubscribed, // Check user subscription status
      backgroundColor: book.backgroundColor,
      categories: book.categories,
      aboutDescription: book.aboutDescription,
      whoIsItFor: book.whoIsItFor,
      authorBio: book.authorBio,
    );
  },
  child: YourBookCard(),
)
```

## Screenshots Implemented

Based on the provided screenshots, this screen implements:

1. ✅ Top bar with back button and title
2. ✅ Book cover with semicircular background
3. ✅ Bookmark and share icons
4. ✅ Action buttons (Read/Play or Unlock)
5. ✅ Book info section with duration, title, subtitle, and category tags
6. ✅ About/Outline tabs
7. ✅ All sections in About tab
8. ✅ Chapter list in Outline tab with timestamps
9. ✅ "You might also like" recommendations
10. ✅ Lock/Unlock state for subscription management

## TODOs

- [ ] Implement actual bookmark functionality with backend
- [ ] Implement share functionality
- [ ] Connect to actual book data from Firestore
- [ ] Implement reading screen navigation
- [ ] Implement audio playback
- [ ] Connect to subscription/payment flow for unlock button
- [ ] Add chapter navigation/playback
- [ ] Load real recommended books from backend
