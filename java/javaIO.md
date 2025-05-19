# Java I/O

java에서는 모든 I/O가 Stream을 통해 이루어짐

## Stream

Byte 형태로 데이터를 운반하는데 사용되는 연결통로

물이 한쪽 방향으로만 흐르듯이 스트림은 단방향 통신만 가능

먼저 보낸 데이터를 먼제 받게 되어있음(FIFO)

### InputStream / OutputStream

java에서는 처리 단위에 따라 Reader - inputStream / Writer - outputStream 으로 나눠 통신

inputStream은 stream을 한 줄 씩 읽고, outputStream으로 데이터를 내보내며 해당 공간을 비움

```java
import java.io.InputStream;
import java.io.OutputStream;		
  
 /*
* InputStream 로 입력받는 경우 맨 앞 문자 1개만 출력됨 && int 형태로 입력받음
 */		
InputStream in = System.in;
OutputStream out = System.out;
        
int idata = in.read(); // input 은 read 와 연결되어있기 때문에 in.read 를 사욯한다.
		
out.write(idata); // output 은 write 와 연결되어있기 때문에 out.write 를 사용한다
out.flush(); // flush 를 써주지 않으면 출력되지 않는다
out.close(); // output 을 끝내는 매서드
```

flush는 write에 저장된 값을 출력함과 동시에 비워주는 역할

close는 끝 마무리해주는 역할

### InputStreamReader / OutputStreamWriter

여러 개의 값을 입/출력

```java
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

/*
 * InputStreamReader 로 입력받는 경우에는 배열을 어떻게 주느냐에 따라 2개 이상의 값을 받을 수 있음
 */
InputStream in = System.in;
InputStreamReader reader = new InputStreamReader(in); // InputStreamReader 사용하기 위해 객체 생성
		
OutputStream out = System.out;
OutputStreamWriter writer = new OutputStreamWriter(out); // OutputStreamWriter 사용하기 위해 객체 생성
		
char cdata[] = new char[2]; // 이제는 char 를 기본형으로 받을 수 있고, 2개 이상의 값을 배열을 통해 받아올수 있다.
reader.read(cdata);
		
int IcData = cdata[0]-'0'; // 배열이기 때문에 char 로 받은 값을 int로 변환하여 계산하고 싶은 경우 이처럼 사용해야한다.
		
writer.write("입력받은 값 : ");
writer.write(cdata);
writer.write("\n");
writer.write("입력받은 첫번째 값 + 10 : ");
writer.write(IcData+10+"\n"); // 입력받은 첫번째 값 +10
		
System.out.println("#######결과#######");
writer.flush(); // 이 매서드를 통해 출력
writer.close();
```

InputStreamReader를 통해 2개 이상의 값을 받아오기 위해서는 배열을 사용해서 값을 받아옴

즉, 고정적인 값 밖에 못 받음

입력 값이 고정 값보다 작으면 그만큼 공간 낭비

입력 값이 고정 값보다 크면 공간 부족 문제 발생

## Buffer

가변적인 값을 받을 수 있음

**입력받은 값을 버퍼에 저장했다가 버퍼가 가득차거나 개행 문자가 나타나면 버퍼의 내용을 한 번에 전송**

![image.png](/java/img/image.png)

한번에 전송하기에 속도가 빠름

scanner에 비해 코드가 조금 더 복잡

띄어쓰기와 개행문자를 경계로 입력 값을 인식하는 Scanner와 달리,

개행문자만 경계로 인식하기 때문에 중간에 띄어쓰기의 경우 가공이 필요

기본 타입이 String 이기에 숫자일 경우 형변환이 필수적

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;

public static void main(String[] args) throws IOException {

	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
	BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

	String s = br.readLine(); // bufferedwriter 의 기본형은 String
		
	int i = Integer.parseInt(s) +10; // String 을 int로 형변환 후 +10 !!
		
	br.close(); // bufferedreader 도 입력을 마쳤다면 닫아주자
		
	bw.write("입력받은 값 : "+ s); // 출력
	bw.newLine(); // 개행 메소드
	bw.write("입력받은 값 +10 : "+i+"\n"); // 이렇게 하니까 제대로 출력됨

	bw.flush(); // 남은 값 출력 && 버퍼 초기화
	bw.close(); // bufferedwriter 닫기
 }
```

BufferedWriter를 사용해서 입력된 내용을 출력

flush()를 통해 꽉 차지 않아도 내용을 강제로 출력 후 버퍼를 비우게 할 수 있다.