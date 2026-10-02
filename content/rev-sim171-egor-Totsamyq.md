https://github.com/Totsamyq/Simulation  
[Егор]

Есть над чем поработать.  
Функционал не полностью соответствует ТЗ.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. В программе не реализована работа с потоками и выполнение старт/стоп во время работы симуляции.  
Хотя это требование тоже есть в ТЗ.

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Не давай двусмысленных названий.

Это `Map` Java Core или кастомный `Map`?
```java
private Map map
```

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относятся.  
И вообще не употребляй венгерскую нотацию. 
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
HashMap<Position, Entity> worldMap
List<Position> listposition = new ArrayList();

//ПРАВИЛЬНО:
List<Position> positions = new ArrayList();
```

- UPPER_CASE только для названий констант
```java
public class PathFinder {
  public List<Position> BFS(Entity entity) {...}  <-- Это не константа, это метод
}

//ПОЧТИ ПРАВИЛЬНО ТАК:
public class PathFinder {
  public List<Position> bfs(Entity entity) {...}  
}
```

Но название метода должно быть глаголом, а BFS это существительное - название алгоритма.  
Поэтому тут совсем правильно будет так:
```java
public class BfsPathFinder {
  public List<Position> find(Entity entity) {...}  
}
```

- Для переменных используй простые, устоявшиеся имена. Коллеги могут не знать английский на уровне Oxford 3000
```java
public abstract class Entity {
  private String appearance;  <-- Изображение существа
 
  //...
}

//ПРАВИЛЬНО:
public abstract class Entity {
  private String sprite;  <-- Спрайт это графический объект в компьютерной графике, устоявшийся термин
 
  //...
}
```

- Название методов должно быть глаголом в повелительном наклонении
```java
abstract public class Action {

  abstract public void toPerform(Map map, PathFinder pathFinder);
}

//ПРАВИЛЬНО:
abstract public class Action {

  abstract public void perform(Map map, PathFinder pathFinder);
}
```

- Названия переменных должны быть в стиле camelCase
```java
int predatornum
int stepcounter

//ПРАВИЛЬНО:
int predatorNum
int stepCounter
```

- Названия методов должны быть в стиле camelCase
```java
public void Process(Map map) {...}

//ПРАВИЛЬНО:
public void process(Map map) {...}
```

- Рендерер рендерит
```java
public class Renderer {

  public void Process(Map map) {...}
}

//ПРАВИЛЬНО:
public class Renderer {

  public void render(Map map) {...}
}
```

- Коллекции нужно называть во множественном числе
```java
List<Action> initAction;
List<Action> turnAction;

//ПРАВИЛЬНО:
List<Action> initActions;
List<Action> turnActions;
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки  
```java
if (position.getX() < 0 || position.getX() >= width || position.getY() < 0 || position.getY() >= height)
  return false;//за пределами карты
else
  return true;

//ПРАВИЛЬНО:
if (position.getX() < 0 || position.getX() >= width || position.getY() < 0 || position.getY() >= height) {
  return false;
}  else {
  return true;
}
```
Исключение- метод equals(), там де-факто разрешается после `if` не выделять блоки скобочками. 

- Конструктор должен стоять выше остальных методов
```java
public abstract class Entity {

  public String getAppearance() {...}

  public Entity(Position position, String appearance) {...}
}

//ПРАВИЛЬНО:
public abstract class Entity {

  public Entity(Position position, String appearance) {...}

  public String getAppearance() {...}
}
```

То же самое касается многих других классов в проекте.

Конструктор должен стоять выше остальных методов.  
И это не просто формальное требование, его несоблюдение имеет практические последствия.  
Если конструктор не стоит первым, то читатель кода может подумать, что класс имеет только конструктор по умолчанию и будет исходить из этого предположения.  
А на самом деле все будет обстоять совсем наоборот.

*"Oracle Java Code Conventions"*

**3. Разделение классов по пакетам**

Сейчас в проекте 17 классов, которые все лежат в одной куче.  
Если в проекте больше 4-5 классов, то их нужно распределить по пакетам.  
Например:
```java
entities
  |
  + -- Entity.java
  + -- Rock.java
  + -- Grass.java
  + -- ...

actions
  |
  + -- Action.java
  + -- UnoAction.java
  + -- DosAction.java
  + -- ...

Main.java    
Simulation.java    
```

**4. class Position**

+ 👍 Нет ничего лишнего, это хорошо. Класс только хранит значения x,y. 

- Класс может быть преобразован в `record` без потери функционала.

Для простых классов-контейнеров данных лучше всего использовать особый тип класса: `record`.  
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**5. class Map**

- Не называй свои классы так же, как называются стандартные классы и интерфейсы Java Core.  

Класс не должен называться "Map" - это название стандартного интерфейса Java.

Следовать ТЗ это хорошо. 
Называть свои пользовательские классы так же, как стандартные классы и интерфейсы- плохо.  
Второе перевешивает, потому что это приводит к постоянной путанице в коде.

Например, такой код будет всё время сбивать с толку, потому что придется всё время на нем останавливаться и думать, 
какой именно из Map'ов имеется в виду- стандартный или кастомный
```java
public class Simulation {
  private Map map; <-- Это Map или Map?
  //...
}
```

Для наименования классов придерживайся терминологии предметной области и среды разработки.  
Если они вступают в противоречие, жертвуй терминологией предметной области. В данном случае, предметная область- это ТЗ.

Конкретно в этом случае вместо "Map", назови свой кастомный класс как-либо иначе: "Board", "GameMap" etc. 

- Избыточно
```java
private HashMap<Position, Entity> worldMap = new HashMap<Position, Entity>();

//ПРАВИЛЬНО:
private HashMap<Position, Entity> worldMap = new HashMap<>();
```

- Нарушение SRP, OCP и ТЗ

Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретными классами-наследниками Entity.  
Здесь карта хранит каждый вид существ в отдельной мапе
```java
private HashMap<Position, Entity> worldMap = new HashMap<Position, Entity>();  <-- ОСТАВИТЬ ТОЛЬКО ЭТО
private LinkedList<Predator> predatorList = new LinkedList<Predator>();  <-- ЛИШНЕЕ
private LinkedList<Herbivore> herbivoreList = new LinkedList<Herbivore>();  <-- ЛИШНЕЕ
private LinkedList<Grass> grassList = new LinkedList<>();  <-- ЛИШНЕЕ
```
Нарушение SRP: карта знает слишком много. 

Нарушение OCP: класс открыт для изменений.  
При добавлении нового существа в проект, придётся добавлять в класс новый код для этого существа:
```java
private LinkedList<Bird> birdList = new LinkedList<>();  
```

Нарушение ТЗ:
```
Чеклист для самопроверки #
Проблемы и ошибки в коде:
Параллельные коллекции для разных типов существ (для каждого типа своя коллекция)
```

Для того, чтобы карта работала так, как надо, достаточно использовать только одну мапу:
```java
private Map<Position, Entity> entities = new HashMap<>();
```

- Если во время жизни объекта не планируешь изменять в нём значение определенных полей, то делай их неизменяемыми 
```java
private int height;
private int width;

//ПРАВИЛЬНО:
private final int height;
private final int width;
```

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Herbivore> getHerbivore() {...}
public List<Predator> getPredator() {...}

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда предаторов и травоядных

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```

- Дублирование кода
```java
public boolean addEntity(Entity entity) {
  if (isCellFree(entity.getPosition()) && isWithinBounds(entity.getPosition())) {...}
}  

public boolean removeEntity(Entity entity) {
  if (!isCellFree(entity.getPosition()) && isWithinBounds(entity.getPosition())) {...}
}

//ПРАВИЛЬНО:
public boolean addEntity(Entity entity) {
  Position position = entity.getPosition(); 
  if (isValidPosition(position)) {...}
}  

public boolean removeEntity(Entity entity) {
  Position position = entity.getPosition(); 
  if (isValidPosition(position)) {...}
}
```

- Особые случаи обрабатывай сразу
```java
public boolean addEntity(Entity entity) {
  if (isCellFree(entity.getPosition()) && isWithinBounds(entity.getPosition())) {
    //8 строк кода

    return true;
  } else {
    return false;
  }
}

//ПРАВИЛЬНО:
public boolean addEntity(Entity entity) {
  Position position =  entity.getPosition(); 
  
  if (!isValidPosition(position)) {
    return false;
  }

  //8 строк кода
  return true;
}
```

- Разделяй команды и запросы. 

Этот метод совмещает команду и запрос.  
Команда: вставить существо.  
Запрос: вернуть статус выполнения операции- успешно или нет
```java
boolean addEntity(Entity entity)
```
Метод должен или выполнять команду, или отвечать на запрос.
 
*Мартин, "Чистый код", гл.3, "Разделение команд и запросов"* 
```java
"Либо функция изменяет состояние объекта, либо возвращает информацию об этом объекте. 
Совмещение двух операций часто создает путаницу." - Мартин.
```

Метод должен делать то, что он обещает своим контрактом.  
Если он не может выполнить свой контракт- он должен кинуть исключение.

Клиент должен перед вставкой существа на координату проверить, свободна ли координата в карте:
```java
if(gameMap.isEmpty(position)) {
  gameMap.addEntity(entity, position);  <-- Если попытаемся вставить на занятую координату, метод должен кинуть исключение  
} else {
  //попытались вставить существо на занятую координату    
}
```

- Нарушение SRP.

Карта должна просто вставлять существо на указанную координату.  
Карта не должна сама опрашивать существо или кого-либо ещё на предмет выяснения, где лежит нужная координата
```java
public boolean addEntity(Entity entity) {
  //...
  Position position = entity.getPosition();
  worldMap.put(position, entity);
}

//ПРАВИЛЬНО:
public boolean addEntity(Entity entity, Position position) {
  //...
  entity.setPosition(position);
  worldMap.put(position, entity);
}
```

- Последствия использования громоздкой системы параллельных коллекций существ
```java
public boolean addEntity(Entity entity) {
  if (isCellFree(entity.getPosition()) && isWithinBounds(entity.getPosition())) {
    worldMap.put(entity.getPosition(), entity);
  if (entity instanceof Herbivore) {
    herbivoreList.add((Herbivore) entity);
  } else if (entity instanceof Predator) {
    predatorList.add((Predator) entity);
  } 
  //...
    return true;
  } else {
    return false;
  }
}

//ПРАВИЛЬНО:
public void addEntity(Entity entity, Position position) {
  validate(position); <-- Если координата не в пределах карты, то бросает исключение 

  entity.setPosition(position);
  worldMap.put(position, entity);
}
```

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность  
```java
public Entity getEntityAt(Position position) {
  return worldMap.get(position);
}

public boolean isCellFree(Position position) {
  if (worldMap.containsKey(position)) {
    return false;
  }
  return true;
}

//ПРАВИЛЬНО:
public Entity position(Position position) {
  validate(position);  <-- Если координата не в пределах карты, то бросает исключение  
  return worldMap.get(position);  
}

public boolean isCellFree(Position position) {
  validate(position);  <-- Если координата не в пределах карты, то бросает исключение  
  return !worldMap.containsKey(position);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Метод совершения хода в карте- нарушение SRP.

Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
public boolean moveEntity(Entity entity, Position newposition) {
  //12 строк логики выполнения хода    
}
```
Должна ли карта учитывать правила логистики зайцев? Если да, то как?

Метод частично отвечает на эти вопросы- в нем 12 строк логики выполнения хода существом.  
То есть карта не только хранит существ, но и хранит правила того, как они должны ходить.  
Тем самым карта забирает себе часть чужой ответственности.

Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Избыточно. 

Результатом любой if-операции итак является значение `boolean` 
```java
public boolean isWithinBounds(Position position) {
  if (position.getX() < 0 || position.getX() >= width || position.getY() < 0 || position.getY() >= height)
   return false;//за пределами карты
  else
    return true;
}

//ПРАВИЛЬНО:
public boolean isWithinBounds(Position position) {
  return position.getX() >= 0 && position.getX() < width && position.getY() >= 0 && position.getY() < height;
}
```

**6. class PathFinder**

- Нарушение SRP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя. Он НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```java
public class PathFinder {
  //...

  public List<Position> BFS(Entity entity) {
    //...
    if (!(map.getEntityAt(newposition) instanceof Rock) && !(map.getEntityAt(newposition) instanceof Tree)) {  <-- КАКОЙ-ТО АД
      if (entity instanceof Herbivore && map.getEntityAt(newposition) instanceof Grass) {...}
    //... 
    if (entity instanceof Predator && map.getEntityAt(newposition) instanceof Herbivore) {...}
    }
  }
}
```

Чтобы найти путь по алгоритму BFS, классу поиска достаточно знать координату начала пути, тип искомого существа и то, что путь ищется по свободным ячейкам карты.

Эту информацию поиск должен получать от клиента и не определять самостоятельно путем опроса существ:
```java
public class BfsPathFinder {

  public List<Coordinates> find(World world, Coordinates start, Class<? extends Entity> target) {
    //ищет путь на карте world от точки start
    //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

- Такие конструкции это абсолютный ад. Вводи вспомогательные методы
```java
if (!(map.getEntityAt(newposition) instanceof Rock) && !(map.getEntityAt(newposition) instanceof Tree)) {

  if (entity instanceof Herbivore && map.getEntityAt(newposition) instanceof Grass) {
    //...
  }
}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняетUno(newposition)) {

  if (isНазваниеКотороеВсеОбъясняетDos(entity)) {
    //...
  }
}
```

- Антипаттерн "Стрела".

Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
while (!queue.isEmpty()) {
  for (int dx = -1; dx <= 1; dx++) {
    for (int dy = -1; dy <= 1; dy++) {
      if (map.isWithinBounds(newposition) && !visitedСells.contains(newposition)) {
        if (!(map.getEntityAt(newposition) instanceof Rock) && !(map.getEntityAt(newposition) instanceof Tree)) {
          if (entity instanceof Herbivore && map.getEntityAt(newposition) instanceof Grass) {
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

**7. abstract class Entity**

- Антипаттерн "Лодочный якорь".

Куча закомментированного мусора.

- Содержит координату
```java
private Position position;
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```java
private String appearance;
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах (консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

+ 👍 Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**8. class Grass extends Entity**

Объекты нужно принимать в себя в готовом виде, а не по запчастям
```java
public class Grass extends Entity {

  public Grass(int x, int y) {
    super(new Position(x, y), "\uD83C\uDF3F");
  }
}

//ПРАВИЛЬНО:
public class Grass extends Entity {
  private final static String SPRITE = "\uD83C\uDF3F";
 
  public Grass(Position position) {
    super(position, SPRITE);
  }
}
```

**9. class Predator/Herbivore extends Creature**

У Зайца и Волка частично дублируется код в `makeMove()`.  
Общий код потомков выноси в методы на уровень предка, здесь- в `Creature`.

ТЗ:
```
Чеклист для самопроверки #
Проблемы и ошибки в коде:
Дублирование кода между классами Herbivore и Predator
```

**10. class Action**

Не всем экшенам для своей работы нужно искать путь на карте.  
Например, это не надо экшену `PopulateMapAction` 
```java
abstract public class Action {
  abstract public void toPerform(Map map, PathFinder pathFinder);
}

//ПРАВИЛЬНО:
abstract public class Action {
  abstract public void perform(GameMap gameMap);
}
```

**11. class MoveCreaturesAction extends Action**

- Крушение поезда
```java
predatorListCopy.get(i).makeMove(map, pathFinder);
```

- Ходить существами нужно на уровне `Creature`, а не на уровне его потомков
```java
List<Herbivore> herbivoreListCopy = new ArrayList<>(map.getHerbivore());
List<Predator> predatorListCopy = new ArrayList<>(map.getPredator());

for (int i = 0; i < herbivoreListCopy.size(); i++) {
  herbivoreListCopy.get(i).makeMove(map, pathFinder);
}

for (int i = 0; i < predatorListCopy.size(); i++) {
  predatorListCopy.get(i).makeMove(map, pathFinder);
}

//ПРАВИЛЬНО:
List<Creature> creatures = gameMap.getEntitiesBy(Creature.class);

for (Creature creature : creatures) {
  creature.makeMove(map, pathFinder);  
}
```

**12. class PopulateMapAction extends Action**

Дублирование кода
```java
public void toPerform(Map map, PathFinder pathFinder) {
  int i = 0;
  while (i < predatornum) {
    int x = random.nextInt(map.getWidth());
    int y = random.nextInt(map.getHeight());
    Position position = new Position(x, y);
    if (map.isCellFree(position)) {
      Predator predator = new Predator(x, y, 100, 1, 35);
      map.addEntity(predator);
    } else continue;
    i++;
  }

  i = 0;
  while (i < herbivorenum) {
    int x = random.nextInt(map.getWidth());
    int y = random.nextInt(map.getHeight());
    Position position = new Position(x, y);
    if (map.isCellFree(position)) {
      Herbivore herbivore = new Herbivore(x, y, 100, 1);
      map.addEntity(herbivore);
    } else continue;
    i++;
  }

  //Ещё три таких же блока
}
```

Правильно так:
```java
public void perform(GameMap gameMap) {
  spawn(gameMap, (pos) -> new Predator(pos, 100, 1, 35), predatorCount);
  spawn(gameMap, (pos) -> new Herbivore(pos, 100, 1), herbivoreCount);
  //...
}

private void spawn(Карта карта, Function<Coordinates, Entity> entityCreator, int count) {
  for (int i = 0; i < count; i++) {
    Position position = getRandomEmptyPosition(карта);
    Entity entity = entityCreator.apply(position);

    карта.addEntity(entity, position);
  }
}

private Position getRandomEmptyPosition(Карта карта) {
  //находит и возвращает случайную пустую координату
}
```

Гугли тему "Стандартные функциональные интерфейсы Java".

**13. class Renderer**

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < map.getHeight(); i++) {
  for (int j = 0; j < map.getWidth(); j++) {
    Position position = new Position(j, i);
    //...
  }
}

//ЛУЧШЕ:
for (int y = 0; y < map.getHeight(); y++) {
  for (int x = 0; x < map.getWidth(); x++) {
    Position position = new Position(x, y);
    //...
  }
}
```

- Определение картинки лучше вынести во вспомогательные метод
```java
public void Process(Map map) {
  //...    
  if (!map.isCellFree(position)) {
    if (map.getEntityAt(position) instanceof Predator) {
      System.out.print("\uD83D\uDC3A");
    }

    if (map.getEntityAt(position) instanceof Herbivore) {
      System.out.print("🦌");
    }
    //...
  }
}

//ПРАВИЛЬНО:
private static final String PREDATOR = "\uD83D\uDC3A";
//...

public void Process(Map map) {
  //...    
  if (!map.isCellFree(position)) {
    Entity entity = map.getEntityAt(position);
    String sprite = toSprite(entity);

    System.out.print(sprite);
    //...
  }
}

private String toSprite(Entity entity) {
  if (entity instanceof Predator) {
    return PREDATOR; 
  }
  if (entity instanceof Herbivore) {
    return HERBIVORE; 
  }
  //...

  throw new IllegalArgumentException(<message>);  //попытка распечатать неизвестное существо
}
```

**14. class Simulation**

- Нарушение паттерна GRASP "Creator"(Создатель)

Здесь класс принимает в конструктор зависимости, которые должен создавать самостоятельно:
```java
public Simulation(Map map, List<Action> initAction, List<Action> turnAction, Renderer renderer, PathFinder pathFinder) {...}

//ПРАВИЛЬНО:
public Simulation(GameMap gameMap, List<Action> initActions, List<Action> turnActions) {...}
  //...
  this.renderer = new Renderer();
  this.pathFinder = new PathFinder(gameMap);
}
```
Creator гласит, что создавать объект должен тот, кто его использует. 

Рассмотрим, что это значит.

В данном случае `Simulation` принимает в конструктор `GameMap`- это правильно.  
Потому что карта может быть создана разных размеров и `Simulation` заранее не знает, какая именно карта нужна в этот раз.  
Поэтому ее создает клиент, а потом инжектит в этот класс.

Еще `Simulation` принимает в конструктор списки экшенов- это тоже правильно.  
Потому что список экшенов может быть разным. 

Но принимать в конструктор `Renderer` и `PathFinder`- уже неправильно.  
Потому что эти классы не параметризуются и `Simulation` их может создать самостоятельно.  
`PathFinder` параметризуется картой, но это может сделать сам `Simulation`.
А экземпляр `Renderer` вообще всегда одинаковый.  

Поэтому, согласно паттерну Creator, эти объекты *здесь* должен создавать сам класс Simulation.

Технически, сейчас можно создать наследника `Renderer`, переопределить в нем методы и передавать в конструктор Simulation.  
Но если программист хочет использовать в своем классе другие классы через полиморфизм, то он должен обозначить свои намерения более явно.  
Например, передавать в конструктор интерфейс или абстрактный класс:
```java
public class Simulation {
  private final Renderer renderer;  
  //...

  public Simulation(..., Renderer renderer) {
    //...
    this.renderer = renderer;
  } 
}

public interface Renderer {
  void render(GameMap gameMap);
}

public class EmojiRenderer implements Renderer {
  
  @Override
  public void render(GameMap gameMap) {
    //распечатывает существ в виде ЭМОДЖИ
  }
}

public class TextRenderer implements Renderer {
  
  @Override
  public void render(GameMap gameMap) {
    //распечатывает существ в виде БУКВ
  }
}
```

- Избыточно. 

Используй `foreach`
```java
for (int i = 0; i < initAction.size(); i++) {
  initAction.get(i).toPerform(map, pathFinder);
}

//ПРАВИЛЬНО:
for (Action action : initActions) {
  action.toPerform(map, pathFinder);
}
```

- Не совмещай инкрементирование с другими операциями
```java
System.out.println(stepcounter++);

//ПРАВИЛЬНО:
System.out.println(stepcounter);
stepcounter++;
```

Потому что у инкремента есть две разные формы, префиксная и постфиксная, они работают по-разному и можно запутаться, как они в таких случаях отработают вместе с распечаткой:
```java
int stepcounter = 10;

System.out.println(stepcounter++);
System.out.println(++stepcounter);
```

- Здесь не может вылететь исключение
```java
public void pauseSimulation() throws InterruptedException {
  running = false;
}
```

**15. class Main**

Содержит точку входа main.

+ 👍 Только создает и запускает Симуляцию, это хорошо.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

Сделать паузу/пуск во время работы.

Посмотри на ютубе ролики Немчинского для новичков:
```
"Правильные методы по Clean Code"
"Как называть переменные, методы и классы? Чистый код"
"Принцип хорошего кода KISS"
"Принцип хорошего кода DRY"
```

И отдельно про SOLID- по одному ролику на каждый принцип:
```
"SOLID принципы: SRP"
"SOLID принципы: OCP"
etc 
```

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.171(371)  
#ревью #симуляция 