---
layout: post
title:  크롬 브라우저 Websocket 클라이언트와 Spring Websocket 서버 연결하기
date:   2020-12-17 14:00:00 +0800
categories: issue
tag: socket
sitemap :
  changefreq : daily
  priority : 1.0
---

## 1. stompEndpoint에 withSockJS()를 붙이면 크롬 클라이언트와 연결이 안된다.

많은 예제를 통해 SockJS 클라이언트와 withSockJS()를 활용하면 문제 없이 웹소켓 연결을 할 수 있다는 것을 알 수 있었습니다. 그런데 이렇게 하니 브라우저에서 연결이 안되더군요. SockJS를 써야만 연결 가능하다는 것이 아쉬웠는데요. 제가 클라이언트로서 WebSocket API를 활용할 때 브라우저를 활용해도 연결이 다 됐었거든요. 그래서 withSockJS 뿐 아니라 withSockJS를 붙이지 않은 endpoint도 등록해줬습니다.

그런데도 불구하고 연결이 안됐습니다. 그래서 브라우저에서 request 보내고 디버깅을 해봤습니다. 이런 저런 삽질 끝에 크롬 브라우저에서 보내는 EOS(End Of String)를 스프링이 EOS라고 인식하지 않기 때문이란 걸 알았습니다.

## 2. 서버 코드 수정을 통한 해결

처음에는 클라이언트에서 잘 보내주면 되니 서버 코드를 바꾸고 싶지 않아 클라이언트에서 EOF를 잘 보내려고 노력해봤는데요. 잘 안되더군요...^^; 어떻게 보내던 스프링은 EOS를 그냥 문자로 인식해버렸습니다.

그래서 [이 문서](https://stackoverflow.com/questions/47373402/spring-stromp-incomplete-frame)를 참고하여 만약 요청에 EOF가 없으면 EOF를 붙여주는 코드를 서버 코드에 넣었습니다. 사실 제대로 된 해결인지 확신하기가 어려운 것이, 만약 이게 제대로 된 해결이라면 많은 사람들이 이슈를 제기했을텐데 관련 문서가 별로 없었습니다. 그래서 편법이 아닌가 좀 걱정이네요. 이후 리팩토링, 혹은 deprecate를 고려해야 할 부분입니다.

```java
class EOSAdderForBrowser extends WebSocketHandlerDecorator {

    private static final Logger logger = LoggerFactory.getLogger(EOSAdderForBrowser.class);
    private static final String END_OF_STRING = "\u0000";
    public static final String NEW_LINE = "\n";

    public EOSAdderForBrowser(WebSocketHandler webSocketHandler) {
        super(webSocketHandler);
    }

    @Override
    public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {
        super.handleMessage(session, updateBodyIfNeeded(message));
    }

    /**
     * Updates the content of the specified message. The message is updated only if it is
     * a {@link TextMessage text message} and if does not contain the <tt>null</tt> character at the end. If
     * carriage returns are missing (when the command does not need a body) there are also added.
     */
    private WebSocketMessage<?> updateBodyIfNeeded(WebSocketMessage<?> message) {
        if (!(message instanceof TextMessage) || ((TextMessage) message).getPayload().endsWith(
            END_OF_STRING)) {
            return message;
        }
        String payload = ((TextMessage) message).getPayload();
        final Optional<StompCommand> stompCommand = getStompCommand(payload);
        if (!stompCommand.isPresent()) {
            return message;
        }
        payload = settleFormOfCommandsWithoutBody(payload, stompCommand.orElseThrow(
            IllegalStateException::new));

        payload += END_OF_STRING;

        return new TextMessage(payload);
    }

    private String settleFormOfCommandsWithoutBody(String payload,
        StompCommand stompCommand) {
        if (!stompCommand.isBodyAllowed() && !payload.endsWith(NEW_LINE + NEW_LINE)) {
            if (payload.endsWith(NEW_LINE)) {
                payload += NEW_LINE;
            } else {
                payload += NEW_LINE + NEW_LINE;
            }
        }
        return payload;
    }

    /**
     * Returns the {@link StompCommand STOMP command} associated to the specified payload.
     */
    private Optional<StompCommand> getStompCommand(String payload) {
        final int firstCarriageReturn = payload.indexOf('\n');

        if (firstCarriageReturn < 0) {
            return Optional.empty();
        }

        try {
            return Optional.of(
                StompCommand.valueOf(payload.substring(0, firstCarriageReturn))
            );
        } catch (IllegalArgumentException e) {
            logger.trace("Error while parsing STOMP command.", e);

            return Optional.empty();
        }
    }
}
```