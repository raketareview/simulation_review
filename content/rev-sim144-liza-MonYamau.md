https://github.com/MonYamau/Simulation  
[Лиза]

В целом норм. Нужно переделать использование потоков.

## ХОРОШО

+ 👍 Визуальная эстетика просто вау!   
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim144/img0.png) 
+ 👍 Альтернативная трактовка: мышки, котики и сыр
+ 👍 Перед началом работы показываются правила
+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализованы пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для избежания путаницы не называй пакеты, классы, интерфейсы так же, как называются классы и интерфейсы стандартной библиотеки.
Есть стандартный интерфейс `Map`, не должно быть кастомного пакета с таким же названием
```java
package main.java.map;

//ПРАВИЛЬНО:
package main.java.gamemap;
```

- Геттер должен что-то возвращать. Геттер, который ничего не возвращает- нонсенс
```java
void getFood(Coordinates targetCoordinates, GameMap gameMap);
```

- Не нужно без особой нужды сокращать названия. Тем более, если полное название будет коротким
```java
record Coordinates(int col, int row)

//ЛУЧШЕ:
record Coordinates(int column, int row)
```

- Старайся использовать стандартные имена.  
Для обозначения геометрических размеров обычно используется пара width/height
```java
public class GameMap {
  private final int maxColumnValue;
  private final int maxRowValue;
  //...
}

//ЛУЧШЕ:
public class GameMap {
  private final int width;
  private final int height;
  //...
}
```

- В названиях придерживайся единообразия
```java
public Optional<Entity> getEntity(Coordinates coordinates)   <-- ПОЧЕМУ БЕЗ of?
public Optional<Coordinates> getCoordinatesOf(Entity entity) <-- ПОЧЕМУ С of?

//ПРАВИЛЬНО:
public Optional<Entity> getEntity(Coordinates coordinates) {
public Optional<Coordinates> getCoordinates(Entity entity) 
```

- Это константа.  
По стандартам не только конвенции Oracle, но и Google, здесь название должно быть написано UPPER_SNAKE-  
`Set.of(...)` создает `immutable` список
```java
private static final Set<Coordinates> shifts = Set.of(...);

//ПРАВИЛЬНО:
private static final Set<Coordinates> SHIFTS = Set.of(...);
```

- Рендереры рисуют изображения.  
Этот класс печатает текстовые сообщения, поэтому он принтер. 
На что нам и намекают названия его методов 
```java
class ScriptRenderer {
  public static void printWelcomeMessages()
  //...
}

//ЛУЧШЕ:
class MessagePrinter
```

- Та же ситуация, но наоборот:
```java
public class GameMapRenderer {
  public void printGameMap(GameMap gameMap) {
    //рисует карту
  }
}

//ПРАВИЛЬНО:
public class GameMapRenderer {
  public void render(GameMap gameMap) {...}
}

//ТАК ТОЖЕ МОЖНО:
public class GameMapPrinter {
  public void print(GameMap gameMap) {...}
}
```

- Избыточно. Мы понимаем, что фабрика существ производит именно существ, а не что-то иное
```java
public class EntityFactory {
  public <T extends Entity> T createEntity(Class<T> entityClass) {...}
}

//ПРАВИЛЬНО:
public class EntityFactory {
  public <T extends Entity> T create(Class<T> entityClass) {...}
}
```

- Не дублируй имена классов в названии их методов
```java
public abstract class Creature бла-бла {
  private void moveCreature(...) {...}
}

//ПРАВИЛЬНО:
public abstract class Creature бла-бла {
  private void move(...) {...}
}
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется.**
 
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без `else` будет выглядеть читабельней
```java
if (isEvenCell(coordinates)) {
  return "%s%s%s".formatted(ANSI_GREY_BACKGROUND_COLOR, EMPTY_CELL, ANSI_RESET);
} else {
  return "%s".formatted(EMPTY_CELL);
}

//ПРАВИЛЬНО:
if (isEvenCell(coordinates)) {
  return "%s%s%s".formatted(ANSI_GREY_BACKGROUND_COLOR, EMPTY_CELL, ANSI_RESET);
} 
return "%s".formatted(EMPTY_CELL);
```

**3. Нарушение конвенции кода**

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки.  
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```java
if (!(coordinates.col() < maxColumnValue && coordinates.col() >= 0)) return false;

//ПРАВИЛЬНО:
if (!(coordinates.col() < maxColumnValue && coordinates.col() >= 0)) {
  return false;
}
```
*"Oracle Java code conventions"*  

**4. record Coordinates(int col, int row)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```java
public record Coordinates(int col, int row) {
}
```

- Расположение класса вызывает вопросы.  
Сейчас он находится в пакете "service".  
Но я бы сказал, что этоот класс скорее тяготеет к карте и лучше ему было бы находиться в пакете "gamemap".

**5. class GameMap**

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей: создать список пустых координат 
```java
public List<Coordinates> getEmptyCells()
```

Проекта в целом нужно иметь метод, который находит все пустые координаты в карте.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно находит все пустые координаты.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

+ 👍 Геттеры не возвращают null, они возвращают Optional, это хорошо
```java
public Optional<Entity> getEntity(Coordinates coordinates)
```

+ 👍 При операциях с участием координаты, она валидируется.  
Это хорошо- это защищает от некоректных операций с координатами, которые находятся вне пределов карты
```java
public Optional<Entity> getEntity(Coordinates coordinates) {
  if (isValidCoordinates(coordinates)) {
    return Optional.ofNullable(entities.get(coordinates));
  }
  throw new IllegalArgumentException("invalid coordinates received: " + coordinates);
}
```

- Но тут, чтобы не повторяться в разных методах и не дублировать код и текстовые сообщения, лучше сделать так:
```java
public void removeEntity(Coordinates coordinates) {
  if (isValidCoordinates(coordinates)) {
    entities.remove(coordinates);
    return;
  }
  throw new IllegalArgumentException("invalid coordinates received: " + coordinates);
}

//ПРАВИЛЬНО:
public Optional<Entity> getEntity(Coordinates coordinates) {
  validate(coordinates);
  return Optional.ofNullable(entities.get(coordinates));
}

public void removeEntity(Coordinates coordinates) {
  validate(coordinates);
  entities.remove(coordinates);
}

private void validate(Coordinates coordinates) {
  if (!isValidCoordinates(coordinates)) {
    throw new IllegalArgumentException("invalid coordinates received: " + coordinates);
  }
}
```

- Попытка удалить существо на несуществующей координате это недопустимая операция
```java
public void removeEntity(Coordinates coordinates) {
  if (isValidCoordinates(coordinates)) {
    entities.remove(coordinates);
    return;
  }
  //...
}

//ПРАВИЛЬНО:
public void removeEntity(Coordinates coordinates) {
  if (isValidCoordinates(coordinates)) {
    if(isCellEmpty(coordinates)) {
      //кинуть исключение- нельзя удалять то, чего нет  
    }
    entities.remove(coordinates);
    return;
  }
  //...
}
```
Если алгоритм пытается удалить существо там, где его нет, значит алгоритм считает, что там существо есть.  
Следовательно, в алгоритме есть какой-то баг и он работает некорректно.  
Иначе, зачем бы ему было выполнять бессмысленное действие.  
Именно поэтому нужно кинуть исключение.

- Дженерик тут лишний.

Карта должна работать со всем семейством существ на уровне базового класса Entity.  
Карту не должно волновать, какого конкретно энтити в нее помещают- траву или крокодила
```java
public <T extends Entity> void putEntity(Coordinates coordinates, T entity)

//ПРАВИЛЬНО:
public void putEntity(Coordinates coordinates, Entity entity)
```

Вообще, использование дженериков в карте это хорошая идея, но на уровне карты в целом.  
Например, так:
```java
public class GameMap<T> {
  private final Map<Coordinates, T> values;
  //...
}
```

Тогда можно делать универсальные игровые карты для разных программ- шахматы, морской бой, что угодно:
```java
public class ChessBoard extends GameMap<ChessPiece> {...}
public class SimulationGameMap extends GameMap<Entity> {...}
```
*Но это уже совсем другая история.*

- Положительные условия читаются лучше отрицательных.  
Обычно одно можно легко развернуть в другое
```java
public boolean isValidCoordinates(Coordinates coordinates) {
  if (!(coordinates.col() < maxColumnValue && coordinates.col() >= 0)) return false;
  //...
}

//ПРАВИЛЬНО:
public boolean isValidCoordinates(Coordinates coordinates) {
  if (coordinates.col() >= 0 && coordinates.col() < maxColumnValue) {
    return true;
  }  
  //...
}
```
*"ЧК", гл.17, G29 "Избегайте отрицательных условий"*

+ 👍 Прекрасный метод
```java
public <T extends Entity> List<T> getEntitiesByType(Class<T> entityClass) 
```
Да, тут тоже используется дженерик, но тут его использование оправдано-
карту просят вернуть список не всех существ, а существ определенного класса, которые заданы дженериком.  
При этом карта все так же одинаково работает со всеми видами существ исключительно на уровне базового Entity.  
А тип класса, заданный дженериком и переданный в метод, выступает просто в качестве критерия для отбора объектов.

+ 👍 В целом, с точки зрения единой ответственности(SRP) класс хороший.  
Имеющиеся нарушения SRP в нем- незначительны.

**Пакет service**

- Термин "service".

"Service" это название, которым принято называть определенные классы или группу классов- то есть, слой.  
Обычно слой "service" используется в конкретных архитектурах- MVC(S), где "S" означает сервис. Например, в Spring MVC.  
Или в других архитектурах с анемичной моделью, где модели содержат только состояние, а их поведение выносится в специальные классы-сервисы.  
В сервисах есть поведение, но у них нет состояния.  
Если программа не сделана в архитектуре, в которой предусмотрены сервисы, то лучше так не называть пакеты и классы.

- В пакете все классы не соответствуют критериям сервисов.  
Например, `Coordinates` это не сервис, а простейшая модель.

**6. interface PathFinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
public interface PathFinder {
  List<Coordinates> find(Coordinates start, Class<?> target, GameMap gameMap);
}
```

- Нужно уточнить дженерик.

Поиск работает с картой. Карта содержит объекты типа `Entity`.  
Поиск должен искать на карте объекты, которые являются Entity(базовый класс или его наследники):
```java
public interface PathFinder {
  List<Coordinates> find(Coordinates start, Class<? extends Entity> target, GameMap gameMap);
}
```

Иначе можно попытаться найти в карте существ, которых там даже теоретически быть не может.  
Например:
```java
List<Coordinates> path1 = pathFinder.find(current, HTMLDocument.HTMLReader.TagAction.class, gameMap);
```

**7. class Node**

👍 Вспомогательный класс для поиска.  
Это упрощает алгоритм поиска и делает его понятнее.

**8. class BfsPathFinder implements PathFinder**

- Не забывай про инкапсуляцию
```java
public class BfsPathFinder implements PathFinder {
  Queue<Node> check;
  Set<Coordinates> checked;
  //...
}

//ПРАВИЛЬНО:
public class BfsPathFinder implements PathFinder {
  private Queue<Node> check;
  private Set<Coordinates> checked;
  //...
}
```

+ 👍 Особых недостатков к классе не вижу.

**9. abstract class Entity и его простые наследники Cheese/Box/Basket**

👍 Идеально
```java
public abstract class Entity {
}

public class Cheese extends Entity {
}
```

**10. abstract class Creature extends Entity**

- Лишняя проверка
```java
public void makeMove(GameMap gameMap) {
  Optional<Coordinates> currentCoordinates = gameMap.getCoordinatesOf(this);
  if (currentCoordinates.isEmpty()) {  <-- ЛИШНЯЯ ПРОВЕРКА
    return;
  }
  //выполнение хода
}
```
Если существо спрашивает свою координату из карты, а в карте такой координаты нет, значит алгоритм карты работает некорректно.  
Или некоректно работает алгоритм программы в целом- раз какой-то несуществующий на карте дух пытается получить из нее свои координаты.

В любом случае, это исключительная ситуация, которую нельзя игнорировать.  
А нужно как можно быстрее выявить и устранить в алгоритме причины ее возникноверрия.  
Для этого можно кинуть исключение:
```java
public void makeMove(GameMap gameMap) {
  Optional<Coordinates> currentCoordinates = gameMap.getCoordinatesOf(this);
  if (currentCoordinates.isEmpty()) { 
    //кинуть исключение
  }
  //выполнение хода
}
```

Но особого смысла тут нет, потому что и без этой проверки при null-координате произойдет исключение, но чуть ниже.  
Поэтому лучший вариант здесь- не использовать дополнительных проверок.  
Если в алгоритме будет баг, то быстрое падение в этом месте и так произойдет само по себе 
```java
public void makeMove(GameMap gameMap) {
  Optional<Coordinates> currentCoordinates = gameMap.getCoordinatesOf(this);
  Coordinates currentCell = currentCoordinates.get();  <-- если в Optional содержит null, то здесь вылетит NoSuchElementException
  //выполнение хода
}
```

- Непонятный геттер- не возвращает еду, только дразнит
```java
protected abstract void getFood(Coordinates targetCoordinates, GameMap gameMap);
```

+ 👍 В целом, класс норм.

**11. abstract class Survivor extends Creature**

- Теперь понятно, что это за метод- он не возвращает еду, а ест еду
```java
public abstract class Survivor extends Creature {
  //...
  @Override
  protected void getFood(Coordinates targetCoordinates, GameMap gameMap) {
    gameMap.removeEntity(targetCoordinates);  <-- Удаляет жертву с карты(съел еду)
    this.setHp(this.getHp() + this.getSaturation());  <-- Поправил свое здоровье съеденными калориями
  }
}

//ПРАВИЛЬНО:
@Override
protected void eat(Coordinates targetCoordinates, GameMap gameMap) {
  gameMap.removeEntity(targetCoordinates); 
  this.setHp(this.getHp() + this.getSaturation());
}
```

Но тут я бы уточнил- если метод должен есть еду, то он для поедания должен получить именно еду.  
А не координату, по которой должен взять еду:
```java
//ЕЩЕ БОЛЕЕ ПРАВИЛЬНО:
@Override
protected void eat(Entity food, GameMap gameMap) {
  gameMap.removeEntity(food); 
  this.setHp(this.getHp() + this.getSaturation());
}
```
Карту этот метод тоже должен получить- потому поедание оканчивается удалением еды из карты.  
В карте можно сделать перегруженные методы удаления существа, это логично и не противоречит её SRP:
```java
public class GameMap {
  //...
  public void removeEntity(Coordinates coordinates) {...}
  public void removeEntity(Entity entity) {...}
}
```

**12. Остальные Entity**

👍 Всё ок. Только переделать методы `getFood(...)` на `eat(...)`.

**13. class EntityFactory**

+ 👍 Фабрика это хорошо
```java
public class EntityFactory {
  //...
  public <T extends Entity> T createEntity(Class<T> entityClass) {...}
}
```

+ 👍 Реализация норм.

- Фабрика существ создает существ, но сама не является существом.  
Поэтому не должна лежать в пакете entity.

**14. abstract class Action**

Всё прекрасно.  
Но чисто из практических соображений, немного лучше не принимать карту в конструктор.  
А принимать ее в метод выполнения действия:
```java
public abstract class Action {
  protected GameMap gameMap;

  public Action(GameMap gameMap) {
    this.gameMap = gameMap;
  }

  public abstract void perform();
}

//ПРАКТИЧНЕЕ:
public abstract class Action {
  public abstract void perform(GameMap gameMap);
}
```

**15. class SpawnAction, ResourceProvider, GameMapInitializer**

👍 Вроде бы, норм.

**16. class TurnMovement extends Action**

👍 Идеально
```java
public class TurnMovement extends Action {
  //...
  @Override
  public void perform() {
    List<Creature> creatures = gameMap.getEntitiesByType(Creature.class);
    for (Creature creature : creatures) {
      if (creature.isAlive()) {
        creature.makeMove(gameMap);
      }
    }
  }
}
```

**17. final class ScriptRenderer**

👍 Отличный класс. 

Здесь происходит распечатка всех текстов.  
Теперь вся текстовая печать идет через него
```java
public final class ScriptRenderer {
  //...
  public static void printWelcomeMessages() {...}
  public static void printTurnMessages(int counter) {...}
  public static void printIncorrectInputScript() {...}
}
```

Остается полшага до создания слоя View и отделения, тем самым, логики в программе от представления:
```java
public interface MessagePrinter {
  //...
  void printWelcomeMessages() {...}
  void printTurnMessages(int counter) {...}
  void printIncorrectInputScript() {...}
}

public class ConsoleMessagePrinter implements MessagePrinter {  <-- ПЕЧАТАЕТ В КОНСОЛЬ
  //...
  @Override
  public void printWelcomeMessages() {...}

  @Override
  public void printTurnMessages(int counter) {...}

  @Override
  public void printIncorrectInputScript() {...}
}

public class LanMessagePrinter implements MessagePrinter {...} <-- ПЕЧАТАЕТ КУДА-ТО ПО СЕТИ
```
Но это совсем не обязательно. Так как есть, уже хорошо.

**18. class GameMapRenderer**

- Класс не должен принимать размеры карты в конструктор.  
Он уже принимает карту, поэтому может узнать размеры карты непосредственно от нее
```java
public class GameMapRenderer {
  public GameMapRenderer(int maxColumnValue, int maxRowValue) {
    this.maxColumnValue = maxColumnValue;
    this.maxRowValue = maxRowValue;
  }

  public void printGameMap(GameMap gameMap) {
    for (int col = 0; col < maxColumnValue; col++) {
      for (int row = 0; row < maxRowValue; row++) {
        //...
      }
    }
  }
}

//ПРАВИЛЬНО:
public class GameMapRenderer {
  public void printGameMap(GameMap gameMap) {
    for (int col = 0; column < gameMap.getWidth(); column++) {
      for (int row = 0; row < gameMap.getHeight(); row++) {
        //...
      }
    }
  }
}
```

Потому что сейчас это может стать причиной багов:
```java
public static void main(String[] args) {
  GameMap gameMap = new GameMap(5, 5);
  gameMap.putEntity(new Coordinates(0, 0), new Cheese());

  GameMapRenderer renderer = new GameMapRenderer(10, 10);
  renderer.printGameMap(gameMap);
}

//РЕЗУЛЬТАТ:
Exception in thread "main" java.lang.IllegalArgumentException: invalid coordinates received: Coordinates[col=0, row=5]
	at main.java.map.GameMap.isCellEmpty(GameMap.java:66)
```

👍 В целом, всё ок. Прикольно, что карта выглядит как шахматная доска.

**19. class ActionManager**

- Необходимость отдельного класса.

Честно говоря, не вижу необходимости в существовании этого класса.  
Наличие этого класса *для меня* выглядит как дробление класса Simulation.  
Все, что он делает, логичнее было бы видеть в Simulation.  

- Нарушение DRY. Общий код выноси во вспомогательный метод
```java
public void executeInitActions() {
  for (Action action : initActions) {
    action.perform();
  }
}

public void executeTurnActions() {
  for (Action action : turnActions) {
    action.perform();
  }
}

//ПРАВИЛЬНО:
public void executeInitActions() {
  executeActions(initActions);
}

public void executeTurnActions() {
  executeActions(turnActions);
}

private void executeActions(List<Action> actions) {
  for (Action action : actions) {
    action.perform();
  }
}
```
Может показаться, что такой рефакторинг не имеет смысла- количество строк кода не уменьшилось, а увеличилось на две строки.  
Но цель здесь в другом- не уменьшить количество строк кода, а избавиться от дублирования кода.

Главная проблема дублированного кода состоит в том, что при изменении одного участка с таким кодом, нужно не забыть точно так же изменить его в другом участке.  
На практике это часто не делается и поэтому программа перестает работать предсказуемо.  
Поэтому лучше не допускать дублирования.

**20. class SimulationInitializer**

Этот класс точно является примером антипаттерна "Дробление".  
Код из него нужно перенести в `Simulation`.

**21. class Simulation**

+ 👍 Принимает достаточное количество зависимостей в конструктор
```java
public Simulation(GameMap gameMap, GameMapRenderer gameMapRenderer, ActionManager actionManager)
```

- Нужно перенести сюда код из указанных мною ранее раздробленных классов.  
`TurnExecutor` тоже сюда.

- На мой взгляд, идея класса Simulation тут понята неправильно.

Сейчас Simulation в себе дергает за хвост какие-то потоки
```java
public class Simulation {
  TurnExecutor turnExecutor;
  SimulationThreadManager simulationThreadManager;  <-- Какие-то потоки

  public void startSimulation() {
    simulationThreadManager.start();  <-- дергает их за хвост
  }

  public void pauseSimulation() {
    simulationThreadManager.pause();
  }
  //...
}
```

**Но смысл в том, что все должно быть ровно наоборот.**

Класс `Simulation` не должен знать про потоки и дергать их за хвост.  
Он должен просто существовать и торчать наружу указанными в ТЗ публичными методами:
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
      Thread.sleep(THREAD_SLEEP);
    } catch (InterruptedException e) {...}
  }

  public void pauseSimulation() {
    //останавливает бесконечный цикл
  }
}
```

Когда мы добавляем в программу потоки, то поток должен принимать команды от юзера и дергать `Simulation` за ее методы:   
`nextTurn()`, `startSimulation()` и `pauseSimulation()`

**21. class SimulationLauncher**

Конструктор создает экземпляр класса(объект) и предназначен только для инициализации данных, а не для выполнения алгоритмов.  
Если в конструкторе запущен бесконечный цикл, значит конструктор не отработал до конца.  
Соответственно, объект не создан до конца, а находится в процессе создания
```java
public class SimulationLauncher {
  //... 
  public SimulationLauncher(Simulation simulation) {
    startGameLoop(simulation);
  }

  public void startGameLoop(Simulation simulation) {
    //бесконечный цикл
  }
} 

//ПРАВИЛЬНО:
public class SimulationLauncher {
  //... 
  public SimulationLauncher() {
    //пустой конструктор
  }

  public void startGameLoop(Simulation simulation) {
    //бесконечный цикл
  }
} 
```

**22. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

Всё шло неплохо, пока дело не дошло до класса Simulation и использования потоков.  
А так, норм.

n.144(305)  
#ревью #симуляция 