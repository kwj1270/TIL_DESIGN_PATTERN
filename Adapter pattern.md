# Adapter pattern
    
`[사진]` 
   
[1. Adapter pattern 이란?](#Adapter-pattern-이란)     
[2. 용어 설명](#용어-설명)     
[3. 설계 방법](#설계-방법)     
[4. 예제](#예제)     
   
# Adapter pattern 이란?        
> 한 클래스의 인터페이스를 클라이언트에서 사용하고자하는 다른 인터페이스로 변환한다.    
> 어댑터를 이용하면 인터페이스 호환성 문제 때문에 같이 쓸 수 없는 클래스들을 연결해서 쓸 수 있다.

이름 그대로 어댑터처럼 사용되는 패턴이다.   
한국이 220V 전압을 사용하는 것과 달리 나라마다 전압의 크기와 규격이 다른 경우가 많다.   
그럴 때 우리는 '어댑터'를 통해 우리의 전자 기기가 사용될 수 있도록 설정을 해준다.     
어댑터 패턴도 마찬가지이다.  

기존에 있는 시스템에 새로운 써드파티 라이브러리가 추가된다던지, 
레거시 인터페이스를 새로운 인터페이스로 교체하는 경우에 
코드의 재사용성을 높일 수 있는 방법이 어댑터 패턴을 사용하는 것이다.

쉽게 설명하자면 라이브러리마다 인터페이스 및 메서드의 이름이 다르다.  
만약 새로운 기능을 추가하고 싶은데 인터페이스와 메스드의 이름이 다르다면?       
일반적인 방법으로는 기존 코드를 수정하여 새로 작성해야 한다.
하지만 우리는 OCP 원칙을 지키고 싶다.   
이럴 때 우리가 사용할 수 있는 방법이 바로 어댑터 패턴이다.   

# 용어 설명  
* **client : 사용자**
  * 써드파티 라이브러리나 외부시스템을 사용하려는 쪽이다.   
  * 전략 패턴을 기준으로 말하자면 `Context`로 라이브러리를 직접 호출한다.        
* **Adaptee : 외부 시스템** 
  * 써드파티 라이브러리나 외부 시스템을 의미한다.
  * 즉, 어댑터를 사용해야 하는 대상이다.   
* **Adapter : 호출자**
  * `Client`와 `Adaptee`중간에서 호환성이 없는 둘을 연결시켜주는 역할을 한다. 
  * `Target Interface`를 구현하며, `Client`는 인터페이스를 통해 어댑터에 요청을 보낸다. 
  * 어댑터는 `Clinet`의 요청을 `Adaptee`가 이해할 수 있는 방법으로 전달하고, 
    처리는 `Adaptee`에서 이루어진다.
* **Target Interface : 수신자**
  * `Adapter`가 구현(implements) 하는 인터페이스이다. 
  * 전략 패턴을 기준으로 말하자면 `Strategy Interface`이다.   
  * `Client`는 `Target Interface`를 통해 `Adaptee`라이브러리를 사용한다.
  
* **ConcreteCommand : 구체적 명령**
  * 실제로 실행되는 기능을 구현한 클래스 
  * 즉, Command 를 구현한 클래스  
  * 인터페이스의 추상 메서드를 오버라이딩 하여 각 클래스에 맞게 정의한다.   
    
# 설계 방법 
1. 변경이 이루어질 코드를 추상 클래스 or 인터페이스로 분리한다.
2. Command 를 상속/구현한 ConcreteCommand를 정의한다.
3. 클라이언트에서 ConcreteCommand 의존성 주입을 시켜준다.    

# 예제
## 기존 코드

```java
package command_before;

public class MyProgram {
    public static void main(String[] args) {
        RobotKit robotKit = new RobotKit();
        robotKit.addCommand(Robot.command.FORWARD);
        robotKit.addCommand(Robot.command.TURN_LEFT);
        robotKit.addCommand(Robot.command.TURN_RIGHT);
        robotKit.addCommand(Robot.command.PICKUP);
    }
}
```
```java
package command_before;

import java.util.ArrayList;

public class RobotKit {
    private Robot robot = new Robot();
    private ArrayList<Robot.command> commands;

    public void addCommand(Robot.command command) {
        commands.add(command);
    }

    public void start() {
        for (Robot.command command : commands) {
            if (command == Robot.command.FORWARD) {
                robot.moveForward(1);
            } else if (command == Robot.command.TURN_LEFT) {
                robot.turn(Robot.Direction.LEFT);
            } else if (command == Robot.command.TURN_RIGHT) {
                robot.turn(Robot.Direction.RIGHT);
            } else if (command == Robot.command.PICKUP) {
                robot.pickup();
            }
        }
    }
}
```
```java
package command_before;

public class Robot {

    public enum Direction {LEFT, RIGHT}

    public enum command {FORWARD, TURN_LEFT, TURN_RIGHT, PICKUP}

    public void moveForward(int space) {
        System.out.println(space + " 칸 전진");
    }

    public void turn(Direction _direction) {
        System.out.println(
                (_direction == Direction.LEFT ? "왼쪽" : "오른쪽") + "으로 방향전환"
        );
    }

    public void pickup() {
        System.out.println("앞의 물건 집어 들기");
    }

}
```

## Command 적용
* client : MyProgram
* Invoker : RobotKit
* Receiver : Robot 
* Command : Command 인터페이스 
* ConcreteCommand : MoveForwardCommand, TurnCommand, PickupCommand

## 변경이 이루어질 코드를 추상 클래스 or 인터페이스로 분리한다.   
**RobotKit 변경 전**
```java
package command_before;

import java.util.ArrayList;

public class RobotKit {
    private Robot robot = new Robot();
    private ArrayList<Robot.command> commands;

    public void addCommand(Robot.command command) {
        commands.add(command);
    }

    public void start() {
        for (Robot.command command : commands) {
        ///////////////// 변경 구문 ///////////////// 
            if (command == Robot.command.FORWARD) {
                robot.moveForward(1);
            } else if (command == Robot.command.TURN_LEFT) {
                robot.turn(Robot.Direction.LEFT);
            } else if (command == Robot.command.TURN_RIGHT) {
                robot.turn(Robot.Direction.RIGHT);
            } else if (command == Robot.command.PICKUP) {
                robot.pickup();
            }
        //////////////////////////////////////////   
        }
    }
}
```
**RobotKit 변경 후**
```java
package command_before;

import java.util.ArrayList;

public class RobotKit {
    private Robot robot = new Robot();
    private ArrayList<Robot.command> commands;

    public void addCommand(Robot.command command) {
        commands.add(command);
    }

    public void start() {
        for (Robot.command command : commands) {
            // 변경 구문이라 삭제 됨
        }
    }
}
```
* 커멘드 종류가 하나 추가되면 조건문이 추가된다.       
* 공통으로 사용할 기능이 추가되면 각 조건문마다 코드를 추가해야한다.       
* 즉, 기능이 추가되면 조건문에 있는 값들이 추가되거나 변결되므로 조건문을 지워준다.   

**Command 추상 클래스 생성**
```java
abstract class Command {
    protected Robot robot;

    public void setRobot(Robot _robot) {
        robot = _robot;
    }
    public abstract void execute();
}
```
* RobotKit에서 지운 구문을 공통으로 묶어줄 수 있는 추상클래스 or 인터페이스를 정의한다.   
* 인터페이스로 정의할 경우 Robot을 ConcreteCommand 에서 의존 받도록 설정한다.     
* 우리는 ConcreteCommand는 Command를 상속받기에 자동으로 의존 관계를 가진다.
* 정확히 말하면 Command에서 Robot을 의존성 주입 받도록 한 것이다.   
* 인터페이스 : ConcreteCommand 에서 주입
* 추상 클래스 : Command 에서 주입      
* 물론 이 같은 방법이 절대적인 것은 아니고 본인의 스타일대로 작성하면 된다.          
    
**RobotKit이 Command를 호출하도록 수정**
```java
package command;

import java.util.ArrayList;

public class RobotKit {
    private Robot robot = new Robot();
    private ArrayList<Command> commands = new ArrayList<>();

    public void addCommand(Command command) {
        commands.add(command);
    }

    public void start() {
        for(Command command : commands) {
            command.setRobot(robot);
            command.execute();
        }
    }
}
```  
* 기존 Enum 타입을 Commnad 타입으로 변경한다.(command enum은 삭제해도 된다.)  
* Command가 Robot을 주입 받도록 설정했으므로 Command에 객체를 넣어준다.    
* 그리고 Command의 공통 메서드인 `execute()`를 실행한다.    
    
## Command 추상 클래스를 상속한 ConcreteCommand를 정의한다.  

**MoveForwardCommand**
```java
public class MoveForwardCommand extends Command {
    int space;

    public MoveForwardCommand(int _space) {
        space = _space;
    }

    @Override
    public void execute() {
        robot.moveForward(space);
    }
}
```
* 이전과 달리 움직이는 칸 갯수도 넣어줬다.       
* 기존 코드에서 이 같은 작업을 하려면 코드가 엄청 길어질 것이다.     
* 왜냐하면 나머지 메서드는 정수형 타입이 필요 없기에 이를 처리하는 로직도 넣어야했다.   
       
**TurnCommand**
```java
class TurnCommand extends Command {
    Robot.Direction direction;
    public TurnCommand(Robot.Direction _direction) {
        direction = _direction;
    }

    @Override
    public void execute() {
        robot.turn(direction);
    }
}
```

**PickupCommand**
```java
class PickupCommand extends Command {

    @Override
    public void execute() {
        robot.pickup();
    }
}
```

## 클라이언트에서 ConcreteCommand 의존성 주입을 시켜준다.    
```java
public class MyProgram {
    public static void main(String[] args) {
        RobotKit robotKit = new RobotKit();

        robotKit.addCommand(new MoveForwardCommand(2));
        robotKit.addCommand(new TurnCommand(Robot.Direction.LEFT));
        robotKit.addCommand(new MoveForwardCommand(1));
        robotKit.addCommand(new TurnCommand(Robot.Direction.RIGHT));
        robotKit.addCommand(new PickupCommand());

        robotKit.start();
    }
}
```  
* 각각의 커멘드를 순서대로 넣어준다.     
* 비유하자면 콘솔기의 버튼을 누르듯이 동작을 하게 되는 것이다.       
* 새로운 ConcreteCommand가 생기더라도 클라이언트에서만 코드를 바꾸면 된다.      
* 즉, RobotKit, Robot 은 수정할 필요가 없고 OCP 원칙을 지킬 수 있다.    
   

# 참고     
* **블로그 :**    
https://yaboong.github.io/design-pattern/2018/10/15/adapter-pattern/ 
   
* **동영상 :**       
얄팍한 코딩지식_디자인패턴1 : https://www.youtube.com/watch?v=lJES5TQTTWE    