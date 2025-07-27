https://github.com/Fristret/LifeSimulation  
[:3]

Есть много над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Каждый раз при запуске игры нужно ввести целую гору параметров, это утомляет
```
Введите размер карты до 50: 30
Введите количество деревьев: 10
Введите корректное количество: <= 5
Введите количество деревьев: 3
<ещё миллион таких же параметров>
```
Нужно сделать возможность запуска игры со значениями по умолчанию. 

Например, для этого можно сделать три разных Main'a: игра по умолчанию, игра через ввод параметров, выбор игры через меню: по умолчанию или через ввод параметров.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Реализовано пауза/пуск во время работы
+ 👍 Алгоритм поиска AStar

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Поля и методы нужно писать с маленькой буквы- стиль `camelCase`
```
int F = G + H;

//ПРАВИЛЬНО:
int f = g + h;
```

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```
HashMap<Coordinates, Entity> map;
Coordinates filterCages(List<Cage> list)

//ПРАВИЛЬНО:
HashMap<Coordinates, Entity> entities;
Coordinates filterCages(List<Cage> cages)
```

- В константах шаблонов указывай эту информацию во избежание путаницы
```
public static final String LOG_TURN_COUNTER = "Ход номер: %d";

//ЛУЧШЕ:
public static final String LOG_TURN_COUNTER_TEMPLATE = "Ход номер: %d";
```

- Метод никуда не идет, он ищет путь
```
List<Coordinates> walk(GameMap gameMap, Creature creature)
```

- Название обманывает. Класс не создает новый мир, он заселяет существами существующий мир
```
class CreateNewWorld
```

- Избыточный контекст. Из сигнатуры мы видим, что символ берется из `Entity`
```
String getEntitySymbol(Entity entity) {...}

//ЛУЧШЕ:
String toSprite(Entity entity) {...}
```

- Придерживайся единообразия, одну и ту же кнцепцию называй одинаково. Здесь непонятно, почему "интент" трансформировался в "инфо"
```
case AttackIntent info -> //...
case DeathIntent info ->  //...

//ПРАВИЛЬНО:
case AttackIntent intent -> //...
case DeathIntent intent ->  //...
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется.  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково и код без `else` будет выглядеть читабельней
```
if (Tree.class.isAssignableFrom(clazz)) {
  return new Tree(coordinates);
} else {
  throw new IllegalArgumentException(Constants.UNKNOWN_CREATURE_CLASS + clazz.getName());
}

//ПРАВИЛЬНО:
if (Tree.class.isAssignableFrom(clazz)) {
  return new Tree(coordinates);
} 
throw new IllegalArgumentException(Constants.UNKNOWN_CREATURE_CLASS + clazz.getName());
```

**3. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Entity> map;

//ПРАВИЛЬНО:
Map<Coordinates, Entity> entities;
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д. 
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с `LinkedList` нужно работать именно как с `LinkedList`, а не с `List`. Но это уже нюансы.

**4. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (openList.isEmpty()) break;
else closedList.add(filterCages(openList));

if (creature.isAlive()) creature.makeMove();
else creature.death();

//ПРАВИЛЬНО:
if (openList.isEmpty()) {
  break;
}
closedList.add(filterCages(openList)); //else после break не пишется

if (creature.isAlive()) {
  creature.makeMove();
} else {
  creature.death();
}
```

**5. Слишком длинные конструкции, которые трудно понять** 

Не экономь строки, приоритет- понятность кода.  
Вот такое вообще не читается:
```
List<Creature> list = new ArrayList<>(Stream.concat(gameMap.getListOf(Predator.class).stream(), gameMap.getListOf(Herbivore.class).stream()).toList());
```
В этой строке 151 символ. Это абсолютно нечитаемая строка, вводи поясняющие переменные.  
*Фаулер, "Рефакторинг", гл.6, "Введение поясняющей переменной"*  

**6. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
System.out.println("Симуляция запущена! Для паузы/продолжения нажмите 'P'");
case "p" -> //...
case "r" -> //...
case "help" -> System.out.println(
  """
    p -> пауза симуляция \n
    r -> продолжение симуляции \n
  """

//ПРАВИЛЬНО: 
private final static String PAUSE = "p";
private final static String RESUME = "r";
private final static String HELP = "help";

System.out.printf("Симуляция запущена! Для паузы/продолжения нажмите '%s' \n", PAUSE);
case PAUSE -> //...
case RESUME -> //...
case HELP -> {
  System.out.println(PAUSE + " пауза симуляция");
  System.out.println(RESUME + " продолжение симуляции");
};
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**7. record Coordinates(int x, int y)**

+ 👍 Record для координаты- идеально.

- Для рекорда необязательно переопределять `toString()`, чтобы он выглядел так
```
@Override
public String toString() {
  return "Coordinates{" +
    "x=" + x +
    ", y=" + y +
  '}';
}

//РЕЗУЛЬТАТ:
Coordinates{x=1, y=2}
```
Record по умолчанию умеет делать такой же `toString()`. Например:
```
public record Coordinates(int x, int y) {
}

public class Main {
  public static void main(String[] args) {
    Coordinates coordinates = new Coordinates(1, 2);
    System.out.println(coordinates);
  }
}

//РЕЗУЛЬТАТ:
Coordinates[x=1, y=2]
```

- В java методы с названием "compare" применяются для сравнивания объектов и возврата результата сравнения в виде целого числа. Например, в интерфейсе `Comparator` есть метод `int compare(T o1, T o2)`.
Здесь же метод "сравнить" ничего не сравнивает, а создает новый объект
```
public Coordinates compare(Coordinates coordinates) {
  return new Coordinates(abs(x - coordinates.x()), abs(y - coordinates.y()));
}
```

**8. class GameMap**

- Нарушение SRP. В конструктор класс принимает экземпляр `GameParameters`, хотя в этом рекорде из 10 параметров только один параметро(`mapSize`) имеет отношение к Карте
```
public GameMap(GameParameters gameParameters) {...}

//ПРАВИЛЬНО:
public GameMap(int width, int height) {...}
```

- Можно создавать только квадратную карту- необоснованное ограничение. 

Потому что за ширину и высоту отвечает только один параметр- `int mapSize`. 
Нет никаких оснований для того, чтобы ограничивать Карту квадратной формой, нужно иметь возможность ввести ее ширину и высоту.

+ 👍 Геттеры не возвращают null, то хорошо.

- При возврате существа по координате, если координата пуста, то возвращается некое "пустое существо" с некорректными координатами
```
public class GameMap {
  //...

  public Entity getEntityAt(Coordinates coordinates) {
    return map.getOrDefault(coordinates, Constants.NOTHING);
  }
}

public class Constants {
  //...
  public static final Entity NOTHING = new Nothing(-1, -1);
}
```
Решение сомнительное. Можно, конечно, интерпритировать это как некую разновидность паттерна `NullObject`, но это не совсем он.
Подробнее `Nothing` рассмотрим ниже.

Другие альтернативы не возвращать `null`: кинуть исключение, если по указанным координтам нет существа или вернуть `Optional`.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Нарушение `Low Coupling`. 

Между классами должно быть как можно меньше необязательных зависимостей.
Здесь Карта напрямую читает данные из константного класса и поэтому зависит от него.

Вместо прямого чтения из констант, класс должен необходимые данные принимать в конструктор.  
Например, в данном случае- принимать заглушку для пустого ентити:
```
public Entity getEntityAt(Coordinates coordinates) {
  return map.getOrDefault(coordinates, Constants.NOTHING);
}

//ПРАВИЛЬНО:
public GameMap(int width, int height, Entity emptyEntity) {...}

public Entity getEntityAt(Coordinates coordinates) {
  return map.getOrDefault(coordinates, emptyEntity);
}
```
Подробнее- при рассмотрении константного класса.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public Entity getEntityAt(Coordinates coordinates) { 
  return map.getOrDefault(coordinates, Constants.NOTHING);
}
```

- Стримы для этого- избыточно
```
private final HashMap<Coordinates, Entity> map;

public List<Entity> getAllEntities() {
  return map.values().stream().toList();
}

//ПРАВИЛЬНО:
public List<Entity> getAllEntities() {
  return new ArrayList<>(map.values());
}
```

- Сложная логика, которая влечет за собой потенциальные баги при пользовании картой.

Сейчас, чтобы узнать, есть ли в карте травоядные(то есть нет ли их), нужно вызвать друг за другом два метода. Первый выставит флаг присутствия травоядного, второй- вернет эту информацию
```
private boolean noHerbivores = false; <-- ФЛАГ

public void checkHerbivores() {
  noHerbivores = getListOf(Herbivore.class).isEmpty();
}

public boolean isNoHerbivores() {
  return noHerbivores;
}
```
Проблема в том, что никто не гарантирует, что клиент будет использовать эти две операции друг за другом и при определенных условиях клиент может прочитать устаревшее значение флага.

Правильно так:  
Не использовать флаги, флаги- зло.  
Не использовать отрицательные условия(isNoHerbivores()- отрицательное условие, isHerbivores()- положительное). Отрицательные условия в методах сложнее интерпритируются головой.  
Дать ответ одной операцией, а не двумя.  
`is`- является, `has`- содержит
```
//ПАВИЛЬНО:
public boolean hasHerbivores() {
  return !getListOf(Herbivore.class).isEmpty();
}
```

- Нарушение SRP. 
Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретным классом представителей и не должна даже знать по именам наследников Entity
```
public void checkHerbivores() {
  noHerbivores = getListOf(Herbivore.class).isEmpty();
}

void moveCreature(Creature creature) {...}
```
Для выполнения Картой ее единой ответстсвенности по хранению/выдаче существ, ей не нужно знать, содержатся ли в ней травоядные или нет. Карта прекрасно обойдется без этой информации.

Нужно найти тот класс, в интересах которого используется этот метод и перенести его туда- это часть единой ответственности того класса.

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
public void moveCreature(Creature creature) {
  removeEntity(creature.getOldCoordinates(), creature);
  addEntity(creature);
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

+ 👍 Это ок 
```
public <T extends Entity> List<T> getListOf(Class<T> clazz)
```

**9. class Constants**

Константный класс здесь *местами* используется неправильно- другие классы напрямую лезут в константы вместо того, чтобы получать необходимые этим классам данные в свой конструктор.

Мы это видели на примере `GameMap`.
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
Про классы констант, конфигурации и их использование я писал [ТУТ](https://t.me/zhukovsd_it_chat/53243/176984)

**10. package utils**

+ 👍 Действительно содержит только утилитные классы. то есть классы, в которых все методы `static`.

- Утилитные классы должны быть `final` и иметь приватный конструктор- не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.

**11. class PathFinderUtils**, поиск пути

- Как пользоваться классом- непонятно. 

В классе два публичных метода, оба из которых возвращают список координат и, надо полагать, в обоих случаях это путь
```
public class PathFinderUtils {
  public static List<Coordinates> walk(GameMap gameMap, Creature creature) {...}
  public static List<Coordinates> pathFinderA(GameMap gameMap, Creature current) {...}
  //...
}
```
Это класс поиска пути- в нем должен быть только один *публичный* метод, который будет будет возвращать путь до цели.

Если в этом классе два публичных метода, которые используются для разного, значит класс содержит больше одной ответственности и его нужно разделить на два.

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем опроса `Creature` на предмет его меню и скорости
```
public static List<Coordinates> pathFinderA(GameMap gameMap, Creature current) {
  //...
  Coordinates end = current.getTarget().getCoordinates();
  int step = current.getSpeed();
  //...
  return closedList;
}

// ПРАВИЛЬНО:
public static List<Coordinates> pathFinderA(GameMap gameMap, Creature current, Class<? extends Entity> target) {...}
```
Класс поиска должен просто найти путь между точками. 
А как этот путь потом будут использовать и как при этом использовании будет задействован параметр `int speed` из класса `Creature`- явно не зона ответственности алгоритма поиска пути.

- Почему 10? Почему 14? Магические числа нужно объяснять- делай их константами
```
int G = (i == 0 || k == 0) ? 10 : 14;
```

**12. class InputHandler**, начальное конфигурирование приложения через диалог с юзером

- Нарушение DRY. Сплошное, тотальное дублирование кода.

Все методы делают одно и то же- принимают от юзера число в заданном диапазоне. 
```
private static int readSpeed(Scanner scanner, String prompt) {
  while (true) {
    int speed = readInt(scanner, prompt);
    if (speed > 0 && speed <= Constants.SPEED_MAX) {
      return speed;
    } else { System.out.println(Constants.RIGHT_SPEED);  }
  }
}

private static int readDamage(Scanner scanner) {
  while (true) {
    int damage = readInt(scanner, Constants.DMG);
    if (damage > 0 && damage <= Constants.DMG_MAX) {
      return damage;
    } else { System.out.println(Constants.RIGHT_DAMAGE); }
  }
}
```
Общий код выноси во вспомогательные методы:
```
private static int readSpeed(Scanner scanner, String prompt) {
  return readParameter(
    Scanner scanner, 
    Constants.SPEED_INPUT_MESSAGE, 
    Constants.SPEED_FAIL_MESSAGE, 
    Constants.SPEED_MIN, 
    Constants.SPEED_MAX
  );
}

private static int readParameter(Scanner scanner, String prompt,String Constants.RIGHT_SPEED, int min, int max) {
  while (true) {
    int value = readInt(scanner, prompt);
    if (value >= min && value <= max) {
      return value;
    } else { 
      System.out.println(failMessage);  
    }
  }
}
```

**13. class EntityFactory**

+ 👍 Фабрика это хорошо.

- Исключения в сигнатурах методов- мусор 
```
public static <T extends Entity> Entity createEntity(...) throws MapIsFullException, IllegalArgumentException
```
Непроверяемые исключения, а эти исключения непроверяемые, не нужно писать в сигнатурах.  
Проверяемые исключения нужно писать в сигнатурах, поэтому их нужно перехватывать и вместо них кидать непроверяемые.  
Изучи отличия проверяемых и непроверяемых исключений.  
*"ЧК", гл.7, "...проверяемые исключения"*

**14. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. 
Поэтому entities должны хранить координату только начиная с уровня `Creature`.

- Неправильная перегрузка конструктора
```
public Entity(int x, int y) {
  this.coordinates = new Coordinates(x, y);
}

public Entity(Coordinates coordinates) {
  this.coordinates = coordinates;
}

//ПРАВИЛЬНО:
public Entity(int x, int y) {
  this(new Coordinates(x, y));
}

public Entity(Coordinates coordinates) {
  this.coordinates = coordinates;
}
```

**15. class Nothing extends Entity**, класс-заглушка для обозначения отсутствия Entity для геттера в Карте.

- Заглушка, используется как что-то вроде паттерна `NullObject`. 

В принципе, почему бы и нет. Но это несуществующее существо почему-то хранит координату
```
public class Nothing extends Entity {
  public Nothing(int x, int y) {...}
}
```
Несуществующее существо не может хранить координату потому что существа не существует, а значит оно не занимает точку в пространстве.  
Хранение координаты в `Nothing` это (плохой)костыль, потому что оно наследуется от `Entity`, а `Entity` хранит координату, поэтому и `Nothing` вынужден хранить фейковую координату.

Выхода два: нормальный костыль или сделать по-нормальному.

Нормальный костыль: `Nothing` не должен возвращать фейковые координаты, а при попытке их спросить должен бросать исключение. Например:
```
public class Nothing extends Entity {
  public Nothing() {
    super(-1, -1);
  }

  @Override
  public Coordinates getCoordinates() {
    throw new UnsupportedOperationException(/* неподдерживаемая операция*/);
  }
  //остальные переопределения тоже бросают исключения
}
```
Плюсы: из `Nothing` нельзя будет получить данные и поведение, которого у него не должно быть- в отличии от предыдущего костыля.  
Минусы: это костыль, нарушение `LSP` по отношению к предку.

Сделать по-нормальному: `Entity` ничего не хранит, от него наследуются остальные существа и синглтон `Nothing`. Примерно, так:
```
public abstract class Entity {
}

public abstract class RealEntity extends Entity {
  //Реальные Entity, с координатами  
}

public final class NullEntity extends Entity{
  public final static NullEntity INSTANCE = new NullEntity();

  private NullEntity() {
  }
}
```
Плюсы: не костыль, внятная иерархия Entity, `NullEntity` не хранит мусор.  
Минусы: вводится дополнительный уровень иерархии- базовый `Entity`, от которого наследуются реальные существа и фейк.

Это не точное соотвествие по букве паттерну `NullObject`(которых есть несколько вариаций), но идеологически- да. 
Использовать здесь паттерн `NullObject` в классическом виде я не вижу необходимости.

И тогда, среди прочих преимуществ, вместо привязки к константному классу, можно будет юзать заглушку напрямую
```
//БЫЛО:
public class GameMap {
  //...  
  public Entity getEntityAt(Coordinates coordinates) {
    return map.getOrDefault(coordinates, Constants.NOTHING);
  }
}

public class EntityFactory {
  //...
  private static Coordinates findEmptyCoordinates(GameMap gameMap) {
    if (gameMap.getEntityAt(coordinates) == Constants.NOTHING) {
      return coordinates;
    }
    //...  
  }
}

//СТАНЕТ:
public class GameMap {
  //...  
  public Entity getEntityAt(Coordinates coordinates) {
    return map.getOrDefault(coordinates, NullEntity.INSTANCE);
  }
}

public class EntityFactory {
  //...
  private static Coordinates findEmptyCoordinates(GameMap gameMap) {
    if (gameMap.getEntityAt(coordinates) == NullEntity.INSTANCE) {
      return coordinates;
    }
    //...  
  }
}
```
*Фаулер "Рефакторинг", гл.9, "введение объекта Null"*

**16. abstract class Creature extends Entity**

- Неправильное использование константного класса: чтение данных из него напрямую вместо получения необходимых данных в конструктор
```
private Entity target = Constants.NOTHING
```

- Снова костыль: в качестве цели ставится отсутсвие существа. Потом в качестве цели будет подставляться конкретный экземпляр Entity
```
private Entity target = Constants.NOTHING;
```
Не злоупотребляй костылями в проекте, продумывай нормальные алгоритмы.

На самом деле `Creature` в качестве цели должен носить не живого `Entity` на кармане, а его фотографию и потенциальную еду сравнивать с этой фотографией.  
Примерно так:
```
private final Class<? extends Entity> food;

public Creature(Class<? extends Entity> food, ...) {
  this.food = food;
}

public Class<? extends Entity> getFood() {...}
```

- Снова костыль: хранение не только текущей координаты, но и старой- той, которая была до этого
```
private Coordinates oldCoordinates;
```
Это костыль в пользу недостатков алгоритма передвижения существа по карте, а для самого существа эта информация- мусор. 
Лучше доведи до ума алгоритм передвижения.

- Нарушение инкапсуляции. 

Публичными должны быть только те методы, которые используют клиенты. Остальные должны быть протектед или приват. 
Например, метод `public abstract void go()` используют только наследники `Creature`, это часть их внутренней механики. А значит, метод должен быть `protected`.

- Нарушение SRP. 

Модель (а это модель) не должна ничего распечатывать самостоятельно. 
Здесь распечатка идет через создание в `Creature` "интентов", которые печатают в логгер.  
Если модель должна что-то сообщить миру, она может это сделать через паттерн `CallBack`.  
[CallBack луноход](https://t.me/zhukovsd_it_chat/53243/139594)  
[CallBack разрушение](https://t.me/zhukovsd_it_chat/53243/184749)  

**17. class CreateNewWorld implements Action**

Нарушение DRY, дублирование кода. Общий код выности во вспомогательные методы
```
for (int i = 0; i < gameMap.getGameParameters().rockCount(); i++) {
  creatingEntity(gameMap, consoleLogger, Rock.class);
}
for (int i = 0; i < gameMap.getGameParameters().treeCount(); i++) {
  creatingEntity(gameMap, consoleLogger, Tree.class);
}
//еще миллион аналогичных циклов
```
От циклов можно избавиться- перенести их в `creatingEntity(...)`.

**18. class MoveAllCreatures implements Action**

Движение существа должно осуществляться в самом существе- в его методе `makeMove(...)`.  
Либо движение должно осуществляться в классе-мувере.

Сейчас единая ответственность по передвижению существа разнесена на три класса: часть в существе, часть в Карте(метод `void moveCreature(...)`). 
А еще часть- в этом классе, который не просто инициирует движение в креатурах, а управляет процессом движения:
```
@Override
public void execute(GameMap gameMap, ConsoleLogger consoleLogger) {
  //...
  list.forEach(creature -> {
    if (creature.isAlive()) creature.makeMove();
    else creature.death();
    for (Intent intent : creature.getIntents()) {
      intent.apply(gameMap, consoleLogger);
    }
    creature.clearIntents();
  });
}
```
Помести логику перемещения существа в одно место.  
Если она будет находиться в самом существе, то экшен должен только инициировать движение всех креатур, а само передвижение будет полностью делать существо.  
Например, так:
```
@Override
public void execute(GameMap gameMap, ...) {
  List<Creature> creatures = getCreatures(gameMap);  
  for(Creature creature : creatures) {
    creature.makeMove(...);
  }  
}

private List<Creature> getCreatures(GameMap gameMap) {...};
```
Ближайшая аналогия: на дороге сидит кошка, мы к ней подходим и даем пинка- кошка сама побежит, нам не придется лично переставлять ее лапы или указывать как именно она должна передвигаться и что при этом делать. 
Здесь пинок- это инициирование движения.


**19. class Renderer**

В switch-case в default нужно кидать исключение. 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
return switch (entity) {
  case Grass _ -> Constants.GRASS;
  case Tree _ -> Constants.TREE;
  //...
  default -> Constants.DEFAULT;
};
```

**20. class Simulation**

- Сейчас Симуляция использует логгер не через интерфейс, а через конкретную его реализацию- консольный логгер
```
private final ConsoleLogger consoleLogger;

public Simulation(GameParameters parameters) {
  consoleLogger = new ConsoleLogger();
  //...
}
```
Раз в проекте есть интерфейс `Ligger`, то Симуляция должна принимать реализацию этого интерфейса в конструктор. 
Тогда симуляция будет иметь возможность одинаково работать с любым возможным логгером через полиморфизм:

```
private final Logger logger;

public Simulation(GameParameters parameters, Logger logger) {
  this.logger = logger;
  //...
}
```

- Создавай вспомогательные методы, делай программу более простой и понятной
```
if (gameMap.getListOf(Grass.class).size() < gameMap.getGameParameters().grassCount()) {...}

//ПРАВИЛЬНО:
if (hasМалоТравы) {...}

private boolean hasМалоТравы() {
  return gameMap.getListOf(Grass.class).size() < gameMap.getGameParameters().grassCount());
}
```

- Восполнение ресурсов, например травы, нужно делать через отдельный `Action`, который на каждом ходе будет контролировать наличие травы.

**21. class Main**, cодержит точку входа main

Класс, содержащий `main()`, должен только конфигурировать и запускать приложение.
Здесь main содержит бизнес логику приложения- управляет процессом игры.  
Меню и управление командами (стоп/пуск и другие) нужно вынести в отдельный класс.  
*Мартин "ЧК", гл.11, "Отделение main"*

## АРХИТЕКТУРА

Код в проекте не всегда читается легко.
В программе много сложных неочевидных связей между классами- спагетти код. 

Есть циклические связи.
Например, класс `MoveIntent` использует `LogFormatter`, а тот в свою очередь использует сам `MoveIntent`:
```
public class MoveIntent implements Intent {
  //...
  @Override
  public void apply(GameMap gameMap, ConsoleLogger consoleLogger) {
    //...
    consoleLogger.log(LogFormatter.addIntent(this));
  }
}

public class LogFormatter {
  //...
  public static String addIntent(Intent intent) {
    //...
      case MoveIntent info ->
          String.format(..., info.getCreature().getClass().getSimpleName(), info.getCreature().getOldCoordinates().toString(), info.getCreature().getCoordinates().toString());
    //...      
  }
}
```

## ВЫВОД

Есть пробелы в базе, нужно изучить "Oracle Java code conventions".  
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.91(204)  
#ревью #симуляция #костыль #nullobject 