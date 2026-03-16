https://github.com/VeraAtnagullova15/Sumulation_Stranger_Things  
[VeraAtnagirl]

Есть много над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim143/img0.png)  
В данном случае, проблема из-за точке- точки и эмодзи имеют разную ширирну.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализованы пауза/пуск во время работы
+ 👍 Механика голода у существ
+ 👍 Цветовая индикация здоровья существ на карте
+ 👍 Возможность создавать карты разного размера через меню
+ 👍 Много разных существ
+ 👍 Расширенный функционал- телепорты

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов нужно писать стилем all lowercase 
```java
Dialogs

//ПРАВИЛЬНО:
dialogs
```

- Константы должны быть final static
```java
protected final int POWER_INFLUENCE_SPORE_PATCH = 20;

//ПРАВИЛЬНО:
protected static final int POWER_INFLUENCE_SPORE_PATCH = 20;
```

- Связка "To" используется для обозначения пребразований из чего-то куда-то: `toMap()`, `toLowerCase`, `toHexString()`.  
Здесь не преобразуется одно в другое, здесь одно хранится вместе с другим
```java
HashMap<Coordinates, Entity> coordinatesToEntities = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Entity> coordinatesWithEntities = new HashMap<>();
```

- Не используй синонимы, для одного и того же явления используй одно и то же название
```java
Coordinates getPosition(Entity entity)  <-- Coordinates называется Position
Entity getEntity(Coordinates coordinates) <-- Coordinates называется coordinates

//ПРАВИЛЬНО:
Coordinates getCoordinates(Entity entity)
Entity getEntity(Coordinates coordinates) 
```

- Метод помещает одно существо в карту, а не несколько. Поэтому не должен называться во множественном числе
```java
void setEntities(Coordinates coordinates, Entity entity)

//ПРАВИЛЬНО:
void setEntitiy(Coordinates coordinates, Entity entity)
```

- Сеттер должен устанавливать какое-то конкретное поле.  

Этот метод не устанавливает конкретное существо, а добавляет существо в группу других существ:
```java
void setEntities(Coordinates coordinates, Entity entity)

//ПРАВИЛЬНО:
void putEntity(Coordinates coordinates, Entity entity) //или addEntity(...)
```

Когда есть какое-то одно существо, то сеттер корректен. Например:
```java
private Entity entity;

void setEntity(Entity entity) {  <-- Правильно, сеттер устанавливает конкретное значение
  this.entity = entity;  
}
```
Когда существо добавляется к другим, это не сеттер. Например:
```java
private List<Entity> entities;

void addEntity(Entity entity) {  <-- Это не сеттер, потому что добавляет новый объект в список других объектов
  entities.add(entity);  
}
```

- Не делай название класса только большими буквами, потому что это выглядит как константа:
```java
class BFS

//ПРАВИЛЬНО:
class Bfs
```

- Если метод называется "проверить"(check), то он должен что-то проверять, а результат проверки возвращать в виде boolean.  
Если метод void, значить он ничего не проверяет, а поэтому не может называться "check"
```java
void checkCellEffects(WorldMap world, Coordinates nextStep)
```

- Избыточно. И так понятно, что в классе редерера карты, метод печати печатает именно карту, а не что-то иное 
```java
RendererWorldMap {
  protected void printMap(WorldMap map) {...}
}

//ПРАВИЛЬНО:
RendererWorldMap {
  protected void print(WorldMap map) {...}
}
```

- Либо "принтер" и "принт", либо "редерер" и "рендер"
```java
RendererWorldMap {
  protected void printMap(WorldMap map) {...}
}

//ПРАВИЛЬНО:
RendererWorldMap {
  protected void render(WorldMap map) {...}
}
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```java
protected HashMap<Coordinates, Entity> coordinatesToEntities = new HashMap<>();
ArrayList<Creature> creatures = new ArrayList<>();

//ПРАВИЛЬНО:
protected Map<Coordinates, Entity> coordinatesToEntities = new HashMap<>();
List<Creature> creatures = new ArrayList<>();
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**3. Нарушение конвенции кода**

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки.  
Исключение- метод `equals()`, там можно после if не выделять блоки скобочками
```java
if (isPlaceEmpty(coordinates)) return coordinates;

if (entity instanceof Inhabitant) countInhabitant++;
else if (entity instanceof Soldier) countSoldier++;

//ПРАВИЛЬНО:
if (isPlaceEmpty(coordinates)) {
  return coordinates;
}

if (entity instanceof Inhabitant) {
  countInhabitant++;
} else if (entity instanceof Soldier) {
  countSoldier++;
}
```
*"Oracle Java code conventions"*  

**4. Разбиение классов по пакетам**

Часть классов здесь разнесены по пакетам.  
Другая часть классов лежит в корне проекта- 10 штук.

Чтобы выглядело гормонично, в корне можно оставить только `Main`, а остальные тоже разнести по пакетам.  
Можно каждый класс вынести в персональный пакет, например класс `BFS` в пакет "pathfinder".  
Или, если не хочется делать много пакетов, можно создать пакет "common" и туда перенести все эти классы-одиночки.

**5. record Coordinates (int row, int column)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**6. class WorldMap**

- Две коллекции- избыточно.

Две коллекции для хранения записей "существо-координата" и "координата-существо" это избыточно и делает код сложнее.  
Теперь при всех операциях с существом, нужно вносить изменения в два этих списка и следить за тем, чтобы информация в них была синхронизирована между собой.  
Для всех операций с существами в карте достаточно иметь только один список
```java
protected HashMap<Coordinates, Entity> coordinatesToEntities = new HashMap<>();  <-- ЭТОГО ДОСТАТОЧНО
protected HashMap<Entity, Coordinates> entitiesToCoordinates = new HashMap<>();  <-- ЭТО ЛИШНЕЕ 
```
Галичие двух синхорнизированных списков(прямой и обратный) могут сделать выполнение некоторых операций быстрее.  
Например, получение координаты существа:
```java
public Coordinates getPosition(Entity entity) {
  return entitiesToCoordinates.get(entity);  <-- КООРДИНАТА ВОЗВРАЩАЕСЯ МАКСИМАЛЬНО БЫСТРО
}
```
Но за это приходится заплатить более громоздким алгоритмом? дублированием хранимой информации, необходимостью дублировать данные в двух коллекциях.  

При этом на *реальном быстродействии* такая оптимизация никак не скажется- 
подавляющую часть времени программа будет пребывать в паузе. 
А выигрыш в 0.000000001 секунды при выполнении некоторых операций тоже невозможно заметить человеческим глазом.

**Преждевременная оптимизация это Антипаттерн.**

Оптимизировать алгоритмы для увеличения их быстродействия за счет ухудшения архитектуры нужно тогда, когда в этом есть реальная необходимость.
В противном случае, отдавай приоритет простоте алгоритма.

В данном случае, при использовании только одной коллекции, координата должна возвращаться так:
```java
public Coordinates getCoordinates(Entity entity) {
  for (var entry : coordinatesToEntities.entrySet()) {
    Entity current = entry.getValue();
    if (current.equals(entity)) {
      return entry.getKey();
    }
  }
  //бросить исключение, если энтити не найден
}
```

- Нарушение конвенции кода.

Конструктор должен стоять первым среди методов, но после всех полей.

- Нарушение инкапсуляции.

Поля классов могуть быть `protected` только в том случае, если к ним обращаются потомки этого класса.  
Тут к этим полям не обращаются классы-потомки `WorldMap`, потому что этих потомков нет
```java
protected HashMap<Coordinates, Entity> coordinatesToEntities = new HashMap<>();
protected HashMap<Entity, Coordinates> entitiesToCoordinates = new HashMap<>();

//ПАРВИЛЬНО:
private final HashMap<Coordinates, Entity> coordinatesToEntities = new HashMap<>();
private final HashMap<Entity, Coordinates> entitiesToCoordinates = new HashMap<>();
```
Так же, делай поля final, если во время работы эти поля не пересоздаются.

- Не нужно в карте держать сеттеры размеров "на всякий случай".  
Для этого просто нет необходимости- карта создается один раз и на протяжении работы программы, ее размеры не меняются
```java
protected void setHeight(int height) {  <-- НЕ ИСПОЛЬЗУЕТСЯ, УБРАТЬ
  this.height = height;
}
```
Все методы в классе должны быть оправданы.  
Не должно быть методов "на всякий случай", которые не используются. 
А эти сеттеры в программе не используются.

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```java
private HashMap<Coordinates, Entity> coordinatesToEntities = new HashMap<>();

public HashMap<Coordinates, Entity> getCoordinatesToEntities() {
  return coordinatesToEntities;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```java
Карта карта = new Карта();
<заселить карту существами>
карта.getCoordinatesToEntities().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

Тут проблема еще в том, что обе коллекции существ, прямая и обратная, возвращаются через геттеры.  
Поэтому клиент может внести изменения в одну колекцию, не синхронизировав данные в другой.  
Например, так:
```java
Map<Coordinates, Entity> entitiesFromWorldMap = карта.getCoordinatesToEntities();

//Клиент напрямую добавляет данные в одну коллекцию (координата-существо),
//но не синхронизирует эти изменения с обратной коллекцией(существо-координата)
entitiesFromWorldMap.put(new Coordinates(99, 99), new Заяц());
```

Есть два варианта, как сделать правильно.  
Либо вернуть не оригинал мапы, а ее копию: 
```java
public Map<Coordinates, Entity> getCoordinatesToEntities() {
  return new HashMap(coordinatesToEntities);
}
```

Либо возвращать существ персонально:
```java
public Entity get(Coordinates coordinates)
public Coordinates getCoordinates(Entity entity)
```
Лично мне больше нравится второй вариант, когда существ возвращают персонально.

- Никогда не возвращай null
```java
private HashMap<Coordinates, Entity> coordinatesToEntities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return coordinatesToEntities.get(coordinates);  <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  найти случайную пустую координату.

Наверное, для проекта в целом полезно иметь метод, который находит случайную пустую координату.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно искать случайную пустую координату.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  

Если координата некорректна(находится вне пределов карты), нужно бросать исключение.  
Потому что сейчас можно вставить существо на координату, которая находится за пределами карты
```java
public void setEntities(Coordinates coordinates, Entity entity) {
  coordinatesToEntities.put(coordinates, entity);
  entitiesToCoordinates.put(entity, coordinates);
}
```

Если спросить, свободна ли ячейка с координатой (+10055, -100500), то карта ответит, что свободна.  
Хотя правильный ответ- такой координаты нет вообще.  
То есть, нужно кинуть исключение
```java
public boolean isPlaceEmpty(Coordinates coordinates) {
  return !coordinatesToEntities.containsKey(coordinates);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Метод совершения хода в карте- нарушение SRP.  
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
Карта карта = new Карта(100, 100);
Заяц заяц = new Заяц();
карта.setEntities(new Координата(0, 0), заяц);
карта.moveEntity(заяц, new Координата (99, 99));

/* class Карта */
public void moveEntity(Entity entity, Coordinates newCoordinates) {
  //7 строк логики совершения хода
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как?  
Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

+ 👍 В целом, с точки зрения SRP класс относительно неплохой.

Имеющиеся нарушения SRP в нем не особо существенны.  
Основная претензия к классу относится не к тому, какой функционал он реализует.  
А к тому, как он его реализует.

**7. class BFS**

- Этот класс не ищет путь.

Путь это последовательность точек от точки старта до точки финиша.  
Этот класс не ищет путь. Его единственный публичный метод возвращает одиночную координату
```java
public Coordinates findPath(WorldMap world, MoveBehavior moveBehavior, Coordinates start, Coordinates finish)
```
Как с помощью этого класса искать путь, совершенно неясно.

- Поиск не соответствует алгоритму BFS.

Сейчас метод поиска принимает в себя в том числе и координату конечной точки поиска
```java
public Coordinates findPath(... , Coordinates finish)
```
А значит, кто-то другой выполняет часть алгоритма BFS и находит координату с едой.  
Путь между двумя точками ищет алгоритм A*.  

BFS должен искать путь от точки старта до точки, соответствующей заданным условиям.  
В данном случае- до точки, в которой находится существо нужного класса(tlf):
```java
public Coordinates findPath(WorldMap world, MoveBehavior moveBehavior, Coordinates start, Coordinates finish)

//ПРАВИЛЬНО:
public List<Coordinates> find(World world, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```

- Мапа координата-координата при реализации поиска обычно используется как эрзац-заменитель связного списка
```java
Map<Coordinates, Coordinates> path = new HashMap<>();
```
Вместо мапы координата-координата сделай класс связного списка, алгоритм тогда будет выглядеть понятнее.
Например, так:
```java
class Node {
  private Coordinates coordinates; 
  private Node previous;
  //...
  public Coordinates getCoordinates() {...}
  public Node getPrevious() {...}
}
```

- Сложные условия, которые трудно понять.  
Вводи поясняющие переменные или вспомогательные методы, которые своим названием будут объяснять, что тут происходит:
```java
if (world.isPlaceInside(upCell) && moveBehavior.canPass(world, upCell, finish) && !path.containsKey(upCell)) {...}

//ПРАВИЛЬНО, ПОЯСНЯЮЩАЯ ПЕРЕМЕННАЯ:
boolean isНазваниеВсеОбъясняет = world.isPlaceInside(upCell) && moveBehavior.canPass(world, upCell, finish) && !path.containsKey(upCell);
if (isНазваниеВсеОбъясняет) {...} 

//ПРАВИЛЬНО, ВСПОМОГАТЕЛЬНЫЙ МЕТОД:
if (isНазваниеВсеОбъясняет(...)) {...} 
```

- Нарушение DRY, дублирование кода. Повторяющиеся действия выполняй через цикл
```java
Coordinates upCell = new Coordinates(current.row() - 1, current.column());
Coordinates lowCell = new Coordinates(current.row() + 1, current.column());
//еще два таких же

if (world.isPlaceInside(upCell) && moveBehavior.canPass(world, upCell, finish) && !path.containsKey(upCell)) {
   neighbors.add(upCell);
   path.put(upCell, current);
}
if (world.isPlaceInside(lowCell) && moveBehavior.canPass(world, lowCell, finish) && !path.containsKey(lowCell)) {
   neighbors.add(lowCell);
   path.put(lowCell, current);
}
//еще два таких же

//ПРАВИЛЬНО:
private final static List<Coordinates> SHIFT_COORDINATES = List.of(new Coordinates(-1, 0), new Coordinates(1, 0), ...);

for(Coordinates shift : SHIFT_COORDINATES) {
  Coordinates coordinates = add(current, shift);
  boolean isНазваниеВсеОбъясняет = world.isPlaceInside(coordinates) && moveBehavior.canPass(world, coordinates, finish) && !path.containsKey(coordinates);
  if (isНазваниеВсеОбъясняет) {
    neighbors.add(coordinates);
    path.put(coordinates, current);
  }
}

private Coordinates add(Coordinates first, Coordinates second) {
  return new Coordinates(firs.row() + second.row(), firs.column() + second.column());
}
```

- Никогда не возвращай null 
```java
public Coordinates findPath(...) {
  //...
  return null;
}
```
Впрочем, когда метод пути будет возвращать именно путь, тогда можно будет просто возвращать пустой список:
```java
public List<Coordinates> findPath(...) {
  List<Coordinates> path = new ArrayList<>();
  //заполняет путь координатами
  return path;  <-- Если не найдет пути, то просто вернет пустой список
}
```

**7. Пакет entities**

Большое семество существ- 12 классов. В ТЗ заявлено 5 существ.  
Из-за неправильного распределения полей в иерархии классов, разобраться во всех этих классах тяжело.

**8. class Entity**

Нарушение дизайна ООП в наследовании.  
Не все наследники `Entity` используют эти поля 
```java
public abstract class Entity {
  protected int powerSatiety;
  protected int powerHealing;
}
```
Например, наследник `Entity`, класс `MapObject` не использует поля `powerSatiety` и `powerHealing`.

В предке должны быть только те поля, которые являются общими для всех его наследников.  
Если какое-то поле не является общим для всех наследников, то это поле должно находиться в том наследнике, где оно используется.

Вот идеальный базовый класс Entity для этого проекта:
```java
public abstract class Entity {
  //да, тут совсем пусто
}
```

**9. Инициализация полей в иерархии классов entity**

- Нарушение дизайна ООП в иерархии entity, неправильная инициализация полей в предках

Во многих классах энтити непраильно инициализируются поля в предках.  
При создании экземпляра класса, значения обязательных полей предка должны устанавливаться через конструктор, а не сетиться  
```java
public abstract class Entity {
  protected int powerSatiety;
  protected int powerHealing;
}

public final class Inhabitant extends Survivor {
  //...
  public Inhabitant() {
    super(RationBox.class, null, new WalkMove());
    powerSatiety = POWER_SATIETY_SURVIVOR;
    powerHealing = POWER_HEALING;
  }
}

//ПРАВИЛЬНО:
public final class Inhabitant extends Survivor {
  //...
  public Inhabitant() {
    super(POWER_SATIETY_SURVIVOR, POWER_HEALING, RationBox.class, null, new WalkMove());
  }
}
```
Да, если так сделать во всех классах, то вдруг окажется, что инициализация предков кое-где будет выглядеть примерно так:
```java
super(первоеПоле, второеПоле, третьеПоле, четвертоеПоле, пятоеПоле, шестоеПоле, ..., десятоеПоле);
```
Но это свидетельствует только о том, что поля неправильно распределены по иерархии классов.  
*Эккель "Философия Java", гл.5, "Конструктор гарантирует инициализацию"*

- Константы из ниоткуда.

Откуда взяты эти константы- из предка?  
Они берутся из предка чтобы установить поля в предке же.  
Так сделано во многих классах, это тут системное явление.
Выглядит это довольно странно
```java
public final class Inhabitant extends Survivor {
  public Inhabitant() {
    super(RationBox.class, null, new WalkMove());
    speed = SPEED_SURVIVOR;
    health = HEALTH_SURVIVOR;
    //...
  }
}

//ПРАВИЛЬНО:
public final class Inhabitant extends Survivor {
  private final static SPEED_SURVIVOR = 2;
  private final static HEALTH_SURVIVOR = 3;

  public Inhabitant() {
    super(RationBox.class, null, new WalkMove());
    speed = SPEED_SURVIVOR;
    health = HEALTH_SURVIVOR;
    //...
  }
}
```

- Передача null в аргументы методов, тем более в конструктор- недопустима.  
Передавать null в методы это еще хуже, чем возвращать null из методов
```java
super(RationBox.class, null, new WalkMove());
```

**10. abstract class Creature extends Entity**

- Много магических цифр
```java
this.hunger += 5;
if (this.hunger >= 100) {
  this.health -= 5;
  if (this.health <= 0) {
    world.removeEntity(this);
    return;
  }
}
if (this.health > 100) this.health = 100;
```
Одна из проблем магических цифр состоит в том, что мы не можем быть уверены, что одна и та же цифра в разных участках кода означает одно и то же.  

Эти цифры значат одно и то же?  
Если меняешь "100" в одном месте, нужно ли миенять "100" в другом месте?
```java
if (this.hunger >= 100) {...}
if (this.health > 100) {...}
```

Они могут сослаться на одну и ту же константу?
```java
if (this.hunger >= MAX_HEALTH) {...}
if (this.health > MAX_HEALTH) {...}
```

Или они означают разные концепции, которые могут быть разными числами?
```java
if (this.hunger >= MAX_HUNGER) {...}
if (this.health > MAX_HEALTH) {...}
```
Без констант никогда не можешь быть в этом уверен, особенно, если разбираешься в чужом коде.

- Нарушение SRP.

Вот мы и нашли того, кто перетянул часть одеяла BFS на себя.
`Creature` забрал часть ответственности поиска пути BFS- ищет координату с едой
```java
Coordinates findTarget(WorldMap world, Class<? extends Entity> targetType) {
  //15 строк поиска координаты с едой
}
```

На самом деле искать координату с едой должен сам класс Bfs, а креатура должна только использовать его.  
Примерно так:
```java
private fonal Class<? extends Entity> food;

public void makeMove(WorldMap world, Bfs bfs) {
  Coordinates current = world.getCoordinates(this);
  List<Coordinates> path = bfs.find(world, current, food);
  //пройти по пути path и съесть еду
}
```

- Нарушение OCP. 

Один наследник Creature может есть и атаковать.  
Другой наследник может есть, но не может атаковать.

При этом методы `eat()` и `attack()` находятся в Creature.  
Из-за этого все наследники наследуют методы `eat()` и `attack()`, даже когда эти методы им не нужны
```java
//Класс содержит методы атаки и еды
public abstract class Creature extends Entity {
  protected Class<? extends Entity> targetTypeForEat;
  protected Class<? extends Creature> targetTypeForAttack;
  //...
    public Creature(Class<? extends Entity> targetTypeForEat,
                    Class<? extends Creature> targetTypeForAttack, 
                    MoveBehavior moveBehavior) {...}
}

//Этот наследник умеет есть и атаковать
public final class Soldier extends Survivor {
  //...
  public Soldier() {
    super(RationBox.class, <-- targetTypeForEat
          Demodog.class, <-- targetTypeForAttack
          new WalkMove());
  }
}

//Этот наследник умеет есть, но не умеет атаковать
public final class Inhabitant extends Survivor {
  //...
  public Inhabitant() {
    super(RationBox.class, <-- targetTypeForEat
          null, <-- targetTypeForAttack
          new WalkMove());
  }
}
```
Таким образом, Creature знает про специфические методы поведения *отдельных* своих предков.  
И реализует эти отличия в своем коде
```java
public void makeMove(WorldMap world, BFS bfs) {
  //...
  if (targetTypeForEat != null) {  <-- Поведение одного класса потомков
    targetEat = findTarget(world, targetTypeForEat);
  }
  if (targetTypeForAttack != null) {  <-- Поведение другого класса потомков
    targetAttack = findTarget(world, targetTypeForAttack);
  }
  //...
}
```
Фактически, это псевдополиморфизм- метод `makeMove(...)` работает по разным алгоритмам в зависимости от того, 
в интересах *какого класса потомка* он работает.

Вот этот метод тоже работает на реализацию этого псевдополиморфизма: 
```java
private Coordinates chooseTarget(Coordinates current, Coordinates targetEat, Coordinates targetAttack) 
```

Любой класс должен знать только ту информация, которая касается только его самого и всех своих потомков.  
Если у одного из потомков есть специфический функционал, который не является общим для всех потомков, 
то такой функционал должен находиться только в этом потомке. 

В данном случае, метод `attack()` должен находиться не в `Creature`, а в том потомке, который атакует.
Метод `makeMove(...)` в `Creature` должен содержать только ОБЩИЙ ДЛЯ ВСЕХ потомков код совершения хода.

Если какой-то наследник, например `Soldier`, ходит как-то специфически(атакует), 
то эти отличия должны находиться в его переопределенном методе `makeMove(...)`

- Большой метод, делает разное
```java
public void makeMove(WorldMap world, BFS bfs)
```

В больших методах ищи обособленные по смыслу блоки кода и выноси их во вспомогательные методы.  
Это не только правильно с точки зрения правила одной операции. Но и делает код более читабельным.  
Например:
```java
public void makeMove(WorldMap world, BFS bfs) {
  this.hunger += 5;
  if (this.hunger >= 100) {
    this.health -= 5;
    if (this.health <= 0) {
      world.removeEntity(this);
      return;
    }
  }
  //остальной код
}

//ПРАВИЛЬНО:
public void makeMove(WorldMap world, BFS bfs) {
  starve();
  if(isDead()) {
    world.removeEntity(this);
    return;
  }
  //остальной код
}

private void starve() {  //голодать
  this.hunger += 5;
  if (this.hunger >= 100) {
    this.health -= 5;
    if (this.health <= 0) {
      this.health = 0;
    }
  }
}

public boolean isDead() {
  return this.health == 0;
}
```
*"ЧК", гл.3, "Правило одной операции", "Один уровень абстракции на функцию"*

- Нарушение SRP.

Метод, который реализует некие эффекты
```java
protected abstract void checkCellEffects(WorldMap world, Coordinates nextStep);
```
Например, один из наследников проверяет, не наступил ли он на телепорт.  
Если наступил, то он себя переносит в случайную точку на карте.  

Да, по функционалу это прикольно, почти как в игре "Heroes of Might and Magic III".  
Но это действие относится к ответственности общей игровой логики, а не к ответственности самого существа.

Не само существо должно себя телепортировать, как барон Мюнгхаузен сам себя переносил за волосы.  
Кто-то другой должен телепортировать существо при наступании на Телепорт.  
Например это должен делать... сам Телепорт.

Я могу предложить как это сделать нормально с точки зрения архитектуры.  
Но считаю, что тебе этим вообще не стоит заморачиваться, это будет просто растрачивание твоих учебных ресурсов на ерунду.  
Лучше просто убрать телепорт.

**11. Остальные наследники Entity**

Всех остальных наследников Entity персонально рассматривать не буду- не вижу в этом смысла.  
Их недостатки являются прежде всего следствием а неправильной иерархии entity и тех архитектурных принципов, на которых она построена.  
Но об этом поговорим в конце.

**12. Пакет action**

- Нарушение DRY, дублирование кода.

Многие классы-экшены делают одно и то же- помещают в карту существ определенного типа.   
Код в этих классах дублируется 
```java
DemobatSpawn  
DemodogSpawn  
GateSpawn  
InhabitantSpawn  
RationBoxSpawn  
SoldierSpawn  
SporePatchSpawn
```
Это нарушение чеклиста ТЗ:
```java
Проблемы и ошибки в коде:
Иерархия классов Action’s - Дублирование кода
```

Если нужен отдельный экземпляр спавнера на каждый экшен, то нужно сделать один универсальный класс. 
Он будет принимать в конструктор количество создаваемых существ и способ их создания.  

Способ создания можно передавать через стандартный интерфейс Supplier
```java
public class SpawnAction extends Action {
  private final Supplier<Entity> entitySupplier;
  private final int count;

  public SpawnAction(Supplier<Entity> entitySupplier, int count) {...}

  @Override
  public void execute(Карта карта) {
    for (int i = 0; i < count; i++) {
      Entity entity = entitySupplier.get();
      //поместить существо в карту на случайную координату
    }
  }
}
```
Использование:
```java
List<Action> initActions = List.of(new SpawnAction(() -> new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ),
                                   new SpawnAction(() -> new Волк(), КОЛИЧЕСТВО_ВОЛКОВ),
                                   new SpawnAction(() -> new Трава(), КОЛИЧЕСТВО_ТРАВЫ),
                                   //...
                                  );
```

**13. abstract class Action**

👍 Идеально
```java
public abstract class Action {
  public abstract void execute(WorldMap world);
}
```

**14. class RefreshResources extends Action**

- Магические числа.

```java
if (countInhabitant <= 3) {...}
if (countSoldier <= 3) {...}

//ПРАВИЛЬНО:
if (countInhabitant <= MIN_INHABITANT_COUNT) {...}
if (countSoldier <= MIN_SOLDIER_COUNT) {...}
```

Нарушение DRY, дублирование кода.  
Участки кода, которые делают одно и то же- помещают на карту существ одного и того же типа в заданном количестве 
```java
public void execute(WorldMap world) {
  //...  
  if (countInhabitant <= 3) {
    Coordinates coordinates = world.getRandomEmptyPlace();
    Entity entity = new InhabitantSpawn().createEntity();
    world.setEntities(coordinates, entity);
  }
  if (countSoldier <= 3) {
    Coordinates coordinates = world.getRandomEmptyPlace();
    Entity entity = new SoldierSpawn().createEntity();
    world.setEntities(coordinates, entity);
  }
  //...
}
```

Повторяющиеся куски кода выноси во вспомогательные методы:
```java

public void execute(WorldMap world) {
  //...
  respawn(world, countInhabitant, MIN_INHABITANT_COUNT, () -> new InhabitantSpawn().createEntity());
  respawn(world, countSoldier, MIN_SOLDIER_COUNT, () -> new SoldierSpawn().createEntity());
  //...
}

private void respawn(WorldMap worldMap, int count, int min, Supplier<Entity> entitySupplier) {
  if(count <= min) {
    Coordinates coordinates = world.getRandomEmptyPlace();
    Entity entity = entitySupplier.get();
    world.setEntities(coordinates, entity);
  }
}
```

Если встанет вопрос о том, как можно уменьшить количество кода и оптимально связать между собой спавпер и респавнер, 
то похожую ситуацию я рассматривал [ТУТ в п.14](https://github.com/raketareview/simulation_review/blob/master/content/rev-sim142-vadim-aveasura.md)

**15. class MoveCreature extends Action**

👍 Тут все ок. Разве что, можно сделать вспомогательный метод для нахождения всех креатур и убрать ненужную проверку
```java
public class MoveCreature extends Action {
  //...
  public void execute(WorldMap world) {
    ArrayList<Creature> creatures = new ArrayList<>();
    //заполняет creatures

    for (Creature creature : creatures) {
      if (world.getPosition(creature) != null) {  <-- этот метод не должен возвращать null, см. п. "WorldMap"
        creature.makeMove(world, bfs);
      }
    }
  }
}

//ЛУЧШЕ:
public class MoveCreature extends Action {
  //...
  public void execute(WorldMap world) {
    List<Creature> creatures = getCreatures(world);

    for (Creature creature : creatures) {
      creature.makeMove(world, bfs);
    }
  }

  private List<Creature> getCreatures(WorldMap world) {...}
}
```

**16. class RendererWorldMap**

+ 👍 Спрайты существ хранятся здесь, а не берутся из вамих существ, это хорошо.

- Нет никаких оснований делать `protected` метод, который предназначен для использования клиентом:
```java
public class RendererWorldMap {
  //...
  protected void printMap(WorldMap map) {...}
}

//ПРАВИЛЬНО:
public class RendererWorldMap {
  //...
  public void printMap(WorldMap map) {...}
}
```

- Нарушение инкапсуляции.  
Использование уровня protected для этого метода в классе ничем не оправдано:
```java
public class RendererWorldMap {
  //...
  protected String getBackgroundColor(Entity entity) {...}  <-- Должен быть private
}
```

- Нарушение конвенции кода.

Здесь нет контсруктора, поэтому метод `printMap(WorldMap map)` должен стоять первым, а не последним.  
Это единственный метод в классе, кторотый предназначен для использования клиентом.

- Нарушение SRP.

Рендерер не должен хранить в себе информацию о том, какой уровень здоровья существ является критическим.  
Задача рендерера- просто печатать карту. Знать критические уровни здоровья существ это не его ответственность.  
Эти данные он должен получить в конструктор
```java
if (creature.getHealth() <= 15 || creature.getHunger() >= 85) {...}
if (creature.getHealth() <= 30 || creature.getHunger() >= 70) {...}
```

Рендерер должен знать, что сущетствуют разные уровни здоровья и эти уровни нужно показывать разным цветом- это часть его ответственности.  
Рендерер должен самостоятельно знать информацию, каким цветом нужно распечатывать предупредительный(WARNING) и опасный(DANGER) уровень здоровья существ.  
Но при этом Рендерер должен получать извне информацию о том, какой конкретно уровень здоровья является предупредительным или опасным.

- В конце цепочки if-else нужно кидать исключение.

Если алгоритм дошел сюда, значит в карте есть существо, спрайт которого мы не знаем.  
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.  
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```java
private String getSymbolEntities(Entity entity) {
  if (entity instanceof Inhabitant) return INHABITANT;
  if (entity instanceof Soldier) return SOLDIER;
  //...
  return EMPTY_SPACE;  <-- ДЕФОЛТНЫЙ СПРАЙТ
}

//ПРАВИЛЬНО:
private String getSymbolEntities(Entity entity) {
  if (entity instanceof Inhabitant) {
    return INHABITANT;
  }
  if (entity instanceof Soldier) {
    return SOLDIER;
  }
  //...
  throw new IllegalArgumentException("Unknown entity: " + entity);
}
```

- Стандартные имена индексов в цикле- i,j, это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < map.getHeight(); i++) {
  for (int j = 0; j < map.getWidth(); j++) {
     Coordinates coordinates = new Coordinates(i, j);
  }
}

//ЛУЧШЕ:
for (int row = 0; row < map.getHeight(); row++) {
  for (int column = 0; column < map.getWidth(); column++) {
     Coordinates coordinates = new Coordinates(row, column);
  }
}
```

**17. class Simulation**

+ 👍 Принимает достаточное количество зависимостей в конструктор, это хорошо.  
В данном случае- принимает карту, этого тут достаточно
```java
public Simulation(WorldMap world) {...}
```

- Не используй базовый класс `Object` непосредственно в виде `Object`.

Если нужен какой-то объект, то подумай какую сущность он из себя представляет и что от него требуется.  
Если от него требуется только "быть самим собой" и что-то манифестировать своим названием, то просто сделай объект без поведения и состояния.  
Все равно он будет наследоваться от Object
```java
protected final Object lock = new Object();

//ПРАВИЛЬНО:
public class Simulation {
  protected final Lock lock = new Lock();
  //...

  private static class Lock {
  }
}
```

- Методы, предназначенные для использования клиентом, должны быть public.

Нет оснований делать доступ `protected` для методов, которые используют клиенты.
Все методы в этом классе, которые предназначены для использования клиентами, должны быть public.

- Одинаковые алгоритмы выноси во вспомогательные методы
```java
protected void start() {
  for (Action action : initActions) {
    action.execute(world);
  }
  renderer.printMap(world);
}

protected void nextTurn() {
  counterTurns++;
  for (Action action : turnActions) {
    action.execute(world);
  }
  //...
}

//ПРАВИЛЬНО:
protected void start() {
  executeActions(initActions);
  renderer.printMap(world);
}

protected void nextTurn() {
  counterTurns++;
  executeActions(turnActions);
  //...
}

private void executeActions(List<Coordinates> actions) {
  for (Action action : actions) {
    action.execute(world);
  }
}
```

**18. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, в данном случае через диалог с пользователем, это хорошо.

- Для более легкого и приятного чтения кода, вводи пояснябщие переменные
```java
Dialog<Character> letterSelectDialog = new EnglishLetterSelectDialog("Введите T - сделать один шаг, " +
  "S - запустить симуляцию, P - пауза," + "\n" + "R - рестарт симуляции после паузы, Q - завершение программы",
  "Неверная команда");

//ПРАВИЛЬНО:
String title = """
    Введите T - сделать один шаг, S - запустить симуляцию, P - пауза
    R - рестарт симуляции после паузы, Q - завершение программы
    """;
String failMessage = "Неверная команда";

Dialog<Character> letterSelectDialog = new EnglishLetterSelectDialog(title, failMessage);
```

Еще лучше- вынести создание диалога во вспомогательный метод:
```java
public static void main(String[] args) {
  //...
  Dialog<Character> letterSelectDialog = getLetterSelectDialog();
}

private static Dialog<Character> getLetterSelectDialog() {
  String title = ...
  String failMessage = ....

  return new EnglishLetterSelectDialog(title, failMessage);
}
```

- В таких случаях, приводи символы к нужному регистру
```java
while (true) {
  char type = letterSelectDialog.input();
  switch (type) {
    case 't', 'T' -> //...
    case 's', 'S' -> //...
  }
}

//ПРАВИЛЬНО:
while (true) {
  char type = letterSelectDialog.input();
  type = Character.toUpperCase(type);
  switch (type) {
    case 'T' -> //...
    case 'S' -> //...
  }
}
```

- Нарушение DRY, магические буквы, числа, слова. Вводи константы 
```java

Dialog<Character> letterSelectDialog = new EnglishLetterSelectDialog("Введите T - сделать один шаг, " +
  "S - запустить симуляцию, P - пауза," + "\n" + "R - рестарт симуляции после паузы, Q - завершение программы",
  "Неверная команда");

while (true) {
  char type = letterSelectDialog.input();
  type = Character.toUpperCase(type);
  switch (type) {
    case 'T' -> //...
    case 'S' -> //...
  }
}

//ПРАВИЛЬНО:
private final static char START = "S";
private final static char STEP = "T";
//...

String title = """
    Введите %c - сделать один шаг, %c - запустить симуляцию, %c - пауза
    %c - рестарт симуляции после паузы, %c - завершение программы
    """.formatted(STEP, START, ....);
String error = "Неверная команда";

Dialog<Character> letterSelectDialog = new EnglishLetterSelectDialog(title, error);

while (true) {
  char type = letterSelectDialog.input();
  type = Character.toUpperCase(type);
  switch (type) {
    case STEP -> //...
    case START -> //...
  }
}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

- Команды в этом классе и в классе символьного диалога должны быть синхронизированы между собой.

Сейчас одинаковые магические буквы-команды существуют независимо друг в `Main` и `EnglishLetterSelectDialog`:
```java
public class Main {
  public static void main(String[] args) {
    //...
    Dialog<Character> letterSelectDialog = new EnglishLetterSelectDialog("Введите T - сделать один шаг, " +
        "S - запустить симуляцию, P - пауза," + "\n" + "R - рестарт симуляции после паузы, Q - завершение программы",
        "Неверная команда");
  }
}

public class EnglishLetterSelectDialog extends AbstractDialog<Character> {
  protected boolean isAllowed(Character result) {
    result = Character.toLowerCase(result);
    return result == 't' || result == 's' || result == 'p' || result == 'r' || result == 'q';
  }
  //...
}
```

Диалог не должен знать про конкретные буквы, он их должен принимать в конструктор.

Среди прочего, это сделает диалог универсальным и готовым для переиспользования в других проектах, где нужно тоже получать конкретный символ от юзера.  
Такая универсальность класса является одной из главных мотиваций существования диалогов.  
Должно это быть примерно так:
```java
public class Main {
  private final static char START = "S";
  private final static char STEP = "T";
  //...
  private final static List<Character> COMMANDS = List.of(START, STEP, ...);

  public static void main(String[] args) {
    //...
    Dialog<Character> letterSelectDialog = new EnglishLetterSelectDialog(title, error, COMMANDS);
  }
}

public class EnglishLetterSelectDialog extends AbstractDialog<Character> {
  //...
  public EnglishLetterSelectDialog(String title, String error, List<Character> keys) {...}

  @Override
  public Character input() {
    //возвращает введенный юзером символ,
    //если он есть среди keys
  }  
}
```

**19. Использование доступа protected в методах классов**

В большинстве своем, тут неправильно используется уровень доступа `protected` в методах классов.

Если метод предназначен для использования *каким-то* клиентом, такой метод должен быть `piblic`.  
Если к методу не должны иметь доступ посторонние классы, а должны обращаться только потомки самого класса, тогда уровень доступа должен быть `protected`.  
Во всех остальных случаях уровень доступа должен быть `private`.  

Есть специфические ситуации, когда можно сделать `protected` метод именно для того, чтобы к нему обращался сторонний класс, а не потомок.  
Например: есть класс Автомобиль и его Билдер- класс, который создает экземпляр автомобиля через паттерн "Builder"(Строитель).  
Допустим, мы не хотим делать этот Билдер внутренним классом Автомобиля, чтобы не перегружать его кодом.  
Вместо этого делаем Билдер внешним классом.

Тогда делаем два класса: Автомобиль и БилдерАвтомобиля, помещаем их в отдельный пакет "car".  
Запрещаем создавать автомобиль через его конструктор, и разрешаем создавать автомобиль только через Билдер.  
Для этого делаем конструктор Автомобиля `protected`.

Теперь экземпляр автомобиля может создать только его Билдер и больше никто.  
Потому что `protected` это пакетный доступ, а в пакете "car" кроме этих двух классов никого больше нет.    
Соответственно никто, кроме Билдера, не сможет обратиться к конструктору Автомобиля, что и требовалось.

Это пример оправданного использования `protected` доступа к методу для использования не потомком, а сторонним классом.  

## ВЫВОД

В програме есть много усложнений- 12 классов существ, телепортация, ручное конфигурирование размера карты.  
Само по себе, это хорошо.

Но эти усложнения достигнуты путем ухудшения архитектуры.  
А приоритет этого проекта состоит в том, чтобы сделать сложный проект с плохой архитектурой.  
Приоритет в том, чтобы сделать простой проект с хорошей архитектурой.

Если человек может написать сложную программу с плохой архитектурой это не значит, что он автоматически может написать простую программу с хорошей архитектурой.  
Вполне возможно, что и простую программу он тоже напишет плохо.

Чтобы выснять твой уровень владения базой ООП, я рекомендую сделать шаг назад и переписать проект строго по ТЗ, без ничего лишнего.  
Без ручной конфигурации карты через мены, без десятков классов существ и телепортов.  
Просто базовый функционал: камень, дерево, трава, волк и заяц.

Тогда можно будет увидеть, как ты строшь иерархию классов- разносишь в них поля и методы.  
Потому что сейчас из-за большой и сложной иерархии классов сделать нормальный рефакторинг на таком уровне программирования не получится.
Иди от простого к сложному- сделай по ТЗ, но сделай хорошо.

Не нужно слишком увлекаться процессом.  
Научиться в результате написания Симуляции правильно строить иерархию классов- это важно.  
Сделать кучу всего интересного сверх ТЗ- нет.  
Потому что проект можно развивать бесконечно и это заберет время у более важных дел- быстрейшего прохождения роадмапа.

Пока я вижу много недостатков в использовании ООП на базовом уровне- 
начиная от неправильного использования уровней доступа в методах и неправильной установки значений полей предка в его потомках.

Всё это не отменяет того, что ты молодец, потому сделала программу с более сложным функционалом.  

n.143(304)  
#ревью #симуляция 