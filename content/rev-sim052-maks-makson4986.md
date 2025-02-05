https://github.com/makson4986/Simulation
[Макс]

Есть над чем поработать.

## ХОРОШО

1. Много разных видов животных и деревьев
2. Есть редактор карты

## ЗАМЕЧАНИЯ

**1. Нейминг**

-Название должно как можно лучше объяснять суть явление. 
"Утилита для ячеек" не объясняет ничего кроме того, что класс как-то использует ячейки в своей работе.
На самом деле класс ищет путь на карте, этот функционал нужно отразить в его названии
```
class CellUtils
```

-Старайся давать стандартные имена стандартным явлениям. 
В данном случае генератор правильно будет назвать EntityFactory, потому что это вариант паттерна Фабрика для создания Entity
```
class Generator
```

-Должно быть названо во множественном числе
```
Map<Coordinates, Cell> checkedCell = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Cell> checkedCells = new HashMap<>();
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  

**2. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками. Исключение- метод equals()
```
if (state != SimulationState.ONGOING) return;

//ПРАВИЛЬНО:
if (state != SimulationState.ONGOING) { 
  return;
}
```

3. Нарушение инкапсуляции. Всегда явно указывай область видимости и final, если переменная final
```
public class Simulation {
  Field field;
  FieldConsoleRenderer renderer;
  //...s
}

//ПРАВИЛЬНО:
public class Simulation {
  private final Field field;
  private final FieldConsoleRenderer renderer;
  //...
}
```

**4. Нарушение конвенции кода.** Методы располагаются черт-те как, вспомогательные приватные методы- выше публичных.

**5. Нарушение DRY**, магические буквы, числа, слова. Вводи константы.
Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов, либо перенеси ее в третий класс
```
//class SettingsFieldAction
if (InputData.inputDefaultOrCustomSimulation() == 1) {
  setDefaultSettings();
}

//class InputData
public static int inputDefaultOrCustomSimulation() {
  while (true) {
    System.out.println("""
      Хотите настроить параметры игры?
      1. Оставить по умолчанию
      2. Настроить""");

    if (Validator.isCorrectChosenAnswer(line, 2)) {...}
    //...
  }
}

//ПРАВИЛЬНО:
//class SettingsFieldAction
private final static int DEFAULT_SETTINGS = 1;
private final static int CUSTOM_SETTINGS = 2;

int settingsMode = InputData.inputDefaultOrCustomSimulation(DEFAULT_SETTINGS, CUSTOM_SETTINGS);
if (settingsMode == DEFAULT_SETTINGS) {
  setDefaultSettings();
}

//class InputData
public static int inputDefaultOrCustomSimulation(int defaultSettingsKey, int customSettingsKey) {
  while (true) {
    System.out.printf("""
      Хотите настроить параметры игры?
      %d. Оставить по умолчанию
      %d. Настроить""", defaultSettingsKey, customSettingsKey);

    if (Validator.isCorrectChosenAnswer(line, customSettingsKey)) {...}
    //...
  }
}
```

*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

-Никогда не возвращай null
```
protected Entity findNearestTarget(Field field) {
  //...
  return null;
}
```
Возврат null повышает риск возникновения NullPointerException в программе

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

**6. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if ((pathToTarget.size() - pathToTarget.indexOf(coordinates) - 1 <= attackRange)
                && (pathToTarget.size() - pathToTarget.indexOf(coordinates) - 1 > 0)) {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(/*args or empty*/)) {...}

private boolean isНазваниеКотороеВсеОбъясняет(/*args or empty*/) {
  return (pathToTarget.size() - pathToTarget.indexOf(coordinates) - 1 <= attackRange)
                && (pathToTarget.size() - pathToTarget.indexOf(coordinates) - 1 > 0);
}
```

**7. record Coordinates(int x, int y)**

+Рекорд для координаты- идеально. Здесь в рекорде нет ничего лишнего(переопределенный toString()- пусть будет)

**8. class Field**, карта симуляции

-Размеры карты класс должен получать в конструктор, а не высасывать из класса настроек
```
private SettingsFieldAction settings;
public int getSize() {
  return settings.getSize();
}
```
**Я уже недавно встречал подобный класс настроек в другом проекте. Скажите тому, кто вас этому научил, что он неправ.**

-При добавлении существа в карту, не проверяет координату на корректность
```
public void setEntity(Entity entity, Coordinates coordinates) {
  field.put(coordinates, entity);
}
```
Сейчас в карту можно вставить существо на координату, выходящую за размер карты
```
Coordinates coordinates = new Coordinates(+100500, -100500);
Карта карта = new Карта(10, 10);
карта.setEntity(coordinates, new Заяц());
```
Перед добавлением существа в карту, нужно проверить координату на корректность.

-При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение.

-При попытке удалить существо по координате, где нет существа, должно бросать исключение.
```
public void removeEntity(Entity entity) {
  field.remove(entity.getCoordinates());
}
```

-Никогда не возвращай null
```
private final Map<Coordinates, Entity> field = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return field.get(coordinates); //может вернуть null
}
```

-Карта хранит в себе экземпляр класса с настройками приложения
```
private SettingsFieldAction settings;
```
Более подробно про этот класс ниже, пока вкратце: в этом классе хранится информация которая касается не только класса Карты(ее размер), но и данные, которые касаются класса SpawnEntityAction.
Таким образом, храня в себе экземпляр SettingsFieldAction, Карта тем самым хранит в себе не только нужную информацию, но еще и лишнюю информацию, которыая ее не касается.

-Также SettingsFieldAction работает напрямую с консолью, печатая туда инфу для юзера и получая от юзера требуемые данные через Scanner.
Поэтому имея зависимость на SettingsFieldAction, Карта становится зависимой от представления и тем самым перестает быть моделью.
Из-за этого использовать класс Карта можно только в среде, которая работает с консолью.

-Нарушение Low Coupling, класс имеет лишние зависимости на большое количество классов: SettingsFieldAction, InputData, Validator, Action.

По-нормальному Карта должна зависеть только от таких классов:
а) Entity - потому что карта их в себе хранит
б) Coordinate - для определения позиции, где находится Entity 
Больше карта не должна знать никаких прикладных классов. 

**9. abstract class Entity**

-Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должен хранить координату только начиная с уровня Creature.

-Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```
public abstract String getEmoji();
``` 
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами.
Спрайты всех существ должны храниться в классе, который распечатывает карту.

**10. abstract class Creature extends Entity**

+Есть возможность прописать несколько съедобных целей, которые будут храниться в списке, это хорошо
```
protected List<Class<? extends Entity>> targetEntities;
```

-targetEntities тоже должен приниматься в конструктор
```
public Creature(int speed, int health) {
  this.speed = speed;
  this.health = health;
  pathToTarget = new ArrayList<>();
}
```

**11. enum HerbivoreType**, хранит виды(типы) травоядных: жираф, лама и т.д. и их спрайты

-Неправильная структура енама
```
public enum HerbivoreType {
  LLAMA,
  GIRAFFE,
  //...
  public String getEmoji() {
    return switch (this) {
      case LLAMA -> "\uD83E\uDD99";
      case GIRAFFE -> "\uD83E\uDD92";
      //...
    };
  }
}
//ПРАВИЛЬНО:
public enum HerbivoreType {
  LLAMA("\uD83E\uDD99"),
  GIRAFFE("\uD83E\uDD92"),
  //...
  
  private final String emoji;

  public HerbivoreType(String emoji) {
    this.emoji = emoji;
  }

  public String getEmoji() {
    return this.emoji;
  }
}
```

**12. class Cell implements Comparable<Cell>**

-Это узел однонаправленного связного списка. Такие узлы принято называть Node. 

**13. class CellUtils**, поиск пути на карте

-Понимать код класса тяжело из-за адски длинных ифов, вводи вспомогательные методы
```
if (Validator.isCorrectCoordinates(coordinates, field.getSize()) && (field.isCoordinateEmpty(coordinates) || coordinates.equals(to))) {...}
```

-Понимать код класса тяжело из-за адски глубокой вложенности, вводи вспомогательные методы
```
for (...) {
  for (...) {
    if (...) {
      //...
      if (...) {
        //...
        if (...) {
          if (....) {
            //тут что-то наконец делаеца полезное (прим. ред.)
          }
        } else {
          //тут делаеца что-то другое
        }
      }
    }
  }
}
```

**14. class SettingsFieldAction extends Action**

-Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники. 
Это своего рода вариация паттерна Command.
Т.е. акции должны быть родственны и одинаково использоваться через полиморфизм.
Поэтому у всех Action'ов должен быть только один публичный метод, в данном случае- public void execute(Field field).

У класса SettingsFieldAction миллион публичных методов, поэтому по своей сути никакого отношения к Action'ам он не имеет.
Класс нужно переименовать, убрать наследование от Action и перенести из пакета actions.

-Что такое этот класс вообще понять трудно 
```
[мем  "что ты такое?" со Шварцнеггером и Хищником]
```
он одновременно содержит настройки приложения(размер карты, количество существ и т.д.) и методы по вводу этих настроек в самое себя через консольный диалог с пользователем.
Класс нужно разделить на два: один будет хранить настройки, а второй будет создавать экземпляр настроек через диалог с пользователем.

**15. class InputData**, класс для ввода данных от пользователя

-Нарушение DRY, два метода, которые по мути делают одно и то же. Их можно заменить на один примерно так
```
public class InputData {

  public static int input(String title, String failMessage, int min, int max) {
    Scanner scanner = new Scanner(System.in);
    while (true) {
      System.out.print(title);
      String line = scanner.nextLine();
      if(isInteger(line)) {
        int number = Integer.parseInt(line);
        if(number>=min && number<=max) {
          return number;
        }
      }
      System.out.println(failMessage);
    }
  }

  private static boolean isInteger(String s) {...}
}
```

**16. class Validator**

-Нарушение SRP. Класс должен только валидировать. Но он кроме этого еще конвертирует данные
```
public static int tryParseToInteger(String string) {
  try {
    return Integer.parseInt(string);
  } catch (NumberFormatException exception) {
    return -1;
  }
}
```

-Не возвращай коды ошибок
```
public static int tryParseToInteger(String string) {
  //...
  return -1;
}
```
*"ЧК", гл.3, п."...вместо возвращения кодов ошибок"*

**17. class Main**

-Здесь не нужно запускать инициализатор Симуляции. InitAction'ы должны запускаться автоматически в Симуляции в конструкторе, либо при вызове метода startSimulation()
```
public static void main(String[] args) {
  Field field = new Field();
  Simulation simulation = new Simulation(field);
  simulation.executeInitialActions();
  simulation.startSimulation();
}

//ПРАВИЛЬНО:
public static void main(String[] args) {
  Field field = new Field();
  Simulation simulation = new Simulation(field);
  simulation.startSimulation();
}
```

## АРХИТЕКТУРА

Использование константных классов и классов настроек.
Раз пошла тенденция использовать константные классы/классы настроек, нужно осветить тему более подробно.

**А-1) Описание проблемы**

Допустим, есть классы Игровая доска и Свинья
```
public class GameBoard {
  private final int width;
  private final int height;

  //конструктор
  //геттеры
}

public class Pig {
  private final int speed;

  //конструктор
  //геттер
}
```

Данные для их инициализации мы хотим держать в одном месте, например в простом константном классе
```
public final class Constants {
  public final static int BOARD_WIDTH = 15;
  public final static int BOARD_HEIGHT = 10;

  public final static int PIG_SPEED = 3;

  private Constants() {}
}
```

Возникает вопрос, как эти настройки перенести из констант в Доску и Свинью?

**А-2) Неправильное использование класса констант**

Распространенная ошибка- обращение к константному классу напрямую
```
public class Main {
  public static void main(String[] args) {
    GameBoard gameBoard = new GameBoard();
    Pig pig = new Pig();
  }
}

public class GameBoard {
  private final int width;
  private final int height;

  public GameBoard() {
    this.width = Constants.BOARD_WIDTH;
    this.height = Constants.BOARD_HEIGHT;
  }

  //геттеры
}

public class Pig {
  private final int speed;

  public Pig() {
    this.speed = Constants.PIG_SPEED;
  }
  //геттер
}
```

Недостатки:

-Нарушение SRP, классы сами себя инициализируют. Для этого они должны знать, куда лезть и где брать нужные данные.
-Нарушение Low Coupling, классы зависят от константного класса и не могут существовать без него. 

-Классы нельзя использовать в других проектах, при переносе в другой проект классы потянут за собой константный класс, иначе они просто не скомпилируются. 
А в новом проекте будут не нужны константы из старого проекта. 

**А-3) Класс настроек или конфигурационный класс**, неправильное использование

Но в данном проекте используется не простой класс с константами, а несколько иная штука. 
Нечто, что можно было бы назвать неким подобием конфигурационного класса(класса настроек), но сути дела это не меняет.
Адаптируя под наш пример с Доской и Свиньей, это выглядело бы примерно так(для простоты сделал без самоконфигуратора, в отличии от оригинального SettingsFieldAction)
```
public class AppSettings {
  private final static int BOARD_WIDTH = 15;
  private final static int BOARD_HEIGHT = 10;
  private final static int PIG_SPEED = 3;

  public AppSettings() {
  }

  public int getBoardWidth() {
    return BOARD_WIDTH;
  }
  public int getBoardHeight() {
    return BOARD_HEIGHT;
  }
  public int getPigSpeed() {
    return PIG_SPEED;
  }
}
```

По форме это будет немного оличаться от классического константного класса, но суть останется той же и недостатки будут те же самые, что при неправильном использовании константного класса
```
public class Main {
  public static void main(String[] args) {
    AppSettings appSettings = new AppSettings();
    GameBoard gameBoard = new GameBoard(appSettings);
    Pig pig = new Pig(appSettings);
  }
}

public class GameBoard {
  private final AppSettings appSettings;

  public GameBoard(AppSettings appSettings) {
    this.appSettings = appSettings;
  }

  public int getWidth() {
    return appSettings.getBoardWidth();
  }
  public int getHeight() {
    return appSettings.getBoardHeight();
  }
}

public class Pig {
  private final AppSettings appSettings;

  public Pig(AppSettings appSettings) {
    this.appSettings = appSettings;
  }

  public int getSpeed() {
    return appSettings.getPigSpeed();
  }
}
```

**А-4) Правильное использование класса констант**

Вне зависимости от того, где хранятся даные для нчальной инициализации классов, эти данные классы должны получать в конструктор
```
public class GameBoard {
  private final int width;
  private final int height;

  public GameBoard(int width, int height) {
    this.width = width;
    this.height = height;
  }

  //геттеры
}

public class Pig {
  private final int speed;

  public Pig(int speed) {
    this.speed = speed;
  }
  //геттер
}
```

Инициализация константами должна происходить при создании объектов(т.е. экземпляров класса)
```
public class Main {
  public static void main(String[] args) {
    GameBoard gameBoard = new GameBoard(Constants.BOARD_WIDTH, Constants.BOARD_HEIGHT);
    Pig pig = new Pig(Constants.PIG_SPEED);
  }
}
```

Из-за того, что Доска и Свинья здесь ничего не знают про класс с константами, они от него никак не зависят.
Поэтому Доску и Свинью теперь можно использовать и с константным классом, и без него
```
public class Main {
  public static void main(String[] args) {
    GameBoard gameBoard = new GameBoard(15, 10);
    Pig pig = new Pig(3);
  }
}
```
Таким образом теперь появилось разделение ответственностей: константный класс только хранит константы, свинья и доска занимаются своими делами и не занимаются дополнительной обязанностью- самоконфигурированием.

**А-5) Инициализация классов дефолтными или пользовательскими данными**

Если настройки могут браться из нескольких мест, например дефолтные или пользовательские, то можно сделать примерно так.
Делаем структуру данных для хранения настроек(отличие структуры данных от объекта: "ЧК", гл.6 )
```
public class AppSettings {
  public int boardWidth;
  public int boardHeight;
  public int pigSpeed;

  @Override
  public String toString() {
    return "AppSettings{" +
        "boardWidth=" + boardWidth +
        ", boardHeight=" + boardHeight +
        ", pigSpeed=" + pigSpeed +
        '}';
  }
}
```

Делаем интерфейс фабрики, которая будет создавать настройки
```
public interface AppSettingsFactory {
  AppSettings get();
}
```

Делаем реализацию с дефолтными настройками
```
public class DefaultAppSettingsFactory implements AppSettingsFactory{
  private final static int BOARD_WIDTH = 15;
  private final static int BOARD_HEIGHT = 10;
  private final static int PIG_SPEED = 3;

  @Override
  public AppSettings get() {
    AppSettings settings = new AppSettings();
    settings.boardWidth = BOARD_WIDTH;
    settings.boardHeight = BOARD_HEIGHT;
    settings.pigSpeed= PIG_SPEED;
    return settings;
  }
}
```

И реализацию, которая будет создавать настройки через диалог с пользователем
```
public class InputAppSettingsFactory implements AppSettingsFactory {
  private final static int BOARD_WIDTH = 15;
  private final static int BOARD_HEIGHT = 10;
  private final static int PIG_SPEED = 3;

  @Override
  public AppSettings get() {
    AppSettings settings = new AppSettings();

    System.out.println("Введите данные");
    String failMessage = "Ошибка ввода";
    settings.boardWidth = input("Ширина карты", failMessage, 3, 99);  //TODO магические числа нужно выести в константы
    settings.boardHeight = input("Высота карты", failMessage, 3, 99);
    settings.pigSpeed = input("Скорость свиньи", failMessage, 1, 5);
    return settings;
  }

  private int input(String title, String failMessage, int min, int max) {
    Scanner scanner = new Scanner(System.in);
    while (true) {
      System.out.printf("%s (%d-%d): ", title, min, max);
      String line = scanner.nextLine();
      if (isInteger(line)) {
        int value = Integer.parseInt(line);
        if(value >=min && value<=max) {
          return value;
        }
      }
      System.out.println(failMessage);
    }
  }

  private static boolean isInteger(String s) {
    try {
      Integer.parseInt(s);
      return true;
    } catch (NumberFormatException e) {
      return false;
    }
  }
}
```

Тогда на старте можно спросить у юзера, в каком режиме он хочет запустить игру- в режиме с настройками по умолчанию, или с пользовательскими настройками, которые он должен ввести собственноручно
```
  private final static int DEFAULT_SETTINGS = 1;
  private final static int INPUT_SETTINGS = 2;

  public static void main(String[] args) {
    int settingsMode = getSettingsMode();
    AppSettingsFactory settingsFactory = getFactory(settingsMode);
    AppSettings settings = settingsFactory.get();
    System.out.println(settings.toString());

    GameBoard gameBoard = new GameBoard(settings.boardWidth, settings.boardHeight);
    Pig pig = new Pig(settings.pigSpeed);
  }


  private static int getSettingsMode() {
    //задаем вопрос, откуда хотим брать настройки,
    //по умолчанию или настроить
    //принимаем ответ 
    //и возвращаем DEFAULT_SETTINGS или INPUT_SETTINGS
  }

  private static AppSettingsFactory getFactory(int settingsMode) {
    return switch (settingsMode) {
      case DEFAULT_SETTINGS -> new DefaultAppSettingsFactory();
      case INPUT_SETTINGS -> new InputAppSettingsFactory();
      default -> throw new IllegalArgumentException();
    };
  }

}
```

Результат работы
```
Хотите настроить параметры игры?
1. Оставить по умолчанию
2. Настроить
2
Введите данные
Ширина карты (3-99): 0
Ошибка ввода
Ширина карты (3-99): 5
Высота карты (3-99): 9
Скорость свиньи (1-5): e
Ошибка ввода
Скорость свиньи (1-5): 6
Ошибка ввода
Скорость свиньи (1-5): 3
AppSettings{boardWidth=5, boardHeight=9, pigSpeed=3}
```

**А-6) Упрощение за счет ухудшения архитектуры**

Можно немного упростить за счет ухудшения архитектуры: убрать интерфейс фабрики, фабрику ручного ввода оставить но сделать статиком ее меод get(). И, наконец, структуру с настройками проинициализировать значениями по умолчанию
```
public class AppSettings {
  public int boardWidth = 15;
  public int boardHeight = 10;
  public int pigSpeed = 3;
}
```

И тогда метод из предыдущего примера будет выглядеть так
```
public class Main {

  private final static int DEFAULT_SETTINGS = 1;
  private final static int INPUT_SETTINGS = 2;

  public static void main(String[] args) {
    int settingsMode = getSettingsMode();
    AppSettings settings = getSettings(settingsMode);
    System.out.println(settings.toString());

    GameBoard gameBoard = new GameBoard(settings.boardWidth, settings.boardHeight);
    Pig pig = new Pig(settings.pigSpeed);
  }

  private static AppSettings getSettings(int settingsMode) {
    return switch (settingsMode) {
      case DEFAULT_SETTINGS -> new AppSettings();
      case INPUT_SETTINGS -> InputAppSettingsFactory.get();
      default -> throw new IllegalArgumentException();
    };
  }

  //oths code
}
```

## ВЫВОД

Местами код трудно читается потому что классы имеют много лишних связей друг с другом.
Для лучшего понимания декомпозиции посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID- по одному ролику на каждую букву.

n.52(120) 
#ревью #симуляция #константныйкласс