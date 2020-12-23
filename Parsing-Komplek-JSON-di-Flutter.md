JSON structure #1 : Simple map
---
```dart
{
  "id":"487349",
  "name":"Indra Gunawan",
  "score" : 1000
}
```
Aturan#1 : 
Identifikasi strukturnya. String Json akan memiliki Map (pasangan key-value) atau List of Maps.

Aturan#2: 
  1. Diawali dengan kurung kurawal? Ini Map.  
  2. Diawali dengan tanda kurung siku? Itulah List of Maps.

Kita coba membuat struktur model student
```java
class Student{
  String studentId;
  String studentName;
  int studentScores;

  Student({
    this.studentId,
    this.studentName,
    this.studentScores
  });
}
````

Kita harus melakukan pekerjaan memetakan anggota kelas ini ke objek json.  Untuk itu perlu dibuat suatu factory metode. Menurut dokumentasi Dart, kami menggunakan factorykata kunci saat mengimplementasikan konstruktor yang tidak selalu membuat instance baru dari kelasnya.
````dart
factory Student.fromJson(Map<String, dynamic> parsedJson){
    return Student(
      studentId: parsedJson['id'],
      studentName : parsedJson['name'],
      studentScores : parsedJson ['score']
    );
  }
````

> *Serialization simply means writing the data(which might be in an object) as a string, and Deserialization is the opposite of that. It takes the raw data and reconstructs the object model.*

Di bagian pertama ini, kami melakukan deserialisasi string json dari student.json

Perhatikan juga parameter dalam fromJsonmetode. Ini Map<String, dynamic>berarti memetakan String kunci dengan dynamic nilai. Jika json strukturnya List of maps, maka parameter juga akan berbeda.

##### Tapi kenapa dinamis?
Mari kita lihat struktur json lain terlebih dahulu untuk menjawab pertanyaan Anda.
````dart
{
"name" : "Indra"
"majors" : ["cs", "math"],
"subjects" : [
	{
		"subjectName" : "math",
		"teacher" : "Ms S"
	},
	{
		"subjectName" : "science",
		"teacher" : "Ms p"
	},
]
}
````
name adalah Map <String, String>, majors adalah Map of String dan List <String> dan subjects merupakan Map of String dan List <Object>
Karena kuncinya selalu a string dan nilainya dapat berupa jenis apa pun, kami menyimpannya agar dynamic aman.

###Accessing the object
Mari tulis student_services.dart yang mana akan memiliki kode untuk dipanggil Student.fromJson dan mengambil nilai dari objek Student.
#### Snippet #1 : imports

````dart
import 'dart:async' show Future;
import 'package:flutter/services.dart' show rootBundle;
import 'dart:convert';
import 'package:flutter_json/student_model.dart';
````
#### Snippet #2 : load Json Asset (optional)
````dart
Future<String> _loadAStudentAsset() async {
  return await rootBundle.loadString('assets/student.json');
}
````
#### Snippet #3 : load the response
````dart
Future loadStudent() async {
  String jsonString = await _loadAStudentAsset();
  final jsonResponse = json.decode(jsonString);
  Student student = new Student.fromJson(jsonResponse);
  print(student.studentScores);
}
````
Dalam loadStudent()metode ini ,
Baris 2 : memuat String json mentah dari aset.
Baris 3 : Mendecode String json mentah yang kita dapatkan.
Baris 4 : Dan sekarang kita deserialisasi respon json yang didekode dengan memanggil Student.fromJsonmetode tersebut sehingga kita sekarang dapat menggunakan objek Student untuk mengakses entitas kita.
Baris 5 : Seperti yang kami lakukan di sini, di mana kami mencetak studentScores dari kelas Student.

JSON structure #2 : Simple structure with arrays
---
Sekarang kita menaklukkan struktur json yang mirip dengan yang di atas, tetapi alih-alih hanya nilai tunggal, mungkin juga memiliki array.
````dart
{
  "city": "Mumbai",
  "streets": [
    "address1",
    "address2"
  ]
}
````
Jadi dalam data address.json, kita mempunyai entitas city yang memiliki nilai String sederhana, tetapi streets adalah Array String. Sejauh ini yang saya tahu dart tidak mempunyai tipe data array, tetapi memiliki List`<datatype>` jadi disini streets akan menjadi List`<String>`.

Sekarang kita harus memeriksa Aturan # 1 dan Aturan # 2 . Ini jelas merupakan Map karena ini dimulai dengan penjepit keriting. streets masih sebuah List. Jadi address_model.dartawalnya akan terlihat seperti ini :
````dart
class Address {
  final String city;
  final List<String> streets;

  Address({
    this.city,
    this.streets
  });
}
````
Jadi method Address.fromJson akan terlihat memiliki parameter Map<String, dynamic>
````dart
factory Address.fromJson(Map<String, dynamic> parsedJson) {
  return new Address(
      city: parsedJson['city'],
      streets: parsedJson['streets'],
  );
}
````
Sekarang buat address_services.dart dengan menambahkan 3snipet yang kami sebutkan di atas. Harus ingat untuk memasukkan nama file dan nama metode yang tepat. Sekarang ketika Anda menjalankan ini, Anda akan mendapatkan kesalahan kecil yang bagus. :
```dart
type 'List<dynamic>' is not a subtype of type 'List<String>'
````
We are requesting a List`<String>` but we are getting a List`<dynamic>` because our application cannot identify the type yet.
Error ini terjadi karna request List`<String>` tetapi kita mendapatkan List `<dynamic>` karna aplikasi tidak mengindentifikasi type data. Sehingga kita harus mengkonversi List<String>
````dart
var streetsFromJson = parsedJson['streets'];
List<String> streetsList = new List<String>.from(streetsFromJson);
````
Sekarang petakan variabel streetsFromJson ke entitas streets . streetsFromJson masih sebuah List`<dynamic>`. Sekarang kami secara eksplisit membuat baru List`<String>` streetsList yang berisi semua elemen dari streetsFromJson.
hasil akhirnya adalah 
````dart
class Address {
  final String city;
  final List<String> streets;

  Address({
    this.city,
    this.streets
  });

  factory Address.fromJson(Map<String, dynamic> parsedJson) {
    var streetsFromJson  = parsedJson['streets'];
    //print(streetsFromJson.runtimeType);
    // List<String> streetsList = new List<String>.from(streetsFromJson);
    List<String> streetsList = streetsFromJson.cast<String>();

    return new Address(
      city: parsedJson['city'],
      streets: streetsList,
    );
  }

}
````

sumber : https://medium.com/flutter-community/parsing-complex-json-in-flutter-747c46655f51
