https://github.com/.../Simulation  
[a...]

Есть над чем поработать.  
Нужно глубже вникнуть в ООП и сделать многопоточность в проекте.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

**1. Нет паузы/пуск во время работы. Хотя это требование есть в ТЗ.**

**2. Команды не работают**
```jav
Для движения конкретной сущностью введите C
c
Для прекращения симуляции введите E
Для продолжения симуляции введите Y
Для движения конкретной сущностью введите C
c
```
Не отработала команда 'c', хотя я ввел именно английскую 'c'.

**3. Нет защиты от дурака**
```java
Введите координату по горизонтали:
fck
Exception in thread "main" java.lang.NumberFormatException: For input string: "fck"
```

**4. Не работает ручное управление ходом сущности**
```java
Симуляция на шаге 5
🌿🌳🟫🦊🟫🟫🟫🟫🟫🦊
🌳🗻🟫🟫🌿🌳🌳🟫🟫🦊
🐰🗻🟫🗻🟫🦊🟫🌳🟫🟫
🌳🌿🌳🌿🌳🌿🌳🟫🟫🟫
🦊🟫🟫🌿🌿🟫🟫🌿🟫🟫
🗻🗻🌿🟫🟫🟫🦊🟫🟫🟫
🟫🌳🌿🗻🟫🌿🟫🗻🌳🟫
🌿🌳🌿🟫🟫🟫🌳🟫🟫🌿
🌿🟫🌿🟫🌳🗻🌳🌿🟫🐰
🌳🟫🟫🗻🗻🐰🟫🟫🟫🗻
Для прекращения симуляции введите E
Для продолжения симуляции введите Y
Для движения конкретной сущностью введите C
C
Введите координату по горизонтали:
1
Введите координату по вертикали:
1
Симуляция на шаге 6
🌿🌳🟫🦊🟫🟫🟫🐰🐰🦊
🌳🗻🟫🟫🌿🌳🌳🟫🟫🦊
🐰🗻🟫🗻🐰🦊🟫🌳🐰🟫
🌳🌿🌳🌿🌳🌿🌳🟫🟫🟫
🦊🟫🟫🌿🌿🟫🟫🌿🟫🟫
🗻🗻🌿🐰🟫🐰🦊🟫🟫🐰
🟫🌳🌿🗻🟫🌿🐰🗻🌳🟫
🌿🌳🌿🟫🟫🟫🌳🟫🟫🌿
🌿🟫🌿🟫🌳🗻🌳🌿🟫🐰
🌳🟫🟫🗻🗻🐰🟫🟫🟫🗻
```

## ХОРОШО

+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Координаты в существах хранятся начиная с Creature
+ 👍 Меню для создания карт разного размера
+ 👍 Красивая распечатка карты  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim158/img0.png) 

## ЗАМЕЧАНИЯ

**1.1. Нейминг**

- Не называй пакет именем "map", это название стандартного интерфейса "Map". 

Не называй пакеты так же, как называются классы, интерфейсы, пакеты библиотеки java core.  
В данном случае- стандартного интерфейса Map
```java
package map;

//ПРАВИЛЬНО:
package worldmap;
```

- "Рендер" это процесс, а "рендерер" это тот, кто выполняет этот процесс
```java
class Render

//ПРАВИЛЬНО:
class Renderer
```

- Одной концепции- одно название.  
Либо "Printer" и "print", либо "Renderer" и "render"
```java
public final class Render {

  public void print() {...}
}
//ПРАВИЛЬНО:
public final class Renderer {

  public void render() {...}
}
```

- Избыточное уточнение. Не дублируй имя класса в названии полей класса
```java
public class WorldMap {
  private final Map<Coordinates, BaseEntity> worldEntities = new HashMap<Coordinates, BaseEntity>();
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private final Map<Coordinates, BaseEntity> entities = new HashMap<>();
  //...
}
```

- Используй стандартные названия, в данном случае это ширина и высота
```java
public class WorldMap {
  private final int sizeN;
  private final int sizeM;
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private final int width;
  private final int height;
  //...
}
```

- Этот объект в единственном числе называется "coordinates"
```java
Coordinates coordinate = pathCoordinates.poll();

//ПРАВИЛЬНО:
Coordinates coordinates = pathCoordinates.poll();
```

- С большой буквы называются только классы
```java
private int Hp;

//ПРАВИЛЬНО:
private int hp;
```

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относится.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
Set<EntityType> availableSet = new HashSet<>();
Set<Coordinates> coordinatesSet = new HashSet<>();  //class Movement

//ПРАВИЛЬНО:
Set<EntityType> availableEntityTypes = new HashSet<>();
Set<Coordinates> neighborCoordinates = new HashSet<>();
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**1.2. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки.  
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```java
if (y != 0 && isAvailableMove(potentialCoordinateYMinus, worldMap, coordinateEntity))
  coordinatesSet.add(potentialCoordinateYMinus);

//ПРАВИЛЬНО:
if (y != 0 && isAvailableMove(potentialCoordinateYMinus, worldMap, coordinateEntity)) {
  coordinatesSet.add(potentialCoordinateYMinus);
}
```

*"Oracle Java code conventions"*

**2. Исключения**

- Перехватывай конкретное исключение.

Всегда перехватывай не базовое, а конкретное исключение, которое может сгенерировать код.  
В данном случае метод `Integer.parseInt()` может бросить не какое угодно исключение, а конкретное- NumberFormatException
```java
try {
  intM = Integer.parseInt(strM);
  //...
} catch (Exception e) {
  System.out.println("Введите целое число");
}

//ПРАВИЛЬНО:
try {
  intM = Integer.parseInt(strM);
  //...
} catch (NumberFormatException e) {
  System.out.println("Введите целое число");
}
```

- Не смешивай бизнес логику и обработку ошибок.

Не используй исключения для организации ветвлений бизнес логики вместо `if-else if`.  
Обработку ошибок изолируй в отдельных методах
```java
public static void main(String[] args) {
  while (true) {
    System.out.println("Введите ширину поля для симуляции:");
    String strM = inputScanner.nextLine();
    try {
      intM = Integer.parseInt(strM);

      if (intM > 0) {
        break;
      }
    } catch (Exception e) {
      System.out.println("Введите целое число");
    }
    //....
  }
}

//ПРАВИЛЬНО:
public static void main(String[] args) {
  while (true) {
    System.out.println("Введите ширину поля для симуляции:");
    String strM = inputScanner.nextLine();
    if(!isNumber(String s)) {
      System.out.println("Введите целое число");
      continue;
    }
    intM = Integer.parseInt(strM); 
    if (intM > 0) {
      break;
    }
    //....
  }
}

private static boolean isNumber(String s) {
  try{
    Integer.parseInt(s);
    return true;
  } catch (...) {
    return false;
  } 
} 
```
*"ЧК", гл.3, "Изолируйте блоки try/catch"*

**3. Пакет map**

В этом пакете много классов, не все из которых идеологически могут быть в пакете с таким названием.  
Название пакета должно объяснять, что он хранит.  
Либо дай пакету более общее название типа "common"(хотя здесь именно это название неудачное).  
Либо, что лучше, сделай несколько пакетов
```java
map
 |
 +--Coordinates
 +--Path
 +--PathFinder
 +--Render
 +--WorldMap

//ПРАВИЛЬНО: 
worldmap
 |
 +--Coordinates
 +--WorldMap

pathfinder
 |
 +--Path
 +--PathFinder

renderer
 |
 +--Render
```

**4. record Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.
```java
public record Coordinates(int x, int y) {}
```

**5. class WorldMap**

- Избыточно. Тут достаточно пустых "<>"
```java
public class WorldMap {
  private final Map<Coordinates, BaseEntity> блаблабла = new HashMap<Coordinates, BaseEntity>();
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private final Map<Coordinates, BaseEntity> блаблабла = new HashMap<>();
  //...
}
```

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Подсчитать количество существ определенного типа    
🔸 Определяет что наступил кризис: `boolean isCrises() {...}`  

Наверное, для проекта в целом полезно иметь метод, который определяет кризис, чтобы это ни значило.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно знать, наступил ли кризис.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

- Никогда не возвращай null
```java
private final Map<Coordinates, BaseEntity> worldEntities = new HashMap<Coordinates, BaseEntity>();

public BaseEntity getEntity(Coordinates coordinates) {
  return worldEntities.get(coordinates);  <-- вернет null если coordinates нет в worldEntities
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность 
```java
public void removeEntity(Position p) {
  Entity entityToRemove = entities.remove(p);
  positions.remove(entityToRemove);
}

//ПРАВИЛЬНО:
public void removeEntity(Position position) {
  validate(position);  <-- Если координата не в пределах карты, то бросает исключение  
  Entity entityToRemove = entities.remove(p);
  positions.remove(entityToRemove);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность 

Сейчас в карту можно вставить существо на координату, выходящую за размер карты.  
Если координата некорректна(находится вне пределов карты), нужно бросать исключение:
```java
public void addEntity(BaseEntity entity, Coordinates coordinates) {
  worldEntities.put(coordinates, entity);
}

public void addEntity(BaseEntity entity, Coordinates coordinates) {
    validate(coordinates);  <-- Если координата вне пределов карты, бросает исключение
  worldEntities.put(coordinates, entity);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Яснее объясняй свои намерения
```java
public int getCountEntities(EntityType entityType) {
  if (worldEntities.get(coordinates).getType().equals(entityType)) {
    result++;
  }
  //...
}

//ПРАВИЛЬНО:
public int getCountEntities(EntityType entityType) {
  Entity current = worldEntities.get(coordinates);
  if (current.getType().equals(entityType)) {
    result++;
  }
  //...
}
```

- В классе не хватает метода "удалить существо по координате".

**6. class Path**

Класс хранит путь.  
Для этого содержит точку начала пути, точку конца и т.д.  
И собственно сам путь в виде очереди:
```java
public class Path {
  private Coordinates startCoordinate;
  private Coordinates endCoordinate;
  public Integer length;
  private Queue<Coordinates> pathQueue = new LinkedList<>();
  //...
}
```

Мне непонятно, какую проблему этот класс решает.  
Если он нужен для процесса поиска BFS, то там достаточно простого класса связного списка, типа такого 
```java
class Node {
  private Coordinates coordinates; 
  private Node previous;
  //...
  public Coordinates getCoordinates() {...}
  public Node getPrevious() {...}
}
```

Для возврата результата поиска лучше всего простой список:
```java
//СЕЙЧАС ТАК:
public class PathFinder {
  public static Patch find(...) {...}
}

//ЛУЧШЕ ТАК:
public class PathFinder {
  public static List<Coordinates> find(...) {...}
}
```

Сейчас класс Path выглядит как общий класс и для процесса поиска пути по алгоритму BFS и возврата этого пути.  
Таким образом, класс `Path` нарушает SRP.

**7. class PathFinder**

- Соблюдай требования утилитных классов. 

Это утилитный класс, он содержит только статические методы.  
Утилитные классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- Нарушение SRP, OCP.

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```java
public static Path find(Coordinates start, WorldMap worldMap) {
  //...  
  if (startCreature.returnType().equals(EntityType.HERBIVORE)) {
    //алгоритм для Зайца
  } else {
    //алгоритм для волка    
  }
}
```

Нарушение SRP: поиск самостоятельно опрашивает существ.

Нарушение OCP: класс открыт для изменений.  
При добавлении нового типа существ в проект, придется изменить код в этом классе:
```java
if (startCreature.returnType().equals(EntityType.HERBIVORE)) {
  //алгоритм для Зайца
} else if (startCreature.returnType().equals(EntityType.PREDATOR)){
  //алгоритм для Волка    
} else if (startCreature.returnType().equals(EntityType.BIRD)){
  //алгоритм для Птицы    
}
```

Все необходимые для поиска данные метод должен принимать в себя:
```java
public List<Coordinates> find(Coordinates start, WorldMap worldMap, Class<? extends Entity> target) {
  //ищет путь на карте worldMap от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```

**8. enum EntityType**

- Дополнительная внешняя типизация

Енам существует для дополнительной внешней типизации классов
```java
public enum EntityType {
  PREDATOR("\uD83E\uDD8A", 0.05, 0.0),
  HERBIVORE("\uD83D\uDC30", 0.1, 0.4),
  GRASS("\uD83C\uDF3F", 0.2, 0.5),
  ROCK("\uD83D\uDDFB", 0.1, 0.0),
  TREE("\uD83C\uDF33", 0.15, 0.0),
  EMPTINESS("\uD83D\uDFEB", null, 0.0);

  private final String value;
  private final Double coefficient;
  private final Double coefficientCrises;
  //...
} 
```
Не используй дополнительную типизацию классов, потому что это дублирует стандартные способы.  
Используй стандартные способы определять принадлежность объектов к типам:
```java
String name = entity.getClass().getSimpleName();

switch (name) {
  case "Herbivore": //...
  case "Predator":  //...
}
```

- Назначение.

Этот енам может использоваться как константа для хранения характеристик существ В ЭТОЙ КОНСОЛЬНОЙ программе.  
Но он не должен храниться в самих существах, потому что это нарушает их SRP. 

**9. abstract class BaseEntity**

- Нарушение ТЗ.

В ТЗ этот класс называется "Entity", ты его назвала "BaseEntity".
Непонятно, что дает такое переименование.

Соблюдай ТЗ, это необходимый навык на работе.  
Нельзя просто брать и нарушать ТЗ без причины.  
Для переименования "Entity" в "BaseEntity" явной причины нет.

- Два метода, которые делают одно и то же
```java
public abstract class BaseEntity {
  public EntityType getType() {
    return returnType();
  }

  public abstract EntityType returnType();
}

//ПРАВИЛЬНО:
public abstract class BaseEntity {
  public abstract EntityType getType();
}
```

- Не нужно использовать дополнительную типизацию
```java
public abstract class BaseEntity {
  public EntityType getType() {
    return returnType();
  }
}  
```
Используй стандартные способы определения принадлежности типов объектов: `instanceof`, `getClass()`.

Хранение в `entities` значений `EntityType` нарушает их SRP.  
Потому что начинают знать много разного, что их не касается: своё изображение, коэффициенты для карты.

- Зависимость модели от представления это плохо.

Изображение существ хранится в `EntityType`.  
А значит, храня в себе `EntityType`, существо в себе хранит своё изображение и тем самым зависит от представления.

Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

Подробнее необходимость отделения модели от представления обоснована в архитектуре MVC(model-view-controller).  
Но необходимость отделения модели от представления появляется не на уровне MVC. Оно появляется уже на уровне SOLID.  
Просто в MVC это требование более явно формализовано.

+ 👍 Вот таким должен быть идеальный Entity и его простые неходящие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**10. abstract class Creature extends BaseEntity**

- Эти данные должны не сетиться, а приниматься в конструктор
```java
public abstract class Creature extends BaseEntity {
  private EntityType target = EntityType.EMPTINESS;
  //...

  public Creature(int speed) {
    this.speed = speed;
  }

  public void setTarget(EntityType target) {
    this.target = target;
  }
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends BaseEntity {
  private final EntityType target;
  //...

  public Creature(int speed, EntityType target) {
    this.speed = speed;
    this.target = target;
  }
}
```

Учитывая, что дополнительная внешняя типизация это плохо, то ещё правильнее будет так:
```java
public abstract class Creature extends BaseEntity {
  private final Class<? extends BaseEntity> target;
  //...

  public Creature(int speed, Class<? extends BaseEntity> target) {
    this.speed = speed;
    this.target = target;    
  }
}
```

- Используй абстрактные методы.

Если метод должен быть обязательно перегружен в наследниках, а на текущем уровне не должен ничего делать,
то ни в коем случае не делай этот метод пустым.  
Вместо этого сделай абстрактный метод
```java
public void makeMove(WorldMap worldMap) {
}

//ПРАВИЛЬНО:
public abstract void makeMove(WorldMap worldMap);
```

- Нарушение инкапсуляции.

Если метод нужен только самому класс и его наследникам, то такой метод должен быть private
```java
public Path getPath() {return path;}

//ПРАВИЛЬНО:
protected Path getPath() {return path;}
```

- Класс должен сам искать свой путь.

Creature должен сам находить свой путь в методе `makeMove()`, а не принимать его извне
```java
public void setPath(Path path) {  <-- Этого метода в классе быть не должно
  this.path = path;
}
```
Иначе получается, что кто-то другой, а не Креатура, принимает решение, куда нужно идти. 

**11. Наследники Creature: классы Creature и Predator**

- Нарушение DRY, нарушение чеклиста ТЗ.

```java
Чеклист для самопроверки #
Дублирование кода между классами Herbivore и Predator
```

Сейчас есть дублирование кода в методе совершения хода `makeMove()` у классов `Зайца` и `Волка`.  
Общий код нужно вынести в общего предка. В данном случае- в `Creature`.

- Сложные правила пользования методом `makeMove(...)`.

Сейчас перед вызовом метода `makeMove(...)` нужно установить в классе путь.  
Если вызвать `makeMove(...)` без предварительной установки пути- вылетит исключение
```java
public class Main1 {
  public static void main(String[] args) {
    WorldMap worldMap = new WorldMap(10, 10);
    Herbivore herbivore = new Herbivore(1, 1);
    Grass grass = new Grass();
    worldMap.addEntity(herbivore, new Coordinates(0, 1));
    worldMap.addEntity(grass, new Coordinates(5, 5));

    herbivore.makeMove(worldMap);
  }
}

//РЕЗУЛЬТАТ:
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "map.Path.getPathQueue()" because the return value of "entity.Herbivore.getPath()" is null
```

Если метод не может работать без каких-то данных, то он эти данные должен в себя принимать при вызове.  
Например:
```java
public void makeMove(WorldMap worldMap) {...}

//ОТНОСИТЕЛЬНО ПРАВИЛЬНО:
public void makeMove(WorldMap worldMap, Path path) {...}
```

НО ПРИНИМАТЬ ПУТЬ В МЕТОД ХОДА ТОЖЕ НЕПРАВИЛЬНО.  
Об этом ниже.

- Существа должны сами получать путь к цели, а не принимать его извне
```java
public void makeMove(WorldMap worldMap) {
  int i = 1;
  Queue<Coordinates> pathCoordinates = this.getPath().getPathQueue();  <-- ЭТОТ ПУТЬ УСТАНАВЛИВАЕТСЯ ДРУГИМ КЛАССОМ
  //...
}

//ПРАВИЛЬНО:
public void makeMove(WorldMap worldMap) {
  int i = 1;
  Coordinates current = worldMap.getCoordinates(this);
  Path path = PathFinder.find(worldMap, current, getFood());
  Queue<Coordinates> pathCoordinates = path.getPathQueue();
  //...
}
```
Существо должно само искать свой путь, а не доверять это кому-то другому.  
Потому что нельзя быть уверенным, что клиент передаст корректный путь.

- Нарушение дизайна ООП. 

При создании экземпляра класса, значения обязательных полей предка должны устанавливаться через конструктор, а не сетиться  
```java
public class Herbivore extends Creature {
  private int Hp;

  public Herbivore(int speed, int hp) {
    super(speed);
    Hp = hp;
    Set<EntityType> availableSet = new HashSet<>();
    availableSet.add(EntityType.EMPTINESS);
    availableSet.add(EntityType.GRASS);
    super.setAvailableEntityTypes(availableSet);
    super.setTarget(EntityType.GRASS);
  }
  //...
}

//ПРАВИЛЬНО:
public class Herbivore extends Creature {
  private final static Set<EntityType> AVAILABLE_TYPES = Set.of(EntityType.EMPTINESS, EntityType.GRASS);
  private final static EntityType TARGET = EntityType.GRASS;

  private int Hp;

  public Herbivore(int speed, int hp) {
    super(speed, AVAILABLE_TYPES, TARGET);
    Hp = hp;
  }
  //...
}
```
*Эккель "Философия Java", гл.5, "Конструктор гарантирует инициализацию"*

**12. final class Actions**

- Идея `Action` здесь не осмыслена. 

Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.  
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.  
Action'ы, изложенные в ТЗ, это вариант реализации паттерна Command. 

Сейчас `Actions` у тебя это просто сборник функций, сделанный в стиле процедурного программирования
```java
public final class Actions {
  public static void topActions(WorldMap worldMap) {...}
  private static WorldMap createEmptyWorld(int sizeN, int sizeM) {...}
  private static void fillOutWorldMap(WorldMap worldMap, EntityType entityType) {...}
  public static void moveAll(WorldMap worldMap) {...}
  public static void moveCoordinates(WorldMap worldMap, int x, int y) {...}
}
```

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.  
В каждом экшене должен быть только один публичный метод(не публичных может быть сколько угодно).  
Это вариация паттерна Command- экшены должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
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

- Экшен совершения хода должен только запускать движение в креатурах, дальше они должны всё сделать сами
Выглядеть это должно примерно так
```java
class MoveAction реализует Action {

  @Override
  public void execute(Карта карта) {
    List<Creature> creatures = getCreatures(карта);
    for(Creature creature: creatures) {
      creature.makeMove(карта);  //даёт пинка чтобы креатура побежала
    }
  }

  private static List<Creature> getCreatures(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

**13. class BaseEntityFactory**

+ 👍 Фабрика существ это хорошо.

- Сразу возвращай результат
```java
BaseEntity entity = null;
switch (entityType) {
  case EntityType.TREE:
    entity = new Tree();
    break;
  case EntityType.GRASS:
    entity = new Grass();
    break;
    //...
    case EntityType.PREDATOR:
      speed = (int) (Math.random() * 150);
      int attackPower = (int) (Math.random() * 230);
      entity = new Predator(speed, attackPower);
      break;
  }
  return entity;
}

//ПРАВИЛЬНО:
return switch (entityType) {
  case TREE -> new Tree();
  case GRASS -> new Grass();
  //...
  case PREDATOR -> createPredator();
  default -> throw new IllegalArgumentException("illegal entityType: " + entityType);
};

//...
private static Predator createPredator() {
  speed = (int) (Math.random() * 150);
  int attackPower = (int) (Math.random() * 230);
  return new Predator(speed, attackPower); 
}
```

**14. final class Movement**

- Повторяющиеся действия делай через циклы
```java
public static Set<Coordinates> getSetPotentialCoordinates(WorldMap worldMap, Coordinates coordinateStart, Coordinates coordinateEntity) {
  //...
  Coordinates potentialCoordinateXMinus = new Coordinates(x - 1, y);
  if (x != 0 && isAvailableMove(potentialCoordinateXMinus, worldMap, coordinateEntity))
      coordinatesSet.add(potentialCoordinateXMinus);

  Coordinates potentialCoordinateYMinus = new Coordinates(x, y - 1);
  if (y != 0 && isAvailableMove(potentialCoordinateYMinus, worldMap, coordinateEntity))
    coordinatesSet.add(potentialCoordinateYMinus);
  //...
}

//ПРАВИЛЬНО:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(
    new Coordinates(1, 0),
    new Coordinates(-1, 0),
    new Coordinates(0, 1),
    new Coordinates(0, -1),
);

public static Set<Coordinates> getSetPotentialCoordinates(WorldMap worldMap, Coordinates coordinateStart, Coordinates coordinateEntity) {
  //...
  for(Coordinates shift : SHIFT_COORDINATES) {
    Coordinates next = new Coordinates(shift.x() + x, shift.y() + y);
    if (x != 0 && isAvailableMove(next, worldMap, coordinateEntity)) {
      neighborCoordinates.add(potentialCoordinateYMinus);
    }
  }
  //...
}
```

**15. class Render**

- Не вижу никаких оснований ограничивать возможность наследование этого класса
```java
public final class Render {...}

//ПРАВИЛЬНО:
public class Render {...}
```

Делать final классы нужно тогда, когда есть понятная необходимость для запрета наследования.  
Например, когда делаешь утилитные или константные классы- от них нельзя наследоваться.

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < sizeM; i++) {
  for (int j = 0; j < sizeN; j++) {
    Coordinates coordinates = new Coordinates(i, j);
    //...
  }
}

//ЛУЧШЕ:
for (int y = y; i < sizeM; y++) {
  for (int x = 0; x < sizeN; x++) {
    Coordinates coordinates = new Coordinates(x, y);
    //...
  }
}
```

**16. class Simulation**, содержит точку входа main

- Не забывай про инкапсуляцию, всегда явно пиши область видимости
```java
public class Simulation {
  static WorldMap worldMap;
  static Render render;
  //...
}

//ПРАВИЛЬНО:
public class Simulation {
  private static WorldMap worldMap;
  private static Render render;
  //...
}
```

- Точка входа `main()` должна находиться в отдельном классе.

Конструирование системы и ее использования- это две разных задачи.  
Класс с точкой входа `main` должен только собирать зависимости и запускать систему.  
Выглядеть это в простейшем виде без использования потоков должно примерно так:
```java
public class FirstMain {
  public static void main(String[] args) {
    WorldMap worldMap = new WorldMap(10, 10);
    Simulation simulation = new Simulation(worldMap);
    simulation.start();
  }
}
```

В твоей случае, если нужно вводить от юзера размеры карты, то *без использования потоков* это будет выглядеть примерно так:
```java
public class SecondMain {
  public static void main(String[] args) {
    int width = inputWidth();
    int hight = inputHight();

    WorldMap worldMap = new WorldMap(width, hight);
    Simulation simulation = new Simulation(worldMap);
    simulation.start();
  }

  private static int inputWidth() {
    //получает от юзера ширину карты
  }
  //...
}
```

Получать от юзера ширину и высоту карты нужно в таком Man-классе, в не в Simulation.  
Потому что эти данные нужны для процесса конструирования системы, а не для ее использования в Simulation.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Нарушение ТЗ, нет указанных для этого класса методов 
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```
В этом классе должны быть публичные методы `startSimulation()`,`pauseSimulation()`,`nextTurn()`.  
Тот, класс, который будет реализовывать многопоточность, должен будет принимать от юзера команды "старт"/"стоп" и дёргать за эти методы.

- Божественный метод `main()`.

Большой метод, который делает разное и поэтому плохо читается.  
Нормальный размер для методов: до 20-30 строк.

Здесь 100+ строк. Столько строк в методе может быть только в одном случае- когда он нарушает правило одной операции на одном уровне абстракции. 

Метод нужно разделить на несколько, каждый из которых должен делать только одно дело.  
Например, смотрим на код и видим отдельные смысловые блоки кода 
```java
public static void main(String[] args) {
  //много кода до 
  do {
    System.out.println("Введите координату по горизонтали:");
    x = Integer.parseInt(inputScanner.nextLine());
  } while (x < 1 || x > worldMap.getSizeM());

  do {
    System.out.println("Введите координату по вертикали:");
    y = Integer.parseInt(inputScanner.nextLine());
  } while (y < 1 || y > worldMap.getSizeN());
  //много кода после
}
```

Выносим эти блоки в отдельные методы согласно правила одной операции *на одном уровне абстракции*:
 ```java
public static void main(String[] args) {
  //много кода до 
  int width = int inputWidth(...)
  int hight = int inputHight(...)
  //много кода после
}

private static int inputWidth(int min, int max) {
    return inputNumber("Введите координату по горизонтали:", WIDTH_MIN, WIDTH_MAX);
}

private static int inputNumber(String title, int min, int max) {
  Scanner scanner = new Scanner(System.in);
  while(true) {
    System.out.println(title);
    String s = scanner.nextLine();
    if(isNumber(s)) {
      int num = Integer.parseInt(s);
      if(num >= min && num <= max) {
        return num;
      }
    }
  }
}

private static boolean isNumber(String s) {
  try{
    Integer.parseInt(s);
    return true;
  } catch (...) {
    return false;
  } 
} 
```
*"ЧК", гл.3, "Правило одной операции", "Один уровень абстракции"*  
*Фаулер "Рефакторинг", гл.6, "Выделение метода"*

- Вообще никогда не используй инструкцию `goto` - в яве это совсем плохо
```java
loop:
while (true) {
  switch (choseNextStep) {
    case END_SIMULATION:
      break loop;  <-- goto
    //...
  }
}  
```
Если какую-то задачу можно решить только через goto, это значит лишь одно- этот код нужно разделить на вспомогательные методы.  
В данном случае, если разделить божественный метод `main()` на отдельные методы по правилу одной операции, необходимость в `goto` отпадет. 

- Магические числа:
```java
if ((counterStep % 5) == 0) {...}
{...} while (x < 1 || x > worldMap.getSizeM());
```
Вводи константы.

## АРХИТЕКТУРА

**Не видно понимания идей объектно-ориентированной архитектуры.**

В ООП программе нужно выявить информационные сущности и сделать из них классы.  
Такие классы должны соответствовать требованиям SOLID и GRASP.

**Один из примеров нарушения SOLID в части SRP: класс `Simulation`.  **

Он совмещает в себе создание и использование системы.  
Практическое следствие- нельзя создать несколько разных Main-классов для создания и запуска разных конфигураций программы.

Одно из главных преимуществ ООП- при правильном распределении ответственностей между классами, классы становится довольно универсальными и их можно собирать в разных конфигурациях.

Например, вот разные конфигурации.  
Одна сразу запускает Симуляцию с параметрами по умолчанию, а другая- через ввод от юзера размеров карты
```java
public class FirstMain {
  public static void main(String[] args) {
    WorldMap worldMap = new WorldMap(10, 10);
    Simulation simulation = new Simulation(worldMap);
    simulation.start();
  }
}

public class SecondMain {
  public static void main(String[] args) {
    int width = inputWidth();
    int hight = inputHight();

    WorldMap worldMap = new WorldMap(width, hight);
    Simulation simulation = new Simulation(worldMap);
    simulation.start();
  }

  private static int inputWidth() {
    //получает от юзера ширину карты
  }
  //...
}
```

**Зависимость от представления, как нарушение SRP модели.**

Для понимания того, почему модель не должна зависеть от представления, представь, что у нас может быть две игровые конфигурации.  
Одна должна рисовать карту в виде эмодзи, как сейчас.  
А вторая например в виде букв ('t' - Tree, 'P' - Predator etc).

Тогда станет ясно, что изображение существа характеризует не само существо(сила, скорость etc.), а то, как ОНО БУДЕТ ПОКАЗАНО.  
А показано оно может быть очень по-разному: через эмодзи, буквы или цветные квадратики.  

Здесь сила и скорость это характеристики существа, которые не зависят от представления.  
А изображение существа это его ПРЕДСТАВЛЕНИЕ.

Поэтому разные конфигурации программы в этом случае выглядели бы так:
```java
public class EmojiMain {
  public static void main(String[] args) {
    WorldMap worldMap = new WorldMap(10, 10);
    Renderer renderer = new EmojiRenderer();

    Simulation simulation = new Simulation(worldMap, renderer);
    simulation.start();
  }
}

public class TextMain {
  public static void main(String[] args) {
    WorldMap worldMap = new WorldMap(10, 10);
    Renderer renderer = new TextRenderer();

    Simulation simulation = new Simulation(worldMap, renderer);
    simulation.start();
  }
}

public abstract class Renderer {
  public void render(Карта карта) {
    //...
    String sprite = toSprite(entity);
    System.out.print(sprite);
    //...
  }    
  
  protected abstract String toSprite(Entity entity);
}  

public class TextRenderer extends Renderer {
  private static final String TREE = "t";
  private static final String HERBIVORE = "H";
  //...
  
  @Переопределить
  protected String toSprite(Entity entity) {
    String name = entity.getClass().getSimpleName();
    return switch(name) {
      case "Tree" -> TREE;
      case "Herbivore" -> HERBIVORE;
      //...
    }
  }
}  
```

И дело здесь не в том, планируешь ли ты делать в программе возможность печатать карты разными картинками.  
Дело в самом принципе- нужно понять, что дает независимость модели от представления и зачем оно придумано.

**Нет многопоточности.**

Одна из фишек этого проекта- управление ходом программы через старт/стоп во время работы программы.  
Для этого нужно ввести многопоточность.

Но многопоточность в этом проекте, на мой взгляд не важна не сама по себе, хотя она важна и сама по себе.  
Но важнее то, как её удастся вписать в архитектуру проекта.

А вписать многопоточность нужно так, чтобы класс `Simulation` можно было запускать и с многопоточностю и без.  

То есть, многопоточность должна быть вынесена в отдельный класс.  
Задача такого класса- ожидать команды от юзера и когда они придут, дергать `Simulation` за его публичные методы, которые описаны в ТЗ
```
Simulation #
Главный класс приложения, включает в себя:

nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

Этот класс должен содержать в себе экземпляр `Simulation` в виде поля.  
Примерно так
```java
public class EmojiMain {
  public static void main(String[] args) {
    WorldMap worldMap = new WorldMap(10, 10);
    Renderer renderer = new EmojiRenderer();

    Simulation simulation = new Simulation(worldMap, renderer);
    SimulationManager manager = new SimulationManager(simulation);  
    manager.execute();
  }
}
```

**Совет:**  
Для тренировки и понимания некоторых ООП концепций, создай несколько майн-классов.  
Один пусть запускает игру так, как это сейчас- в одном потоке с конфигурированием карты через диалог с юзером.  
Второй майн пусть сразу запускает в одном потоке игру сразу с картой размером 10x10.  
Третий пусть запускает игру с возможность управления через старт/стоп во время работы программы. То есть, с потоками.

Если захочешь, сюда же- разные рендереры.

## ВЫВОД

Доделать программу согласно ТЗ: сделать паузу/пуск во время работы программы.  
Это важно для отработки азов работы с многопоточностью.

Подробнее разберись с Action'ами, посмотри ролики на ютубе про паттерн "Command", чтобы получить о нем общее представление.

Не видно и понимания декомпозиции- как делить программу на классы.  

Видимо, твой рабочий опыт более процедурный.  
Это действительно позволяет тебе успешно решать алгоритмические проблемы.  
Но решаешь ты их привычным образом- через приёмы процедурного программирования.  
ООП подход принципиально отличается от процедурного, нужно развивать объектно-ориентированное мышление.

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. 

Сравни различия между ООП и процедурным стилями программирования при решении одной и той же задачи:  
Стрим Сергея [Крестики-нолики в процедурном стиле](https://www.youtube.com/watch?v=PPikj1qHxrA)  
Мой стрим [Крестики-нолики в ООП стиле](https://t.me/zhukovsd_it_chat/53243/187097)

Посмотри на ютубе ролики Немчинского про SOLID. У него есть ещё хорошие лекции про GRASP.

n.158(336)  
#ревью #симуляция 