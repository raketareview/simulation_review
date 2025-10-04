https://github.com/Metty1337/Simulation  
[Anton]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Длинные команды: 
```
List of available commands: unpause, stop.
```
Не нужно излишне напрягать юзера, делай короткие команды, например: 0/1 или p/s. 

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Реализовано пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Пакеты нужно писать стилем lower_snake
```
gamemap

//ПРАВИЛЬНО:
game_map
```

- Избыточно. Естественно, мы ищем короткий путь
```
List<Coordinates> getShortestPath(...)

//ПРАВИЛЬНО:
List<Coordinates> getPath(...)
```
Название сбивает с толку- я даже поискал методы, которые ищут *другие* пути. 
Естественно, других методов по поиску путей в классе нет.

- В циклах индекс имя "i" принято ассоциировать с типом `int`. 
Поэтому, собственно, оно и "i". Здесь же индекс- объект координаты
```
for (Coordinates i = end; i != start; i = parent.get(i))

//ПРАВИЛЬНО:
for (Coordinates c = end; c != start; c = parent.get(i))
```

- Название должно объяснять суть явления. Эта мапа хранит не предка, а прямой путь до цели. 
Этот функционал нужно отразить в названии
```
Map<Coordinates, Coordinates> parent = new HashMap<>();
```

- Не давай названия, которые вводят в заблуждение. Под конструктором в ООП понимается вполне конкретная штука и это точно не она
```
private final Supplier<? extends Entity> constructor;

//ПРАВИЛЬНО:
private final Supplier<? extends Entity> creator;
```

- Название вводит в заблуждение. Этот енам не хранит типы существ, это фабрика существ
```
enum EntityType

//ПРАВИЛЬНО:
enum EntityFactory
```

- Однобуквенное имя "e" принято использовать в блоке `catch` для названия Exception'ов 
```
for (EntityType e : EntityType.values())

//ПРАВИЛЬНО:
for (EntityType type : EntityType.values())
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

В некоторых классах конструктор стоит ниже всех остальных методов. 
Кроме того, что это противоречит конвенции, это плохо тем, что складывается впечатление, будто бы в классе есть только конструктор по умолчанию. 
А эти конструкторы часто не по умолчанию. Например, они приватные- и это хотелось бы сразу видеть.

**3. Утилитные классы**

👍 Утилитные классы здесь организованы правильно: они `final` и имеют приватный конструктор.

**4. record Coordinates(int column, int row)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**5. class GameMap**

- Неправильное использование константного класса, нарушение Low Coupling.

Свои размеры Карта должна только принимать в конструктор. Иначе создается ненужная зависимость на другой класс- то есть, на класс констант
```
public GameMap() {  <-- УБРАТЬ
  this(GameMapConfig.HEIGHT, GameMapConfig.WIDTH);
}

public GameMap(int height, int width) {...}  <-- ОСТАВИТЬ
```

- Особые случаи обрабатывай сразу
```
if (isCoordinatesValid(coordinates)) {
  <some code>
} else {
  throw new IllegalArgumentException(coordinates + " is not valid");
}

//ПРАВИЛЬНО:
if (!isCoordinatesValid(coordinates)) {
  throw new IllegalArgumentException(coordinates + " is not valid");
} 
<some code>
```

+ 👍 При всех операциях с координатами, выполняется валидация- проверяется, что она входит в пределы карты. Если не входит- бросает исключение.
Это хорошо.

- Дублирующийся код при валидации
```
public void addEntity(Coordinates coordinates, Entity entity) {
  if (isCoordinatesValid(coordinates)) {
    //...
  } else {
    throw new IllegalArgumentException(coordinates + " is not valid");
  }
}

public Entity getEntity(Coordinates coordinates) {
  if (isCoordinatesValid(coordinates)) {
    //...
  } else {
    throw new IllegalArgumentException(coordinates + " is not valid");
  }
}

//ПРАВИЛЬНО:
public void addEntity(Coordinates coordinates, Entity entity) {
  validate(coordinates);
  //....  
}

public Entity getEntity(Coordinates coordinates) {
  validate(coordinates);
  //....  
}

private void validate(Coordinates coordinates) {
  if (!isCoordinatesValid(coordinates)) {
    throw new IllegalArgumentException(coordinates + " is not valid");
  }
}
```

- Сейчас если спросить у карты, пустая ли координата (+100500, -100500), то карта скажет, что ЗАНЯТА. 
А правильный ответ- такой координаты в карте вообще нет
```
public boolean isSquareEmpty(Coordinates coordinates) {
  if (isCoordinatesValid(coordinates)) {
    return !entities.containsKey(coordinates);
  }
  return false;
}
```
Провереяем:
```
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap();
    Coordinates coordinates = new Coordinates(+100500, -100500);
    System.out.println("Координата пустая: " + gameMap.isSquareEmpty(coordinates));
  }
}

//РЕЗУЛЬТАТ:
Координата пустая: false
```

Метод `isSquareEmpty()` судя по его нвзванию, должен возвращать "TRUE", если координата пустая. 
Сейчас, если у этого метода спросить, свободна ли координата, которая выходит за границу карты, он ответит "FALSE".  
То есть, координата занята.

На самом деле, если клиент в метод передает некорректную координату, это баг алгоритма. 
Его нужно как можно быстрее выявить и устранить. То есть, нужно кинуть исключеие.

- В сообщении исключения принято сначала писать текст сообщения
```
throw new IllegalArgumentException(coordinates + " is not valid");

//ПРАВИЛЬНО:
throw new IllegalArgumentException("not valid: " + coordinates);
```

- При вставке существ в карту *на корректную координату*, программа вылетает:
```
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(30, 30);
    Coordinates coordinates = new Coordinates(25, 25);
    Entity entity = new Rock();
    gameMap.addEntity(coordinates, entity);
  }
}

//РЕЗУЛЬТАТ:
Exception in thread "main" java.lang.IllegalArgumentException: Coordinates[column=25, row=25] is not valid
  at metty1337.simulation.gamemap.GameMap.addEntity(GameMap.java:37)
  at metty1337.simulation.temp.Main.main(Main.java:14)
```

+ 👍 В целом, с точки зрения SRP класс хороший. 
Он содержит в себе только те методы, которые составляют его единую ответственность.

**6. final class GameMapUtils**

+ 👍 Сюда вынесены вспомогательные методы для работы с картой. 
Хорошо, что они находятся здесь, а не в самой карте.

- Нарушение конвенции кода. Конструктор должен стоять выше остальных методов.

- Для подсчета поголовья существ, метод должен принимать класс, а не его экземпляр
```
int getEntityCount(Entity target, GameMap gameMap)

//ПРАВИЛЬНО:
int getEntityCount(Class<? extends Entity> target, GameMap gameMap)
```
Потому что сейчас контракт метода подразумевает, что ты должен считать количество присутствий конкретного экземпляра в карте, а не всех экземпляров заданного класса.
Но метод реализован так, что считает количество именно экземпляров определенного класса, а не одну персоналию, которую сейчас передают в метод.

**7. class PathFinder**

- Поиск принимает координату цели, к которой нужно проложить путь
```
List<Coordinates> getShortestPath(Coordinates start, Coordinates end, GameMap gameMap)
```
Таким образом, клиент должен *УЖЕ* предварительно найти для себя цель и передать ее координаты в поиск.  
Следовательно, единая ответственость по поиску пути размазывается между несколькими классами: один класс ищет конкретную координату с едой, а класс поиска прокладывает путь к этой еде.  
На саммом деле, класс поиска должен все это делать самостоятельно.

Поиск должен просто искать путь от точки старта до точки, соответствующей заданным условиям.
В данном случае- до точки, в которой находится существо нужного класса.  
Например, так:
```
List<Coordinates> getShortestPath(GameMap gameMap, Coordinates start, Coordinates end, Class<? extends Entity> target)
```
Если для поиска A* нужна координата цели, то метод должен сам найти эту координату:
```
List<Coordinates> getShortestPath(GameMap gameMap, Coordinates start, Coordinates end, Class<? extends Entity> target) {
  Coordinates end = //находит координату цели
  //other code
}
```

- Используй циклы как циклы, а не как не пойми шо. 

От цикла `for(...)` всегда ожидаешь простой итерации по индексу. 
Для всего прочего используй `while`:
```
for (Coordinates i = end; i != start; i = parent.get(i)) {
  path.add(i);
}

//ПРАВИЛЬНО:
Coordinates current = end;
do {
  current = parent.get(current);
  path.add(current);   
} while(current != start);
```
Да, *может быть* здесь цикл `for(...)` кажется компактнее и лаконичнее, но он совершенно точно примерно абсолютно нечитаем.

- Мапа координата-координата обычно при реализации поиска используется как эрзац-заменитель связного списка
```
Map<Coordinates, Coordinates> parent = new HashMap<>();
```
Вместо мапы координата-координата сделай класс связного списка, алгоритм тогда будет выглядеть понятнее.
Например, так:
```
class Node {
  private Coordinate coordinate; 
  private Node parent;
  //...
  public Coordinate getCoordinate() {...}
  public Node getParent() {...}
}
```

- Пожалуй, тут код можно сделать понятнее:
```
private static List<Coordinates> getNeighbors(...) {
  List<Coordinates> possibleNeighbors = List.of(
    new Coordinates(coordinates.column(), coordinates.row() + STEP),
    new Coordinates(coordinates.column() + STEP, coordinates.row()),
    new Coordinates(coordinates.column(), coordinates.row() - STEP),
    new Coordinates(coordinates.column() - STEP, coordinates.row())
  );
  //...
}  

//ЛУЧШЕ:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(
  new Coordinates(0, 1), 
  new Coordinates(1, 0), 
  //...
);

private static List<Coordinates> getNeighbors(...) {
  List<Coordinates> possibleNeighbors = new ArrayList<>();
  for(Coordinates shift : SHIFT_COORDINATES) {
    Coordinates c = add(shift, coordinates); <-- складывает две координаты
    possibleNeighbors(c);  
  }
  //...
}
```

**8. abstract class Entity**

- Нарушение конвенции кода- конструкторы стоят ниже метода.

- Не передавай принудительно `null` в методы и конструкторы, даже если они в состоянии принять null. 
Поля класса и так по умолчанию инициализируется null'ом
```
private Coordinates coordinates;

public Entity(Coordinates coordinates) {
  this.coordinates = coordinates;
}

public Entity() {
  this(null);
}

//ПРАВИЛЬНО:
private Coordinates coordinates;

public Entity(Coordinates coordinates) {...}

public Entity() { <-- ПРОСТО ПУСТОЙ КОНСТРУКТОР
}
```

- Содержит координату. Но координата нужна только тому существу, которое ходит. 
Поэтому entities должны хранить координату только начиная с уровня `Creature`.

- Здесь это излишняя предосторожность. Координата- record, а значит, иммутабельна
```
public Coordinates getCoordinates() {
  return new Coordinates(coordinates.column(), coordinates.row());
}

//ПРАВИЛЬНО:
public Coordinates getCoordinates() {
  return coordinates;
}
```

- Нарушение SRP и Low Coupling.

Существо, по крайней мере на уровне базового класса Entity, не должно ничего знать про класс GameMap- это для Сущности лишняя зависимость.  
Существо не должно ни на каком уровне проверять, пришла ли в нее корректная с точки зрения доски координата или нет- это не ответственность существа
```
public void setCoordinates(Coordinates coordinates) {
  if (GameMap.isCoordinatesValid(coordinates)) {
    this.coordinates = coordinates;
  } else {
    throw new IllegalArgumentException(coordinates + " is not valid");
  }
}

//ПРАВИЛЬНО:
public void setCoordinates(Coordinates coordinates) {
  this.coordinates = coordinates;
}
```
Координата уже валидируется при вставке существа в карту и этого достаточно.

- Нарушение SRP. 

Существо не должно знать, сколько его экземпляров должно быть помещено на карту. 
Это ответственность общеигровой логики
```
public abstract int getSpawnRateValue();
```
Деревья в лесу не предъявляют леснику планы по насаждениям- ему их спускают из лесного ведомства.

**9. Простые наследники Entity: Rock, Tree, Grass**

Неправильное использование конфигурационного класса- существа лезут напрямую в класс конфигурации и читают оттуда данные. 
Это делает класс зависимым от Конфига
```
public class Tree extends Entity {
  private static final int SPAWN_RATE_VALUE = EntityConfig.TREE_SPAWN_VALUE;
  //...
}
```
Все необходимые для себя данные класс должен принимать в конструктор, либо получать через сеттеры.

**10. abstract class Creature extends Entity implements Eatable**

- Креатура не должна вызывать экшены, они придуманы не для этого.  

Логику поедения креатура не должна делегировать третьим классам- что это еще за внешнее пищеварение? 
Этот код должен лежать в самой креатуре
```
public abstract class Creature extends Entity implements Eatable {
  public void makeMove(GameMap gameMap) {
    new TurnEat(this, gameMap.getEntity(targetCoordinates)).execute(gameMap);
    //...
  }
}

public class TurnEat implements Action {...}
```
Экшена поедания вообще не должно быть в проекте.

- Неправильное использование экшена хода.

Как я писал выше, экшены придуманы не для такого использования. 
```
public abstract class Creature extends Entity implements Eatable {
  public void makeMove(GameMap gameMap) {
    new TurnMove(this, this.getFood()).execute(gameMap);
    //...
  }
}

public class TurnMove implements Action {...}
```
Подробнее- в пункте про экшены.

- Нарушение SRP. 

Для единой ответственности по релизации ходячего и едячего существа, этому существу абсолютно не нужно знать перечень всех едящих существ на поляне
```
public static List<Entity> getEdibleEntities()
```

**11. class Herbivore/Predator extends Creature implements Eatable**

- Нарушение SRP.

Чисто логически, на каком основании Волк рожает Зайца, а Заяц- Траву? 
А они именно рожают, то есть создают новые объекты
```
public class Predator extends Creature implements Eatable {
  @Override
  public Entity getFood() {
    return EntityType.HERBIVORE.create();
  }
  //...
}

//ПРАВИЛЬНО:
public class Predator extends Creature implements Eatable {
  private static final Class<? extends Entity> FOOD = HERBIVORE.class;

  @Override
  public Class<? extends Entity> getFood() {
    return FOOD;
  }
  //...
}
```

- Нарушение ООП дизайна в наследовании. Необходимые предку данные нужно инжектить в его конструктор, а не сетить
```
public Predator() {
  super();
  setHp(CreatureConfig.PREDATOR_HP);
}

//ПРАВИЛЬНО:
public Predator() {
  super(CreatureConfig.PREDATOR_HP);
}
```

- Нарушение DRY. Идентичный код в методах `canEat(Entity target)` у Волка и Зайца. 
Общий код выноси в предка.

**12. interface Action**

👍 Идеально
```
public interface Action {
  void execute(GameMap gameMap);
}
```

**13. Action'ы**

Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.  
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.  
Action'ы, изложенные в ТЗ, это вариант реализации паттерна Command. 

Поэтому все экшены должны иметь только один публичный метод `execute()`(не публичных методов могут иметь сколько угодно).

Здесь все экшены, которые имеют более одного публичного метода `execute()`, или вообще не имеют этого метода, 
как `class TurnEat implements Action`, по сути не являются и не могут считаться Action'ами.

Action'ы должны вызываться только в классе Simulation: на старте(initActions) и на каждом ходе(turnActions).

**14. class TurnMove implements Action**

Класс реализует часть логики передвижения существа. Другая часть логики- реализуется в самом существе. 
Таким образом, единое действие, перемещение существа по карте, размазано на два класса.

Эта логика должна лежать либо в самой креатуре и быть реализована в его методе `makeMove(...)`.  
И тогда креатура интерпритируется как сущность с собственной волей.

Либо находиться в отдельном классе, например `TurnMove`, но тогда в самой креатуре не должно быть `makeMove(...)`.
И тогда сущность интерпритируется как сущность без собственной воли, как фигура в шахматах, которую передвигает кто-то еще.

**15. class PopulationRestorer implements Action**

То же самое дробление сущности на два класса: PopulationRestorer и PopulationChecker.

**16. class TurnMoveAllCreatures implements Action**

👍 Идеально

**17. class InitSpawnEntities implements Action**

Дробление одной сущности на несколько классов.  
Судя по всему, код из `SpawnManager` можно смело перенести сюда. 
В любом случае, эти два класса в сумме занимаются одним делом.

**18. class SpawnManager**

Никогда не возвращай null
```
public static Entity entityOrEmptySpawner() {
  if (wouldSquareBeEmpty()) {
    return null;
  }
  //...
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"* 

**19. Классы конфигурации**

Конфигурационные классы здесь часто  используются неправильно- другие классы напрямую лезут в константы вместо того, чтобы получать необходимые этим классам данные в свой конструктор.  
Это делает классы неуниверсальными и зависимыми от класса конфигурации.

Условный пример
```
public final class Settings {
  //приватный конструктор
  public static final int ROOMS_AMOUNT = 5;
}

ПЛОХО:
public static void main(String[] args) {
  House house = new House();
  //oth code
}

public class House {
  private final int roomsAmount;

  public House() {
    this.rooms = Settings.ROOMS_AMOUNT;
  }

  public int getRoomsAmount() {
    return roomsAmount;
  }
}

//ХОРОШО:
public static void main(String[] args) {
  House house = new House(Settings.ROOMS_AMOUNT);
  //oth code
}

public class House {
  private final int roomsAmount;

  public House(int roomsAmount) {
    this.rooms = roomsAmount;
  }

  public int getRoomsAmount() {
    return roomsAmount;
  }
}
```
Про классы констант, конфигурации и их использование я писал [ТУТ](https://t.me/zhukovsd_it_chat/53243/176984)

**20. class MoveCounter**

- Не вижу необходимости в существовании этого класса. 
Он не делает ничего больше, чем делала бы обычная перменная `int moveCounter`.

- Переопределение `toString()` здесь плохо тем, что оно делает модель(а счетчик это модель) зависимым от представления. 
А именно конкретного текста информационного сообщения и конкретного языка UI
```
public String toString() {
  return "Count of moves: " + counter;
}
```

Да, сейчас в программе нет распечатки счетчика типа такого: `System.out.println(moveCounter.toString())`.
Но подобное переопределение тустринг наводит на мысль, что оно сделано для возможности именно такого использования.

В таком простом классе toString может быть упрощенным.
Но я бы советовал всегда делать тустринги по умолчанию, как делает IDE по Alt+Ins и использовать его только для отладки:
```
@Override
public String toString() {
  return "MoveCounter{" +
    "counter=" + counter +
 '}';
}
```
Подробнее про toString() читай тут: 
```
*Блох, "Java эффективное программирование" гл.3.3*  
```

**21. class MessagePrinter**

Хороший класс, который потенциально может отделать логику от представления. 
То есть, от вывода информации в конкретную среду: консоль, COM-порт и т.д.
```
public class MessagePrinter {
  public void printMessage(String message) {
    System.out.println(message);
  }
}
```

Проблема в том, что сейчас толка от этого класса и смысла в нем нет вообще. 
Сейчас этот класс- просто класс-посредник без внятной роли.

Чтобы появился смысл, должен быть интерфейс и разные его реализации. 
Даже, если реализация будет только одна- консольная.  
Примерно, так:
```
public interface MessagePrinter {
  void printMessage(String message);
}

public class ConsoleMessagePrinter extends MessagePrinter {
  @Override
  public void printMessage(String message) {
    System.out.println(message);
  }
}

public class ColorConsoleMessagePrinter extends MessagePrinter {
  private final Color color;

  public ColorConsoleMessagePrinter(Color color) {...}

  @Override
  public void printMessage(String message) {
    System.out.print(color.getCode());
    super.printMessage(message);
    System.out.print(Color.DEFAULT.getCode());
  }
}

public enum Color {
   DEFAULT(""\u001B[0m")") 
   RED("\u001B[31m"),
   //oth's color

   private final String code;
   //...
}
```

И тогда классы через полиморфизм должны работать с Принтером не как с конкретной реализацией, а как с интерфейсом. 
Например:
```
public class Main {
  public static void main(String[] args) {
    MessagePrinter messagePrinter = new ColorConsoleMessagePrinter(Color.RED);
    Game game = new Game(messagePrinter);
    game.start();
  }
}
```

**22. class MoveCounterView**

Если ты ввел в проект специальный ПринтерСообщений, то вообще вся печать должна идти через этот принтер.  
А не печатать напрямую в консоль, как сейчас:
```
public final class MoveCounterView {
  public static void display(MoveCounter moveCounter) {
    System.out.println("Move counter: " + moveCounter.getCounter()); <-- ПЕЧАТЬ НАПРЯМУЮ В КОНСОЛЬ
  }
}
```

**23. class CommandListener implements Runnable**

+ 👍 Реализовано ок.

- Название вводит в заблуждение. 

"Listener" это название стандартного паттерна, а этот класс не имеет отношения к данному паттерну. 
Название класса нужно поменять.

**24. class GameMapConsoleRenderer**

- Эта строка вообще не читается, вводи поясняющие переменные
```
System.out.print(SpriteColorizer.colorizeSprite(EntitySpriteFactory.getEntitySprite(gameMap.getEntity(coordinates))));
```
*Фаулер, "Рефакторинг", гл.6, "Введение поясняющей переменной"*  

- Распечатка должна идти через ПринтерСообщений, а не напрямую в консоль.

**25. class Simulation implements Runnable**

- Нарушение DI. Класс не должен инициализировать сам себя. 
В данном случае класс инициализирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```
  public Simulation() {
    this.gameMap = new GameMap();
    //...
  }
```
Таким образом нельзя создать несколько игровых конфигураций с разными картами.  
*Мартин, "ЧК", гл.11.2*

Simulation должен иметь возможность принимать в себя карту произвольных размеров.

- Внезапно, нарушение инкапсуляции. 
Всегда явно указывай область видимости и final, если переменная final
```
GameMap gameMap;
```

- initActions должны один раз вызываться в конструкторе класса, а не вызываться по условию при совершении хода.

+ 👍 Правильное использование экшенов- они хранятся в списках и используются одинаково через полиморфизм. 
Вот в этих списках мы и видим ИСТИННЫЕ экшены, которые есть в программе
```
this.initActions = List.of(new InitSpawnEntities());
this.turnActions = List.of(new PopulationRestorer(), new TurnMoveAllCreatures());
```

**26. class Main**, содержит точку входа main

👍 Только создает и запускает Simulation, это хорошо.

## ВЫВОД

Есть простейшие нарушения конвенции, почитать *Oracle Java code conventions*.  
Видна тенденция к избыточному дроблению классов.  
Иногда намерения непонятны из кода. Например, зачем введен специальный Принтер, если через него идет не вся распечатка.  
Разобраться с экшенами.

В целом, более-менее норм.

n.104(236)  
#ревью #симуляция 