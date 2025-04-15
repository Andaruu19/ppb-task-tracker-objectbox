# Task Tracker App

**Aplikasi dibuat menggunakan database objectbox**

| Name               | NRP        | Class |
|--------------------|------------|-------|
| Farel Hanif Andaru | 5025221253 | C     |

## Fitur

- CRUD Task
- Penyimpanan data menggunakan objectbox

## Project Sebelumnya
https://github.com/Andaruu19/ppb-task-tracker

## Penjelasan

Aplikasi dibuat menggunakan database yang terpasang di lokal, yaitu **objectbox**.

### Install Dependencies

Sebelum menggunakan Objectbox, perlu diinstall terlebih dahulu dependency nya langsung dari terminal.

```bash
flutter pub add objectbox objectbox_flutter_libs:any
flutter pub add --dev build_runner objectbox_generator:any
```

### Membuat Model

Model sebelumnya sudah ada, ada beberapa bagian dari model yang perlu diperbarui seperti jika sebelumnya menggunakan index list disini akan diganti dengan menggunakan id dan menggunakan beberapa anotasi dari objectbox

```dart
import 'package:objectbox/objectbox.dart';

enum TaskPriority {
  low,
  medium,
  high
}

@Entity()
class Task {
  @Id()
  int id = 0;

  String title;
  bool isCompleted;

  int priorityValue;

  @Transient()
  TaskPriority get priority => TaskPriority.values[priorityValue];

  Task({
    this.id = 0,
    required this.title,
    this.isCompleted = false,
    TaskPriority priority = TaskPriority.medium,
  }) : priorityValue = priority.index;

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'title': title,
      'isCompleted': isCompleted,
      'priority': priorityValue,
    };
  }

  factory Task.fromJson(Map<String, dynamic> json) {
    return Task(
      id: json['id'] ?? 0,
      title: json['title'],
      isCompleted: json['isCompleted'] ?? false,
      priority: TaskPriority.values[json['priority'] ?? 1],
    );
  }

  Task copyWith({
    int? id,
    String? title,
    bool? isCompleted,
    TaskPriority? priority,
  }) {
    return Task(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
      priority: priority ?? this.priority,
    );
  }
}
```

### Membuat Setup File Objectbox

objectbox.dart berperan sebagai database acces layer, kode ini akan menyediakan aplikasi flutter sebelumnya API untuk berinteraksi dengan Objectbox NoSQL. Kode ini mengenkapsulasi agar operasi pada database tidak terlihat pada kode UI, sehingga kode UI dapat lebih fokus ke logika presentasi daripada berisi logika penyimpanan data.

```dart
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';
import 'package:task_tracker/models/task.dart';
import 'objectbox.g.dart';

class ObjectBox {
  late final Store _store;
  
  late final Box<Task> taskBox;

  ObjectBox._create(this._store) {
    taskBox = Box<Task>(_store);
  }

  static Future<ObjectBox> create() async {
    final docsDir = await getApplicationDocumentsDirectory();
    final store = await openStore(directory: p.join(docsDir.path, "task-db"));
    return ObjectBox._create(store);
  }

  List<Task> getAllTasks() {
    return taskBox.getAll();
  }

  int addTask(Task task) {
    return taskBox.put(task);
  }

  List<int> addTasks(List<Task> tasks) {
    return taskBox.putMany(tasks);
  }

  bool deleteTask(int id) {
    return taskBox.remove(id);
  }

  int updateTask(Task task) {
    return taskBox.put(task);
  }
}
```

### Menjalankan Kode Objectbox

Untuk menghasilkan kode binding yang diperlukan untuk menggunakan ObjectBox, jalankan perintah:
```bash
dart run build_runner build
```

Generator ObjectBox akan mencari semua anotasi @Entity di folder lib Anda dan membuat:

sebuah definisi database tunggal di lib/objectbox-model.json, dan kode pendukung di lib/objectbox.g.dart.











