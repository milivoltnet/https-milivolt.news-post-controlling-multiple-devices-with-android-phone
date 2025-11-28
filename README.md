Controlling multiple devices with Android phone
 Applications   


Nowadays, the number of Bluetooth controlled devices has increased tremendously. It is possible to see Android-based control applications everywhere, both in variety and in numbers. From our homes to our workplaces, from entertainment venues to the streets, we see Android applications that make our lives easier.


In this application, we wanted to make an application on how to control some electronic elements and devices.


In our application, there are two different colored led diodes, a buzzer, a DC motor and a 220 volt lamp connected to the relay. These components are connected to our microcontroller. We also connected a Bluetooth module to receive incoming commands. We reused the HC-05 bluetooth module that we used in our previous applications.

 


On the Android phone, we installed the application we wrote in Flutter-Dart language. There are basically two important functions in our Android application. One is the Connect function and the other is the sendData function. Since we got the MAC address, we wrote this MAC address into the connect function.
The sendData function, on the other hand, is a function that takes a string type value. For example, when we write sendData('ledon') in the codes, when the button with this function is pressed, the bluetooth adapter sends this data to the connected bluetooth module on the opposite side. Events on other buttons occur in a similar way.


Below are the Flutter-Dart codes:
import 'dart:async';
//import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bluetooth_serial/flutter_bluetooth_serial.dart';


void main() => runApp(const MyApp());

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  // ignore: library_private_types_in_public_api
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  List<BluetoothDevice> _devices = [];
  late BluetoothConnection connection;
  String adr="00:21:07:00:50:69"; // my bluetooth device MAC Adres

  @override
  void initState() {
    super.initState();
    _loadDevices();
  }

  Future<void> _loadDevices() async {
    List<BluetoothDevice> devices = await FlutterBluetoothSerial.instance.getBondedDevices();

    setState(() {   _devices = devices;    });
  }
//----------------------------
  Future<void> sendData(String data)  async {

    data = data.trim();
    try {
      List<int> list = data.codeUnits;
      Uint8List bytes = Uint8List.fromList(list);
      connection.output.add(bytes);
      await connection.output.allSent;
      if (kDebugMode) {
        // print('Data sent successfully');
      }
    } catch (e) {
      //print(e.toString());
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        home: Scaffold(
          appBar: AppBar(
            title: const Text("MULTI CONTROL with bluetooth"),
          ),
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text("MAC Adress: 00:21:07:00:50:69"),

                ElevatedButton(child:Text("Connect"),onPressed: () {
                  connect(adr);                },),
                SizedBox(height: 30.0,),

             Row(
                 children:[
               ElevatedButton(onPressed:(){sendData("redon");}, child: Text('RedOn')),
               ElevatedButton(onPressed:(){sendData("redoff");}, child: Text('RedOFF')),

             ]),

                const SizedBox(height: 30.0,),

                Row(
                    children:[
                  ElevatedButton(onPressed:() {sendData("greenon");}, child: Text('GreenOn')),
                  ElevatedButton(onPressed:() {sendData("greenoff");}, child: Text('GreenOff')),
                ]),

                const SizedBox(height: 30.0,),

                Row(
                    children:[
                  ElevatedButton(onPressed: () {sendData("buzon");}, child: Text('BuzzerOn')),
                  ElevatedButton(onPressed: () {sendData("buzoff");}, child: Text('BuzzerOff')),
                ]),

                const SizedBox(height: 30.0,),

                Row(
                    children:[
                  ElevatedButton(onPressed: (){sendData("moton");}, child: Text('MotorOn')),
                  ElevatedButton(onPressed: (){sendData("motoff");}, child: Text('MotorOff')),
                ]),

                const SizedBox(height: 30.0,),


                Row(
                    children:[
                  ElevatedButton(onPressed: (){sendData("lampon");}, child: Text('LampOn'),),
                  ElevatedButton(onPressed:(){sendData("lampoff");}, child: Text('LampOff'),),
                ]),
              ],
            ),
          ),
        )

    );
  }

  Future connect(String address) async {
    try {
      connection = await BluetoothConnection.toAddress(address);
      sendData('111');
      //durum="Connected to the device";
      connection.input!.listen((Uint8List data) {
        //Data entry point
        // durum=ascii.decode(data);
      });



    } catch (exception) {
      // durum="Cannot connect, exception occured";
    }
  }

// --------------**************data gonder
//Future send(Uint8List data) async {
//connection.output.add(data);
//await connection.output.allSent;
}
//------------*********** data gonder end


Below are the microcontroller codes:
#include <SoftwareSerial.h>

SoftwareSerial mySerial(10, 11); // RX, TX
String rec_data="5";

void setup() {
  pinMode(7, OUTPUT);  // RED led
   pinMode(6, OUTPUT);  // green led
    pinMode(5, OUTPUT);  // Buzzer
     pinMode(3, OUTPUT);   // --------------> motor
       pinMode(2, OUTPUT);  // 220 volt lamp 
   
   
  Serial.begin(9600);
  mySerial.begin(9600);   // BlueTooth Data baud,set the data rate for the SoftwareSerial port
}

void loop() { // run over and over
 
  if (mySerial.available()) {
    
     rec_data=mySerial.readString();
          
           if(rec_data=="redon"){digitalWrite(7,HIGH);}  // RED LED ON
           if(rec_data=="redoff"){digitalWrite(7,LOW);}    // RED LED OFF
           if(rec_data=="greenon"){digitalWrite(6,HIGH);}   // GREEN LED ON
           if(rec_data=="greenoff"){digitalWrite(6,LOW);}   // GREEN LED OFF

           if(rec_data=="buzon"){tone(5,260);}   //BUZZER ON
           if(rec_data=="buzoff"){ noTone(5);}    // BUZZER OFF

           if(rec_data=="moton"){analogWrite(3, 100);}  // MOTOR ON
           if(rec_data=="motoff"){analogWrite(3, 10);}    // MOTOR OFF    
           
           if(rec_data=="lampon"){digitalWrite(2,HIGH);}   // 220 volt lamp relay on
           if(rec_data=="lampoff"){digitalWrite(2,LOW);}   // 220 volt lamp relay off

     delay(500);
       
   Serial.println(rec_data);
    
  }
  
}


Other details: https://milivolt.news/post/controlling-multiple-devices-with-android-phone
