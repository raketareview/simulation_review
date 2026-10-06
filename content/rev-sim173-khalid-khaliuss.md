https://github.com/khaliuss/Simulation  
[Khalid]

Есть над чем поработать.  
Функционал не полностью соответствует ТЗ.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Команды не работают
```java
Запустить бесконечный цикл симуляции [1] Остановить симуляцию [2] сделать один ход [3]:
```
В программе не реализована работа с потоками и выполнение старт/стоп во время работы симуляции.  

## ХОРОШО

+ 👍 Есть механика голода существ 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов нужно писать стилем alllowercase 
```java
package org.example.static_entity;
package org.example.dynamic_entity;

//ПРАВИЛЬНО:
package org.example.staticentity;
package org.example.dynamicentity;
```

- Не дублируй имя класса в названии полей класса
```java
public class Coordinate {
  public int coordinateX;
  public int coordinateY;
  //...
}

//ПРАВИЛЬНО:
public class Coordinate {
  public int x;
  public int y;
  //...
}
```

- Названия методов должны быть глаголами в повелительном наклонении
```java
int carrotAmount()

//ПРАВИЛЬНО:
int calculateCarrotAmount()
```

- Тут тоже
```java
public class PathFinder {
  public List<Coordinate> targetCoordinate(Coordinate currentPosition, Entity foodSeeker) {...}
}

//ПРАВИЛЬНО:
public class PathFinder {
  public List<Coordinate> find(Coordinate currentPosition, Entity foodSeeker) {...}
}
```

- В названии переменных не используй предлог "To". 

Предлог "To" используется в методах конвертации: `toList()`, `toString()`, `toInt()`, `meterToFeet()`.   
В названии переменных не используй "To", потому что переменная хранит данные, а не конвертирует их.  

Здесь поле хранит класс цели для поиска пути
```java
private Class<? extends Entity> toHunt;

//ПРАВИЛЬНО:
private Class<? extends Entity> target;
```

- Просто интересно- что тебе дала здесь экономия одной буквы?
```java
public void creat()

//ПРАВИЛЬНО:
public void create()
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки  
```java
if (creature.isDead()) continue;

//ПРАВИЛЬНО:
if (creature.isDead()) {
  continue;
}
```
Исключение- метод equals(), там де-факто разрешается после `if` не выделять блоки скобочками. 

*"Oracle Java Code Conventions"*  

**3. class Constants**

- Соблюдай требования константных и утилитных классов. 

Константные классы, так же как и утилитные классы, должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.
  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

**4. class Coordinate**

+ 👍 Нет ничего лишнего, это хорошо. Класс только хранит значения x,y. 

- Стандартные названия осей в координате- x,y
```java
public class Coordinate {
  public int coordinateX;
  public int coordinateY;
  //...
}
//ПРАВИЛЬНО:
public class Coordinate {
  public int x;
  public int y;
  //...
}
```

Википедия:
```
"Положение точки A на плоскости определяется двумя координатами x и y"
```

- Если во время жизни объекта не планируешь изменять в нём значение определенных полей, то делай их неизменяемыми 
```java
public int coordinateX;
public int coordinateY;

//ПРАВИЛЬНО:
public final int x;
public final int y;
```
Такие простые классы, как координата, нужно делать immutable
```java
"Классы должны быть неизменяемыми, если только нет очень важной причины, чтобы сделать их изменяемыми",
"Вы всегда должны делать объекты с небольшими значениями, такими как PhoneNumber или Complex, неизменяемыми" - Блох
```
*Блох "Java. Эффективное программирование", изд.3, гл.4.3.*

- Класс может быть преобразован в `record` без потери функционала.

Для простых классов-контейнеров данных лучше всего использовать особый тип класса: `record`.  
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**5. class GameMap**

- Используй классы через их интерфейсы
```java
HashMap<Coordinate, Entity> entities = new HashMap();
HashMap<Coordinate, Entity> getEntities() {..}

//ПРАВИЛЬНО:
Map<Coordinate, Entity> entities = new HashMap();
Map<Coordinate, Entity> getEntities() {..}
```
Общее правило: ArrayList нужно использовать через List, HashMap через Map, HashSet через Set и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

- Карта должна знать свои размеры.

Для корректной работы класса карты, она должна знать свои собственные размеры
```java
public class GameMap {
  private HashMap<Coordinate, Entity> entities = new HashMap();
  
  //конструктор по умолчанию
}

//ПРАВИЛЬНО:
public class GameMap {
  private final Map<Coordinate, Entity> entities = new HashMap();
  private final int width;
  private final int height;
  
  public GameMap(int width, int height) {...}  //конструктор принимает размеры карты
}
```

- Нарушение инкапсуляции
```java
private HashMap<Coordinate, Entity> entities = new HashMap();

public HashMap<Coordinate, Entity> getEntities() {
  return entities;
}
```

Класс отдаёт свои внутренности наружу и клиент может делать с ними всё, что захочет.
Например:
```java
GameMap gameMap = FactoryGameMap.get(); //получаем из какой-то фабрики готовую карту с заселенными на неё существами
gameMap.getEntities().clear();  //минуя все методы класса карты, делаем геноцид существ- потому что хотим и можем
```

Фактически, сейчас класс является гибридом: совмещает в себе признаки объекта и структуры.  
Гибриды это плохо
```
*"ЧК", гл.6, "Крушение поезда", "Гибриды"*
```

В таких случаях нужно возвращать не оригинал объекта, а его копию.  
Тогда клиент, проводя манипуляции над полученным объектом, не сможет навредить внутреннему устройству карты:
```java
private Map<Coordinate, Entity> entities = new HashMap();

public Map<Coordinate, Entity> getEntities() {
  return new HashMap(entities);
}
```

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Подсчитать количество морковок на карте    
🔸 Вернуть список Creature-существ   

Наверное, для проекта в целом полезно иметь метод, который считает морковку.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно считать морковку.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

Конкретно здесь подсчёт морковки нужен не карте для организации хранения существ.  
Подсчёт морковок нужен общей игровой логике для (видимо?) воспроизводства морковок на карте.  
Значит эта "морковная логика" должна находиться где-то вне карты.  
Например в отдельном экшене, который отслеживает количество морковок на карте и восполняет их в случае необходимости.

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Creature> getCreatures()

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда креатуры

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getBirds()`.

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность.

Если координата некорректна (находится вне пределов карты), нужно бросать исключение.  
Потому что сейчас можно поместить существо на координату, которая будет выходить за пределы карты:
```java
public void putEntity(Coordinate coordinate, Entity entity) {  <-- МОЖНО ВСТАВИТЬ СУЩЕСТВО НА КООРДИНАТУ (+100500, -100500) ПРИ РАЗМЕРЕ КАРТЫ 10 x 10
  entities.put(coordinate, entity);
}

//ПРАВИЛЬНО:
public void putEntity(Coordinate coordinate, Entity entity) {  <-- МОЖНО ВСТАВИТЬ СУЩЕСТВО НА КООРДИНАТУ (+100500, -100500) ПРИ РАЗМЕРЕ КАРТЫ 10 x 10
  validate(coordinates);  <-- Если координата вне пределов карты, бросает исключение
  entities.put(coordinate, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Никогда не возвращай null
```java
private HashMap<Coordinate, Entity> entities = new HashMap();

public Entity getEntity(Coordinate coordinate) {
  return entities.get(coordinate);  <-- вернет null если в entities нет coordinate
}
```
Возврат null повышает риск возникновения NullPointerException в программе.

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Чё-та ад какой-то. Последствия нарушения морковного SRP
```java
public void deleteEntity(Coordinate coordinate) {
  if (carrotAmount() < 1) {
    new CarrotInit(this).create();  <-- создаёт и выполняет init-экшен
  }
  entities.remove(coordinate);
}

//ПРАВИЛЬНО:
public void deleteEntity(Coordinate coordinate) {
  validate(coordinates);  <-- Если координата вне пределов карты, бросает исключение

  if(isEmpty(coordinates)) {
    //пытались удалить пустую ячейку - бросить исключение
  }
  entities.remove(coordinate);
}
```

- Циклическая связь.

Карта использует экшен, а экшен используют карту
```java
new CarrotInit(this).create();
```

Класс карты не должен создавать и запускать экшены.  
Смысл экшенов по ТЗ ровно противоположный- это экшены должны работать с картой.  
Карта не должны ничего знать про экшены.

Вообще никто не должен знать про экшены, кроме класса `Simulation`.  
Разве что, чисто теоретически, про них может ещё знать какая-нибудь фабрика экшенов.

Циклические связи между классами в большинстве случаев означают,  

- Побочный эффект. 

Среди прочего, метод имеет побочный эффект.  
Его контракт обещает удалить существо. Но кроме этого он еще и создаёт существа- это побочный эффект
```java
public void deleteEntity(Coordinate coordinate) {
  //удаляет существо по координате
  //создаёт и выполняет init-экшен - заселяет морковку на карту
}
```
*Мартин "Чистый код", гл.3, "Избавьтесь от побочных эффектов"*  
*Фаулер "Рефакторинг", гл.6, "Извлечение метода"*  

**6. class PathFinder**

- Нарушение инкапсуляции.

Всегда явно указывай уровень доступа.   
Потому что неясно: то ли ты забыл его указать, то ли оно специально задумано как default.  
Здесь- забыл
```java
public class PathFinder {
  Queue<Coordinate> queue = new LinkedList<>();
  HashMap<Coordinate, Coordinate> visited = new HashMap<>();
  //...
}

//ПРАВИЛЬНО:
public class PathFinder {
  private Queue<Coordinate> queue = new LinkedList<>();
  private Map<Coordinate, Coordinate> visited = new HashMap<>();
  //...
}
```

- Нарушение конвенции кода.

Сначала должны быть перечислены поля, за ними- конструкторы, за конструкторами- остальные методы.  
Сейчас поля и методы перемешены.

- Главные публичные методы должны стоять выше вспомогательных приватных методов
```java
public PathFinder(GameMap gameMap) {...}

private void findAvailable(Coordinate cameFrom, Entity foodSeeker) {...}

public List<Coordinate> targetCoordinate(Coordinate currentPosition, Entity foodSeeker) {...}

//ПРАВИЛЬНО:
public PathFinder(GameMap gameMap) {...}

public List<Coordinate> targetCoordinate(Coordinate currentPosition, Entity foodSeeker) {...}

private void findAvailable(Coordinate cameFrom, Entity foodSeeker) {...}
```

- Нарушение SRP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно. 

Например, сейчас поиск хранит в себе меню для существ и решает за них, что они должны есть: волк - зайца, а заяц - морковку
```java
public List<Coordinate> targetCoordinate(Coordinate currentPosition, Entity foodSeeker) {
  //...
  if (foodSeeker instanceof Wolf) {
    toHunt = Rabbit.class;
  } else if (foodSeeker instanceof Rabbit) {
    toHunt = Carrot.class;
  }
  //...
}
```

Чтобы найти путь по алгоритму BFS, классу поиска достаточно знать координату начала пути, тип искомого существа и то, что путь ищется по свободным ячейкам карты.  
Для алгоритма BFS правильная сигнатура выглядит так:
```java
public class BfsPathFinder {

  public List<Coordinates> find(GameMap gameMap, Coordinate start, Class<? extends Entity> target) {
    //ищет путь на карте gameMap от точки start
    //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

- Пара x,y - это координата
```java
int[] dRow = new int[]{0, -1, -1, -1, 0, +1, +1, +1};
int[] dCol = new int[]{+1, +1, 0, -1, -1, -1, 0, +1};

private void findAvailable(Coordinate cameFrom, Entity foodSeeker) {
  for (int i = 0; i < 8; i++) {
    int moveRow = dRow[i] + cameFrom.coordinateX;
    int moveCol = dCol[i] + cameFrom.coordinateY;
    Coordinate coordinate = new Coordinate(moveRow, moveCol);
    //...
  }
}

//ПРАВИЛЬНО:
private static final List<Coordinate> SHIFT_COORDINATES = List.of(new Coordinate(0, 1), new Coordinate(-1, 1), ...); 

private void findAvailable(Coordinate cameFrom, Entity foodSeeker) {
  for (Coordinate shift : SHIFT_COORDINATES) {
    Coordinate coordinate = new Coordinate(cameFrom.coordinateX + shift.coordinateX, cameFrom.coordinateY + shift.coordinateY);
    //...
  }
}
```

- Антипаттерн "Стрела".

Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
while (!queue.isEmpty()) {
  if (toHunt.isInstance(currentEntity)) {
    while (true) {
      if (neighbor == null) {
        //наконечник стрелы
      }
    }
  }
}
```
"Стрела" значит, что метод делает несколько дел сразу и его нужно разделить на несколько вспомогательных.
```java
"Если вам нужно более трех уровней вложенности, вы все равно запутались, так что исправьте программу" - Линус Торвальдс
```

**7. Пакеты существ**

Разделение существ по трём равнозначным пакетам выглядит странно
```java
abstraction
  |
  +--- Coordinate.java
  +--- Entity.java
  +--- Creature.java
  +--- Predator.java

dynamic_entity
  |
  +-- Carrot.java   
  +-- Rabbit.java  
  +-- ...   

static_entity
  |
  +-- Rock.java   
  +-- ...   
```

Кстати, что сейчас морковка делает в пакете динамичных существ?  
Морковка же вроде не бегает.

Это всё entity, они должны лежать в одном пакете.  
Группировать абстрактные классы в отдельном пакете тоже не надо, нужно группировать классы по смыслу.    
Тем более, что в абстракции почему-то затесался неабстрактный Coordinate 
```java
entities
  |
  +--- immobile 
  |       |
  |       +-- Rock.java
  |       +-- Carrot.java
  |       +-- ...
  |
  +--- mobile
  |       |
  |       +-- Creature.java
  |       +-- Predator.java      
  |       +-- Rabbit.java  
  |       +-- ...    
  |
  +--- Entity.java
  +--- Coordinate.java
```

**8. abstract class Entity**

- Содержит координату
```java
protected Coordinate coordinate;
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

- Ок, содержит координату. Но как клиент её должен получать и устанавливать? 

Клиент должен находиться внутри пакета с Entity чтоле? Непонятно
```java
public abstract class Entity {
  protected Coordinate coordinate;

  public Entity(Coordinate coordinate) {
    this.coordinate = coordinate;
  }
}

//НУ ХОТЯ БЫ ТАК:
public abstract class Entity {
  protected Coordinate coordinate;

  public Entity(Coordinate coordinate) {
    this.coordinate = coordinate;
  }

  public Coordinate getCoordinate() {...}

  public void setCoordinate(Coordinate coordinate) {...}
}
```

**9. class Carrot extends Entity**

- Не используй статические импорты. 

При использовании статических импортов становится непонятно, что используемый в классе метод или константа не его собственные, а чьи-то чужие
```java
import static org.example.Constants.CARROT_EMOJI;

//...
public String toString() {
  return CARROT_EMOJI;
}

//ПРАВИЛЬНО:
public String toString() {
  return Constants.CARROT_EMOJI;
}
```
*"ЧК", гл.17, G18*

У тебя в проекте везде используются эти статические импорты, убери их во всех классах.

- Нарушение SRP, Неправильный toString()
```java
@Override
public String toString() {
  return CARROT_EMOJI;
}
```
Зависимость модели от представления- существо хранит спрайт с собственным изображением и возвращает его через toString().

Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах (консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  

**Спрайты всех существ должны храниться в классе, который распечатывает карту.**

toString() должен быть стандартным(как делает IDE по Alt+Ins) и использоваться только для отладки.  
toString() должен содержать значение всех значимых полей класса.  
toString() не должен использоваться для представления, за исключением предельно простых классов, вроде PhoneNumber
*Блох, "Java. Эффективное программирование", изд.3, гл.3.3*

**10. abstract class Creature extends Entity**

Эти поля у всех потомков будут иметь те же начальные значения 1 и 100?
```java
public abstract class Creature extends Entity {
  protected final int speed = 1;
  protected int hp = 100;
  //...
}
```

Если да, то правильно так:
```java
public abstract class Creature extends Entity {
  protected static final int SPEED = 1;
  protected int hp = 100;
  //...
}
```

Если нет и у потомков будут разные значения скорости и жизни, то правильно так:
```java
public abstract class Creature extends Entity {
  protected final int speed;
  protected int hp;
  //...
  
  public Creature(int hp, int speed, ...) {
    this.hp = hp;
    //...
  }
}
```

- Циклическая зависимость.

Карта хранит креатур, а креатуры хранят карту
```java
public abstract class Creature extends Entity {
  protected int hp;    
  protected GameMap gameMap;
  //...
}
```
Если Заяц хранит в себе жизни и Карту, значит Заяц состоит из жизни и Карты.  
Является ли жизнь частью зайца? Да.  

Является ли карта частью зайца?  
Разве что в смысле буддийской философии, в рамках которой и карта и симуляция это лишь сон зайца.

Короче, с точки зрения отношений между классами, нельзя в креатуре хранить карту.

А получать карту в методы можно. Например:
```java
public abstract class Creature extends Entity {
  protected GameMap gameMap;
  //...

  public Creature(Coordinate coordinate, GameMap gameMap, PathFinder pathFinder) {...}

  public abstract void makeMove();
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //...

  public Creature(Coordinate coordinate, PathFinder pathFinder) {...}

  public abstract void makeMove(GameMap gameMap);
}
```

- Лучше всего в существах вообще не хранить координат.

Координату можно хранить в существах начиная с уровня Creature.  
Но ещё лучше- вообще не хранить координату в существах.

Хранение координат внутри существ приводит к дублированию хранения координат- они хранятся и в классе карты и в классе существ.  
Эти координаты нужно синхронизировать между собой, из-за чего код становится более громоздким и сложным.

Свою координату существо может получать прямо из карты. Например:
```java
private final Class<? extends Entity> food;

public void makeMove(GameMap gameMap) {
  Coordinate current = gameMap.getCoordinate(this);
  List<Coordinate> path = pathFinder.find(gameMap, current, food);
  // Пройти по пути path и съесть еду
}
```

Аналогия:  
Человек не хранит свою координату внутри себя, он её считывает глазами из окружающего пространства.  
С завязанными глазами, не считывая свою координату извне, трудно дойти даже до пивного ларька.

**11. abstract class Herbivore extends Creature**

- Этот метод используется только у потомков `Herbivore`
```java
public abstract void eat();

//ПРАВИЛЬНО:
protected abstract void eat();
```
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

**12. class Wolf extends Predator**

- Перекрытие поля предка.

То, о чём я писал в пункте Creature
```java
public class Wolf extends Predator {
  protected final int speed = 1;

  public Wolf(...) {
    super(...);
  }
}

//ПРАВИЛЬНО:
public class Wolf extends Predator {
  private static final int SPEED = 1;

  public Wolf(...) {
    super(SPEED, ...);
  }
}
```

- Общий код в методе `makeMove(...)` у классов `Wolf` и `Rabbit`.

ТЗ:
```java
Чеклист для самопроверки #

Проблемы и ошибки в коде:
Дублирование кода между классами Herbivore и Predator
```
Общий код выноси в предка, в данном случае- в класс `Creature`.

- Магические числа: 3, 100, 2.

**13. Иерархия классов entity**

Ты переусложнил иерархию классов Entity и, кажется, запутался в ней.
Упрости иерархию, убери всё лишнее, например, промежуточные пустые классы типа `Obstacle`.  

Примерно так, UML:
```java
Entity
  △
  |
  +--- Creature
  |       △
  |       |
  |       +--- Wolf
  |       +--- Rabbit
  |
  +--- Rock  
  +--- Tree  
  +--- Carrot  
```
Тогда будет проще разобраться, на каких уровнях иерархии нужно вводить поля и методы.

Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**14. Action**

Не должно быть промежуточного уровня иерархии InitAction и TurnAction
```java
public class Action {
}

public abstract class TurnAction extends Action {
  //...
  public abstract void makeMove();

}

public abstract class InitAction extends Action {
  //...
  public abstract void create();
}
```

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.  
В каждом экшене должен быть только один публичный метод(не публичных может быть сколько угодно).  
Это вариация паттерна Command- экшены должны быть родственны и одинаково использоваться через полиморфизм.  
Примерно так
```java
interface Action{
 void execute(Карта карта);
}

class ХодитьAction реализует Action {
  void execute(Карта карта) {
    //обойти всю карту
    //найти каждую креатуру
    //и дать ей пинка чтоб побежала
  }
}

class ДатьСигаретуВсемЗайцамAction реализует Action {
  void execute(Карта карта) {
    //обойти всю карту
    //найти всех зайцев и дать им по сигарете 
  }
}

List<Action> actions = List.of(new ДатьСигаретуВсемЗайцамAction(), new ХодитьAction());
for(Action a: actions) {
  a.execute(карта);
}

/*
Результат: программа обойдет всю карту, найдет всех креатур и даст им пинка, чтобы они побежали.
А каждому зайцу предварительно даст сигарету.
*/
```

Все экшены должны одинаково использоваться через полиморфизм:
```java
public class Simulation {
  private final List<InitAction> initActions = List.of(...);
  private final List<TurnAction> turnActions = List.of(...);
  //...
}

//ПРАВИЛЬНО:
public class Simulation {
  private final List<Action> initActions = List.of(...);
  private final List<Action> turnActions = List.of(...);
  //...
}
```

**15. Семейство экшенов - спавнеров**

Есть целое семейство одинаковых классов-спавнеров, код в которых фактически дублируется
```java
class RabbitInit extends InitAction
class RockInit extends InitAction
etc
```

Они делают одно то же- заселяют карту определенным видом существ. 

ТЗ:
```java
Чеклист для самопроверки #

Проблемы и ошибки в коде:

Иерархия классов Action’s
Дублирование кода
```

Вместо отдельного класса на каждый вид существ, нужно сделать универсальный класс. 
Он будет принимать в конструктор количество создаваемых существ и способ их создания.  

Способ создания можно передавать через стандартный интерфейс Supplier
```java
public class SpawnAction extends Action {
  private final Function<Coordinate, Entity> entityCreator;
  private final int count;

  public SpawnAction(Function<Coordinate, Entity> entityCreator, int count) {...}

  @Override
  public void execute(Карта карта) {
    for (int i = 0; i < count; i++) {
      Coordinate coordinate = getRandomFreeCoordinate(карта);
      Entity entity = entityCreator.apply(coordinate);

      карта.put(entity, coordinate);
    }
  }

  private Coordinate getRandomFreeCoordinate(карта) {
    //найти случайную пустую координату в карте
  }
}
```

Использование:
```java
SpawnAction tigerSpawnAction = new SpawnAction((coordinate) -> new Tiger(coordinate), TIGER_COUNT);
SpawnAction birdSpawnAction = new SpawnAction((coordinate) -> new Bird(coordinate), BIRD_COUNT);
tigerSpawnAction.execute(карта); 
birdSpawnAction.execute(карта); 
```

Если существа не хранят координаты, то:
```java
public class SpawnAction extends Action {
  private final Supplier<Entity> entityCreator;
  //...

  @Override
  public void execute(Карта карта) {
    //...
    Entity entity = entityCreator.get();
  }

  //...
}

//Использование:
SpawnAction tigerSpawnAction = new SpawnAction(() -> new Tiger(), TIGER_COUNT);
```

Тема: стандартные функциональные интерфейсы Java.

**16. InitAction's неправильно работают с классом констант**

InitAction's напрямую читают данные из класса констант, а должны *только* получать в свой конструктор необходимые данные из него.  
Это делает классы неуниверсальными и зависимыми от конфигурационного класса.  

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

Это касается и других классов, которые напрямую лезут в константы, а не получают необходимые данные в конструктор.  
Читать из констант здесь напрямую может только класс `Simulation` и, возможно, `Main`:
```java
public class Simulation {

  private final List<InitAction> initActions = List.of(
      new RockInit(gameMap),
      new TreeInit(gameMap),
      //...
  );
}

//ПРАВИЛЬНО:
public class Simulation {

  private final List<InitAction> initActions;
  //...
  
  public class Simulation(GameMap gameMap) {
    this.gameMap = gameMap;

    initActions = List.of(
        new RockInit(gameMap, Constants.ROCK_COUNT),
        new TreeInit(gameMap, Constants.TREE_COUNT),
        //...
    );
    //...
  }
}
```

Но в целом, здесь от класса `Constants` должен зависеть только класс `Simulation`.

**17. class CreatureTurn extends TurnAction**

- Нарушение SRP.

Экшен хода должен только ходить креатурами.  
Он не имеет права принимать решение о прекращении работы программы в целом
```java
if (coordinates.isEmpty()) {
  System.exit(0);
}
```

Правила прекращения работы симуляции должны находиться в классе Simulation.  
И даже там нельзя выходить через `exit()`, можно только через `return`.

Каждый метод и класс имеют право завершать только свою работу. Например, через return.  
Потому что методы и классы не должны знать логику работы более высоких слоев программы, у которых могут быть свои планы на тему того, когда и почему нужно завершать работу программы.

Кроме того, при выходе через `exit()` могут не закрыться некоторые ресурсы программы.

**18. Восполнение существ на карте**

Если во время работы программы нужно отслеживать количество существ и восполнять их на карте, то для этого должен использоваться отдельный экшен.  
Например:
```java
public class ReSpawnAction extends Action {
  //...

  public SpawnAction(Function<Coordinate, Entity> entityCreator, int min, int max) {...}

  @Override
  public void execute(Карта карта) {
    //если количество существ на карте достигло min
    //то увеличить их количество на карте до max
  }
}
```
Использоваться этот экшен должен так же, как и все остальные:
```java
turnActions = List.of(
  new MoveAction(),
  new ReSpawnAction((c) -> new Carrot(c), Constants.CARROT_MIN_COUNT, Constants.CARROT_COUNT),
);
```

**19. class Simulation**

- Нарушение DI. 

Класс не должен сам себя конструировать.  
В данном случае класс конструирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
Таким образом нельзя без изменения кода в классе `Simulation` создать несколько игровых конфигураций с разными картами.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Никогда нельзя выходить через exit
```java
public void pauseSimulation() {
  System.exit(0);
}
```

- Не бросай базовый exception. Конкретизируй ситуацию- бросай то исключение, которое подходит под этот конкретный случай
```java
throw new RuntimeException(e);
```
*Хорстманн "Java. Библиотека профессионала", т.1, гл.11*
```java
"Не ограничивайтесь генерацией RuntimeException. 
Найдите подходящий подкласс или создайте собственный." - Хорстманн
```

**20. class Main**

- Нарушение SRP.

Main должен только сконфигурировать зависимости и запустить программу.  
Управлять работой программы этот класс не должен.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

Определи сущности, которые сейчас сидят в этом классе и вынеси их в отдельные классы.  
Например, сущность, которая ДОЛЖНА через использование потоков управлять симуляцией, получая от юзера команды пауза/пуск.

Примерно так:
```java
public class MainWithThreads {

  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    Simulation simulation = new Simulation(gameMap);

    SimulationManager manager = new SimulationManager(simulation);  //работает с потоками, дергает симуляцию за методы пауза/пуск 

    manager.start(); //симуляция с потоками и командами пауза/пуск
  }
}
```

- Нарушение DRY.

Магические буквы, числа, слова. Вводи константы 
```java
case "1" -> simulation.startSimulation();
case "2" -> simulation.pauseSimulation();
case "3" -> simulation.nextTurn();

System.out.print("Запустить бесконечный цикл симуляции [1] Остановить симуляцию [2] сделать один ход [3]: ");

//ПРАВИЛЬНО:
private static final int START = "1";
private static final int PAUSE = "2";
private static final int ONE_TURN = "3";

case START -> simulation.startSimulation();
case PAUSE -> simulation.pauseSimulation();
case ONE_TURN -> simulation.nextTurn();

System.out.printf("Запустить бесконечный цикл симуляции [%s] Остановить симуляцию [%s] сделать один ход [%s]: ", START, PAUSE, ONE_TURN);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru, "Замена магического числа символьной константой"*

## ВЫВОД

Программа сделана частично- не реализована возможность сделать паузу во время работы симуляции.  
Для второго проекта важно поработать с потоками. Для этого нужно придумать, как делать паузу/пуск во время работы симуляции.

Не переусложняй проекты, придерживайся ТЗ и правила KISS.  
Здесь ты переусложнил иерархию entities; добавил константный класс, который используешь неправильно- сделал через него высокую связность классов.

Есть ошибки иерархии классов: 
Поля и методы появляются не на тех уровнях, где должны; дублирование кода в потомках; перекрытие полей предка.

Переделай Action's по моим рекомендациям.

Посмотри на ютубе ролики Немчинского для новичков:
```
"Правильные методы по Clean Code"
"Как называть переменные, методы и классы? Чистый код (Clean Code)"
"Принцип хорошего кода KISS"
"Принцип хорошего кода DRY"
```

И отдельно ролики про SOLID:
```
"SOLID принципы: SRP"
"SOLID принципы: OCP"
etc 
```

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.173(374)  
#ревью #симуляция 