https://github.com/xoaltie/simulation  
[Daria M]

В целом пример хорошей реализации проекта "Симуляция", программист опытный.  
Но кое-где есть над чем поработать.

## ХОРОШО

+ 👍 Есть пауза/пуск во время работы
+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах (мне так больше нравится)
+ 👍 Точки расширения программы через использование интерфейсов: поиск, рендерер etc.

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Это валидация
```java
private void ensureWithinBounds(Coordinates coordinates) {
  if (!isWithinBounds(coordinates)) {
    throw new IllegalArgumentException("coordinates %s are out of bounds. map size: %dx%d".formatted(coordinates, width, height));
  }
}

//ПРАВИЛЬНО:
private void validateWithinBounds(Coordinates coordinates) {...}
```

Слово "ensure" встречается в названиях методов, которые не только проверяют условие, но при необходимости изменяют состояние объекта, чтобы условие выполнялось.  
Например `class ArrayList`
```java
// Increases the capacity of this ArrayList instance, if necessary, 
// to ensure that it can hold at least the number of elements specified by the minimum capacity argument.
public void ensureCapacity(int minCapacity) 
```

- Почему бы не согласовать названия
```java
public List<Coordinates> getEmptyCells() {
  List<Coordinates> emptyCell = new ArrayList<>();
  //...
  return emptyCell;
}

//ЛУЧШЕ:
public List<Coordinates> getEmptyCells() {
  List<Coordinates> emptyCells = new ArrayList<>();
  //...
  return emptyCells;
}
```

+ 👍 Хорошо, что аббревиатуры написаны по общим правилам, а не всеми большими буквами: "Bfs", а не "BFS"
```java
class BfsPathFinder
```

+ 👍 В целом нейминг ок.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение DRY**

Магические буквы, числа, слова. Вводи константы. 

- Совместная магия. 

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов.  
Либо перенеси ее в третий класс и из первых двух классов обращайся к этим константам- эти данные должны быть синхронизированы между собой
```java
public class ConsoleMenuRenderer implements MenuRenderer {
  public void printStartMenu() {
    System.out.println("Команды:\n1. Начать симуляцию\n2. Выход");
  }

  public void printPauseMenu() {
    System.out.println("Команды:\n1. Возобновить симуляцию\n2. Остановить симуляцию");
  }
}

public class Main {

  static void main() {
    //...
    case "1": //...
    case "2": //...
  }

  private static boolean handlePauseMenu(Simulation simulation) {
    //...
    case "1": //...
    case "2": //...
  }
}
```

Нужно иметь возможность в одном месте заменить "1", "2" на "S", "Q" и наоборот таким образом, чтобы эти команды одновременно изменились в рендерере и стали работать в Main. 

*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru, "Замена магического числа символьной константой"*

**3. record Coordinates**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```java
public record Coordinates(int x, int y) {
}
```

**4. class WorldMap**

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Вернуть все пустые координаты  
🔸 Сделать ход существом   

Наверное, для проекта в целом полезно иметь метод, который возвращает все пустые координаты.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно возвращать пустые координаты.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

- Поиск пустых координат.

Поиск в карте всех пустых координат можно было бы оправдать, если бы внутри карты это происходило с использованием более эффективного алгоритма.  
Типа, оптимизация- имея доступ к внутренней хешмапе, можно было бы быстрее формировать список этих координат.

Но нет, здесь используется обычный алгоритм: проход по всем координатам 
```java
public List<Coordinates> getEmptyCells() {
  List<Coordinates> emptyCell = new ArrayList<>();

  for (int y = 0; y < height; y++) {
    for (int x = 0; x < width; x++) {...}
  }

  return emptyCell;
}
```
То же самое с тем же успехом может делать клиент, так что хранить этот метод в самой карте точно не стоит.

- Метод совершения хода в карте- нарушение SRP.

Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой, минуя все правила игровой логистики
```java
public void moveEntity(Coordinates from, Coordinates to) {
  //Содержит часть общеигровой логики совершения хода:
  //координата from должна быть занята существом
  //координата to должна быть свободна
}
```
Должна ли карта учитывать правила логистики существ? Если да, то как?

Например, имеет ли право заяц телепортироваться на координату, которая уже занята?  
Может ли телепортироваться камень?  
Может ли телепортироваться пустое место на голову волка?

Метод частично отвечает на эти вопросы: он содержит часть правил выполнения хода существами.  
Тем самым единые правила выполнения хода существами разделяются на несколько классов: этот и `Creature`.   

Правила совершения хода существами - это дополнительная причина для изменения этого класса.  
Поэтому карта вообще не должна иметь в себе этого метода.

+ 👍 Геттеры не возвращают null, они возвращают Optional, это хорошо.

- Неподходящий тип исключения.

Карта находится в правильном состоянии. Просто в карте не хранится объект, по которому запрашивается координата 
```java
public Coordinates getCoordinates(Entity entity) {
  Objects.requireNonNull(entity, "entity must not be null");
  return entities.entrySet().stream()
    //...
    .orElseThrow(() -> new IllegalStateException("cannot find coordinates for entity %s".formatted(entity.getClass().getSimpleName())));
}
```

+ 👍 Хорошо, что возвращается копия коллекции, а не оригинал
```java
public Collection<Entity> getEntities() {
  return List.copyOf(entities.values());
}
```
Здесь нужно возвращать копию, потому что `entities.values()` возвращает коллекцию, которая является внутренним устройством хэшмапы `entities`.  
Тем самым соблюдается инкапсуляция и защищается внутреннее устройство класса.  

+ 👍 В целом с точки зрения SRP класс хороший. Имеющиеся нарушения SRP в нем - незначительные.

**5. interface PathFinder**

+ 👍 Интерфейс поиска пути- это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
public interface PathFinder {
  List<Coordinates> find(WorldMap map, Coordinates start, Class<? extends Entity> target);
}
```

**6. class BfsPathFinder implements PathFinder**

- Пары чисел x,y - это координаты. В данном случае это координаты смещения относительно точки 0.0
```java
private static final int[][] DIRECTIONS = {{0, -1}, {1, 0}, {0, 1}, {-1, 0}};

private List<Coordinates> getNeighbors(WorldMap map, Coordinates start) {
  //...
  for (int[] direction : DIRECTIONS) {
    Coordinates neighbor = new Coordinates(start.x() + direction[0], start.y() + direction[1]);
    //...
  }
}

//ЛУЧШЕ:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(new Coordinates(0, -1), ...);

private List<Coordinates> getNeighbors(WorldMap map, Coordinates start) {
  //...
  for (Coordinates shift : SHIFT_COORDINATES) {
    Coordinates neighbor = new Coordinates(start.x() + shift.x(), start.y() + shift.y());
    //...
  }
}
```

+ 👍 В случае ненахода возвращает пустой путь, это хорошо
```java
public List<Coordinates> find(WorldMap map, Coordinates start, Class<? extends Entity> target) {
   //...
  return List.of();
}
```

+ 👍 Класс хороший, важных замечаний к нему нет.

**7. abstract sealed class Entity permits Creature, Grass, Rock, Tree**

* 👻 Не вижу причин ограничивать наследование от класса `Entity`
```java
public abstract sealed class Entity permits Creature, Grass, Rock, Tree {
}
```

Смысл ООП ведь в том, чтобы расширять функционал программы через наследование классов.  
При добавлении новых существ, например Крокодила, придется дописывать этих существ в сигнатуру класса `Entity`.  
А значит этот класс открыт для изменений- нарушение OCP. 

Но учитывая, что про sealed-классы я узнал в сегодня лет, то возможно ты используешь какой-то прогрессивный архитектурный подход, про который я просто не в курсе.

**8. class Grass/Tree/Rock extends Entity**

+ 👍 Простые неходячие наследники без состояния и поведения - идеально
```java
public final class Grass extends Entity {
}
```

- Ограничение наследования классов через final.

Опять же, не вижу причин ограничивать наследование от простых неходячих существ.  
Почему бы в будущем не унаследовать класс Фруктовое Дерево от обычного Дерева?  
Возможно, есть какие-то причины для ограничения *здесь* наследования, но я не вижу ни одной. 

**9. abstract sealed class Creature extends Entity permits Herbivore, Predator**

+ 👍 Хорошо, что Креатура принимает интерфейс поиска в конструктор
```java
public Creature(... , PathFinder pathFinder) {...}
```

Тем самым, появляется точка расширения: разным существам можно давать разные алгоритмы поиска.  

- Сложные правила валидации в конструкторе. Возможно, стоит их вынести во вспомогательный метод.

+ 👍 Класс хороший, важных замечаний к нему нет.

**10. final class Herbivore extends Creature**

Вот здесь ломается всё преимущество от использования интерфейса поиска в проекте:
```java
public final class Herbivore extends Creature {
  public Herbivore(...) {
    super(..., new BfsPathFinder());
  }
  //...
}

//ПРАВИЛЬНО:
public final class Herbivore extends Creature {
  public Herbivore(..., PathFinder pathFinder) {
    super(..., pathFinder);
  }
  //...
}
```

Существо не должно самостоятельно выбирать тип своего поиска пути.  
Иначе нельзя будет использовать разные игровые конфигурации, где существа будут использовать разные типы поиска.  

Эти зависимости должны собираться где-то наверху, например в main'e, а оттуда инжектиться вниз. 

Например, должна быть возможность создать два main-класса.  
По запуску одного, должна запускаться симуляция, в которой существа используют поиск BFS.  
По запуску другого- симуляция с поиском A*.

Сейчас, чтобы поменять в проекте тип поиска, нужно переписывать код в классах `Herbivore` и `Predator`.  
А значит, эти классы открыты для изменений - это нарушение OCP.

**11. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(WorldMap map);
}
```

**12. MoveCreaturesAction implements Action**

👍 Идеально.

**13. final class EntitySpawner**

Хороший универсальный класс-спавнер.  
Его универсальность обеспечивается использованием стандартного функционального интерфейса Java: через этот аргумент передается способ создания любого нужного существа
```java
public final class EntitySpawner {
  public void spawn(WorldMap map, int countToSpawn, Supplier<? extends Entity> supplier) {...}
}
```

**14. class MapPopulationAction/ReplenishEntitiesAction implements Action**

- Высокая связанность классов: нарушение Low Coupling.

Класс не должен напрямую лезть в константы конфига.  
Он должен получать необходимые данные из конфига в свой конструктор.  
Или должен получать в свой конструктор экземпляр конфига в качестве объекта-аргумента.

Иначе классы зависят от конкретного конфига и нельзя делать разные игровые конфигурации симуляции.  
Например, нельзя сделать дополнительный тестовый main-класс, который запустит игру с одним зайчиком и одной морковкой. 

Условный пример: 
```java
public class Settings {
  public static final int ROOMS = 5;
}

//ПЛОХО:
public static void main(String[] args) {
  House house = new House();
  //oth code
}

public class House {
  private final int rooms;

  public House() {
    this.rooms = Settings.ROOMS;
  }

  public int getRooms() {
    return rooms;
  }
}

//ХОРОШО:
public static void main(String[] args) {
  House house = new House(Settings.ROOMS);
  //oth code
}

public class House {
  private final int rooms;

  public House(int rooms) {
    this.rooms = rooms;
  }

  public int getRooms() {
    return rooms;
  }
}
```

+ 👍 В остальном в этих экшенах всё ок. 

**15. final class SimulationConfig**

+ 👍 Это константный класс и он соответствует требованиям для утилитных и константных классов
```java
public final class SimulationConfig {  <-- final
  //ТУТ КОНСТАНТЫ

  private SimulationConfig() {  <-- закрытый конструктор
  }
}
```

Невозможно унаследоваться от этого класса или сделать его экземпляр, это хорошо.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

**16. interface MapRenderer**

+ 👍 Интерфейс рендерера это хорошо
```java
public interface MapRenderer {
  void render(...);
}
```

Теперь можно делать разные рендереры для разных визуальных сред (консоль, интерфейс виндовс, http, матричный принтер etc)  
и разного отображения информации (цветной, черно-белый и пр.)

- Нарушение SRP.

Рендерер карты должен печатать только карту.  
Печатать номер хода должен кто-то другой
```java
void render(WorldMap map, int turnCounter);
```

Первая, основная причина для изменения класса: то, как будет печататься карта.  
Например, буквами или эмодзи.

Вторая, лишняя причина для изменения класса - то, как будет распечатываться сообщение о номере хода.  
Например, на каком языке это будет происходить.

**17. final class ConsoleMapRenderer implements MapRenderer**

- Нарушение SRP.  

Рендерер карты должен только рисовать карту.  
Сейчас кроме рисования карты, рендерер печатает номер хода и игровые правила:
```java
sb.append("Текущий ход: ").append(turnCounter).append(System.lineSeparator());
sb.append("Для остановки игры на паузу нажмите Enter").append(System.lineSeparator());
```

Почему рендерер КАРТЫ вообще знает, что его используют в контексте совершения ходов, а не просто для распечатки карты?  
Почему рендерер карты знает правила игровой логики - то, что для остановки игры нужно нажать Enter?  

- Нарушение принципа GRASP Creator.

Создавать объект должен тот, кто его использует
```java
public final class ConsoleMapRenderer implements MapRenderer {
  private final SpriteMapper spriteMapper;

  public ConsoleMapRenderer(SpriteMapper spriteMapper) {
    this.spriteMapper = spriteMapper;
  }
  //...
}

//СЕЙЧАС было бы правильно так:
public final class ConsoleMapRenderer implements MapRenderer {
  private final SpriteMapper spriteMapper;

  public ConsoleMapRenderer() {
    this.spriteMapper = new SpriteMapper();
  }
  //...
}
```

SpriteMapper - это не интерфейс и не абстрактный класс. Этот класс при создании не конфигурируется.  
И чтоб совсем наверняка, он final:
```java
public final class SpriteMapper {
  public String getSprite(Entity entity) {
    // Возвращает эмодзи спрайт существа
  }

  public String getEmptyCellSprite() {
    // Возвращает эмодзи спрайт земли
  }
}
```

Поэтому согласно Creator, объект этого класса должен создавать сам `ConsoleMapRenderer`.  
Просто потому, что у него есть вся нужная инфа для его создания маппера самого себя.

Но, конечно, можно повысить накал ООП и для этого вот так принимать мапер спрайтов в рендерер карты:
```java
//КЛИЕНТ МАППЕРА:
public final class ConsoleMapRenderer implements MapRenderer {
  private final SpriteMapper spriteMapper;

  public ConsoleMapRenderer(SpriteMapper spriteMapper) {...}
  //...
}

//МАППЕР СПРАЙТОВ:
public interface SpriteMapper {
  String getSprite(Entity entity);

  String getEmptyCellSprite();
}

//РАЗНЫЕ РЕАЛИЗАЦИИ МАППЕРА: 
public final class EmojiSpriteMapper implements SpriteMapper {  //Эмодзи маппер
  @Override 
  public String getSprite(Entity entity) {
    // Возвращает эмодзи спрайт существа
  }

  @Override 
  public String getEmptyCellSprite() {
    // Возвращает эмодзи спрайт земли
  }
}

public final class TextSpriteMapper implements SpriteMapper {  //Текстовый маппер
  private static final String TEMPLATE = " %c ";
  private static final String EMPTY = TEMPLATE.formatted('.');

  @Override 
  public String getSprite(Entity entity) {
    char letter = entity.getClass().getSimpleName().charAt(0);  //Начальная буква названия класса всегда большая

    if (!(entity instanceof Creature)) {
      letter = Character.toLowerCase();  //неходячие энтити- маленькими буквами, остальные- большими
    }

    return TEMPLATE.formatted(letter);  
  }

  @Override 
  public String getEmptyCellSprite() {
    return EMPTY;
  }
}
```

+ 👍 Правильное использование альтернативных названий индексов циклов вместо стандартных i,j 
```java
for (int y = 0; y < map.getHeight(); y++) {
  for (int x = 0; x < map.getWidth(); x++) {
    Coordinates coordinates = new Coordinates(x, y);
    //...
  }
}
```

**18. Интерфейс рендерера меню и его русская реализация**

+ 👍 В целом ок. Нужно согласовать названия команд ("1", "2") с их использованием в классе Main. 

**19. class Simulation**

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий.  
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе `Simulation`
```java
public Simulation(WorldMap map, MapRenderer renderer, List<Action> initActions, List<Action> turnActions) {...}
```

- Неправильное использование потоков в проекте.

Внутри класса `Simulation` не должно быть никакой работы с потоками
```java
public final class Simulation {
  //...

  public void start() {
    //...
    Thread thread = new Thread(() -> {...}  //Какой-то поток
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
Но и запустить без потоков просто в бесконечном цикле и он должен бесконечно работать:
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

Один простой Main должен просто запустить симуляцию в бесконечном цикле без возможности управлять её работой через команды пауза/пуск.  
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

По запуску второго симуляция должна работать так, как сейчас, - с возможностью делать паузу/пуск.  
Примерно так:
```java
public class MainWithThreads {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);

    SimulationManager manager = new SimulationManager(simulation);  //работает с потоками, дергает симуляцию за методы пауза/пуск 

    manager.start(); //симуляция с потоками и командами пауза/пуск
  }
}
```

**20. class Main**

- Нарушение SRP.

Main должен только сконфигурировать зависимости и запустить программу.  
Управлять работой программы этот класс не должен.  

В main'e может быть только начальный диалог "Играть или выйти"- это всё ещё ответственность создания и запуска системы.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

Определи сущности, которые сейчас сидят в этом классе и вынеси их в отдельные классы.

Например, сущность, которая управляет симуляцией, получая от юзера команды пауза/пуск.  
Создание экземпляра симуляции тоже выглядит сложно- эту работу при желании можно вынести в Фабрику. 

## ВЫВОД

Стремление всегда запрещать наследование классов в этом проекте мне непонятно.  
Также неясно, почему в одних случаях наследование запрещается, а в других нет:  
`final class Predator`: наследование запрещено.  
`class MapPopulationAction`: наследование разрешено.  

В целом норм 👍  
Виден опыт программирования, хорошее алгоритмическое мышление и общее понимание принципов ООП. 
 
n.176(379)  
#ревью #симуляция 