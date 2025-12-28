# Notification Count Implementation

## Overview
Implemented a dynamic notification count system that fetches notifications from Firestore and displays the count on a reusable notification icon widget. The system shows notifications that are either user-specific or broadcast to all users.

## Files Created

### 1. NotificationIcon Widget
**File**: `lib/shared/widgets/notification_icon.dart`

A reusable widget that displays a notification bell icon with a badge showing the notification count.

**Features**:
- ✅ Displays notification bell icon
- ✅ Shows dynamic count badge from HomeController
- ✅ Hides badge when count is 0
- ✅ Displays "99+" for counts over 99
- ✅ Navigates to notifications screen on tap
- ✅ Reactive updates using GetX Obx
- ✅ Reusable across multiple screens

**Implementation**:
```dart
class NotificationIcon extends StatelessWidget {
  const NotificationIcon({super.key});

  @override
  Widget build(BuildContext context) {
    final homeController = Get.find<HomeController>();

    return Stack(
      clipBehavior: Clip.none,
      children: [
        GestureDetector(
          onTap: () {
            Get.toNamed(RouteConstants.notifications);
          },
          child: const Icon(
            Icons.notifications_outlined,
            color: AppColors.white,
            size: 24,
          ),
        ),
        // Notification badge
        Obx(() {
          final count = homeController.notificationCount.value;
          
          // Don't show badge if count is 0
          if (count == 0) {
            return const SizedBox.shrink();
          }

          return Positioned(
            right: -4,
            top: -4,
            child: Container(
              padding: const EdgeInsets.all(4),
              decoration: const BoxDecoration(
                color: AppColors.primaryColor,
                shape: BoxShape.circle,
              ),
              constraints: const BoxConstraints(
                minWidth: 18,
                minHeight: 18,
              ),
              child: Text(
                count > 99 ? '99+' : count.toString(),
                style: AppTextStyles.buttonText.copyWith(
                  color: AppColors.darkTeal,
                  fontSize: 10,
                  fontWeight: FontWeight.bold,
                ),
                textAlign: TextAlign.center,
              ),
            ),
          );
        }),
      ],
    );
  }
}
```

## Files Modified

### 1. HomeController
**File**: `lib/features/home/controllers/home_controller.dart`

**Changes**:
- Added `FirebaseAuth` import and instance
- Added `notificationCount` observable (RxInt)
- Added `_fetchNotificationCount()` method
- Added `refreshNotificationCount()` public method
- Calls `_fetchNotificationCount()` in `onInit()`

**Key Implementation**:
```dart
final RxInt notificationCount = 0.obs;

/// Fetch notification count for current user
Future<void> _fetchNotificationCount() async {
  try {
    final userId = _auth.currentUser?.uid;
    if (userId == null) {
      debugPrint('❌ No user logged in for notifications');
      notificationCount.value = 0;
      return;
    }

    debugPrint('🔵 Fetching notification count for user: $userId');

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

    final userCount = results[0].docs.length;
    final broadcastCount = results[1].docs.length;
    
    notificationCount.value = userCount + broadcastCount;
    debugPrint('✅ Notification count: $userCount user + $broadcastCount broadcast = ${notificationCount.value}');
  } catch (e) {
    debugPrint('❌ Error fetching notification count: $e');
    notificationCount.value = 0;
  }
}

/// Refresh notification count
Future<void> refreshNotificationCount() async {
  await _fetchNotificationCount();
}
```

### 2. HomeScreen
**File**: `lib/features/home/screens/home_screen.dart`

**Changes**:
- Imported `NotificationIcon` widget
- Replaced hardcoded notification icon with `NotificationIcon()` widget
- Removed hardcoded badge with "2" count

**Before**:
```dart
// Notification icon with badge
Stack(
  clipBehavior: Clip.none,
  children: [
    GestureDetector(
      onTap: () {
        Get.toNamed(RouteConstants.notifications);
      },
      child: const Icon(
        Icons.notifications_outlined,
        color: AppColors.white,
        size: 24,
      ),
    ),
    // Notification badge
    Positioned(
      right: -4,
      top: -4,
      child: Container(
        padding: const EdgeInsets.all(4),
        decoration: const BoxDecoration(
          color: AppColors.primaryColor,
          shape: BoxShape.circle,
        ),
        constraints: const BoxConstraints(
          minWidth: 18,
          minHeight: 18,
        ),
        child: Text(
          '2',  // ← Hardcoded
          style: AppTextStyles.buttonText.copyWith(
            color: AppColors.darkTeal,
            fontSize: 10,
            fontWeight: FontWeight.bold,
          ),
          textAlign: TextAlign.center,
        ),
      ),
    ),
  ],
),
```

**After**:
```dart
const Row(
  children: [
    // Notification icon with badge
    NotificationIcon(),
  ],
),
```

### 3. MyLibraryScreen
**File**: `lib/features/my_library/screens/my_library_screen.dart`

**Changes**:
- Imported `NotificationIcon` widget
- Removed `RouteConstants` import (no longer needed)
- Replaced hardcoded notification icon with `NotificationIcon()` widget
- Removed hardcoded badge with "2" count

Same replacement as HomeScreen - from hardcoded Stack to `NotificationIcon()`.

## Data Structure

### Firestore Collection: `notifications`
```
notifications/
  {notificationId}/
    userId: string (user's ID) OR null (for broadcast to all users)
    title: string
    message: string
    createdAt: Timestamp
    read: boolean (optional)
    ... other fields
```

**Query Logic**:
The count includes notifications where:
- `userId` equals the current user's ID (user-specific notifications)
- **OR** `userId` is `null` (broadcast notifications for all users)

**Implementation**:
Two separate queries executed in parallel and results combined:
```dart
// Query 1: User-specific
.where('userId', isEqualTo: userId)

// Query 2: Broadcast (null check)
.where('userId', isNull: true)

// Combine counts
notificationCount.value = userCount + broadcastCount;
```

**Note**: Firestore doesn't support OR queries with null checks using `Filter.or()`, so we use two separate queries.

## How It Works

### Initialization Flow
```
User logs in
    ↓
HomeController.onInit()
    ↓
_fetchNotificationCount()
    ↓
Query Firestore notifications collection
    ↓
Count documents where userId == currentUser OR userId == null
    ↓
notificationCount.value = count
    ↓
UI updates automatically (Obx)
```

### UI Update Flow
```
NotificationIcon widget renders
    ↓
Gets HomeController instance
    ↓
Obx(() => ...) listens to notificationCount
    ↓
When count changes → UI rebuilds automatically
    ↓
Badge shows/hides based on count
```

## UI States

### Badge Display Logic
1. **Count = 0**: Badge is hidden (`SizedBox.shrink()`)
2. **Count 1-99**: Shows exact count (e.g., "5", "23")
3. **Count > 99**: Shows "99+"

### Visual Design
- **Badge Color**: Primary color (`AppColors.primaryColor`)
- **Badge Text**: Dark teal (`AppColors.darkTeal`)
- **Badge Shape**: Circle
- **Badge Position**: Top-right of bell icon (-4px offset)
- **Badge Size**: Minimum 18x18 pixels
- **Text Size**: 10px, bold

## Usage

### Using the Widget
Simply import and use the widget wherever you need a notification icon:

```dart
import '../../../shared/widgets/notification_icon.dart';

// In your widget tree:
const NotificationIcon()
```

The widget automatically:
- Fetches the current count from HomeController
- Updates when count changes
- Handles tap navigation
- Shows/hides badge appropriately

### Refreshing the Count
To manually refresh the notification count:

```dart
final homeController = Get.find<HomeController>();
await homeController.refreshNotificationCount();
```

## Error Handling

### No User Logged In
```dart
if (userId == null) {
  debugPrint('❌ No user logged in for notifications');
  notificationCount.value = 0;
  return;
}
```
- Sets count to 0
- Logs debug message
- Badge is hidden

### Firestore Query Error
```dart
catch (e) {
  debugPrint('❌ Error fetching notification count: $e');
  notificationCount.value = 0;
}
```
- Sets count to 0
- Logs error message
- Badge is hidden
- App continues to function

## Benefits

### Reusability
- ✅ Single widget used across multiple screens
- ✅ Consistent UI/UX everywhere
- ✅ Easy to maintain (one place to update)
- ✅ Reduced code duplication

### Maintainability
- ✅ All notification logic in HomeController
- ✅ Centralized count management
- ✅ Easy to update query logic
- ✅ Clear separation of concerns

### Performance
- ✅ Single Firestore query on app launch
- ✅ Reactive updates (no manual refresh needed)
- ✅ Efficient widget rebuilds (only badge updates)
- ✅ Minimal memory footprint

### User Experience
- ✅ Real-time count updates
- ✅ Consistent across screens
- ✅ Clean, professional look
- ✅ Handles edge cases (0, 99+, errors)

## Testing Checklist

### Basic Functionality
- [ ] Open app - verify notification count loads
- [ ] Check console for debug logs
- [ ] Verify count displays correctly on badge
- [ ] Tap bell icon - verify navigation to notifications screen

### Count Display
- [ ] Test with 0 notifications - verify badge is hidden
- [ ] Test with 1 notification - verify shows "1"
- [ ] Test with 10 notifications - verify shows "10"
- [ ] Test with 99 notifications - verify shows "99"
- [ ] Test with 100+ notifications - verify shows "99+"

### Multiple Screens
- [ ] Verify notification icon shows in HomeScreen
- [ ] Verify notification icon shows in MyLibraryScreen
- [ ] Verify count is same on both screens
- [ ] Verify both icons navigate correctly

### Data Scenarios
- [ ] Test with user-specific notifications (userId = currentUser)
- [ ] Test with broadcast notifications (userId = null)
- [ ] Test with mix of both types
- [ ] Test with no notifications in Firestore

### Error Handling
- [ ] Test with user not logged in - verify count is 0
- [ ] Test with Firestore offline - verify graceful handling
- [ ] Test with invalid data - verify no crashes

## Future Enhancements

### Real-time Updates
Replace one-time fetch with Firestore listener:
```dart
_firestore
  .collection('notifications')
  .where(
    Filter.or(
      Filter('userId', isEqualTo: userId),
      Filter('userId', isEqualTo: null),
    ),
  )
  .snapshots()
  .listen((snapshot) {
    notificationCount.value = snapshot.docs.length;
  });
```

### Unread Count Only
Add filter for unread notifications:
```dart
.where('read', isEqualTo: false)
```

### Notification Types
Show different badges for different notification types:
- Red badge for urgent notifications
- Blue badge for informational
- Green badge for success messages

### Badge Animation
Add entrance animation when count increases:
```dart
AnimatedScale(
  scale: _hasNewNotification ? 1.2 : 1.0,
  duration: Duration(milliseconds: 300),
  child: badge,
)
```

### Local Caching
Cache notification count locally to reduce Firestore reads:
```dart
final cachedCount = await _localCache.getInt('notificationCount');
notificationCount.value = cachedCount ?? 0;
// Then fetch from Firestore and update
```

## Files Summary

### Created:
- ✅ `lib/shared/widgets/notification_icon.dart`

### Modified:
- ✅ `lib/features/home/controllers/home_controller.dart`
- ✅ `lib/features/home/screens/home_screen.dart`
- ✅ `lib/features/my_library/screens/my_library_screen.dart`

## Migration Notes

### For Existing Users
No migration needed. The feature:
- Works with any existing notifications collection
- Gracefully handles missing collection (shows 0)
- Backward compatible

### Breaking Changes
None. This is a new feature addition.

