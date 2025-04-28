# Request & Response Annotation 정리

# @RequestParam

- 쿼리 파라미터로 전달되는 데이터를 받을 수 있게 해줌
- 파라미터 이름으로 바인딩
- Primitive 타입, Map 과 같은 컬렉션 등 이용 가능
- 만약 괄호 안의 값이 없다면 변수명으로 바인딩 됨

# @ModelAttribute

- @RequestParam과 동일하게 쿼리 파라미터로 날라오는 데이터를 받을 수 있음
- 차이점은 필요한 객체를 생성하고 그 객체에 값을 넣어주는 것까지 가능
- 객체 생성 → 넘어온 쿼리 파라미터 이름으로 객체에 프로퍼티를 찾음 → 해당 객체의 프로터피의 setter를 호출해 파라미터 값을 바인딩
- @RequestParam으로 쿼리 파라미터를 줄줄이 받는 것 보다, 적절한 객체를 생성해서 @ModelAttribute로 받으면 훨씬 깔끔한 코드를 작성할 수 있다.

# @RequestBody

- 위 두 가지 어노테이션과 가장 큰 차이점은 어떤 대상을 타겟으로 하나에 있다.
- 위 어노테이션들은 쿼리 파라미터 대상으로 하고, @RequestBody의 경우 HTTP 메세지 바디에서 오는 JSON 형태의 데이터를 Java 객체에 매핑할 때 사용함
- 넘어오는 JSON형태의 데이터에 맞게 객체를 만들어 어노테이션을 붙여주면 객체에 값들이 맵핑되어 사용할 수 있음

# @ResponseBody

- Spring에서 Controller의 반환 타입이 String일 때 기본적으로 View를 내려주는데, @ResponseBody 어노테이션을 사용하면 HTTP 메세지의 바디에 return 값을 넣어준다.
- 객체가 반환타입일 때 @ResponseBody가 있으면 객체를 HTTP 메세지 바디에 넣어 알맞게 JSON 형식으로 변환됨

# 원리

모든 것은 Spring이 HTTP Message Converter를 사용하여 가능

HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용

canRead(), canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크

read() , write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

ByteArrayHttpMessageConverter : byte[] 데이터를 처리한다.클래스 타입: byte[] , 미디어타입: */* ,요청 예) @RequestBody byte[] data응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-streamStringHttpMessageConverter : String 문자로 데이터를 처리한다.

클래스 타입: String , 미디어타입: */*요청 예) @RequestBody String data응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain

MappingJackson2HttpMessageConverter : application/json클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련요청 예) @RequestBody HelloData data응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

### 실제 요청이 왔을 때

HTTP 요청 데이터 읽기

HTTP 요청이 오고, 컨트롤러에서 @RequestBody , HttpEntity 파라미터를 사용한다.

메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 를 호출한다.

대상 클래스 타입을 지원하는가?

예) @RequestBody 의 대상 클래스 ( byte[] , String , HelloData )HTTP 요청의 Content-Type 미디어 타입을 지원하는가?

예) text/plain , application/json , */*canRead() 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환한다.

HTTP 응답 데이터 생성컨트롤러에서 @ResponseBody , HttpEntity 로 값이 반환된다.

메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출한다.

대상 클래스 타입을 지원하는가.

예) return의 대상 클래스 ( byte[] , String , HelloData )

HTTP 요청의 Accept 미디어 타입을 지원하는가.

(더 정확히는 @RequestMapping 의 produces )

예) text/plain , application/json , */*

canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.