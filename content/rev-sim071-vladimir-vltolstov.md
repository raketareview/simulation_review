https://github.com/vltolstov/Simulation  
[Владимир]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Не видно границ карты  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim071/img0.png)

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Существа совершают случайные ходы, если не видят еды

## ЗАМЕЧАНИЯ

**1. Нейминг**

- UPPER_SNAKE только для констант, а это не константа
```
private static ArrayList<Action> INIT_ACTIONS = new ArrayList<Action>();
private static ArrayList<Action> TURN_ACTIONS = new ArrayList<Action>();

//ПРАВИЛЬНО ТАК:
private static final ArrayList<Action> INIT_ACTIONS = new ArrayList<Action>();
private static final ArrayList<Action> TURN_ACTIONS = new ArrayList<Action>();

//ИЛИ ТАК:
private static ArrayList<Action> initActions = new ArrayList<Action>();
private static ArrayList<Action> turnActions = new ArrayList<Action>();
```

- Если в проекте есть класс World то все переменные с именем, включающим это название, должны быть экземплярами этого класса
```
public class World {
  private Map<Coordinates, Entity> world = new HashMap<Coordinates, Entity>();
  //...
}

//ПРАВИЛЬНО:
private Map<Coordinates, Entity> entities = new HashMap<Coordinates, Entity>();
```

- В яве названия интерфейсов не пишут с префиксом "I", так принято делать в C#
```
interface ICreature
```
В яве названия интерфейсов пишут либо с постфиксом "able", например Runnable. Либо просто существительное, например List.

- Несоответствие названия тому, что делает метод. Обещает сделать шаг к цели, а на самом деле возвращает координату.  
И это не рассматривая общую бессмысленность метода, который принимает координату и тут же ее возвращет взад😁
```
private Coordinates makeMoveToTarget(Coordinates targetCoordinates) {
  return targetCoordinates;
}
```

- Избыточный контекст, тяжело читается. Чем меньше область видимости переменной, тем короче можно делать ее название
```
int randomEntityXCoordinate = random.nextInt(world.getWidth());
int randomEntityYCoordinate = random.nextInt(world.getHeight());
Coordinates coordinates = new Coordinates(randomEntityXCoordinate, randomEntityYCoordinate);

//ПРАВИЛЬНО:
int x = random.nextInt(world.getWidth());
int y = random.nextInt(world.getHeight());
Coordinates coordinates = new Coordinates(x, y);
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Используй классы через их интерфейсы**
```
ArrayList<Action> INIT_ACTIONS = new ArrayList<Action>();
ArrayList<Action> TURN_ACTIONS = new ArrayList<Action>();
ArrayList<Entity> getEntities()

//ПРАВИЛЬНО:
List<Action> INIT_ACTIONS = new ArrayList<Action>();
List<Action> TURN_ACTIONS = new ArrayList<Action>();
List<Entity> getEntities()
```

**3. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if (speed > 1 && isAvailableCoordinateForAction(path.get(speed - 1), world))  {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(...)) {...}

private boolean isНазваниеКотороеВсеОбъясняет(...) {
  return speed > 1 && isAvailableCoordinateForAction(path.get(speed - 1), world);
}
```

**4. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется - в этом случае неважно, будет else или нет, так как программа будет работать одинаково и код выглядит читабельней
```
if (speed > 1 && isAvailableCoordinateForAction(path.get(speed - 1), world)) {
  return makeMoveToTarget(path.get(speed - 1));
} else {
  return makeMoveToTarget(path.get(0));
}

//ПРАВИЛЬНО:
if (speed > 1 && isAvailableCoordinateForAction(path.get(speed - 1), world)) {
  return makeMoveToTarget(path.get(speed - 1));
} 
return makeMoveToTarget(path.get(0));
```

**5. class Coordinates**

- Сдвиг(shift) или складывание(add) координаты должно происходить с экземпляром своего класса. Заводить для этого специальный класс сдвиговой координаты не надо
```
public Coordinates shift(CoordinatesShift shift) 

//ПРАВИЛЬНО:
public Coordinates shift(Coordinates shift) 
```
Обычную координату можно использовать как координату сдвига, например так
```
Coordinates coordinates = new Coordinates(10, 10);
Coordinates shiftCoordinates = new Coordinates(-1, -1);  //сдвиг влево-вниз
coordinates = coordinates.shift(shiftCoordinates);
```

- Нарушение SRP, Low Coupling, циклическая зависимость координата-карта. Координата не должна определять, можно ли ее сдвинуть в интересах какого-то другого класса или нельзя.
Потому что это не касается единой ответственности координаты- хранение информации для идентификации точки в пространстве.  
Сейчас координата вынуждена знать про совершенно посторонние для себя вещи- про существование класса Карта, и правила, по каким можно сдвигать координату, чтобы это считалось приемлемым для карты 
```
public boolean canShift(CoordinatesShift shift, World world)
```
Метод отсюда убрать. 

**6. class CoordinateShift**, координата для сдвига

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates. Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```
Coordinates shifDownRightCoordinates = new Coordinates(-1, 1);
```

**7. class World**

- Нарушение конвенции кода, константы должны стоять вверху.

- Изменять размеры карты при работе программы не предполагается, поэтому поля с размерами должны быть final: `private int width;` -> `private final int width;`.

- Никогда не возвращай null
```
private Map<Coordinates, Entity> world = new HashMap<Coordinates, Entity>();

public Entity getEntity(Coordinates coordinates) {
  return world.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- Нарушение SRP. Класс не должен сам себя конфигурировать. В данном случае он самоконфигурируется путем ведения диалога с пользователем.  
Кроме всего прочего, это не позволит сделать карту с заранее заданными размерами, потому что диалог с юзером вызывается прямо в конструкторе.  
Класс должен получать свои размеры в конструктор. Если для этого нужно опросить юзера, то пусть это делает кто-то другой, но не сам класс карты  
```
public World() {
  ConsoleRenderer.renderMessage(SETTINGS_WORLD_WIDTH_MESSAGE);
  width = InputReader.getUserCommands();

  ConsoleRenderer.renderMessage(SETTINGS_WORLD_HEIGHT_MESSAGE);
  height = InputReader.getUserCommands();
}

//ПРАВИЛЬНО:
public World(int width, int height) {
  this.width = width;
  this.height = height;
}
```
*Мартин "ЧК", гл.11, "Отделение конструирования системы..."*

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: `getRandomCellCoordinates()`, `getCreatures()`.

Методы чужих ответственностей должны находиться в тех классах, в интересх которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Нарушение SRP. 
Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретным классом представителей и не должна даже знать по именам наследников Entity
```
ArrayList<Creature> getCreatures() 

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент отбирает отсюда креатур

//ИЛИ ТАК:
public List<? extends Entity> getAllBy(Class<? extends Entity> clazz)  //вернуть существ определенного класса 
```

- Нарушение SRP, зависимость модели от представления
```
private final static String SETTINGS_WORLD_WIDTH_MESSAGE = "Введите ширину мира:";
private final static String SETTINGS_WORLD_HEIGHT_MESSAGE = "Введите высоту мира:";
```
SRP звучит так: в классе должна быть одна и только одна причина для изменения. Благодаря хранению мессаджей, появляется лишняя причина для изменения: 
другой язык интерфейса или изменение стилистики надписей("Введите ширину" -> "ВВЕДИТЕ ШИРИНУ").  
Безусловно, в каком-то классе должны находиться эти месседжи. Но явно не в том классе, который занимается вопросами хранения существ.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность
```
public void removeEntity(Coordinates coordinates) {
  world.remove(coordinates);
}
```
Если координата некорректна(находится вне пределов карты), нужно бросать исключение.

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
карта.putEntity(new Coordinates(0, 0), new Заяц());
карта.moveEntity(new Coordinates(0, 0), new Coordinates(99, 99));

/* class World */
public void moveEntity(Coordinates from, Coordinates to) {
  Entity entity = this.world.get(from);
  removeEntity(from);
  setEntity(to, entity);
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.  
Если вдруг игровая логика более высокого порядка подразумевает телепортацию (а-ля игра "HMM-3" и присутствующий там двусторонний монолит), а она имеет на это право, то логика более высокого порядка пусть и реализует телепорт.

**8. class WorldFactory**

👍 Фабрика карты это хорошо.

**9. class PathFinder**

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```
public static List<Coordinates> getPath(Coordinates coordinates, Creature creature, World world) {
  //...
  if ((creature instanceof Predator) && world.getEntity(item) instanceof Herbivore) {...}
  if ((creature instanceof Herbivore) && world.getEntity(item) instanceof Grass) {...}
  //...
}
```

Поиск вообще не должен быть в курсе взаимоотношений разных видов креатур между собой. Поиск должен просто искать путь до цели. Параметры цели Поиск должен принимать извне, например, так
```
public List<Coordinates> getPatch(World world, Coordinates start, Class<? extends Entity> target) {...}
```

- Раз этот класс содержит только статические методы, то он должен быть final и иметь приватный конструктор- чтобы нельзя было от него унаследоваться или сделать экземпляр класса.

**10. interface ICreature**

Несоответствие названия интерфейса хранимому методу: интерфейс называется "Креатура", а содержит метод, который из сета координат выбирает случайную координату  
```
public interface ICreature {
  default Coordinates selectMoveCoordinate(Set<Coordinates> availableCoordinates) {

    Random rand = new Random();
    int randomCoordinatesIndex = rand.nextInt(availableCoordinates.size());

    List<Coordinates> availableCoordinatesList = new ArrayList<>(availableCoordinates);
    Coordinates newRandomCoordinates = availableCoordinatesList.get(randomCoordinatesIndex);

    return newRandomCoordinates;
  }
}
```
Скорее, это должно быть интерфейсом "Выбиратель случайной координаты".  
Но лучше просто перенеси этот метод в Креатуру или переделай этот интерфейс на утилитный класс.

**11. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня Creature.

- Нарушение инкапсуляции. Доступ к полю в классе должен быть только через сеттеры и геттеры
```
public Coordinates coordinates;
```

**12. abstract class Creature extends Entity implements ICreature**

- Придерживайся единой терминалогии в проекте. Если в проекте есть класс `Action`, то в рамках проекта метод `makeAction(...)` должен делать экземпляр класса `Action`, а не Координаты
```
public Coordinates makeAction(World world)
```

- Разделяй команды и запросы. Этот метод должен или выполнить действие, то есть совершить ход. Или вернуть координату, то есть выполнить запрос. Но не то и другое вместе
```
public Coordinates makeAction(World world)
```
*Мартин, "Чистый код", гл.3, п."Разделение команд и запросов"*  

- Принимай все в конструктор
```
public abstract class Creature extends Entity implements ICreature {
  protected int speed;
  protected int health;

  public Creature(Coordinates coordinates) {
    super(coordinates);
  }
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity implements ICreature {
  private int speed;
  private int health;

  public Creature(Coordinates coordinates, int speed, int health) {
    super(coordinates);
    this.speed = speed;
    this.health = health;
  }
  //...
}
```

**13. class Herbivore/Predator extends Creature**

Не пойму мотивацию, почему часть данных инжектится в супер, а часть сетится
```
public class Herbivore extends Creature {
  public Herbivore(Coordinates coordinates) {
    super(coordinates);
    speed = 1;
    health = 100;
  }
  //...
}

//ПРАВИЛЬНО:
public class Herbivore extends Creature {
  private static final int SPEED = 1;  
  private static final int HEALTH = 100;

  public Herbivore(Coordinates coordinates) {
    super(coordinates, SPEED, HEALTH);
  }
  //...
}
```

**14. class MoveAction extends Action**

👍 Только инициирует движение в креатурах(дает им пинка, чтобы они побежали), это хорошо.

**15. class EntityFactory**

+ 👍 Фабрика норм.

- Сейчас есть более четкая форма записи свича
```
switch (entityIndex) {
  case 0:
    return new Herbivore(coordinates);
  case 1:
    return new Predator(coordinates);
  default:
    throw new RuntimeException("Unknown entity type");
}

//ЧЁТКО:
return switch (entityIndex) {
  case 0 -> new Herbivore(coordinates);
  case 1 -> new Predator(coordinates);
  default -> throw new RuntimeException("Unknown entity type");
};
```

- Делай текст в исключениях более информативным
```
throw new RuntimeException("Unknown entity type");

//ПРАВИЛЬНО:
throw new RuntimeException("Unknown entity entity index: " + entityIndex);
```

**16. class Menu extends Thread**

Меню в процедурном стиле. Для простой программы- почему бы и нет. Про меню в ООП стиле я писал тут: https://t.me/zhukovsd_it_chat/53243/114908

- Не забывай ставить аннотацию при переопределении методов
```
public void run()

//ПРАВИЛЬНО:
@Override
public void run()
```

**17. class Simulation**

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий. 
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе симуляции
```
public Simulation(ArrayList<Action> initActions, ArrayList<Action> turnActions, World world)
```

- Эти поля не должны быть static. Статические поля в ООП программе нужно применять очень осторожно. 
Потому что сама идея классов состоит в том, что у класса может быть несколько экземпляров одновременно. И при этом все они должны корректно работать.
В данном случае если в программе будет существовать несколько экземпляров Симуляции одновременно, они будут работать некорректно, потому что будут одновременно вносить изменения в одни и те же статические поля.
И тут вопрос не в том, должно ли в проекте быть два экземплера одного класса, или не должно. 
Если класс не синглтон, то по умолчанию мы признаем, что его экземпляров в проекте может быть более одного.
```
private static ArrayList<Action> INIT_ACTIONS = new ArrayList<Action>();
private static ArrayList<Action> TURN_ACTIONS = new ArrayList<Action>();
private static World world;
```

- Распечатку карты вынеси в отдельный класс `WorldRenderer`.

- Плюсование строк в цикле это расточительно- постоянно пересоздаются объекты. 
Посмотри, idea подчеркивает здесь плюсование желтым цветом, это она предлагает заменить складывание строк использованием стрингбилдера.
Поставь курсор на желтое подчеркивание, слева появится желтая лампочка, клацни на нее, выбери пункт "Convert..." и идея автоматически заменит плюсование на стрингбилдер
```
String line = "";
for (...) {
  line += Sprite.getEmptySprite();
}

//ПРАВИЛЬНО:
StringBuilder line = new StringBuilder();
for (...) {
  line.append(Sprite.getEmptySprite());
}  
``` 

**18. class Main**, содержит точку входа main

👍 Только создает и запускает симуляцию, это хорошо.

## ВЫВОД

В целом, серьезных замечаний немного.  
Часто неправильно называешь переменные стилем UPPER_SNAKE, когда они не являются константами. Константы это только `static final` и только они должны писаться через UPPER_SNAKE.  
Ролик про шахматы смотрел, ок.
Посмотреть ролики Немчинского про SOLID, обратить внимание на SRP. 

n.71(158)  
#ревью #симуляция