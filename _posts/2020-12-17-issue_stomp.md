---
layout: post
title:  react-native에서 Stomp Client(SockJS 없이) 사용하기
date:   2020-12-17 13:00:00 +0800
categories: issue
tag: socket
sitemap :
  changefreq : daily
  priority : 1.0
---

## 1. Stomp.over(() => new SockJS(url)) 을 수정하고 싶다.

많은 예제 코드에서 클라이언트 코드에 SockJS를 활용하고, 서버(스프링)에도 withSockJS를 활용했습니다.

저는 SockJS가 일반 웹소켓과 뭐가 다른지 궁금해하고 있었습니다. 저는 처음 SockJS를 클라이언트 코드에서 봤을 때 WebSocket 클라이언트의 한 방식이라고 생각했고, 그래서 서버 코드와는 관련이 없을 것이라고 생각했습니다. 하지만 서버 코드에 withSockJS를 붙여야 했고, withSockJS를 붙였더니 [크롬의 웹소켓 클라이언트](https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo?hl=ko)를 썼을 때 연결이 안됐습니다. 그래서 확실히 SockJS가 일반적인 WebSocket Client와 다르다는 것을 알게 됐습니다(아직도 SockJS에 대해서 깊게는 알지 못하지만요).

그러던 중 클라이언트에서 Stomp Client를 테스트하기 위해 목 웹소켓 서버를 사용했을 때입니다. 처음에 목 서버와 연결이 잘 안됐는데, 아마 Stomp Client 내부에는 SockJS를 쓰고있다보니 그런 것 같았습니다. 그래서 그냥 WebSocket 클라이언트(SockJS가 아닌)을 활용해봐야겠다고 생각했습니다. Sotmp API Document를 보다가 Stomp.over()가 리턴하는 [CompatClient](https://stomp-js.github.io/api-docs/latest/classes/CompatClient.html)가 Deprecate 될거라는 얘기를 봣는데요. 이걸 보고나니 더욱 Stomp.over()를 통해 Stomp.over() 대신 new Client(stompConfig)를 써야겠다고 생각하게 됐습니다.

## 2. 서버에 메시지를 전송하지만, 연결이 안된다.

스프링 서버에 메시지가 잘 도착했지만, 소켓 연결이 되질 않았습니다. 디버깅을 통해 서버는 SessionConnectedEvent를 publish하는 것을 확인했음에도 말이지요. 이런 저런 문서를 확인하다가 [이 문서](https://stomp-js.github.io/workaround/stompjs/rx-stomp/ng2-stompjs/react-native-additional-notes.html)를 통해 NULL chopping 이라는 문제라는 걸 알았습니다. 메시지를 파싱할 때 null과 관련된 문제가 있다는 것 같은데요. 

## 3. 해결

위 문서를 통해 [appendMissingNULLonIncoming](https://stomp-js.github.io/api-docs/latest/classes/Client.html#appendMissingNULLonIncoming)를 true로 설정해주어 해결했습니다. 그런데 이 설정은 큰 데이터를 전달할 때 문제가 될 수 있다고 하네요. 다음은 관련 코멘트입니다.

> A bug in ReactNative chops a string on occurrence of a NULL. See issue [https://github.com/stomp-js/stompjs/issues/89]{@link https://github.com/stomp-js/stompjs/issues/89}. This makes incoming WebSocket messages invalid STOMP packets. Setting this flag attempts to reverse the damage by appending a NULL. If the broker splits a large message into multiple WebSocket messages, this flag will cause data loss and abnormal termination of connection.
>
> This is not an ideal solution, but a stop gap until the underlying issue is fixed at ReactNative library.