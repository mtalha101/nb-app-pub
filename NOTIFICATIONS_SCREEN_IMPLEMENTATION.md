# Notifications Screen - Dynamic Implementation

## Overview
The Notifications screen has been fully implemented with a controller to fetch and display actual notifications from Firestore. The implementation includes adaptive loading indicators, empty states, image handling, and action routing.

## Files Created

### 1. NotificationModel
**File**: `lib/features/notifications/models/notification_model.dart`

A model representing a notification document from Firestore.

**Fields**:
- `id` - Notification document ID
- `userId` - User ID (null for broadcast notifications)
- `title` - Notification title
- `body` - Notification message
- `imageUrl` - Optional image URL for visual content
- `actionCall` - Optional action (route or URL) to execute on tap
- `createdAt` - Timestamp when notification was created

**Helper Methods**:
```dart
String get timeAgo         // Returns "Just now", "5m ago", "2h ago", "Yesterday", "3d ago", "2w ago"
bool get isBroadcast       // Returns true if userId is null
```

**Time Ago Logic**:
- < 1 minute: "Just now"
- < 1 hour: "Xm ago" (e.g., "5m ago", "45m ago")
- < 1 day: "Xh ago" (e.g., "2h ago", "18h ago")
- 1 day: "Yesterday"
- < 1 week: "Xd ago" (e.g., "3d ago", "6d ago")
- ≥ 1 week: "Xw ago" (e.g., "2w ago", "4w ago")

### 2. NotificationsController
**File**: `lib/features/notifications/controllers/notifications_controller.dart`

A controller that manages notifications fetching and actions.

**Features**:
- ✅ Fetches notifications from Firestore
- ✅ Filters by current user + broadcast notifications
- ✅ Sorts by creation time (newest first)
- ✅ Handles notification tap actions
- ✅ Supports marking as read (future)
- ✅ Supports deletion (future)
- ✅ Updates HomeController count when needed

**Key Methods**:
```dart
Future<void> _fetchNotifications()                      // Fetch all notifications
Future<void> refreshNotifications()                      // Manually refresh
void handleNotificationAction(notification)             // Handle tap action
Future<void> markAsRead(String notificationId)          // Mark as read
Future<void> deleteNotification(String notificationId)  // Delete notification
```

**Firestore Query**:
```dart
// Query 1: User-specific notifications
final userNotificationsQuery = _firestore
    .collection('notifications')
    .where('userId', isEqualTo: userId);

// Query 2: Broadcast notifications (userId is null)
final broadcastNotificationsQuery = _firestore
    .collection('notifications')
    .where('userId', isNull: true);

// Execute both queries in parallel
final results = await Future.wait([
  userNotificationsQuery.get(),
  broadcastNotificationsQuery.get(),
]);

// Combine and sort results by createdAt (newest first)
notificationList.sort((a, b) => b.createdAt.compareTo(a.createdAt));
```

**Note**: Two separate queries are used instead of `Filter.or()` because Firestore doesn't support OR queries with null checks in a single query.

**Action Handling**:
```dart
void handleNotificationAction(NotificationModel notification) {
  final action = notification.actionCall;
  
  if (action.startsWith('/')) {
    // Navigation action (e.g., "/subscription", "/book-details")
    Get.toNamed(action);
  } else if (action.startsWith('http')) {
    // External URL - Open in browser
  } else {
    // Custom action handler
  }
}
```

### 3. NotificationsBinding
**File**: `lib/features/notifications/bindings/notifications_binding.dart`

GetX binding for lazy loading the NotificationsController.

### 4. Updated NotificationsScreen
**File**: `lib/features/notifications/screens/notifications_screen.dart`

Complete refactor to use the controller with dynamic data.

**Features**:
- ✅ Adaptive loading indicator (Cupertino for iOS, Material for Android)
- ✅ Empty state with icon and message
- ✅ Dynamic notification list from Firestore
- ✅ Image display (if imageUrl provided)
- ✅ Time ago formatting
- ✅ Action indicators ("Tap to view" with arrow)
- ✅ Tap handling for navigation/actions
- ✅ Star badge on notifications with images
- ✅ Reactive updates using Obx

## Files Modified

### 1. AppPages (Routes)
**File**: `lib/core/routes/pages.dart`

**Changes**:
- Added `NotificationsBinding` import
- Added binding to notifications route

```dart
GetPage(
  name: RouteConstants.notifications,
  page: () => const NotificationsScreen(),
  binding: NotificationsBinding(),  // ← Added
),
```

## Data Structure

### Firestore Collection: `notifications`
```javascript
notifications/
  {notificationId}/
    userId: string | null          // null = broadcast to all users
    title: string                  // "🔥 Najeeb for half the price"
    body: string                   // "Limited time offer! Get..."
    imageUrl: string (optional)    // URL to notification image
    actionCall: string (optional)  // "/subscription" or "http://..."
    createdAt: Timestamp          // When notification was created
    read: boolean (optional)       // Whether user has read it
```

**Example Documents**:

```javascript
// Broadcast offer notification
{
  userId: null,
  title: "🔥 Najeeb for half the price",
  body: "Limited time offer! Get Najeeb for 50% off...",
  imageUrl: "https://firebasestorage.../offer.png",
  actionCall: "/subscription",
  createdAt: Timestamp(2025, 11, 1, 10, 30),
  read: false
}

// User-specific notification
{
  userId: "abc123xyz",
  title: "New book in your favorite category",
  body: "Check out the latest addition to Psychology...",
  imageUrl: "https://firebasestorage.../book-cover.jpg",
  actionCall: "/book-details?id=psychology-101",
  createdAt: Timestamp(2025, 11, 2, 15, 45),
  read: false
}

// Simple notification without image
{
  userId: "abc123xyz",
  title: "Reading streak reminder",
  body: "You're on a 7-day reading streak! Keep it up!",
  actionCall: null,
  createdAt: Timestamp(2025, 11, 3, 09, 00),
  read: true
}
```

## UI States

### Loading State
- Shows adaptive loading indicator
  - **iOS**: `CupertinoActivityIndicator` with radius 20
  - **Android**: `CircularProgressIndicator`
- Centered on screen
- Primary color

### Empty State
- Icon: Large notification bell (`Icons.notifications_outlined`)
- Title: "No notifications yet"
- Subtitle: "We'll notify you when there's something new"
- Centered vertically and horizontally
- Muted white color with transparency

### Notification Card States

#### With Image
```
┌─────────────────────────┐
│     [Image 160px]       │ ← imageUrl displayed
│      [Star Badge]       │
├─────────────────────────┤
│ Timestamp               │
│ Title (Bold)            │
│ Body text               │
│ [Tap to view →]         │ ← If actionCall exists
└─────────────────────────┘
```

#### Without Image
```
┌─────────────────────────┐
│ Timestamp               │
│ Title (Bold)            │
│ Body text               │
│ [Tap to view →]         │ ← If actionCall exists
└─────────────────────────┘
```

## UI Components

### Notification Card Layout
1. **Image Section** (optional, 160px height)
   - Full-width image
   - Star badge in top-right corner
   - Rounded top corners
   - Loading placeholder with spinner
   - Error fallback with icon

2. **Content Section** (16px padding)
   - Timestamp (small, muted)
   - Title (18px, bold, white)
   - Body (14px, white, line height 1.4)
   - Action indicator (if actionCall exists)

3. **Action Indicator**
   - Text: "Tap to view"
   - Arrow icon
   - Primary color
   - Only shows if actionCall is provided

### Visual Design
- **Card Background**: `AppColors.bookDarkBlue`
- **Card Radius**: 12px
- **Card Spacing**: 16px between cards
- **Padding**: 16px all around

## How It Works

### Initialization Flow
```
User taps notification icon
    ↓
Navigate to NotificationsScreen
    ↓
NotificationsBinding creates NotificationsController
    ↓
Controller.onInit() calls _fetchNotifications()
    ↓
Query Firestore for user's notifications + broadcasts
    ↓
Parse and sort by createdAt (newest first)
    ↓
Display in ListView
```

### Notification Types Supported

#### 1. Simple Text Notification
```dart
{
  title: "Welcome to Najeeb!",
  body: "Start exploring our library of books",
  // No image, no action
}
```

#### 2. Image Notification
```dart
{
  title: "New book available",
  body: "Check out our latest addition",
  imageUrl: "https://...",
  // Has image, no action
}
```

#### 3. Actionable Notification
```dart
{
  title: "Special offer",
  body: "Get 50% off today!",
  imageUrl: "https://...",
  actionCall: "/subscription",  // Navigate to subscription
}
```

#### 4. Broadcast Notification
```dart
{
  userId: null,  // Sent to all users
  title: "Platform update",
  body: "New features available",
}
```

## Action Types

### Navigation Actions (routes)
```dart
actionCall: "/subscription"       // Navigate to subscription screen
actionCall: "/book-details"       // Navigate to book details
actionCall: "/home"               // Navigate to home
```

### URL Actions
```dart
actionCall: "https://najeeb.com"  // Open external URL
actionCall: "https://blog.najeeb.com/article"
```

### Custom Actions (future)
```dart
actionCall: "open_book:abc123"    // Custom action format
actionCall: "play_audio:xyz789"
```

## Error Handling

### No User Logged In
```dart
if (userId == null) {
  debugPrint('❌ No user logged in');
  isLoading.value = false;
  return;
}
```
- Shows loading state then empty state
- No error message displayed to user

### Firestore Query Error
```dart
catch (e) {
  debugPrint('❌ Error fetching notifications: $e');
  isLoading.value = false;
}
```
- Logs error to console
- Shows empty state to user
- App continues to function

### Individual Notification Parse Error
```dart
try {
  final notification = NotificationModel.fromFirestore(doc.data(), doc.id);
  notificationList.add(notification);
} catch (e) {
  debugPrint('❌ Error parsing notification ${doc.id}: $e');
  // Continue processing other notifications
}
```
- Skips problematic notification
- Continues loading others
- No UI disruption

### Image Load Error
```dart
errorWidget: (context, url, error) => Container(
  color: AppColors.bookDarkBlue,
  child: Icon(
    Icons.image_outlined,
    color: AppColors.white.withValues(alpha: 0.5),
  ),
),
```
- Shows placeholder icon
- Maintains card layout
- No crash or blank space

## Integration with HomeController

The NotificationsController integrates with HomeController for count management:

### When Notifications are Read
```dart
await controller.markAsRead(notificationId);
    ↓
Firestore updated: { read: true }
    ↓
HomeController.refreshNotificationCount()
    ↓
Badge count updates
```

### When Notifications are Deleted
```dart
await controller.deleteNotification(notificationId);
    ↓
Document deleted from Firestore
    ↓
NotificationsController.refreshNotifications()
    ↓
HomeController.refreshNotificationCount()
    ↓
Both list and count update
```

## Testing Checklist

### Basic Functionality
- [ ] Navigate to notifications screen
- [ ] Verify adaptive loading indicator shows
  - [ ] iOS: Cupertino spinner
  - [ ] Android: Material spinner
- [ ] Verify notifications load from Firestore
- [ ] Verify notifications sorted by newest first

### Empty State
- [ ] Test with no notifications - verify empty state shows
- [ ] Verify icon, title, and subtitle display correctly
- [ ] Verify centered layout

### Notification Display
- [ ] Test notification with image - verify image loads
- [ ] Test notification without image - verify no broken layout
- [ ] Test notification with action - verify "Tap to view" shows
- [ ] Test notification without action - verify no action text
- [ ] Verify time ago formatting (minutes, hours, days, weeks)
- [ ] Verify star badge shows on notifications with images

### Notification Types
- [ ] Test user-specific notification (userId = currentUser)
- [ ] Test broadcast notification (userId = null)
- [ ] Test mix of both types
- [ ] Verify all show correctly

### Actions
- [ ] Tap notification with route action - verify navigation works
- [ ] Tap notification with URL action - verify handling
- [ ] Tap notification without action - verify no error

### Edge Cases
- [ ] Test with 0 notifications - verify empty state
- [ ] Test with 20+ notifications - verify scrolling works
- [ ] Test with very long title - verify truncation
- [ ] Test with very long body - verify line wrapping
- [ ] Test with broken image URL - verify placeholder shows
- [ ] Test with missing fields - verify graceful handling

## Future Enhancements

### Real-time Updates
Use Firestore snapshot listener:
```dart
_firestore
  .collection('notifications')
  .where(...)
  .orderBy('createdAt', descending: true)
  .snapshots()
  .listen((snapshot) {
    // Update notifications automatically
  });
```

### Read/Unread Filtering
Add tabs or filters:
- All notifications
- Unread only
- Read only

```dart
.where('read', isEqualTo: false)  // Unread only
```

### Notification Categories
Group notifications by type:
- Offers
- Updates
- Reminders
- Social

### Swipe Actions
Add swipe-to-delete or swipe-to-mark-read:
```dart
Dismissible(
  key: Key(notification.id),
  onDismissed: (direction) {
    controller.deleteNotification(notification.id);
  },
  child: notificationCard,
)
```

### Push Notifications Integration
Link with Firebase Cloud Messaging:
- Receive push notifications
- Auto-add to Firestore collection
- Show badge on app icon
- Handle background notifications

### Rich Media Support
- Video thumbnails
- GIFs
- Multiple images
- Audio previews

### Actions Enhancement
- Deep linking support
- Custom action handlers
- Action buttons (Accept/Decline)
- Inline actions without navigation

## Performance Considerations

### Pagination
For users with many notifications:
```dart
.limit(20)                    // Load 20 at a time
.startAfter(lastDocument)     // Load next page
```

### Image Optimization
- Use `cached_network_image` for caching
- Compress images on server
- Lazy load images as user scrolls

### Query Optimization
- Create composite index in Firestore
- Cache recent notifications locally
- Use `get()` instead of `snapshots()` for initial load

## Files Summary

### Created:
- ✅ `lib/features/notifications/models/notification_model.dart`
- ✅ `lib/features/notifications/controllers/notifications_controller.dart`
- ✅ `lib/features/notifications/bindings/notifications_binding.dart`

### Modified:
- ✅ `lib/features/notifications/screens/notifications_screen.dart` (Complete refactor)
- ✅ `lib/core/routes/pages.dart` (Added binding)

## Integration Points

### HomeController
- Fetches notification count
- Displays in NotificationIcon widget
- Refreshed when notifications are read/deleted

### NotificationIcon Widget
- Displays count from HomeController
- Navigates to NotificationsScreen
- Badge auto-hides when count is 0

### Navigation
- Route: `RouteConstants.notifications`
- Binding: `NotificationsBinding()`
- Navigation: `Get.toNamed(RouteConstants.notifications)`

## Sample Data for Testing

To test the screen, add documents to Firestore:

```javascript
// In Firestore Console: Create collection "notifications"

// Document 1: Broadcast offer with image
{
  userId: null,
  title: "🔥 Najeeb for half the price",
  body: "Limited time offer! Get Najeeb for 50% off and access all +7,500 book titles.",
  imageUrl: "https://firebasestorage.googleapis.com/.../offer-banner.png",
  actionCall: "/subscription",
  createdAt: Timestamp.now(),
  read: false
}

// Document 2: User-specific without image
{
  userId: "your-user-id-here",
  title: "Your reading streak is on fire! 🔥",
  body: "You've read 3 books this week. Keep up the great work!",
  actionCall: null,
  createdAt: Timestamp.now(),
  read: false
}

// Document 3: Broadcast with custom action
{
  userId: null,
  title: "What would Elon read? 🤷‍♂️",
  body: "No need to wonder, we've compiled his reading list for you.",
  imageUrl: "https://firebasestorage.googleapis.com/.../elon-reading-list.png",
  actionCall: "/explore?list=elon-musk",
  createdAt: Timestamp.now(),
  read: false
}
```

## Dependencies

No new dependencies added. Using existing:
- `get` - State management and navigation
- `cloud_firestore` - Database
- `firebase_auth` - User authentication
- `cached_network_image` - Image caching and loading
- `flutter/cupertino` - iOS-style widgets
- `flutter/material` - Android-style widgets

## Migration Notes

### For Existing Users
- Works with existing or new notifications collection
- Gracefully handles missing collection (shows empty state)
- No breaking changes to existing data

### Firestore Indexes Required

No composite indexes are required! Since we use two simple queries (one for userId equality, one for null check), Firestore can handle them with automatic single-field indexes.

If you later add additional filters (e.g., `.where('read', isEqualTo: false)`), you may need composite indexes like:

```
Collection: notifications
Fields:
  - userId (Ascending)
  - read (Ascending)
  - createdAt (Descending)
```

Firestore will automatically prompt you to create the index when needed.

## Notes

### Why Two Separate Queries?
Firestore's `Filter.or()` doesn't support OR queries with null checks (`isEqualTo: null`). The error was:
```
Failed assertion: 'Exactly one operator must be specified'
```

**Solution**: Execute two queries in parallel using `Future.wait()`:
1. Query for user-specific notifications: `where('userId', isEqualTo: userId)`
2. Query for broadcast notifications: `where('userId', isNull: true)`
3. Combine results and sort in memory

**Benefits**:
- ✅ Works reliably with Firestore's query limitations
- ✅ Parallel execution is fast (both queries run simultaneously)
- ✅ No composite indexes needed
- ✅ Clean, maintainable code

### Why Two Queries in HomeController?
- **HomeController**: Gets count only (faster)
- **NotificationsController**: Gets full documents (when screen opens)
- Avoids loading all notification data until user opens the screen

### Controller Lifecycle
- Created when NotificationsScreen opens (via binding)
- Disposed when screen closes
- Automatically cleaned up by GetX

### User-specific vs Broadcast
- **User-specific** (`userId: "abc123"`): Only that user sees it
- **Broadcast** (`userId: null`): All users see it
- Query combines both using `Filter.or()`

### Action Call Flexibility
The `actionCall` field supports multiple formats:
- Routes: `/subscription`, `/book-details`, `/home`
- URLs: `https://najeeb.com`, `https://blog.najeeb.com`
- Custom: `open_book:123`, `play_audio:456` (requires custom handler)

This flexibility allows you to:
- Deep link to any screen in the app
- Open external web pages
- Trigger custom actions (with additional implementation)

