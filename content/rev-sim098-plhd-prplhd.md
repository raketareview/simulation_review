https://github.com/prplhd/phSimulation  
[plhd]

В целом ок.

## ХОРОШО

+ 👍 Прикольная заставка  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim098/img0.png)

+ 👍 На старте лаконично сообщает правила 
```
Это пошаговая симуляция реального мира, где 🦊 хищники охотятся на 🐇 травоядных, а те питаются 🌱 травой.
Желаю интересного наблюдения!
```

+ 👍 Прятная цветовая гамма карты  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim098/img1.png) 

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Широко применяются константы
+ 👍 Реализовано пауза/пуск во время работы
+ 👍 Цветовая индикация здоровья существ
+ 🚀 Меню для создания карт разного размера

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название должно как можно лучше объяснять суть явления. Данном случае поисходит складывание(или смещение) координат
```
public interface Direction {
    //...
  default Coordinate apply(Coordinate coordinate) {
    return new Coordinate(coordinate.x() + dx(), coordinate.y() + dy());
  }
}
//ЛУЧШЕ:
public interface Direction {
    //...
  default Coordinate add(Coordinate coordinate) {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Утилитные классы**

👍 Сделаны правильно: final, имеют только статические методы и приватный конструктор.

**3. record Coordinate**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```
public record Coordinate(int x, int y) {
}
```

**4. interface Direction**

Интерфейс для хранения направления-смещения
```
public interface Direction {
  int dx();
  int dy();

  default Coordinate apply(Coordinate coordinate) {
    return new Coordinate(coordinate.x() + dx(), coordinate.y() + dy());
  }
}
```
В проекте уже есть сущность, которая хранит пару x/y, это класс Coordinate. 
Не нужно иметь вторую дублирующую сущность в виде этого интерфейса.

Если убрать из проекта `interface Direction`, то метод складывания координат можно перенести в саму координату, например, так:
```
public record Coordinate(int x, int y) {
  public Coordinate add(Coordinate other) {
    return new Coordinate(x + other.x(), y + other.y());
  }
}
```

Координата должна использоваться для обозначения сдвига-смещения вот так:
```
Coordinate shiftUpCoordinate = new Coordinate(0, 1);
```

**5. enum AxisDirection/DiagonalDirection implements Direction**

В связи с прошлым замечанием, предлагаю переосмыслить `AxisDirection/DiagonalDirection`.

Итак, идея в том, что разные существа могут ходить по разным направлениям: одни только по диагоналям, а другие только по вертикалям.
Нужно комплекты направлений хранить в одном месте.  
Работать с этими комплектами нужно одинаково, поэтому для полиморфизма нужно ввести интерфейс:
```
public interface Direction { 
  List<Coordinate> get();
}
```

И его реализации:
```
public class AxisDirection implements Direction {
  private final static List<Coordinate> SHIFT_COORDINATES = List.of(
      new Coordinate(0, 1),
      new Coordinate(1, 0),
      new Coordinate(0, -1),
      new Coordinate(-1, 0)
  );

  @Override
  public List<Coordinate> get() {
    return SHIFT_COORDINATES;
  }
}

public class DiagonalDirection implements Direction {...}
```

**6. class WorldMap**

+ 👍 Хорошо, что класс возвращает копию мапы, а не оригинал- тем самым соблюдает инкапсуляцию, защищает внутреннее устройство класса от потенциальных повреждений.
```
public Map<Coordinate, Entity> getEntitiesCopy() {
  return new HashMap<>(entities);
}
```

- Название `public Map<Coordinate, Entity> getEntitiesCopy()` неудачное- оно подразумевает, что клиент знает подробности внутреннего устройства класса. 
И то, что класс хранит отношения координата-сущность в виде мапы, копию которой он может вернуть.

Лучше назвать метод `public Map<Coordinate, Entity> toMap()`- это название избавляет клиентский код от необходимости знать подробности внутреннего устройства класса.
А из названия публичного метода станет ясно, что как бы внутри себя `WorldMap` ни хранил координаты + сущности, он может их сообщить в виде мапы.  
В аналогичных случаях, когда класс возвращает копию листа, метод принято называть `toList()`.

+ 👍 При всех операциях с участием координаты, проводится валидация координаты на предмет нахождения ее в пределах карты.

+ 👍 Геттер не возвращает null, а для возврата существ использует `Optional`, это хорошо
```
public Optional<Entity> getEntity(Coordinate coordinate) {...}
```

+ 👍 С точки зрения SRP, класс хороший, нет ничего лишнего.

**7. interface Pathfinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, AStar и т.д.

- Как пользоваться поиском- неясно. Здесь метод поиска пути получает в аргументы только точку старта
```
public interface Pathfinder {
  List<Coordinate> findPath(Coordinate start);
}
```
Поиск должен искать путь от точки старта до точки, соответствующей заданным условиям.
В данном случае- до точки, в которой находится существо нужного класса.
Например, так
```
List<Coordinate> findPath(Coordinate start, Class<? extends Entity> target);
```

**8. class BreadthFirstSearchPathfinder implements Pathfinder**

- Здесь становится понятно, как нужно пользоваться поиском- для этого в конструктор класса передается тот самый таргет, про который я писал выше
```
public BreadthFirstSearchPathfinder(WorldMap worldMap, Class<? extends Entity> target) {
  this.worldMap = worldMap;
  this.target = target;
}
```
Тоже вариант, конечно. Но этот подход имеет некоторый недостаток: теперь в проекте для каждого вида ходячих существ нужно иметь персональный экземпляр класса поиска. 
Потому что для зайчика нужно делать экземпляр поиска и передавать ему в конструктор таргет травы. А для волка- экземпляр поиска с таргетом зайчика.
Если в проекте будет тридцать видов хадячих существ, то для них придется делать тридцать персонализированных екземпляров поиска.

Если же принимать таргет в аргументы метода поиска, а не в аргументы конструктора класса, то любое существо может пользоваться одним и тем же экземпляром класса поиска.

- Мапа координата-координата обычно при реализации поиска используется как эрзац-заменитель связного списка
```
Map<Coordinate, Coordinate> cameFrom = new HashMap<>();
```
Вместо мапы координата-координата сделай класс связного списка, алгоритм тогда будет выглядеть понятнее.
Например, так:
```
class Node {
  private Coordinate coordinate; 
  private Node parent;
  //...
  public Coordinate getCoordinate() {...}
  public Node getParent() {...}
}
```

+ 👍 Если поиск не находит путь, то возвращает пустой список, это хорошо
```
public List<Coordinate> findPath(Coordinate start) {
  //...
  return List.of();
}

//ДО JAVA 9 ЭТО ДЕЛАЛОСЬ ТАК:
return Collections.emptyList();
```

**9. abstract class Entity и его простые наследники Tree/Rock/Grass**

👍 Идеально
```
public abstract class Entity {
}

public class Tree extends Entity {
}
```

**10. abstract class Creature extends Entity**

- Если существо не хранит в себе координату, но знает Карту, то свою координату существо должно получить из карты, а не принимать ее в метод хода от третьих лиц 
```
public void makeMove(WorldMap worldMap, Coordinate currentPos) {
  //...  
}

//ПРАВИЛЬНО:
public void makeMove(WorldMap worldMap) {
   Coordinate coordinate =  worldMap.getCoordinate(this);
  //...  
}
```
Потому что если метод хода получает во входящие аргументы текущую координату своего положения на карте, это потенциально опасная ситуация: 
никто не гарантирует, что клиентский код передаст корректную стартовую координату.

- Вот тут и проявляется недостаток енама направления- креатура использует конкретный набор направлений 
```
for (Direction direction : AxisDirection.values()) {...}
```
В то время как потомки креатуры, использующие другие направления(Predator), вынуждены переопределять метод под свой набор направлений. Тем самым дублируется код.

Предлагаю сделать сборник направлений так как я предлагал в предыдущих замечаниях. 
И тогда креатура будет хранить этот сборник направлений в виде поля, и использовать любые направления одинаково через полиморфизм.  
Общая идея такая:
```
public abstract class Creature extends Entity {
  protected final Direction direction;
  //...

  protected Creature(int speed, int maxHp, Direction direction, Class<? extends Entity> target) {...}
}

public final class Predator extends Creature {
  private final static Direction DIRECTION = new DiagonalDirection();  
  private final int attackPower;

  public Predator(int speed, int maxHp, int attackPower) {
    super(speed, maxHp, DIRECTION, Herbivore.class);
    this.attackPower = attackPower;
  }
  //...
}
```

**11. class Herbivore/Predator extends Creature**

- Есть ли фактическия разница между `int attackPower` и `int hpRestoredPerGrass`? Кажется, это одно и то же
```
public class Herbivore extends Creature {
  private final int hpRestoredPerGrass;
  //...

  @Override
  protected void interactWithTarget(WorldMap worldMap, Coordinate currentPos, Coordinate targetPos) {
    //...
    hp += hpRestoredPerGrass;
    if (hp > maxHp) {
      hp = maxHp;
    }
  }
}

public final class Predator extends Creature {
  private final int attackPower;
  //...

  @Override
  protected void interactWithTarget(WorldMap worldMap, Coordinate currentPos, Coordinate targetPos) {
    //...
    hp += attackPower;
    if (hp > maxHp) {
      hp = maxHp;
    }
  }
}
```
Общий код и общие поля выноси в предка. В данном случае- в креатуру.

- Нарушение DRY. Дублирование кода между `Predator` и `Creature`
```
public abstract class Creature extends Entity {
  //...
  protected Optional<Coordinate> findTargetNearby(WorldMap worldMap, Coordinate currentPos) {
    for (Direction direction : AxisDirection.values())
    //ОТСЮДА И НИЖЕ- ОДИНАКОВЫЙ КОД
  }
}

public final class Predator extends Creature {
  //...
  protected Optional<Coordinate> findTargetNearby(WorldMap worldMap, Coordinate currentPos) {
    //...
    for (Direction direction : DiagonalDirection.values()) 
    //ОТСЮДА И НИЖЕ- ОДИНАКОВЫЙ КОД 
  }
}
```
Общий код выноси в предка.

**12. class MaintainPopulationAction implements Action**

Нарушение DRY
```
if (currentHerbivoreCount < cfg.getHerbivoreMinCount()) {
  //одинаковый код
}

if (currentGrassCount < cfg.getGrassMinCount()) {
  //одинаковый код
}
```

**13. class MoveAllCreaturesAction/PopulateWorldAction implements Action**

👍 Все ок.

**14. class SimulationPreset**

- Стандартным явлениям давай стандартные названия. Этот загадочный `Preset` ничто иное, как простая фабрика.  
Также, в подобных случаях вместо констант примитивных типов используй перечисления
```
public final class SimulationPreset {
  public static final int SMALL_WORLD_KEY = 1;
  public static final int MEDIUM_WORLD_KEY = 2;
  public static final int LARGE_WORLD_KEY = 3;

  public static SimulationConfig getForSize(int size) {...}
}

//ПРАВИЛЬНО:
public final class SimulationConfigFactory {
  public static SimulationConfig get(Size size) {...}

  public enum Size {
    SMALL, MEDIUM, LARGE;
  }
}
```

- Я в рабочих проектах видал адские свич-кейсы на полторы тысячи строк, но здесь он тоже слишком большой.
Switch-case должен быть компактым
```
return switch (size) {
  case SMALL_WORLD_KEY -> new SimulationConfig.Builder()
    //27 строк

  case MEDIUM_WORLD_KEY -> new SimulationConfig.Builder()
    //27 строк

  case LARGE_WORLD_KEY -> new SimulationConfig.Builder()
    //27 строк

  default -> throw new UnknownWorldSizeException(size);
};

//ПРАВИЛЬНО:
return switch (size) {
  case SMALL_WORLD_KEY -> createSmallSimulationConfig();
  case MEDIUM_WORLD_KEY -> createMediumSimulationConfig();
  case LARGE_WORLD_KEY -> createLargeSimulationConfig();
  default -> throw new UnknownWorldSizeException(size);
};
```
*"ЧК", гл.3, "Команды Switch"*

**15. пакет dialogs**

👍 Знакомые диалоги.

**16. class UnknownEntityException extends SimulationException**

Сообщение в исключении принято делать в одну строку 
```
public class UnknownEntityException extends SimulationException {
  public UnknownEntityException(String entityType) {
    super("""
        
        Unknown entity type: %s.
        """.formatted(entityType));
  }
}

//ПРАВИЛЬНО:
public class UnknownEntityException extends SimulationException {
  public UnknownEntityException(String entityType) {
    super("Unknown entity type: %s.".formatted(entityType));
  }
}
```
Здесь сообщение в несколько строк, первые из которых пустые. 
Это ты подогнал исключение *под способ показа информации в консоли*, который соответствует твоим эстетическим представлениям. 
Но это противоречит принятому стандартному подходу.

**17. class InvalidMoveException extends SimulationException**

Класс описывает две разных исключительных ситуации. Класс нужно разделить на два.

**18. interface WorldMapRenderer**

👍 Интерфейс рендерера это хорошо. Теперь можно делать разные рендереры для разных визульных сред(консоль, интерфейс виндовс, http, матричный принтер etc)
и разного отображения информации(цветной, черно-белый и пр.)

**19. class ConsoleWorldMapRenderer implements WorldMapRenderer**

- Особые случаи обрабатывай сразу
```
Entity entity = worldMap.getEntity(coordinate).orElse(null);

if (entity != null) {
  //миллион строк
}
return SpriteType.EMPTY.getSprite();

//ПРАВИЛЬНО:
Entity entity = worldMap.getEntity(coordinate).orElse(null);

if (entity == null) {
  return SpriteType.EMPTY.getSprite();  
}

//миллион строк
```

- Ну и в чем тогда смысл использовать `Optional` таким образом, что в `Entity entity` по итогу все равно попадает null и приходится сравнивать entity c null'ом? 🤷‍♀🤷‍♀🤷‍♀
```
Entity entity = worldMap.getEntity(coordinate).orElse(null);
if (entity != null) {...}

//ПРАВИЛЬНО:
Optional<Entity> optional = worldMap.getEntity(coordinate);
if(optional.isEmpty()) {
  return SpriteType.EMPTY.getSprite();
} 
Entity entity = optional.get();
```

- Если константы енама со спрайтами называются аналогично классам энтити, то алгоритм можно упростить
```
return switch (entity.getClass().getSimpleName()) {
  case "Predator" -> SpriteType.PREDATOR.getSprite();
  case "Herbivore" -> SpriteType.HERBIVORE.getSprite();
  //...
};

private enum SpriteType {
  EMPTY("\u2B1B"),
  PREDATOR("\uD83E\uDD8A"),
  HERBIVORE("\uD83D\uDC07"),
  //...
}

//ПРАВИЛЬНО:
String name = entity.getClass().getSimpleName().toUpperCase();
return SpriteType.valueOf(name).getSprite();
```

- Нарушение SRP. Вот эти уровни рендерер должен не устанавливать самостоятельно, а принимать в конструктор:
```
private static final double HEALTH_CRITICAL_THRESHOLD = 0.33;
private static final double HEALTH_WARNING_THRESHOLD = 0.66;
```
Потому что ответственность рендерера- просто печатать данные. Решать, какой уровень здоровья является критическим, это ответственность игровой логики, а не представления данных.  
Рендерер может только самостоятельно решать, каким цветом он будет показывать ослабших существ- красным, фиолетовым или еще каким-то. 

**20. class Simulation**

Рендерер нужно принимать в конструктор
```
public Simulation(SimulationConfig cfg) {
  this.worldMapRenderer = new ConsoleWorldMapRenderer();
  //...
}

//ПРАВИЛЬНО:
public Simulation(SimulationConfig cfg, WorldMapRenderer worldMapRenderer) {
  this.worldMapRenderer = worldMapRenderer;
  //...
}
```
Сейчас Simulation самостоятельно инициализирует `WorldMapRenderer`, тем самым делая из интерфейса `WorldMapRenderer` карго культ. 
Потому что сейчас нет возможности в Simulation пользоваться разными реализациями этого интерфейса- в нем жестко прописано использование конкретного рендерера `ConsoleWorldMapRenderer`.

Для тренировки полиморфизма можешь сделать еще одну реализацию рендерара. Например который будет распечатывать существ не экартинками, а в виде одиночных букв. 
И сделать два майна: один майн будет запускать приложение, которое будет печатать существ картинками, а второй майн будет распечатывать существ буквами.

**21. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, в данном случае- через Launcher, это хорошо.

## ВЫВОД

В целом, ок. Видно, что изучал лучшие практики.

n.98(220)  
#ревью #симуляция #диалоги 