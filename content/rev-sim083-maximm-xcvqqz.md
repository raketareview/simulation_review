https://github.com/xcvqqz/TheRealSimulation  
[Maxim M]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Во время работы симуляции примерно абсолютно невозможно ввести команды, например "stop". 
Потому что пока будешь набирать, в консоль выведет очередную распечатку карты и ввод собьется
```
Commands: 
[start] - begin simulation 
[pause] - pause simulation 
[resume] - resume paused simuation 
[stop] - stop simulation completely  
[exit] - quit program
```
Вот здесь я успел ввести только первые две буквы("st") команды
```
st-------- Turn: 5 ---------
⛰️🟫🌿🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🐺🟫🟫🟫🟫🟫⛰️🟫🟫
🟫🟫🦌🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🌿🟫🦌🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🦌🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🌳🟫🟫🟫🟫🟫🟫
🟫🟫🟫🌳🟫🌳🟫🟫⛰️🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫⛰️🟫🟫🟫🟫🟫🟫🟫🟫🟫
🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫🟫⛰️
🟫🌿🟫🟫🟫🟫🟫🟫🟫🌳⛰️🟫
```
Просто сделай команды длиной в 1 символ.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов пишутся стилем lower_snake
```
gameUtils

//ПРАВИЛЬНО:
game_utils
```

- Раз есть "ширина", то должна быть и "высота"
```
public class GameBoard {
  private final int length;
  private final int width;
  //...
}

//ПРАВИЛЬНО:
public class GameBoard {
  private final int height();
  private final int width;
  //...
}
```

- Излишний контекст. Из сигнатуры и так понятно, что ентити возвращается по координате
```
public Entity getEntityAt(Coordinates coordinates)

//ПРАВИЛЬНО:
public Entity getEntity(Coordinates coordinates)
```

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```
private ArrayList<Entity> entityList;

//ПРАВИЛЬНО:
private ArrayList<Entity> entities;
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Entity> entities = new HashMap<>();
HashMap<Coordinates, Entity> getEntities()
ArrayList<Coordinates> visitedPath;

//ПРАВИЛЬНО:
Map<Coordinates, Entity> entities = new HashMap<>();
Map<Coordinates, Entity> getEntities()
List<Coordinates> visitedPath;
```

**3. Слишком длинные конструкции**, которые трудно понять.

Приоритет- простота и понятность кода, а не экономия строчек. Вводи поясняющие переменные
```
List<Coordinates> pathToFood = new PathFinder(gameBoard).searchFood(gameBoard.getCoordinates(this), Grass.class);

//ПРАВИЛЬНО:
PathFinder pathFinder = new PathFinder(gameBoard);
Coordinates start = gameBoard.getCoordinates(this);
List<Coordinates> pathToFood = pathFinder.searchFood(start, Grass.class);
```
*Фаулер, "Рефакторинг", гл.6, "Введение поясняющей переменной".*

**4. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (herbivore == null) return;

//ПРАВИЛЬНО:
if (herbivore == null) {
  return;
}
```

**5. Нарушение DRY**, магические буквы, числа, слова. Вводи константы. А если они уже есть- пользуйся
```
public static final String START = "start";
public static final String PAUSE = "pause";
public static final String RESUME = "resume";

public static final String START_MENU_INFO_MESSAGE = 
    "\nCommands: \n[start] - begin simulation" +
    " \n[pause] - pause simulation" +
    " \n[resume] - resume paused simuation" + ...

//ПРАВИЛЬНО:
public static final String START_MENU_INFO_MESSAGE = 
    "\nCommands: \n[%s] - begin simulation".formatted(START) +
    " \n[%s] - pause simulation".formatted(PAUSE) +
    " \n[%s] - resume paused simuation".formatted(RESUME) + ...
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой" *  
*refactoring.guru "Замена магического числа символьной константой"*  

**6. Приоритет- читаемость кода**, а не экономия строчек
```
System.out.println(STOPPED_INFO_MESSAGE + "\n" + RESTART_INFO_MESSAGE);

//ПРАВИЛЬНО:
System.out.println(STOPPED_INFO_MESSAGE);
System.out.println(RESTART_INFO_MESSAGE);
```

**7. Пакет gameUtils**

В пакете нет ни одной утилиты. Гугли, что в яве понимается под утилитным классом.

**8. record Coordinates(int column, int row)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**9. class Constants**

- Константный класс здесь *иногда* используется неправильно- другие классы напрямую лезут в константы вместо того, чтобы получать необходимые этим классам данные в свой конструктор.
Это делает классы неуниверсальными и зависимыми от класса констант.  

Условный пример
```
public final class Settings {
  //приватный конструктор
  public static final int HOUSE_ROOMS = 5;
}

/ПЛОХО:
public static void main(String[] args) {
  House house = new House();
  //oth code
}

public class House {
  private final int rooms;

  public House() {
    this.rooms = Settings.HOUSE_ROOMS;
  }

  public int getRooms() {
    return rooms;
  }
}

//ХОРОШО:
public static void main(String[] args) {
  House house = new House(Settings.HOUSE_ROOMS);
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
Про классы констант, конфигурации и их использование я писал [тут](https://t.me/zhukovsd_it_chat/53243/176984)

- Используй многострочные строки
```
public static final String START_MENU_INFO_MESSAGE = "\nCommands: \n[start] - begin simulation" +
    " \n[pause] - pause simulation" + ...

//ПРАВИЛЬНО:
public static final String START_MENU_INFO_MESSAGE = """
      Commands: \n[start] - begin simulation
      [pause] - pause simulation
      ...
    """;
```

**10. class GameBoard**, карта

- Константный класс здесь используется неправильно, ширину и высоту `GameBoard` должен получать в конструктор
```
public GameBoard() {
  this.length = Constants.GAMEBOARD_LENGTH;
  this.width = Constants.GAMEBOARD_WIDTH;
}

//ПРАВИЛЬНО:
public GameBoard(int length, int width) {
  this.length = length;
  this.width = width;
}
```

- Никогда не возвращай null
```
private final HashMap<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntityAt(Coordinates coordinates) {
  return getEntities().get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public Entity getEntityAt(Coordinates coordinates) {
  return getEntities().get(coordinates);
}
```

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: выдать случайную пустую координату.

Наверное, для проекта в целом полезно иметь метод, который выдает случайную пустую координату. 
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно генерировать случайную координату.  

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
private final HashMap<Coordinates, Entity> entities = new HashMap<>();

public HashMap<Coordinates, Entity> getEntities() {
  return entities;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта(100, 100);
//заселить карту существами
карта.getEntities().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

- Нарушение SRP, отсутствует метод удаления существа из карты.

Очевидно, что для работы программы, в карту нужно добавлять существ и удалять из нее. 
Здесь же метод добавления в карте есть, а метода удаления в карте нет нет. 
Следовательно, удаление делает кто-то другой(Predator и проч.). 

Но делает это напрямую, получая из карты ее внутренности через `getEntities()`. 
Таким образом, эти классы забирают часть ответственности карты себе.

Методы вставки и удаления существа должны находиться в классе карты.

**11. class PathFinder**

- Дополнительное параметризирование тут не дает никаких выгод, но делает сигнатуру более сложной и непонятной
```
public <T extends Entity> List<Coordinates> searchFood(Coordinates start, Class<T> typeOfFood)

//ЛУЧШЕ:
public List<Coordinates> searchFood(Coordinates start, Class<T extends Entity> typeOfFood)
```

- Избыточно. Не нужно каждый раз пересоздавать этот массив, а он пересоздается каждый раз при вызове метода. Сделай его константой
```
private Coordinates[] getShiftDirections() {
  return new Coordinates[]{
      new Coordinates(-1, 0),
      new Coordinates(1, 0),
      new Coordinates(0, -1),
      new Coordinates(0, 1)
  };
}

//ПРАВИЛЬНО:
private static final Coordinates[] SHIFT_COORDINATES = { new Coordinates(-1, 0), ...};
```

**12. abstract class Entity** и его "статичные" наследники

👍 Норм.

**13. abstract class Creature extends Entity**

Из этих трех методов только один должен быть публичным- `makeMove(...)`
```
public abstract void makeMove(GameBoard gameBoard);
public abstract void attackFood(GameBoard gameBoard, Coordinates coordinates);
public abstract void moveCreature(GameBoard gameBoard, Coordinates from, Coordinates to);
```
Остальные методы являются подробностями внутренней механики перемещения существ и работают в интересах `makeMove(...)`. 
А потому должны быть *в данном случае* protected.

**14. class Herbivore/Predator extends Creature**

- Нарушение DRY. Абсолютно одинаковая реализация `moveCreature(...)`. 
Просто вынеси этот общий код в предка.

- Нарушение DRY. Метод `makeMove(...)` тоже без труда можно сделать общим и вынести в предка
```
public class Predator extends Creature {
  //...  
  @Override
  public void makeMove(GameBoard gameBoard) {
    List<Coordinates> pathToFood = new PathFinder(gameBoard).searchFood(gameBoard.getCoordinates(this), Herbivore.class);
    if (!pathToFood.isEmpty()) {
      moveAlongPath(gameBoard, pathToFood);
    }
  }
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //...
  public void makeMove(GameBoard gameBoard) {
    PathFinder pathFinder = new PathFinder(gameBoard);
    Coordinates start = gameBoard.getCoordinates(this);
    List<Coordinates> pathToFood = pathFinder.searchFood(start, getFood());

    if (!pathToFood.isEmpty()) {
      moveAlongPath(gameBoard, pathToFood);
    }
  }
  protected Class<? extends Entity> getFood();
}    
```

**15. class SpawnAction extends Action**

Снова перемудрил с сигнатурой
```
private <T extends Entity> Entity createEntity(Class<T> typeOfEntity)

//ПРАВИЛЬНО:
private Entity createEntity(Class<<T extends Entity>> typeOfEntity)
```

- Нарушение DRY
```
entityList.add(createEntity(Grass.class));
entityList.add(createEntity(Grass.class));
//еще миллион таких же строк

entityList.add(createEntity(Tree.class));
entityList.add(createEntity(Tree.class));
//еще миллион таких же строк

//ПРАВИЛЬНО:
spawn(Grass.class, GRASS_AMOUNT);
spawn(Tree.class, TREE_AMOUNT);

private void spawn(Class<T extends Entity> clazz, int amount) {
  for(int i = 0; i < amount, i++) {
    Entity entity = createEntity(clazz);
    entityList.add(entity);
  }
}
```

**16. class MapConsoleRenderer**

В конце цепочки if'ов' нужно кидать исключение- значит в карте есть существо, спрайт которого мы не знаем. 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
private String colorizeAndGetSprite(Coordinates coordinates) {
  Entity entity = gameBoard.getEntities().get(coordinates);
  if (entity instanceof Herbivore) {
    return HERBIVORE;
  }
  if (entity instanceof Grass) {
    return GRASS;
  }
  //...
  return GROUND;
}

//ПРАВИЛЬНО:
private String getSprite(Entity entity) {
  if (entity instanceof Herbivore) {
    return HERBIVORE;
  }
  if (entity instanceof Grass) {
    return GRASS;
  }
  //...
  throw new IllegalArgumentException("illegal entity: " + entity);
}
```

**17. class Simulation**

👍 Принимает достаточное количество зависимостей в конструктор. В данном случае- карту
```
public Simulation(GameBoard gameBoard)
```

**18. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.83(187)  
#ревью #симуляция 