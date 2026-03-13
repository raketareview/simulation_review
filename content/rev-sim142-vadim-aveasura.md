https://github.com/aveasura/simulation  
[Vadim]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается буквами- само по себе это неплохо.  
Но трудно понять, кто есть кто и за кем нужно следить  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim142/img0.png)  
Я бы советовал статические объекты писать маленькими буквами(t - Tree), 
а ходячие- большими(R - rabbit)

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализованы пауза/пуск во время работы
+ 👍 Расширенное меню на старте- можно почитать правила
+ 🚀 Архитектура MVC

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для стандартных действий используй стандартные названия.  

Слово "place" неоднозначное, оно может быть использовано в разных значениях- как "место", например.  
Если нужно что-то куда-то добавить или поместить, используй слова "put" или "add".  
При прочих равных, всегда употребляй стандартные названия, чтобы всем были понятны твои намерения
```java
class GameMap {
 public void place(Position position, Entity entity) {
    //помещает существо в карту
 }
}

//ПРАВИЛЬНО: 
public void put(Position position, Entity entity) //или add(...)
```

- Не вижу, что добавляет частица "at" к пониманию того, что делает метод
```java
public Entity getAt(Position position)

//ЛУЧШЕ:
public Entity get(Position position)
```
Кроме того, возникает вопрос- если getAt, то почему не placeOn?
```java
Entity getAt(Position position) {...}
void place(Position position, Entity entity) {...}
```

- Придерживайся единообразия
```java
Entity getAt(Position position) 
void removeEntity(Position target)  

//ПРАВИЛЬНО:
Entity getAt(Position position) 
void removeEntity(Position position)  
```

- Название должно объяснять суть переменной.  
Эта переменная означает количество случайных позиций, которые требуется вернуть
```java
public List<Position> generate(GameMap gameMap, int entitiesToSpawn) {
  //возвращает случайные позиции в количестве entitiesToSpawn штук
}

//ПРАВИЛЬНО:
public List<Position> generate(GameMap gameMap, int count)
```
Этому методу не нужно знать ни про энтити, ни про то, что их будут спавнить.  
Это лишняя информация для него, которая не имеет отношения к тому, что **на самом деле** делает этот метод- 
возвращает заданное количество пустых позиций на карте.

- Для одной и той же концепции используй одно и то же название. Сделай все или "render" или "print" 
```java
public class ConsoleMainMenuRenderer implements MainMenuRenderer {
  public void renderRules() {
    output.println(RULES);  <-- ПЕЧАТАЕТ ТЕКСТОВОЕ СООБЩЕНИЕ
  }

  public void printExitMessage() {
    output.println("Выход...");  <-- ПЕЧАТАЕТ ТЕКСТОВОЕ СООБЩЕНИЕ
  }
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы. 

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов.  
Либо перенеси ее в третий класс и из первых двух классов обращайся к этим константам- эти данные должны быть синхронизированы между собой
```java
public class ConsoleControlMenu implements Menu {
  private static final String PAUSE = "1";
  private static final String RESUME = "2";
  private static final String STOP = "3";
  //...
}

public class ConsoleControlHintRenderer implements HintRenderer {
  private static final String CONTROLS_HINTS = """
      1. Приостановить симуляцию
      2. Продолжить симуляцию
      3. Остановить симуляцию
      """;
  //...      
```

То же самое касается `ConsoleMainMenuRenderer` и `ConsoleMapRenderer`- спрайты существ.  
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**3. record Position(int x, int y)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**4. class GameMap**

+ 👍 При выполнении операций, связанных с координатой, происходит валидация координаты. Это хорошо
```java
public void removeEntity(Position target) {
  validateInside(target);
  entities.remove(target);
}
```

+ 👍 Хорошо, что возвращается копия мапы, а не оригинал.  
Тем самым соблюдается инкапсуляция и защищается внутреннее устройство класса
```java
public Map<Position, Entity> getEntities() {
  return new HashMap<>(entities);
}
```

- Но я бы назвал этот метод более точнее
```java
public Map<Position, Entity> getEntities() {
  return new HashMap<>(entities);
}

//ЛУЧШЕ:
public Map<Position, Entity> toMap() {
  return new HashMap<>(entities);
}
```
По аналогии с применяющимися в стандартных классах в аналогичных ситуациях методах `toList(`), `toCharArray()` etc.

- Нарушение DRY. Переиспользуй имеющиеся методы
```java
public void move(Position from, Position to) {
  validateInside(from);
  validateInside(to);
  if (!entities.containsKey(from)) {
    throw new IllegalStateException("No entity at source position: " + from);
  }

  if (entities.containsKey(to)) {
    throw new IllegalArgumentException("Position already busy: " + to);
  }
  //...
}

//ПРАВИЛЬНО:
public void move(Position from, Position to) {
  if (!isOccupied(from)) {
    throw new IllegalStateException("No entity at source position: " + from);
  }

  if (isOccupied(to)) {
    throw new IllegalArgumentException("Position already busy: " + to);
  }
  //...
}
```

- Нарушение SRP.

Для выполнения своей единой ответственности, хранению существ, карте не нужно считать их количество
```java
public int getEntitiesCount() {
  return entities.size();
}
```
Определи тот класс, который не может жить без этого метода и перенеси его туда.  
Клиент может самостоятельно посчитать количество существ через подcчет размера мапы из уже имеющегося метода `public Map<Position, Entity> getEntities()`.

- Метод совершения хода в карте- нарушение SRP.  
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
Карта карта = new Карта(100, 100);
Заяц заяц = new Заяц();
карта.place(new Координата(0, 0), заяц);
карта.move(new Координата(0, 0), new Координата (99, 99));

/* class Карта */
public void move(Position from, Position to) {
  //12 строк логики совершения телепортации  
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как?  
Метод частично отвечает на этот вопрос- в нем содержится 12 строк кода, который отвечает за логику телепортации.  
Тем самым карта кроме своей единой ответственности, хранения существ, берет на себе дополнительную ответстсвенность- знать правила совершения хода существами.

Свое передвижение по карте осуществляет класс Creature, он и должен удалять себя из одной координаты в карте и вставлять по другой координате.

+ 👍 В целом, класс хороший. Имеющиеся в нем нарушения SRP несущественные.

**5. interface PathFinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
List<Position> find(GameMap gameMap, Position currentPosition, Predicate<Position> isTarget, Predicate<Position> canStep);
```

+ 👍 Предикат для определения цели это хорошо.  

Благодаря этому у существ есть возможность есть разные виды еды.  
То есть, Травоядное потенциально может есть не только Траву, но и какой-нибудь Фрукт.
```java
List<Position> find(..., Predicate<Position> isTarget);
```

+ 👍 Предикат для определения проходимых поверхностей само по себе тоже хорошо
```java
List<Position> find(..., Predicate<Position> canStep);
```
Вопрос в другом- не является ли для этого проекта избыточным дополнительно проверять поверхность на проходимость?   
Или достаточно считать проходимыми только пустые клетки в карте?  
Могут быть доводы и за и против. Лично я не против такой опции даже в виде точки расширения будущего функционала.

**6. class BfsPathFinder implements PathFinder**

+ 👍 Благодаря отдельному интерфейсу поиска ближайших координат, можно делать более унииверсальные поиски
```java
private final NeighborFinder neighborFinder;
```
Например, можно выполнять поиск только по горизонталям и вертикалям(так сейчас) или по всем 8 направлениям.

+ 👍 В классе все ок.

**7. enum EntityType**, дополнительная внешняя типизация

Енам существует для дополнительной внешней типизации классов
```java
public enum EntityType {
  RABBIT, FOX, GRASS, TREE, MOUNTAIN
}
```
Не используй дополнительную типизацию классов, потому что это дублирует стандартные способы.  
Используй стандарные способы определять принадлежность объектов к типам:
```java
EntityType type = entity.getType();

return switch (type) {
  case RABBIT     -> new Rabbit();
  case FOX        -> new Fox();
  //...
}

//ПРАВИЛЬНО:
String name = entity.getClass().getSimpleName();

switch (name) {
  case "Rabbit"     -> new Rabbit();
  case "Fox"        -> new Fox();
  //...
}
```

**8. abstract class Entity**

Не нужно хранить в классе еще дополнительное поле внешней типизации, которое дублирует стандартный `getClass()`
```java
public abstract class Entity {
  private final EntityType type;
  public EntityType getType() {...}
  //...
}
```

**9. abstract class Creature extends Entity implements Movable**

- Совмещение команды и запроса.

Методы должны или выполнить команду, или ответить на запрос. Но не то и другое вместе.

Этот метод выполняет команду- совершает перемещение по карте.  
И отвечает на запрос- сообщает результаты совершения хода
```java
public boolean makeMove(GameMap gameMap, Position from, List<Position> path) 

//ПРАИВЛЬНО:
public void makeMove(GameMap gameMap, Position from, List<Position> path) 
```
*Мартин, "Чистый код", гл.3, "Разделение команд и запросов"* 
```java
"Либо функция изменяет состояние объекта, либо возвращает информацию об этом объекте. 
Совмещение двух операций часто создает путаницу." - Мартин.
```
Принцип разделения команд и запросов это общее правило.  
Если ты считаешь, что в данном конкретном случае совмещение команды и запроса делает код проще и понятнее, то совмещай.  
Нужно просто знать, что такое правило существует и если ты его нарушаешь, то делай это осознанно.

- Нарушение SRP.

Единое действие,- совершение существом хода,- разделено на три класса:  
Этот, GameMap(метод `move()`) и MoveEntityAction.

Совершение хода должно происходить только в одном месте.  
В данном случае- в этом классе, в методе `makeMove(...)`:
```java
public boolean makeMove(GameMap gameMap, Position from, List<Position> path) 

//ПРАВИЛЬНО:
public void makeMove(GameMap gameMap, PathFinder pathFinder) {
  Position current = gameMap.getPosition(this);
  //...
  List<Position> path = pathFinder.find(gameMap, current, ...);
  //пройти по пути path и съесть еду
} 
```

**10. abstract class Predator extends Creature**

Если ты сделал алгоритм с более широкими возможностями, то подумай, как он будет применяться на практике.  
Например, у каждого хищника есть метод определения еды и метод взаимодействия с едой
```java
public boolean isFood(Entity entity) {
  return entity instanceof Herbivore;
}

protected void interactWithTarget(GameMap gameMap, Position finalTarget, Entity entityTarget) {
  if (entityTarget instanceof Herbivore herbivore) {
     this.attack(herbivore);
  }
  //...
}
```

Эти методы должны быть как-то связаны между собой.  
Представь, что будет, если хищник будет есть не только травоядного, но еще и птичку
```java
public boolean isFood(Entity entity) {
  return entity instanceof Herbivore || entity instanceof Bird;
}

protected void interactWithTarget(GameMap gameMap, Position finalTarget, Entity entityTarget) {
  if (entityTarget instanceof Herbivore herbivore) {  <-- А КАК ЖЕ ПТИЧКА?
     this.attack(herbivore);
  }
  //...
}
```

Подобного бы не возникло, если бы каждая креатуры ела только один вид еды. Например:
```java
class Creature бла-бла {
  private final Class<? extends Entity> food;
  //...
  public Creature(Class<? extends Entity> food, ...) {...}
}
```

Но раз ты расширил возможности существ, то это обязывает более тщательно продумывать алгоритмы.

**11. interface Action**

Вот здесь точно нет необходимости совмещать команду и запрос
```java
public interface Action {
  boolean execute(GameMap gameMap);
}

//ПРАВИЛЬНО:
public interface Action {
  void execute(GameMap gameMap);
}
```

Если экшен должен каким-то обазом сообщить результат выполнения скоего действия(хотя я не вижу, зачем это здесь было бы нужно экшенам),
то он он должен это сделать через паттерн Callback:  
[CallBack луноход](https://t.me/zhukovsd_it_chat/53243/139594)  
[CallBack разрушение](https://t.me/zhukovsd_it_chat/53243/184749)

**12. class MoveEntityAction implements Action**

Класс содежит в себе часть логики совершения хода существом, 
для этого в его методе `execute(...)` почти 40 строк кода. 

Этот экшен должен не выполнять часть логики по совершению хода существом, а только инициировать начало движения.  
Для этого ему нужно просто обойти всю карту, найти каждую креатуру и дать ей пинка, чтобы она побежала.  
Как при этом будет бежать креатура и что при этом делать, не должно волновать экшен хода.  
Выглядеть это должно примерно так:
```java
class MoveAction реализует Action {
  //...

  @Override
  public void execute(Карта карта) {
    List<Creature> creatures = getCreatures(карта);
    for(Creature creature: creatures) {
      creature.makeMove(карта, pathFinder);  //даёт пинка
    }
  }

  private static List<Creature> getCreatures(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

**13. class SpawnInitialEntitiesAction implements Action**

- Нарушение инкапсуляции. 

Публичными должны быть только те методы, которые предназначены для использования клиентом.  
Вспомогательные методы, которые используются только в самом классе или его потомках, должны быть private/protect
```java
public class SpawnInitialEntitiesAction implements Action {
  @Override
  public boolean execute(GameMap gameMap) {
    //...
    spawn(gameMap, randomPositions, entities);
  }
  public void spawn(GameMap gameMap, List<Position> positions, List<Entity> entities) {...}
}

//ПРАВИЛЬНО:
public class SpawnInitialEntitiesAction implements Action {
  @Override
  public boolean execute(GameMap gameMap) {
    //...
    spawn(gameMap, randomPositions, entities);
  }
  private void spawn(GameMap gameMap, List<Position> positions, List<Entity> entities) {...}
}
```
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

- Нарушение DRY, дублирование кода, индусский код.

Повторяющиеся действия делай через циклы
```java
public boolean execute(GameMap gameMap) {
  List<Entity> entities = createInitialEntities();
  //бла-бла
  spawn(gameMap, randomPositions, entities);
}

public void spawn(GameMap gameMap, List<Position> positions, List<Entity> entities) {
  for (int i = 0; i < positions.size(); i++) {
    Position position = positions.get(i);
    Entity entity = entities.get(i);
    gameMap.place(position, entity);
  }
  //...
}

private List<Entity> createInitialEntities() {
  List<Entity> entities = new ArrayList<>();

  entities.add(entityFactory.create(EntityType.RABBIT));
  entities.add(entityFactory.create(EntityType.RABBIT));
  entities.add(entityFactory.create(EntityType.RABBIT));

  entities.add(entityFactory.create(EntityType.FOX));

  entities.add(entityFactory.create(EntityType.GRASS));
  entities.add(entityFactory.create(EntityType.GRASS));
  entities.add(entityFactory.create(EntityType.GRASS));
  //...
  return entities;
}

//ПРАВИЛЬНО:
private final static Map<EntityType, Integer> ENTITY_NUMBERS = Map.of(
    EntityType.RABBIT, 3,
    EntityType.FOX, 1,
    EntityType.GRASS, 3
);

public void execute(GameMap gameMap) {
  for(Map.Entry<EntityType, Integer> entry : ENTITY_NUMBERS.entrySet() ) {
    EntityType type = entry.getKey();
    int count = entry.getValue();
    spawn(gameMap, type, count);
  }
}

private void spawn(GameMap gameMap, EntityType type, int count) {
  for(int i = 0; i < count; i++) {
    Position position = getRandomFreePosition();
    Entity entity = entityFactory.create(type);
    gameMap.place(position, entity);
  }
}
```

**14. class RespawnEntitiesAction implements Action**

Нарушение DRY.  
Во многом класс повторяет `SpawnInitialEntitiesAction`.  
Оба эти класса выполняют похожие действия- заполняют карту существами заданного типа в заданном количестве.

Либо выноси общий код во вспомогательный класс. 
Либо создай общего предка и вынеси туда:
```java
public abstract class SpawnAction implements Action {
  protected final static Map<EntityType, Integer> ENTITY_MAX_NUMBERS = Map.of(
    EntityType.RABBIT, 3,
    EntityType.FOX, 1,
    //...
  );

  protected void spawn(GameMap gameMap, EntityType type, int count) {
    //...
  }

  protected Position getRandomFreePosition(GameMap gameMap) {
    //...
  }
}

public class InitSpawnAction implements SpawnAction {
  @Override
  public boolean execute(GameMap gameMap) {
    //заполняет карту существами
    //тип существ и их количество берет из ENTITY_MAX_NUMBERS
  }
}

public class RespawnAction implements SpawnAction {
  
  private final static Map<EntityType, Integer> ENTITY_MIN_NUMBERS = Map.of(
    EntityType.RABBIT, 1,
    EntityType.FOX, 1,
    //...
  );

  @Override
  public boolean execute(GameMap gameMap) {
    for(Map.Entry<EntityType, Integer> entry : ENTITY_MIN_NUMBERS.entrySet() ) {
      EntityType type = entry.getKey();
      int minCount = entry.getValue();
      int currentCount = calcCount(type);

      if(minCount > current) {  //существ меньше минимального- восполнить количество существ этого типа до максимального
        int maxCount = ENTITY_MAX_NUMBERS.get(type);
        int count = maxCount - currentCount;
        spawn(gameMap, type, count);
      }
    }
  }
  //...
}
```

**15. class SimulationEndCondition**

+ 👍 Отдельный класс, который определяет условия окончания работы симуляции. Ничего не имею против.

- Здесь бы пригодилось использование метода, который считает количество существ по их типу.  
Например, такой метод мог бы находиться в утилитном классе SimulationMapUtils:
```java
public boolean isFinished(GameMap gameMap, boolean turnChanged) {
  for (Entity entity : gameMap.getEntities().values()) {
    if (entity instanceof Predator) {
      hasPredators = true;
    } else if (entity instanceof Herbivore) {
      hasHerbivores = true;
    } else if (entity instanceof Grass) {
      hasGrass = true;
    }
  }
  //...
  boolean missingRequiredEntities = !hasPredators || !hasHerbivores || !hasGrass;
  boolean noChangesThisTurn = !turnChanged;

  return missingRequiredEntities || noChangesThisTurn;
}

//ЛУЧШЕ:
public boolean isFinished(GameMap gameMap, boolean turnChanged) {
  return SimulationMapUtils.count(Predator.class) == 0 ||
      SimulationMapUtils.count(Herbivore.class) == 0 ||
      SimulationMapUtils.count(Grass.class) == 0 ||
      !turnChanged;
}
```
Этот же метод можно было бы использовать для подсчета текущего количества существ в `RespawnAction`.

**16. Фабрики**

+ 👍 Фабрики это хорошо.

- Фабрика энтити должна принимать класс существа. А не тип, который по факту его дублирует
```java
public interface EntityFactory {
  Entity create(EntityType type);
}

//ПРАВИЛЬНО:
public interface EntityFactory {
  Entity create(Class <? extends Entity> type);
}
```

**17. Пакет console**

Эти интерфейсы не завязаны на консоль или любую другую среду ввода-вывода, поэтому не должны лежать в пакете "console"
```java
package org.simulation.console;  <-- пакет консоль

public interface Menu {
  void start();
}
```

Теоретически, да и практически, можно сделать реализацию этого интерфейса для виндовс UI, Http или аркадного автомата на лампочках
```java
public interface Menu  <-- Не должен находиться в пакете "console"
public class ConsoleMainMenu implements Menu  <-- Может находиться в пакете "console"
public class ArcadeMashineMainMenu implements Menu  <-- Гипотетическая реализация меню для аркадного автомата на лампочках
```

**18. Рендереры**

+ 👍 Есть группа интерфейсов для рендеринга и их реализации для консоли. Это хорошо.

+ 👍 Слой View, архитектура MVC.

Хорошо, что есть отдельные интерфейсы и их реализации для ввода-вывода информации от юзера.  
Таким образом, получилосось что-то типа слоя View, через который происходит все общение с юзером.  
Фактически, благоря этому архитектура программы соответствует MVC.

- Интерфейсов и классов для рендеринга так много, что выглядит это как оверинжиниринг.

Тем более, что в коде не продемонстрировано, что дает такой подход- нет даже разных реализаций рендерера для карты.  
Например, в дополнение к текущему текстовому рендереру можно сделать альтернативный рендерер, который бы печатал карту в виде эмодзи.

Существование некоторых рендереров непонятно.  
Например, неясно зачем нужен этот отдельный рендерер(плюс его интерфейс) с пунктами меню:
```java
public class ConsoleControlHintRenderer implements HintRenderer {
  private static final String CONTROLS_HINTS = """
      1. Приостановить симуляцию
      2. Продолжить симуляцию
      3. Остановить симуляцию
      """;
  //...
}
```
Лучше создать такую архитектуру меню, чтобы можно было создавать разные меню без использования для этого персональных рендереров.  
Подробнее- ниже, в пункте про меню.

- Название многих рендереров не соответствует действительности.  

Это не консольные рендереры- потому что печатают в интерфейс, а не в консоль
```java
public class ConsoleControlMenuRenderer implements ControlMenuRenderer {

  public void printPaused() {
    output.println("Симуляция поставлена на паузу.");
  }

  public void printResumed() {
    output.println("Симуляция продолжается.");
  }
  //...
}
```
Я могу создать не консольный Output, передать его в `ConsoleControlMenuRenderer` и это будет работать.  
Например, могу сделать реализацию Output, которая через порт LPT будет печатать на матричный принтер.

Этот рендерер просто последовательно печатает текст *куда-то*, а не конкретно в консоль.  
Это нужно отразить в названии класса, чтобы не вводить в заблуждение.

**19. Меню**

Группа интерфейсов и классов для меню
```java
interface Menu
class MenuLoop
class ConsoleMainMenu implements Menu
class ConsoleControlMenu implements Menu
interface MainMenuRenderer
interface ControlMenuRenderer
class ConsoleMainMenuRenderer implements MainMenuRenderer
//... и другие
```
Несмотря на кажующуюся объектно-ориентированность(интерфейсы, наследование), эти меню сделаны в процедурном стиле.  
Для каждого конкретного меню здесь существует отдельный интерфейс + его реализация.

Для простой программы с одним меню було бы норм.  
Но здесь куча меню и желание делать ООП.
Поэтому и меню нужно сделать в ООП стиле.

Про меню в ООП стиле я писал [ТУТ](https://t.me/zhukovsd_it_chat/53243/114908).

Общая идея такова- нужно создать такой класс меню, который был бы универсальным для всех действий типа "выбрал пункт меню-запустил его".  
Например, так:
```java
//Создание
Menu mainMenu = new Menu(Ouptut output, Input input, "Главное меню");
mainMenu.add("Показать правила", () -> showRules());
mainMenu.add("Запустить симуляцию", () -> startSimulation());
mainMenu.add("Выход", () -> quit());

//Использование
mainMenu.show();
mainMenu.execute();
```

**20. class ConsoleMapRenderer implements MapRenderer**

+ 👍 Спрайты существ хранятся здесь, а не берутся из самих существ. Это хорошо.

- Нарушение SRP.

Класс должен печатать только карту, а не сопроводительную информацию
```java
public void render(GameMap gameMap, int step) {
  output.println("Ход: " + step);
  //печатает карту
}

//ПРАВИЛЬНО:
public void render(GameMap gameMap) {
  //печатает карту
}
```

- Предусматривай простоту редактирования. 

Если нужно будет поменять количество пробелов до и после буквы существа, придется вносить изменения во многих строках сразу:
```java
//ДО:
private String getEntitySymbol(Entity entity) {
  return switch (entity.getType()) {
    case FOX -> " F ";
    case RABBIT -> " R ";
    //...
  };
}

//ПОСЛЕ:
private String getEntitySymbol(Entity entity) {
  return switch (entity.getType()) {
    case FOX -> "F";
    case RABBIT -> "R";
    //...
  };
}
```

А вот так нужно будет поменять шаблон только в одном месте:
```java
private String getEntitySymbol(Entity entity) {
  String letter = switch (entity.getType()) {
    case FOX -> "F";
    case RABBIT -> "R";
    //...
  };
  return " %s ".formatted(letter);
}
```

**21. class Simulation**

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий.  
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе `Simulation`.  
Это хорошо.

- Слишком много входящих аргументов.

Если метод(а конструктор тоже метод) принимает во входящие больше 3-4 аргументов, то нужно либо использовать объект-аргумент, либо паттерн "Builder"
```java
public Simulation(GameMap gameMap,  <-- МЕТОД ПРИНИМАЕТ 7 АРГУМЕНТОВ
                  List<Action> initialActions,
                  List<Action> turnActions,
                  MapRenderer mapRenderer,
                  HintRenderer hintRenderer,
                  SimulationEndCondition endCondition,
                  Sleeper sleeper) {...}
```
*"ЧК", гл.3, "Объекты как аргументы"*

- Нарушение паттерна GRASP "Creator"(Создатель)

Здесь класс принимает в конструктор зависимость, которую должен создавать самостоятельно:
```java
public Simulation(GameMap gameMap, List<Action> initialActions, ... , SimulationEndCondition endCondition, ...) {...}

//ПРАВИЛЬНО:
public Simulation(GameMap gameMap, List<Action> initialActions, ...) {
  //...
  this.endCondition = new SimulationEndCondition();
}
```
Creator гласит, что создавать объект должен тот, кто его использует. 

Рассмотрим, что это значит.

В данном случае `Simulation` принимает в конструктор `GameMap`- это правильно.  
Потому что карта может быть создана разных размеров. Поэтому ее создает клиент, а потом инжектит в этот класс.

Еще `Simulation` принимает в конструктор `initActions`- это тоже правильно.  
Потому что список экшенов может быть разным. 

Но принимать в конструктор `SimulationEndCondition`- уже неправильно.  
Потому что `SimulationEndCondition` это конкретный класс, а не интерфейс и этот класс не параметризируется. 

Экземпляр `SimulationEndCondition` всегда одинаковый.  
Поэтому, согласно паттерна Creator, объект `SimulationEndCondition` *здесь* должен создавать сам класс Simulation.

Технически, можно создать наследника `SimulationEndCondition`, переопределить в нем методы и передавать в конструктор Simulation.  
Но если программист хочет использовать в своем классе другие классы через полиморфизм, то он должен обозначить свои намерения более явно.  
Например, передавать в конструктор интерфейс или абстрактный класс.

- Совмещение команды и запроса
```java
boolean executeActions(List<Action> actions)
```
Зачем совмещать- неясно. Все равно возвращаемое значение никак не учитывается
```java
public void startSimulation() {
  executeActions(initialActions);
  //...
}
```
Еще совмещение команды и запроса:
```java
boolean finishIfNeeded(boolean turnChanged) {
  //что-то выполняет
  //что-то возвращает
}
```

**22. class Main**, содержит точку входа main

👍 Только создает зависимости, инжектит их в Симуляцию и запускает ее. Это хорошо.

## ВЫВОД

Архитектура немного поплыла, когда ты решил отделить слой View через использование интерфейсов пользовательского ввода-вывода.  
Но получилось MVC, это хорошо.

В целом, норм.

n.142(303)  
#ревью #симуляция 