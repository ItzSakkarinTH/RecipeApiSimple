# Flutter Recipe App 🍳

แอปพลิเคชัน Flutter สำหรับแสดงรายการสูตรอาหารจาก DummyJSON Recipes API ด้วยการออกแบบ UI แบบ modern และสวยงาม

## หลักการทำงาน

แอปนี้ใช้ **FutureBuilder** เป็นหัวใจหลักในการจัดการ asynchronous operations สำหรับดึงข้อมูลจาก API และแสดงผลอย่างมีประสิทธิภาพ พร้อมด้วยการออกแบบ UI ที่ทันสมัยด้วยธีมสีเขียว

### การจัดการสถานะ (State Management)

แอปจัดการ 4 สถานะหลัก:
1. **Loading State** - แสดง CircularProgressIndicator ขณะรอข้อมูล
2. **Error State** - แสดงข้อความและไอคอน error เมื่อเกิดข้อผิดพลาด
3. **Empty State** - แสดงข้อความเมื่อไม่มีข้อมูลสูตรอาหาร
4. **Success State** - แสดงรายการสูตรอาหารในรูปแบบ Modern Cards

## อธิบายการทำงานของโค้ด

### 1. Recipe Model
```dart
class Recipe {
  final int id;
  final String name;
  final List<String> ingredients;
  final List<String> instructions;
  final double prepTimeMinutes;
  final double cookTimeMinutes;
  final double servings;
  final String difficulty;
  final String cuisine;
  final int caloriesPerServing;
  final List<String> tags;
  final String image;
  final double rating;
  final double reviewCount;
  final List<String> mealType;

  factory Recipe.fromJson(Map<String, dynamic> json) {
    return Recipe(
      id: json['id'] ?? 0,
      name: json['name'] ?? '',
      // Convert List<dynamic> to List<String>
      ingredients: (json['ingredients'] as List<dynamic>?)
          ?.map((e) => e.toString())
          .toList() ?? [],
      instructions: (json['instructions'] as List<dynamic>?)
          ?.map((e) => e.toString())
          .toList() ?? [],
      prepTimeMinutes: (json['prepTimeMinutes'] ?? 0).toDouble(),
      cookTimeMinutes: (json['cookTimeMinutes'] ?? 0).toDouble(),
      servings: (json['servings'] ?? 0).toDouble(),
      difficulty: json['difficulty'] ?? 'Unknown',
      cuisine: json['cuisine'] ?? 'Unknown',
      caloriesPerServing: json['caloriesPerServing'] ?? 0,
      tags: (json['tags'] as List<dynamic>?)
          ?.map((e) => e.toString())
          .toList() ?? [],
      image: json['image'] ?? '',
      rating: (json['rating'] ?? 0.0).toDouble(),
      reviewCount: (json['reviewCount'] ?? 0).toDouble(),
      mealType: (json['mealType'] as List<dynamic>?)
          ?.map((e) => e.toString())
          .toList() ?? [],
    );
  }
}
```
- สร้างโมเดลข้อมูลสูตรอาหารเพื่อจัดเก็บข้อมูลจาก API
- ใช้ `factory Recipe.fromJson()` แปลงข้อมูล JSON response เป็น Recipe object
- จัดการ **Type Casting** สำหรับ `List<dynamic>` เป็น `List<String>` ด้วย `.map((e) => e.toString()).toList()`
- จัดการกรณี fields เป็น null ด้วย null operator `??` และ null safety

### 2. ProductsService - การเรียกใช้ API
```dart
class ProductsService {
  static const String apiUrl = 'https://dummyjson.com/recipes';

  static Future<List<Recipe>> fetchProducts() async {
    final response = await http.get(Uri.parse(apiUrl));

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      final List<dynamic> recipesJson = data['recipes'];
      return recipesJson.map((json) => Recipe.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load recipes');
    }
  }
}
```

**ขั้นตอนการเรียก API:**
1. **HTTP GET Request** - ใช้ `http.get()` เรียก DummyJSON Recipes API
2. **Status Code Check** - ตรวจสอบ response status (200 = สำเร็จ)
3. **JSON Parsing** - แปลง response body จาก String เป็น Map
4. **Data Extraction** - ดึงข้อมูล recipes array จาก response
5. **Object Mapping** - แปลงแต่ละ recipe เป็น Recipe object
6. **Error Handling** - throw Exception เมื่อ status code ไม่เป็น 200

### 3. FutureBuilder Implementation
```dart
FutureBuilder<List<Recipe>>(
  future: ProductsService.fetchProducts(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return Center(
        child: CircularProgressIndicator(
          strokeWidth: 2,
          valueColor: AlwaysStoppedAnimation<Color>(Colors.blue),
        ),
      ); // Loading state
    }
    
    if (snapshot.hasError) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.error_outline, size: 48, color: Colors.red[300]),
            Text('เกิดข้อผิดพลาด'),
            Text('${snapshot.error}'),
          ],
        ),
      ); // Error state
    }
    
    if (!snapshot.hasData || snapshot.data!.isEmpty) {
      return Center(
        child: Text('ไม่พบข้อมูลสินค้า'),
      ); // Empty state
    }
    
    return ListView.builder(
      padding: EdgeInsets.all(16),
      itemCount: snapshot.data!.length,
      itemBuilder: (context, index) {
        final recipes = snapshot.data![index];
        return RecipeCards(recipes: recipes);
      },
    ); // Success state
  },
)
```

**การทำงานของ FutureBuilder:**
- **future**: กำหนด async function ที่จะเรียกใช้
- **builder**: function ที่สร้าง widget ตาม snapshot state
- **ConnectionState.waiting**: สถานะกำลังรอผลลัพธ์
- **snapshot.hasError**: ตรวจสอบว่าเกิด error หรือไม่
- **snapshot.hasData**: ตรวจสอบว่ามีข้อมูลหรือไม่
- **snapshot.data**: เข้าถึงข้อมูลที่ได้จาก Future

### 4. RecipeCard Widget - Modern Design
```dart
class RecipeCards extends StatelessWidget {
  final Recipe recipes;

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.only(bottom: 20),
      decoration: BoxDecoration(
        borderRadius: BorderRadius.circular(20),
        boxShadow: [
          BoxShadow(
            color: Colors.green.withOpacity(0.08),
            spreadRadius: 0,
            blurRadius: 20,
            offset: const Offset(0, 8),
          ),
        ],
      ),
      child: ClipRRect(
        borderRadius: BorderRadius.circular(20),
        child: Container(
          decoration: BoxDecoration(
            color: Colors.white,
            border: Border.all(
              color: Colors.green.withOpacity(0.1),
              width: 1,
            ),
          ),
          child: Column(
            children: [
              // Hero Image Section
              Stack(/* Image with overlay and badges */),
              // Content Section
              Padding(/* Recipe details and info */),
            ],
          ),
        ),
      ),
    );
  }
}
```

**คุณสมบัติของ Recipe Card:**
- **Modern Shadow** - ใช้ `BoxShadow` สีเขียวอ่อนเพื่อความนุ่มตา
- **Rounded Corners** - `BorderRadius.circular(20)` สำหรับขอบมน
- **Hero Image** - แสดงรูปภาพอาหารพร้อม gradient overlay
- **Badge System** - Rating และ Difficulty badges แยกสีตามประเภท
- **Information Row** - เวลาทำ, จำนวนคน, แคลอรี่
- **Action Button** - ปุ่มดูรายละเอียดแบบ gradient

### 5. การจัดการรูปภาพ
```dart
Image.network(
  recipes.image,
  fit: BoxFit.cover,
  errorBuilder: (context, error, stackTrace) {
    return Container(
      color: Colors.green[50],
      child: Icon(
        Icons.restaurant_menu,
        size: 60,
        color: Colors.green[300],
      ),
    );
  },
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return Container(
      color: Colors.green[50],
      child: Center(
        child: CircularProgressIndicator(
          strokeWidth: 2,
          valueColor: AlwaysStoppedAnimation<Color>(
            Colors.green[400]!,
          ),
        ),
      ),
    );
  },
)
```

**การจัดการ Image States:**
- **Loading**: แสดง CircularProgressIndicator สีเขียวขณะโหลด
- **Success**: แสดงรูปภาพเมื่อโหลดสำเร็จ
- **Error**: แสดงไอคอน restaurant_menu เมื่อโหลดไม่สำเร็จ

### 6. Difficulty Badge System
```dart
Color _getDifficultyColor(String difficulty) {
  switch (difficulty.toLowerCase()) {
    case 'easy':
      return Colors.green[500]!;
    case 'medium':
      return Colors.orange[500]!;
    case 'hard':
      return Colors.red[500]!;
    default:
      return Colors.grey[500]!;
  }
}
```
- ใช้ color coding ตามระดับความยาก
- สีเขียว: Easy, สีส้ม: Medium, สีแดง: Hard

### 7. Gradient Action Button
```dart
Container(
  decoration: BoxDecoration(
    gradient: LinearGradient(
      colors: [
        Colors.green[400]!,
        Colors.green[600]!,
      ],
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
    ),
    borderRadius: BorderRadius.circular(12),
    boxShadow: [
      BoxShadow(
        color: Colors.green.withOpacity(0.3),
        blurRadius: 12,
        offset: const Offset(0, 6),
      ),
    ],
  ),
  child: Material(
    color: Colors.transparent,
    child: InkWell(
      borderRadius: BorderRadius.circular(12),
      onTap: () {
        // Handle recipe tap
      },
      child: // Button content
    ),
  ),
)
```

## Flow การทำงาน

1. **App Launch** → MainApp widget เริ่มต้น MaterialApp
2. **ProductsScreen Init** → สร้าง Scaffold และ FutureBuilder
3. **API Call Trigger** → FutureBuilder เรียก `ProductsService.fetchProducts()`
4. **HTTP Request** → ส่ง GET request ไปยัง `https://dummyjson.com/recipes`
5. **Response Processing** → แปลง JSON response เป็น `List<Recipe>`
6. **Type Casting** → จัดการ `List<dynamic>` เป็น `List<String>`
7. **UI Update** → FutureBuilder rebuild widget ตาม snapshot state
8. **ListView Rendering** → สร้าง RecipeCard สำหรับแต่ละสูตรอาหาร
9. **Image Loading** → โหลดรูปภาพแบบ asynchronous พร้อม fallback

## วิธีการเรียกใช้ API

### API Endpoint
```
GET https://dummyjson.com/recipes
```

### Response Structure
```json
{
  "recipes": [
    {
      "id": 1,
      "name": "Classic Margherita Pizza",
      "ingredients": ["Pizza dough", "Tomato sauce", "Fresh mozzarella cheese", "Fresh basil leaves", "Olive oil", "Salt", "Pepper"],
      "instructions": [
        "Preheat the oven to 475°F (245°C).",
        "Roll out the pizza dough and spread tomato sauce evenly.",
        "Add fresh mozzarella and bake for 10-12 minutes.",
        "Garnish with fresh basil leaves before serving."
      ],
      "prepTimeMinutes": 20,
      "cookTimeMinutes": 15,
      "servings": 4,
      "difficulty": "Easy",
      "cuisine": "Italian",
      "caloriesPerServing": 300,
      "tags": ["Pizza", "Italian"],
      "image": "https://cdn.dummyjson.com/recipe-images/1.webp",
      "rating": 4.6,
      "reviewCount": 98,
      "mealType": ["Dinner"]
    }
  ],
  "total": 50,
  "skip": 0,
  "limit": 30
}
```

### HTTP Client Configuration
```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

// การเรียกใช้
final response = await http.get(Uri.parse(apiUrl));
final data = json.decode(response.body);
```

## ข้อกำหนดระบบ

- Flutter SDK (3.0+)
- HTTP package: `http: ^1.1.0`
- Internet permission (Android/iOS)

## คุณสมบัติเด่น

✅ **Modern UI Design** - Card design ที่ทันสมัยด้วยธีมสีเขียว  
✅ **Asynchronous Data Loading** - FutureBuilder จัดการ API calls  
✅ **Type Safety** - จัดการ List<dynamic> casting อย่างปลอดภัย  
✅ **Error Handling** - จัดการ network และ parsing errors  
✅ **Loading States** - UX indicators สำหรับทุกสถานะ  
✅ **Image Optimization** - lazy loading พร้อม error fallback  
✅ **Responsive Layout** - adaptive design สำหรับหน้าจอต่างๆ  
✅ **Badge System** - แสดง rating และ difficulty แบบ visual  
✅ **Gradient Design** - modern button และ overlay effects  
✅ **Thai Localization** - รองรับข้อความภาษาไทย  

## การจัดการ State

แอปใช้ **Stateless Architecture** ร่วมกับ **FutureBuilder** ซึ่งเหมาะสำหรับ:
- ข้อมูลที่ไม่เปลี่ยนแปลงบ่อย
- การดึงข้อมูลครั้งเดียวเมื่อเริ่มต้น
- UI ที่ไม่ต้องการ complex state management

FutureBuilder จัดการ lifecycle ของ async operations และ rebuild UI อัตโนมัติเมื่อสถานะเปลี่ยนแปลง

## การแก้ปัญหาที่พบบ่อย

### Type Casting Error
```dart
// ❌ ปัญหา: type 'List<dynamic>' is not a subtype of 'List<String>'
ingredients: json['ingredients'],

// ✅ วิธีแก้: Type casting พร้อม null safety
ingredients: (json['ingredients'] as List<dynamic>?)
    ?.map((e) => e.toString())
    .toList() ?? [],
```

### Image Loading Optimization
```dart
// รองรับทั้ง loading, success, และ error states
Image.network(
  imageUrl,
  fit: BoxFit.cover,
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return CircularProgressIndicator();
  },
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.restaurant_menu);
  },
)
```

---

## ตัวอย่างการใช้งาน

การสร้างแอปนี้เหมาะสำหรับผู้ที่ต้องการเรียนรู้:

1. **HTTP API Integration** - การเชื่อมต่อ REST API
2. **JSON Parsing** - การแปลงข้อมูล JSON เป็น Dart Objects
3. **Type Safety** - การจัดการ type casting อย่างปลอดภัย
4. **Async Programming** - FutureBuilder และ async/await patterns
5. **Modern UI Design** - Card-based layout พร้อม animations
6. **Error Handling** - การจัดการ errors ในทุกระดับ
7. **Image Loading** - lazy loading พร้อม fallback mechanisms
8. **State Management** - การใช้ Stateless widgets อย่างมีประสิทธิภาพ

แอปนี้เป็นตัวอย่างที่ดีสำหรับการเรียนรู้ Flutter ในระดับกลาง พร้อมกับการประยุกต์ใช้ best practices ในการพัฒนา mobile applications