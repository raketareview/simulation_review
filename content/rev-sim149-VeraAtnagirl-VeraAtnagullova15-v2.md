https://github.com/VeraAtnagullova15/Simulation2  
[Вера Атнагуллова]

Это вторая версия программы. 
[Ревью на первую версию тут.](https://github.com/raketareview/simulation_review/blob/master/content/rev-sim143-VeraAtnagirl-VeraAtnagullova15.md)

После рефакторинга стало значительно лучше.  

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализована пауза/пуск во время работы
+ 👍 Цветовая индикация здоровья существ на карте

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Пакеты нужно назывть стилем alllowercase.  
Не пиши в названиях слово "объект"- в java все классы относятся к типу Object
```java
package simulation.entities.map_objects;

//ПРАВИЛЬНО:
package simulation.entities.basic;  <-- например так
```

- UPPER_SNAKE только для констант, а константы это static final поля.  
Это не не константы, а просто финальные поля 
```java
private final int ROW_COUNT;
private final int COLUMN_COUNT;

//ПРАВИЛЬНО:
private final int rowCount;
private final int columnCount;
```

- Возврат констант это не повод нарушать конвенцию наименований.  
Названия полей должны писаться стилем camelCase
```java
public int getROW_COUNT() {
  return ROW_COUNT;
}

//ПРАВИЛЬНО:
public int getRowCount() {
  return ROW_COUNT;
}
```

- Название ментода не соответствует тому, что он делает.  
Судя по сигнатуре, метод может вернуть не только  список креатур, но и любых других объектов, класс которых задан дженериком.  
Например, список камней или травы
```java
public <T extends Entity> List<T> getAllCreatures(Class<T> clazz)
```

- Название метода не соответствует тому, что он делает
```java
public boolean isAttacked() {
  return hp <= 50;
}
```
Метод не определяет того, что существо было атаковано.  
Метод определяет, что жизни у существа меньше какого-то магического числа.

- Название класса должно отражать суть этого класса.  

"Реальная фабрика" скорее озодачивает- почему она реальная, есть еще нереальная?  
Если под "реальной" понимается, что это класс, а не интерфейс, то просто добавь "Imp"
```java
class RealEntityFactory implements EntityFactory

//ПРАВИЛЬНО:
class EntityFactoryImp implements EntityFactory
```

- Либо "Printer" и "print", либо "Renderer" и "render"
```java
class RendererWorldMap {
  public void print(WorldMap worldMap) {...}
}

//ПРАВИЛЬНО:
class RendererWorldMap {
  public void render(WorldMap worldMap) {...}
}
```

- Избыточно. Мы и так понимаем, что метод поиска в классе поиска пути ищет именно путь, а не что-то иное
```java
public interface PathFinder {
  public List<Coordinates> findPath(...)
}

//ПРАВИЛЬНО:
public interface PathFinder {
  public List<Coordinates> find(...)
}
```

- "PathFinder" это интерфейс поиска, а "BFS" это один из алгоритмов поиска.  
У PathFinder могут быть и другие реализации поиска пути: AStar и БегатьХаотичноКакПьяныйЛось.  
Поэтому нельзя общее назвать как его частное
```java
PathFinder bfs;

//ПРАВИЛЬНО:
PathFinder pathfinder;
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется.**
 
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без `else` будет выглядеть читабельней
```java
if (entity instanceof Herbivore) {
  return HERBIVORE;
} else if (entity instanceof Predator) {
  return PREDATOR;
}

//ПРАВИЛЬНО:
if (entity instanceof Herbivore) {
  return HERBIVORE;
} 
if (entity instanceof Predator) {
  return PREDATOR;
}
```

**3. Вводи поясняющие переменные**

Такие длинные конструкции вообще нечитабельны.  
Здесь 5 инструкций, собранных в одну. Или 9.  
Вводи поясняющие переменные
```java
line.append(background).append(String.format("%-3s", getSymbolEntities(entity))).append(ANSI_RESET);
```
*Фаулер, "Рефакторинг", гл.6, "Введение поясняющей переменной".*

**4. Нарушение DRY**, магические буквы, числа, слова. Вводи константы. А если они уже есть- пользуйся 
```java
private final static char START = 'S';
private final static char STEP = 'T';
//...
private final static List<Character> COMMANDS = List.of(START, STEP, PAUSE, RESTART, QUIT);

String title = """
        Введите T - сделать один шаг, S - запустить симуляцию, P - пауза
            R - рестарт симуляции после паузы, Q - завершение программы
        """;

//...
case 'T' -> {...}
case 'S' -> {...}

//ПРАВИЛЬНО:
case START -> {...}
case STEP -> {...}

String title = """
        Введите %c - сделать один шаг, %c - запустить симуляцию, %c - пауза
            %c - рестарт симуляции после паузы, %c - завершение программы
        """.formatted(STEP, START, ...);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**5. record Coordinates(int row, int column)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**6. class WorldMap**

- Нельзя создать карту произвольных размеров.

Карта содержит в себе фиксированные размеры, поэтому становится неуниверсальной- нельзя создать игровые конфигурации с картами разных размеров.  
Если хочется иметь возможность создавать карту с размерами по умолчанию, сделай перегруженный конструктор
```java
private final int ROW_COUNT;
private final int COLUMN_COUNT;

public WorldMap() {
    ROW_COUNT = 12;
    COLUMN_COUNT = 12;
}

//ПРАВИЛЬНО:
private static final int DEFAULT_ROW_COUNT = 12;
private static final int DEFAULT_COLUMN_COUNT = 12;

private final int rowCount;
private final int columnCount;

public WorldMap() {
  this(DEFAULT_ROW_COUNT, DEFAULT_COLUMN_COUNT);
}

public WorldMap(int rowCount, int columnCount) {
  this.rowCount = rowCount;
  this.columnCount = columnCount;
}
```
Фиксированные размеры могут быть у шахматной доски(8x8) или игры крестики-нолики(3x3).  
То есть в тех играх, где размер игрового поля жестко фиксирован правилами.  
Здесь размеры должны приниматься в конструктор, чтобы можно было создавать карту произвольных размеров.

+ 👍 Хорошо, что возвращается копия мапы, а не оригинал.  
Тем самым соблюдается инкапсуляция и защищается внутреннее устройство класса
```java
private final Map<Coordinates, Entity> entities = new HashMap<>();

public Map<Coordinates, Entity> getEntities() {
  return new HashMap<>(entities);
}
```

- Никогда не возвращай null
```java
private final Map<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return entities.get(coordinates);  <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Всегда возвращай исключения с текстом, который объясняет причину возникновения этого исключения
```java
throw new NoSuchElementException();

//ПРАВИЛЬНО:
throw new NoSuchElementException("Тут на английской языке написана причина возникновения исключения");
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```java
public void putEntity(Coordinates coordinates, Entity entity) {
  if (isCellAvailablePutEntity(coordinates)) {
    entities.put(coordinates, entity);
  }
}

//ПРАВИЛЬНО:
public void putEntity(Coordinates coordinates, Entity entity) {
  validate(coordinates);  <-- Бросает исключение если координата вне карты
  entities.put(coordinates, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

Сейчас если координата находится вне карты, то операция вставки существа молча проигнорируется.  
Это потенциально может привести к багам, когда существа будут просто исчезать с карты- 
при попытке совершить ход, они будут удаляться по старому адресу и не вставляться по новому.

+ 👍 За исключением названия, метод хороший. Он может вернуть список любых объектов определенного типа
```java
public <T extends Entity> List<T> getAllCreatures(Class<T> clazz)
```

+ 👍 При попытке удалить существо, которого нет в карте, бросается иссключение. Это хорошо
```java
public void removeEntity(Entity entity) {
  Coordinates coordinates = getCoordinates(entity);  <-- тут бросит исключение если существа нет в карте
  entities.remove(coordinates);
}
```

- Метод совершения хода в карте- нарушение SRP.  
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
Карта карта = new Карта();
Заяц заяц = new Заяц();
карта.putEntity(new Координата(0, 0), заяц);
карта.moveEntity(заяц, new Координата (11, 11));

/* class Карта */
public void moveEntity(Entity entity, Coordinates to) {
  if (isCellAvailablePutEntity(to)) {
    removeEntity(entity);
    putEntity(to, entity);
  }
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как?  
Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

+ 👍 В целом, с точки зрения SRP класс хороший.  
Кроме метода `moveEntity(...)` в нем нет лишних ответственностей.

**7. interface PathFinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

+ 👍 Идеально. Из сигнатуры метода понятно, как пользоватьсяч поиском пути
```java
List<Coordinates> findPath(WorldMap worldMap, Coordinates from, Class<? extends Entity> target);
```

- Особенности сигнатур в интерфейсах.

В интерфейсах не пишут "public", потому что там можно разместить только public-методы.  
То же самое относится к константам.  
Посмотри, идея здесь пишет эти слова серым, как бы намекая, что их можно опустить
```java
public interface PathFinder {
  public final static List<Coordinates> SHIFTS = List.of(...);
  public List<Coordinates> findPath(WorldMap worldMap, Coordinates from, Class<? extends Entity> target);
}

//ПРАВИЛЬНО:
public interface PathFinder {
  List<Coordinates> SHIFTS = List.of(...);
  List<Coordinates> findPath(WorldMap worldMap, Coordinates from, Class<? extends Entity> target);
}
```

- Нарушение ООП дизайна интерфейсов.

Интерфейсы предназначены для того, чтобы описывать ЧТО нужно делать, а не КАК это делать.  
Список сдвиговых координат это подробность того, как именно нужно искать
```java
List<Coordinates> SHIFTS = List.of(
  new Coordinates(-1, 0),
  //еще три направления
);
```
То есть, этот интерфейс сразу накладывает ограничения на свои реализации.  
Например, сейчас можно сделать *только* поиск по четырем направлениям, а не по восьми, например.

**8. class BfsPathFinder implements PathFinder**

- Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
while (!neighbors.isEmpty()) {
  for (Coordinates shift : SHIFTS) {
    if (nextEntityOptional.isEmpty()) {
      //...
    } else {
      if (!target.isInstance(nextEntity)) {
        //...
        } else {
          //наконечник стрелы
        }
      }
    }
  }
}
```
Стрела значит, что метод делает несколько дел сразу и его нужно разделить на несколько вспомогательных.
```java
"Если вам нужно более трех уровней вложенности, вы все равно запутались, так что исправьте программу" - Линус Торвальдс.
```

- Неправильный подход к борьбе против null'ов.  
Методы(в данном случае метод карты) в принципе не дролжны возвращать null
```java
Optional<Entity> nextEntityOptional = Optional.ofNullable(worldMap.getEntity(nextStep));
if (nextEntityOptional.isEmpty()) {
  //действия если клетка пустая
} else {
  //действия если клетка занята
}

//ПРАВИЛЬНО:
if(worldMap.isПусто(nextStep)) {
  //действия если клетка пустая
} else {
  Entity entity = worldMap.getEntity(nextStep); <-- геттер должен бросить исключение если существа нет по координате
  //действия если клетка занята
}
```

+ 👍 В целом всё ок.

**9. abstract class Entity и его простые наследники Tree/Rock/Grass**

👍 Идеально
```java
public abstract class Entity {
}

public class Tree extends Entity {
}
```

**10. abstract class Creature extends Entity**

- Нарушение конвенции кода. Константы должны стоять выше остальных полей.

- Не забывай про инкапсуляцию. Всегда явно указывай область видимости
```java
public abstract class Creature extends Entity {
  PathFinder bfs;
  //...
  abstract void interact(WorldMap worldMap, Coordinates target);
}
```

- Придерживайся единообразия
```java
//...
protected int speed;
PathFinder bfs;

public Creature(int speed, int hp, Class<? extends Entity> target) {
  //...
  this.speed = speed;
  bfs = new BfsWithNodePathFinder();
}

//ПРАВИЛЬНО:
public Creature(int speed, int hp, Class<? extends Entity> target) {
  //...
  this.speed = speed;
  this.bfs = new BfsWithNodePathFinder();
}
```

- Нарушение DIP, ошибка в применении интерфейсов. 

Если есть интерфейс поиска, то Существо должно использовать его именно как интерфейс, а не как конкретную реализацию.  
Иначе в классе Creature нельзя будет использовать другие поиски без перекомпиляции
```java
public abstract class Creature extends Entity {
  //...
  PathFinder bfs;

  public Creature(int speed, int hp, Class<? extends Entity> target) {
    this.speed = speed;
    //...
    bfs = new BfsWithNodePathFinder();
  }
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //...
  private final PathFinder pathFinder;

  public Creature(int speed, int hp, Class<? extends Entity> target, PathFinder pathFinder) {
    this.speed = speed;
    //...
    this.pathFinder = pathFinder;
  }
}
```

**Условные примеры.**

Правильно без интерфейсов:
```java
class Main {
  public static void main() {
    Заяц заяц = new Заяц();
  }
}

class PathFinder {...}  <-- ТУТ ЭТО КЛАСС

class Заяц {
  private final PathFinder pathFinder;

  public Заяц() {
    this.pathFinder = new PathFinder();
  }
}
```

Правильно с интерфейсами:
```java
class FirstMain {
  public static void main() {
    PathFinder pathFinder = new BfsPathFinder();
    Заяц заяц = new Заяц(pathFinder);
  }
}

class SecondMain {
  public static void main() {
    PathFinder pathFinder = new AstarPathFinder();
    Заяц заяц = new Заяц(pathFinder);
  }
}

interface PathFinder {...}  <-- ТУТ ЭТО ИНТЕРФЕЙС

class Заяц {
  private final PathFinder pathFinder;

  public Заяц(PathFinder pathFinder) {
    this.pathFinder = pathFinder;
  }
}
```

- Упрощай
```java
if (path.size() == 1) {
  Coordinates targetStep = path.get(0);
  interact(worldMap, targetStep);
  worldMap.moveEntity(this, path.get(0));
  return;
}
worldMap.moveEntity(this, path.get(0));

//ПРАВИЛЬНО:
Coordinates step = path.get(0);
if (path.size() == 1) {
  interact(worldMap, step);
  worldMap.moveEntity(this, step);
  return;
}
worldMap.moveEntity(this, step);
```

- Участки с try-catch не смешивай с бизнес логикой, изолируй во вспомогательных методах
```java
public void makeMove(WorldMap worldMap) {
  for (int i = 0; i < speed; i++) {
    try {
      //миллион строк бизнес логики
    } catch (NoSuchElementException e) {
      return;
    }
  }
}
```
*"ЧК", гл.3*  
*Хорстманн "Java. Библиотека профессионала", т1., гл.11*
```java
"Обработка исключений не может заменить собой простую проверку" - Хорстманн
```

- Не вижу причин, зачем нужно ограничивать область видимости этого метода
```java
protected boolean isDead() {
  return hp <= 0;
}

//ПРАВИЛЬНО:
public boolean isDead() {...}
```
Все классы, которые используют существ, должны иметь возможность узнать, живо существо или нет.

- Магическое число. Что такое "50" мне неочевидно
```java
return hp <= 50;
```

+ 👍 С точки зрения SOLID, в целом класс хороший.

**11. class Herbivore/Predator extends Creature**

Это тоже имеет смысл сделать константой
```java
public class Herbivore extends Creature {
  //...
  public Herbivore() {
    super(HERBIVORE_SPEED, MAX_HP, Grass.class);
  }
}

//ПРАВИЛЬНО:
public class Herbivore extends Creature {
  //...
  public Herbivore() {
    super(HERBIVORE_SPEED, MAX_HP, FOOD);
  }
}
```

+ 👍 В остальном всё ок.

**12. Фабрика существ**

Интерфейс фабрики и простая ее реализация
```java
interface EntityFactory 
class RealEntityFactory implements EntityFactory 
```

- Зачем в этом случае нужна фабрика существ и нужна ли она- вопрос дискуссионный.

Паттерн Фабрика применяется тогда, когда объект создается по сложным правилам.
Здесь объекты создаются очень просто- через конструктор по умолчанию.
```java
public interface EntityFactory {
  Entity create(Class<? extends Entity> entityType);
}

public class RealEntityFactory implements EntityFactory {  <-- ФАБРИКА
  private final Map<Class<? extends Entity>, Supplier<? extends Entity>> creators = Map.of(
    Herbivore.class, () -> new Herbivore(), <-- Создание объектов через конструктор по умолчания
    Predator.class, () -> new Predator(), <-- Создание объектов через конструктор по умолчания
    //все остальные тоже создаются через конструктор по умолчанию
  );

  @Override
  public Entity create(Class<? extends Entity> entityType) {
    Supplier<? extends Entity> creator = creators.get(entityType);
    return creator.get();
  }
}
```
Зачем в этом случае нужна фабрика, мне неясно.

- Интерфейс тут избыточен.

Ок, фабрика так фабрика. Но зачем к этой фабрике интерфейс?  
Интерфейсы вводят тогда, когда подразумевается возможность создать разные реализации хотя бы в теории.  
Здесь невозможно сделать разные реализации интерфейса потому что при создании объекты не параметризируются.

То есть в этом смысла нет(условный пример):
```java
interface PredatorFactory {
  Predator get();
}

class PredatorFactoryImp implements PredatorFactory {
  @Переопределено
  public Predator get() {
    return new Predator();
  }
}
```
В этом примере невозможно придумать вторую реализацию интерфейса PredatorFactory.  
Поэтому интерфейс в этом примере- лишний бессмысленный уровень абстракции.

Чтобы появился смысл в интерфейсе, должна быть возможность сделать разные его реализации.  
Например так:
```java
interface PredatorFactory {
  Predator get();
}

class SlowPredatorFactory implements PredatorFactory {
  private final static int SPEED = 2;

  @Переопределено
  public Predator get() {
    return new Predator(SPEED);
  }
}

class FastPredatorFactory implements PredatorFactory {
  private final static int SPEED = 5;

  @Переопределено
  public Predator get() {
    return new Predator(SPEED);
  }
}
```

Введение в проект любого нового интерфейса или класса должно быть оправдано.  
Если оно неоправдано- это ухудшает архитектуру проекта, а не улучшает её.

- Метод `create(...)` в `RealEntityFactory` может вернуть null.

**13. abstract class Action**

👍 Идеально
```java
public abstract class Action {
  public abstract void perform(WorldMap worldMap);
}
```

**14. Классы-реализации Action**

👍 Всё ок.

**15. class RendererWorldMap**

👍 Всё ок.

**16. class Simulation**

+ 👍 Принимает достаточное количество зависимостей в конструктор.

- Нарушение паттерна GRASP "Creator"(Создатель)

Здесь класс принимает в конструктор зависимость, которую должен создавать самостоятельно:
```java
public Simulation(WorldMap worldMap, RendererWorldMap renderer) {...}

//ПРАВИЛЬНО:
public Simulation(WorldMap worldMap) {
  //...
  this.renderer = new RendererWorldMap();
}
```
Creator гласит, что создавать объект должен тот, кто его использует. 

Рассмотрим, что это значит.

В данном случае `Simulation` принимает в конструктор `GameMap`- это правильно.  
Потому что карта может быть создана разных размеров и Simulation не знает, какая именно карта нужна в этот раз.  
Поэтому ее создает клиент, а потом инжектит в этот класс.

Но принимать в конструктор `RendererWorldMap`- уже неправильно.  
Потому что `RendererWorldMap` это конкретный класс, а не интерфейс и этот класс не параметризируется. 

Экземпляр `RendererWorldMap` всегда одинаковый.  
Поэтому, согласно паттерна Creator, объект `RendererWorldMap` *здесь* должен создавать сам класс Simulation.

Технически, можно создать наследника `RendererWorldMap`, переопределить в нем методы и передавать в конструктор Simulation.  
Но если программист хочет использовать в своем классе другие классы через полиморфизм, то он должен обозначить свои намерения более явно.  
Например, передавать в конструктор интерфейс или абстрактный класс.

👍 В остальном всё ок.

**17. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

**18. Использование многопоточности**

👍 Всё ок.

**19. Использование интерфейсов**

Интерфейсы это важная концепция в ООП языках, поэтому обрати внимание на правильное их использование.  

**Вводи интерфейсы тогда, когда есть практическая возможность сделать разные их реализации.** 

Например, у тебя есть интерфейс поиска пути и две его реализации
```java
public interface PathFinder {
  public List<Coordinates> findPath(WorldMap worldMap, Coordinates from, Class<? extends Entity> target);
}

class BfsWithNodePathFinder implements PathFinder {...}
class BfsPathFinder implements PathFinder {...}
```
Даже если бы реализаций было не две, а одна, то этот интерфейс имел бы смысл как точка расширения программы.  
Потому для этого интерфейса можно сделать разные реализации.

Для интерфейса `EntityFactory` можно сделать только одну реализацию.  
Поэтому здесь интерфейс не нужен, достаточно просто класса.

**Если существует интерфейс, то нужно его корректно использовать.**  

Сам по себе интерфейс `PathFinder` в проекте это хорошо.  
Но сейчас существа используют его не как интерфейс, а как конкретную реализацию.  
И поэтому в нем тоже пропадает смысл.

Если в проекте есть интерфейс, то клиенты должны с ним работать не как с конкретной реализацией, а как с интерфейсом согласно принципу DIP.

## ВЫВОД

Работа над ошибками пошла на пользу, устранены грубые ошибки архитектуры прошлой версии.  
Сейчас ООП архитектура программы норм и в целом всё ок 👍👍👍

n.149(315)  
#ревью #симуляция 