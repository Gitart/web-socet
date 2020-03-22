# Пример работы с  WEB Socket

## Server


## Library
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


Пример создания нововго соединения
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
