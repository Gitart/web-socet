# Пример работы с  WEB Socket

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
    http.Handle("/echo",         websocket.Handler(echoHandler))
    http.Handle("/eh",           websocket.Handler(echoHand))

    // Имитация отправки в канал 
    // http.HandleFunc("/chan/",  echoHandChans)
    http.HandleFunc("/in/",      echoHandChans_inp)          // Отправка в канал сообщения на стороне сервера
    http.HandleFunc("/out/",     echoHandChans_read)         // Чтение канала

    // Здесь ловим канал в скоетах
    http.Handle("/ehs",        websocket.Handler(echoHandChan))    // Сокет для прослушивания канала в HTML 

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

## Html айл

```html
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


```
