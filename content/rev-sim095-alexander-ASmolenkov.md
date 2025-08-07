https://github.com/ASmolenkov/Simulation  
[Александр]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Программа сразу стартует и юзер не успевает увидеть меню с командами.
Нужно стартовать только по команде.

2. Длинные команды: start/stop. Ввод команд и рисование карты используют одну и ту же консоль. 
Поэтому игрок между циклами распечатки карты не всегда успевает набрать и ввести длинные команды.
Делай короткие команды, например: 0/1. 

## ХОРОШО

+ 👍 Классный read.me 
+ 👍 Ровная распечатка карты и красивое меню  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim095/img0.png)
+ 👍 Механика голода у существ
+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Два алгоритма поиска: AStart и BFS
+ 👍 Для информирования о действиях моделей используется паттерн "Подписчик"
+ 👍 Широко применяются константы

## ЗАМЕЧАНИЯ

**1. Нейминг**

+ 👍 Хорошо, что шаблоны подписаны "_TEMPLATE"- становится понятно, что это не просто текст, а шаблон для текста
```
static final String EXCEPTION_COORDINATE_OCCUPIED_TEMPLATE = "Coordinate %s is already occupied";
```

- Координата описывает положение точки в пространстве, а не размеры геометрической фигуры
```
record Coordinate(int width, int height)
//ПРАВИЛЬНО: row/column или x/y.
```

- По правилам английского языка должно быть `WorldMap`
```
class MapWorld

//ПРАВИЛЬНО:
class WorldMap
```

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```
Map <Coordinate, Entity> entityPositionMap;

//ПРАВИЛЬНО:
Map <Coordinate, Entity> entityPositions;
```

- Для одной концепции используй одно название
```
Coordinate position;

//ПРАВИЛЬНО:
Coordinate coordinate;
```

- Старайся использовать стандартные термины. Добавить- add, вычесть- subtract
```
void plusHealth(int plusHealth);
void minusHealth(int health)

//ПРАВИЛЬНО:
void addHealth(int health);
void subHealth(int health)
```

- Название класса должно быть существительным
```
class SimulationWelcomePrint

//ПРАВИЛЬНО:
class SimulationWelcomePrinter
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Избыточно**
```
coordinate.width() <= width - 1

//ПРАВИЛЬНО:
coordinate.width() < width
```

**3. record Coordinate(int width, int height)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**4. class MapWorld**

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
private final Map<Coordinate, Entity> entityPositionMap;
  
public Map<Coordinate, Entity> getEntityPositionMap() {
  return entityPositionMap;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта(100, 100);
<заселить карту существами>
карта.getEntityPositionMap().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

Тут есть два варианта, как сделать правильно:  
Либо вернуть не оригинал мапы, а ее копию: `return new HashMap<>(entityPositionMap);`  
Либо возвращать существ персонально:
```
public Entity get(Coordinate coordinate)
public Coordinate getCoordinate(Entity entity)
```
Мне больше нравится второй вариант, когда существа возвращают персонально. Но ты, исходя из своего алгоритма, можешь выбрать любой вариант из двух. 

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public boolean isPositionAvailable(Coordinate position) {
  return !entityPositionMap.containsKey(position);
}
```
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что свободна. 
А правильный ответ- такой координаты в карте вообще нет.

- Нарушение SRP. Расcчет площади не имеет отношения к единой ответстсвенности карты
```
public int getSize() {
  return width * height;
}
```
Если в карте есть метод, который считает ее площадь, то почему нет методов, которые считают периметр карты и диаметр вписаной окружности?
Потому что это не надо алгоритму игры в целом, а площадь- надо. Но *самой карте* не нужно знать ни свою площадь, ни периметр, а нужно знать только свои ширину и высоту.  

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
Entity заяц = new Заяц(new Coordinates(0, 0));
карта.addEntity(заяц);
карта.moveEntity(заяц, new Coordinates(99, 99));

/* class MapWorld */
void updatePosition(Creature creature, Coordinate newPosition) {
  //13 строк кода
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- В методе телепортации есть очень подозрительный код- по старой координате существа перед его перемещением записывается некая сущность "Пустое место". 
```
public void updatePosition(Creature creature, Coordinate newPosition) {
  //
  entityPositionMap.put(creature.getPosition(), new EmptyArea(creature.getPosition()));
  //...
  entityPositionMap.put(newPosition, creature);
}
```
Смысл этого действия мне не понятен. Мы используем HashMap как раз для того, чтобы хранить только актуальную информацию "координата-существо".
Хранить "координата-пустое место" так же бессмысленно, как хранить "координата-null".

В дальнейшем, как мы увидим, при создании карты ВСЕ ее пустые ячейки заполняются заглушками `EmptyArea` и тем самым отпадает всякий смысл в использовании HashMap для хранения существ.

Потому что сейчас HashMap становится аналогом обычного массива, только хуже, бесполезно забивая память.
Например, есть игровая доска 100*100 и в ней единомоментно находится общим числом 150 разных существ: камней, травы, зайцев и т.д.
Тогда при нормальном хранении этих данных в мапе с помощью ключ-значение, потребуется 150 записей.
Если в этой же мапе кроме этих значимых данных хранить еще пустые сущности, то потребуется уже 10.000 записей.

Если нужно очистить старое место в хешмапе после ухода существа с координаты, нужно удалить существо со старой координаты и вставить на новую координату
```
entityPositionMap.remove(creature.getPosition());
//...
entityPositionMap.put(newPosition, creature);
```

- Как только уберешь отсюда метод телепортации, то и паттерн "Слушатель" придется отсюда убрать и перенести его либо в соответствующий Action, либо в Creature.

+ 👍 В целом, с *точки зрения SRP*, класс хороший. Нюансы с площадью и телепортацией- несущественные.
Заполнение пустых мест карты заглушками `EmptyArea` это конечно ужасающе, но  это относится к недостаткам алгоритма, а не к нарушению SRP класса.

**5. interface Pathfinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, AStar и т.д.
+ 👍 Сделано две ерализации интерфейса: поиск BFS и AStar, это очень хорошо.

- Нарушение SRP.

Поиск должен просто искать путь от точки старта до точки, соответствующей заданным условиям.  
В данном случае- до точки, в которой находится существо нужного класса.
Если поиск получает во входящие координату цели, значит часть работы поиска уже выполнил кто-то другой. 
То есть, кто-то другой уже нашел цель и передал ее координаты в поиск пути
```
List<Coordinate> findPathToTarget(Coordinate start, Coordinate target);
```
На самом деле поиск пути должен сам искать цель и прокладывать к ней путь. 
Сигнатура метода для этого должна выглядеть примерно так
```
List<Coordinate> find(World world, Coordinate start, Class<? extends Entity> target) {
  //ищет путь на карте world от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```
Конечной целью(target) должен быть класс еды, а не координата с едой.

**6. class BFSExplorer**

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем анализа принадлежности Entity тому или иному виду существ
```
return entity == null || entity instanceof EmptyArea || entity instanceof Grass || entity instanceof Herbivore;
```

- Смещения нужно вынести из метода и сделать константами.  

Потому что сейчас при вызове метода `getNeighbors(...)` каждый раз пересоздается объект(а массив это объект) со смещениями. 
И, что такое пара х/у? Правильно, координата
```
private List<Coordinate> getNeighbors(Coordinate coordinate) {
  int[][] directions = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
  //...
}

//ПРАВИЛЬНО:
private static final List<Coordinate> SHIFT_COORDINATES = List.of(
    new Coordinate(0, 1),
    new Coordinate(1, 0),
    //oth's coord's
);

private List<Coordinate> getNeighbors(Coordinate coordinate) {
  //...
}
```

- Какая-то непонятная конструкция
```
BiConsumer<Coordinate, Map<Coordinate, Coordinate>>
```
Обычно мапу "координата-координата" исользуют как эрзац-заменитель однонаправленного связного списка. 
Вместо такой мапы используй ноды однонаправленного связного списка.

- Нарушение SRP. Класс анализирует существо на принадлежность к тому или иному классу. 
Класс не должен этого делать, для поиска пути этого не нужно и если он это все-таки делает, значит выполняет часть чей-то чужой логики
```
return entity == null || entity instanceof EmptyArea || entity instanceof Grass || entity instanceof Herbivore;
```
Поиск пути должен искать путь по пустым клеткам карты от точки старта до точки, где находится цель определенного класса. 
Это должно быть передано в аргументы метода поиска, как я указал в замечаниях к `interface Pathfinder`.

Если поиск более продвинутый, например, если хищник может ходить не только по пустым клеткам, но и по траве, 
то список этих "проходимых" поверхностей поиск тоже должен принимать в аргументы метода, а не определять их самостоятельно.

- Эта константа относится к внутреннему классу
```
public class BFSExplorer {
  private static final String TARGET_FOUND = "BFS: Target found, early exit";
  //...
  protected static class BFSResultFoundException extends RuntimeException {
    public BFSResultFoundException() {
      super(TARGET_FOUND);
    }
  }
}

//ПРАВИЛЬНО:
public class BFSExplorer {
  //...
  protected static class BFSResultFoundException extends RuntimeException {
    private static final String MESSAGE = "BFS: Target found, early exit";

    public BFSResultFoundException() {
      super(MESSAGE);
    }
  }
}
```

**7. class Node**

Это нода для поиска A* и в этом качестве к ней нет притензий. 

Но для поиска BFS тоже, вероятно, нужна нода, чтобы не пользоваться мапой "координата-координата".
Использовать для BFS ноду от А* конечно можно, но выглядеть будет это несколько коряво- нода A* хранит в себе лишнюю для BFS информацию.  
Поэтому можно сделать семейство нод, примерно так:
```
public abstract class Node<T> {
  protected final Coordinate coordinate;
  protected T parent;

  public Node(Coordinate coordinate) {...}
  public Coordinate getCoordinate() {...}
  public T getParent()  {...}
  public void setParent(T parent)  {...}
}

public class BfsNode extends Node<BfsNode>{
  public BfsNode(Coordinate coordinate) {
    super(coordinate);
  }
}

public class AstarNode extends Node<AstarNode>{
  private double g;  // Стоимость пути от старта
  private double h;  // Эвристическая оценка до цели
  private double f;  // f = g + h
  //...
}
```

**8. Пакет entity**

Сейчас этот пакет находится в пакете `world`, но для этого нет оснований. Пакет entity должен быть таким же равноправным, как и пакет world.

**9. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. 
Поэтому entities должны хранить координату только начиная с уровня `Creature`.

- Нет оснований для того, чтобы метод был final
```
public final Coordinate getPosition()
```

- Нет оснований для того, чтобы конструктор этого класса был protected
```
  protected Entity(Coordinate position) {
    this.position = position;
  }
```

**10. class EmptyArea extends LandScape**

Как я писал выше, не должно быть вообще такого класса. 
Вернее, он может быть, если будет выполнять *только* роль паттерна NullObject.  
Например, когда у карты спрашивают существо по какой-то координате, а там пусто.  
Вот тогда геттер, чтобы не возвращать null, может вернуть экземпляр `EmptyArea`.  
*Фаулер "Рефакторинг", гл.9, "Введение объекта Null"*

**11. abstract class Creature extends Entity**

- Непонятно, зачем креатуре целых два класса, которые отвечают за поиск цели
```
protected final TargetFinder targetFinder;
protected final Pathfinder pathfinder;
```
При нормальной организации поиска, должно быть достаточно одного- Pathfinder. 
Этот класс должен отвечать и за наход цели и за прокладывания к нему пути.
Когда ты переделаешь поиск по-нормальному, в креатуре отпадет необходимость держать несколько классов, связанных с поиском.

- Метод, который определяет, что еда рядом. 
Ок, но почему этот метод такой большой? Судя по тому, что делает метод, он в некотором роде на дистанции в 1 шаг дублирует логику поиска пути.
У тебя уже есть поиск пути, используй его
```
protected boolean isTargetNearby(MapWorld mapWorld, Coordinate pos, Class<?> entityType) {
  //14 строк  
}

//ПРАВИЛЬНО:
protected boolean isTargetNearby(MapWorld mapWorld, Coordinate pos, Class<?> entityType) {
  List<Coordinate> путь = поискПути.найти(mapWorld, pos, entityType);
  return путь.size() == 1; //в списке координат всего 1 координата, значит еда рядом
}
```
Возможно, расписать `isTargetNearby(...)` на такое количество строк ты решил для оптимизации скорости работы алгоритма. 
В таком случае это антипаттерн "Преждевременная оптимизация". 
Оптимизация скорости работы очень важна в геймдейве, но там другие приоритеты.

В этом проекте приоритет- хорошая архитектура, а не оптимизация скорости работы. 
Потому что оптимизация скорости приводит к ухудшению архитектуры, как в этом случае. 
А выгоды от такой оптимизации юзер здесь все равно не заметит- большую часть процессорного времени такая простая программа будет простаивать.

**12. class Rabbit extends Herbivore**

- Нарушение DRY, дублирование кода в методе `void plusHealth(int plusHealth)` у Зайца и Волка. 
Общий код выноси в предка
```
public class Wolf extends Predator {
  //...
  @Override
  public void plusHealth(int plusHealth) {
    this.health += plusHealth;
    if (this.health > MAX_HEALTH) {
      this.health = MAX_HEALTH;
    }
  }
}

//ПРАВИЛЬНО:
public class Creature бла-бла {
  //...
  @Override
  public void plusHealth(int health) {
    this.health += health;
    if (this.health > getMaxHealth()) {
      this.health = getMaxHealth();
    }
  }
}
```

- Неправильное использование паттерна "Подписчик".

Сейчас во время события передается  строка с сообщением о том, что там произошло
```
protected void notifyAttack(MapWorld world, Creature target, Coordinate targetPosition) {
  String message = String.format("⚔️ %s attacked %s at %s",
    getClass().getSimpleName(),
    target.getClass().getSimpleName(),
    position);
  world.notifyListeners(new SimulationEvent(EventType.ATTACK, message, this));
}
```
Представь, что программа локализируется на несколько языков. Как в таком случае предусмотреть язык передаваемого сообщения и т.д.? 
То есть, мадель(а бизнес логика евента это модель) становится зависима от представления.

В таких случаях нужно передавать не конкретное сообщение, а параметры произошедшего события.  
Например, так:
```
protected void notifyAttack(MapWorld world, Creature target, Coordinate targetCoordinate) {
  Event event = new AttackEvent(this, target, targetCoordinate);  
  Sworld.notifyListeners(event);
}
```
Получив такое сообщение, подписчик должен с ним сделать то, что ему нужно. 
Например, подставить информацию из евента в текстовое сообщение и распечатать его в консоль.  
Или в реализации на Swing'e или Андроиде воспроизвести анимацию атаки и т.д.

**13. enum CreatureType**

Енам выполняет роль внешней типизации существ
```
public enum CreatureType {
  PREDATOR, HERBIVORE
}
```
Не нужно использовать дублирующую систему типизации, потому что у объекта есть стандартные способы для этого- `Class getClass()`.

**14. class RabbitFactory implements CreatureFactory<Rabbit>**

- Не понял, зачем в классе одновременно существует и конструктор и фабричный метод. 
Сделай или два перегруженных фабричных метода или два перегруженных конструктора
```
public class RabbitFactory implements CreatureFactory<Rabbit> {
  private final HerbivoreConfig defaultConfig;

  public RabbitFactory(HerbivoreConfig defaultConfig) {
    this.defaultConfig = Objects.requireNonNull(defaultConfig);
  }

  public static RabbitFactory withDefaultConfig(){
    return new RabbitFactory(new HerbivoreConfig.Builder().setSpeed(DEFAULT_SPEED).setHealth(DEFAULT_HEALTH).setSatiety(DEFAULT_SATIETY).setMaxSearchDepth(DEFAULT_MAX_SEARCH_DEPTH).build());
  }
  //...
}

//ПРАВИЛЬНО:
public class RabbitFactory implements CreatureFactory<Rabbit> {
  private static final HerbivoreConfig DEFAULT_CONFIG = new HerbivoreConfig.Builder().setSpeed(DEFAULT_SPEED).<...>.build();
     
  private final HerbivoreConfig config;

  public RabbitFactory(HerbivoreConfig config) {
    this.config = Objects.requireNonNull(config);
  }

  public RabbitFactory() {
    this(DEFAULT_CONFIG);
  }
  //...
}
```

**15. class GenerateLandscapeAction implements Action**

- Первоначальное заполнение карты статичными существами и заглушками "Пустое место", про которые я писал выше. 
Этот класс нужно совместить с `class GenerateCreatureAction`. 
Будущий совмещенный класс должен выполнить первоначальное заселение карты всеми видами существ- ходячими и неходячими.

- Сложная логика заселения существ. Заселить существа очень легко примерно по такому принципу:
```
public class SpawnAction extends Action {
  private final static int КОЛИЧЕСТВО_ТРАВЫ = 10;
  
  //...  
  private final Supplier<Entity> entitySupplier;
  private final int amount;

  public SpawnAction(Supplier<Entity> entitySupplier, int amount) {
    this.entitySupplier = entitySupplier;
    this.amount = amount;
  }

  @Override
  public void execute(Карта карта) {
    spawn(карта, (coordinates) -> new Grass(coordinates), КОЛИЧЕСТВО_ТРАВЫ);
    spawn(карта, (coordinates) -> new Rock(coordinates), КОЛИЧЕСТВО_КАМНЕЙ);
    //...
  }

  private void spawn(Карта карта, Function<Coordinates, Entity> mapper, int amount) {
    for (int i = 0; i < amount; i++) {
      Coordinates coordinates = getRandomEmptyCoordinates(карта);
      Entity entity = mapper.apply(coordinates);
      карта.put(entity, coordinates);
    }
  }

  private Coordinates getRandomEmptyCoordinates(Карта карта) {
    //находит и возвращает случайную пустую координату
  }
}
```
Если нужно заселять не конкретное количество, а процент, то тоже не проблема- сначала рассчитываем количество для заселения, потом заселяем это количество на карту.

**16. class AddingCreatureAction/AddingGrassAction implements Action**

Нарушение DRY. Два класса, которые делают одно и то же- восполняют ресурсы определенного вида на карте.
Это должен быть один класс, который можно сделать разными способами. 
Например, он должен просто принимать в себя тип существа, максимальный процент не карте и минимальный процент- триггер, по которому запустится восполнение ресурсов.  
Что-то типа такого:
```
public class RespawnAction implements Action {
  private final Class type;
  private final int minAmount;
  private final int maxAmount;

  public RespawnAction(MapWorld mapWorld, Class type, int minPercent, int maxPercent) {
    //...
    minAmount = ...//посчитать исходя из minPercent
    maxAmount = ...//посчитать исходя из maxPercent
  }

  @Override
  public void perform() {
    //если количество существ класса type
    //на карте меньше minAmount
    //то дополнительно заселить на карту существ
    // этого класса
    //в количестве = maxAmount - minAmount
  }
}
```

**17. class MoveCreaturesAction  implements Action**

+ 👍 Только инициирует движение- пинает креатур, а дальше они двигаются сами, это хорошо
```
@Override
public void perform() {
  new HashMap<>(mapWorld.getEntityPositionMap()).forEach((coordinate, entity) -> {
    if (entity instanceof Creature creature) {
      creature.performMovementAction(mapWorld);
    }
  });
}
```
Не знаю, правда, зачем здесь `new HashMap<>()`, возможно только для того, чтобы запутать читателя.
Вот так было бы понятнее:
```
Map<Coordinates, Entity> entities = mapWorld.getEntityPositionMap();
entities.forEach((coordinate, entity) -> {
  if (entity instanceof Creature creature) {
    creature.performMovementAction(mapWorld);
  }
}
```
Кстати, вот здесь и проявляется, в том числе, недостаток хранения заглушек на месте пустых ячеек- 
для выявления креатур, цикл вынужден перебирать не только реальные существа, но и заглушки.

**18. class SimulationSettings**

Что это такое, я не могу понять.  
С одной стороны это, вроде бы, класс с настройками. Про такие классы я писал [ТУТ](https://t.me/zhukovsd_it_chat/53243/176984).

Но из 9 хранимых параметров, через геттеры можно получить только два. Как получить остальные, неясно. Кажется, никак.

**19. class SimulationWelcomePrint**

- Сложная система цветной печати
```
System.out.println(cyan + "║" + reset + WELCOME + cyan + "║" + reset);
```
Если делаешь цветную печать, выяви сущность- цвет:
```
public enum Color {
  RESET("\u001B[0m"),
  RED("\u001B[31m"),
  //...
  ;

  private final String code;
  public Color(String code) {...}
  public String getCode() {...}
}
```
Далее сделай методы, которые печатают цветом:
```
print(Color.CYAN, "║");
print(Color.RESET, WELCOME);
println(Color.CYAN, "║");

public void print(Color color, String s) {...} //
public void println(Color color, String s) {
   print(color, s);
   System.out.println();
}
```

Но то такое. Возможно, ты просто хочешь сохранить в коде красивый вид прямоугольников
```
System.out.println(cyan + "╔════════════════════════════════════╗" + reset);
System.out.println(cyan + "║" + reset + WELCOME + cyan + "║" + reset);
System.out.println(cyan + "╠════════════════════════════════════╣" + reset);
```

**20. class ConsoleRenderer implements Renderer**

Если пришло неизвестное существо, то нужно кидать исключение- значит в карте есть существо, спрайт которого мы не знаем. 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
private void printSprite(Entity entity) {
  String sprite = SPRITES.getOrDefault(entity.getClass(), EMPTY_AREA_SPRITE);
  System.out.print(sprite);
}

//ПРАВИЛЬНО:
private void printSprite(Entity entity) {
  String sprite = SPRITES.get(entity.getClass());
  if(sprite == null) {
    //кинуть исключение
  }
  System.out.print(sprite);
}
```

**21. class Simulation**

- Получает недостаточное количество заисимостей в конструктор.
Например, в проекте есть интерфейс Рендерер, который распечатывает карту. 
Этот рендерер Симуляция должна не самостоятельно инициализировать, а получать в конструктор, чтобы можно было в симуляцию инжектить любую его реализацию
```
private final ConsoleRenderer consoleRenderer;

public Simulation(MapWorld mapWorld) {
  this.consoleRenderer = new ConsoleRenderer();
  //...
}

//ПРАВИЛЬНО:
private final Renderer renderer;

public Simulation(MapWorld mapWorld, Renderer renderer) {
  this.renderer = renderer;
  //...
}
```
Стоит внимательно посмотреть и на другие интерфейсы, которые есть в проекте- как они используются и где они инициализируются в качестве зависимостей.

- Не прописывай исключения в сигнатуре методов. Проверяемые иссключения перехватывай и кидай вместо них непроверяемые
```
public void starSimulation() throws InterruptedException {
 //...
 Thread.sleep(TIME_PAUSE);
 //...
}

//ПРАВИЛЬНО:
public void starSimulation() {
  //...  
  try {
    Thread.sleep(TIME_PAUSE);
  } catch (InterruptedException e) {
    throw new RuntimeException(e);
  }
  //...
}
```
*"ЧК", гл.7, "Используйте непроверяемые исключения"*

**22. class Main**, содержит точку входа main

👍 Только создает и запускает симуляцию, это хорошо.

## ВЫВОД

В целом неплохо.  
Но в учебных проектах не увлекайся добавлением дополнительного функционала.
Потому что при этом неизбежно ухудшается архитектура, но за расширенным функционалом это становится не так заметно- масштабирование программы без ухудшения архитектуры это отдельное мастерство.  
А приоритет здесь все-таки состоит именно в отработке построения хорошой архитектуры, поэтому лучше меньше, но лучше.

n.95(213)  
#ревью #симуляция #подписчик 