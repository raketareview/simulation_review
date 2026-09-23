https://github.com/KruachenkaDB/ProjectTwoSimulation  
[Круаченя]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Нет возможности сделать один ход.

## ХОРОШО

+ 👍 Красивый Windows UI Swing  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim170/img0.png) 
+ 👍 Есть пауза/пуск во время работы
+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Механика голода у существ

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Не называй объекты словом "that" это созвучно с "this" и ты их будешь путать по запарке 
```java
Coordinates that = (Coordinates) object;

//ПРАВИЛЬНО:
Coordinates other = (Coordinates) object;
```

- IDE подчеркивает ошибки в орфографии, обращай на это внимание
```java
Map<Coordinates, Entity> entitys

//ПРАВИЛЬНО:
Map<Coordinates, Entity> entities
```

- Название метода должно объяснять, что делает метод.  
Этот метод не просто возвращает случайную координату. Он возвращает случайную пустую координату
```java
Coordinates getRandomCoordinates()

//ПРАВИЛЬНО:
Coordinates getRandomFreeCoordinates()
```

- Это не сеттер.

Сеттер должен устанавливать значение какого-то конкретного поля.  
А этот метод не устанавливает конкретное существ, он добавляет существо в группу других существ:
```java
private final java.util.Map<Coordinates, Entity> entitys = new HashMap<>();

public void setEntitys(Coordinates coordinates, Entity entity) {
  //добавляет entity в entitys    
}

//ПРАВИЛЬНО:
void putEntity(Coordinates coordinates, Entity entity) //или addEntity(...)
```

Когда есть какое-то ОДНО существо, то сеттер корректен. Условный пример:
```java
private Entity entity;

void setEntity(Entity entity) {
  this.entity = entity;  <-- Правильно, сеттер устанавливает конкретное значение
}
```

Когда существо добавляется к другим, то это не сеттер. Условный пример:
```java
private List<Entity> entities;

void addEntity(Entity entity) {
  entities.add(entity);  <-- Это не сеттер, потому что добавляет новый объект в список других
}
```

- Избыточно
```java
public class PathFinder {
  public static LinkedList<Coordinates> findPathBfs(...) {...}
}

//ПРАВИЛЬНО:
public class BfsPathFinder {
  public static LinkedList<Coordinates> find(...) {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки  
```java
if ((v < 0) || (v >= GameSettings.MAP_HEIGHT)) return false;
if ((h < 0) || (h >= GameSettings.MAP_WIDTH)) return false;

//ПРАВИЛЬНО:
if ((v < 0) || (v >= GameSettings.MAP_HEIGHT)) {
  return false;
}
if ((h < 0) || (h >= GameSettings.MAP_WIDTH)) {
  return false;
}
```
Исключение- метод equals(), там де-факто разрешается после `if` не выделять блоки скобочками.

*"Oracle Java Code Conventions"*  

**3. Используй классы через их интерфейсы**
```java
LinkedList<Coordinates> findPathBfs(...)

//ПРАВИЛЬНО:
List<Coordinates> findPathBfs(...)
```
Общее правило: ArrayList нужно использовать через List, HashMap через Map, HashSet через Set и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**4. Exceptions**

- Текст в Exception всегда должен быть на английском языке.
```java
throw new IllegalStateException("На поле недостаточно места для всех объектов");  <-- Неправильно: сообщение в исключении не на английском языке
```
Исключение это не просто телеграмма, которая летит сквозь слои.

У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.  
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты.  
А значит, сообщение должно быть на английском.

Интерпретация исключения и перевод его на локальный язык должны происходить там, где это соответствует архитектуре программы.  
Или не происходить вовсе, если исключение не планируется перехватывать.

**5. class GameSettings**

- Соблюдай требования константных и утилитных классов. 

Константные классы, так же как и утилитные, классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.
  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- Константный класс в программе используется неправильно.

Другие классы напрямую лезут в константный класс.  
А должны только получать необходимые данные из константного класса в свой конструктор.

Это делает классы неуниверсальными и зависимыми от константного класса.  

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

**6. class CoordinatesShift**

Координата для сдвига.

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс `Coordinates`.  
Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```java
Coordinates shiftDownRightCoordinates = new Coordinates(-1, 1);
```

**7. class Coordinates**

- Стандартные названия осей в координате- x,y
```java
public class Coordinates {
  public final Integer vertical;
  public final Integer horizontal;
  //...
}

//ПРАВИЛЬНО:
public class Coordinates {
  public final int x;
  public final int y;
  //...
}
```

Википедия:
```
"Положение точки A на плоскости определяется двумя координатами x и y" 
```
Вместо имён "x,y" можно использовать "row/column".

- При прочих равных, нужно использовать примитивный тип, а не класс-обертку
```java
public class Coordinates {
  public final Integer vertical;
  public final Integer horizontal;
  //...
}

//ПРАВИЛЬНО:
public class Coordinates {
  public final int vertical;
  public final int horizontal;
  //...
}
Для применения класса-обертки над примитивным типом данных, должна быть какая-то причина.  
Здесь использование обертки не дает ничего большего по сравнению с примитивом.

*Блох "Java. Эффективное программирование", изд.3, гл.9.5*
```java
"Предпочитайте примитивные типы упакованным примитивным типам" - Блох.
```

- Доступ к полям объектов делай через методы
```java
public class Coordinates {
  public final Integer vertical;
  public final Integer horizontal;
  //...
}

//ПРАВИЛЬНО:
public class Coordinates {
  private final int vertical;
  private final int horizontal;
  //...

  public getVertical() {...}
  public getHorizontal() {...}
}
```
Доступ к полям объектов всегда должен быть через методы. Например, геттеры и сеттеры.  
Доступ к полям структур можно делать прямым- через доступ public.

Структура отличается от объекта тем, что хранит только данные и не хранит поведение.  
Этот класс хранит много сложного поведения, поэтому не является структурой.

*"ЧК", гл.6, "Объекты и структуры данных"*

- Нарушение SRP, Low Coupling
```java
public boolean canShift(CoordinatesShift shift) {
  int v = vertical + shift.verticalShift;
  int h = horizontal + shift.horizontalShift;

  if ((v < 0) || (v >= GameSettings.MAP_HEIGHT)) return false;
  if ((h < 0) || (h >= GameSettings.MAP_WIDTH)) return false;

  return true;
}
```
Нарушение LC состоит в том, что класс координаты знает о существовании класса `GameSettings`, про который он знать не должен.  

Нарушение SRP состоит в том, что определение того, можно ли координату сдвинуть так, чтобы она не выходила за пределы *какой-то* карты, не является ответственностью координатного класса.  
Единая ответственность координатного класса состоит в хранении положения точки в пространстве.  

Для того, чтобы хранить положение точки в пространстве, классу не нужно знать не только про существование карты или константного класса, где хранится информация про карту.  
Ему не нужно знать даже про то, как именно будут использоваться координаты в пространстве: будут ли они храниться в карте, списке или использоваться сами по себе. 

- Метод сдвига может быть в этом классе, он не нарушает SRP.  
Но сдвигаться он должен с объектом своего класса
```java
public Coordinates shift(CoordinatesShift shift) {
  return new Coordinates(
    this.vertical + shift.verticalShift,
    this.horizontal + shift.horizontalShift
  );
}

//ПРАВИЛЬНО:
public Coordinates shift(Coordinates coordinates) {
  return new Coordinates(this.x + coordinates.x, this.y + coordinates.y); 
  );
}
```

- Для координаты лучшим решением будет использовать `record`.

Это подойдет для симуляции и любой другой игры на прямоугольной доске(Морской бой, Шахматы и т.д.).  
Вот идеальная координата:
```java
public record Coordinates (int x, int y) {

  public Coordinates shift(Coordinates coordinates) {...}
}
```
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**8. class Map**

- Не называй свои классы так же, как называются стандартные классы и интерфейсы Java Core.  

Класс не должен называться "Map" - это название стандартного интерфейса Java.

Следовать ТЗ это хорошо. 
Называть свои пользовательские классы так же, как стандартные классы и интерфейсы- плохо.  
Второе перевешивает, потому что это приводит к постоянной путанице в коде.

Например, такой код будет всё время сбивать с толку, потому что придется всё время на нем останавливаться и думать, 
какой именно из Map'ов имеется в виду- стандартный или кастомный
```java
Map map = new Map(30, 15);
```

Чтобы компилятор понимал, про какой именно `Map` идет речь, приходится писать полный путь к стандартному интерфейсу Map через прописывание полного пути к этому интерфейсу
```java
public class Map {
  private final java.util.Map<Coordinates, Entity> entitys = new HashMap<>();  <-- Полный путь к интерфейсу "Map" чтобы отличать от кастомного класса "Map"
  //...
}
```

Для наименования классов придерживайся терминологии предметной области и среды разработки.  
Если они вступают в противоречие, жертвуй терминологией предметной области. В данном случае, предметная область- это ТЗ.

Конкретно в этом случае вместо "Map", назови свой кастомный класс как-либо иначе: "Board", "GameMap" etc. 

- Нарушение SRP, карта не знает своих размеров.

Карта должна свои размеры получать в конструктор и хранить внутри себя.  
Свои собственные размеры по ширине и высоте карта не должна лично считывать из другого класса, например константного. 

Даже если в проекте есть класс с константами(GameSettings), то третий класс должен прочитать эти константы из константного класса и передать в конструктор Карты.  
Например, так:
```java
GameMap gameMap = new GameMap(Settings.GAME_MAP_X, Settings.GAME_MAP_Y);
```

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Найти случайную пустую координату    
🔸 Определить, что в константном классе указано корректное количество существ для начального заполнения карты    
🔸 Сделать ход существом  
etc.

Наверное, для проекта в целом полезно иметь метод, который находит случайную пустую координату.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно искать случайные пустые координаты.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

В данном случае нужно определить тот класс, для ответственности которого нужно найти пустую координату в карте и перенести в него этот метод.  
Найти этот класс очень просто- нужно посмотреть, какой класс использует метод `getRandomCoordinates()`. Это класс `SpawnAction`.

- Метод совершения хода в карте- нарушение SRP.

Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
Карта карта = new Карта(100, 100);
Заяц заяц = new Заяц();
карта.setEntity(new Координата(0, 0), заяц);
карта.moveEntity(new Координата(0, 0), new Координата (99, 99));

/* class Карта */
public void moveEntity(Coordinates from, Coordinates to) {
  Entity entity = getEntity(from);
  removeEntity(from);
  setEntitys(to, entity);
}
```
Должна ли карта учитывать правила логистики зайцев? Если да, то как?

Например, имеет ли право заяц телепортироваться на координату, которая уже занята?  
Может ли телепортироваться камень?  
Может ли телепортироваться пустое место на голову волка?

Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Никогда не возвращай null
```java
private final java.util.Map<Coordinates, Entity> entitys = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return entitys.get(coordinates);  <-- вернет null если в entitys нет coordinates
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность.

Сейчас в карту можно вставить существо на координату, выходящую за размер карты.  
Если координата некорректна (находится вне пределов карты), нужно бросать исключение:
```java
public void setEntitys(Coordinates coordinates, Entity entity) {
  entity.setCoordinates(coordinates);
  entitys.put(coordinates, entity);
}

//ПРАВИЛЬНО:
public void setEntitys(Coordinates coordinates, Entity entity) {
  validate(coordinates);  <-- Если координата вне пределов карты, бросает исключение
  entity.setCoordinates(coordinates);
  entitys.put(coordinates, entity);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, то бросается исключение.

- Нарушение DRY.

Общий код выноси во вспомогательные методы
```java
public List<Predator> getPredators() {
  List<Predator> predators = new ArrayList<>();

  for (Entity entity : entitys.values()) {
    if (entity instanceof Predator) {
      predators.add((Predator) entity);
    }
  }
  return predators;
}

public List<Creature> getCreatures() {
  List<Creature> creatures = new ArrayList<>();

  for (Entity entity : entitys.values()) {
    if (entity instanceof Creature) {
      creatures.add((Creature) entity);
    }
  }
  return creatures;
}

//ПРАВИЛЬНО:
public List<Predator> getPredators() {
  return getEntitiesBy(Predator.class);
}

public List<Creature> getCreatures() {
  return getEntitiesBy(Creature.class);
}

public <T extends Entity> List<T> getEntitiesBy(Class<T> type) {...}
```

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Herbivore> getHerbivores()

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда существ нужного класса

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getBirds()`.

**9. class PathFinder**

- Соблюдай требования утилитных классов: `final` и закрытый конструктор.

- Нарушение SRP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно
```java
//Самостоятельно находит точку старта- нарушение SRP 
public static LinkedList<Coordinates> findPathBfs(Entity entity, Map map, TargetType targetType) {...}

//ПРАВИЛЬНО:
public static List<Coordinates> find(GameMap gameMap, Coordinates start, Class<? extends Entity> target) {...}
```

- Всегда явно указывай уровень доступа.  

Потому что неясно- то ли ты забыла его указать(а однажды забудешь), то ли оно специально задумано как default.  
В этом случае модификатор доступа неправильный- вместо `package-private` (default) должен быть `private`
```java
static boolean isTarget(Entity entity, Entity target, TargetType targetType) {

//ПРАВИЛЬНО:
private static boolean isTarget(Entity entity, Entity target, TargetType targetType) {
```
Этот метод является частью внутренней архитектуры класса и предназначен для использования только внутри своего класса.

- Антипаттерн "Стрела".

Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
while (!queue.isEmpty()) {
  for (Coordinates newCoordinates : neighbours) {
    if (isTarget(entity, target, targetType)) {
      while (!step.equals(entity.getCoordinates())) {
        //наконечник стрелы
      }
    }
  }
}
```
"Стрела" значит, что метод делает несколько дел сразу и его нужно разделить на несколько вспомогательных.
```java
"Если вам нужно более трех уровней вложенности, вы все равно запутались, так что исправьте программу" - Линус Торвальдс.
```

- Нарушение SRP, OCP
```java
if (targetType == TargetType.FOOD) {
  return ((entity instanceof Predator) && (target instanceof Herbivore)) || ((entity instanceof Herbivore) && (target instanceof Grass));
}
```

Нарушение SRP:   
Класс поиска зачем-то знает правила определения еды у существ.  
Хотя должен просто искать путь от точки старта до точки, где находится существо нужного класса.

Нарушение OCP:   
При добавлении новых типов существ в проект, придется вносить изменение в этот класс  
Например: 
```java
if (targetType == TargetType.FOOD) {
  return ((entity instanceof Predator) && (target instanceof Herbivore)) || ... || ((entity instanceof Bird) && (target instanceof Mosquito)));
}
```
То есть, класс открыт для изменений.

- Не используй аргументы-флаги.

Здесь `targetType`- управляющий флаг. В зависимости от его значения, алгоритм поиска работает тем или иным способом
```java
public class PathFinder {

  public static LinkedList<Coordinates> findPathBfs(Entity entity, Map map, TargetType targetType) {
    if (isTarget(entity, target, targetType)) {..}
    //...
  }

  static boolean isTarget(Entity entity, Entity target, TargetType targetType) {
    //...
    if (targetType == TargetType.FOOD) {  <-- ПЕРЕКЛЮЧЕНИЕ АЛГОРИТМА УПРАВЛЯЮЩИМ ФЛАГОМ
      return ((entity instanceof Predator) && (target instanceof Herbivore)) || ((entity instanceof Herbivore) && (target instanceof Grass));  <-- ПЕРВЫЙ АЛГОРИТМ ОПРЕДЕЛЕНИЯ ЦЕЛИ
    }
    //   else if (targetType == map.TargetType.PARTNER) {}
    //   тут должна была быть логика поиска партнера
    return false;  <-- ВТОРОЙ АЛГОРИТМ ОПРЕДЕЛЕНИЯ ЦЕЛИ
  }
}
```
*Мартин "ЧК", гл.3, "Аргументы-флаги"*
```
"Аргументы-флаги уродливы... функция выполняет более одной операции" - Мартин.
```

Если рассматривать ситуацию как сферического коня в вакууме, то в подобных случаях вместо управляющего флага нужно использовать полиморфизм:
```java
public abstract class PathFinder {

  public List<Coordinates> find(...) {
    if (isTarget(...)) {..}
    //...
  }
   
   protected abstract boolean isTarget(...);
}

public class FoodPathFinder extends PathFinder{
   
   @Override
   protected boolean isTarget(...) {
    //логика определения еды 
   }
}

public class PartnerPathFinder extends PathFinder{
   
   @Override
   protected boolean isTarget(...) {
    //логика определения ПАРТНЁРА 
   }
}
```
*Фаулер "Рефакторинг", гл.12, "Выделение иерархии (Extract Hierarchy)"*

Но в данном случае, нужно просто поменять саму концепцию того, как происходит поиск.

- Если когда-нибудь придётся искать партнера, то сути поиска это не меняет
```java
//        else if (targetType == map.TargetType.PARTNER) {}
//            тут должна была быть логика поиска партнера
```

Нужно использовать всё тот же поиск, но передавать в него для поиска класс цели не как "цель-еда", а как "цель-партнёр"
```java
public abstract class Creature extends Entity {
  //...

  public void makeMove(Map map) {
    LinkedList<Coordinates> path = PathFinder.findPathBfs(this, map, TargetType.FOOD);
    //...
  }
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  private final Class<? extends Entity> food;  
  //...

  public Creature(..., Class<? extends Entity> food) {
    //...
    this.food = food;
  }

  public void makeMove(GameMap gameMap) {
    Coordinates start = gameMap.getCoordinates(this);

    List<Coordinates> foodPath = BfsPathFinder.find(gameMap, start, food);
    List<Coordinates> partnerPath = BfsPathFinder.find(gameMap, start, this.getClass()); //партнёр - существо своего класса
    //...
  }
}

public class BfsPathFinder {

  public static List<Coordinates> find(GameMap gameMap, Coordinates start, Class<? extends Entity> target) {...}
}
```

Правда, если искать вот так, то найдёшь сам себя (свой же экземпляр)
```java
List<Coordinates> partnerPath = BfsPathFinder.find(gameMap, start, this.getClass());
```
Но тут думай, как изменить алгоритм BFS поиска, чтобы в такой ситуации не ловить самого себя за хвост.

**10. abstract class Entity**

- Содержит координату
```java
public abstract class Entity {
  private Coords coords;
  //...
}
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

- Содержит направления движения
```java
public Set<CoordinatesShift> getEntityMoves() {
  return new HashSet<>(Arrays.asList(
    new CoordinatesShift(1, 0),
    new CoordinatesShift(-1, 0),
    //...
  ));
}
```
Но не все потомки Entity умеют ходить.  
Камень, трава и дерево ходить не умеют и потому не должны получать в наследство от предка ненужную им информацию.

В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

+ 👍 Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**11. class Tree extends Entity**

- Антипаттерн "Отказ от наследства".

Последствие неправильного расположения методов на уровне родительского класса Entity.  
Потомок переопределил метод предка и сделал его пустым
```java
public class Tree extends Entity {
  //...

  @Override
  public Set<CoordinatesShift> getEntityMoves() {
    return Set.of();
  }
}
```
*Фаулер "Рефакторинг", гл.3, "Отказ от наследства"*

**12. abstract class Creature extends Entity**

- Нарушение Low Coupling.

Класс напрямую лезет в общие константы проекта
```java
public void restoreHunger() {
  hunger = GameSettings.MAX_HUNGER;
}
```

**13. Параллельные иерархии Action**

Ты неправильно поняла ТЗ относительно Actions.  
Нужно одно семейство классов Action, не два параллельных
```java
public interface TurnAction {
  void execute(Map map, int turnCounter);
}

public interface InitAction {
  void execute(Map map);
}

public class SpawnGrassTurnAction implements TurnAction {...}
public class InitEntitiesAction implements InitAction {...}

//ПРАВИЛЬНО:
public interface Action {
  void execute(GameMap gameMap);
}

public class SpawnGrassTurnAction implements Action {...}
public class InitEntitiesAction implements Action {...}
```

**14. class InitEntitiesAction implements InitAction**

- Нарушение DRY.  

- Нарушение DRY.  

Повторяющиеся действия выноси во вспомогательные методы.  
"Действия" можно передавать из метод в метод через использование стандартных функциональных интерфейсов Java
```java
for (int i = 0; i < GameSettings.INITIAL_PREDATOR_COUNT; i++) {
  new PredatorSpawnAction().execute(map);
}

for (int i = 0; i < GameSettings.INITIAL_HERBIVORE_COUNT; i++) {
  new HerbivoreSpawnAction().execute(map);
}
//...

//ПРАВИЛЬНО:
spawn( () -> new PredatorSpawnAction().execute(map), predatorCount);
spawn( () -> new HerbivoreSpawnAction().execute(map), herbivoreCount);
//...

private void spawn(Runnable entitySpawner, int count) {
  for (int i = 0; i < count; i++) {
    entitySpawner.run();
  }
} 
```
  
Гугли список стандартных функциональных интерфейсов Java, чтобы знать, какие они есть.  

Например, аналогичный функционал через другой интерфейс:
```java
spawn( (coordinates) -> new Predator(coordinates, 3, 1, ...), predatorCount);
spawn( (coordinates) -> new Herbivore(coordinates, 1, 1, ...), herbivoreCount);
spawn( (coordinates) -> new Tree(coordinates), treeCount);
//...

private void spawn(GameMap gameMap, Function<Coordinates, Entity> entityCreator, int count) {
  for (int i = 0; i < count; i++) {
    Coordinates coordinates = getRandomFreeCoordinates(gameMap);
    Entity entity = entityCreator.apply(coordinates);

    gameMap.setEntitys(coordinates, entity);
  }
} 

private Coordinates getRandomFreeCoordinates(GameMap gameMap) {
  //находит на карте и возвращает случайную пустую координату
}
```

Не обязательно "передавать действия" в методы именно стандартными интерфейсами, можно использовать любые.  
Но стандартные интерфейсы подходят в большинстве случаев.

*Блох "Java. Эффективное программирование", изд.3, 7.3. "Предпочитайте использовать стандартные функциональные интерфейсы"*

**15. MoveCreaturesAction implements TurnAction**

- Зачем в аргументы метода приходит `int turnCounter`, если в теле метода он не используется?
```java
public void execute(Map map, int turnCounter) {
  //turnCounter тут не используется
  <some code>
}
```

- Избыточно. 

Все креатуры ходят, поэтому нужно вызывать метод ходьбы на уровне `Creature`, а не на уровне каждого из его потомков
```java
public void execute(Map map, int turnCounter) {
  List<Herbivore> herbivores = map.getHerbivores();
  for (Herbivore herbivore : herbivores) {
    herbivore.makeMove(map);
  }

  List<Predator> predators = map.getPredators();
  for (Predator predator : predators) {
    predator.makeMove(map);
  }
}

//ПРАВИЛЬНО:
public void execute(GameMap gameMap) {
  List<Creature> creatures = gameMap.getEntitiesBy(Creature.class);

  for (Creature creature : creatures) {
    creature.makeMove(gameMap);
  }
}
```

**16. abstract class SpawnAction**

- Этот класс не Action.

Не смотря на слово "Action" в названии, этот класс не экшен.  
Потому что он не состоит в семействе классов `Action`- ни в одном из двух существующих параллельных экшен-семейств. 

**17. class Simulation**

- Контроллер + вью.

Этот класс совмещает в себе два слоя: контроллер и представление(view).  

Если ты используешь Swing, то программа должна быть написана в архитектуре MVC.  
Контроллер должен быть отдельно, представление- отдельно.  

Написание проекта в Swing увеличивает требования к архитектуре, а не понижает их.

ТЗ:
```
Чеклист для самопроверки #
Проблемы и ошибки в коде:
Интеграция библиотеки графического интерфейса ценой чистоты и понятности кода.
```

- Не должно быть двух параллельных семейств экшенов 
```java
private final List<InitAction> initActions = new ArrayList<>();
private final List<TurnAction> turnActions = new ArrayList<>();

//ПРАВИЛЬНО:
private final List<Action> initActions = new ArrayList<>();
private final List<Action> turnActions = new ArrayList<>();
```

Тут кстати можно сразу заполнять списки экшенами:
```java
private final List<Action> initActions = List.of(new UnoAction());
private final List<Action> turnActions = List.of(new DosAction(), new TresAction(), new MoveAction());
```

**18. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

На удивление, интеграция со Swing сделана не так ужасно, как обычно бывает у новичков.
Хотя всё равно неправильно: не разделены Swing'овый контроллер и представление.
Чтобы правильно это делать в Swing, нужно понимать основы архитектуры MVC.

Но классы-модели в проекте при этом сделаны хорошо- они одинаково могут быть использованы и в консольной программе и в Swing.
Я имею ввиду, что они созданы хорошо, как слой- при использовании в других визуальных средах (консоль) в эти модели не нужно будет вносить изменения.  
То есть модели, как слой, не зависят от представления. И само по себе это уже очень хорошо.

В остальном, замечаний к моделям тоже хватает.

Нужно больше подумать над единой ответственностью классов и перераспределить между ними методы в соответствии с SOLID.  
Есть простые ошибки в иерархии классов `Entity`- поведение разных потомков вынесено на базовый уровень. 
И часто потомки получают в наследство ненужный им функционал.

В остальном в алгоритмах на уровне методов видно хорошее логическое мышление.

Посмотреть на ютубе ролики Немчинского про SOLID- по одному ролику на каждый принцип.  
Мой стрим по объектно-ориентированной декомпозиции [Крестики-нолики в ООП стиле](https://t.me/zhukovsd_it_chat/53243/187097)

n.170(365)  
#ревью #симуляция #swing 