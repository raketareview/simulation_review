https://github.com/ArtemYaskov03/Simulation2  
[Peen]

После рефакторинга: вторая версия программы.  
Ревью на первую версию [ТУТ](https://github.com/raketareview/simulation_review/blob/master/content/rev-sim146-peen-ArtemYaskov03.md).

После рефакторинга стало лучше.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Ввод размеров куда-то съехал

```java
Введите высоту поля (мин 10 макс 100):
Введите длину поля (мин 10 макс 100):100
100
```

2. Паузу и повторный пуск нужно запускать одной и той же командой, а то прям сложные правила какие-то.  
Почему одни команды состоят из одного символа, а другие- из 2-х?
```java
Команды: 
Запустить симуляцию: 1; Сделать один ход: 2; Cелать паузу: 0; Возобновить симуляцию: 11; Завершить: 00; 

//ЛУЧШЕ:
1 - Бесконечная симуляция/пауза, 2 - Выполнить один ход, 3 - Выход. 
```

3. Я хочу смотреть на карту с волками и зайчиками, а не на кучу текстов
```
Steps 4
Wolf: Coordinates: Coordinates[x=1, y=4]
Wolf: Coordinates: Coordinates[x=2, y=0]
Cow: Hp: 10 Coordinates: Coordinates[x=0, y=63]
Cow: Hp: 10 Coordinates: Coordinates[x=0, y=65]
Cow: Hp: 10 Coordinates: Coordinates[x=1, y=37]

<В СУММЕ 300 СТРОК ТЕКСТА МЕЖДУ РАСПЕЧАТКОЙ КАРТ>
```
Убери распечатку этих текстов, оно никому не надо.

3. Игра вылетает
```java
Wolf: Coordinates: Coordinates[x=98, y=85]
Cow: Hp: 5 Coordinates: Coordinates[x=98, y=95]
Wolf: Coordinates: Coordinates[x=99, y=93]
Exception in thread "Thread-0" java.lang.RuntimeException: Ошибка в поиске пути
	at way.AStar.findWay(AStar.java:112)
	at entity.Creature.makeMove(Creature.java:21)
```

4. Карта печатается неправильно.

## ХОРОШО

+ 👍 Алгоритм поиска AStar
+ 👍 Создание карты произвольных размеров через меню

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов нужно писать стилем alllowercase 
```java
package worldMap;

//ПРАВИЛЬНО:
package worldmap;
```

- Одним и тем же названием нельзя называть разные типы данных.  

Здесь "coordinatesAndEntity" в одном случае это `Map`- в контексте возвращаемого значения метода `Map<Coordinates, Entity> getCoordinatesWithEntity()`.  
А в другом случае переменная с таким же названием относится к типу `Map.Entry`
```java
for (Map.Entry<Coordinates, Entity> coordinatesAndEntity : getCoordinatesWithEntity().entrySet()) {
  Coordinates coordinates = coordinatesAndEntity.getKey();
  //...
}

//ПРАВИЛЬНО:
for (Map.Entry<Coordinates, Entity> entry : getCoordinatesWithEntity().entrySet()) {
  Coordinates coordinates = entry.getKey();
  //...
}
```

- Не называй поля "переменная1", "переменная2", это совсем плохой тон.

Всегда можно найти более осмысленные названия.  
Если в текущем пространстве имен название "entity" занято, то можно дать уточняющее название
```java
public Coordinates getCoordinates(Entity entity) {
  for (...) {
    Coordinates coordinates = coordinatesAndEntity.getKey();
    Entity entity1 = coordinatesAndEntity.getValue();
    //...
  }
}

//ПРАВИЛЬНО:
public Coordinates getCoordinates(Entity entity) {
  for (...) {
    Coordinates currentCoordinates = coordinatesAndEntity.getKey();
    Entity currentEntity = coordinatesAndEntity.getValue();
    //...
  }
}
```

- Коллекции нужно называть во множественном числе
```java
List<Entity> getAllEntity()

//ПРАВИЛЬНО:
List<Entity> getAllEntities()
```

- Правильная дихотомия здесь будет "my-your" или "start-finish"
```java
int searchH(Coordinates myCoordinates, Coordinates finishCoordinates)

//ПРАВИЛЬНО:
int searchH(Coordinates start, Coordinates finish)
```

- Класс не должен называться в множественном числе
```java
class Items

//ПРАВИЛЬНО:
class Item
```

- Название метода должно быть глаголом в повелительном наклонении.  
Это название метода- существительное
```java
void actionEntity(Entity entity, Coordinates coordinates, WorldMap worldMap)
```

- В названия не нужно вставлять частицы "Of", "The", "Are" и т.д.
Это только делает названия более громоздкими.  
Если, конечно, там "Of" не используется в контексте "valueOf()"
```java
void populateTheMap()
void executeTheCommand()
```

- Метод печатает `WorldMap`, а не `Board`
```java
public void printBord(WorldMap worldMap) {...}

//ПРАВИЛЬНО:
public void print(WorldMap worldMap) {...}
```

- Придерживайся единообразия.

Или "Printer/print", или "Renderer/render"
```java
public class Renderer {

  public void printBord(WorldMap worldMap) {...}
}

//ПРАВИЛЬНО:
public class Renderer {

  public void render(WorldMap worldMap) {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки  
```java
if (withinMap(coordinates)) {
  return coordinatesWithEntity.get(coordinates);
} else throw new RuntimeException("Данная координата находится вне карты");

//ПРАВИЛЬНО:
if (withinMap(coordinates)) {
  return coordinatesWithEntity.get(coordinates);
} else {
  throw new RuntimeException("Данная координата находится вне карты");
}
```
Исключение- метод equals(), там можно после if не выделять блоки скобочками.  
*"Oracle Java Code Conventions"*  

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```java
System.out.print("Запустить симуляцию: 1; Сделать один ход: 2; Cелать паузу: 0; Возобновить симуляцию: 11; Завершить: 00; ");

switch (command) {
  case "1": //...
  case "0": //...

  //...
}

//ПРАВИЛЬНО:
private static final String START = "1";
private static final String PAUSE = "0";
//...

System.out.printf("Запустить симуляцию: %s; Сделать один ход: %s; Cелать паузу: %s; Возобновить симуляцию: %s; Завершить: %s;  \n", START, ONE_TURN, ...);

switch (command) {
  case START: //...
  case PAUSE: //...

  //...
}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru, "Замена магического числа символьной константой"*

**3. Exceptions**

- Не бросай базовый exception. Конкретизируй ситуацию- бросай то исключение, которое подходит под этот конкретный случай
```java
throw new RuntimeException("Данная координата находится вне карты");
```
*Хорстманн "Java. Библиотека профессионала", т.1, гл.11*
```java
"Не ограничивайтесь генерацией RuntimeException. 
Найдите подходящий подкласс или создайте собственный." - Хорстманн
```

- Текст в Exception всегда должен быть на английском языке.
```java
throw new RuntimeException("Данное существо не существует");
```
Исключение это не просто телеграмма, которая летит сквозь слои.

У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.  
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты.  
А значит, сообщение должно быть на английском.

Интерпретация исключения и перевод его на локальный язык должны происходить там, где это соответствует архитектуре программы.  
Или не происходить вовсе, если исключение не планируется перехватывать.

**4. Используй классы через их интерфейсы**
```java
ArrayList<Coordinates> getNeighbors(Coordinates coordinates)
ArrayList<Coordinates> neighbors = new ArrayList<>();

//ПРАВИЛЬНО:
List<Coordinates> getNeighbors(Coordinates coordinates)
List<Coordinates> neighbors = new ArrayList<>();
```
Общее правило: ArrayList нужно использовать через List, HashMap через Map, HashSet через Set и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**5. Повторяющийся код выноси во вспомогательные методы**

```java
public void populateTheMap() {
  for (int numCow = 0; numCow < countCow; numCow++) {
    worldMap.putEntityOnRandomCoordinates(new Cow());
  }
  for (int numWolf = 0; numWolf < countWolf; numWolf++) {
    worldMap.putEntityOnRandomCoordinates(new Wolf());
  }
  for (int numGrass = 0; numGrass < countGrass; numGrass++) {...}
  //...
}

//ПРАВИЛЬНО:
public void populateTheMap() {
  spawn(() -> new Cow(), countCow);
  spawn(() -> new Wolf(), countWolf);
  //...
}

private void spawn(Supplier<Entity> entitySupplier, int count) {
  for (int i = 0; i < count; i++) {
    Entity entity = entitySupplier.get();
    worldMap.putEntityOnRandomCoordinates(entity);
  }
}
```

"Передачу действий" между методами можно делать через стандартные функциональные интерфейсы Java (гугли тему).  
Например, тут я через стандартный интерфейс `Supplier` передаю способ создания объекта-энтити в метод, который создает нужно количество существ определенного типа. 

**6. record Coordinates(int x,int y)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```java
public record Coordinates(int x,int y) {
}
```

**7. class WorldMap**

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.), нужно проверять координату на корректность  
```java
public void deleteEntity(Coordinates coordinates) {
  coordinatesWithEntity.remove(coordinates);
}

//ПРАВИЛЬНО:
public void deleteEntity(Coordinates coordinates) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение  
  coordinatesWithEntity.remove(coordinates);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```java
private final Map<Coordinates, Entity> coordinatesWithEntity = new HashMap<>();

Map<Coordinates, Entity> getCoordinatesWithEntity() {
  return coordinatesWithEntity;
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
карта.getCoordinatesWithEntity().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

Нужно вернуть не оригинал мапы, а ее копию: 
```java
Map<Coordinates, Entity> getCoordinatesWithEntity() {
  return new HashMap(coordinatesWithEntity);
}
```

- Лишние методы.
```java
Map<Coordinates, Entity> getCoordinatesWithEntity() {  <-- Этого метода достаточно
  return coordinatesWithEntity;
}

public List<Coordinates> getAllCoordinates() {  <-- Этот метод лишний
  return new ArrayList<>(coordinatesWithEntity.keySet());
}

public List<Entity> getAllEntity() {  <-- Этот метод лишний
  return new ArrayList<>(coordinatesWithEntity.values());
}
```

В Карте есть уже метод, который возвращает мапу с существами и координатами:
```java
Map<Coordinates, Entity> getCoordinatesWithEntity()
```
Поэтому в Карте нет смысла содержать еще 2 отдельных метода, которые возвращают список координат и список существ.  
Если какому-то клиенту это понадобится, он может сам их получить:  
```java
Map<Coordinates, Entity> coordinatesWithEntity = worldMap.getCoordinatesWithEntity();
List<Coordinates> allEntities = coordinatesWithEntity.values();
```

- Нарушение SRP.

Кажется, это единственный метод в классе, который нарушает его SRP:
```java
public void putEntityOnRandomCoordinates(Entity entity) {
  //ставит entity на случайную координату
}
```

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Метод, который записывает существо на случайную координату в карте, не нужен карте для выполнения своих обязанностей по хранению существ.    
Метод, который записывает существо на конкретную координату- нужен карте.  
Метод, который записывает существо на случайную координату- не нужен карте.  

Этот метод нужно перенести в тот класс, частью SRP которого он является.  
Найти такой класс очень просто- нужно посмотреть, какой класс вызывает этот метод.  
Такой класс есть только один, это `InitActions`.

+ 👍 В целом с точки зрения SRP класс хороший. Имеющиеся нарушения SRP в нем- незначительные

**8. class Way**

+ 👍 Хороший класс, упрощает алгоритм поиска.

- Нарушение инкапсуляции.

Всегда явно указывай модификатор доступа, даже когда он protected
```java
public class Way {
  Coordinates coordinates;
  int h;
  //...
}

//ПРАВИЛЬНО:
public class Way {
  protected Coordinates coordinates;
  protected int h;
  //...
}
```

Поля и методы без явно указанного уровня доступа вызывают подозрения в том, что автор просто забыл указать нужный уровень, а уровень default при этом может не соответствовать необходимому.

**9. class AStar**

- Расположение методов в классе. 

Публичные методы должны стоять выше вспомогательных приватных методов.  
Если в классе есть только один публичный метод, то он должен стоять сразу после конструктора.

- Если в проекте есть класс `Way`, то все переменные с именем, включающим это название, должны быть экземплярами этого класса.  

Когда разные концепции называются одним и тем же именем, это приводит к путанице
```java
public class Way {...}

public class AStar {

  public List<Coordinates> findWay() {...}  <-- Метод возвращает List, а не Way
}

//ПРАВИЛЬНО:
public class Way {...}

public class AStar {

  public List<Coordinates> findPath() {...}  
}
```

- Из сигнатуры метода поиска неясно, как им пользоваться.
```java
public class AStar {
  private final WorldMap worldMap;
  private final Entity entity;

  public AStar(WorldMap worldMap, Entity entity) {...}

  public List<Coordinates> findWay() {...}
}
```

Поиск по алгоритму A* должен искать на карте путь между двумя точками.  
Например так:
```java
public class AStar {
  private final WorldMap worldMap;

  public AStar(WorldMap worldMap) {...}

  public List<Coordinates> findPath(Coordinates start, Coordinates finish) {...}
}
```

Также поиск пути по алгоритму A* может использовать ту же сигнатуру, что и BFS.  
Тогда поиск пути в нем интерпретируется не как поиск пути между двумя точками, а как поиск цели и прокладывание пути к ней.  
Например: 
```java
public class AstarPathFinder {

  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
    //сначала находит координату цели
    //потом прокладывает путь по алгоритму A* между точкой старта и точкой цели
  }
}
```
Плюсы- унификация сигнатуры разных алгоритмов поиска для полиморфизма.  
То есть, можно сделать семейство родственных классов поиска, объединенных общим интерфейсом.

- Нарушение SRP. 

Класс должен просто искать путь на карте согласно алгоритмам BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем опроса существа или карты
```java
public List<Coordinates> findWay() {
  Coordinates start = worldMap.getCoordinates(entity);
  Coordinates finish = findNearlyCoordinates((Creature) entity);
  //ищет путь между start и finish
}

//ПРАВИЛЬНО:
public List<Coordinates> findPath(Coordinates start, Coordinates finish) {
  //ищет путь между start и finish
}
```

- Не передавай `null` в методы в качестве корректного значения
```java
new Way(coordinates, null, 0, searchH(coordinates, finish)

public class Way {
  //...

  Way(Coordinates coordinates, Way parent, int g, int h) {
    this.coordinates = coordinates;
    //...
  }
}

//ПРАВИЛЬНО:
new Way(coordinates, 0, searchH(coordinates, finish)

public class Way {
  //...

  Way(Coordinates coordinates, Way parent, int g, int h) {
    this(coordinates, g, h);
    this.parent = parent;
  } 

  Way(Coordinates coordinates, int g, int h) {
    this.coordinates = coordinates;
    //...
  }
}
```
*Мартин "ЧК", гл.7*
```
"Возвращать null из методов плохо, но передавать null при вызове еще хуже" - Мартин.
```

- Повторяющиеся действия делай через циклы
```java
private ArrayList<Coordinates> getNeighbors(Coordinates coordinates) {
  //...
  List<Coordinates> potentialNeighbor = new ArrayList<>();
  potentialNeighbor.add(new Coordinates(x, y + 1));
  potentialNeighbor.add(new Coordinates(x, y - 1));
  potentialNeighbor.add(new Coordinates(x + 1, y));
  potentialNeighbor.add(new Coordinates(x - 1, y));
  for (Coordinates neighbor : potentialNeighbor) {
    //...
  }
  return neighbors;
}

//ПРАВИЛЬНО:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(new Coordinates(0, 1), new Coordinates(0, -1), ...);

private List<Coordinates> getNeighbors(Coordinates coordinates) {
  //...
  for (Coordinates shift : SHIFT_COORDINATES) {
    Coordinate potential = new Coordinates(coordinates.x() + shift.x(), coordinates.y() + shift.y());
    //...
  }
  return neighbors;
}
```

**10. abstract class Entity и его простые наследники Tree/Rock/Grass**

- Норм. Но непонятно, зачем нужен пустой класс-посредник Items
```java
public abstract class Entity {
}

public abstract class Items extends Entity{
}

public class Rock extends Items{
}
```

Вернее я знаю, зачем- его ввели для использования в алгоритме поиска класса `AStar`
```java
public class AStar {

  public List<Coordinates> findWay() {
    //...
    if (worldMap.getEntity(coordinates) instanceof Items) {...}
    //...
  }
}
```
Но это просто еще одно свидетельство того, что класс поиска нарушает SRP.  
Для корректного алгоритма, поиску не нужно анализировать объект на карте на принадлежность к типу `Items`.  
Поиск A* должен просто искать путь между двумя точками.

Поэтому правильная иерархия должна выглядеть так
```java
public abstract class Entity {
}

public class Rock extends Entity {
}
```

**11. abstract class Creature extends Entity**

+ 👍 В целом ок.

**12. Наследники Creature: Herbivore, Predator, Cow etc**

+ 👍 В целом ок.

**13. Пакет command**

В ТЗ указаны классы Action's, поэтому пакет должен называться action, содержащиеся в нем классы- Action'ми. 

**14. class Actions**

Этот класс не соответствует описанию классов Action's в ТЗ:
```java
public class Actions {
  protected final WorldMap worldMap;
  protected final int height;
  protected final int width;
  //...

  public Actions(WorldMap worldMap) {
    this.worldMap = worldMap;
    height = worldMap.getHeight();
    width = worldMap.getWidth();
    this.square = height * width;
    this.countCow = square / 50;
    //...
  }
}
```

**15. interface Command**

+ 👍 Отлично. Но по ТЗ это должно называться "Action"
```java
public interface Command {
  public void execute();
}

//ПРАВИЛЬНО:
public interface Action {
  void execute();
}
```
Писать `public` в сигнатурах методов в интерфейсах- избыточно. 

**16. class CommandExecutor**

Ацкий ад. Класс удалить. 

**17. class GenerateNewEntityCommand**

Ацкий ад. Класс удалить.

**18. Классы с названием "Action"**

Здесь есть много классов с названиями, в которых есть слово "Action"
```java
public class InitActions extends Actions {

  public void populateTheMap() {..}
}

public class TurnActions extends Actions {

  public void makeMoveEveryone() {...}
  public void generateNewEntity() {...}
}
```

Но эти классы не соответствуют описанию классов Action's в ТЗ:
```java
Actions #
Action - действие, совершаемое над миром. Например - сходить всеми существами... 
Каждое действие описывается отдельным классом и совершает операции над картой. 
```

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.

**В каждом экшене должен быть только один публичный метод(не публичных может быть сколько угодно).**

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

В этом проекте семейство экшенов должно выглядеть примерно так:
```java
public interface Action {
  void execute(Карта карта);
}

public class InitSpawnAction implements Action {

  @Override
  public void execute(Карта карта) {
     //НАЧАЛЬНОЕ ЗАПОЛНЕНИЕ КАРТЫ МНОЖЕСТВОМ СУЩЕСТВ 
  }
}

public class MoveAction implements Action {

  @Override
  public void execute(Карта карта) {
     //ПЕРЕДВИЖЕНИЕ СУЩЕСТВ ПО КАРТЕ
  }
}
```

Дополнительно могут быть еще другие классы-экшены, например:  
Экшен поддержания популяции существ на карте; экшен внезапной эпидемии etc.

В любом случае, каждый из этих экшенов должен делать только ОДНО действие.  
И это действие он должен делать над КАРТОЙ.

Поэтому рассматривать подробно каждый класс в этом пакете я не буду- их все нужно переписать заново.

**19. class Renderer**

+ 👍 Спрайты существ хранятся здесь, а не берутся из самих существ, это хорошо.

- Неправильно распечатывает карту
```java
public class TestMain {
  static void main() {
    WorldMap worldMap = new WorldMap(6, 6);
    Renderer renderer = new Renderer();

    int x = 0;
    int y = 4;

    worldMap.putEntity(new Tree(), new Coordinates(x, y));

    renderer.printBord(worldMap);
  }
}

//РЕЗУЛЬТАТ:
 *   *   *   *   🌳  *   
 *   *   *   *   *   *   
 *   *   *   *   *   *   
 *   *   *   *   *   *   
 *   *   *   *   *   *   
 *   *   *   *   *   *   
```

**20. class Simulation**

- Нарушение инкапсуляции.

Всегда явно указывай модификаторы доступа полей.

- Нарушение DI. Класс не должен сам себя конструировать.  

В данном случае класс конструирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```java
public class Simulation {
  WorldMapCreator worldMapCreator = new WorldMapCreator();
  WorldMap worldMap = worldMapCreator.createMap();
  //...
}

//ПРАВИЛЬНО:
public class Simulation {
  private final WorldMap worldMap;
  //...

  public Simulation(WorldMap worldMap) {
     this.worldMap = worldMap;
     //...
  }
}
```
Таким образом нельзя без изменения кода в классе `Simulation` создать несколько игровых конфигураций с разными картами.  
Например, с картой заранее выбранного размера.   

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Неправильное использование потоков в проекте.

Внутри класса `Simulation` не должно быть никакой работы с потоками
```java
public class Simulation {
  //...

  public void executeTheCommand() {
    Thread thread = new Thread(this::startSimulation);
    //...
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

Один простой Main должен просто запустить симуляцию в бесконечном цикле без возможности управлять её работой через команды паузу/пуск.
Примерно так:
```java
public class MainWithoutThreads {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);
    simulation.startSimulation(); //бесконечная симуляция без потоков 
  }
}

По запуску второго симуляция должна работать так, как сейчас- с возможностью делать паузу/пуск.  
Примерно так:
```java
public class MainWithThreads {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);

    SimulationManager manager = new SimulationManager(simulation);  //принимает от юзера команды и дергает симуляцию за методы паузу/пуск 

    manager.start(); //симуляция с потоками и командами  пауза/пуск
  }
}
```

## ВЫВОД

Разобраться с классами Action, сделать их по ТЗ и моим рекомендациям.  
Работу с потоками вынести в отдельный класс.

Рефакторинг пошел на пользу- сейчас программа выглядит значительно лучше.  
Значит, ты растёшь как программист, а в этом и состоит цель рефакторинга после ревью.

n.168(363)  
#ревью #симуляция 