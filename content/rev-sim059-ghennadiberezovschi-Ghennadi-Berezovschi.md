https://github.com/Ghennadi-Berezovschi/Simulation2.git  
[Ghennadi Berezovschi]

После первого рефакторинга, стало лучше.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Это совершенно несерьезно. Всегда делай защиту от дурака
```
Enter the width of the map: 
цу
Exception in thread "main" java.util.InputMismatchException
```

2. Чтобы запустить симуляцию, сначала нужно ее настроить- ввести 8 параметров, делать это каждый раз довольно утомительно.
Кастомное параметризирование это хорошо, но нужно предусмотреть возможность запустить дефолтную конфигурацию. Для этого можно сделать второй Main, который сразу запустит конфигурацию по умолчанию.

## ХОРОШО

+ 👍 Есть начальное конфигурирование симуляции
+ 👍 Спрайты существ не хранятся в самих существах
+ 🚀 Два разных рендерера карты: с емоджи и буквами

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов пишутся стилем lower_case
```
Entities

//ПРАВИЛЬНО:
entities
```

- Избыточный контекст
```
public Entity getEntityAtPosition(Position position)
List<Position> getPathToTarget(GameBoard board, Position startPosition, Class<? extends Entity> targetType)
Class<? extends Entity> targetType
BoardRender.displayBoard(GameBoard board);

//ПРАВИЛЬНО:
public Entity get(Position position)
List<Position> getPath(GameBoard board, Position startPosition, Class<? extends Entity> targetType)
Class<? extends Entity> target
BoardRender.display(GameBoard board);
```

- Это не сеттер
```
GameBoard.setEntity(Position position, Entity entity)

//ПРАВИЛЬНО:
GameBoard.putEntity(Position position, Entity entity) //или addEntity(...)
```

- Rock?
```
class Roc 
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. class Position**, координаты

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

- Сеттеров в классе нет, поэтому поля row и column должны быть final.

- (±)Класс может быть заменен record'ом без потери функционала. 

**3. class GameBoard**, карта игры

- Нарушение конвенции кода, поля в классе должны находится вверху, а методы- под ними, в том числе конструктор.

- Два одинаковых метода
```
public Entity getEntityAtPosition(Position position) {
  return entities.get(position);
}

public Entity getEntityAt(Position position) {
  return entities.get(position);
}
```

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
protected Map<Position, Entity> entities = new HashMap<>();
```
Пусть не вводит в заблуждение то, что модификатор доступа здесь protected- такие поля доступны не только потомкам, но и всем классам в пакете.
Например, здесь к этому полю напрямую обращаются другие классы
```
public class GameStatusChecker {
  public boolean isRabbitLeft(GameBoard board) {
    //...
    for (Entity entity : board.entities.values()) {...}  <-- берет данные из другого класса напрямую, нарушая инкапсуляцию
  }
}      
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6, п."Закон Деметры"*  
Поле нужно сделать приватным и обращться к данным из entities нужно с помощью геттера `Entity getEntityAt(...)` и других методов чтения, например `List<Entity> getAll()`

- Никогда не возвращай null
```
protected Map<Position, Entity> entities = new HashMap<>();

public Entity getEntityAtPosition(Position position) {
  return entities.get(position);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void removeEntity(Position position) {
  entities.remove(position);
}
```

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
public void moveEntity(Position oldPosition, Position newPosition) {
  Entity entity = entities.remove(oldPosition);
  if (entity != null) {
    setEntity(newPosition, entity);
  }
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

+ 👍 В целом класс содержит минимальное количество недостатков.

**4. class GameStatusChecker**, проверяет условия окончания работы приложения

- Нарушение принципа разделения запросов и команд, метод выполняет более одного действия. Судя по названию, метод должен только осуществить проверку
```
  public boolean isRabbitLeft(GameBoard board) {
    for (Entity entity : board.entities.values()) {
      if (entity instanceof Rabbit) {
        return false; <-- выполнение запроса, действие 1
      }
    }
    System.out.println("All rabbits have been eaten. Ending simulation."); <-- выполнение команды, действие 2
    return true;
  }
```
*Мартин, "Чистый код", гл.3, п."Разделение команд и запросов"*  

- Нарушение SRP, чужая ответственность, зависимость модели от представления.
Модель(а это модель) не должна ничего печатать в консоль
```
System.out.println("All rabbits have been eaten. Ending simulation."); <-- выполнение запроскомандыа, действие 2
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.
Бизнес логика по проверке состояния приложения, которая явлена в этом классе, это тоже модель.

Логика использования этого класса должна быть следующей
```
//СЕЙЧАС ТАК:
class SimulationManager {
  //...  
  if (statusChecker.isRabbitLeft(board)) {
    break;
  }
  //...  
}

//ПРАВИЛЬНО:
class SimulationManager {
  //...  
  if (statusChecker.isRabbitLeft(board)) {
    System.out.println("All rabbits have been eaten. Ending simulation.");
    break;
  }
  //...  
}
```

**5. class SetupEntities**, что-то типа фабрики

- В отличии от фабрики существ, этот класс не создает, заселяет и возвращает карту, а принимает уже готовую карту и заселяет ее
```
class SetupEntities {
  public void setupCustomPositions(GameBoard board, int numberOfRabbits, /*другие количества*/) {...} 
}     
``` 

Вместо этого класса сделай простую фабрику карты, это стандартная и понятная всем практика
```
public class InteractiveMain {
  public static void main(String[] args) {
  //...
  GameBoard board = new GameBoard(width, height);
  setup.setupCustomPositions(board, КОЛИЧЕСТВО_ЗАЙЦЕВ, /*oths*/);
  //...
  }  
}

//ПРАВИЛЬНО С ИСПОЛЬЗОВАНИЕМ ФАБРИКИ:
GameBoard board = gameBoardFactory.get(ШИРИНА, ВЫСОТА, КОЛИЧЕСТВО_ЗАЙЦЕВ, /*oths*/);
```

**6. class NeighboursCells**, утилитный класс для создания списка соседних клеток

- Раз это утилитный класс со статическими методами, он должен быть final и иметь закрытый конструктор.

- Смещение нужно вынести в константы, чтобы каждый раз не пересоздавать их при вызове метода
```
public class NeighboursCells {
  public static List<Position> getAllNeighboursCells(...) { 
    int[][] directions = {
      {-1, 0}, {1, 0}, {0, -1}, {0, 1},
      {-1, -1}, {-1, 1}, {1, -1}, {1, 1}
    };
  }
}

//ПРАВИЛЬНО:
public class NeighboursCells {
  private final static int[][] SHIFTS = {
      {-1, 0}, {1, 0}, {0, -1}, {0, 1},
      {-1, -1}, {-1, 1}, {1, -1}, {1, 1}
  };

  public static List<Position> getAllNeighboursCells(...) {...}   
}
```

**7. class Node**, узел однонаправленного связного списка

- Поля должны быть final.

**8. class FindWayToTarget**, поиск пути

+ 👍 Использует в работе класс Node, это упрощает код.
+ 👍 Сигнатура поиска то, что надо
```
public List<Position> getPathToTarget(GameBoard board, Position startPosition, Class<? extends Entity> targetType) 
```

- Если пользовательские классы не будут возвращать null, то не придется проверять результат на null. А значит, если забудешь это сделать(проверить на null), не будет NPE.  
Метод `board.getEntityAt(...)` должен возвращать Entity или бросать исключение, если клетка пустая
```
Entity entity = board.getEntityAt(currentPosition);

if (entity != null && targetType.isInstance(entity)) {
  return constructPath(currentNode);
}

//ПРАВИЛЬНО:
if(!board.isPositionEmpty()) {
  Entity entity = board.getEntityAt(currentPosition);
  if (targetType.isInstance(entity)) {
    return constructPath(currentNode);
  }
}  
```

+ 👍 В целом, в классе простой понятный алгоритм.

**9. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня Creature.

**10. abstract class Creature extends Entity**

- У креатуры все время будет вызываться метод передвижения. А значит `FindWayToTarget pathFinder` должен быть полем класса а не метода- тогда он создастся один раз, а не будет пересоздаваться каждый раз при вызове метода.

- Вспомогательный метод съедения должен быть приватным, это часть внутренней механики креатуры
```
public void eatTarget(GameBoard board, Position targetPosition)
```

**11. class Rabbit/Wolf extends Creature**

- Магические числа: 20, 100.

- Нарушение SRP, зависимость модели от представления.
Модель(а это модель) не должна ничего печатать в консоль. 
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде или какое-либо сообщение о себе в этой среде- в данном случае в консоли.
В других средах(напр. Андроид) модель нельзя будет использовать
```
if (!emptyNeighborsCells.isEmpty()) {
  //...
  System.out.println("A new rabbit was born at position: " + newRabbitPosition);
} else {
  System.out.println("No place found for reproducing.");
}
```
Если модель во время выполнения каких-либо действий должна проинформировать об этом систему, то может использовать паттерн Callback: https://t.me/zhukovsd_it_chat/53243/184749

**12. interface BoardRender и его реализации**

+ 🚀 Интерфейс рендерера это хорошо- можно делать разные его реализации. Сейчас есть две реализации: одна показывает существ в виде emoji, другая- буквами.

- Интерфейс и его реализации нужно вынести в пакет renderer

- Нарушение DRY, дублирование кода в BoardLetterRender и BoardConsoleRender
```
public class BoardConsoleRender бла-бла {
  
  public void displayBoard(GameBoard board) {
    //...
    Entity entity = board.getEntityAtPosition(position);
    if (entity instanceof Rabbit) {
      System.out.print(" 🐇");
    } else if (entity instanceof Wolf) {
      System.out.print(" 🐺");
    }
    //...
  }
}        

public class BoardLetterRender бла-бла {
  
  public void displayBoard(GameBoard board) {
    //...
    Entity entity = board.getEntityAtPosition(position);

    if (entity instanceof Rabbit) {
      System.out.print(" R ");
    } else if (entity instanceof Wolf) {
      System.out.print(" W ");
    }
    //...
  }
}  
```

Нужно создать общего предка, в который вынести общий код
```
public abstract class AbstractBoardRender implements BoardRender {

  @Override
  public void displayBoard(GameBoard board) {
    //..
    Entity entity = board.getEntityAtPosition(position);
    if (entity instanceof Rabbit) {
      System.out.print(geRabbitSprite());
    } else if (entity instanceof Wolf) {
      System.out.print(getWolfSprite());
    } 
    //...
  }

  protected abstract String geRabbitSprite();
  protected abstract String getWolfSprite();
  //..
}
```

- В конце цепочки if-else нужно кидать исключение- значит в карте есть существо, спрайт которого мы не знаем. Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
else if (entity instanceof Roc) {
  System.out.print(" r ");
} else {
  System.out.print(" . ");
}

//ПРАВИЛЬНО:
else if (entity instanceof Roc) {
  System.out.print(" r ");
} else {
  //бросить исключение
}
``` 

**13. Отсутствуют Action's**

- Нарушать ТЗ можно, когда это улучшает проект. Отказавшись от Action'ов ты ухудшил проект. Введи их в проект.  

Идея Actions состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.
Actions, изложенный в ТЗ, это вариант реализации паттерна Command.

То есть акции должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
```
interface Action{
 void execute(Карта карта);
}

class ХодитьAction реализует Action {
  void execute(Карта карта) {
    //обойти всю карту
    //найти каждую креатуру
    //и дать ей пинка чтоб побежала
  }
}

class ДатьСигаретуВсемЗайцамAction реализует Action {
  void execute(Карта карта) {
    //обойти всю карту
    //найти всех зайцев и дать им по сигарете 
  }
}

List<Action> actions = List.of{new ДатьСигаретуВсемЗайцамAction(), new ХодитьAction()};
for(Action a: actions) {
  a.execute(карта);
}

/*
Результат: программа обойдет всю карту, найдет всех креатур и даст им пинка, чтобы они побежали.
А каждому зайцу предварительно даст сигарету.
*/
```

**14. class SimulationManager**, Симуляция

- Название не только неудачное, но сбивающее с толку. Под менеджерами/утилитами/хелперами в java понимаются довольно специфические классы. Прежде всего, методы в утилитных классах должны быть статик.
Поэтому приставку `Manager` отсюда нужно убрать.

+ 👍 Класс принимает достаточное количество зависимостей в конструктор, включая Рендерер.

- Класс принимает в конструктор кучу полей, которые не использует: `numberOfRabbits`, `numberOfWolves` etc.

**15. class InteractiveMain**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

- Нужно сделать отдельный класс Main, который будет запускать симуляцию с дефолтными характеристиками: размер карты, количество существ и так далее.

## ВЫВОД

После рефакторинга стало значительно лучше, но не все прошлые замечания были учтены. Прежде всего нужно ввести в проект Action'ы, как указано в ТЗ.  
Action начального заселения карты существами вводить не нужно, потому что уже есть фабрика существ.  

Необходимые Action:
+ TurnAction, который каждую итерацию игрового цикла будет инициировать движение креатур 
+ Action для возобновления ресурсов на карте 

n.59(135)  
#ревью #симуляция