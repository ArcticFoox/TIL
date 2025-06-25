# Objects/chapter1

소프트웨어 모듈 목적은 ‘제대로된 실행 동작’, ‘변경 용이성’, ‘코드를 읽는 사람과의 의사소통’

객체는 자신의 데이터를 스스로 처리하는 자율적인 존재

객체는 캡슐화를 이용해 의존성을 적절히 관리하여 결합도를 낮추는 것

설계는 여러 방법이 될 수 있는, 트레이드오프의 산물

훌륭한 객체지향 설계는 모든 객체들이 자율적으로 행동하며, 내일의 변경을 매끄럽게 수용할 수 있는 설계

## 문제

### 예상을 빗나가는 코드

**문제1**

이해 가능한 코드: 우리가 예상하는 방식으로 동작하는 코드

예상: 관람객이 초대장이나 돈을 지불하여 티켓을 얻음

코드: Theater의 enter method는 ‘소극장’이 관람객의 가방을 확인

예상과 다른 방식이기 때문에 코드를 읽는 사람이 제대로 의사소통하지 못함

```java
public void enter(Audience audience) {
    if (audience.getBag().hasInvitation()) {
        Ticket ticket = ticketSeller.getTicketOffice().getTickets();
        audience.getBag().setTicket(ticket);
    } else {
        Ticket ticket = ticketSeller.getTicketOffice().getTickets();
        audience.getBag().minusAmount(ticket.getFee());
        ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
        audience.getBag().setTicket(ticket);
    }
}
```

**문제2**

Theather의 enter 코드를 이해하기 위해 여러 세부 사항을 한꺼번에 기억해야 함

audience가 bag을 가짐

audience의 bag에 현금과 티켓이 있음

ticketSeller가 ticketOffice에서 일함

ticketOffice에는 돈과 티켓이 보관

### 변경에 취약한 코드

Audience와 TicketSeller를 변경할 경우 Theater도 함께 변경해야 함

수많은 발생 가능한 변경 사항 존재

→ 고객이 현금말고 카드 사용 시, 판매원이 판매소 밖에서 티켓 판매 시

## 설계 개선

관람객과 판매원을 자율적인 존재로 만드는 것을 목표

Theater가 원하는 것은 관람객이 소극장에 입장하는 것뿐

Audience와 TicketSeller의 내부를 알지 못하게 차단

```java
public class Theater {
    // ...
    public void enter(Audience audience) {
        ticketSeller.sellTo(audience);
    }
}
```

```java
public class TicketSeller {
    //... 

    // 외부에서 알 필요 없음 - 접근 차단
    // public TicketOffice getTicketOffice() {
    //     return ticketOffice;
    // }
  
    public void sellTo(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketOffice.getTickets();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketOffice.getTickets();
            audience.getBag().minusAmount(ticket.getFee());
            ticketOffice.plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```

Theater는 TicketSeller의 Interface에만 의존

- Theater는 TicketOffice가 TicketSeller 내부에 존재함을 모름
- TicketSeller 내부에 TicketOffice 존재 → Implementation 영역

```java
public class Audience {
    // ...
    
    // 외부에서 알 필요 없음 - 접근 차단
    // public Bag getBag() {
    //     return bag;
    // } 

    public Long buy(Ticket ticket) {
        if (bag.hasInvitation()) {
            bag.setTicket(ticket);
            return 0L;
        } else {
            bag.setTicket(ticket);
            bag.minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
```

```java
public class TicketSeller {
    // ...
    public void sellTo(Audience audience) {
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTickets()));
    }
}
```

Theater는 Audience나 TicketSeller의 내부에 직접하지 않음

TicketSeller는 Bag 내부의 내용물을 직접 확인하지 않음

## 추가 개선

```java

public class Bag { 
  // ...
  public Long hold(Ticket ticket) {
    if (hasInvitation()) {
      setTicket(ticket);
      return 0L;
    } else {
      setTicket(ticket);
      minusAmount(ticket.getFee());
      return ticket.getFee();
    }
  }
  
  // change to private
  private boolean hasInvitation() {
    return invitation != null;
  }

  // change to private
  private void setTicket(Ticket ticket) {
    this.ticket = ticket;
  }

  // change to private
  private void minusAmount(Long amount) {
    this.amount -= amount;
  }
}

```

```java
public class Audience {
    //...
    public Long buy(Ticket ticket) {
        return bag.hold(ticket);
    }
}
```

```java
public class TicketOffice { 
  // ...
  public void sellTicketTo(Audience audience) {
    plusAmount(audience.buy(getTickets()));
  }
  // change to private
  private void plusAmount(Long amount) {
    this.amount += amount;
  }
}
```

```java
public class TicketSeller {
    // ...
    public void sellTo(Audience audience) {
        ticketOffice.sellTicketTo(audience);
    }
}
```

하지만, TicketOffice와 Audience의 의존성이 추가됨

어떤 기능 설계 방법은 한 가지 이상일 수 있음

동일한 기능을 한 가지 이상의 방법으로 설계할 수 있기 때문에 결국 설계는 트레이드오프의 산물

좋은 설계란

- 요구하는 기능을 구현하는 코드를 짜는 동시에, 내일 쉽게 변경할 수 있는 코드를 짜야 함
- 오늘 요구하는 기능을 온전히 수행하면서 내일의 변경을 매끄럽게 수용할 수 있는 설계