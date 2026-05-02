https://github.com/RocknRollNotDead/Simulation2  
[Тёма]

В целом- удручающе.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. В консоли Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim151/img0.png) 

2. В консольной версии не реализована работа с потоками и выполнение старт/стоп во время работы симуляции.  
Хотя это требование тоже есть в ТЗ

## ХОРОШО

+ 👍 Несколько разных интерфейсов: консоль, WEB, Windows (последние два- навайбкодены)
+ 👍 Windows UI красивый, возможно лучший, что я тут видел(спасибо AI)  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim151/img0.png) 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Если в проекте есть класс `Position`, то геттеры с именем, включающим это название, должны возвращать экземпляры этого класса.
Если разные концепции называются одним и тем же именем, это приводит к путанице
```java
public class Position {
  //...

  public int[] getPosition() {  <-- Этот метод возвращает не экземпляр Position
    return new int[]{x, y};
  }
}
```

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относится.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
String stringName = clazz.getSimpleName().toUpperCase() + "_MAX_LEVEL";
Position[] arrPos

//ПРАВИЛЬНО:
String name = clazz.getSimpleName().toUpperCase() + "_MAX_LEVEL";
Position[] positoins
```

- Хуже венгерской нотации только венгерская нотация, которая обманывает.  
Здесь в названии переменной имеется слово "List", но эта переменная не принадлежит к типу List, это Set
```java
Set<Entity> eatingList = new HashSet<>();

//ПРАВИЛЬНО:
Set<Entity> eatingEntities = new HashSet<>();
```

- Частица "To" используется в методах конвертации: `toList()`, `toString()`, `toInt()`, `meterToFeet()`.   
В остальных случаях воздерживайся от этого. Иначе кажется, что этот метод конвертирует id в счётчик
```java
public void incIdToCounter(Class<?> clazz) {
  counter.incId(clazz);
}

//ПРАВИЛЬНО:
public void incIdCounter(Class<?> clazz) {
  counter.incId(clazz);
}
```

- Не сокращай названий без особой необходимости
```java
void createEnts() 

//ПРАВИЛЬНО:
void createEntities() 
```

- Оканчиваться на "able"(ble) могут только названия интерфейсов. Названия классов не должны так оканчиваться
```java
class Edible
```

- Называй поля нормальными английскими словами, translitom ne nazyvay
```java
int znakX = random.nextInt(2);
```

- "Фон" по-английски это "Background"
```java
String colorFon = "\u001B[40m";

//ПРАВИЛЬНО:
String backgroundColor = "\u001B[40m";
```

- Придерживайся единообразия
```java
public static final String reset = "\u001B[0m";
public static final String colorFon = "\u001B[40m";

//ПРАВИЛЬНО:
public static final String resetColor = "\u001B[0m";
public static final String backgroundColor = "\u001B[40m";
```

- Константы нужно писать стилем UPPER_CASE
```java
private static final String gray = "\u001B[97m";

//ПРАВИЛЬНО:
private static final String GRAY = "\u001B[97m";
```

- Придерживайся единообразия. Называй классы или во множественном, или в единственном числе
```java
class Animals
class Tree
class Wolf

//ПРАВИЛЬНО:
class Animal
class Tree
class Wolf
```

- Не называй переменные "object"/"objects".

В java всё- объекты. 

Поэтому назвать что-то "объекты" это все равно, что никик не назвать.  
Название должно объяснять суть переменной
```java
Map<Position, Entity> objects

//ПРАВИЛЬНО:
Map<Position, Entity> entities //или positionWithEntity 
```

- Это название читается как "установить Pos(ition)" и похоже на обычный сеттер, но это не сеттер
```java
Set<Position> setPos

//ПРАВИЛЬНО:
Set<Position> positions
```
Префикс "set" может быть только у названия метода-сеттера, больше нигде.

- В названия не нужно вставлять частицы "Of", "The" и т.д.

Это только делает названия более громоздкими.  
Если конечно там "Of" не используется в контексте valueOf()
```java
class OutForConsole

//ПРАВИЛЬНО:
class ConsoleOutput
```

- Название обманывает. Метод возвращает не Move, а Position
```java
public Position getMove()

//ПРАВИЛЬНО:
public Position getPosition()
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Каждое поле объявляй в отдельной строке
```java
private final int width, height;

//ПРАВИЛЬНО:
private final int width;
private final int height;
```

**3. Создавай вспомогательные методы, делай программу более простой и понятной**
```java
if (counter.getCount(clazz) < counter.getMaxCount(clazz) && (objects.get(new Position(x, y)) == null)) {
  //какие-то действия
}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(...)) {
  //какие-то действия
}

private boolean isНазваниеКотороеВсеОбъясняет(...) {
  return counter.getCount(clazz) < counter.getMaxCount(clazz) && (objects.get(new Position(x, y)) == null;
}
```
Даже если метод состоит из одной строки, но при этом делает код программы читабельней, то этот метод имеет право на существование.

**4. Исключения**

- Перехватывай конкретное исключение.

Всегда перехватывай не базовое, а конкретное исключение, которое может сгенерировать код.  
В данном случае метод `Integer.parseInt()` может бросить не какое угодно исключение, а конкретное- NumberFormatException
```java
try {
  return Integer.parseInt(env.get(stringName));
} catch (RuntimeException e) {
  log.error("Не удалось найти {}", stringName);
  return 0;
}

//ПРАВИЛЬНО:
try {
  return Integer.parseInt(env.get(stringName));
} catch (NumberFormatException e) {
  log.error("Не удалось найти {}", stringName);
  return 0;
}
```

**5. Форматирование строк**

Если нужно печатать или создавать строку с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, 
используй форматирование- тогда сразу будет виден весь шаблон
```java
log.info("object was created: "/* + clazz.getSimpleName()*/ + entity.getSymbol() + " " +
    entity.getPosition().getX() + " " + entity.getPosition().getY() + " count "
    + counter.getCount(clazz) + "  maxCount " + counter.getMaxCount(clazz));

//ПРАВИЛЬНО:
String message = "object was created: %s %d %d count %d maxCount %d".formatted(
    entity.getSymbol(), 
    entity.getPosition().getX(), 
    entity.getPosition().getY(), 
    counter.getCount(clazz), 
    counter.getMaxCount(clazz)
);

log.info(message);
```

**6. class Config**, конфигурационный класс

- Соблюдай требования утилитных классов.

Это утилитный класс, он содержит только статические методы.  
Утилитные классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- Нарушение DRY. Дублировние логики, используй вспомогательные методы
```java
public static int getMaxElements(Class<?> clazz) {
  String stringName = clazz.getSimpleName().toUpperCase() + "_MAX_COUNT";
  try {
    return Integer.parseInt(env.get(stringName));
  } catch (RuntimeException e) {
    log.error("Не удалось найти {}", stringName);
    return 0;
  }
}

public static int getMaxLevel(Class<?> clazz) {
  String stringName = clazz.getSimpleName().toUpperCase() + "_MAX_LEVEL";
  try {
    return Integer.parseInt(env.get(stringName));
  } catch (RuntimeException e) {
    log.error("Не удалось найти {}", stringName);
    return 0;
  }
}

//ПРАВИЛЬНО:
public static int getMaxElements(Class<?> clazz) {
  return someMethod(clazz, "_MAX_COUNT");
}

public static int getMaxLevel(Class<?> clazz) {
  return someMethod(clazz, "_MAX_LEVEL");
}

private static int someMethod(Class<?> clazz, String key) {
  String name = clazz.getSimpleName().toUpperCase() + key;
  try {
    return Integer.parseInt(env.get(key));
  } catch (RuntimeException e) {
    log.error("Не удалось найти {}", key);
    return 0;
  }
} 
```

- Если не получилось прочитать значения из файла, обозначь это внятной реакцией.

Сейчас, если не удается прочитать файл настроек, методы возвращают ноль.  
Что это, код ошибки или значение по умолчанию- неясно
```java
public static int getMaxLevel(Class<?> clazz) {
  String stringName = clazz.getSimpleName().toUpperCase() + "_MAX_LEVEL";
  try {
    return Integer.parseInt(env.get(stringName));
  } catch (RuntimeException e) {
    log.error("Не удалось найти {}", stringName);
    return 0;  <-- Код ошибки или значение по умолчанию?
  }
}
```
Клиентский код должен четко понимать, получил ли он актуальные значения из файла настроек или что-то пошло не так.  
Сейчас клиенту это понять без лишних пояснений невозможно.  
Получив число "0" он не может быть увереным в том, что это число актуальное и взято из файла, а не дефолтное.

Короче, не извращайся.  
Лучше просто кидай исключение- так программа будет работать предсказуемо.

- Конфигурационный класс в программе используется неправильно.

Другие класс напрямую лезут в константный классы
```java
public class Simulation {
  public Simulation() {
    this(Config.getWidth(), Config.getHeigh());  <-- Обращается напрямую в конфиг
  }
  //...
}

public class Berries extends Edible {
  public static final int MAX_LEVEL_LIFE = Config.getMaxLevel(Berries.class);   <-- Обращается напрямую в конфиг
  //...
}
```

Классы не должны напрямую лезть в конфиг за своими данными, они должны получать необходимые данные в конструктор.  
Условные примеры:
```java
public class Settings {
  public static final int ROOM_NUMBERS = 5;
}

//ПЛОХО:
public static void main(String[] args) {
  House house = new House();
  //oth code
}

public class House {
  private final int roomNumbers;

  public House() {
    this.roomNumbers = Settings.ROOM_NUMBERS;
  }

  public int getRoomNumbers() {
    return roomNumbers;
  }
}

//ХОРОШО:
public static void main(String[] args) {
  House house = new House(Settings.ROOM_NUMBERS);
  //oth code
}

public class House {
  private final int roomNumbers;

  public House(int roomNumbers) {
    this.roomNumbers = roomNumbers;
  }

  public int getRoomNumbers() {
    return roomNumbers;
  }
}
```

**7. class Position**

+ 👍 Хранит только необходимые поля(x,y) это хорошо. 

- Метод getPosition() тут вообще не нужен, потому что для осей x,y есть отдельные геттеры
```java
public class Position {
  //...

  public int[] getPosition() {  <-- Это лишнее
    return new int[]{x, y};
  }

  public void setPosition(int x, int y) {
    this.x = x;
    this.y = y;
  }

  public int getX() {...}

  public int getY() {...}
}
```

- Этот класс стоит сделать иммутабельным
```java
private int x;
private int y;

//ЛУЧШЕ:  
private final int x;
private final int y;
```
*Блох "Java. Эффективное программирование", изд.3, гл.4.3.*
```java
"Классы должны быть неизменяемыми, если только нет очень важной причины, чтобы сделать их изменяемыми" - Блох.
```

- Класс стоит сделать record'ом.

Координата это очень простой класс, его лучше сделать record'ом.  
Рекорды по умолчанию правильно реализуют `equals()`, `hashCode()` etc.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**8. class Simulation**, карта игры.

- Нарушение ТЗ.

Код в классе не соответствует описанию класса "Simulation".  
Код в классе скорее соотвентствует классу "Map" из ТЗ.  
"Скорее" потому что это божественный класс, а значит про него трудно сказать что-то определенное.  
Но для простоты будем его критиковать с точки зрения требований к классу Карта.

- Коммментарий с описанием класса.

Так как из кода класса трудно понять, что это такое, в классе есть специальный большой камент, который призван это объяснить
```java
  /**
   * forDelete - туда попадают сущности, которые надо удалить, чтобы удалять их не сразу, а один ход = одно удаление
   * у каждого класса
   * Счётчик следит за тем, сколько сущностей конкретного класса
   * eatingList - лист тех, кого сьедают. Управляется из классов <? наслед Animals>
   * objsMap хранит основную полноценную карту обьектов.
   * *
   * По objsMap мы иттерируемся, но записываем все объекты в newObsMap, и после итерации тупо записываем
   * newObjsMap в objsMap.    (как раз из-за objsMap = newObjsMap, objsMap не final)
   * eatingEvents создал ИИ помощник для того, чтобы прописывать события съедения (а также смерти от голода) в веб-версии
   *
   */
```
 
Но истинное назначение этого комментария- оправдать плохой код.  
Хороший код не нуждается в комментариях, тем более в таких пространных.  
Код должен объяснять сам себя через хороший нейминг и простые понятные методы.

- Нарушение SRP, божественный класс. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
Выполнить ход, добавить death событие(что бы это ни значило), логирование, увеличение счетчика id...  
И миллион других методов, которые не имеют никокго отношения к единой ответственности Карты и тем самым делают её божественным объектом.  

Наверное, для проекта в целом полезно иметь метод, который увеличивает счетчик уникальных номеров.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно увеличивать счетчики уникальных номеров.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Класс хранит кучу лишних полей.

Исходя из предыдущего замечания, класс хранит кучу ненужных для его ответственности полей.  
Чтобы класс Карты функционировал в рамках своих полномочий, ему достаточно иметь вот такие поля:
```java
private final Map<Position, Entity> entities;
private final int width;
private final int height;
```

Для всех операций с существами(вставить, удалить, получить) достаточно только одной коллекции типа `Map<Position, Entity>`.  
Не нужно персональных коллекций для едящих, водоплавывающих и любых других типов существ.

Также не нужно одновременно с главной коллекцией держать обратную ей коллекцию вида `Map<Entity, Position>`.  

Наличие двух коллекций, прямой и обратной, может немного увеличить скорость выполнения некоторых операций.  
Но при этом делает код более сложным и запутанным. Потому что нужно сделать синхронизацию хранения данных в этих коллекциях.    
Учитывая, что реально на производительности это никак не отобразиться(подавляющую часть времени программа будет стоять в паузе),
такой код подпадает под определение антипаттерна "Преждевременная оптимизация".

- Нарушение SRP, зависимость модели от представления.

Модель(а это модель) не должна НАПРЯМУЮ печатать тексты куда-то, напрмер, в логи 
```java
public class Simulation {
  private static final Logger log = LoggerFactory.getLogger(Simulation.class);
  //...

  public Simulation(int width, int heigh) {...}

  public void addEatingEvent(Entity eater, Entity eaten) {
    String event = eater.getSymbol() + " eat " + eaten.getSymbol();
    eatingEvents.add(event);
    log.info(event);
  }
}
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах нужно будет менять код в этом классе, чтобы вывод осуществлялся по правилам этой среды, то есть, это лишняя причина для изменения класса.
В других средах(напр. Андроид) эту модель нельзя будет использовать- она там просто не скомпилируется. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

Если модель должна что-то сообщить миру, она может это сделать через паттерн [CallBack](https://t.me/zhukovsd_it_chat/53243/139594).  

Вот к чему приводит зависимость модели от представления, карта рисуется в виндовый интерфейс, а модель [пишет в консоль](https://github.com/raketareview/simulation_review/blob/master/content/rev-sim064-davidtagirov-DavidTagirov.md).

Если модель должна что-то сообщить миру, она может это сделать через паттерн CallBack.  
[CallBack луноход](https://t.me/zhukovsd_it_chat/53243/139594)  
[CallBack разрушение](https://t.me/zhukovsd_it_chat/53243/184749)

Если на этом этапе разобраться с колбеком слишком сложно, то хотя бы огородись абстракцией от конкретной среды вывода:
```java
//ПЛОХО:
public class Simulation {
  private static final Logger log = LoggerFactory.getLogger(Simulation.class);  
  //...
  public Simulation(int width, int heigh) {...}

  public void addEatingEvent(Entity eater, Entity eaten) {
    String event = eater.getSymbol() + " eat " + eaten.getSymbol();
    eatingEvents.add(event);
    log.info(event);
  }
}

//ПЛОХО, НО ЛУЧШЕ:
public class Simulation {
  //...
  private final Log log;

  public Simulation(int width, int heigh, Log log) {...}

  public void addEatingEvent(Entity eater, Entity eaten) {
    String event = eater.getSymbol() + " eat " + eaten.getSymbol();
    eatingEvents.add(event);
    log.out(event);
  }

  public interface Log {
    void out(String message);
  }
}
```
Это тоже плохо. Но не так ужасно, как с печатью в стандартный Logger непосредственно из модели.

- Нарушение OCP.

Из-за того, что класс знает много лишнего(нарушает SRP), при добавлении новых видов entity в проект, 
нужно будет изменить код в этом классе. То есть, класс открыт для изменений.  
Чтобы не забыть об этом, здесь есть дадже специальный комментарий 
```java
{
  //...
  types.add(Wolf.class);
  types.add(Berries.class);

  // добавить еще классов когда они будут готовы

  for (Class<?> clazz : types) {
    counter.addClass(clazz);
  }
}
```

- Нарушение Low Coupling. Константный класс в программе используется неправильно.

Этот класс напрямую лезет в константый класс, а должен *только* получать необходимые данные в конструктор
```java
public class Simulation {
  //...

  public Simulation() {  <-- Лишний конструктор
    this(Config.getWidth(), Config.getHeigh());
  }

  public Simulation(int width, int heigh) {  <-- Единственный нужный конструктор
    this.width = width;
    this.height = heigh;
    this.creator = new Creator(counter, width, heigh);
  }
}
```

- Карта, которая не карта.

Как я писал, этот класс соответствует антипаттерну "Божественный объект", который делаем много разного.  
Но с точки зрения ООП фактически это не объект, а модуль из процедурного стиля программировния.

Здесь я даже не нашел методов, которые персонально добавляют существо в карту по координате,
удаляют существо по координате и возвращают существо по координате и координату по существу(опционально).  
Нашел только публичные методы, которые возвращают слепки мап
```java
public Map<Position, Entity> getObjsMap()
public Map<Position, Entity> getNewObjsMap()
```
То есть, этот класс плох и в качестве Карты игры.  

- В целом, класс нечитаем. Что в нём происходит(а главное- зачем), понять невозможно
```java
private int getNumPriority(Map.Entry<Position, Entity> e) {
  return switch (e.getValue()) {
    case EnvironmentObject environmentObject -> 1;
    case PeacefulAnimal peacefulAnimal -> 2;
    case Predator predator -> 3;
    case null, default -> 0;
  };
}
```

В качестве эксперимента, попробуй открыть этот код после перерыва в пару недель и ты сам тут ничего не поймешь.  

**9. abstract class Entity**

- Принимай координату как объект, а не как запчасти объекта
```java
public Entity(int x, int y) {
  this.position = new Position(x, y);
}

//ПРАВИЛЬНО:
public Entity(Position position) {
  this.position = position;
}
```

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```java
public abstract String getSymbol();
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

- Нарушение SRP и ООП дизайна в наследовании.  

Это тоже божественный класс, который содержит методы на все случаи жизни для всех своих наследников.

Класс содержит координату, но координата нужна только тому существу, которое ходит.  
Поэтому координата должна быть начиная с уровня Creature.

Класс содержит метод ходьбы, но у Entity есть и ходящие и не ходящие наследники.  
Метод ходьбы должен быть только у ходящих наследников.

То же самое с методом `boolean isDead()`.  
По ТЗ, есть классы, которые умирают(поедаются)- Grass и Creature.  
И все остальные, которые не умирают.  
Умирать и сообщать освоем умирании должны только те entity, для которых предусмотрено умирание.

Поэтому иерархия классов Entity построена неправильно.  
Нельзя функционал всех наследников объявить на базовом уровне.  
Функционал нужно вводить на тех уровнях иерархии, где он появляется.

Вот таким должен быть идеальный Entity и его простые неходячие наследники:
```java
public abstract class Entity {
}

public class Tree extends Entity {
}
```

+ 👍 Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**10. class Iwe extends EnvironmentObject**

Нарушение SRP. Хранит какие-то цвета
```java
  private static final String reset = "\u001B[39m"; // сброс только буквы
  private static final String gray = "\u001B[97m";
```

**11. Общие проблемы в наследниках Entity**

- Зависимость от представления- печатают непосредственно в логгер
```java
public abstract class Animals extends Entity {

  private static final Logger log = LoggerFactory.getLogger(Animals.class);
  //...
  
  public Position doMove(Simulation simulation) {
    //...
    log.trace(" " + this.getClass().getSimpleName() + getId() + " " + newPosition + "  lifes: " + lifes);
  }
}
```

- Неправильно используют константные классы- лезут в них напрямую
```java
public class Iwe extends EnvironmentObject {
  public static final int MAX_LEVEL_LIFE = Config.getMaxLevel(Iwe.class);
  //...
}
```

**12. abstract class Animals extends Entity**

- Нарушение ТЗ.

По ТЗ этот класс должен называться "Creature".  
Почему здесь переименовано в "Animals", неясно.

Или ты невнимательно читал ТЗ или соблюдать его не считаешь нужным.  
И то и другое плохо- на любой работе придется соблюдать ТЗ и его требования.

- Нарушение SRP. Большой класс с кучей кода, который тяжело читается.

Поиск пути нужно вынести в отдельный класс.  
Чеклист ТЗ:
```java
Проблемы и ошибки в коде:
...
Реализация алгоритма поиска пути внутри классов существ. Следует вынести это в отдельный класс
```

- Никогда не возвращай null
```java
private Position leavingFromDanger(Set<Position> setPos, Position predPos, Simulation simulation) {
  return setPos.stream()
    .max(Comparator
    .comparingInt(...)
    .orElse(null);  <-- Возрат null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Неправильный toString(). 

toString() должен быть стандартным(как делает IDE по Alt+Ins) и использоваться только для отладки.  
toString() должен содержать значение всех значимых полей класса.  
toString() не должен использоваться для представления, за исключением предельно простых классов, вроде PhoneNumber
```java
public String toString() {
  return getClass().getSimpleName() + getId() + " " + getPosition();
}

//ПРАВИЛЬНО:
public String toString() {
  return "Animals{" +
    "id=" + id +
    ", lifes=" + lifes +
    ", isDead=" + isDead +
    ", Position=" + getPosition() +
    // + все остальные значимые поля
    '}';
}
```
*Блох, "Java. Эффективное программирование", изд.3, гл.3.3*

- Нет никакого смысла хранить Simulation в виде поля.

Симуляция(а вообще-то это должны выть Карта), нужна только для совершения хода.  
Поэтому Симуляцию(Карту) нужно только принимать в `makeMove(...)`, её не нужно хранить в виде поля в классе Animals
```java
public abstract class Animals extends Entity {
  protected Simulation simulation = null;  <-- Хранить это в виде поля класса- лишнее
  //...

  public Position doMove(Simulation simulation) {
    if (this.simulation == null) {
      this.simulation = simulation;
      //...
    }
  }
}
```

- Код в классе читается плохо. Он запутанный и непонятный.

Для начала нужно вынести отсюда поиск пути.  
А потом посмотреть, останется ли что-то еще лишнее в этом классе.

**13. Пакет util**

Содержит не только утилитные классы.  
Например, `class Randomizer` не утилита.

Утилитными являются классы, которые содержат только статические методы и не содержат изменяемых полей. 

**14. class Creator**

Стандартным явлениям давай стандартные названия. Этот класс- фабрика существ
```java
public class Creator {
  //....

  public Entity execute(Class<? extends Entity> clazz, Map<Position, Entity> objects) {...}
}

//ПРАВИЛЬНО:
public class EntityFactory {
  //....

  public Entity create(Class<? extends Entity> clazz, Map<Position, Entity> objects) {...}
}
```

**15. class OutForConsole**

- Обычно это называется рендерером, например: ConsoleRenderer.

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < simulation.getHeight(); i++) {
  for (int j = 0; j < simulation.getWidth(); j++) {
    if (map.get(new Position(j, i)) == null) {...}
  }
}

//ЛУЧШЕ:
for (int y = 0; y < simulation.getHeight(); y++) {
  for (int x = 0; x < simulation.getWidth(); x++) {
    if (map.get(new Position(x, y)) == null) {...}
  }
}
```

**16. Классы, которых нет**

Нет классов, которые должны быть по ТЗ.

**17. Непонятные классы**

Классы, которые непонятно, что это такое: EntityCounter, EntityCompare.  
Какую сущность из себя являют эти классы, мне неясно. Есть подозрение, что никакую.

Класс Randomizer, тоже не представляет из себя объектно-ориентированную сущность.  
Это просто контейнер(в процедурном стиле) для двух специфических методов создания "чего-то случайного".  
В данном случае это что-то- случайная координата.

**18. class PlayingArea**

Мусор- класс, который не используется в проекте. Просто существует.

**19. Пакет web**

Вайбкод.

**20. class SimulationApp extends Application**

Класс визуализации Windows UI с помощью библиотеки JavaFX.

Вайбкод.

Обсуждать этот класс не имеет смысла, как и пакет "web", потому что это сделал AI и твоей заслуги в этом нет.  

Замечу только, что при вайбкодинге программист не отвечает за полученный код и полагается на компетенции ИИ.  
Здесь ИИ навайбкодил действительно очень классный красивый UI с прикольными эффектами анимации.  
При этом внутри- полное архитектурное дно. 

Не нужно ориентировать на этот вайбкод как пример того, как нужно писать в JavaFX.

**Программа на JavaFX должна быть построена по архитектуре MVC.**

То есть, все классы при такой архитектуре должны соответствовать одному из трех слоёв: model, view, controller.
Не будем душнить, что можно писать и в других архитектурах, просто изначально для JavaFX предусматривалась именно MVC.

Навайбкоденый `class SimulationApp` совмещает в себе два слоя: view и controlller.  
При этом содержит еще и точку входа main, что есть дно дна.

## ВЫВОД

Чтобы оценить эту реализацию, нужно сначала понять мотивацию её написания.

Если это написано чисто по фану с целью интересно провести время, то почему бы и нет.  
Как развлечение- прикольное занятие 👍

Если поставить галочку прохождения второго задания- тоже норм.  
Устроиться работать магом спринга наверное можно будет и без понимания базового ООП.  
Потому что Spring всё равно заставит писать в жёстких рамках собственной архитектуры.

Если цель была разобраться в ООП, то это провал.  
Тут нет ООП вообще, даже на уровне понимания того, как строить иерархию классов.

В этом проекте игнорируется ТЗ и сделана своя декомпозиция.  
В ТЗ классы декомпозированы по правилам ООП.  
Здесь классы это в большинстве своем просто контейнеры функций в стиле процедурного программирования.

Не видно понимания приципов SOLID.

В ридми есть много оговорок по поводу кода, например:
```
Map
Да, в задании писалось, что map должен быть отдельным классом. Но я не понимаю зачем так усложнять, если уже есть класс HashMap<>.
```
Именно для понимания таких моментов и нужно писать этот проект.  
Пока ты не разберешься, по каким принципам нужно в объектно-ориентированном программировании делать классы, ты так и будешь писать процедурный код.
Если бы ты делал проект по ТЗ, возможно ты бы уловил логику, по которой Сергей декомпозировал этот проект на классы.

И в первом и во втором проекте ты сделал одну и ту же ошибку.  
Не научившись нормально писать в рамках ТЗ, усложнил проект сверх ТЗ и получилось еще хуже.

И то, если отбросить вайбкодинг, то в этом проекте не сделан даже базовый функционал- в консольной версии нет паузы/стоп.  
То есть ты не поработал с потоками.

Старайся делать проекты по ТЗ.  
Это тоже очень важный навык для работы- делать то, что от тебя требуется, а не то, что хочется по вдохновению.

Сравнить различия между ООП и процедурным стилями:  
Стрим Сергея [Крестики-нолики в процедурном стиле](https://www.youtube.com/watch?v=PPikj1qHxrA)  
Мой стрим [Крестики-нолики в ООП стиле](https://t.me/zhukovsd_it_chat/53243/187097)

Мои стримы про декомпозицию:  
[Крестики-Нолики](https://t.me/zhukovsd_it_chat/53243/187097)  
[Четыре в ряд](https://t.me/zhukovsd_it_chat/53243/218788)

Посмотреть на ютубе ролики Немчинского про SOLID.

n.151(322)  
#ревью #симуляция #вайбкодинг #ai