https://github.com/ppixiss/simulation  
[Анастасия]

Есть над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализована пауза/пуск во время работы
+ 👍 Возможность конфигурирования карты через меню
+ 🚀 Интересная интерпретация- водный мир  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim159/img0.png) 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Если в проекте есть класс `WorldMap`, то все переменные с именем, включающим это название, должны быть экземплярами этого класса.  

Когда разные концепции называются одним и тем же именем, это приводит к путанице
```java
public class WorldMap {
  private final HashMap<Position, Entity> worldMap = new HashMap<>();
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private final HashMap<Position, Entity> entities = new HashMap<>();
  //...
}
```

- Название обманывает.

Метод не возвращает пустую случайную координату.  
Он возвращает пустую случайную *свободную* координату
```java
public static Position getRandomPosition(WorldMap worldMap) {
  //...
  if (worldMap.isCellEmpty(position)) {
    return position;
  }
}
```

- Используй стандартные названия, это ширина и высота
```java
public class WorldMap {
  public static final int HORIZONTAL_SIZE = 12;
  public static final int VERTICAL_SIZE = 12;
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  public static final int WIDTH = 12;
  public static final int HEIGHT = 12;
  //...
}
```

- Это не сеттер.

Сеттер должен устанавливать значение какого-то конкретного поля.  
А этот метод не устанавливает конкретное существо, он добавляет существо в группу других существ:
```java
private final HashMap<Position, Entity> worldMap = new HashMap<>();  //<K, V>

public void setEntity(Position position, Entity entity) {
  worldMap.put(position, entity);
}

//ПРАВИЛЬНО:
public void putEntity(Position position, Entity entity) //или addEntity(...)
```

Когда есть какое-то ОДНО существо, то сеттер корректен. Условный пример:
```java
private Entity entity;

void setEntity(Entity entity) {
  this.entity = entity;  <-- Правильно, сеттер устанавливает конкретное значение
}
```

Когда существо добавляется к другим, это не сеттер. Условный пример:
```java
private List<Entity> entities;

void addEntity(Entity entity) {
  entities.add(entity);  <-- Это не сеттер, потому что добавляет новый объект в список других
}
```

- Одной концепции- одно название
```java
public void moveEntity(Position currentPosition, Position nextCell, Entity entity)

//ПРАВИЛЬНО:
public void moveEntity(Position currentPosition, Position nextPosition, Entity entity)
```

Хотя конкретно тут лучше просто:
```java
public void moveEntity(Position current, Position next, Entity entity)
```

- Избыточно
```java
public class WorldValidator {
  public static boolean isWorldValid(WorldMap worldMap) {...}
  //...
}

//ПРАВИЛЬНО:
public class WorldValidator {
  public static boolean isValid(WorldMap worldMap) {...}
  //...
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```java
HashMap<Position, Entity> worldMap = new HashMap<>();

//ПРАВИЛЬНО:
Map<Position, Entity> worldMap = new HashMap<>();
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**3. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется**  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без else будет выглядеть читабельней
```java
if (isPreyEaten(target)) {
  worldMap.moveEntity(currentPosition, nextCell, this);
  return;
} else {
  attack(target);
  count++;
}

//...
if (entity instanceof Island) {
  return ISLAND;
} else {
  return WAVE;
}

//ПРАВИЛЬНО:
if (isPreyEaten(target)) {
  worldMap.moveEntity(currentPosition, nextCell, this);
  return;
} 
attack(target);
count++;

//...
if (entity instanceof Island) {
  return ISLAND;
} 
return WAVE;
```

**4. class Position**

- При прочих равных, нужно использовать примитивный тип, а не класс-обертку
```java
public class Position {
  private Integer x;
  private Integer y;
  //...
}

//ПРАВИЛЬНО:
public class Position {
  private int x;
  private int y;
  //...
}
```
Для применения класса-обертки над примитивным типом данных, должна быть какая-то причина.  
Здесь использование обертки не дает ничего большего по сравнению с примитивом.  
*Блох "Java. Эффективное программирование", изд.3, гл.9.5*
```java
"Предпочитайте примитивные типы упакованным примитивным типам" - Блох.
```

- Нарушение SRP, Low Coupling.

Единая ответственность координаты- хранить данные для идентификации точки в двумерном пространстве. 
В данном случае- хранить и выдавать значения x и y.
Для выполнения этой ответственности координате не нужен метод, который находит случайную координату в карте
```java
public static Position getRandomPosition(WorldMap worldMap)
```
Класс Координата может отлично жить без этого метода. Определи тот класс, который не может жить без этого метода и перенеси его туда.

Нарушение Low Coupling состоит в том, класс Координата сейчас зависит от класса Карта.  
Вернее, получается циклическая зависимость: Карта зависит от Координаты и наоборот.

Это неправильное направление зависимостей.  
На самом деле только Карта должна зависеть от координаты- то есть, использовать координату.  
Координата не должна знать вообще ни про какие пользовательские классы, она должна быть максимально простых классом.  

- Для координаты лучшим решением будет использовать record- это подойдет для симуляции и любой другой игры на прямоугольной доске(Морской бой, Шахматы и т.д.).  
Вот идеальная координата:
```java
public record Coordinates (int row, int column) {
}
```
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Единственный нюанс- record'ы immutable, а значит их значения могут быть только для чтения и сеттеры для них сделать невозможно.
Но это для координаты даже лучше- нет особых оснований делать поля координаты row и column изменяемыми.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

*Блох "Java. Эффективное программирование", изд.3, гл.4.3., "Минимизируйте изменяемость"*

**5. class WorldMap**

- Карта должна иметь возможность быть созданной с произвольными размерами.

Карта содержит в себе фиксированные размеры, поэтому становится неуниверсальной- нельзя создать игровые конфигурации с картами разных размеров
```java
public class WorldMap {
  public static final int HORIZONTAL_SIZE = 12;
  public static final int VERTICAL_SIZE = 12;
  //...
}
```
Фиксированные размеры могут быть у шахматной доски(8x8) или игры крестики-нолики(3x3).  
То есть в тех играх, где размер игрового поля жестко фиксирован правилами.  
Здесь размеры должны приниматься в конструктор, чтобы можно было создавать карту произвольных размеров.

- Никогда не возвращай null
```java
private final HashMap<Position, Entity> worldMap = new HashMap<>();  

public Entity getEntityAt(Position position) {
  return worldMap.get(position);  <-- вернет null если position нет в position
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность 

Сейчас в карту можно вставить существо на координату, выходящую за размер карты.  
Если координата некорректна(находится вне пределов карты), нужно бросать исключение:
```java
public boolean isCellEmpty(Position position) {
  return !worldMap.containsKey(position);
}

//ПРАВИЛЬНО:
public boolean isCellEmpty(Position position) {
  validate(position);  <-- Если координата вне пределов карты, бросает исключение
  return !worldMap.containsKey(position);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, то бросается исключение.

 Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public boolean hasPrey() {
  //...
  if (entity instanceof Prey) {
    return true;
  }
  //...
}
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `hasBird()`.

Если клиенту надо узнать, есть ли Жертвы в Карте, то пусть он сам это узнает: качает всех существ из карты и смотрит, есть ли среди них зайцы. 

**6. class MapPathFinder**

- Соблюдай требования утилитных классов. 

Это утилитный класс, он содержит только статические методы.  
Утилитные классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- Нарушение дизайна ООП.

```java
public class MapPathFinder {
  static Position target;
  //...
}
```
Из-за этого статического поля нельзя в проекте использовать больше одного экземпляра этого класса.  
Потому что все они будут одновременно вносить изменения в значение этого поля.

В ООП программе никогда не используй статические ИЗМЕНЯЕМЫЕ поля.  
Можно использовать только константы- статические неизменяемые поля
```java
public class MapPathFinder {
  static Position target;

  public static List<Position> computePathToTarget(Position position, WorldMap worldMap) {...}
}

//ПРАВИЛЬНО:
public class MapPathFinder {
  public static List<Position> computePathToTarget(Position position, WorldMap worldMap, Position target;) {...}
}
```

- Нарушение SRP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем опроса Creature про то, шо оно ест
```java
public static List<Position> computePathToTarget(Position position, WorldMap worldMap) {...}

private static boolean isEatable(Position position, Creature creature, WorldMap worldMap) {
  Entity target = worldMap.getEntityAt(position);
  return target != null && creature.canEat(target);  <-- НАРУШЕНИЕ SRP, класс поиска самостоятельно опрашивает креатур об их меню 
}

//ПРАВИЛЬНО:
public List<Position> find(WorldMap worldMap, Position from, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```

- Нарушение SRP.

Если класс предназначен для поиска пути, то он должен ТОЛЬКО искать путь.  
Путь это последовательность точек от старта до финиша.

В классе должен быть только один публичный метод- тот, который возвращает путь
```java
public class MapPathFinder {  <-- ЭТО КЛАСС ПОИСКА ПУТИ
  static Position target;

  public Position findNextCell(List<Position> path) {...}  <-- Этот метод НЕ ИЩЕТ путь

  public static List<Position> computePathToTarget(Position position, WorldMap worldMap) {...}  <-- Этот метод ИЩЕТ путь
}
```

**7. abstract class Entity и его простые наследники Coral/Island/Wave**

+ 👍 Идеально
```java
public abstract class Entity {
}

public class Coral extends Entity {
}
```

**8. abstract class Creature extends Entity**

- Нарушение ООП дизайна в наследовании.

Нарушение чеклиста ТЗ:
```java
Чеклист для самопроверки #
Дублирование кода между классами Herbivore и Predator
```

Наследники Creature имеют общие поля и поведение, эти общие данные наследников должны храниться здесь
```java
public abstract class Creature extends Entity {

  public abstract void makeMove(WorldMap worldMap, Position currentPosition);

  public abstract boolean canEat(Entity entity);
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {

  private final int speed;
  private final Class<? extends Entity> food;

  public void Creature(int speed, Class<? extends Entity> food) {...}


  public void makeMove(WorldMap worldMap, Position currentPosition) {
    //общий для всех потомков код совершения хода
  }

  public boolean canEat(Entity entity) {
    return entity.getClass() == food;
  };
}
```

Код в методах `makeMove()` у потомков практически одинаков.  
Общий код совершения хода нужно вынести сюда, на уровень `Creature`.  
В потомках должна быть только та часть совершения хода, которая отличается у потомков.

- Креатура должна сама искать свои текущие координаты
```java
public abstract void makeMove(WorldMap worldMap, Position currentPosition);

//ПРАВИЛЬНО:
public void makeMove(WorldMap worldMap) {
  Position current = worldMap.getPosition(this);
  List<Position> path = PathFinder.find(worldMap, current, food);
  //...
}
```
Креатура все равно в метод хода принимает `WorldMap`, поэтому может спросить у карты свое текущее местоположение.  

Если креатура будет принимать свое текущее местоположение извне, а не получать самостоятельно из карты, то это потенциально может привести к багам.  
Потому что креатура не может гарантировать, что пришедшая извне координата *действительно* соответствует ее текущему местоположению.  
Слепая вера клиенту тут вредна.

- Идея `Action` здесь не осмыслена. 

Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.  
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.  
Action'ы, изложенные в ТЗ, это вариант реализации паттерна Command. 

Сейчас `Actions` у тебя это просто разные классы, которые не имеют друг с другом ничего общего, кроме названия.  
Большая часть из них вообще утилиты
```java
public class WaveSpawnerAction {
  //...
  public void spawnWave(WorldMap worldMap) {...}
}

public class CreatureMovementAction {
  //...
  public void makeCreaturesMove(WorldMap worldMap) {...}

  public static Position getNextCell(Position position, List<Position> shortestWay, MapPathFinder pathFinder) {...}
}

public class EntityPlacerAction {
  public static void placeEntities(List<Entity> entities, WorldMap worldMap) {...}
```

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.  
В каждом экшене должен быть только один публичный метод(не публичных может быть сколько угодно).  
Это вариация паттерна Command- экшены должны быть родственны и одинаково использоваться через полиморфизм.  
Примерно так
```java
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

List<Action> actions = List.of(new ДатьСигаретуВсемЗайцамAction(), new ХодитьAction());
for(Action a: actions) {
  a.execute(карта);
}

/*
Результат: программа обойдет всю карту, найдет всех креатур и даст им пинка, чтобы они побежали.
А каждому зайцу предварительно даст сигарету.
*/
```

**9. class SimulationMenu**

- Нарушение DRY.

Просто два ИДЕНТИЧНЫХ метода.
```java
public static boolean shouldStartSimulation(Scanner scanner) {...}

public static boolean shouldRestartSimulation(Scanner scanner) {...}
```

- Если нужно много печатать цветным, сделай цветной принтер
```java
private static final String START = "s";
private static final String EXIT = "e";
private static final String CONTINUE = "Enter";
private static final String RESET = "\u001B[0m";
private static final String ITALIC = "\u001B[3m";
private static final String PURPLE = "\u001B[35m";

//...
printColor(Color.PURPLE, Type.ITALIC, " WELCOME TO SIMULATION! ");

private static void printColor(Color color, Type type, String message) {...}

//ПРАВИЛЬНО:
private enum Color {
  RESET("\u001B[0m"),
  PURPLE("\u001B[35m"),
  //...
  private final String code;
  //...
}

private enum Type {
  ITALIC("\u001B[3m"),
  BOLD(...),
  //...
  private final String code;
  //...
}
```

**10. class MapConsoleRenderer**

- Нарушение инкапсуляции.

Публичным тут должен быть только метод
```java
public void render(WorldMap worldMap)
```

- Объявляй переменные там, где они используются.

Минимизируй область видимости локальных переменных
```java
Position position = new Position();
for (int y = 0; y < WorldMap.VERTICAL_SIZE; y++) {
  for (int x = 0; x < WorldMap.HORIZONTAL_SIZE; x++) {
    position.setX(x);
    position.setY(y);
    //...
  }
}

//ПРАВИЛЬНО:
for (int y = 0; y < WorldMap.VERTICAL_SIZE; y++) {
  for (int x = 0; x < WorldMap.HORIZONTAL_SIZE; x++) {
    Position position = new Position(x, y);
    //...
  }
}
```
*Блох, "Java. Эффективное программирование", изд.3, гл.9.1*  

- Название метода должно соответствовать тому, что метод делает.

Если метод должен распечатать существо, то он должен распечатать существо.  
А не *взять существо из карты* и распечатать его
```java
public void renderEntity(Position position, WorldMap worldMap) {...}

//ПРАВИЛЬНО:
public void renderEntity(Entity entity) {...}
```
*"ЧК", гл.3, "Правило одной операции", "Один уровень абстракции"*

**11. class Simulation**

- Нарушение DI. 

Класс должен принимать необходимые зависимости в конструктор.  
Экземпляр карты должен являться подробностью внутреннего устройства класса `Simulation`
```java
public class Simulation {
  //...

  public void start(WorldMap worldMap) {...}
  //...
}

//ПРАВИЛЬНО:
public class Simulation {
  private final WorldMap worldMap
  //...

  public Simulation(WorldMap worldMap){...}

  public void startSimulation() {...}
  //...
}
```

- Action'ы используются неправильно- они здесь объявляются и используются индивидуально
```java
private final CreatureMovementAction movementAction = new CreatureMovementAction();
private final WaveSpawnerAction waveSpawnerAction = new WaveSpawnerAction();

private void generateValidWorld(WorldMap worldMap) {
  EntityPlacerAction.placeEntities(entities, worldMap);
  //...
}

private void nextTurn(WorldMap worldMap) {
  movementAction.makeCreaturesMove(worldMap);
  waveSpawnerAction.spawnWave(worldMap);
}
```

На самом деле экшены нужно объявлять не индивидуально, а группой и дальше использовать через полиморфизм.  
Примерно так:
```java
private final List<Action> initActions = List.of(new UnoAction());
private final List<Action> turnActions = List.of(new DosAction(), new TresAction(), new MoveAction());

public Simulation(...) {
  //...
  executeActions(initActions);  //выполнение экшенов на старте
}

public void nextTurn() {
  //...
  executeActions(turnActions);  //выполнение экшенов на каждом ходе
}

private void executeActions(List<Action> actions) {
  for(Action a : actions) {
    a.execute(карта);
  }
}
```
Для этого экшены должны соответствовать паттерну `Command`.

Даже если сейчас есть только один init-экшен и один turn-экшен, все равно их нужно хранить группой- например, в листе.  
Это позволит при необходимости быстро добавить в эти группы новые экшены, которые так же будут использоваться через полиморфизм.

- Неправильное использование потоков в проекте.

Внутри класса Simulation не должно быть никакой работы с потоками
```java
inputThread = new Thread(...);

while (running && !Thread.currentThread().isInterrupted()) {...}
```

Этот класс не должен управлять потоками.  
Наоборот, какие-то потоки из другого класса должны управлять этим классом и дергать за его публичные методы, которые указаны в ТЗ:
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

  public Simulation(...) {
    //выполняет все initActions
  }

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

**Для тренировки можно сделать два Main'a в проекте.**    
По запуску одного симуляция должна работать так, как сейчас- с возможностью делать паузу/пуск.  
А другой Main такой:
```java
public class MainWithoutThreads {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);
    simulation.startSimulation(); //бесконечная симуляция без потоков 
  }
}
```

- Нарушение SRP. Метод не должен завершать работу программы через exit()
```java
private void startInputListener() {
  //...
  System.exit(0);
}
```
Каждый метод и класс имеют право завершать только свою работу. Например, через return.  
Потому что методы и классы не должны знать логику работу более высоких слоев программы, у которых могут быть свои планы на тему того, когда и почему нужно завершать работу программы.

Кроме того, при выходе через `exit()` могут не закрыться некоторые ресурсы программы.

- Нарушение SRP.

Этот класс не должен запустить диалог "Введите размеры карты".  
Потому что `Simulation` должен только управлять работой системы.  
А создание карты это создание системы.

Диалог "Введите размеры карты" должен происходить на уровень выше.  
В данном случае- в Main. 

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*  

**Совет:**  
Для тренировки и понимания некоторых ООП концепций, создай несколько майн-классов.  
Один пусть запускает игру так, как это сейчас- с конфигурированием карты через диалог с юзером.  
Второй майн пусть сразу запускает игру сразу с картой размером 10x10.
Третий пусть сразу запускает игру картой размером 10x10, но без использования потоков и команд старт/стоп.

**12. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

- Плохо то, что не инжектит необходимые зависимости в Симуляцию.
Main должен выглядеть примерно так:
```java
public static void main(String[] args) {
  int width = inputWidth();  //получить размеры карты через диалог с юзером
  int height = inputHeight();

  WorldMap worldMap = new WorldMap(20, 20);  

  Simulation simulation = new Simulation(worldMap);
  SimulationManager simulationManager = new SimulationManager(simulation);  //класс который в себе реализует многопоточность
  simulation.execute();
}
```
*Про особенности использования main ищи соответствующую главу в "Чистом коде"*  

## ВЫВОД

Переделать экшены, чтобы они были сделаны в ООП стиле.  
Подробнее разберись с ними- посмотри ролики на ютубе про паттерн "Command", чтобы получить о нем общее представление.

Работа с потоками неправильно встроена в архитектуру программы.

Посмотреть на ютубе ролики Немчинского про SOLID.

n.159(338)  
#ревью #симуляция 