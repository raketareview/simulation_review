https://github.com/xkodxdf/simulation   
[YanDa]

Огромное количество классов. Местами запутанно. Есть удачные решения.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Непонятно почему перед "Info" палочка, после "Sellect" стрелочка, а перед "exit"- абракадабра. 
Почему пункт "exit" с маленькой, а остальные нет? 
```
### Main menu ###
 1. [Start simulation]
 2. Settings->
 3. -Info
 4. <!exit
Enter number from 1 to 4
```

## ХОРОШО

1. Развитое меню
2. Прикольная цветная ASCII графика при распечатке карты, довольно оригинально
3. Спрайты существ не хранятся в самих существах
4. Координаты существ не хранятся в самих существах(мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название класса должно быть существительным
```
class WorldMapManage

//ПРАВИЛЬНО:
class WorldMapManager
```

- Не сокращай без необходимости
```
String msg

//ПРАВИЛЬНО:
String message
```

- Не сокращай без необходимости. Желание симметрии(одинаковая длина названий), не является необходимостью
```
private int rows;
private int cols;

//ПРАВИЛЬНО:
private int rows;
private int columns;
```

- Обман ожидания. Любой подумает, что `map` здесь это экземпляр Map а не WorldMap
```
map = new WorldHashMap(config.getWidth(), config.getHeight());

//ПРАВИЛЬНО:
worldMap = new WorldHashMap(config.getWidth(), config.getHeight());
```

- Применяй стандартные названия. Для синглтона это будет так
```
public static Config getConfig() {
  if (config == null) {
    config = new Config();
  }
  return config;
}

//ПРАВИЛЬНО:
public static Config getInstance() {...}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  

**2. Лишние скобочки**
```
if ((msg != null) && (!msg.isBlank())) {...}

//ПРАВИЛЬНО:
if (msg != null && !msg.isBlank()) {...}
```

**3. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if (!path.isEmpty() && path.size() != reachableRadius) {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(/*args or empty*/)) {...}

private boolean isНазваниеКотороеВсеОбъясняет(/*args or empty*/) {
  return if (!path.isEmpty() && path.size() != reachableRadius) ;
}
```

**4. Сразу кастуй**
```
if (food instanceof Herbivore) {
  Herbivore herb = (Herbivore) food;
  attackHerbivore(herb);
}

//ПРАВИЛЬНО:
if (food instanceof Herbivore herbivore) {
  attackHerbivore(herbivore);
}
```

**5. Свои пользовательские исключения делай непроверяемыми(unchecked)**
```
public class EntityNotFoundException extends Exception 

//ПРАВИЛЬНО:
public class EntityNotFoundException extends RuntimeException 
```

Тогда среди прочего не придется усложнять сигнатуры методов
```
private void eat(Coordinates foodCoordinates) throws InvalidParametersException {...}

//ПРАВИЛЬНО:
private void eat(Coordinates foodCoordinates) {...}
```
*"ЧК", гл.7, п."...проверяемые исключения"*  


**6. class Coordinates**

+ (+)Нет ничего лишнего, это хорошо.

- (±)Класс может быть преобразован в рекорд без потери функционала.

**7. Пакет worldmap**

+ (+)Сделал разные реализации карты, потренировался с наследованием и полиморфизмом- это хорошо.

**8. interface WorldMap<C, V>**, интерфейс универсальной игровой карты

Более радикальная реализация моей идеи универсальной доски: https://t.me/zhukovsd_it_chat/53243/132434 п."Архитектура".  
Но здесь интерфейс принимает два дженерика, где второй дженерик это класс хранимого в доске объекта(шахматная фигура, ентити симуляции, клетка морского боя etc).
А первый дженерик это класс координаты.

Вроде бы два джененрика должны дать еще бОльшую универсальность доске. Например, координаты могут описывать точку в одномерном(веревка), двумерном и трехмерном пространстве. 
Но это не работает. По крайней мере не так, как задумывалось.

- В качестве ключа теперь можно задавать не только специальный класс `Coordinates`, но и вообще все что угодно. Например, класс Утюг.
Нужно ввести маркерный интерфейс для всех потенциальных координатообразных координат, например так
```
public interface Coordinates {
}

public class Coordinates2d implements Coordinates {
  private final int x;
  private final int y;
  //...
}  

public class Coordinates3d implements Coordinates {
  private final int x;
  private final int y;
  private final int z;
  //...
}  

public interface WorldMap<V> {
    Optional<V> getValue(Coordinates coordinates);
    //...
}    
```
Это конечно тоже не помешает использовать Утюг в качестве ключа, если очень захочется. Но тогда Утюг придется сознательно промаркировать координатой. 

- Методы интерфейса говорят о том, что в любом случае Карта будет именно двумерной. Так что и Координата для нее по-любому тоже должна быть двумерной и причины для дженерика `C` становятся все менее очевидными
```
public interface WorldMap<C, V> {
  int getWidth();
  int getHeight();
  //...
}  
```

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними- вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: подсчитать площадь, выдать случайные и "taken" координаты.

Методы чужих ответственностей должны находиться в тех классах, в интересх которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

**9. abstract class BaseWorldMap<C, V> implements WorldMap<C, V>**

- Карта не должна хранить пустые координаты
```
private final Set<C> freeCoordinates;
private final Set<C> takenCoordinates;

public BaseWorldMap(int width, int height) {
  this.freeCoordinates = generateMapCoordinates();
  this.takenCoordinates = new HashSet<>();
  //...
}

protected Set<C> generateMapCoordinates() {
  Set<C> result = new HashSet<>();
  for (int y = 0; y < height; y++) {
    for (int x = 0; x < width; x++) {
      result.add(createCoordinate(x, y));
    }
  }
  return result;
}
```

Например, есть игровая доска 100*100. Тогда даже без учета хранения существ в виде массива или хешмапы, в самих сетах будет хранится 10.000 записей-координат, без особой пользы забивая память.

- Здесь также юзается какая-то сложная логика "перебрасывания" координат из двух ящиков
```
protected void releaseCoordinates(C coordinates) {
  takenCoordinates.remove(coordinates);
  freeCoordinates.add(coordinates);
}
```
Эти сеты нужно убрать.

- Нарушение SRP, создавать координаты это точно не ответственность Карты
```
protected abstract C createCoordinate(int x, int y);
```

- Базовая карта хранит объекты подстановочного типа. Она на этом уровне не должна знать, даже на уровне нейминга, что это будет- энтити симуляции, шахматные фигуры или что-либо иное
```
protected abstract void clearEntities();

//ПРАВИЛЬНО:
protected abstract void clearValues();
```

**10. class WorldArrayMap extends BaseWorldMap<Coordinates, Entity>**, реализация карты на массиве

- Для невозврата null нужно либо кидать исключение при попытке прочитать отсутствующее существо.  
Либо возвращать Optional, как обертку над значением или null.  
Но не то и другое вместе
```
@Override
public Optional<Entity> getValue(Coordinates coordinates) throws InvalidCoordinatesException {
  validateCoordinates(coordinates);
  return Optional.ofNullable(entities[coordinates.getY()][coordinates.getX()]);
}

@Override
public void validateCoordinates(C coordinates) throws InvalidCoordinatesException {
  if (!freeCoordinates.contains(coordinates) && !takenCoordinates.contains(coordinates)) {
    throw new InvalidCoordinatesException(ErrorMessages.INCORRECT_COORDINATES_ARE_SPECIFIED);
  }
}
```

**11. class WorldHashMap extends BaseWorldMap<Coordinates, Entity>**, реализация карты на хешмапе

- Нарушение DRY. Такая же реализация метода есть в `WorldArrayMap`
```
@Override
protected Coordinates createCoordinate(int x, int y) {
  return new Coordinates(x, y);
}
```

**12. class WorldMapManage**, утилитный(местами) класс для Карты

- Трудно понять, что это такое.  
Название класса и некоторые находящиеся в нем методы типа `public <T> List<T> getEntitiesByType(Class<T> type)` говорят, что это утилитный класс. 
Но при этом класс хранит какие-то поля и методы работы с ними
```
private final Config config;
private WorldMap<Coordinates, Entity> map;
```
Очевидно, в этом классе обитают несколько сущностей. Одна сущность это утилитный класс и содержит полезные методы для разных применений карты, типа вернуть случайную пустую координату.
 а другая сущность/сущности это что-то другое. Нужно разделить этот класс на несколько.

- Утилитный класс должен быть final, иметь закрытый конструктор, все его методы должны быть static, не должен хранить поля.

**13. class Config**, класс настроек

Про константные и конфигурационные классы я подробно писал тут: https://t.me/zhukovsd_it_chat/53243/176984

- Еще один франкенштейн-нарушитель SRP. Кроме хранения и выдачи настроек содержит сеттеры, валидатор и каку-то сложную логику в `int getEntityMapFillingPercentage(EntityType entityType)`

**14. class ConfigValidator**

- Снова не пойму что это такое и зачем. Сложная мне недоступная логика
```
protected static void validateMapFillingPercentages(Config config) throws InvalidFillingPercentageException {
  int[] inanimateValues = {
    config.getRocksMapFillingPercentage(),
    config.getTreesMapFillingPercentage(),
    config.getGrassMapFillingPercentage()
  };
  // и т.д.
}
```

- Циклическая зависимость на class Config: Config использует ConfigValidator, а ConfigValidator использует Config
```
(config.getHeight() < Config.MIN_HEIGHT || config.getHeight() > Config.MAX_HEIGHT)
``` 

**15. abstract class Entity**

- Зачем нужен id, совершенно непонятно
```
private final int id;
```

**16. class Tree extends Entity**

- По сумме применений синглтонов в проекте констатирую синглетонизм
```
public static Tree getInstance() {
  if (instance == null) {
    instance = new Tree();
  }
  return instance;
}
```
Не увлекайся синглтонами, в данном случае для внедрения синглтона нет оснований. 
Если хочется сэкономить на хранении объектов в карте и реализация Карты в данном проекте позволяет использовать объекты-константы, пусть объекты-константы хранит в себе тот, кто их помещает в Карту
```
public class EntitiesDeployment бла-бла {
  //...
  private void deployEntity(...) {
    //...  
    mapManager.setEntity(coordinate, getTree()); // создание Дерева через синглтон (прим. ред.)
    //...
  }
}

//ПРАВИЛЬНО:
public class EntitiesDeployment бла-бла {
  private final static Tree TREE_INSTANCE = new Tree();

  //...
  private void deployEntity(...) {
    //...  
    mapManager.setEntity(coordinate, TREE_INSTANCE);
    //...
  }
}
```

**17. class Corpse extends Entity**, труп

+ (+)Труп это прикольно.

- Магическое число: 6.

**18. abstract class Creature extends Entity**

- Нарушение SRP. Креатура хранит в себе менеджер карты и конфиг, а значит знает много ненужного для себя
```
protected final WorldMapManage mapManager;
```

- Нарушение Low Coupling. Класс имеет большое количество ненужных зависимостей: WorldMapManage, а через него- Config и ConfigValidator. 
Из прикладных классов Creature должен знать только про Карту(WorldMap) потому что Creature самостоятельно передвигает себя в рамках этой Карты. 

- Магические числа: 2, 1, 1, 1.

- Большое количество кода, который связан с перемещением существа по карте. Складывается впечатление, что часть этого кода относится к ответственности поиска пути, а не передвижения существа.
Например, `findNearestFoodCoordinatesInViewRadius(...)`

**19. class Herbivore extends Creature**

- Не забивай конструктор лишними подробностями
```
public Herbivore(WorldMapManage mapManager) {
  this(
    new CharacteristicsBuilder()
      .setHealthPointsInRange(MIN_HP, MAX_HP)
      .setViewRadiusInRange(MIN_VIEW_RADIUS, MAX_VIEW_RADIUS)
      //еще миллион строк
      .build(),
    mapManager
  );
}

//ЛУЧШЕ:
private static final Characteristics CHARACTERISTICS = new CharacteristicsBuilder()
      .setHealthPointsInRange(MIN_HP, MAX_HP)
      .setViewRadiusInRange(MIN_VIEW_RADIUS, MAX_VIEW_RADIUS)
      //еще миллион строк
      .build();

public Herbivore(WorldMapManage mapManager) {
  this(CHARACTERISTICS, mapManager);
}
```

**20. interface PathFinder<C>**

- Интерфейс сделан настолько базовым и абстрактным, что по сигнатуре метода я не могу даже предположить, как им можно пользоваться
```
public interface PathFinder<C> {
  Set<C> getPath(C source, C target, Set<C> availableCoordinates);
}
```

**21. class PathFinderBFS implements PathFinder<Coordinates>**, поиск пути

- Мне непонятно, как можно корректно возвращать путь через сет координат
```
public Set<Coordinates> getPath(...)
```
Есть сомнения, что поиск работает правильно.

Путь это последовательность соседствующих друг с другом кординат, для описания пути нужно чтобы координаты находились в строго определенном последовательном порядке, а это возможно только в List'e.  
Set не гарантирует последовательный порядок размещения в нем объектов 
```
public class Main {
  public static void main(String[] args) {
    List<Character> list = new ArrayList<>();
    Set<Character> set = new HashSet<>();

    list.add('z');
    set.add('z');

    list.add('x');
    set.add('x');

    list.add('a');
    set.add('a');

    list.add('n');
    set.add('n');

    System.out.println("List: " + list);
    System.out.println("Set: " + set);
  }
}

//РЕЗУЛЬТАТ:
List: [z, x, a, n]
Set: [a, x, z, n]
```

- Нарушение SRP. Здесь оно заключается в том, что часть работы по поиску пути выполняет тот класс, который использует PathFinderBFS.
Сейчас клиент самостоятельно находит координату цели и передает ее в поиск пути, а так же подготавливает для Поиска перечень свободных координат
```
public class PathFinderBFS implements PathFinder<Coordinates> {
  @Override
  public Set<Coordinates> getPath(Coordinates source, Coordinates target, Set<Coordinates> freeCoordinates) {...}
```

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям. 
В данном случае- до точки, в которой находится существо нужного класса. В рамках этого поиска PathFinderBFS должен сам находить цель, доступные для ходьбы координаты и выполнять прочие вспомогательные действия.
Сигнатура поиска должна выглядеть примерно так
```
public List<Coordinates> getPath(Карта карта, Coordinates tart, Class<? extends Entity> target) {...}
```

Или так, если в качестве цели могут выступать несколько разных классов
```
public class Лиса {
  //...
  public bool isFood(Entity entity) {
    return entity instanceof Колобок || entity instanceof Заяц;
  }

  public void makeMove(Карта карта) {
    List<Coordinates> путь =  pathFinderBFS.getPath(карта, карта.getCoordinate(this), this::isFood);  
    //...
  }

}

public class PathFinderBFS бла-бла {
  public List<Coordinates> getPath(Карта карта, Coordinates target, Predicate<Entity> targetDetector) {...}
}
```

**22. enum EntityNotation implements EntityNotationProvider**, ужас летящий на крыльях ночи, яркий пример того, как не надо делать енамы

- Енамы нужно использовать как енамы, то есть в качестве продвинутых констант. Не нужно с помощью хитрых фокусов делать из енама недоклассы типа такого
```
public enum EntityNotation implements EntityNotationProvider {

  EMOJI {
    @Override
    public String getNotation(Entity entity) {
      if (entity == null) {
        return EmojiNotation.EMPTY;
      }
      //еще миллион строк
      throw new IllegalArgumentException(ErrorMessages.NO_NOTATION_FOUND_FOR_ENTITY
          + entity.getClass().getSimpleName());
    }
  },
  SYMBOL {
    @Override
    public String getNotation(Entity entity) {
      if (entity == null) {
        return AnsiColor.colorizeNotation(AnsiColor.WHITE, SymbolNotation.EMPTY);
      }
      //еще миллион строк
      throw new IllegalArgumentException(ErrorMessages.NO_NOTATION_FOUND_FOR_ENTITY
          + entity.getClass().getSimpleName());
    }
  };

  public static class SymbolNotation {
    //миллион строк
  }

  private static class EmojiNotation {
    //миллион строк
  }

  private static class AnsiColor {
    //миллион строк

    private static String colorizeNotation(int mainColor, String notation) {
      return String.format(TEMPLATE, mainColor, notation);
    }
  }
}
```

Как сделать правильно, рассмотрим в конце.

**23. class Render**, распечатывает разную информацию

- Методы, которые распечатывают данные из объектов, должны принимать в себя сами объекты, а не их кишки
```
public void renderMenu(String title, List<String> items, String prompt) {...}
public void renderTurn(Map<Coordinates, Entity> map) {...}

//ПРАВИЛЬНО:
public void renderMenu(Menu menu) {
  System.out.println(menu.getTitle());
  //...
}

public void renderWorldMap(WorldMap worldMap) {...}
```

**24. class Simulation**

+ (+)Принимает в конструктор достаточное количество зависимостей, это хорошо.

## АРХИТЕКТУРА

- Большое количество классов, во многих из которых трудно разобраться. 
- Неочевидные зависимости между классами, приводящие к спагетти-коду.

- Синглетонизм, от которого нужно избавляться.  
Бывает очень мало ситуаций в программировании, когда применение синглтонов в пользовательских классах бывает оправдано. 
Сейчас консолидированное мнение сообщества в основном считает синглтон антипаттерном. 
Воздержись от применения синглтонов по крайней мере до того, как улучшишь свое понимание ООП. 
Ничего страшного, если вообще не будешь применять синглтоны.

- Неправильное использование енамов, не перегружай их сложной логикой.

Enum это всего-лишь продвинутые константы, используй их как простые перечисления
```
public enum Currency {
  USD, EURO
}
```

Или как более сложные перечисления
```
public enum Currency {
  USD("USD", '$'),
  EURO("EUR", '€');

  private final String code;
  private final char symbol;

  Currency(String code, char symbol) {
    this.code = code;
    this.symbol = symbol;
  }

  public String getCode() {
    return code;
  }

  public char getSymbol() {
    return symbol;
  }
}
```

Но не как псевдоклассы
```
public enum Currency {
  USD("USD", '$'),
  EURO("EUR", '€');

  private final String code;
  private final char symbol;

  Currency(String code, char symbol) {
    this.code = code;
    this.symbol = symbol;
  }

  public String getCode() {
    return code;
  }

  public char getSymbol() {
    return symbol;
  }

  public BigDecimal convertEuroToUsd(BigDecimal value) {
    BigDecimal coefficient = new BigDecimal("1.04");
    return value.multiply(coefficient);
  }
}
```

- Используй ООП подходы для реализации типичных задач.

**Типичная задача: разное представление объекта**

Есть игровая карта, которая хранит Entity. Нужно распечатать карту, но предусмотреть возможность показа Entity в виде разных изображений: буквой или emoji.  

Задачу можно решить по-разному, например, вот так

1. Чтобы распечатать существо, нужно знать, как оно выглядит.  
Значит нужен класс, в который мы кинем существо, а обратно получим его изображение(спрайт). Нам нужно два класса: один будет возвращать буквы, другой- emoji.
Классы должны быть родственны и взамозаменяемы, а значит должны работать через полиморфизм и реализовывать общий интерфейс
```
public interface SpritePool {
  String get(Entity entity);
}
```

2. Реализации интерфейса для букв и эмоджи будут иметь общий код, поэтому делаем общий для них абстрактный класс
```
public abstract class AbstractSpritePool implements SpritePool {
  @Override
  public String get(Entity entity) {
    return switch (entity.getClass().getSimpleName()) {
      case "Fox" -> fox();
      case "Rabbit" -> rabbit();
      default -> throw new IllegalStateException("illegal entity: " + entity);
    };
  }

  protected abstract String rabbit();
  protected abstract String fox();
}
```

3. И собственно делаем две реализации
```
public class EmojiSpritePool extends AbstractSpritePool {

  private final static String RABBIT = "🐇";
  private final static String FOX = "🦊";

  @Override
  protected String rabbit() {
    return RABBIT;
  }

  @Override
  protected String fox() {
    return FOX;
  }
}

public class TextSpritePool extends AbstractSpritePool {

  private final static String RABBIT = "R";
  private final static String FOX = "F";

  @Override
  protected String rabbit() {
    return RABBIT;
  }

  @Override
  protected String fox() {
    return FOX;
  }
}
```

4. Теперь создаем рендерер для карты
```
public class BoardRenderer {

  private static final String GROUND_SPRITE = " ";
  private final SpritePool spritePool;

  public BoardRenderer(SpritePool spritePool) {
    this.spritePool = spritePool;
  }

  public void render(Board board) {
    for (int row = 0; row < board.getRows(); row++) {
      for (int column = 0; column < board.getColumns(); column++) {
        Coordinates coordinates = new Coordinates(row, column);
        if(board.isEmpty(coordinates)) {
          System.out.print(GROUND_SPRITE);
        } else {
          Entity entity = board.get(coordinates);
          String sprite = spritePool.get(entity);
          System.out.print(sprite);
        }
      }
      System.out.println();
    }
  }
}
```

5. В майне определяемся, в каком виде хотим видеть карту, создаем зависимости, инжектим и запускаем приложение
```
public class Main {
  public static void main(String[] args) {
    Board board = new Board(10, 10);
    //заселить карту зайцами и лисами

    SpritePool spritePool = new EmojiSpritePool();
    BoardRenderer boardRenderer = new BoardRenderer(spritePool);

    Game game = new Game(board, boardRenderer);
    game.start();
  }
}
```

## ВЫВОД

Тренироваться. Для лучшего понимания декомпозиции посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID- по одному ролику на каждую букву.

n.56(127)  
#ревью #симуляция #енам #enum #рендерер #renderer