# firebase_flutter

Flutter Firebase NoSQL : register

กฤษณพงศ์ พงศ์มณเฑียร 62020471

หน้าจอregister

![16 10 2021_23 13 37_REC](https://user-images.githubusercontent.com/92374367/137594635-2ce65266-d3ff-45c8-ba8d-98b059fa6ee9.png)

หน้าการสร้างบัญชีผู้ใช้

![16 10 2021_23 16 19_REC](https://user-images.githubusercontent.com/92374367/137594721-d8c43c08-f376-4df7-bb8d-d80d12c28d8b.png)

เป็นการสร้างบัญชีผู้ใช้ในFirebase เมื่อผู้ใช้กดปุ่มลงทะเบียน กรอกe-mail,password เมื่อผู้ใช้กดลงทะเทียน ข้อมูลe-mail,password จะเก็บลงในobject profile นำข้อมูลในobject profile 
เข้าไปทำงานในFirebase ให้ทำการสร้างบัญชีผู้ใช้ขึ้นมาตาม e-mail,password ที่กรอกไว้

import 'package:firebase_auth/firebase_auth.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'package:fluttertoast/fluttertoast.dart';
import 'package:form_field_validator/form_field_validator.dart';
import 'package:loginsystem/model/profile.dart';

import 'home.dart';

class RegisterScreen extends StatefulWidget {
  @override
  _RegisterScreenState createState() => _RegisterScreenState();
}

class _RegisterScreenState extends State<RegisterScreen> {
  final formKey = GlobalKey<FormState>();
  Profile profile = Profile();
  final Future<FirebaseApp> firebase = Firebase.initializeApp();

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
        future: firebase,
        builder: (context, snapshot) {
          if(snapshot.hasError){
            return Scaffold(
                appBar: AppBar(
                  title: Text("Error"),
                  ),
                body: Center(child: Text("${snapshot.error}"),
                ),
            );
          }
          if (snapshot.connectionState == ConnectionState.done) {
            return Scaffold(
              appBar: AppBar(
                title: Text("สร้างบัญชีผู้ใช้"),
              ),
              body: Container(
                child: Padding(
                  padding: const EdgeInsets.all(20.0),
                  child: Form(
                    key: formKey,
                    child: SingleChildScrollView(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text("อีเมล", style: TextStyle(fontSize: 20)),
                          TextFormField(
                            validator: MultiValidator([
                              RequiredValidator(errorText: "กรุณาป้อนอีเมลด้วยครับ"),
                              EmailValidator(errorText: "รูปแบบอีเมลไม่ถูกต้อง")
                            ]),
                            keyboardType: TextInputType.emailAddress,
                            onSaved: (String email) {
                              profile.email = email;
                            },
                          ),
                          SizedBox(
                            height: 15,
                          ),
                          Text("รหัสผ่าน", style: TextStyle(fontSize: 20)),
                          TextFormField(
                            validator: RequiredValidator(errorText: "กรุณาป้อนรหัสผ่านด้วยครับ"),
                              obscureText: true,
                              onSaved: (String password) {
                              profile.password = password;
                            },
                          ),
                          SizedBox(
                            width: double.infinity,
                            child: ElevatedButton(
                              child: Text("ลงทะเบียน",style: TextStyle(fontSize: 20)),
                              onPressed: () async{
                                if (formKey.currentState.validate()) {
                                  formKey.currentState.save();
                                  try{
                                    await FirebaseAuth.instance.createUserWithEmailAndPassword(
                                      email: profile.email, 
                                      password: profile.password
                                    ).then((value){
                                      formKey.currentState.reset();
                                      Fluttertoast.showToast(
                                        msg: "สร้างบัญชีผู้ใช้เรียบร้อยแล้ว",
                                        gravity: ToastGravity.TOP
                                      );
                                      Navigator.pushReplacement(context,
                                      MaterialPageRoute(builder: (context){
                                          return HomeScreen();
                                      }));
                                    });
                                  }on FirebaseAuthException catch(e){
                                      print(e.code);
                                      String message;
                                      if(e.code == 'email-already-in-use'){
                                          message = "มีอีเมลนี้ในระบบแล้วครับ โปรดใช้อีเมลอื่นแทน";
                                      }else if(e.code == 'weak-password'){
                                          message = "รหัสผ่านต้องมีความยาว 6 ตัวอักษรขึ้นไป";
                                      }else{
                                          message = e.message;
                                      }
                                      Fluttertoast.showToast(
                                        msg: message,
                                        gravity: ToastGravity.CENTER
                                      );
                                  }
                                }
                              },
                            ),
                          )
                        ],
                      ),
                    ),
                  ),
                ),
              ),
            );
          }
          return Scaffold(
            body: Center(
              child: CircularProgressIndicator(),
            ),
          );
        });
  }
}
