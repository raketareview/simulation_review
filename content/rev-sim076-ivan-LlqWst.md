https://github.com/LlqWst/Simulation.git  
[Ivan]

Есть над чем поработать

## ХОРОШО

+ 👍 Карта распечатывается ровно  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim076/img0.png)
+ 👍 Две реализации поиска: алгоритмы BFS и AStar 
+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

- UPPER_SNAKE только для констант, а это не константы
```
private final int MAX_ROW;
private final int MAX_COLUMN;
```

- Названия должны как можно лучше объяснять суть явления
```
GameMap.MAX_ROW;  <-- высота
GameMap.MAX_COLUMN;  <-- ширина

//ПРАВИЛЬНО:
GameMap.HEIGHT;
GameMap.WIDTH;
```

- Название обманывает. Метод не проверяет, живое существо или нет, метод проверяет, имеется ли такое существо в карте
```
private final Map<Coordinates, Entity> entities = new HashMap<>();

public boolean isAlive(Entity entity) {
  //...
  return entities.containsValue(entity);
}
```

- Название класса должно быть существительным
```
class PrintMoves

//ПРАВИЛЬНО:
class MovePrinter
```

- Название класса  должно быть глаголом в повелительном наклонении
```
String entitySprite(Coordinates coordinates)
void moving(Creature creature, Coordinates startCoordinates, Coordinates nextMove) 

//ПРАВИЛЬНО:
String toSprite(Coordinates coordinates)
void move(Creature creature, Coordinates startCoordinates, Coordinates nextMove) 
```

- Название обманывает. Метод не проверяет ввод, он получает команды от пользователя и выполняет дальнейшие действия 
```
void checkInput()
```

- Обычно так называют пакеты или классы, которые трудно объяснить, что это такое: `handler`. В данном случае в пакете хендлер лежит класс, который не используется в проекте.  
Если что-то хочется назвать "хендлером", то сначала нужно для самого себя ясно сформулировать, что конкретно делает этот пакет или класс, а потом дать осмысленное название.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (!isEmpty(coordinates) && isValidCoordinate(coordinates)) {
  return entities.get(coordinates);
} else throw new IllegalArgumentException();

//ПРАВИЛЬНО:
if (!isEmpty(coordinates) && isValidCoordinate(coordinates)) {
  return entities.get(coordinates);
} else {
  throw new IllegalArgumentException();
}
```

**3. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется - в этом случае неважно, будет else или нет, так как программа будет работать одинаково и код выглядит читабельней
```
if (!isEmpty(coordinates) && isValidCoordinate(coordinates)) {
  return entities.get(coordinates);
} else throw new IllegalArgumentException();

//ПРАВИЛЬНО:
if (!isEmpty(coordinates) && isValidCoordinate(coordinates)) {
  return entities.get(coordinates);
} 
throw new IllegalArgumentException();
```

**4. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Creature> creatures = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Creature> creatures = new HashMap<>();
```

**5. record Coordinates(int row, int column)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

**6. class GameMap**

- Нарушение DRY. Дублирование кода в перегруженных конструкторах
```
public GameMap() {
  this.MAX_ROW = 8;
  this.MAX_COLUMN = 12;
}

public GameMap(int maxRow, int maxColumn) {
  //...
  this.MAX_ROW = maxRow;
  this.MAX_COLUMN = maxColumn;
}

//ПРАВИЛЬНО:
public GameMap() {
  this(DEFAULT_HEIGHT, DEFAULT_WIDTH);
}

public GameMap(int maxRow, int maxColumn) {
  //...
  this.MAX_ROW = maxRow;
  this.MAX_COLUMN = maxColumn;
}
```

- Координаты в карте должны начинаться с точки 0,0 а не 1,1
```
if (maxRow < 1 || maxColumn < 1) {
  throw new IllegalArgumentException("maxRow and maxColumn must be greater than 0");
}
```

Википедия по этому поводу пишет:
```
В декартовой системе координат, начало координат — это точка, в которой пересекаются все оси координат. 
Это означает, что все координаты этой точки равны нулю. 
Например, на плоскости она имеет координаты (0,0)
```

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: 
  - подсчитать количество существ 
  - вернуть мапу креатур
  - создать случайную пустую координату
  - `isCoordinatesContains(...)`
  - `boolean isContains(Class<? extends Entity> clazz)`

Наверное, для проекта в целом полезно иметь метод, который создает случайную пустую координату. 
Но этот процесс не имеет никакого отношения к единой ответственности(SRP) карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно использовать случайные пустые координаты.  

Методы чужих ответственностей должны находиться в тех классах, в интересх которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- В данном случае, при попытке вставить существо на невалидную координату, нужно бросать исключение, а не просто игнорировать
```
public void setEntity(Coordinates coordinates, Entity entity) {
  if (entity == null) {
    throw new IllegalArgumentException();
  }
  if (isValidCoordinate(coordinates)) {
    entities.put(coordinates, entity);
  }
}
```
Аналогия: если попытаться вставить данные в массив в ячейку с некорректным индексом, то вылетит исключение.

+ 👍 Этот метод норм
```
public List<Coordinates> getEntitiesCoordinates(Class<? extends Entity> clazz) 
```

- Нарушение SRP. 
Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретным классом представителей и не должна даже знать по именам наследников Entity
```
public HashMap<Coordinates, Creature> getCreatures()
```
Если по какой-то причине нужно получить из карты мапу координата-креатура, что само по себе сомнительно с точки зрения SRP, то нужно делать универсальный метод по аналогии с `List<Coordinates> getEntitiesCoordinates(Class<? extends Entity> clazz)`

- Нарушение SRP. Давай попробуем сформулировать, что делает этот метод
```
public boolean isCoordinatesContains(Coordinates coordinates, Class<? extends Entity> clazz) {
  return isValidCoordinate(coordinates)
      && !isEmpty(coordinates)
      && getEntity(coordinates).getClass() == clazz;
}
```
Этот метод определяет, находится ли на заданной координате существо заданного класса. Наверное, это полезный метод, но!  
Вне зависимости от того, будет этот метод вообще существовать в проекте и тем более в Карте, это не будет влиять на исполние картой своих обязанностей в рамках своего SRP.

- Нарушение SRP. Этот метод нужен кому угодно, но не самой карте для исполнения своих обязанностей по SRP
```
public boolean isContains(Class<? extends Entity> clazz) {
  return entities.values().stream().anyMatch(value -> value.getClass() == clazz);
}
```
Этот метод определяет, есть ли в карте хоть одно существо заданного класса. Для выполнения своих обязанностей в рамках SRP, карте не нужно знать, содержит она в себе хоть одного Зайца и Колобка, или не содержит.

**7. class Parameters**

Нарушение SRP. Божественный класс, который содержит несвязанные друг с другом методы.  
Гибрид константного класса и класса, содержащего прикладную логику.  
Прикладную логику нужно вынести туда, где эти методы составляют часть единой ответственности.  
Если перенести из этого класса такое поведение в другие классы, то здесь останутся только константы и класс станет обычным константным классом.
Конкретный пример избавления от лишних методов в этом классе, напишу ниже.

Про классы констант, конфигурации и их использование я писал тут: https://t.me/zhukovsd_it_chat/53243/176984

**8. abstract class PathFinder**

+ 👍 Абстрактный класс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, AStar и т.д.

+ 👍 Сигнатура метода поиска ок
```
abstract public List<Coordinates> find(Coordinates start, Class<? extends Entity> goal, GameMap gameMap);
```
Единственное, что я бы поменял- порядок входящих аргументов. Чтобы сигнатура читалась более естественно: найти путь в *КАРТЕ* начиная от точки *СТАРТ* до точки соответствующей условиям *ЦЕЛИ*.

**9. public class Entity и его "статичные" потомки Tree/Rock etc**

+ 👍 Все ок.

**10. abstract class Creature extends Entity**

+ 👍 Поиск пути принимает в конструктор, это хорошо.

- Нарушение конвенции кода, константы должны находиться выше обычных полей.

- Метод хода не должен принимать в себя еще и свою координату
```
Coordinates makeMove(Coordinates coordinates, GameMap gameMap);
```
Потому что тогда ты делаешь допущение, что клиентский код всегда правильно будет передавать текущую координату этого существа, а это совсем не факт.  

Свою координату существо должно получать само одним из двух способов: либо дублировать в себе свою координату.  
Но этот способ мне не нравится по многим причинам, начиная от самого факта дублирования координаты в существе и в карте.  
Либо, существо должно считывать свою координату из карты примерно так: `Coordinated coordinates = gameMap.getCoordinates(this);`

- Название обманывает. Этот метод не делает ход, этот метод рассчитывает координату, на которую нужно сделать ход
```
Coordinates makeMove(Coordinates coordinates, GameMap gameMap)
```

**11. class CreatureActivity**

- Сюда вынесено поведение наследников Креатуры(Зайца и Волка):
  - Передвижение
  - Заяц есть траву
  - Волк ест зайца
  - Сообщает, жив или нет

В данном случае это нарушение SRP: разделение единой ответственности на несколько классов, дробление.  
Интерпритировать связку `Creature` + `CreatureActivity` как анемичную модель и сервис для этой модели тоже нельзя, потому что `Creature` все равно еще содержит какое-то поведение.

Класс `CreatureActivity` убрать, код из него вернуть в Креатуру/Зайца/Волка.

**12. abstract class Actions**

👍 Ок.

**13. class MakeMove extends Actions**

- Название метода должно быть существительным, например `MoveAction`

- Мувер должен действовать одним из двух способов.

Первый способ: просто обойти всю карту, найти каждую креатуру и дать ей пинка, чтобы она побежала.
При этом как будет бежать креатура и что при этом делать, не должно волновать Мувер, это все должна делать сама креатура.
Выглядеть это должно примерно так
```
class Мувер реализует Action {

  @Переопределить
  public void execute(Карта карта) {
    List<Creature> creatures = getКреатурыНаКарте(карта);
    for(Creature c: creatures) {
      c.бежатьЖивотное(/* args */);  //даёт пинка
    }
  }

  private static List<Creature> getКреатурыНаКарте(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

Второй способ: Мувер должен содержать в себе весь код по передвижению существа. Такой мувер будет своеобразным сервисом, который будет брать на себя все заботы по перемещению существа на карте. 
Типа, как человек передвигает фигуры на шахматной доске, где сами фигуры лишены собственной воли.

В данной риализации мы видим ни то, ни сё.

**14. AddNewEntities extends Actions**

👍 Создание существ через `Supplier` это хорошо.

**15. class ParseIntInput**

Я вообще не смог понять, что это такое и зачем. Но одно ясно точно: класс парсит не Input, а Строку и не в int а в Object.

**16. class Menu**

Вот метод, который использует тот таинственный парсер
```
private void checkInput() {
  String line = scanner.nextLine();
  Object answer = ParseIntInput.parse(line, START, EXIT);
  if (isInputInt(answer)) {
    int input = (int) answer;
    commandSelection(input);
  } else {
    printErrorMessage();
  }
}
```
Сформулируем, что делает этот метод: он принимает команду от пользователя и выполняет ее, если команда корректна. Если некорректна- пишет сообщение об ошибке.

Переделаем, чтобы смысл происходящего был более понятен
```
private final static String START = "1";
private final static String PAUSE = "2";
//...

  private void somename() {
    String command = scanner.nextLine().toUpperCase(); <-- перевод в верхний регистр на случай буквенных команд
    switch(command) {
        case START -> start();
        case PAUSE -> pause();
        //...
        default -> printErrorCommand();
    }
  }
```

**17. class GameMapRenderer**

Нарушение SRP на уровне метода
```
public String entitySprite(Coordinates coordinates) {
  if (gameMap.isEmpty(coordinates)) {
    return EMOJI_EARTH;
  }

  return switch (gameMap.getEntity(coordinates).getClass().getSimpleName()) {
    case "Herbivore" -> EMOJI_HERBIVORE;
    case "Predator" -> EMOJI_PREDATOR;
    //...
    case null, default -> throw new IllegalArgumentException();
  };
}
```
Метод делает несколько дел сразу, а должен делать только одно- преобразовывать существо в спрайт. Примерно, так
```
public String toSprite(Entity entity) {
  return switch (entity.getClass().getSimpleName()) {
    case "Herbivore" -> EMOJI_HERBIVORE;
    case "Predator" -> EMOJI_PREDATOR;
    //...
    default -> throw new IllegalArgumentException("illegal entity: " + entity);
  };
}
```
Использование:
```
for (int column = 0; column < gameMap.getMaxColumn(); column++) {
  line.append(ANSI_BLACK_BACKGROUND);

  Coordinates coordinates = new Coordinates(row, column);
  if(gameMap.isEmpty(coordinates)) {
    line.append(EMOJI_EARTH);
  } else {
    Entity entity = gameMap.getEntity(coordinates);
    line.append(toSprite(entity));
  }
}
```

**18. class Simulation**

👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий. 
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе симуляции
```
Simulation(GameMap gameMap, List<Actions> initActions, List<Actions> turnActions)
```

**19. class Main**

+ 👍 Только создает и запускает симуляцию, это хорошо.

- Вот здесь неправильно используется класс `Parameters`. В константном классе не должно быть методов типа `getRandomSpeed()`
```
Supplier<Entity> supplierPredator = () -> new Predator(Parameters.getRandomSpeed(), Parameters.getRandomDamage(), Parameters.getRandomRange(), new AStarAlgorithm());

//ПРАВИЛЬНО:
Random random = new Random();

int speed = random.next(Parameters.PREDATOR_MIN_SPEED, Parameters.PREDATOR_MAX_SPEED);
int damage = random.next(Parameters.PREDATOR_MIN_DAMAGE, Parameters.PREDATOR_MAX_DAMAGE);
int range = random.next(Parameters.PREDATOR_MIN_RANGE, Parameters.PREDATOR_MAX_RANGE);

Supplier<Entity> supplierPredator = () -> new Predator(speed, damage, range, new AStarAlgorithm()); 
```

## АРХИТЕКТУРА

Вместо сомнительного класса `ParseIntInput` могу предложить концепцию диалогов.  
Диалог это процесс ввода данных пользователем до тех пор, пока ввод не будет соответствовать заданным требованиям.  
Эти универсальные диалоги будут полезны в любой консольной программе.

Делаем интерфейс диалога
```
public interface Dialog<T> {
  T input();
}
```

Теперь делаем реализацию, при которой пользователь должен будет ввести одну из заданных строк
```
public class StringSelectDialog implements Dialog<String> {
  private final String title;
  private final String failMessage;
  private final List<String> keys;
  private final Scanner scanner = new Scanner(System.in);

  public StringSelectDialog(String title, String failMessage, List<String> keys) {
    this.title = title;
    this.failMessage = failMessage;
    this.keys = keys;
  }

  public StringSelectDialog(String title, String failMessage, String... keys) {
    this(title, failMessage, List.of(keys));
  }

  @Override
  public String input() {
    System.out.println(title);
    while (true) {
      String s = scanner.nextLine();
      for (String key : keys) {
        if (key.equalsIgnoreCase(s)) {
          return key;
        }
      }
      System.out.println(failMessage);
    }
  }
}
```
Работать это будет так:
```
public static void main(String[] args) {
  List<String> keys = List.of("start", "stop", "pause");

  String title = "Введите команду %s: ".formatted(keys);
  String failMessage = "Неизвестная команда";
    
  Dialog<String> dialog = new StringSelectDialog(title, failMessage, keys);
  String command = dialog.input();

  System.out.println("Введена команда: " + command);
}

//РЕЗУЛЬТАТ:
Введите команду [start, stop, pause]: 
ssstart
Неизвестная команда
111
Неизвестная команда
STarT
Введена команда: start
```

Можно сделать аналогичные диалоги для других типод данных: Integer, Char, Boolean и т.д.  
Более подробно в моем стриме начиная с 6:28 тут: https://t.me/zhukovsd_it_chat/53243/176204

## ВЫВОД

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.76(165)  
#ревью #симуляция #диалоги #стримдиалоги