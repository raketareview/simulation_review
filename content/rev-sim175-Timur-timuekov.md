https://github.com/timuekov/simulation-java  
[Timur]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim175/img0.png) 

## ХОРОШО

+ 👍 Есть пауза/пуск во время работы
+ 👍 Спрайты существ не хранятся в самих существах

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Избыточный контекст. 

Мы и так понимаем, что метод поиска в классе поиска пути ищет именно путь, а не что-то иное
```java
public class PathFinder {
  public static List<Coordinates> findPath(...) {...}
}

//ПРАВИЛЬНО:
public class PathFinder {
  public static List<Coordinates> find(...) {...}
}
```

+ 👍 В целом нейминг норм.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. Класс только хранит значения x,y. 

- Класс может быть преобразован в `record` без потери функционала.

Для простых классов-контейнеров данных лучше всего использовать особый тип класса: `record`.  
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**3. class SimulationMap**

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Creature> getAllCreatures()

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда креатуры

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getAllBirds()`.

- Никогда не возвращай null
```java
private final Map<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return entities.get(coordinates);  <-- вернет null если в entities нет coordinates
}
```
Возврат null повышает риск возникновения NullPointerException в программе.

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Проверяй, находится ли координата в пределах карты. 

При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность.  
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что она свободна.  
А правильный ответ - в карте вообще нет ячейки с такой координатой
```java
  public boolean isCellEmpty(Coordinates coordinates) {
    return (entities.get(coordinates) == null);
 }

//ПРАВИЛЬНО:
public boolean isCellEmpty(Coordinates coordinates) {
  validate(coordinates);  <-- Бросает исключение, если координата не находится в пределах карты
  return entities.get(coordinates) == null;
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Если по какой-то причине невозможно вставить существо на координату, то нужно бросить исключение
```java
public void setEntity(Coordinates coordinates, Entity entity) {
  if (isCellEmpty(coordinates)) {
    entities.put(coordinates, entity);
  }
}

//ПРАВИЛЬНО:
public void setEntity(Coordinates coordinates, Entity entity) {
  validate(coordinates);

  if (!isCellEmpty(coordinates)) {
    //нельзя поставить существо на эту координату - кинуть исключение
  }
  entities.put(coordinates, entity);
}
```
Потому что если команда вставить существо на указанную координату будет молча проигнорирована, это баг. 
Тогда на карте во время работы программы будут таинственно исчезать существа: в одном месте ты их удалишь, а в другом они не вставятся.

+ 👍 В целом нарушения SOLID в классе минимальны.

**4. class PathFinder**

- Соблюдай требования утилитных классов. 

Это утилитный класс, он содержит только статические методы.  
Утилитные классы, как и классы констант, должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.

*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- В Java не используй метки
```java
BFS:
{
  //...  
  break BFS;
}
```

Строй алгоритмы так, чтобы не приходилось перелетать код по меткам - это плохо читается.  
Нет такой ситуации, с которой можно было бы справиться только с помощью метки.

- Нарушение SRP.

Поиск пути между двумя координатами нужен только для алгоритма A*.  
Алгоритм BFS должен искать путь от точки старта до точки, где находится объект нужного класса:
```java
public static List<Coordinates> findPath(Coordinates start, Coordinates target, SimulationMap map) {
  //Ищет путь по алгоритму BFS
  //Но не сравнивает найденное по координате существо с заданным типом,
  //а сравнивает найденную координату с заданной
}

//ПРАВИЛЬНО ДЛЯ BFS:
public static List<Coordinates> findPath(SimulationMap simulationMap, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)    
}
```

Сейчас алгоритм поиска принимает в себя координату, до которой нужно проложить путь.  
Следовательно, клиент уже выполнил часть работы по поиску пути- предварительно нашел цель на карте и узнал её координату.  
Таким образом, единая ответственность по поиску пути *по алгоритму BFS* сейчас разделена на несколько классов.

- Пара x,y - это координата
```java
public static List<Coordinates> findPath(...) {
  int[][] directions = {
    {0, -1},
    {0, 1},
    {-1, 0},
    {1, 0}
  };

  for (int[] direction : directions) {
    int dx = direction[0];
    int dy = direction[1];
    Coordinates neighbor = new Coordinates(current.getX() + dx, current.getY() + dy);
    //...
  }
}

//ПРАВИЛЬНО:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(new Coordinates(0, -1), ...);

public static List<Coordinates> findPath(...) {
  for (Coordinates shift : SHIFT_COORDINATES) {
    Coordinates neighbor = new Coordinates(current.getX() + shift.getX(), current.getY() + shift.getY());
    //...
  }
}
```

- Нельзя возвращать null
```java
if (!cameFrom.containsKey(target) && !start.equals(target)) {
  return null; // пути не существует
}
```
Если пути не существует- просто возвращай пустой список координат.  

*Блох "Java. Эффективное программирование", изд.3, 8.6. "Возвращайте пустые массивы и коллекции, а не null"*

**5. abstract class Entity**

- Содержит координату
```java
private Coordinates coordinates;
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

**6. abstract class Creature extends Entity**

- Нарушение инкапсуляции и правила минимального открытого интерфейса.

Этот метод используется только потомками- он должен быть видим только потомкам
```java
public Coordinates findNearestTarget(List<Coordinates> targets) 

//ПРАВИЛЬНО:
protected Coordinates findNearestTarget(List<Coordinates> targets) 
```
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

- Нарушение SRP. Существо не должно лично искать координаты целей
```java 
public Coordinates findNearestTarget(List<Coordinates> targets) {
  //ищет координаты целей    
}
```

Существо должно использовать класс поиска:
```java
private final Class<? extends Entity> food;

public void makeMove(SimulationMap simulationMap) {
  List<Coordinates> path = PathFinder.find(simulationMap, coordinates, food);
  //...    
}
```

Если координату не хранить в самом существе, то:
```java
private final Class<? extends Entity> food;

public void makeMove(SimulationMap simulationMap) {
  Coordinates current = simulationMap.getCoordinates(this);
  List<Coordinates> path = PathFinder.find(simulationMap, current, food);
  //...    
}
```

- Никогда не возвращай null
```java
public Coordinates findNearestTarget(List<Coordinates> targets) {
  Coordinates nearest = null;

  for (Coordinates targetCoords : targets) {...}

  return nearest;  <-- Вернёт null если не найдёт ни одной цели
}
```

**7. class Herbivore/Predator extends Creature**

- Общий код в методе `makeMove(...)` у классов `Herbivore` и `Predator`.

ТЗ:
```java
Чеклист для самопроверки #

Проблемы и ошибки в коде:
Дублирование кода между классами Herbivore и Predator
```
Общий код выноси в предка, в данном случае- в класс `Creature`.

**8. Оценка кода иерархии классов entities**

👍 В целом неплохо.

**9. abstract class Action**

👍 Идеально
```java
public abstract class Action {
  public abstract void perform(SimulationMap map);
}
```

**10. class MoveAction extends Action**

👍 Хорошо, но в Карте не должно быть метода `getAllCreatures()`.

**11. abstract class SpawnAction extends Action**

- Нарушение SRP, чужая ответственность, зависимость модели от представления.

Модель не должна ничего печатать в консоль
```java
System.out.println("Свободных клеток больше нет!");
```
Экшены это модели (а по сути это сервисы с бизнес-логикой), они не должны ничего печатать, тем более в консоль.  
Иначе при использовании в других визуальных средах (Swing, Android) они или не скомпилируются, или просто ничего не смогут напечатать в визуальный интерфейс.

- Главные публичные методы должны стоять выше вспомогательных приватных.

**12. class MaintainGrassAction/MaintainHerbivoreAction extends Action**

- Нарушение конвенции кода. Конструктор должен стоять выше всех остальных методов.

- Нарушение DRY.

Код в этих классах во многом дублируется.  
Оба класса по сути делают одно и то же: если количество существ определенного типа меньше минимального, то заселить на карту нужное количество новых существ.

Общий код нужно вынести в общего предка.  
Для упрощения кода можно использовать стандартные функциональные интерфейсы Java.  
Работу не проверял, но общая идея такова:
```java
public class ReSpawnAction<T extends Entity> extends Action {
  private final Class<T> type;
  private final Function<Coordinates, T> entityCreator;
  private final int min;
  private final int max;

  public ReSpawnAction(Class<T> type, Function<Coordinates, T> entityCreator, int min, int max) {
    this.type = type;
    this.entityCreator = entityCreator;
    this.min = min;
    this.max = max;
  }


  @Override
  public void perform(SimulationMap simulationMap) {
    List<T> entities = simulationMap.getEntitiesBy(type);
    int current = entities.size();

    if (min > current) {
       reSpawn(simulationMap, max - current);
    }
  }

  private void reSpawn(SimulationMap simulationMap, int count) {
    for(int i = 0; i < count; i++) {
       Coordinates coordinates = getRandomFreeCoordinates(simulationMap);
       Entity entity = entityCreator.apply(coordinates);

       simulationMap.putEntity(entity, coordinates);
    }
  }
}
```

Использование:
```java
List<Action> turnActions = List.of (
  //....
  new ReSpawnAction<>(Grass.class, (c) -> new Grass(c), MIN_GRASS_COUNT, MAX_GRASS_COUNT),
  new ReSpawnAction<>(Herbivore.class, (c) -> new Herbivore(c, 1, 2), MIN_HERBIVORE_COUNT, MAX_HERBIVORE_COUNT)
);
```

Или так:
```java
List<Action> turnActions = List.of ()
  //....
  new GrassReSpawnAction(MIN_GRASS_COUNT, MAX_GRASS_COUNT),
  new HerbivoreReSpawnAction(MIN_HERBIVORE_COUNT, MAX_HERBIVORE_COUNT)
);

//...
public class GrassReSpawnAction extends ReSpawnAction<Grass> {

  public GrassReSpawnAction(int min, int max) {
    super(Grass.class, (c) -> new Grass(c), min, max);
  }
}
```

Темы: стандартные функциональные интерфейсы Java, дженерики.

**13. Классы-спавнеры: PredatorSpawnAction, RockSpawnAction etc**

Используя функциональные интерфейсы, можно упростить код и в других спавнерах.  
Сейчас так:
```java
public class GrassSpawnAction extends SpawnAction {

  public GrassSpawnAction(int count) {
    super(count); 
  }

  @Override
  protected Entity createEntity(Coordinates coordinates) {
    return new Grass(coordinates);
  }
}
```

Можно сделать так:
```java
public class GrassSpawnAction extends SpawnAction {

  public GrassSpawnAction(int count) {
    super((c) -> new Grass(c), count);  
  }
}

public abstract class SpawnAction extends Action {
  private final Function<Coordinates, Entity> entityCreator;
  private final int count;
  //...
}
```

**14. class Simulation**

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий.  
Это позволяет делать main'ы с разными игровыми конфигурациями, не меняя код в классе `Simulation`
```java
public Simulation(SimulationMap map, MapRenderer renderer, List<Action> initActions, List<Action> turnActions)
```

- Нарушение паттерна GRASP "Creator"(Создатель)

Здесь класс принимает в конструктор зависимость, которую должен создавать самостоятельно:
```java
public Simulation(SimulationMap map, MapRenderer renderer, List<Action> initActions, List<Action> turnActions) {...}

//ПРАВИЛЬНО:
public Simulation(SimulationMap map, List<Action> initActions, List<Action> turnActions) {
  //...
  this.renderer = new MapRenderer();
}
```
Creator гласит, что создавать объект должен тот, кто его использует. 

Рассмотрим, что это значит.

В данном случае `Simulation` принимает в конструктор `SimulationMap`- это правильно.  
Потому что карта может быть создана разных размеров и `Simulation` заранее не знает, какая именно карта нужна в этот раз.   
Поэтому ее создает клиент, а потом инжектит в этот класс.

Еще `Simulation` принимает в конструктор `actions`- это тоже правильно.  
Потому что список экшенов может быть разным. 

Но принимать в конструктор `MapRenderer`- уже неправильно.  
Потому что `MapRenderer` - это конкретный класс, а не интерфейс и этот класс не параметризируется. 

Экземпляр `MapRenderer` всегда одинаковый.  
Поэтому, согласно паттерну Creator, объект `MapRenderer` *здесь* должен создавать сам класс Simulation.

Технически можно создать наследника `MapRenderer`, переопределить в нем методы и передавать в конструктор Simulation.  
Но если программист хочет использовать в своем классе другие классы через полиморфизм, то он должен обозначить свои намерения более явно.  
Например, передавать в конструктор интерфейс или абстрактный класс.

- initActions должны выполняться в конструкторе.  

Метод `startSimulation()` должен делать первоначальный пуск симуляции и запускать её после паузы
```java
public Simulation(...) { //конструктор
  //...
}

public void startSimulation() {
  for (Action action : initActions) {
    action.perform(map);
  }
  //...
}

public void nextTurn() {
  for (Action action : turnActions) {
    action.perform(map);
  }
  //...
}

//ПРАВИЛЬНО:
public Simulation(...) { //конструктор
  //...
  performActions(initActions);
}

public void startSimulation() {
  //...
}

public void nextTurn() {
  performActions(turnActions);
  //...
}

private void performActions(List<Action> actions) {
  for (Action action : actions) {
    action.perform(map);
  }
}
```

- Используй инкремент
```java
turnCounter += 1;

//ПРАВИЛЬНО:
turnCounter++;
```

**15. class Main**

- Нарушение SRP.

Main должен только сконфигурировать зависимости и запустить программу.  
Управлять работой программы этот класс не должен.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

Определи сущности, которые сейчас сидят в этом классе и вынеси их в отдельные классы.  
Например, сущность, которая управляет симуляцией, получая от юзера команды пауза/пуск.

Создание списков экшенов в мейне тоже выглядит сложным- эту логику можно вынести в фабрику экшенов.

## ВЫВОД

Замечаний меньше, чем обычно. В целом норм.

n.175(377)  
#ревью #симуляция  