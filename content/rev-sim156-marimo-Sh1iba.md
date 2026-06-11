https://github.com/Sh1iba/Simulation  
[Маримо]

В целом норм, но стоило бы переделать работу с потоками.

## ХОРОШО

+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализована пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1.1. Нейминг**

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относится.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
private final HashMap<Coordinates, Entity> map = new HashMap<>();
Map<Coordinates, Coordinates> roadMap = new HashMap<>();


//ПРАВИЛЬНО:
private final HashMap<Coordinates, Entity> entities = new HashMap<>();  //или coordinatesWithEntities
Map<Coordinates, Coordinates> road = new HashMap<>();
```

- Названия классов не должны оканчиваться на "able".

На "able" могут заканчиваться только названия интерфейсов, эта частица означает "способный(делать что-то)".  
Слова на "able" являются прилагательными(бегающий, итерирующий), а названия классов должны быть существительными.  
В крайнем случае просто называй этот класс как интерфейс, добавив к названию "Imp"
```java
class InputRunnable implements Runnable 

//ПРАВИЛЬНО НАПРИМЕР ТАК:
class InputHandler implements Runnable 
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**1.2. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки.  
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```java
if (entity instanceof Predator) return false;
if (pathToTarget.size() < 2) return;
if (running) this.notifyAll();

//ПРАВИЛЬНО:
if (entity instanceof Predator) {
  return false; 
}

if (pathToTarget.size() < 2) {
  return;
}

if (running) {
  this.notifyAll();
}
```

- Конструктор должен стоять первым среди методов
```java
public abstract class PlaceEntityAction implements Action {

  private final Random random = new Random();
  protected abstract Entity getEntity();
  protected abstract double getDensity();

  @Override
  public void perform(GameMap map) {...}
}

//ПРАВИЛЬНО:
public abstract class PlaceEntityAction implements Action {

  private final Random random = new Random();

  @Override
  public void perform(GameMap map) {...}

  protected abstract Entity getEntity();
  protected abstract double getDensity();
}
```

Это не формальность, это действительно важно- не обнаружив конструктор на привычном месте, 
можно неправильно понять как работает класс.

- Второстепенные вспомогательные методы должны стоять ниже главных публичных
```java
public abstract class MoveEntityAction implements Action {

  protected abstract Class<? extends Creature> getEntityClass();

  @Override
  public void perform(GameMap map) {...}
}

//ПРАВИЛЬНО:
public abstract class MoveEntityAction implements Action {

  @Override
  public void perform(GameMap map) {...}

  protected abstract Class<? extends Creature> getEntityClass();
}
```

*"Oracle Java code conventions"*  

**2. Используй классы через их интерфейсы**
```java
private final HashMap<Coordinates, Entity> map = new HashMap<>();

//ПРАВИЛЬНО:
private final Map<Coordinates, Entity> map = new HashMap<>();
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
if (map.getEntity(next) == null) {
  //...
} else {
  return;
}

//ПРАВИЛЬНО:
if (map.getEntity(next) == null) {
  //...
}
return;
```

**4. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```java
 private static final String SIMULATION_MENU_BANNER = """
      ╔══════════════════════════════════════════════════════╗
      ║  Введите 'F' или 'f' для остановки/запуска симуляции  ║
      ╚══════════════════════════════════════════════════════╝
      """;
 
if (str.equals("F") || str.equals("f")) {...}

//ПРАВИЛЬНО:
private static final String STOP = "F";

private static final String SIMULATION_MENU_BANNER = """
      ╔══════════════════════════════════════════════════════╗
      ║  Введите '%s' или '%s' для остановки/запуска симуляции  ║
      ╚══════════════════════════════════════════════════════╝
      """.formatted(STOP, STOP.toLowerCase());

if (str.equalsIgnoreCase(STOP)) {...}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**5. class EntityConfig**

- Соблюдай требования утилитных/константных классов. 

Это утилитный класс, он содержит только статические методы.  
Утилитные классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- По сути, это просто константный класс.

Инкапсуляция и публичные методы никакой полезной нагрузки тут не несут
```java
public class EntityConfig {

  private static final double GRASS_DENSITY = 0.07;
  private static final double HERBIVORES_DENSITY = 0.05;

  public static double getGrassDensity() {
    return GRASS_DENSITY;
  }

  public static double getHerbivoresDensity() {
    return HERBIVORES_DENSITY;
  }
}

//ЛУЧШЕ:
public final class EntityConstants {
  //закрытый конструктор

  public static final double GRASS_DENSITY = 0.07;
  public static final double HERBIVORES_DENSITY = 0.05;
}
```

Содержать методы в конфиге имеет смысл тогда, когда конфиги могут быть разными, но должны использоваться через полиморфизм.  
Например:
```java
public interface EntityConfig {
  double grassDensity();
  double herbivoresDensity();
}

public class EntityConfigEasy implements EntityConfig {
  private static final double GRASS_DENSITY = 0.07;
  private static final double HERBIVORES_DENSITY = 0.05;

  public static double grassDensity() {
    return GRASS_DENSITY;
  }

  public static double herbivoresDensity() {
    return HERBIVORES_DENSITY;
  }
}

public class EntityConfigHard implements EntityConfig {
  private static final double GRASS_DENSITY = 0.12;
  private static final double HERBIVORES_DENSITY = 0.15;

  public static double grassDensity() {
    return GRASS_DENSITY;
  }

  public static double herbivoresDensity() {
    return HERBIVORES_DENSITY;
  }
}
```

**5. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо.

- Класс может быть преобразован в record без потери функционала.

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**6.  class GameMap**

- Никогда не возвращай null
```java
private final HashMap<Coordinates, Entity> map = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return map.get(coordinates);  <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Проверяй, находится ли координата в пределах карты. 

При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что она свободна.  
А правильный ответ- ячейки с такой координатой в карте нет вообще
```java
public boolean isSquareEmpty(Coordinates coordinates) {
  return !map.containsKey(coordinates);
}

//ПРАВИЛЬНО:
public boolean isSquareEmpty(Coordinates coordinates) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение
  return !map.containsKey(coordinates);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

Сейчас в карту можно вставить существо на координату, выходящую за размер карты
```java
Coordinates coordinates = new Coordinates(+100500, -100500);
Карта карта = new Карта(10, 10);
карта.setEntity(coordinates, new Заяц());
```

+ 👍 С точки зрения соблюдения SOLID, я бы назвал этот класс почти идеальным.

Тут не хватает только метода `Coordinates getCoordinates(Entity entity)`.

**7. interface Search**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
public interface Search {
  List<Coordinates> findPath(GameMap map, Coordinates start, Class<? extends Entity> target);
}
```

**8. class BreadthFirstSearch implements Search**

- Нарушение SRP, OCP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```java
if (entity instanceof Rock) return false;
if (entity instanceof Tree) return false;
```

Нарушение OCP здесь состоит в том, что при добавлении в проект нового существа, например, Птицы, 
придется изменить код класса. То есть, класс открыт для изменений:
```java
if (entity instanceof Rock) return false;
if (entity instanceof Tree) return false;

if (entity instanceof Bird) return false;
```

- Повторяющиеся действия делай через цикл
```java
if (isValidCoordinate(map, new Coordinates(x, y + 1))) {
  neighboringCoordinates.add(new Coordinates(x, y + 1));
}
if (isValidCoordinate(map, new Coordinates(x, y - 1))) {
  neighboringCoordinates.add(new Coordinates(x, y - 1));
}

//ПРАВИЛЬНО:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(new Coordinates(1, 0), ...);

for(Coordinates shift : SHIFT_COORDINATES) {
  Coordinates current = new Coordinates(shift.getX() + x, shift.getY() + y);
  neighboringCoordinates.add(current);
}
```

**9. abstract class Entity**

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```java
public abstract class Entity {
  public abstract String getSymbol();
}
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**10. class Creature extends Entity**

- Избыточно. Числовые поля при создании и так инициализируются нулями
```java
public abstract class Creature extends Entity {
  protected int speed = 0;
  protected int healthPoint = 0;
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  protected int speed;
  protected int healthPoint;
  //...
}
```

- Неправильное использование интерфейса.  

В программе есть интерфейс `Search`. Зачем он нужен?

Я полагаю, чтобы можно было использовать разные его имплементации без переписывания кода в других классах.  
Но сейчас так не получится- для использования другой имплементации поиска, нужно будет изменять код класса Creature.  
То есть, класс открыт для изменений
```java
public abstract class Creature extends Entity {
  private final Search search = new BreadthFirstSearch();
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  private final Search search;
  //...
  public Creature(Search search) {...}
}
```

- Креатура должна сама искать свои текущие координаты
```java
public void makeMove(GameMap map, Coordinates coordinates) {
  Coordinates current = coordinates;
  //...
}

//ПРАВИЛЬНО:
public void makeMove(GameMap gameMap) {
  Coordinates current = gameMap.getCoordinates(this);
  //...
}
```
Креатура все равно в метод хода принимает `GameMap`, поэтому может спросить у карты свое текущее местоположение.  

Если креатура будет принимать свое текущее местоположение извне, а не получать самостоятельно из карты, то это потенциально может привести к багам.  
Потому что креатура не может гарантировать, что пришедшая извне координата *действительно* соответствует ее текущему местоположению.  
Слепая вера клиенту тут вредна.

**11. class Herbivore/Predator extends Creature**

- Нарушение дизайна ООП в наследовании. 

При создании экземпляра класса, значения обязательных полей класса должны устанавливаться через конструктор, а не сетиться
```java
public class Herbivore extends Creature {
  //...

  public Herbivore() {
    this.healthPoint = MAX_HEALTH_POINT;
    this.speed = 1;
  }
}

//ПРАВИЛЬНО:
public class Herbivore extends Creature {
  //...

  public Herbivore() {
    super(MAX_HEALTH_POINT, SPEED);
  }
}
```

тут тоже
```java
public class Predator extends Creature {
  //...    
  private final int attackPower;

  public Predator() {
    this.healthPoint = 100;
    this.speed = 2;
    this.attackPower = 25;
  }
}

//ПРАВИЛЬНО:
public class Predator extends Creature {
  //...    
  private final int attackPower;

  public Predator() {
    super(MAX_HEALTH_POINT, SPEED);
    this.attackPower = 25;
  }
}
```
*Эккель "Философия Java", гл.5, "Конструктор гарантирует инициализацию"* 

+ 👍 В целом все семейство entities сделано хорошо.

**12. interface Action**

👍 Идеально
```java
public abstract class Action {
  public abstract void perform(WorldMap worldMap);
}
```

**13. Init Action's**

Семейство классов в пакете `actions.init`.

**14. abstract class PlaceEntityAction implements Action**

- Чтобы не делать наследников такими сложными, как они сейчас есть или вообще их не делать(потому что не очень-то и надо),
способ создания существ принимай в виде стандартного интерфейса `Supplier`
```java
public abstract class PlaceEntityAction implements Action {

  @Override
  public void perform(GameMap map) {
    //...
    map.setEntity(coordinates, getEntity());
  }

  protected abstract Entity getEntity();
  //...
}

//ЛУЧШЕ:
public class PlaceEntityAction implements Action {

  private final Supplier<Entity> entitySupplier;
  private final int count;

  public PlaceEntityAction(Supplier<Entity> entitySupplier, int count) {...}

  @Override
  public void perform(Карта карта) {
    for (int i = 0; i < count; i++) {
      Entity entity = entitySupplier.get();
      Coordinates coordinates = getRandomFreeCoordinates(карта);
      карта.put(entity, coordinates);
    }
  }
  //...
}
```

И тогда этот класс будет удобно использовать даже сам по себе, без наследников:
```java
SpawnAction заяцSpawnAction = new SpawnAction(() -> new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ);
SpawnAction волкSpawnAction = new SpawnAction(() -> new Волк(), КОЛИЧЕСТВО_ВОЛКОВ);
заяцSpawnAction.perform(карта); 
волкSpawnAction.perform(карта); 
```

Но если для удобства захочется создать специальные наследники-спавнеры, то эти наследники будут выглядеть значительно проще
```java
//БЫЛО:
public class PlacePredatorsAction extends PlaceEntityAction {

  @Override
  protected Entity getEntity() {
    return new Predator();
  }

  @Override
  protected double getDensity() {
    return EntityConfig.getPredatorsDensity();
  }
}

//СТАНЕТ:
public class PlacePredatorsAction extends PlaceEntityAction {
 private static final Supplier<Entity> ENTITY_SUPPLIER =  () -> new Predator();  

  public PlacePredatorsAction(int count) {
    super(ENTITY_SUPPLIER, count);
  }
}
```

**16. Наследники PlaceEntityAction**

Есть много наследников PlaceEntityAction:
```java
class PlaceHerbivoresAction extends PlaceEntityAction
class PlacePredatorsAction extends PlaceEntityAction
etc
```

Все они неправильно используют конфигурационные классы.  
Они напрямую лезут в константный класс, а должны *только* получать необходимые данные в конструктор
```java
public class PlacePredatorsAction extends PlaceEntityAction {

  @Override
  protected Entity getEntity() {
    return new Predator();
  }

  @Override
  protected double getDensity() {
    return EntityConfig.getPredatorsDensity();
  }
}

//ПРАВИЛЬНО:
public class PlacePredatorsAction extends PlaceEntityAction {

  public PlacePredatorsAction(double density) {
    super(density);
  }

  @Override
  protected Entity getEntity() {
    return new Predator();
  }
}
```
Это делает классы неуниверсальными и зависимыми от константного класса.  

Условные примеры:
```java
public class Settings {
  public static final int ROOM_NUMBERS = 5;
}

//ПЛОХО:
public static void main(String[] args) {
  House house = new House();
  //oth code
}

public class House {
  private final int roomNumbers;

  public House() {
    this.roomNumbers = Settings.ROOM_NUMBERS;
  }

  public int getRoomNumbers() {
    return roomNumbers;
  }
}

//ХОРОШО:
public static void main(String[] args) {
  House house = new House(Settings.ROOM_NUMBERS);
  //oth code
}

public class House {
  private final int roomNumbers;

  public House(int roomNumbers) {
    this.roomNumbers = roomNumbers;
  }

  public int getRoomNumbers() {
    return roomNumbers;
  }
}
```
Про классы констант, конфигурации и их использование я писал [ТУТ](https://t.me/zhukovsd_it_chat/53243/176984)

**17. Turn Action's**

Семейство классов в пакете `actions.turn`.

**18. Action's для добавления существ**

Это такие классы:
```java
abstract class AddEntityAction implements Action 
class AddHerbivoresAction extends AddEntityAction
class AddGrassAction extends AddEntityAction 
etc
```

Как и в случае с init-экшенами, здесь есть группа классов для добавления существ в карту.  
Эти классы используются для поддержания поголовья существ в карте на протяжении игры. 

Ситуация с ними та же самая, что и с init-экшенами:  
Можно или сделать один универсальный экшен.  
Или упростить код родительского класса так, чтобы его наследники были предельно простые.

**19. Action's для передвижения существ**

Это такие классы:
```java
abstract class MoveEntityAction implements Action
class MoveHerbivoresAction extends MoveEntityAction
class MovePredatorsAction extends MoveEntityAction 
```

Та же самая история- не вижу смысла в куче классов, когда можно сделать один универсальный мувер.  
Представим, что проект разросся с 2 видов ходящих существ до 20-ти.  
То что, каждому существу свой мувер делать?

То есть потенциал для масштабирования у такой архитектуры весьма небольшой.

Ну ок, я смысл понимаю- разные муверы для разных существ сделаны для того, чтобы можно было сначала походить зайцами, а потом волками.  
Но я не думаю, что параметризация одного универсального мувера это настолько неудобнее, что нужно вводить в проект вместо этого кучу других классов
```java
//СЕЙЧАС - ТАК, ПЕРСОНАЛЬНЫЕ МУВЕРЫ:
turnActions.add(new MoveHerbivoresAction());
turnActions.add(new MovePredatorsAction());

//ТО ЖЕ САМОЕ ЧЕРЕЗ ПАРАМЕТРИЗАЦИЮ УНИВЕРСАЛЬНОГО КЛАССА:
turnActions.add(new MoveAction(Herbivore.class));
turnActions.add(new MoveAction(Predator.class));
```

**20. class MapConsoleRenderer**

- Спрайты существ должны храниться здесь, а не браться из самих существ.

- Инкапсуляция.

Нет смысла в том, чтобы эта константа была публичной
```java
public static final String EMPTY_CELL_CONTENT = "⬛";
```

Если нет веской причины для обратного, то поля и методы в классах должны быть `private`.

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < map.getHeight(); i++) {
  for (int j = 0; j < map.getWidth(); j++) {
    Entity entity = map.getEntity(new Coordinates(j, i));
  }
}

//ЛУЧШЕ:
for (int y = 0; y < map.getHeight(); y++) {
  for (int x = 0; x < map.getWidth(); x++) {
    Entity entity = map.getEntity(new Coordinates(x, y));
  }
}
```

**21. class Simulation**

- Нарушение DI. Класс не должен сам себя конструировать.  
В данном случае класс конструирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```java
public class Simulation {
  private final GameMap map = new GameMap(50, 20);
  //...

  public Simulation() {...}
  //...
}

//ПРАВИЛЬНО:
public class Simulation {
  private final GameMap gameMap;
  //...
  
  public Simulation(GameMap gameMap) {
    this.gameMap = gameMap;
  }
  //...
}
```
Таким образом нельзя без изменения кода в классе `Simulation` создать несколько игровых конфигураций с разными картами.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Неправильное использование потоков в проекте.

Внутри класса Simulation не должно быть никакой работы с потоками
```java
public class Simulation {
  //...

  public void startSimulation() {
    while (true) {
      synchronized (this) {
        while (!running) {
          try {
            this.wait();
          } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return;
          }
        }
      }
      nextTurn();
    }
  }

  public void launch() {
    var runnable = new InputRunnable(this);
    var thread = new Thread(runnable);
    //var thread = new Thread(this::pauseSimulation);
    thread.setDaemon(true);
    thread.start();
  }
  //...
}
```

Этот класс не должен управлять потоками.  
Наоборот, какие-то потоки должны управлять этим классом и дергать за его публичные методы, которые указаны в ТЗ:
```java
Simulation #
Главный класс приложения, включает в себя:

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
По запуску одного симуляция должна работать так, как сейчас- с потоками и возможностью делать паузу/пуск.  

А другой Main без потоков, просто запускает Симуляцию, которая будет работать вечно.  
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
```

- Избыточно
```java
if (str.equals("F") || str.equals("f")) {...}

//ПРАВИЛЬНО:
if (str.equalsIgnoreCase("F")) {...}
```

## ВЫВОД

Работу с потоками следует переделать- класс Simulation не должен создавать поток, который работает с командами юзера.  
В целом, норм. Каких-то грубых ошибок архитектуры не заметил 👍
 
n.156(331)  
#ревью #симуляция 