https://github.com/HuviHonor/Simulation  
[Huvi]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Печатает кракозябры:  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim155/img0.png) 

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализована пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для избежания путаницы не называй пакеты, классы, интерфейсы так же, как называются классы и интерфейсы стандартной библиотеки.  
Есть стандартный интерфейс `Map`, не должно быть кастомного пакета с таким же названием
```java
package map;

//ПРАВИЛЬНО:
package simulationmap
```

- Избыточно. Мы и так понимаем, что метод поиска в классе поиска пути ищет именно путь, а не что-то иное
```java
public class PathFinder {

  public static Coordinates findPath(...)
  //...
}

//ПРАВИЛЬНО:
public class PathFinder {

  public List<Coordinates> find(...)
  //...
}
```

- Не называй объект именем "map", если это не реализация стандартного интерфейса "Map". 

Не называй объекты одного класса так, что их в коде можно спутать с объектом другого класса или реализацией другого интерфейса.  
В данном случае- стандартного Map
```java
public static Coordinates findPath(SimulationMap map, ...) 

//ПРАВИЛЬНО:
public static Coordinates findPath(SimulationMap simulationMap, ...) 
```

- Что возможно?
```java
private static boolean isPassable(...)
```

- Возврат констант- не повод нарушать конвенцию наименований 
```java
public int getHP() {
  return HP;
}

//ПРАВИЛЬНО:
public int getHp()
```

- Аббревиатуры это не повод нарушать конвенцию наименований
```java
private int HP;

//ПРАВИЛЬНО:
private int hp;
```

- Избыточно.

В названии полей класса не повторяй название класса
```java
public class Predator extends Creature {
  //...
  public Predator(int predatorSpeed, int predatorHP, int attackPower) {...}
}

//ПРАВИЛЬНО:
public class Predator extends Creature {
  //...
  public Predator(int speed, int hp, int attackPower) {...}
}
```

- Константы нужно писать стилем UPPER_SNAKE
```java
private final static String incorrectInputErrorMessage = "Ошибка: нужно ввести число ";

//ПРАВИЛЬНО:          
private final static String INCORRECT_INPUT_ERROR_MESSAGE = "Ошибка: нужно ввести число ";
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется**  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без else будет выглядеть читабельней
```java
if (entity != null && targetCondition.test(entity)) {
  //...
  return nextStep;
} else {
  int currentX = currentCoordinate.getX();
  //миллион строк кода
}

//ПРАВИЛЬНО:
if (entity != null && targetCondition.test(entity)) {
  //...
  return nextStep;
} 
int currentX = currentCoordinate.getX();
//миллион строк кода
```

**3. Форматирование строк**

Если нужно печатать или создавать строку с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, 
используй форматирование- тогда сразу будет виден весь шаблон
```java
private final static String incorrectInputErrorMessage =
            "Ошибка: нужно ввести число " +  START +  ", " + PAUSE + " или " + QUIT + "\n";

//ПРАВИЛЬНО:
private final static String INCORRECT_INPUT_ERROR_MESSAGE =  "Ошибка: нужно ввести число %d, %d или %d".formatted( START, PAUSE , QUIT);
```

**4. Нарушение DRY**, магические буквы, числа, слова. Вводи константы. 

- Совместная магия. 

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов.  
И используй оттуда.  
Либо перенеси ее в третий класс и из первых двух классов обращайся к этим константам- эти данные должны быть синхронизированы между собой
```java
public class SimulationUI {
  private static final int START = 1;
  private static final int PAUSE = 2;
  //...
}

public class ConsoleRenderer {

  public void render(SimulationMap map, int turnCounter) {
    //...
    System.out.println("1. Запустить симуляцию");
    System.out.println("2. Остановить симуляцию");
  }
}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**5. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо.

- Класс может быть преобразован в record без потери функционала.

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**6. class SimulationMap**

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Найти пустую случайную координату  
🔸 Подсчитать количество существ определенного типа  
🔸 Вернуть список всех креатур  
🔸 Подсчитать количество свободных клеток

Наверное, для проекта в целом полезно иметь метод, который находит случайную пустую координату.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно искать случайную пустую координату.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Coordinates> getAllCreaturesCoordinates() 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getAllBirdsCoordinates()`.

В данном случае, искать координаты креатур должен тот класс, в интересах которого этот функционал используется: `MoveCreaturesAction`.

- Никогда не возвращай null
```java
private final Map<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  validateCoordinates(coordinates);
  return entities.get(coordinates);  <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

+ 👍 Хорошо, что при операциях с координатой, эта координата валидируется-
если она не в пределах карты, бросается исключение 
```java
public Entity getEntity(Coordinates coordinates) {
  validateCoordinates(coordinates);  <-- БРОСАЕТ ИСКЛЮЧЕНИЕ ЕСЛИ КООРДИНАТА НЕ В ПРЕДЕЛАХ КАРТЫ
  return entities.get(coordinates);
}
```

**7. class PathFinder**

- Соблюдай требования утилитных классов.

Это утилитный класс, он содержит только статические методы.  
Утилитные классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- Этот класс не ищет путь.

Путь это последовательность точек от точки старта до точки финиша.  
Этот класс не ищет путь. Его единственный публичный метод возвращает одиночную координату.  
То есть, класс ищет не путь, а первый шаг этого пути. Это тоже хорошо, но не то, что нужно:  
```java
public static Coordinates findPath(Coordinates currentCreatureCoordinate, SimulationMap map, Predicate<Entity> targetCondition) 
```

Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритма BFS или A*.  
В случае алгоритма BFS- до точки, в которой находится существо нужного класса
```java
public class BfsPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

Или поиск должен искать путь между точками старта и финиша, согласно алгоритма A*
```java
public class AstarPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates from, Coordinates to) {...}
}
```

Так же, поиск пути по алгоритму A* может использовать ту же сигнатуру, что и BFS.  
Тогда поиск пути в нем интерпретируется не как поиск пути между двумя точками, а как поиск цели и прокладывание пути к ней.  
Например: 
```java
public class AstarPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
    //сначала находит координату цели
    //потом прокладывает путь по алгоритму A* между точкой стартой и точкой цели
  }
}
```
Плюсы- унификация сигнатуры разных алгоритмов поиска для полиморфизма.  
То есть, можно сделать семейство родственных классов поиска, объединенных общим интерфейсом.

+ 👍 Хорошо, что поиск цели происходит через предикат.  
Благодаря этому у существ есть потенциальная возможность есть более одного вида еды
```java
public static Coordinates findPath(..., Predicate<Entity> targetCondition) {...}
```

- Повторяющиеся действия делай через циклы
```java
Coordinates oneStepRightNeighbor = new Coordinates(currentX + 1, currentY);
Coordinates oneStepUpNeighbor = new Coordinates(currentX, currentY + 1);
Coordinates oneStepLeftNeighbor = new Coordinates(currentX - 1, currentY);
Coordinates oneStepDownNeighbor = new Coordinates(currentX, currentY - 1);

addNeighborIfValid(oneStepRightNeighbor, currentCoordinate, queue, pathRecreatingMap, map, targetCondition);
addNeighborIfValid(oneStepUpNeighbor, currentCoordinate, queue, pathRecreatingMap, map, targetCondition);
addNeighborIfValid(oneStepLeftNeighbor, currentCoordinate, queue, pathRecreatingMap, map, targetCondition);
addNeighborIfValid(oneStepDownNeighbor, currentCoordinate, queue, pathRecreatingMap, map, targetCondition);

//ПРАВИЛЬНО:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(new Coordinates(1, 0), List.of(new Coordinates(-1, 0), ...); 

for(Coordinates shift : SHIFT_COORDINATES) {
  Coordinates coordinates = new Coordinates(current.getX() + shift.getX(), current.getY() + shift.getY());
  addNeighborIfValid(coordinates, current, queue, pathRecreatingMap, map, targetCondition);
}
```

- Никогда не возвращай null
```java
public static Coordinates findPath(Coordinates currentCreatureCoordinate, SimulationMap map, Predicate<Entity> targetCondition) {
  //...
  return null;
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Нейминг

Поиску без разницы на креатур, кактусы или черепах. Он просто ищет путь между двумя точками
```java
public static Coordinates findPath(Coordinates currentCreatureCoordinate, SimulationMap map, Predicate<Entity> targetCondition)

//ПРАВИЛЬНО:
public static Coordinates findPath(Coordinates start, SimulationMap map, Predicate<Entity> targetCondition)
```
И проблема тут не только в названии, она концептуальнее.  
Зная про "креатур" даже на уровне нейминга, класс поиска берет на себя ответственность знать больше, чем ему положено знать согласно его SRP.

**8. abstract class Entity и его простые наследники Tree/Rock/Grass**

+ 👍 Идеально
```java
public abstract class Entity {
}

public class Tree extends Entity {
}
```

**9. abstract class Creature extends Entity**

- Креатура должна сама искать свои текущие координаты
```java
public abstract void makeMove(Coordinates coordinates, SimulationMap map);

//ПРАВИЛЬНО:
public abstract void makeMove(SimulationMap simulationMap);
```
Креатура все равно в метод хода принимает `SimulationMap`, поэтому может спросить у карты свое текущее местоположение.  

Если креатура будет принимать свое текущее местоположение извне, а не получать самостоятельно из карты, то это потенциально может привести к багам.  
Потому что креатура не может гарантировать, что пришедшая извне координата *действительно* соответствует ее текущему местоположению.  
Слепая вера клиенту тут вредна.

- Тут можно уйти в минуса
```java
public void takeDamage(int damage) {
  HP = HP - damage;
}

//ПРАВИЛЬНО:
public void takeDamage(int damage) {
  hp = hp - damage;
  if(hp < 0) {
    hp = 0;
  }
}
```

- Нарушение инкапсуляции.

Публичными должны быть только те методы, которые предназначены для использования клиентами.  
Методы, предназначенные для использования потомками, должны быть `protected`.  
Остальные методы в классах должны быть `private`
```java
public abstract void makeMove(Coordinates coordinates, SimulationMap map);  <-- ПОТОМКИ ТУТ ВЫЗЫВАЮТ makeStep(...)

public void makeStep(Coordinates from, Coordinates to, SimulationMap map) {...}

//ПРАВИЛЬНО:
public abstract void makeMove(Coordinates coordinates, SimulationMap map);  <-- ПОТОМКИ ТУТ ВЫЗЫВАЮТ makeStep(...)

protected void makeStep(Coordinates from, Coordinates to, SimulationMap map) {...}
```

**10. class Herbivore extends Creature**

Сделай на уровне Creature метод, который определяет еду
```java
public class Herbivore extends Creature {

  @Override
  public void makeMove(Coordinates currentCoordinates, SimulationMap map) {
    for (int i = 0; i < this.getSpeed(); i++) {
      Coordinates nextStep = PathFinder.findPath(..., entity -> entity instanceof Grass);
      Entity checkingEntity = map.getEntity(nextStep);
      if (checkingEntity instanceof Grass) {...}
    }
  }
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
    //...
    public abstract boolean isFood(Entity entity);
}

public class Herbivore extends Creature {

  @Override
  public void makeMove(Coordinates currentCoordinates, SimulationMap map) {
    for (int i = 0; i < this.getSpeed(); i++) {
      Coordinates nextStep = PathFinder.findPath(..., this::isFood);
      Entity checkingEntity = map.getEntity(nextStep);
      if (isFood(checkingEntity)) {...}
    }
  }

  @Override
  public boolean isFood(Entity entity) {
    return entity instanceof Grass;
  }
}
```

Такой метод среди прочего легко может наделить существ способностью есть более одного вида еду. Например:
```java
public boolean isFood(Entity entity) {
  return entity instanceof Grass || entity instanceof Яблоко;
}
```

- Нарушение DRY.

Много общего кода совершения хода у Herbivore и Predator.  
Этот пункт есть в чеклисте ТЗ:
```java
Чеклист для самопроверки #
Дублирование кода между классами Herbivore и Predator
```
Общий код выноси в общего предка, в данном случае- в Creature.

**11. class ConsoleRenderer**

+ 👍 Спрайты существ хранятся здесь, а не берутся из самих существ, это хорошо.

- Нарушение SRP.

В одном и том же методе распечатывается и карта и меню для этой карты
```java
 public void render(SimulationMap map, int turnCounter) {
  for (int y = 0; y < map.getHeight(); y++) {
   for (int x = 0; x < map.getWidth(); x++) {
     //тут распечатывается карта
   }
   
  //ТУТ РАСПЕЧАТЫВАЕТСЯ МЕНЮ:
  System.out.println("Счетчик ходов: " + turnCounter);
  System.out.println("1. Запустить симуляцию");
  //...
}
```

Что будет, если изменится количество команд? Язык интерфейса?  
Или даже просто захочется написать весь текст меню БОЛЬШИМИ БУКВАМИ?  
Придется изменить класс. То есть, распечатка меню здесь это лишняя причина для изменения класса- нарушение SRP.

В *этом* рендерере нужно оставить только распечатку карты.

- Метод делает несколько дел на одном уровне абстракции.  
Вводи вспомогательные методы
```java
public void render(SimulationMap map, int turnCounter) {
  for (int y = 0; y < map.getHeight(); y++) {
    for (int x = 0; x < map.getWidth(); x++) {

      Entity entity = map.getEntity(new Coordinates(x, y));

      switch (entity) {
        case Grass ignored -> System.out.print(GRASS_EMOJI);
        case Rock ignored -> System.out.print(ROCK_EMOJI);
        //...
      }
    }
  }
  //...
}    

//ПРАВИЛЬНО:
public void render(SimulationMap simulationMap, int turnCounter) {
  for (int y = 0; y < simulationMap.getHeight(); y++) {
    for (int x = 0; x < simulationMap.getWidth(); x++) {
      
    Coordinates coordinates = new Coordinates(x, y);
    if(simulationMap.isEmpty(coordinates)) {
      //распечатать землю
    } else {
      Entity entity = simulationMap.getEntity(coordinates);
      String sprite = toSprite(entity);
      System.out.print(sprite)
    }
  }
  //...
}   

private String toSprite(Entity entity) {...}
```

**12. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(SimulationMap map);
}
```

**13. Классы реализации Action**

👍 В целом ок.

**14. class Simulation**

- Нарушение конвенции кода- константы должны стоять выше остальных полей.

- Нарушение DI. Класс не должен сам себя конструировать.  
В данном случае класс конструирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```java
public class Simulation {
  //...

  public Simulation() {
    this.map = new SimulationMap(MAP_HEIGHT, MAP_WIDTH);
    //...
  }
}

//ПРАВИЛЬНО:
public class Simulation {
  //...
  
  public Simulation(SimulationMap simulationMap) {
    this.simulationMap = simulationMap;
    //...
  }
}
```
Таким образом нельзя создать несколько игровых конфигураций с разными картами.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Начальная инициализация должна происходить не в методе `startSimulation()`, а в конструкторе.

- Неправильное использование потоков в проекте.

Внутри класса Simulation не должно быть никакой работы с потоками
```java
public void startSimulation() {
  synchronized (this) {
    for (Action action : initActions) {
      action.execute(map);
    }
    renderer.render(map, turnCounter);
    while (true) {
      try {
        this.wait();
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      }
      while (isRunning) {
        nextTurn();
        sleep();
      }
    }
  }
}
```

Этот класс не должен управлять потоками.  
Наоборот, какие-то потоки должны управлять этим классом и дергать за его публичные методы, которые указаны в ТЗ:
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

Это должен быть такой класс, который можно будет не только запускать из потоков.  
Но и запустить без потоков просто в бесконечном цикле и он должен(бесконечно) работать:
```java
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);
    simulation.startSimulation(); //бесконечная симуляция без потоков 
  }
}
```

Для этого класс должен выглядеть примерно так:
```java
public class Simulation {
  //...

  public void nextTurn() {
    //выполняет все turnActions
    //печатает карту
    //печатает сопроводительную информацию: n.хода и прочее
  }

  public void startSimulation() {
    isRunning = true;
    while(isRunning) {
      nextTurn();
      sleep();
    }
  }

  private void sleep() {
    try {
      Thread.sleep(SLEEP_TIME);
    } catch (InterruptedException e) {...}
  }

  public void pauseSimulation() {
    //останавливает бесконечный цикл
  }
}
```

Когда мы добавляем в программу потоки, то поток должен принимать команды от юзера и дергать `Simulation` за ее методы:   
`nextTurn()`, `startSimulation()` и `pauseSimulation()`.

Для тренировки можно сделать два Main'a в проекте.  
По запуску одного симуляция должна работать так, как сейчас- с возможностью делать паузу/пуск.

А другой Main вообще без использования потоков, например, такой:
```java
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);
    simulation.startSimulation(); //бесконечная симуляция без потоков 
  }
}
```

## ВЫВОД

В целом, замечаний меньше, чем обычно.  
Работу с потоками нужно переосмыслить.

Посмотреть ролики Немчинского про SOLID, прежде всего про SRP и OCP.  
Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.155(329)  
#ревью #симуляция 