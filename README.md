# firebase_flutter

Flutter Firebase NoSQL : register

กฤษณพงศ์ พงศ์มณเฑียร 62020471

หน้าจอหลัก
![16 10 2021_23 09 56_REC](https://user-images.githubusercontent.com/92374367/137594551-802bb310-d7e1-419b-8f1f-e0cb64466fc3.png)



- หน้าregister
- แสดงหน้าสร้างบัญชีผู้ใช้
![16 10 2021_23 03 10_REC](https://user-images.githubusercontent.com/92374367/137594369-dcb33868-235e-43da-a248-ea2b07a3cba6.png)


### Sold Out

[![245090266-1030718941097166-3275938168215394-n.jpg](https://i.postimg.cc/RZ635kLD/245090266-1030718941097166-3275938168215394-n.jpg)](https://postimg.cc/LJS4j07t)

- ครัวซองขายหมดแล้ว
- จะแสดง ' Sold Out Croissants '

### Cloud Firestore

[![245851782-896813327610221-2297258345555869679-n.png](https://i.postimg.cc/BvtcL7sW/245851782-896813327610221-2297258345555869679-n.png)](https://postimg.cc/LqKZGTnv)

- เก็บ อัพเดท แก้ไข ลบ Data ใน Firebase
- เมนูครัวซองมี รหัส ชื่อ ราคา ของสินค้า

### Model

```dart
//ใช้เพื่อกำหนดเค้าโครงข้อมูลของสินค้า ซึ่งใช้เก็บข้อมูลที่ได้มาจากฐานข้อมูล และถูกนำมาใช้ในการแสดงผล
class ProductModel {
  String id; //เก็บรหัสสินค้า
  String croissant; //เก็บชื่อสินค้า คือ ชื่อเมนูครัวซองแต่ละประเภท
  int price; //ราคาสินค้า
  ProductModel({
    required this.id,
    required this.croissant,
    required this.price,
  });
  factory ProductModel.fromMap(Map<String, dynamic>? product) {
    //factory นำหน้า แสดงว่าภายใน constructor จะต้อง return ค่ากลับมา เป็น object ของ class ProductModel
    //ใส่ ? เเทนการเขียนโค้ด if (product == null) {return null;}
    String id = product?['id'];
    //ข้อมูล id เก็บค่าที่ได้มาจากฟิลด์ id ของฐานข้อมูล
    String croissant = product?['croissant'];
    //ข้อมูล croissant เก็บค่าที่ได้มาจากฟิลด์ croissant ของฐานข้อมูล
    int price = product?['price'];
    //ข้อมูล price เก็บค่าที่ได้มาจากฟิลด์ price ของฐานข้อมูล
    return ProductModel(id: id, croissant: croissant, price: price);
  }
  Map<String, dynamic> toMap() {
    //สร้างเพื่อเเปลง property ภายใน object ให้เป็น Map<String, dynamic> ซึ่งเป็นประเภทข้อมูลที่เหมาะสำหรับ Cloud Firestore
    return {
      'id': id,
      'croissant': croissant,
      'price': price,
    };
  }
}
```

### Service

```dart
//ใช้เพื่อติดต่อและจัดการข้อมูลไปยังฐานข้อมูลไปยังฐานข้อมูล Cloud Firestore
//การอัพเดทข้อมูลสินค้า
class Database {
  static Database instance = Database._();
  Database._();
  Stream<List<ProductModel>> getAllProductStream() {
    //เลือกใช้ stream เพื่อดูข้อมูลได้ แบบ real time
    print('getall');
    final reference = FirebaseFirestore.instance.collection('Croissants'); //หัวข้อหลัก
    final query = reference.orderBy('price',
        descending: false); //เป็นตัวที่เอาไว้เรียงลำดับ จากน้อยไปมาก
    final snapshots = query.snapshots();
    //QuerySnapshot<Map<String, dynamic>> snapshot
    //QuerySnapshot<Object?> snapshot
    return snapshots.map((snapshot) {
      return snapshot.docs.map((doc) {
        return ProductModel.fromMap(doc.data());
      }).toList();
    });
  }
}
```

### Screan

```dart
class ProductLists extends StatelessWidget {
  const ProductLists({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    Database db = Database.instance;
    Stream<List<ProductModel>> stream = db.getAllProductStream();
    return Container(
      padding: EdgeInsets.only(top: 10),
      child: StreamBuilder<List<ProductModel>>(
        stream: stream,
        builder: (context, snapshot) {
          if (snapshot.hasData) {
            //เช็คว่ามีข้อมูลไหมใน firebase ไหม
            if (snapshot.data?.length == 0) {
              //มีสมาชิกข้างใน firebase ไหม
              return Center(
                child: Text('Sold Out Croissants'), //แสดงเมื่อ ไม่มี
              );
            }
            return ListView.builder(
              itemCount: snapshot.data?.length,
              itemBuilder: (context, index) {
                return ProductItem(
                  product: snapshot.data![index],
                );
              },
            );
          }
          return Center(child: CircularProgressIndicator());
        },
      ),
    );
  }
}
```

