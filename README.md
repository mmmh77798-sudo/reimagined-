# reimagined-flutter create --platforms=android .
flutter pub get
flutter run
name: hitham
description: لعبة ألغاز عربية أصلية مبنية باستخدام Flutter.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.3.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0

flutter:
  uses-material-design: true
  import 'dart:math';
import 'package:flutter/material.dart';

void main() {
  runApp(const HithamGameApp());
}

class HithamGameApp extends StatelessWidget {
  const HithamGameApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ألغاز هيثم',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF087F73),
        ),
        scaffoldBackgroundColor: const Color(0xFFF3F8F7),
      ),
      home: const PuzzlePage(),
    );
  }
}

class Puzzle {
  const Puzzle(this.word, this.hint);
  final String word;
  final String hint;
}

class PuzzlePage extends StatefulWidget {
  const PuzzlePage({super.key});

  @override
  State<PuzzlePage> createState() => _PuzzlePageState();
}

class _PuzzlePageState extends State<PuzzlePage> {
  static const puzzles = [
    Puzzle('مدينة', 'مكان كبير يعيش فيه الناس وتنتشر فيه الأسواق.'),
    Puzzle('شجرة', 'لها جذور وأغصان وقد تعطينا الظل والثمار.'),
    Puzzle('كتاب', 'نقرأه لنتعلم ونكتشف أفكارًا جديدة.'),
    Puzzle('قمر', 'جسم مضيء نراه في السماء ليلًا.'),
    Puzzle('نخلة', 'شجرة مشهورة في بلادنا وتنتج التمر.'),
  ];

  final random = Random();
  int level = 0;
  int score = 0;
  List<String> letters = [];
  final selected = <int>[];
  bool solved = false;
  bool finished = false;

  Puzzle get puzzle => puzzles[level];
  String get answer => selected.map((i) => letters[i]).join();

  @override
  void initState() {
    super.initState();
    _shuffleLetters();
  }

  void _shuffleLetters() {
    final shuffled = puzzle.word.split('')..shuffle(random);
    setState(() {
      letters = shuffled;
      selected.clear();
      solved = false;
    });
  }

  void _notify(String text) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(text, textDirection: TextDirection.rtl)),
    );
  }

  void _check() {
    if (selected.length != letters.length) {
      _notify('اختر جميع الحروف أولًا.');
      return;
    }

    if (answer == puzzle.word) {
      setState(() {
        score += 10;
        solved = true;
      });
    } else {
      setState(() {
        score = max(0, score - 2);
        selected.clear();
      });
      _notify('إجابة غير صحيحة، حاول مرة أخرى.');
    }
  }

  void _next() {
    if (level == puzzles.length - 1) {
      setState(() => finished = true);
    } else {
      setState(() => level++);
      _shuffleLetters();
    }
  }

  void _restart() {
    setState(() {
      level = 0;
      score = 0;
      finished = false;
    });
    _shuffleLetters();
  }

  @override
  Widget build(BuildContext context) {
    return Directionality(
      textDirection: TextDirection.rtl,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('ألغاز هيثم'),
          centerTitle: true,
        ),
        body: SafeArea(
          child: finished ? _results() : _game(),
        ),
      ),
    );
  }

  Widget _game() {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(20),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Chip(label: Text('المرحلة ${level + 1} من ${puzzles.length}')),
              Chip(label: Text('النقاط: $score')),
            ],
          ),
          const SizedBox(height: 20),

          Card(
            child: Padding(
              padding: const EdgeInsets.all(22),
              child: Column(
                children: [
                  const Icon(
                    Icons.lightbulb_outline,
                    color: Color(0xFF087F73),
                    size: 32,
                  ),
                  const SizedBox(height: 10),
                  const Text(
                    'فكّر في الإجابة',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 12),
                  Text(
                    puzzle.hint,
                    textAlign: TextAlign.center,
                    style: const TextStyle(fontSize: 19, height: 1.6),
                  ),
                ],
              ),
            ),
          ),

          const SizedBox(height: 22),

          Container(
            constraints: const BoxConstraints(minHeight: 76),
            alignment: Alignment.center,
            padding: const EdgeInsets.all(14),
            decoration: BoxDecoration(
              color: Colors.teal.shade50,
              borderRadius: BorderRadius.circular(18),
              border: Border.all(color: Colors.teal.shade200),
            ),
            child: Text(
              selected.isEmpty ? 'رتّب الحروف لتكوين كلمة' : answer,
              textAlign: TextAlign.center,
              style: const TextStyle(
                fontSize: 30,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),

          const SizedBox(height: 22),

          Wrap(
            alignment: WrapAlignment.center,
            spacing: 10,
            runSpacing: 10,
            children: List.generate(letters.length, (i) {
              final used = selected.contains(i);

              return SizedBox(
                width: 58,
                height: 58,
                child: FilledButton(
                  onPressed: used || solved ? null : () => setState(() => selected.add(i)),
                  style: FilledButton.styleFrom(
                    backgroundColor: Colors.white,
                    foregroundColor: const Color(0xFF087F73),
                    disabledBackgroundColor: Colors.teal.shade100,
                  ),
                  child: Text(
                    letters[i],
                    style: const TextStyle(fontSize: 26, fontWeight: FontWeight.bold),
                  ),
                ),
              );
            }),
          ),

          const SizedBox(height: 20),

          Row(
            children: [
              Expanded(
                child: OutlinedButton.icon(
                  onPressed: selected.isEmpty || solved
                      ? null
                      : () => setState(() => selected.removeLast()),
                  icon: const Icon(Icons.backspace_outlined),
                  label: const Text('تراجع'),
                ),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: FilledButton.icon(
                  onPressed: solved ? null : _check,
                  icon: const Icon(Icons.check_circle_outline),
                  label: const Text('تحقق'),
                ),
              ),
            ],
          ),

          if (solved) ...[
            const SizedBox(height: 14),
            const Text(
              'إجابة صحيحة! +١٠ نقاط',
              textAlign: TextAlign.center,
              style: TextStyle(
                color: Colors.green,
                fontWeight: FontWeight.bold,
                fontSize: 18,
              ),
            ),
            const SizedBox(height: 10),
            FilledButton(
              onPressed: _next,
              child: Text(
                level == puzzles.length - 1 ? 'عرض النتيجة' : 'المرحلة التالية',
              ),
            ),
          ],
        ],
      ),
    );
  }

  Widget _results() {
    return Center(
      child: Card(
        margin: const EdgeInsets.all(24),
        child: Padding(
          padding: const EdgeInsets.all(28),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              const Icon(Icons.emoji_events, size: 72, color: Colors.amber),
              const SizedBox(height: 12),
              const Text(
                'أحسنت!',
                style: TextStyle(fontSize: 30, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 8),
              Text(
                'نتيجتك النهائية: $score نقطة',
                style: const TextStyle(fontSize: 20),
              ),
              const SizedBox(height: 20),
              FilledButton.icon(
                onPressed: _restart,
                icon: const Icon(Icons.restart_alt),
                label: const Text('العب مرة أخرى'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
