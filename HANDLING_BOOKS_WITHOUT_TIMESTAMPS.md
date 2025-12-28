# 📚 Handling Books Without Timestamps

## 🎯 Problem

Current implementation **crashes** if timestamp data is missing because:

- `_updateHighlight()` assumes `sentence.start` and `sentence.end` exist
- `_seekToSentence()` tries to access timing that might not exist
- Models don't handle optional/null values

---

## 🏗️ Database Structure for Both Scenarios

### **Scenario 1: Book WITH Audio + Timestamps (Active Highlighting)**

```javascript
books/{bookId}/chapters/{chapterId}
{
  "title": "الملخص",
  "order": 0,
  "hasAudio": true,  // ← Indicates audio is available
  "audioLink": "https://...",
  "audioLength": "12:03",
  "sentences": [
    {
      "text": "العاداتُ هي الأشياءُ التي تُميّزُنا",
      "start": 0,
      "end": 6
    },
    {
      "text": "تتكوَّن العادةُ عندما تقومُ بنشاطٍ معيّن",
      "start": 6,
      "end": 11
    }
    // ... more with timestamps
  ]
}
```

### **Scenario 2: Book WITHOUT Audio (Text-Only Reading)**

```javascript
books/{bookId}/chapters/{chapterId}
{
  "title": "الفصل الأول",
  "order": 0,
  "hasAudio": false,  // ← No audio available
  "content": "العاداتُ هي الأشياءُ التي تُميّزُنا تتكوَّن العادةُ عندما تقومُ بنشاطٍ معيّن...",

  // OR split into sentences without timing
  "sentences": [
    { "text": "العاداتُ هي الأشياءُ التي تُميّزُنا" },
    { "text": "تتكوَّن العادةُ عندما تقومُ بنشاطٍ معيّن" }
    // No start/end fields
  ],

  // OR structured paragraphs
  "paragraphs": [
    {
      "title": "المقدمة",
      "content": "العاداتُ هي الأشياءُ التي تُميّزُنا..."
    },
    {
      "title": "التعريف",
      "content": "تتكوَّن العادةُ عندما..."
    }
  ]
}
```

---

## 🔧 Updated Models

### **1. Update SentenceModel (Make timing optional)**

```dart
// lib/features/reading/models/sentence_model.dart
class SentenceModel {
  final String text;
  final double? start;  // ← Made optional
  final double? end;    // ← Made optional

  SentenceModel({
    required this.text,
    this.start,  // No 'required'
    this.end,    // No 'required'
  });

  factory SentenceModel.fromMap(Map<String, dynamic> map) {
    return SentenceModel(
      text: map['text'] as String? ?? '',
      start: map['start'] != null ? (map['start'] as num).toDouble() : null,
      end: map['end'] != null ? (map['end'] as num).toDouble() : null,
    );
  }

  Map<String, dynamic> toMap() {
    return {
      'text': text,
      if (start != null) 'start': start,  // Only include if present
      if (end != null) 'end': end,
    };
  }

  // Check if this sentence has timing data
  bool get hasTiming => start != null && end != null;
}
```

### **2. Update ChapterModel (Support multiple formats)**

```dart
// lib/features/reading/models/chapter_model.dart
class ChapterModel {
  final String id;
  final String title;
  final int order;
  final bool hasAudio;
  final String? audioLink;
  final String? audioLength;
  final List<SentenceModel> sentences;
  final String? fullContent;  // For text-only books
  final double startTime;
  final double endTime;
  final int sentenceCount;

  ChapterModel({
    required this.id,
    required this.title,
    required this.order,
    required this.hasAudio,
    this.audioLink,
    this.audioLength,
    required this.sentences,
    this.fullContent,
    required this.startTime,
    required this.endTime,
    required this.sentenceCount,
  });

  factory ChapterModel.fromMap(String id, Map<String, dynamic> map) {
    // Parse sentences if they exist
    final sentencesData = map['sentences'] as List<dynamic>? ?? [];
    final sentences = sentencesData
        .map((s) => SentenceModel.fromMap(s as Map<String, dynamic>))
        .toList();

    // Check if audio is available
    final hasAudio = map['hasAudio'] as bool? ??
                     map['audioLink'] != null;

    // Get full content if available
    final fullContent = map['content'] as String?;

    return ChapterModel(
      id: id,
      title: map['title'] as String? ?? '',
      order: map['order'] as int? ?? 0,
      hasAudio: hasAudio,
      audioLink: map['audioLink'] as String?,
      audioLength: map['audioLength'] as String?,
      sentences: sentences,
      fullContent: fullContent,
      startTime: (map['startTime'] as num?)?.toDouble() ?? 0.0,
      endTime: (map['endTime'] as num?)?.toDouble() ?? 0.0,
      sentenceCount: map['sentenceCount'] as int? ?? sentences.length,
    );
  }

  // Check if this chapter has timing data for highlighting
  bool get hasTimingData {
    if (!hasAudio) return false;
    if (sentences.isEmpty) return false;
    return sentences.any((s) => s.hasTiming);
  }

  // Get text content (from sentences or fullContent)
  String get contentText {
    if (fullContent != null && fullContent!.isNotEmpty) {
      return fullContent!;
    }
    return sentences.map((s) => s.text).join(' ');
  }
}
```

---

## 🎨 Updated Reading Screen (Defensive Implementation)

```dart
// lib/features/reading/screens/reading_screen_with_audio.dart

class _ReadingScreenWithAudioState extends State<ReadingScreenWithAudio> {
  // ... existing fields ...

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  Future<void> _loadData() async {
    try {
      // Load book data
      final bookDoc = await FirebaseFirestore.instance
          .collection('books')
          .doc(widget.bookId)
          .get();

      if (bookDoc.exists) {
        _book = BookModel.fromMap(widget.bookId, bookDoc.data()!);
      }

      // Load chapter data
      final chapterDoc = await FirebaseFirestore.instance
          .collection('books')
          .doc(widget.bookId)
          .collection('chapters')
          .doc(widget.chapterId)
          .get();

      if (chapterDoc.exists) {
        _chapter = ChapterModel.fromMap(widget.chapterId, chapterDoc.data()!);
      }

      setState(() {
        _isLoading = false;
      });

      // ✅ Only initialize audio if chapter has audio
      if (_book != null && _chapter != null && _chapter!.hasAudio && _chapter!.audioLink != null) {
        await _initAudioPlayer();
      } else {
        print('📖 Text-only chapter (no audio)');
      }
    } catch (e) {
      print('Error loading data: $e');
      setState(() {
        _isLoading = false;
      });
    }
  }

  Future<void> _initAudioPlayer() async {
    try {
      await _audioPlayer.setUrl(_book!.audioLink);

      // Listen to player state
      _audioPlayer.playerStateStream.listen((state) {
        if (mounted) {
          setState(() {
            _isPlaying = state.playing;
          });
        }
      });

      // ✅ Only listen to position if chapter has timing data
      if (_chapter!.hasTimingData) {
        _audioPlayer.positionStream.listen((position) {
          if (mounted) {
            setState(() {
              _currentPosition = position;
            });
            _updateHighlight(position.inSeconds.toDouble());
          }
        });
      } else {
        print('⚠️ No timing data available for highlighting');
      }

      _audioPlayer.durationStream.listen((duration) {
        if (mounted && duration != null) {
          setState(() {
            _totalDuration = duration;
          });
        }
      });

      await _audioPlayer.setSpeed(_playbackSpeed);
    } catch (e) {
      print('Error initializing audio player: $e');
    }
  }

  void _updateHighlight(double currentTime) {
    // ✅ Guard: Check if timing data exists
    if (_chapter == null ||
        _chapter!.sentences.isEmpty ||
        !_chapter!.hasTimingData) {
      return;
    }

    final adjustedTime = currentTime - _audioOffsetSeconds;

    if (adjustedTime < 0) {
      if (_currentSentenceIndex != -1) {
        setState(() {
          _currentSentenceIndex = -1;
        });
      }
      return;
    }

    // Find the current sentence based on adjusted audio position
    for (int i = 0; i < _chapter!.sentences.length; i++) {
      final sentence = _chapter!.sentences[i];

      // ✅ Guard: Check if this sentence has timing
      if (sentence.hasTiming &&
          adjustedTime >= sentence.start! &&
          adjustedTime < sentence.end!) {
        if (_currentSentenceIndex != i) {
          setState(() {
            _currentSentenceIndex = i;
          });
          _scrollToSentence(i);
        }
        break;
      }
    }
  }

  void _seekToSentence(int index) {
    // ✅ Guard: Only seek if audio exists and sentence has timing
    if (_chapter == null ||
        !_chapter!.hasAudio ||
        index >= _chapter!.sentences.length) {
      return;
    }

    final sentence = _chapter!.sentences[index];
    if (sentence.hasTiming) {
      final seekTime = sentence.start! + _audioOffsetSeconds;
      _audioPlayer.seek(Duration(seconds: seekTime.toInt()));
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return Scaffold(
        backgroundColor: AppColors.darkTeal,
        body: Center(
          child: CircularProgressIndicator(
            color: AppColors.primaryGreen,
          ),
        ),
      );
    }

    if (_chapter == null) {
      return Scaffold(
        backgroundColor: AppColors.darkTeal,
        body: Center(
          child: Text(
            'فشل في تحميل المحتوى',
            style: AppTextStyles.welcomeTitle.copyWith(
              color: AppColors.white,
            ),
          ),
        ),
      );
    }

    return Scaffold(
      backgroundColor: AppColors.darkTeal,
      body: SafeArea(
        child: Column(
          children: [
            _buildTopBar(),
            Expanded(
              child: _buildContent(),
            ),
            // ✅ Only show audio controls if chapter has audio
            if (_chapter!.hasAudio && _chapter!.audioLink != null)
              _buildAudioControls(),
          ],
        ),
      ),
    );
  }

  Widget _buildParagraphWidget(List<int> sentenceIndices) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 24),
      child: RichText(
        textAlign: TextAlign.right,
        textDirection: TextDirection.rtl,
        text: TextSpan(
          style: AppTextStyles.welcomeSubtitle.copyWith(
            color: AppColors.white,
            fontSize: 18,
            height: 2.0,
            fontWeight: FontWeight.w400,
          ),
          children: sentenceIndices.map((index) {
            final sentence = _chapter!.sentences[index];
            // ✅ Only highlight if timing data exists
            final isHighlighted = _chapter!.hasTimingData &&
                                  _currentSentenceIndex == index;

            return TextSpan(
              text: '${sentence.text} ',
              style: TextStyle(
                color: isHighlighted ? AppColors.primaryGreen : AppColors.white,
                fontWeight: isHighlighted ? FontWeight.w600 : FontWeight.w400,
                backgroundColor: isHighlighted
                    ? AppColors.primaryGreen.withOpacity(0.2)
                    : Colors.transparent,
              ),
              // ✅ Only make tappable if timing exists
              recognizer: _chapter!.hasTimingData && sentence.hasTiming
                  ? (TapGestureRecognizer()..onTap = () => _seekToSentence(index))
                  : null,
            );
          }).toList(),
        ),
      ),
    );
  }

  Widget _buildAudioControls() {
    return Container(
      padding: const EdgeInsets.all(24),
      decoration: BoxDecoration(
        color: AppColors.white.withOpacity(0.05),
        border: Border(
          top: BorderSide(color: AppColors.white.withOpacity(0.1), width: 1),
        ),
      ),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          // Progress bar
          _buildProgressBar(),
          const SizedBox(height: 20),
          // Control buttons
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              // Speed control
              _buildSpeedButton(),
              // Play/Pause
              IconButton(
                icon: Icon(
                  _isPlaying ? Icons.pause_circle_filled : Icons.play_circle_filled,
                  size: 64,
                  color: AppColors.primaryGreen,
                ),
                onPressed: _togglePlayPause,
              ),
              // Placeholder for symmetry
              const SizedBox(width: 48),
            ],
          ),
        ],
      ),
    );
  }
}
```

---

## 📝 Alternative: Text-Only Reading Screen

For books without audio, you might want a separate, simpler screen:

```dart
// lib/features/reading/screens/text_reading_screen.dart
class TextReadingScreen extends StatefulWidget {
  final String bookId;
  final String chapterId;

  const TextReadingScreen({
    Key? key,
    required this.bookId,
    required this.chapterId,
  }) : super(key: key);

  @override
  State<TextReadingScreen> createState() => _TextReadingScreenState();
}

class _TextReadingScreenState extends State<TextReadingScreen> {
  ChapterModel? _chapter;
  bool _isLoading = true;
  final ScrollController _scrollController = ScrollController();
  double _fontSize = 18.0;

  @override
  void initState() {
    super.initState();
    _loadChapter();
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  Future<void> _loadChapter() async {
    try {
      final chapterDoc = await FirebaseFirestore.instance
          .collection('books')
          .doc(widget.bookId)
          .collection('chapters')
          .doc(widget.chapterId)
          .get();

      if (chapterDoc.exists) {
        setState(() {
          _chapter = ChapterModel.fromMap(widget.chapterId, chapterDoc.data()!);
          _isLoading = false;
        });
      }
    } catch (e) {
      print('Error loading chapter: $e');
      setState(() {
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return Scaffold(
        backgroundColor: AppColors.darkTeal,
        body: Center(
          child: CircularProgressIndicator(color: AppColors.primaryGreen),
        ),
      );
    }

    if (_chapter == null) {
      return Scaffold(
        backgroundColor: AppColors.darkTeal,
        body: Center(
          child: Text(
            'فشل في تحميل الفصل',
            style: TextStyle(color: AppColors.white),
          ),
        ),
      );
    }

    return Scaffold(
      backgroundColor: AppColors.darkTeal,
      appBar: AppBar(
        backgroundColor: AppColors.darkTeal,
        title: Text(
          _chapter!.title,
          style: TextStyle(color: AppColors.white),
        ),
        actions: [
          // Font size controls
          IconButton(
            icon: Icon(Icons.text_decrease, color: AppColors.white),
            onPressed: () {
              setState(() {
                _fontSize = (_fontSize - 2).clamp(12.0, 32.0);
              });
            },
          ),
          IconButton(
            icon: Icon(Icons.text_increase, color: AppColors.white),
            onPressed: () {
              setState(() {
                _fontSize = (_fontSize + 2).clamp(12.0, 32.0);
              });
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        controller: _scrollController,
        padding: const EdgeInsets.all(24),
        child: SelectableText(
          _chapter!.contentText,
          style: TextStyle(
            color: AppColors.white,
            fontSize: _fontSize,
            height: 2.0,
            fontWeight: FontWeight.w400,
          ),
          textAlign: TextAlign.right,
          textDirection: TextDirection.rtl,
        ),
      ),
    );
  }
}
```

---

## 🚦 Smart Router (Automatically Choose Screen)

```dart
// lib/core/navigation/reading_navigation.dart
class ReadingNavigation {
  static void toReading({
    required String bookId,
    required String chapterId,
  }) async {
    // Fetch chapter to determine type
    final chapterDoc = await FirebaseFirestore.instance
        .collection('books')
        .doc(bookId)
        .collection('chapters')
        .doc(chapterId)
        .get();

    if (!chapterDoc.exists) {
      Get.snackbar(
        'خطأ',
        'الفصل غير موجود',
        backgroundColor: Colors.red,
        colorText: Colors.white,
      );
      return;
    }

    final data = chapterDoc.data()!;
    final hasAudio = data['hasAudio'] as bool? ?? data['audioLink'] != null;

    if (hasAudio) {
      // Navigate to audio reading screen
      Get.toNamed(
        RouteConstants.readingWithAudio,
        arguments: {
          'bookId': bookId,
          'chapterId': chapterId,
        },
      );
    } else {
      // Navigate to text-only reading screen
      Get.toNamed(
        RouteConstants.textReading,
        arguments: {
          'bookId': bookId,
          'chapterId': chapterId,
        },
      );
    }
  }
}
```

---

## 🎯 Usage Examples

### **Example 1: Book with Audio + Timestamps**

```javascript
// Firestore
books/the_7_habits/chapters/summary
{
  "title": "الملخص",
  "hasAudio": true,
  "audioLink": "https://...",
  "audioLength": "12:03",
  "sentences": [
    { "text": "العاداتُ...", "start": 0, "end": 6 },
    { "text": "تتكوَّن...", "start": 6, "end": 11 }
  ]
}

// Result: Full audio player with active highlighting
```

### **Example 2: Book with Audio but NO Timestamps**

```javascript
// Firestore
books/philosophy_basics/chapters/intro
{
  "title": "مقدمة",
  "hasAudio": true,
  "audioLink": "https://...",
  "audioLength": "8:45",
  "sentences": [
    { "text": "الفلسفة هي..." },  // No start/end
    { "text": "تبحث في..." }      // No start/end
  ]
}

// Result: Audio player works, but NO highlighting
```

### **Example 3: Text-Only Book**

```javascript
// Firestore
books/classic_literature/chapters/chapter_1
{
  "title": "الفصل الأول",
  "hasAudio": false,
  "content": "في بداية القصة، كان هناك رجل..."
  // OR
  "sentences": [
    { "text": "في بداية القصة..." },
    { "text": "كان هناك رجل..." }
  ]
}

// Result: Simple text reader (no audio controls)
```

---

## ✅ Checklist for Safe Implementation

- [x] Make `start` and `end` optional in `SentenceModel`
- [x] Add `hasAudio` field to `ChapterModel`
- [x] Add `hasTimingData` getter to check if highlighting is possible
- [x] Guard all timing-related code with null checks
- [x] Only show audio controls if `hasAudio` is true
- [x] Only attempt highlighting if `hasTimingData` is true
- [x] Make tap-to-seek conditional on timing existence
- [x] Handle both `content` (string) and `sentences` (array) formats
- [x] Create separate text-only reading screen (optional)
- [x] Add smart router to choose correct screen type

---

## 🎨 UI States

### **State 1: Audio + Highlighting (Full Features)**

```
┌─────────────────────────────────┐
│  [Back]  الملخص        [•••]   │
├─────────────────────────────────┤
│                                 │
│  العاداتُ هي الأشياءُ التي     │ ← White
│  تُميّزُنا تتكوَّن العادةُ      │ ← Green (highlighted)
│  عندما تقومُ بنشاطٍ معيّن       │ ← White
│                                 │
├─────────────────────────────────┤
│  [====|===============] 2:35    │ ← Progress bar
│     [⏪] [▶️] [⏩] [1.0x]       │ ← Controls
└─────────────────────────────────┘
```

### **State 2: Audio WITHOUT Highlighting**

```
┌─────────────────────────────────┐
│  [Back]  مقدمة         [•••]   │
├─────────────────────────────────┤
│                                 │
│  الفلسفة هي دراسة الحكمة...    │ ← All white
│  تبحث في الأسئلة الكبرى...     │ ← No highlighting
│                                 │
├─────────────────────────────────┤
│  [====|===============] 3:12    │ ← Progress bar
│     [⏪] [▶️] [⏩] [1.0x]       │ ← Controls
└─────────────────────────────────┘
```

### **State 3: Text-Only (No Audio)**

```
┌─────────────────────────────────┐
│  [Back]  الفصل الأول    [A+] [A-]│
├─────────────────────────────────┤
│                                 │
│  في بداية القصة، كان هناك...   │ ← Selectable
│  عاش في قرية صغيرة...          │ ← text
│                                 │
│  (No audio controls)            │
│                                 │
└─────────────────────────────────┘
```

---

## 🚀 Summary

**Key Changes:**

1. ✅ Made timing fields optional in models
2. ✅ Added `hasAudio` and `hasTimingData` checks
3. ✅ Guarded all timing-dependent code
4. ✅ Screen gracefully handles all 3 scenarios
5. ✅ No crashes if data is missing

**Result:**

- **With audio + timestamps:** Full active highlighting
- **With audio, no timestamps:** Audio playback only
- **No audio:** Text-only reading mode

**Your app now handles books at different stages of content creation!** 🎉
