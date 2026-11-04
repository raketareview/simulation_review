https://github.com/ivasn2/simulation  
[Иван]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неправильно.

## ХОРОШО

+ 👍 Есть пауза/пуск во время работы
+ 👍 Размножение существ

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Не называй пакет именем "map", это название стандартного интерфейса "Map". 

Не называй пакеты так же, как называются классы, интерфейсы, пакеты библиотеки java core.  
В данном случае- стандартного интерфейса Map
```java
package map;

//ПРАВИЛЬНО:
package worldmap;
```

- Стандартные названия для ширины и высоты - это "ширина" и "высота"
```java
public class WorldMap {
  private final int rows;
  private final int columns;
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private final int height;
  private final int width;
  //...
}
```

- Придерживайся единообразия
```java
public boolean isCellEmpty(Coordinates coordinates) {...}

public boolean isCellOnTheMap(Coordinates cell) {...}

//ПРАВИЛЬНО:
public boolean isCellEmpty(Coordinates coordinates) {...}

public boolean isCellOnTheMap(Coordinates coordinates) {...}
```

- В названия не нужно вставлять частицы "Of", "The", "Are" и т.д.

Это только делает названия более громоздкими.  
"Of" в названиях методов можно использовать только в контексте "valueOf()"
```java
boolean isCellOnTheMap(Coordinates cell) {...}

//ПРАВИЛЬНО:
boolean isWithin(Coordinates coordinates) {...}
```

- В названии переменных не используй предлог "To". 

Предлог "To" используется в методах конвертации: `toList()`, `toString()`, `toInt()`, `meterToFeet()`.   
В названии переменных не используй "To", потому что переменная хранит данные, а не конвертирует их
```java
int stepsToTake = Math.min(path.size(), getSpeed());

//ПРАВИЛЬНО:
int stepCount = Math.min(path.size(), getSpeed());
```

- Из-за "To" читается как "преобразовать добавочный экшен в инит экшен"
```java
void addActionToInitActions(Action action) 

//ПРАВИЛЬНО:
void addInitAction(Action action) 
```

- Н вбрсв глсн бкв в нзвнх
```java
Random rnd = new Random();

//ПРАВИЛЬНО:
Random random = new Random();
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```java
HashMap<Coordinates, Entity> entityPositions;

//ПРАВИЛЬНО:
Map<Coordinates, Entity> entityPositions;
```
Общее правило: ArrayList нужно использовать через List, HashMap через Map, HashSet через Set и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**3. Нарушение DRY**

Магические буквы, числа, слова. Вводи константы. 

- Совместная магия.

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов.  
Либо перенеси ее в третий класс и из первых двух классов обращайся к этим константам- эти данные должны быть синхронизированы между собой
```java
public class ConsoleRenderer {

  public void render(WorldMap worldMap, int turnCount) {
    //...
    System.out.println("Commands: p - pause, s - resume, q - quit");
  }
}

public class Main {
  public static void main(String[] args) {
    final String PAUSE = "p";
    final String RESUME = "s";
    final String QUIT = "q";
    //...
  }
}
```

*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru, "Замена магического числа символьной константой"*

**4. Форматирование**

- Форматирование строк.

Если нужно печатать или создавать строку с более чем одним подстановочным значением или значение вставляется внутрь сообщения, 
используй форматирование - тогда сразу будет виден весь шаблон сообщения
```java
System.out.println("=== Ход №" + turnCount + " ===");

//ПРАВИЛЬНО:
System.out.printf("=== Ход № %d ===  \n", turnCount);
```

**5. Exceptions**

- Не бросай базовый exception. Конкретизируй ситуацию- бросай то исключение, которое подходит под этот конкретный случай
```java
throw new RuntimeException(e);
```
*Хорстманн "Java. Библиотека профессионала", т.1, гл.11*
```java
"Не ограничивайтесь генерацией RuntimeException. 
Найдите подходящий подкласс или создайте собственный." - Хорстманн
```

**6. class Coordinates**

- Класс может быть преобразован в record без потери функционала. Например вот так:
```java
public class Coordinates {
  //30 строк кода

  public List<Coordinates> neighborCells() {...}
}

//ЛУЧШЕ:
public record Coordinates (int x, int y) {
  public List<Coordinates> neighborCells() {...}
}
```
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [ТУТ](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

- Нарушение SRP 

Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: x,y или row,column.  
Лучше даже, если этот класс будет record'ом.

Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия, тем самым нарушая SRP: 
```java
public List<Coordinates> neighborCells() {  <-- Создает список соседних координат
  List<Coordinates> neighbors = new ArrayList<>();

  neighbors.add(new Coordinates(getX() + 1, getY())); // Вниз
  neighbors.add(new Coordinates(getX() - 1, getY())); // Вверх
  neighbors.add(new Coordinates(getX(), getY() + 1)); // Вправо
  neighbors.add(new Coordinates(getX(), getY() - 1)); // Влево

  return neighbors;
}
```

Сейчас соседними координатами считаются четыре: справа, слева, сверху, снизу.  
Если концепция поиска существ на карте поменяется и ты будешь искать путь к цели не по 4-м, а по 8-и направлениям (с учётом диагоналей), то придется вносить изменение в код этого класса.  
Следовательно, в этом классе есть вторая причина для изменения - способ формирования списка соседних координат в зависимости от типа поиска существ на карте. 

- Для координаты лучшим решением будет использовать `record`.

Это подойдет для симуляции и любой другой игры на прямоугольной доске (Морской бой, Шахматы и т.д.)  
Вот идеальная координата:
```java
public record Coordinates (int x, int y) {
  //тут пусто
}
```

**7. class WorldMap**

- Конструктор должен стоять выше всех остальных методов.

- Никогда не возвращай null
```java
private final HashMap<Coordinates, Entity> entityPositions;

public Entity getEntity(Coordinates coordinates) {
  return entityPositions.get(coordinates);  <-- вернет null если coordinates нет в entityPositions 
}
```
Возврат null повышает риск возникновения NullPointerException в программе.

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность  
```java
public void removeEntity(Coordinates coordinates) {
  entityPositions.remove(coordinates);
}


public void addEntity(Coordinates coordinates, Entity entity) {
  if (isCellEmpty(coordinates)) {
    entity.setCoordinates(coordinates);
    entityPositions.put(coordinates, entity);
  }
}

//ПРАВИЛЬНО:
public void removeEntity(Coordinates coordinates) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение    
  entityPositions.remove(coordinates);
}

public void addEntity(Coordinates coordinates, Entity entity) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение 
  if (!isCellEmpty(coordinates)) {
    //если правилами запрещено вставлять существо на занятую координату, то бросить исключение
  }
  entity.setCoordinates(coordinates);
  entityPositions.put(coordinates, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Нарушение SRP.

Этот метод не нужен карте для выполнения ею единой ответственности по хранению в себе существ:
```java
public boolean isCellContainsRelative(Coordinates neighborCell, Class<? extends Entity> targetClassEntity) {
  return ((!isCellEmpty(neighborCell)) && (getEntity(neighborCell).getClass() == targetClassEntity));
}
```
Что бы ни делал этот метод (а что он делает я хз), он не нужен для выполнения картой своих обязанностей по хранению существ внутри себя. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "WorldMapUtils".

- Нарушение SRP.

Метод телепортации существ тоже не относится к ответственности хранения существ в карте
```java
public void moveEntity(Coordinates destination, Entity entityToMove) {
  removeEntity(entityToMove.getCoordinates());
  addEntity(destination, entityToMove);
}
```
Вся логика по выполнению перемещения существа по карте должна находиться в одном месте.  
Например, в `Creature.makeMove(...)`.  
Эта логика не должна быть размазана по нескольким классам.

+ 👍 В целом с точки зрения SRP класс хороший. Имеющиеся нарушения SRP в нем - незначительные.

**8. abstract class Entity**

- Содержит координату
```java
public abstract class Entity {
  private Coords coords;
  //...
}
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```java
private final String symbol;
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах (консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

**9. abstract class Creature extends Entity**

- Не вижу причин для разных подходов при хранении скорости и хранении типа еды
```java
public abstract class Creature extends Entity {
  private final int speed;
  //...

  public int getSpeed() {...}

  public abstract Class<? extends Entity> getTargetClass();
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  private final int speed;
  private final Class<? extends Entity> food;
  //...

  public int getSpeed() {...}

  public Class<? extends Entity> getFood() {...}
}
```

- Нарушение SRP.

Существо не должно само себя удалять из карты.  
Попытка двинуть мёртвое существо, это некорректное действие, нужно бросать исключение - мертвый негр не идет играть в баскетбол
```java
public void makeMove(WorldMap worldMap) {
  if (isDead()) {
    worldMap.removeEntity(getCoordinates());
    return;
  }
  //...
}

//ПРАВИЛЬНО:
public void makeMove(WorldMap worldMap) {
  if (isDead()) {
    //Бросить исключение: мертвому существу запрещено ходить
  }
  //...
}
```

Существо из карты может удалить другое существо.  
Например, Волк, съев Зайца, может его удалить из карты.  

Альтернативный подход: мёртвых существ с карты может убирать специальный экшен.  
А пока он это не сделает, мертвые существа можно показывать на карте вот так: 💀  
Ну это так, если хочется по приколу.

- Нарушение инкапсуляции.

Публичными должны быть только те методы, которые предназначены для использования клиентами.  
Методы и поля, предназначенные для использования потомками, должны быть `protected`.  
Остальные методы и поля в классах должны быть `private`
```java
public abstract void interact(Entity targetEntity, WorldMap worldMap, List<Coordinates> path);

//ПРАВИЛЬНО:
protected abstract void interact(Entity targetEntity, WorldMap worldMap, List<Coordinates> path);
```

Клиент не должен иметь возможности вызвать у объекта метод, который предназначен только для использования внутри своего класса.  
Придерживайся правила минимального открытого интерфейса.

*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

+ 👍 В целом класс норм.

**10. class Herbivore extends Creature**

Используй константы
```java
public class Herbivore extends Creature {

  public Herbivore(String symbol, int hp, int speed, int starvationDamage) {
    super(symbol, hp, speed, starvationDamage);
  }

  public Herbivore() {
    this("\uD83E\uDD8C", 100, 2, 5);
  }

  @Override
  public Class<? extends Entity> getTargetClass() {
    return Grass.class;
  }
  //...
}


//ЛУЧШЕ:
public class Herbivore extends Creature {
  private static final String SPRITE = "\uD83E\uDD8C";
  private static final int HP = 100;
  private static final int SPEED = 2;
  private static final int STARVATION_DAMAGE = 5;
  
  private static final Class<? extends Entity> TARGET_CLASS = Grass.class;

  public Herbivore() {
    this(SPRITE, HP, SPEED, STARVATION_DAMAGE);
  }

  public Herbivore(String symbol, int hp, int speed, int starvationDamage) {
    super(symbol, hp, speed, starvationDamage);
  }

  @Override
  public Class<? extends Entity> getTargetClass() {
    return TARGET_CLASS;
  }
  //...
}
```

**11. class BreadthFirstSearch**

- Соблюдай требования утилитных классов. 

Это утилитный класс, он содержит только статические методы.  
Утилитные классы, как и классы констант, должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.

*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
public static List<Coordinates> findPath(Coordinates start, WorldMap worldMap, Class<? extends Entity> targetClass) {...}
```

- Антипаттерн "Стрела".

Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания

Сейчас 6 уровней
```java
while (!cellsToExplore.isEmpty()) {
  for (Coordinates neighbor : neighborCells) {
    if ((worldMap.isCellOnTheMap(neighbor)) && ....) {
      if (worldMap.isCellContainsRelative(neighbor, targetClass)) {
        while (true) {
          if (previousCell.equals(start)) {
            //наконечник стрелы
          }
        }
      }
    }
  }
}
```
"Стрела" значит, что метод делает несколько дел сразу и его нужно разделить на несколько вспомогательных.
```java
"Если вам нужно более трех уровней вложенности, вы все равно запутались, так что исправьте программу" - Линус Торвальдс
```

**12. interface Action**

+ 👍 Идеально
```java
public interface Action {
  void perform(WorldMap worldMap);
}
```

**13. class MoveAllCreaturesAction implements Action**

- Лишняя перестраховка
```java
public class MoveAllCreaturesAction implements Action {

  @Override
  public void perform(WorldMap worldMap) {
    List<Entity> snapshot = new ArrayList<>(worldMap.getAllEntities());  <-- Делает копию КОПИИ (избыточная перестраховка)
    //...
  }
}

public class WorldMap {
  //...

  public Collection<Entity> getAllEntities() {
    return List.copyOf(entityPositions.values()); <-- Делает копию (это правильно)
  }
}
```

**14. class ReproduceAction implements Action**

- Никогда не возвращай null
```java
private Creature findNeighborPartner(Creature parent, WorldMap worldMap) {
  //...
  return null;
}
```

- Стрела
```java
for (Entity entity : worldMap.getAllEntities()) {
  if ((entity.getClass() == creatureClass)) {
    if ((parent2 != null) && (parent2.canReproduce())) {
      if (babyPlace != null) {
        try {
          // наконечник стрелы
        }
      }
    }
  }
}
```

- Старайся не использовать рефлексию в бизнес коде.

Рефлексия в бизнес коде - это проявление алгоритмического бессилия
```java
Creature newBabyEntity = creatureClass.getDeclaredConstructor().newInstance();
```

Минус этого подхода состоит еще и в том, что таким образом у креатур вызывается только конструктор по умолчанию.  
Но у Волков и Зайцев есть и альтернативные перегруженные конструкторы.  
Получается, что `ReproduceAction` не может работать в проекте, где Креатуры будут создаваться с кастомными параметрами, а не дефолтными.

Чтобы репродюсер был универсальным, он должен принимать в себя не тип класса, а интерфейс для создания экземпляра этого класса
```java
public class Main {
  public static void main(String[] args) {
    //...
    simulation.addActionToTurnActions(new ReproduceAction(Herbivore.class));
    simulation.addActionToTurnActions(new ReproduceAction(Predator.class));
    //...
  }
}

public class ReproduceAction implements Action {

  private final Class<? extends Creature> creatureClass;

  public ReproduceAction(Class<? extends Creature> creatureClass) {
    this.creatureClass = creatureClass;
  }

  @Override
  public void perform(WorldMap worldMap) {
    //...
    Creature newBabyEntity = creatureClass.getDeclaredConstructor().newInstance();
    //...
  }
}

//ПРАВИЛЬНО:
public class FirstMain {
  public static void main(String[] args) {
    //...
    simulation.addActionToTurnActions(() -> new Herbivore());
    simulation.addActionToTurnActions(() -> new Predator());
    //...
  }
}

public class SecondMain {
  public static void main(String[] args) {
    //...
    simulation.addActionToTurnActions(() -> new Herbivore(1, 2, 3));
    simulation.addActionToTurnActions(() -> new Predator(2, 3, 4));
    //...
  }
}

public class ReproduceAction implements Action {

  private final Supplier<Creature> creatureCreator;

  public ReproduceAction(Supplier<Creature> creatureCreator) {...}

  @Override
  public void perform(WorldMap worldMap) {
    //...
    Creature newBabyEntity = creatureCreator.get();
    //...    
  }
}
```

**15. class SpawnEntitiesAction implements Action**

+ 👍 Использование стандартного интерфейса Java для универсального размножения существ - это хорошо 
```java
private final Supplier<? extends Entity> entityFactory;
```

- Не делай адовых условий в `fori`
```java
for (int i = 0; ((i < toSpawnCount) && (!worldMap.isFull())); i++) {
  //...  
}

//ПРАВИЛЬНО:
for (int i = 0; i < count; i++) {
  if(worldMap.isFull()) {
    break;  
  }
  //...
}
```

**16. class ConsoleRenderer**

- Нарушение SRP.

Рендерер карты должен распечатывать только карту.  
Рендерер карты не должен знать, что его используют в контексте совершения каких-то ходов
```java
public void render(WorldMap worldMap, int turnCount) {...}

//ПРАВИЛЬНО:
public void render(WorldMap worldMap) {...}
```

Тем более он не должен знать про правила игры
```java
System.out.println("Commands: p - pause, s - resume, q - quit");
```

- Карта распечатывается неправильно
```java
static void main() {
  WorldMap worldMap = new WorldMap(6, 6);
  int x = 0;
  int y = 5;

  worldMap.addEntity(new Coordinates(x, y), new Tree());

  ConsoleRenderer renderer = new ConsoleRenderer();
  renderer.render(worldMap, 0);
}

//РЕЗУЛЬТАТ:
🟫🟫🟫🟫🟫🌳
🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫
```

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < worldMap.getRows(); i++) {
  for (int j = 0; j < worldMap.getColumns(); j++) {
    Coordinates coordinates = new Coordinates(i, j);
    //...
  }
}

//ЛУЧШЕ:
for (int y = 0; y < worldMap.getRows(); y++) {
  for (int x = 0; x < worldMap.getColumns(); x++) {
    Coordinates coordinates = new Coordinates(y, x);
    //...
  }
}
```

И тогда сразу понятно, почему карта распечатывается неправильно.

**17. class Simulation**

- Симуляция должна принимать карту целиком, а не по запчастям
```java
public Simulation(int length, int width) {
  this.worldMap = new WorldMap(length, width);
}

//ПРАВИЛЬНО:
public Simulation(World worldMap) {
  this.worldMap = worldMap;
}
```

- Добавление экшенов.

Хорошо, что есть возможность добавления экшенов клиентами - это делает проект более универсальным.  
Благодаря этому можно делать разные игровые конфигурации (разные Main'ы) с разным набором экшенов без изменения кода в классе `Similation` 
```java
public class Main {
  public static void main(String[] args) {
    Simulation simulation = new Simulation(15, 30);
    simulation.addActionToInitActions(new SpawnEntitiesAction(Grass.class, Grass::new, 50, 100));
    //...
    simulation.startSimulation();
  }
}
```
Тем самым класс `Similation` закрыт для изменения, но открыт для расширения - соблюдает OCP, это хорошо.

Но сейчас через публичный метод можно добавлять экшены во время работы Симуляции, а не только *на этапе создания её экземпляра*.

Если имеется в виду начальное конфигурирование экземпляра симуляции и не предполагается добавление экшенов во время её работы, а здесь не предполагается, 
то для создания экземпляра Симуляции нужно использовать паттерн `Builder`.

Например, так:    
```java
public class Main {
  public static void main(String[] args) {
    Simulation simulation = new Simulation.Builder()
        .addWorldMap(new WorldMap(10, 10))
        .addInitAction(new UnoAction())
        .addTurnAction(new DosAction())
        .addTurnAction(new TresAction())
        .addTurnAction(new MoveAction())
        .build();

    simulation.startSimulation();
  }
}

public class Simulation {
  //...

  private Simulation(WorldMap worldMap, List<Action> initActions, ...) {...}
  //...
  
  public static class Builder {
    private WorldMap worldMap;
    private final List<Action> initActions = new ArrayList<>();
    //...
  
    public Builder addWorldMap(WorldMap worldMap) {
      this.worldMap = worldMap;
      return this;
    }

    public Builder addInitAction(Action action) {
      initActions.add(action);
      return this;
    }

    //...
    public Simulation build() {...}
  }
}
```

**18. class Main**

Содержит точку входа main.

- Нарушение SRP.

Main должен только сконфигурировать зависимости и запустить программу.  
Управлять работой программы этот класс не должен.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

Определи сущности, которые сейчас сидят в этом классе и вынеси их в отдельные классы.

Например, сущность, которая управляет симуляцией, получая от юзера команды пауза/пуск.

## ВЫВОД

Замечаний меньше, чем обычно. Но всё ещё есть над чем поработать.

n.177(380)  
#ревью #симуляция 