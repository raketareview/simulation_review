https://github.com/roadmapjava/simulation  
[распознается графически]

Ещё есть над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Реализованы пауза/пуск во время работы
+ 👍 Меню для создания карт разного размера

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для избежания путаницы не называй пакеты, классы, интерфейсы так же, как называются классы и интерфейсы стандартной библиотеки.
Есть стандартный интерфейс `Map`, не должно быть кастомного пакета с таким же названием
```java
package map;

//ПРАВИЛЬНО:
package gamemap;
```

- Методы должны называться стилем camelCase.  
Возврат константы это не повод нарушать конвенцию наименования
```java
public int getMAP_SIZE()

//ПРАВИЛЬНО:
public int getMapSize()
```

- Константы нужно писать стилем UPPER_CASE, но это не константа.  
Константы должны быть final static
```java
private final int X_SHIFT;

//ПРАВИЛЬНО:
private final int xShift;
```

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относится.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
private final Map<Coordinates, Entity> MAP;

//ПРАВИЛЬНО:
private final Map<Coordinates, Entity> entities; // или coordinatesWithEntities
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется.**
 
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без `else` будет выглядеть читабельней
```java
if (isEmpty(coordinates)) {
  addEntity(coordinates, new Predator(2, 100));
  break;
} else {
  continue;
}

//ПРАВИЛЬНО:
if (isEmpty(coordinates)) {
  addEntity(coordinates, new Predator(2, 100));
  break;
} 
continue;
```

**3. Упрощай код, вводи поясняющие переменные**
```java
if (entity.getClass().getSimpleName().equals("Predator")) {
  target = getTarget(worldMap, entity, Herbivore.class);
} else if (entity.getClass().getSimpleName().equals("Herbivore")) {
  target = getTarget(worldMap, entity, Grass.class);
}

//ПРАВИЛЬНО:
String name = className = entity.getClass().getSimpleName();
if (className.equals("Predator")) {
  target = getTarget(worldMap, entity, Herbivore.class);
} else if (className.equals("Herbivore")) {
  target = getTarget(worldMap, entity, Grass.class);
}
```

Хотя конкретно тут лучше подойдет switch:
```java
String className = entity.getClass().getSimpleName();
Entity target = switch(className) {
  case"Predator" -> getTarget(worldMap, entity, Herbivore.class);
  case"Herbivore" -> getTarget(worldMap, entity, Grass.class); 
  default -> //бросить исключение
};
```

**4. Не обращайся к таблице ASCII по индексам символов, обращайся непосредственно к символам**
```java
if (input.length() != 1 || input.charAt(0) < 49 || input.charAt(0) > 51 || input.equals(" "))

//ПРАВИЛЬНО:
if (input.length() != 1 || input.charAt(0) < '1' || input.charAt(0) > '3' || input.equals(" "))
```

**Нарушение конвенции кода, в одном файле должен быть один класс**

В файле Simulation.java находится несколько равнозначных классов.  
А должен быть только один класс.


**5. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. 

- Класс может быть преобразован в record без потери функционала.

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**6. class CoordinateShift**

Это координата для сдвига
```java
public class CoordinatesShift {
  private final int X_SHIFT;
  private final int Y_SHIFT;
  //...
}
```

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates. 
Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```java
Coordinates shifDownRightCoordinates = new Coordinates(-1, 1);
```

- Кроме хранения самой координаты, класс содержит метод, который возвращает соседние координаты 
```java
public static List<CoordinatesShift> getDefaultShifts() {
  List<CoordinatesShift> shifts = new ArrayList<>();
  shifts.add(new CoordinatesShift(-1, -1));
  shifts.add(new CoordinatesShift(-1, 0));
  //...  
  return shifts;
}
```

Находить соседние координаты это явно не ответственность самой координаты.  

Метод для нахождения соседних координат нужен процессу поиска пути.  
Значит его нормальное месторасположение- в классе поиска пути.

Если разные классы ищут соседние координаты, значит этот метод нужно вынести в третий класс.  
Например:
```java
public final class CageMapUtils {
  //приватный конструктор

  public static List<Coordinates> getAdjacentCoordinates(GameMap gameMap, Coordinates coordinates) {
    //возвращает список соседних координат,
    //которые находятся внутри gameMap
  }
}
```

то луч

**7. class WorldMap**

В таких случаях используй перечисления
```java
public class WorldMap {
  private final int MAP_SIZE;  
  //...
  public WorldMap(int MAP_SIZE) {
    if (MAP_SIZE == 1) {
      this.MAP_SIZE = 10;
    } else if (MAP_SIZE == 2) {
      this.MAP_SIZE = 20;
    } else {
      this.MAP_SIZE = 30;
    }
  }
  //...
}

//ПАРВИЛЬНО:
public class WorldMap {
  private final int side;
  //...
  public WorldMap(Size size) {
    this.side = size.getValue();
  }

  //...
  public enum Size{
    SMALL(10),
    MIDDLE(20),
    LARGE(30);
    private final int value;

    public Size(int value) {...}

    public int getValue() {...}
  } 
}
```

- Нет оснований ограничивать размеры карты тремя вариантами.  
Нет оснований ограничивать геометрию карты квадратом
```java
public class WorldMap {
  private final int MAP_SIZE;  
  //...
  public WorldMap(int MAP_SIZE) {...}
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private final int width;  
  private final int height;  
  //...
  public WorldMap(int width, int height) {...}
  //...
}
```

- Нарушение OCP.

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Creature> getCreatures()

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент отбирает отсюда креатур

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ определенного класса 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getBirds()`.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 

Если координата некорректна(находится вне пределов карты), нужно бросать исключение.  
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что она свободна.  
А правильный ответ- ячейки с такой координатой в карте нет вообще
```java
public boolean isEmpty(Coordinates coordinates) {
  return !MAP.containsKey(coordinates);
}

//ПРАВИЛЬНО:
public boolean isEmpty(Coordinates coordinates) {
  validate(coordinates);  <-- Бросает исключение, если координата не находится в пределеах карты
  return !coordinatesWithEntities.containsKey(coordinates);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива. 
При попытке обратитья к ним по несуществующему индексу, бросается исключение.

- Никогда не возвращай null
```java
private final Map<Coordinates, Entity> MAP;

public Entity getEntity(Coordinates coordinates) {
  return MAP.get(coordinates);  <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Нарушение DRYY, дублирование кода.  
Общий код выноси во вспомогательные методы
```java
public void setupDefaultEntityPositions() {
  while (true) {
    int randomX = RANDOM.nextInt(maxRandom);
    int randomY = RANDOM.nextInt(maxRandom);
    Coordinates coordinates = new Coordinates(randomX, randomY);
    if (isEmpty(coordinates)) {
      addEntity(coordinates, new Predator(2, 100));
      break;
    } else {
      continue;
    }
  }

  while (true) {
    //то же самое      
    if (isEmpty(coordinates)) {
      addEntity(coordinates, new Herbivore(1, 100));
      //то же самое
    }
  }
}

//ПРАВИЛЬНО:

public void setupDefaultEntityPositions() {
  spawn(() -> new Predator(2, 100), КОЛИЧЕСТВО_ВОЛКОВ);
  spawn(() -> new Herbivore(1, 100), КОЛИЧЕСТВО_ЗАЙЦЕВ);
  //...
}

private void spawn(Supplier<Entity> entitySupplier, int count) {
  for(int i = 0; i < count; i++) {
    Coordinates coordinates = getRandomFreeCoordinates();
    Entity entity = entitySupplier.get();
    addEntity(coordinates, entity);
  }  
}    
```

- Карта не должна сама себя заселять существами.

Иначе она становится не универсальной и нельзя будет создать несколько конфигураций игры с разными начальными комбинациями существ на карте
```java
public void setupDefaultEntityPositions() {
  //начальное заселение карты существами  
}
```
Объект не должен сам себя конструировать.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

В ТЗ указано, кто должен делать начальное заселение существ в карту- специальный экшен.

+ 👍 Нарушения SRP есть, но их относительно немного. Для этого класса это уже хорошо.

**8. class PathSearcher**

Нарушение SRP.

Метод поиска не получает нужные данные для выполнения поиска пути по алгоритму BFS или A*.  
А значит, поиск пути каким-то образом самостоятельно получает эти данные.  
То есть, выполняет часть чужой ответственности
```java
public List<Coordinates> search(WorldMap worldMap, Entity entity)
```

Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритма BFS.  
В данном случае- до точки, в которой находится существо нужного класса
```java
public class BfsPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

Или поиск должен искать путь между точками старта и финиша, согласно алгоритма A*
```java
public class AstarPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates from, Coordinates to) {...}
}
```

Так же, поиск пути по алгоритму A* может использовать ту же сигнатуру, что и BFS.  
Тогда поиск пути в нем интерпритируется не как поиск пути между двумя точками, а как поиск цели и прокладывание пути к ней.  
Например: 
```java
public class AstarPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
    //сначала находит координату цели
    //потом прокладывает путь по алгоритму A* между точкой стартой и точкой цели
  }
}
```
Плюсы- унификация сигнатуры разных алгоритмов поиска для полиморфизма.  
То есть, можно сделать семейство родственных классов поиска, объединенных общим интерфейсом.

**9. abstract class Entity**

Содержит координату. Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`
```java
private Coordinates coordinates;
```

**10.abstract class Creature extends Entity**

- Это не константа
```java
private final int SPEED;

//ПРАВИЛЬНО:
private final int speed;
```

+ 👍 Особых замечаний нет.

**11. class Herbivore/Predator extends Creature**

- Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
for (int i = 0; i < moves.size(); i++) {
  if (!worldMap.isEmpty(moves.get(i))) {
    if (worldMap.getEntity(moves.get(i)).getClass() == Herbivore.class) {
      if (i == 0) {
        //наконечник стрелы
      }
    }
  }
}
```
Стрела значит, что метод делает несколько дел сразу и его нужно разделить на несколько вспомогательных.
```java
"Если вам нужно более трех уровней вложенности, вы все равно запутались, так что исправьте программу" - Линус Торвальдс.
```

- Упрощай сложные условия в if'ах
```java
if (worldMap.getEntity(moves.get(i)).getClass() == Herbivore.class) {...}

//ПРАВИЛЬНО:
boolean isFood = worldMap.getEntity(moves.get(i)).getClass() == Herbivore.class;
if (isFood) {...}
```

- Индусский код.  

Оба куска кода делают одно и то же.  
Только первый участок кода выполняет это действие два раза, а второй участок- один раз.  
Сделай это через цикл 
```java
if (i == 0) {
  herbivore.setHp(herbivore.getHp() - this.getPOWER());
  herbivore.setHp(herbivore.getHp() - this.getPOWER());
 speed -= 2;
} else {
 herbivore.setHp(herbivore.getHp() - this.getPOWER());
 speed--;
}
```

**12. abstract class Action**

👍 Идеально
```java
public abstract class Action {
  public abstract void perform(WorldMap worldMap);
}
```

**13. class AddingEntitiesAction extends Action**

Нарушение DRY, дублирование кода.  
Общий код выноси во вспомогательные методы
```java
if (herbivoreCounter <= 2) {
  //куча кода
  worldMap.addEntity(coordinates, new Herbivore(1, 100));
  //куча кода
}
if (grassCounter <= 2) {
  //тот же код
  worldMap.addEntity(coordinates, new Grass());
  //тот же код    
}
```

**14. class MoveAction extends Action**

👍 Почти идеально
```java
public class MoveAction extends Action {
  @Override
  public void perform(WorldMap worldMap) {
    List<Creature> creatures = worldMap.getCreatures();
    for (Creature creature : creatures) {
      if (creature != worldMap.getEntity(creature.getCoordinates())) {  <-- Какая-то лишняя перестраховка
        continue;
      }
      creature.makeMove(worldMap);
    }
  }
}

//ПРАВИЛЬНО:
public class MoveAction extends Action {
  @Override
  public void perform(WorldMap worldMap) {
    List<Creature> creatures = getCreatures();
    for (Creature creature : creatures) {
      creature.makeMove(worldMap);
    }
  }

  private List<Creature> getCreatures(WorldMap worldMap) {
    return worldMap.getEntitiesBy(Creature.class); //например, так
  }
}
```

**15. class WorldMapCreator**

- Стандартным являениям давай стандартные названия. Это простая фабрика
```java
public class WorldMapCreator {
  public static WorldMap create() {...}
}

//ПРАВИЛЬНО:
public class WorldMapFactory {
  public static WorldMap create() {...}
}
```

- Соблюдай требования утилитных классов. 

Это утилитный класс, он содержит только статические методы.  
Утилитные классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- Нарушение DRY, магические буквы, числа, слова. Вводи константы 
```java
System.out.println("""
        Добро пожаловать в симуляцию!
    Пожалуйста, введите цифры от 1 до 3, обозначающие размер карты, где
    1. Маленький
    2. Средний
    3. Большой""");

case 1:  //...
case 2:  //...
case 3:  //...

//ПРАВИЛЬНО:
private final static int SMALL = 1;
//...

System.out.printf("Пожалуйста, введите цифры от %d до %d, обозначающие размер карты, где %n", SMALL, LARGE);
System.out.println(SMALL + ". Маленький");
System.out.println(MIDDLE + ". Средний");
System.out.println(LARGE + ". Большой");

case SMALL:  //...
case MIDDLE:  //...
case LARGE:  //...
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**16. class Simulation**

- Нарушение инкапсуляции. Эти поля должны быть приватными, а доступ к ним- только через методы
```java
public static volatile boolean isOngoing = true;
public static volatile boolean isPaused = false;
```

- Нарушение DI. Класс не должен сам себя конструировать.  
В данном случае класс конструируетсам себя тем, что сам создает карту, а не принимает ее в конструктор
```java
public class Simulation {
  private final WorldMap worldMap = WorldMapCreator.create();
  //...
}
```
Таким образом нельзя создать экземпляр Симуляции с заранее заданными размерами.  
Правильно так:
```java
public class Simulation {
  private final WorldMap worldMap;
  //...
  public Simulation(WorldMap worldMap) {
    this.worldMap = worldMap;
  }
}
```
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Если во время работы программы не планируется изменять состав списка, а здесь не планируется, то используй неизменяемый список
```java
private List<Action> initActions = new ArrayList<>(List.of(
      new InitialEntitiesPlacementAction()
  ));

//ПРАВИЛЬНО:
private List<Action> initActions = List.of(new InitialEntitiesPlacementAction());
```

- Нарушение SRP.

Класс не управляет потоками, поэтому не должен печатать меню управляющего потока
```java
private void nextTurn() {
  //...
  System.out.println("Количество ходов: " + movesCyclesCounter);
  System.out.println("""
    Введите цифры 1, 2 или 3, где:
    1. Поставить симуляцию на паузу
    2. Возобновить симуляцию
    3. Прекратить симуляцию и завершить работу""");
}
```
Это меню нужно перенести в `class InputThread`.

- Точка входа main должна быть не здесь, а в отдельном классе
```java
public class Simulation {
  //...

  public static void main(String[] args) {
    Simulation simulation = new Simulation();
    new InputThread().start();
    simulation.startSimulation();
  }
}
```
*"ЧК", гл.11, "Отделение main"*

## ВЫВОД

Замечаний к реализации меньше, чем обычно.  
Посмотреть на ютубе ролики по SOLID.

n.147(309)  
#ревью #симуляция 