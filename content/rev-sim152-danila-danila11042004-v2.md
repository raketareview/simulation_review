https://github.com/danila11042004/newSimulation  
[Danila]

Есть над чем поработать, но замечаний меньше обычного.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неправильно
```java
public static void main(String[] args) {
  MapEntity mapEntity = new MapEntity();
  Coordinates coordinates = new Coordinates(5, 0);
  mapEntity.put(coordinates, new Tree());
  Render render = new Render();
  render.rendering(mapEntity);
}

//РЕЗУЛЬТАТ:
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟩🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
```

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты в существах хранятся начиная с Creature
+ 👍 Реализована пауза/пуск во время работы
+ 👍 Алгоритм поиска A*

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Массивы и коллекции называй во множественном числе
```java
int[][] move = new int[][]{{-1, 0}, {1, 0}, {0, 1}, {0, -1}};

//ПРАВИЛЬНО:
int[][] moves = new int[][]{{-1, 0}, {1, 0}, {0, 1}, {0, -1}};
```

- Значения енамов это константы. Они должны называться стилем UPPER_SNAKE
```java
public enum EntityType {
  Tree, Rock, Grass, ...
}

//ПРАВИЛЬНО:
public enum EntityType {
  TREE, ROCK, GRASS, ...
}
```

- Не называй методы через префикс "try".

Методы не должны "пытаться" что-то делать. Они должны делать.  
Если метод по какой-то причине не может сделать то, что обещает, он должен кинуть исключение
```java
protected abstract boolean tryToEat(MapEntity mapEntity, EntityType targetType);

//ПРАВИЛЬНО:
protected abstract boolean eat(MapEntity mapEntity, EntityType targetType);
```

- Очень странное название. Старайся использовать стандартные названия 
```java
public abstract class Action {
  public abstract void operate(MapEntity mapEntity);
}

//ПРАВИЛЬНО:
public abstract void perform(MapEntity mapEntity);  //или execute()
```

- Названия классов должны быть существительными, а методы- глаголами.

"Render" это действие, глагол. А "Renderer" это тот, кто выполняет это действие
```java
public class Render {
 
  public void rendering(MapEntity mapEntity) {...}
}

//ПРАВИЛЬНО:
public class Renderer {
 
  public void render(MapEntity mapEntity) {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Порядок расположения методов в классе.

Во всем проекте системно нарушаются правила расположения методов.  
Публичные методы должны стоять выше приватных.  
Например:
```java
public class ActionBalanceNumberEntity extends ActionEntityArrangement {

    private void addEntity(Function<Coordinates, Entity> function, MapEntity mapEntity, int counter) {...}

    public void operate(MapEntity mapEntity) {...}
}

//ПРАВИЛЬНО:
public class ActionBalanceNumberEntity extends ActionEntityArrangement {

  public void operate(MapEntity mapEntity) {...}

  private void addEntity(Function<Coordinates, Entity> function, MapEntity mapEntity, int counter) {...}
}
```

Публичные и приватные методы могут располагаться вперемешку только тогда, когда публичные метод и его вспомогательные методы идут одной связкой.  
Но в этих связках все равно публичные методы должны стоять выше вспомогательных им приватных методов.   
Условный пример:
```java
public void first() {
  doSomething1() {...}
  doSomething2() {...}
}

private void doSomething1() {...}
private void doSomething2() {...}

public void second() {
  doSomething3() {...}
}

public void doSomething3() {...}
```

**3. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется**
 
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без `else` будет выглядеть читабельней
```java
if (entity instanceof Grass) {
  return iconGrass;
} else if (entity instanceof Tree) {
  return iconTree;
}

//ПРАВИЛЬНО:
if (entity instanceof Grass) {
  return iconGrass;
} 
if (entity instanceof Tree) {
  return iconTree;
}
```

**4. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```java
System.out.println("Нажмите 1 - для продолжения\n" +
  "Нажмите 2 чтобы отрендерить один ход\n" +
  "Нажмите 3 чтобы выйти из игры");
case "1":  //...
case "2":  //...
case "3":  //...

//ПРАВИЛЬНО:
private final static String START = "1";
private final static String STEP = "2";
private final static String QUIT = "3";


System.out.printf("Введите %s - для продолжения  %n", START);
System.out.printf("Введите %s чтобы отрендерить один ход  %n", STEP);
System.out.printf("Введите %s чтобы выйти из игры  %n", QUIT);

case START:  //...
case STEP:  //...
case QUIT:  //...
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**5. record Coordinates(int x,int y)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```java
public record Coordinates(int x,int y) {
}
```

**6. class MapEntity**

- Карта должна иметь возможность быть созданной с произвольными размерами.

Карта содержит в себе фиксированные размеры.  
Поэтому становится неуниверсальной- нельзя создать игровые конфигурации с картами разных размеров.  
Нет оснований ограничивать форму карты квадратом
```java
public class MapEntity {
  public static final int WORLD_SIZE = 15;
  //...

  public MapEntity() {
    locationWithEntities = new HashMap<>();
  }
  //...
}

//ПРАВИЛЬНО:
public class MapEntity {
  private final int width;
  private final int height;
  //...

  public MapEntity(int width, int height) {
    this.width = width;
    this.height = height;
    locationWithEntities = new HashMap<>();
  }
  //...
}
```
Фиксированные размеры могут быть у шахматной доски(8x8) или игры крестики-нолики(3x3).  
То есть в тех играх, где размер игрового поля жестко фиксирован правилами.  
Здесь размеры должны приниматься в конструктор, чтобы можно было создавать карту произвольных размеров.

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
Хранить количество единиц травы и других существ + геттеры и сеттеры для них
```java
private int grassNumber;
private int herbivoreNumber;
private int predatorNumber;

public int getGrassNumber() 
public void setGrassNumber(int grassNumber)
```

Наверное, для проекта нужно зачем-то держать счетчики существ(а может и нет).  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно подсчитывать количество разных существ.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Никогда не возвращай null
```java
private final Map<Coordinates, Entity> locationWithEntities;
 
public Entity getEntity(Coordinates coordinates) {
  return locationWithEntities.get(coordinates);  <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```java
private final Map<Coordinates, Entity> locationWithEntities;

public Map<Coordinates, Entity> getLocationWithEntities() {
  return new HashMap<>(locationWithEntities);
}
```

Чеклист ТЗ:
```java
Проблемы и ошибки в коде:
...
Недостаточная инкапсуляция - “протекающее” наружу внутреннее устройство класса Map (например, прямой доступ к коллекции ячеек) 
вместо набора методов с говорящими названиями (AddEntity, RemoveEntity, и так далее)
```

В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```java
Карта карта = new Карта();
<заселить карту существами>
карта.getEntities().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

Тут есть два варианта, как сделать правильно.  
Либо вернуть не оригинал мапы, а ее копию: 
```java
public Map<Coordinates, Entity> getLocationWithEntities() {
  return new HashMap(locationWithEntities);
}
```

Либо возвращать существ персонально:
```java
public Entity get(Coordinates coordinates)
public Coordinates getCoordinates(Entity entity)
```
Лично мне больше нравится второй вариант, когда существ возвращают персонально. 

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.

Если координата некорректна(находится вне пределов карты), нужно бросать исключение.
Потому что сейчас можно поместить существо на координату, которая будет выходить за пределы карты:
```java
private final Map<Coordinates, Entity> locationWithEntities;

public void put(Coordinates coordinates, Entity entity) {  <-- Можно передать координату (+100500,-100500)
  locationWithEntities.put(coordinates, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратитья к ним по несуществующему индексу, бросается исключение.

- Более явно обозначай свои намерения.

Клиент не должен знать ни про какие ключи в Карте.  
Потому что ключ это подробность внутреннего устройства Карты.  
Клиента интересует другой вопрос- свободна ячейка карты, или нет
```java
public boolean containsKey(Coordinates coordinates) {
  return locationWithEntities.containsKey(coordinates);
}

//ПРАВИЛЬНО:
public boolean isEmpty(Coordinates coordinates) {
  Coordinates coordinates = locationWithEntities.containsKey(coordinates);
  //если координата вне пределов карты- бросить исключение
  return coordinates == null;
}
```

**7. class PathFinder**

- Нейминг.

Название методов должно быть глаголом
```java
public class PathFinder {
  public List<Coordinates> algorithmFindAStar(...) {...}
}

//ПРАВИЛЬНО:
public class AstarPathFinder {
  public List<Coordinates> find(...) {...}
}
```

- Нарушение конвенции кода.

Публичные методы должны стоять выше приватных.  
В этом классе должен быть только один публичный метод и он должен стоять самым первым.

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
public List<Coordinates> algorithmFindAStar(Coordinates hunter, Coordinates target, MapEntity mapEntity) {...}
```

- Смещения нужно вынести из метода и сделать константами.  

Потому что сейчас при вызове метода каждый раз пересоздается объект(а массив это объект) с направлениями смещения.  
Пары x/y это координаты. Направление можно представить как координату, смещенную относительно начала движения
```java
private void expandNode(MapEntity mapEntity, SearchCell currentCell,
                           Queue<SearchCell> openSet, Set<SearchCell> closedSet) {
  int[][] move = new int[][]{{-1, 0}, {1, 0}, {0, 1}, {0, -1}};
  for (int[] m : move) {...}
  //...
}

//ПРАВИЛЬНО:
private final static Coordinates[] SHIFTS = {new Coordinates(-1, 0), new Coordinates(1, 0), ...}; 

private void expandNode(MapEntity mapEntity, SearchCell currentCell,
                           Queue<SearchCell> openSet, Set<SearchCell> closedSet) {
  for (Coordinates shift : SHIFTS) {...}
  //...
}
```

- Можно проще
```java
int[][] move = new int[][]{{-1, 0}, {1, 0}, {0, 1}, {0, -1}};

//ПРАВИЛЬНО:
int[][] move = {{-1, 0}, {1, 0}, {0, 1}, {0, -1}};
```

**8. abstract class Entity и его простые наследники Tree/Rock/Grass**

👍 Идеально
```java
public abstract class Entity {
}

public class Rock extends NonMovable {
}
```

**9. enum EntityType**

Енам существует для дополнительной типизации классов
```java
public enum EntityType {
  Tree, Rock, Grass, Herbivore, Predator
}
```
Не используй дополнительную типизацию классов, потому что это дублирует стандартные способы.  
Используй стандартные способы определять принадлежность объектов к классам:
```java
EntityType type = ...

switch (type) {
  case HERBIVORE: //...
  case PREDATOR:  //...
}

//ПРАВИЛЬНО:
String name = entity.getClass().getSimpleName();

switch (name) {
  case "Herbivore": //...
  case "Predator":  //...
}
```

**10. abstract class Creature extends Entity**

- Не используй енамы с внешней типизацией.  
Используй стандартные способы определения принадлежности объекта к типам
```java
public abstract class Creature extends Entity {
  private final EntityType targetType;
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  private final Class<? extends Entity> target;
  //...
}
```

**11. class CreatureMovement**

Контейнер с функциями в стиле процедурного программирования.  
Класс нужно расформировать, соответствующий код из него перенести в Creature, Herbivore и Predator.

Общий код совершения хода перенести в Creature.
В Herbivore и Predator нужно перенести специфический для этих классов код хода.

**12. abstract class Action**

👍 Кроме названия метода, всё ок
```java
public abstract class Action {
  public abstract void operate(MapEntity mapEntity);
}
```

**13. public class ActionMovementCreatures extends Action**

Содержит слишком много кода для этого экшена.  
Экшен совершения хода существами должен просто обойти всю карту, найти каждую креатуру и дать ей пинка, чтобы она побежала.

Как при этом будет бежать креатура и что при этом делать, не должно волновать этот экшен. 
Выглядеть это должно примерно так
```java
class MoveAction реализует Action {

  @Override
  public void execute(Карта карта) {
    List<Creature> creatures = getCreatures(карта);
    for(Creature creature: creatures) {
      creature.makeMove(карта);  //даёт пинка
    }
  }

  private static List<Creature> getCreatures(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

**14. class ActionEntityArrangement extends Action**

- Нарушение инкапсуляции.

Всегда явно указывай область видимости.  
Не делай поля публичными, если они используются только внутри самого класса
```java
public class ActionEntityArrangement extends Action {
  public static final int NUMBER_ENTITY = 20;
  Random random = new Random();
  //...
}

//ПРАВИЛЬНО:
public class ActionEntityArrangement extends Action {
  private static final int NUMBER_ENTITY = 20;
  private final Random random = new Random();
  //...
}
```

- Сложная логика заселения существ
```java
public static final int NUMBER_ENTITY = 20;

public void operate(MapEntity mapEntity) {
  for (int i = 0; i < NUMBER_ENTITY; i++) {
    Coordinates coordinates = getRandomEmptyCell(mapEntity);
    switch (random.nextInt(5)) {
      case 0:
         Grass newGrass = new Grass();
         mapEntity.put(coordinates, newGrass);
         mapEntity.setGrassNumber(mapEntity.getGrassNumber() + 1);
         break;
      case 1:
        Rock newRock = new Rock();
        mapEntity.put(coordinates, newRock);
        break;
      //...
    }
  }
}
```

С помощью стандартного интерфейса Function это можно сделать проще: 
```java
public class SpawnAction extends Action {
  private static final int ENTITY_COUNT = 20;

  private final static List<Function<Coordinates, Entity>> FUNCTIONS = List.of(
      (coordinates) -> new Grass(),
      (coordinates) -> new Herbivore(coordinates, 2, 10, EntityType.Grass),
      (coordinates) -> new Predator(coordinates, 3, 10, EntityType.Herbivore, 5),
      //...
  );
  //...

  public void execute(Карта карта) {
    for (int i = 0; i < ENTITY_COUNT; i++) {
      int index = random.nextInt(FUNCTIONS.size());
      Coordinates coordinates = getRandomEmptyCoordinates(карта);

      Function<Coordinates, Entity> function = FUNCTIONS.get(index);
      Entity entity = function.apply(coordinates);

      карта.put(coordinates, entity);
    }

  private Coordinates getRandomEmptyCoordinates(Карта карта) {
    //находит и возвращает случайную пустую координату
  }
}
```

Да, в твоем оригинальном алгоритме еще ведется подсчет заселяемых существ
```java
mapEntity.setGrassNumber(mapEntity.getGrassNumber() + 1);
```
Но это что-то совсем ненужное.  
Как я писал выше, класс Карты не должен хранить счётчики поголовья существ.

**15. class Render**

+ 👍 Спрайты существ хранятся здесь, а не берутся из самих существ, это хорошо.

- Эти поля должны быть константами
```java
public class Render {
  private final String iconGrass;
  private final String iconTree;
  //...

  public Render() {
    iconGrass = "🍏";
    iconTree = "🟩";
    //...
  }
}

//ПРАВИЛЬНО:
public class Render {
  private final static String ICON_GRASS = "🍏";
  private final static String ICON_TREE = "🟩";
  //...
}
```

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < MapEntity.WORLD_SIZE; i++) {
  for (int j = 0; j < MapEntity.WORLD_SIZE; j++) {
    Coordinates coordinates = new Coordinates(i, j);
    //...
  }
}

//ЛУЧШЕ:
for (int y = 0; y < MapEntity.WORLD_SIZE; y++) {
  for (int x = 0; x < MapEntity.WORLD_SIZE; x++) {
    Coordinates coordinates = new Coordinates(y, x);
    //...
  }
}
```

И тогда сразу станет ясно, почему карта распечатывается неправильно.

**16. class Simulation**

- Нарушение ТЗ. Нет указанных в ТЗ методов
```java
Simulation #
Главный класс приложения, включает в себя:
...
Методы:

nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

- Неправильное использование потоков в проекте.

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

Это должен быть такой класс, который можно будет не только запустить из потоков.  
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
А другой Main такой:
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

- Нарушение SRP. Метод не должен завершать работу программы через exit()
```java
public class Simulation {
  //...

  private void callPauseMenu() {
    //...
    System.exit(0);
  }
}
```
Каждый метод и класс имеют право завершать только свою работу. Например, через return.  
Потому что методы и классы не должны знать логику работу более высоких слоев программы, у которых могут быть свои планы на тему того, 
когда и почему нужно завершать работу программы.

Кроме того, при выходе через `exit()` могут не закрыться некоторые ресурсы программы.

Возможно, если прекращать работу здесь, то это сильно упрощает код. Тогда можно оставить, как есть.  
Но нужно понимать, что с архитектурной точки зрения это хуже и это что-то вроде костыля.

**17. class Main**

👍 Только создает и запускает проект, это хорошо.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

В коде не замечено ничего особо ужасного.  
В целом замечаний меньше, чем обычно.

n.152(323)  
#ревью #симуляция 