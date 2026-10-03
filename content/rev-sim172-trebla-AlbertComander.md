https://github.com/AlbertComander/Simulation  
[Trebla]

Есть над чем поработать.  
Визуальный интерфейс сделан в Swing, интеграция с этой графической библиотекой выполнена хорошо.

## ХОРОШО

+ 👍 Есть пауза/пуск во время работы
+ 👍 Красивый визуальный Windows - интерфейс Swing
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim172/img0.png) 

+ 👍 Две скорости работы
+ 👍 Возобновление ресурсов во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Правильнее такие методы называть по аналогии с тем, как это делается в стандартной библиотеке: `toList()`, `toString()` etc
```java
public class WorldMap {
  private final Map<Coordinate, Entity> entities = new HashMap<>();
  //...

  public Map<Coordinate, Entity> getEntitiesSnapshot() {
    return new HashMap<>(entities);
  }
}

//ЛУЧШЕ:
public class WorldMap {
  private final Map<Coordinate, Entity> entities = new HashMap<>();
  //...

  public Map<Coordinate, Entity> toMap() {
    return new HashMap<>(entities);
  }
}
```
Потому что тут ты фактически один класс - хранилище координат и существ конвертируешь в другой класс - хранилище координат и существ.

- Предлог "To" используй только в методах конвертации из одного в другое. 

Предлог "To" используется в методах конвертации: `toMap()`, `toString()`, `toInt()`, `meterToFeet()`.   
В остальных случаях воздерживайся от этого. Иначе сейчас кажется, что метод конвертирует существо в индексы
```java
void addEntityToIndexes(Coordinate coordinate, Entity entity) 

//ПРАВИЛЬНО:
void addIndexEntity(Coordinate coordinate, Entity entity) 
```

- Избыточный контекст. 

Мы и так понимаем, что метод поиска в классе поиска пути ищет именно путь, а не что-то иное
```java
public class PathFinder {
  //...
  public PathResult findPath(...) {...}
}

//ПРАВИЛЬНО:
public class PathFinder {
  //...
  public PathResult find(...) {...}
}
```

- Неинформативное название. Какой именно процесс происходит с соседями?
```java
private void processNeighbours(...)
```
Так любой метод можно назвать просто процессом, например `processEntity()` вместо `moveEntity()`.

- Переменные должны называться стилем camelCase
```java
int healthpoints;

//ПРАВИЛЬНО:
int healthPoints;
```

- Слишком похожие названия двух методов одного класса.  

Непонятно, в каких случаях клиент должен показать карту(show), а в каких- перерисовать её(repaint).
Звучит, как синоним
```java
public class View {
  //...
  
  public void showMap() {...}
  public void repaintMap() {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение DRY**

Магические буквы, числа, слова. Вводи константы 
```java
JButton simulationSpeedButton = new JButton("1x");
setText(controls.updateSimulationSpeed() ? "2x" : "1x");

//ПРАВИЛЬНО:
private static final String FIRST_SPEED = "1x"
private static final String SECOND_SPEED = "2x"

JButton simulationSpeedButton = new JButton(FIRST_SPEED);
setText(controls.updateSimulationSpeed() ? SECOND_SPEED : FIRST_SPEED);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru, "Замена магического числа символьной константой"*

**3. Exceptions**

- Текст в Exception всегда должен быть на английском языке.
```java
throw new IllegalStateException("Не удалось загрузить файл: " + path, e); <-- Неправильно: сообщение в исключении не на английском языке
```
Исключение это не просто телеграмма, которая летит сквозь слои.

У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.  
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты.  
А значит, сообщение должно быть на английском.

Интерпретация исключения и перевод его на локальный язык должны происходить там, где это соответствует архитектуре программы.  
Или не происходить вовсе, если исключение не планируется перехватывать.

**4. class Coordinate**

+ 👍 Нет ничего лишнего, это хорошо. Класс только хранит значения x,y. 

- Класс может быть преобразован в `record` без потери функционала.

Для простых классов-контейнеров данных лучше всего использовать особый тип класса: `record`.  
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

- Зависимость модели от представления.

- Неправильный `toString()`, модель зависит от представления. 

toString() должен быть стандартным(как делает IDE по Alt+Ins) и использоваться только для отладки.  
toString() должен содержать значение всех значимых полей класса.  
toString() не должен использоваться для представления, за исключением предельно простых классов, вроде PhoneNumber
```java
public String toString() {
  return "Координата x: " + x + " Координата y: " + y + " ";
}

//ПРАВИЛЬНО:
public String toString() {
  return "Coordinate{" +
    "x=" + x +
    ", y=" + y +
    '}';
}
```
*Блох, "Java. Эффективное программирование", изд.3, гл.3.3*  

Хотя этот класс предельно простой и его `toString()` можно использовать для отображения, но с ним другая проблема. 

Конкретно здесь сообщение на русском языке в модели координаты(а координата это модель) приводят к тому, что этот класс можно использовать только в русскоязычной программе.  
То есть зависимость этой модели от представления состоит в том, что этот класс можно использовать только для визуального интерфейса с определенным языком.

**5. class WorldMap**

- Нарушение SRP, OCP и ТЗ

Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретными классами-наследниками Entity.  
Здесь карта хранит каждый вид существ в отдельной мапе
```java
private final Map<Coordinate, Entity> entities = new HashMap<>();  <-- ОСТАВИТЬ ТОЛЬКО ЭТО
private final Set<Coordinate> grassPositions = new HashSet<>();  <-- ЛИШНЕЕ
private final Set<Coordinate> herbivorePositions = new HashSet<>();  <-- ЛИШНЕЕ
private final Set<Coordinate> predatorsPositions = new HashSet<>();  <-- ЛИШНЕЕ
```
Нарушение SRP: карта знает слишком много. 

Нарушение OCP: класс открыт для изменений.  
При добавлении нового существа в проект, придётся добавлять в класс новый код для этого существа:
```java
private final Set<Coordinate> birdPositions = new HashSet<>();
```

Нарушение ТЗ:
```
Чеклист для самопроверки #
Проблемы и ошибки в коде:
Параллельные коллекции для разных типов существ (для каждого типа своя коллекция)
```

Для того, чтобы карта работала так, как надо, достаточно использовать только одну мапу:
```java
private final Map<Coordinate, Entity> entities = new HashMap<>();
```

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Посчитать количество существ определенного типа    
🔸 Вернуть все пустые координаты   
🔸 Определить, можно ли сделать ход существом 
🔸 Сделать ход существом  
etc 

Наверное, для проекта в целом полезно иметь метод, который считает количество травы.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно считать траву.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public int getPredatorsCount() {
  return predatorsPositions.size();
}

public int getHerbivoreCount() {
  return herbivorePositions.size();
}

public int getGrassCount() {
  return grassPositions.size();
}
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getBirdCount()`.

У тебя уже есть метод, который возвращает копию карты
```java
public Map<Coordinate, Entity> getEntitiesSnapshot() {
  return new HashMap<>(entities);
}
```

Если клиенту надо, то пусть он через этот метод получит всех существ, и сам пересчитает, сколько среди них травы, грибов, зайчиков.

- Последствия использования параллельных коллекций для хранения существ: громоздкий запутанный код
```java
private void addEntityToIndexes(Coordinate coordinate, Entity entity) {
  if (entity.getClass() == Grass.class) {
    grassPositions.add(coordinate);
  } else if (entity.getClass() == Herbivore.class) {
    herbivorePositions.add(coordinate);
  } else if (entity.getClass() == Predator.class) {
    predatorsPositions.add(coordinate);
  }
}
```

- Метод совершения хода в карте- нарушение SRP.

Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
public boolean moveEntity(Coordinate from, Coordinate to) {
  if (canMove(from, to)) {
    Entity entity = entities.get(from);
    placeEntity(to, entity);
    deleteEntityAt(from);
    return true;
  }
  return false;
}

private boolean canMove(Coordinate from, Coordinate to) {
  return (!isOccupied(to)
    && isOccupied(from)
    && isWithinBounds(to)
    && isWithinBounds(from)
  );
}
```
Должна ли карта учитывать правила логистики зайцев? Если да, то как?

Например, имеет ли право заяц телепортироваться на координату, которая уже занята?  
Может ли телепортироваться камень?  
Может ли телепортироваться пустое место на голову волка?

Метод частично отвечает на эти вопросы- вместе со вспомогательным методом, за это отвечает 16 строк кода.  
То есть карта не только хранит существ, но и хранит правила того, как они должны ходить.  
Тем самым карта забирает себе часть чужой ответственности.

Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность 
```java
public Entity getEntityAt(Coordinate coordinate) {
  return entities.get(coordinate);
}

public boolean isOccupied(Coordinate coordinate) {
  return entities.containsKey(coordinate);
}

//ПРАВИЛЬНО:
public Entity getEntityAt(Coordinate coordinate) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение  
  return entities.get(coordinate);
}

public boolean isOccupied(Coordinate coordinate) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение  
  return entities.containsKey(coordinate);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Разделяй команды и запросы. 

Этот метод совмещает команду и запрос.  
Команда: вставить существо.  
Запрос: вернуть статус выполнения операции- успешно или нет
```java
boolean placeEntity(Coordinate coordinate, Entity entity)
```
Метод должен или выполнять команду, или отвечать на запрос.
 
*Мартин, "Чистый код", гл.3, "Разделение команд и запросов"* 
```java
"Либо функция изменяет состояние объекта, либо возвращает информацию об этом объекте. 
Совмещение двух операций часто создает путаницу." - Мартин.
```

**6. class PathFinder**

- Как пользоваться поиском- неясно.

Поиск возвращает что-то предельно непонятное
```java
public class PathFinder {
  //...
  public PathResult findPath(...) {...}
}

public record PathResult(
    List<Coordinate> pathToTarget,
    Optional<Coordinate> targetCoordinate
) {
}
```

От поиска пути я хочу просто получить путь, то есть последовательность точек от начала пути до конца пути.  
Правильно так:
```java
public class PathFinder {
  //...
  public List<Coordinates> find(...) {...}
}
```

- Не перегружай блоки `for` сложными условиями
```java
for (Coordinate neighbourCoordinate : findNeighbourCoordinates(currentNode.getCoordinate())) {...}

//ПРАВИЛЬНО:
List neighbourCoordinates = findNeighbourCoordinates(currentNode.getCoordinate());
for (Coordinate coordinate : neighbourCoordinates) {...}
```

**7. class Entity**

- Класс должен быть абстрактным. Не должно быть возможности создать "просто Entity".

- Содержит координату
```java
public class Entity {
  Coordinate coordinate;
}
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

**8. abstract class Creature extends Entity**

- Креатура должна сама искать свои текущие координаты
```java
public void makeMove(WorldMap worldMap, PathFinder pathFinder, Coordinate currentLocation) {
  PathResult pathResult = pathFinder.findPath(currentLocation, targets);
  //...
}

//ПРАВИЛЬНО:
public void makeMove(WorldMap worldMap, PathFinder pathFinder, Coordinate currentLocation) {
  Coordinate current = worldMap.getCoordinate(this);  
  PathResult pathResult = pathFinder.findPath(current, targets);
  //...
}
```
Креатура все равно в метод хода принимает `WorldMap`, поэтому может спросить у карты свое текущее местоположение.  

Если креатура будет принимать свое текущее местоположение извне, а не получать самостоятельно из карты, то это потенциально может привести к багам.  
Потому что креатура не может гарантировать, что пришедшая извне координата *действительно* соответствует ее текущему местоположению.  
Слепая вера клиенту тут вредна.

**9. record SimulationConfig**

- Неправильное использование record'a.

Классы типа `record` должны использоваться только как контейнеры данных.  
Этот рекорд имеет развитое поведение, где по хитрым формулам что-то считает:
```java
public int calculateRockCount(int totalCells) {
  return calculateCount(totalCells, rockPercent);
}

private int calculateCount(int totalCells, int percent) {
  return (int) Math.round(totalCells * percent / 100.0);
}
```

- Гибрид конфигурационного класса и класса с прикладной логикой
```java
public record SimulationConfig(
    //КОНФИГИ ⬇️
    int predatorPercent,  
    int herbivorePercent,
    //...
) {
  
  //Методы прикладной логики ⬇️
  public int calculateRockCount(int totalCells) {
    return calculateCount(totalCells, rockPercent);
  }

  private int calculateCount(int totalCells, int percent) {
    return (int) Math.round(totalCells * percent / 100.0);
  }
  //...
}
```
Конфиг должен хранить только конфигурационные данные.  
Если с этими данными нужно делать какие-то хитрые манипуляции, то это должен делать другой класс.

**10. interface Action**

👍 Норм. Но "abstract" в интерфейсах не пишется- там все методы по умолчанию абстракт 
```java
public interface Action {
  abstract void execute();
}
```

**11. class InitializeMapAction implements Action**

+ 👍 Заселение существ через `Supplier` это хорошо
```java
placeEntities(Predator::new, config.calculatePredatorsCount(totalCells), iterator);
placeEntities(Herbivore::new, config.calculateHerbivoreCount(totalCells), iterator);
//...

private void placeEntities(Supplier<? extends Entity> factory, int count, Iterator<Coordinate> coordinateIterator) {...}
```

**12. class MoveAllCreaturesAction implements Action**
```java
public void execute() {
  Map<Coordinate, Entity> worldMapSnapshot = worldMap.getEntitiesSnapshot();
  for (int x = 0; x < worldMap.getWidth(); x++) {
    for (int y = 0; y < worldMap.getHeight(); y++) {
      Coordinate coordinate = new Coordinate(x, y);
      Entity entity = worldMapSnapshot.get(coordinate);
      if (entity instanceof Creature creature && (entity == worldMap.getEntityAt(coordinate))) {
        creature.makeMove(worldMap, pathFinder, coordinate);
      }
    }
  }
}

//ПРАВИЛЬНО:
public void execute() {
  List<Creature> creatures = getCreatures(worldMap);
  for(Creature creature : creatures) {
    creature.makeMove(worldMap, pathFinder);  
  }
}

private static List<Creature> getCreatures(Карта карта) {
  //найти и вернуть все креатуры из карты
}
```

**13. class MapRenderer**

Представление(view) для Swing.

+ 👍 Соответствует требованиям для слоя представления(view), это хорошо.  
Если делал сам без ИИ, то вообще молодец.

- Главные публичные методы должны стоять выше, чем вспомогательные приватные.

- Нейминг. 

Этот класс не только рисует карту. Это в целом визуальный интерфейс
```java
public class MapRenderer {
  //...

  public void showMap() {...}
  public void updateTurnCount(int turnCount) {...}
  public void repaintMap() {...}
}

//ПРАВИЛЬНО:

public class View {
  //...
  
  public void showMap() {...}
  public void updateTurnCount(int turnCount) {...}
  public void repaintMap() {...}
}
```

**14. class Simulation implements SimulationControls**

- Вся инициализация должна выполняться в конструкторе
```java
public class Main {
  //...  
  Simulation simulation = new Simulation(worldMap);
  simulation.initializeSimulation();
}

public class Simulation implements SimulationControls {
  //...

  public Simulation(WorldMap worldMap) {
    //...
  }

  public void initializeSimulation() {
    //что-то инициализирует
  }
}

//ПРАВИЛЬНО:
public class Main {
  //...  
  Simulation simulation = new Simulation(worldMap);
  simulation.start();
}

public class Simulation implements SimulationControls {
  //...

  public Simulation(WorldMap worldMap) {
    //...
    initialize();
  }

  private void initialize() {
    //что-то инициализирует
  }
}
```

+ 👍 В целом норм.

**15. class Main**

Содержит точку входа main.

+ 👍 Только создает и запускает Симуляцию, это хорошо.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Инициализация это одно, запуск работы- другое
```java
static void main(String[] args) {
  WorldMap worldMap = new WorldMap(MAP_HEIGHT, MAP_WIDTH);
  Simulation simulation = new Simulation(worldMap);
  simulation.initializeSimulation();
}

//ПРАВИЛЬНО:
static void main(String[] args) {
  WorldMap worldMap = new WorldMap(MAP_HEIGHT, MAP_WIDTH);
  Simulation simulation = new Simulation(worldMap);  //Вся инициализация должна происходить в конструкторе
  simulation.start();  //Здесь должен происходить запуск симуляции
}
```

## ВЫВОД

Интеграция с графической библиотекой Swing сделана хорошо.

n.172(372)  
#ревью #симуляция #swing  