dependencies:
  flutter:
    sdk: flutter

  get: ^4.6.6
  http: ^1.2.0
  shared_preferences: ^2.2.2



  # Flutter GetX JWT Login + CRUD (Full Quick Exam Template)

## 📁 Folder Structure

```id="fs1"
lib/
│
├── main.dart
│
├── models/
│   └── user_model.dart
│
├── controllers/
│   ├── auth_controller.dart
│   └── user_controller.dart
│
├── services/
│   └── api_service.dart
│
├── storage/
│   └── storage_service.dart
│
└── views/
    ├── login_page.dart
    ├── home_page.dart
    ├── add_user_page.dart
    └── edit_user_page.dart
```

---

# main.dart

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'views/login_page.dart';

void main(){
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context){
    return GetMaterialApp(
      debugShowCheckedModeBanner:false,
      home: LoginPage(),
    );
  }
}
```

---

# models/user_model.dart

```dart
class UserModel{

  int? id;
  String? name;
  String? email;

  UserModel({this.id,this.name,this.email});

  factory UserModel.fromJson(Map<String,dynamic> json){
    return UserModel(
      id: json['id'],
      name: json['name'],
      email: json['email']
    );
  }

  Map<String,dynamic> toJson(){
    return {
      "name":name,
      "email":email
    };
  }
}
```

---

# storage/storage_service.dart

```dart
import 'package:shared_preferences/shared_preferences.dart';

class StorageService{

  static Future saveToken(String token) async{
    final prefs = await SharedPreferences.getInstance();
    prefs.setString("token", token);
  }

  static Future<String?> getToken() async{
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString("token");
  }

}
```

---

# services/api_service.dart

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiService{

  static const baseUrl="https://api.example.com";

  static Future get(String endpoint,String token) async{
    var res = await http.get(
      Uri.parse("$baseUrl/$endpoint"),
      headers:{"Authorization":"Bearer $token"}
    );
    return jsonDecode(res.body);
  }

  static Future post(String endpoint,Map data,String token) async{
    var res = await http.post(
      Uri.parse("$baseUrl/$endpoint"),
      headers:{
        "Authorization":"Bearer $token",
        "Content-Type":"application/json"
      },
      body: jsonEncode(data)
    );
    return jsonDecode(res.body);
  }

  static Future put(String endpoint,Map data,String token) async{
    var res = await http.put(
      Uri.parse("$baseUrl/$endpoint"),
      headers:{
        "Authorization":"Bearer $token",
        "Content-Type":"application/json"
      },
      body: jsonEncode(data)
    );
    return jsonDecode(res.body);
  }

  static Future delete(String endpoint,String token) async{
    var res = await http.delete(
      Uri.parse("$baseUrl/$endpoint"),
      headers:{"Authorization":"Bearer $token"}
    );
    return jsonDecode(res.body);
  }

}
```

---

# controllers/auth_controller.dart (Login)

```dart
import 'package:get/get.dart';
import '../services/api_service.dart';
import '../storage/storage_service.dart';
import '../views/home_page.dart';

class AuthController extends GetxController{

  Future login(String email,String password) async{

    var data={
      "email":email,
      "password":password
    };

    var res = await ApiService.post("login",data,"");

    if(res['token']!=null){

      await StorageService.saveToken(res['token']);

      Get.offAll(HomePage());

    }else{
      Get.snackbar("Error","Login Failed");
    }

  }

}
```

---

# controllers/user_controller.dart (CRUD)

```dart
import 'package:get/get.dart';
import '../models/user_model.dart';
import '../services/api_service.dart';
import '../storage/storage_service.dart';

class UserController extends GetxController{

  var users=<UserModel>[].obs;

  @override
  void onInit(){
    getUsers();
    super.onInit();
  }

  Future getUsers() async{

    String? token=await StorageService.getToken();

    var res = await ApiService.get("users",token!);

    users.value = List<UserModel>.from(
      res.map((x)=>UserModel.fromJson(x))
    );
  }

  Future addUser(UserModel user) async{

    String? token=await StorageService.getToken();

    await ApiService.post("users",user.toJson(),token!);

    getUsers();
  }

  Future updateUser(int id,UserModel user) async{

    String? token=await StorageService.getToken();

    await ApiService.put("users/$id",user.toJson(),token!);

    getUsers();
  }

  Future deleteUser(int id) async{

    String? token=await StorageService.getToken();

    await ApiService.delete("users/$id",token!);

    getUsers();
  }

}
```

---

# views/login_page.dart

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import '../controllers/auth_controller.dart';

class LoginPage extends StatelessWidget{

  final AuthCattontroller controller = Get.put(AuthController());

  final email = TextEditingController();
  final password = TextEditingController();

  @override
  Widget build(BuildContext context){

    return Scaffold(
      appBar: AppBar(title: Text("Login")),
      body: Column(
        children: [

          TextField(controller: email,decoration:InputDecoration(labelText:"Email")),
          TextField(controller: password,decoration:InputDecoration(labelText:"Password")),

          ElevatedButton(
            onPressed: (){
              controller.login(email.text,password.text);
            },
            child: Text("Login"),
          )

        ],
      ),
    );
  }
}
```

---

# views/home_page.dart (List + Delete)

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import '../controllers/user_controller.dart';
import 'add_user_page.dart';
import 'edit_user_page.dart';

class HomePage extends StatelessWidget{

  final UserController controller = Get.put(UserController());

  @override
  Widget build(BuildContext context){

    return Scaffold(
      appBar: AppBar(title: Text("Users")),

      body: Obx(()=>ListView.builder(
        itemCount: controller.users.length,
        itemBuilder:(context,index){

          var user = controller.users[index];

          return ListTile(
            title: Text(user.name ?? ""),
            subtitle: Text(user.email ?? ""),

            trailing: Row(
              mainAxisSize: MainAxisSize.min,
              children: [

                IconButton(
                  icon: Icon(Icons.edit),
                  onPressed: (){
                    Get.to(EditUserPage(user:user));
                  },
                ),

                IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: (){
                    controller.deleteUser(user.id!);
                  },
                )

              ],
            ),
          );
        }
      )),

      floatingActionButton: FloatingActionButton(
        onPressed: (){
          Get.to(AddUserPage());
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

---

# views/add_user_page.dart (Create)

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import '../controllers/user_controller.dart';
import '../models/user_model.dart';

class AddUserPage extends StatelessWidget{

  final UserController controller = Get.find();

  final name=TextEditingController();
  final email=TextEditingController();

  @override
  Widget build(BuildContext context){

    return Scaffold(
      appBar: AppBar(title: Text("Add User")),

      body: Column(
        children: [

          TextField(controller:name,decoration:InputDecoration(labelText:"Name")),
          TextField(controller:email,decoration:InputDecoration(labelText:"Email")),

          ElevatedButton(
            onPressed: (){
              controller.addUser(
                UserModel(name:name.text,email:email.text)
              );
              Get.back();
            },
            child: Text("Save"),
          )

        ],
      ),
    );
  }
}
```

---

# views/edit_user_page.dart (Update)

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import '../controllers/user_controller.dart';
import '../models/user_model.dart';

class EditUserPage extends StatelessWidget{

  final UserModel user;

  EditUserPage({required this.user});

  final UserController controller = Get.find();

  final name=TextEditingController();
  final email=TextEditingController();

  @override
  Widget build(BuildContext context){

    name.text=user.name ?? "";
    email.text=user.email ?? "";

    return Scaffold(
      appBar: AppBar(title: Text("Edit User")),

      body: Column(
        children: [

          TextField(controller:name),
          TextField(controller:email),

          ElevatedButton(
            onPressed: (){
              controller.updateUser(
                user.id!,
                UserModel(name:name.text,email:email.text)
              );
              Get.back();
            },
            child: Text("Update"),
          )

        ],
      ),
    );
  }
}
```

---

# API Endpoints

```id="apiend"
POST   /login
GET    /users
POST   /users
PUT    /users/:id
DELETE /users/:id
```

---

# Authorization Header

```id="authheader"
Authorization: Bearer JWT_TOKEN
```

---

# GetX Quick Syntax

```id="getxquick"
var users = [].obs
Obx(()=>Widget())

Get.put(Controller())
Get.find<Controller>()

Get.to(Page())
Get.back()
Get.offAll(Page())
```
