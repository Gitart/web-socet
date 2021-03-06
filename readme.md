# Пример работы с  WEB Socket

Пример работы сокетов с использованием каналов.

1. В канал веб сокетов можно послать сообщение на сервере и на клиенте.
2. Получить сообщения  можно как на клиенте так и на сервере

В примере приведен пример с посылкой сообщения со стороны сервера,
а получение как на стороне сервера так и настороне клиента (файл html)

## Server

### Library
```golang
package main

import (
	"code.google.com/p/go.net/websocket"
	"fmt"
	"log"
	"net/http"
	"time"
)
```

### Объявление канала глобального
```golang
var Mss = make(chan string,100)
```

### Routing 
```golang
func main() {
 
    // Объявление сокетов 
    // Здесь ловим канал в скоетах
    // Сокет для прослушивания канала в HTML 
    http.Handle("/ehs",          websocket.Handler(echoHandChan))    

    // Имитация отправки в канал 
    http.HandleFunc("/in/",       echoHandChans_inp)          // Отправка в канал сообщения на стороне сервера
    http.HandleFunc("/out/",      echoHandChans_read)         // Чтение канала

    fmt.Println("Server start 4444 port...")
    err := http.ListenAndServe(":4444", nil)

    if err != nil {
	log.Println("ListenAndServe: " + err.Error())
    }
}
```

### WEB Socket
Пример создания нового соединения
Новости получают из этого сообщения

```golang

func echoHandChan(ws *websocket.Conn) {
fmt.Println("Channel Open ")   
    	
 for{
     res:=<-Mss  
     ws.Write([]byte(res)	)
   }
}
```

### Посылка сообщения в канала на сервере
```golang

// *******************************************************************
// Запись в канал сообщения
// http://localhost:4444/in/Пример сообщения посылаемого в канал
// *******************************************************************
func echoHandChans_inp(w http.ResponseWriter, r *http.Request){
   pt:= r.URL.Path[len("/in/"):]
   
   Ii++
   I:=fmt.Sprintf("%v",Ii)

   // Отправка в канал сообщения
   go func (){
       Mss  <- I + " " + pt
    }()

   
    log.Println("Сообщение отпарвлено в канал ", I + " " + pt)
}
```

### Чтение сообщений из канала на сервере
```golang
// *******************************************************************
// Чтение канала сообщения
// http://localhost:4444/out/
// *******************************************************************
// Чтение из канала
func echoHandChans_read(w http.ResponseWriter, r *http.Request){
   for{
   select {
     case res:=<-Mss : 
     	fmt.Println(res)
     // default : 
     // 	fmt.Println("No")
   }
}
}
```

### Еще один пример посылки сообщений в канал на стороне сервера

```golang
// Новости можно еще больше переключать
// Пример создания нового соединения
// Новости получают из этого сообщения
func echoHandler(ws *websocket.Conn) {
       fmt.Println("Ok")

	msg    := make([]byte, 512)
	n, err := ws.Read(msg)

	if err != nil {
   	   log.Fatal(err)
	}

	fmt.Printf("Receive: %s\n", msg[:n])
	m, err := ws.Write(msg[:n])

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Send: %s\n", msg[:m])
}
```

## Html файл

```html
<!DOCTYPE HTML>
<html>
<head>
      <script src="https://code.jquery.com/jquery-1.12.4.js"></script>
      <script src="https://code.jquery.com/ui/1.12.1/jquery-ui.js"></script>
      <link rel="stylesheet" href="https://getbootstrap.com/docs/4.2/dist/css/bootstrap.min.css" >
      <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/js/bootstrap.min.js" ></script>
      <link href="https://fonts.googleapis.com/css?family=Open+Sans&display=swap" rel="stylesheet">
      <script src="notify.js"></script>
      <link rel="stylesheet" href="main.css" >

      <style type="text/css">
         .soob      {color:#CCC;}
         .soobnew   {color:#03A9F4;}
         .soobclose {color:#EE3006;}
         .soobopen  {color:#E74C3C;}
         .bul       {font-size: 50px; color: #CCC;}
         .buln      {font-size: 50px; color: green;}
      </style> 


<script type="text/javascript">
function WbS(){
               var ws = new WebSocket("ws://localhost:4444/ehs");

               // Open socket
               ws.onopen = function(){
                  $("#soob").html("Cоединение открылось");
                  $("#soob").switchClass("soob", "soobopen", 3000 );
                  console.log("open connection...");
               };

               // Get message
               ws.onmessage = function (evt){
                  var msg = evt.data;
                  $.notify("Сервер : "+ msg ,"minfo");
                   $("#soob").html(msg);
                  console.log("-->" + msg);
               };

               // websocket is closed.
               ws.onclose = function(){ 
                  $( "#soob" ).switchClass("soobopen", "soobclose", 1000).delay(1000);   //.hide(2000);
                  $.notify("Message is closed...");
                  $("#soob").html("Cоединение закрылось.");
               };
      }
</script>
	
</head>
<body onload="WbS()">
   <div class="container">
      <h3>Работа WEB Socket с каналом ... </h3>
      <h1 id="soob" class="soob"> Канал закрыт ... </h1>
   </div>
</body>
</html> 
```


### Testing with Curl
```sh
curl --location --request GET http://localhost:4444/in/Start message to chanel
sleep 5
curl --location --request GET http://localhost:4444/in/Second message to chanel
sleep 15
curl --location --request GET http://localhost:4444/in/Tree message to chanel
sleep 5
curl --location --request GET http://localhost:4444/in/And also message to chanel
sleep 15
curl --location --request GET http://localhost:4444/in/Five message to chanel
sleep 25
curl --location --request GET http://localhost:4444/in/Six message to chanel
sleep 35
curl --location --request GET http://localhost:4444/in/Seven message to chanel
```






