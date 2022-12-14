# 5장

## 책임과 메시지

### 자율적인 책임

객체가 어떤 행동을 하는 유일한 이유는 다른 객체로부터 요청을 수신했기 때문이다.

요청을 처리하기 위해 객체가 수행하는 행동을 책임이라고 한다. 따라서 자율적인 객체란 스스로의 의지와 판단에 따라 각자 맡은 책임을 수행하는 객체를 의미한다.

적절한 책임이 자율적인 객체를 낳고, 자율적인 객체들이 모여 유연하고 단순한 협력을 낳는다. 따라서 협력에 참여하는 객체가 얼마나 자율적인지가 전체 애플리케이션의 품질을 결정한다.

### 너무 추상적인 책임

포괄적이고 추상적인 책임을 선택한다고 해서 무조건 좋은 것은 아니다. 책임이 수행 방법을 제한할 정도로 구체적인 것도 문제지만 협력의 의도를 명확하게 표현하지 못할 정도로 추상적인 것 역시 문제다.

책임은 협력에 참여하는 의도를 명확하게 설명할 수 있는 수준 안에서 추상적이어야 한다.

### 메시지와 메서드

**메시지**

하나의 객체는 메시지를 전송함으로써 다른 객체에 접근한다. 메시지는 메시지 이름과 인자의 두 부분으로 구성되고, 메시지 전송은 수신자와 메시지의 조합으로 결국 메시지 전송은 수신자, 메시지 이름, 인자의 조합이 된다. 

**메서드**

메시지를 처리하기 위해 내부적으로 선택하는 방법을 메서드라고 한다.

객체는 메시지를 수신하면 먼저 해당 메시지를 처리할 수 있는지 여부를 확인한다. 메시지를 처리할 수 있으면 책임을 수행하기 위해 메시지를 처리할 방법인 메서드를 선택하게 된다.

메시지를 수신한 객체가 실행 시간에 메서드를 선택할 수 있다는 사실은 다른 프로그래밍 언어와 객체지향 프로그래밍 언어를 구분 짓는 핵심적인 특징 중 하나다.

**다형성**

메시지와 메서드의 차이와 관계를 이해하고 나면 다형성을 쉽게 이해할 수 있다.

다형성이란 서로 다른 타입에 속하는 객체들이 동일한 메시지를 수신할 경우 서로 다른 메서드를 이용해 메시지를 처리할 수 있는 메커니즘을 가리킨다.

다형성은 역할, 책임, 협력과 깊은 관련이 있다. 서로 다른 객체들이 다형성을 만족시킨다는 것은 객체들이 동일한 책임을 공유한다는 것을 의미한다. 메시지 수신자들이 동일한 오퍼레이션을 서로 다른 방식으로 처리하더라도 메시지 송신자의 관점에서 동일한 책임을 수행하는 것이다.

다형성은 객체들의 대체 가능성을 이용해 설계를 유연하고 재사용 가능하게 만든다. 다형성을 사용하면 송신자가 수신자의 종류를 모르더라도 메시지를 재사용할 수 있다. 즉, 다형성은 수신자의 종류를 캡슐화한다.

**다형성을 통한 유연한 협력의 장점**

1. 수신자를 다른 타입의 객체로 대체하더라도 송신자는 알지 못한다. 따라서 송신자에 대한 파급효과 없이 협력을 변경할 수 있다.
2. 송신자에게 영향을 미치지 않고 수신자를 교체할 수 있기 때문에 협력의 세부적인 수행 방식을 쉽게 수정할 수 있다.
3. 협력에 영향을 미치지 않고서도 다양한 객체들이 수신자의 자리를 대체할 수 있기 때문에 다양한 문맥에서 협력을 재사용할 수 있다.

**What/Who 사이클**

책임 주도 설계의 핵심은 어떤 행위가 필요한지를 먼저 결정한 후 행위를 수행할 객체를 찾는 것이다. 이 과정을 흔히 What/Who 사이클이라고 한다.

협력이라는 문맥 안에서 필요한 메시지를 먼저 결정한 후에 메시지를 수신하기에 적합한 객체를 선택한다. 그리고 수신된 메시지가 객체의 책임을 결정한다.

메시지를 먼저 결정함으로써 객체의 인터페이스를 발견할 수 있다.

### 객체 인터페이스

일반적으로 인터페이스는 다음과 같은 세 가지 특징을 지닌다.

1. 인터페이스의 사용법을 익히기만 하면 내부 구조나 동작 방식을 몰라도 대상을 조작하거나 의사를 전달할 수 있다.
2. 인터페이스 자체는 변경하지 않고 단순히 내부 구성이나 작동 방식만을 변경하는 것은 인터페이스 사용자에게 어떤 영향도 미치지 않는다.
3. 대상이 변경되더라도 동일한 인터페이스를 제공하기만 하면 문제 없이 상호작용 할 수 있다.

객체가 어떤 메시지를 수신할 수 있느냐가 어떤 책임을 수행할 수 있느냐와 어떤 인터페이스를 가질 것인지를 결정한다.