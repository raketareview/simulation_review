https://github.com/Ciltonn/SimGame  
[Елена Казакова]

В целом, неплохо. Но есть, над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты в существах хранятся начиная с Creature 
+ 👍 Реализовано пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов нужно писать стилем lower_snake
```
Factory
BFS

//ПРАВИЛЬНО:
factory
bfs
```

- Эта пара обычно называется row/column
```
public class Coordinates {
  private int line;
  private int column;
  //...
}
```

- Название метода должно быть глаголом в повелительном наклонении
```
List<Coordinates> pathFinder(...)

//ПРАВИЛЬНО:
List<Coordinates> findPath(...)
```

- Геттер должен что-то возвращать. Геттер, который ничего не возвращает- нонсенс
```
void getSprite(Location location, Coordinates coordinates)
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
System.out.println("[e] для выхода");
if (input.equals("e")) {...}

//ПРАВИЛЬНО:
private final static String EXIT = "e";

System.out.printf("[%s] для выхода \n", EXIT);
if (input.equals(EXIT)) {...}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**3. Инкремент в одной строке с другими операциями плохо понимается**
```
System.out.println("Turn" + turnCount++);
```
Внимание вопрос, что распечатает инструкция: 1 или 2?
```
int turnCount = 1;
System.out.println("Turn" + turnCount++);
```

Правильный ответ 1, но при прочтении кода на лету ответ отгадают не только лишь все. Поэтому лучше упрощать:
```
System.out.println("Turn" + turnCount);
turnCount++
```

**4. class Coordinates**

- Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: x,y или row,column.
Лучше даже, если этот класс будет record'ом.

Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия, тем самым нарушая SRP: проверяет координату на валидность для класса Карты и содержит в себе размеры Карты
```
public class Coordinates {
  public static final int MAX_LINE = 10;  <-- ЭТО ПРОБЛЕМЫ КАРТЫ
  public static final int MAX_COLUMN = 15; <-- ЭТО ПРОБЛЕМЫ КАРТЫ
  //...

  public boolean isValid() {  <-- ЭТО ПРОБЛЕМЫ КАРТЫ
    return ((getLine() < MAX_LINE && getLine() >= 0) &&
        (getColumn() < MAX_COLUMN && getColumn() >= 0));
  }
  //...
}
```
Координата не должна вообще знать про существование карты. И тем боле не должна решать за карту ее проблемы.

- Для координаты оптимальным решением будет использовать record- это подойдет для симуляции и любой другой игры на прямоугольной доске(Морсксой бой, Шахматы и т.д.):
```
public record Coordinates (int row, int column){
}
```
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Единственный нюанс- record'ы immutable, а значит их значения могуть быть только для чтения и сеттеры для них сделать невозможно.
Но это для координаты даже лучше- нет особых оснований делать поля координаты row и column изменяемыми.

**5. class Location**, карта симуляции

Одна из самых компактных реализация Карты
```
public class Location {
  private final Map<Coordinates, Entity> entities = new HashMap<>();

  public void addEntity(Coordinates coordinates, Entity entity) {
    entities.put(coordinates, entity);
  }
  public void removeEntity(Coordinates coordinates) { entities.remove(coordinates); }
  public Entity getEntity(Coordinates coordinates) { return entities.get(coordinates); }
  public List<Entity> getEntities() { return new ArrayList<>(entities.values()); }
}
```

- Нарушение SRP. Карта здесь не отвечает за свои размеры, эту часть ее ответственности берет на себя Координата. 
Координата хранит в себе в виде констант размеры карты и проверяет координаты на расположение в пределах карты.
Методы `boolean isValid(Coordinates coordinates)` нужно переместить в карту.

- Карта содержит в себе фиксированные размеры- которые, опять же, в виде констант хранятся не в ней, а в Координате. 
Карта с фиксированнными размерами становится не универсальной.  

Фиксированные размеры могут быть у шахматной доски(8x8) или игры крестики-нолики(3x3).  
Здесь размеры должны приниматься в конструктор, чтобы можно было создавать карту произвольных размеров.

Хранение констант с размерами карты это еще одно нарушение единой ответстсвенности класса карты- эту часть ее ответственности присвоила себе Координата. 
За свои размеры карта должна отвечать сама, а не перекладывать эту обязанность на сторонние классы.
Другие классы, когда им нужно узнать размеры карты, например Рендерер, должны спрашивать эти размеры у самой карты, а не у сторонних классов.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void addEntity(Coordinates coordinates, Entity entity) {
  entities.put(coordinates, entity);
}

//ПРАВИЛЬНО:
public void addEntity(Coordinates coordinates, Entity entity) {
  if( /*координата не находится в пределах карты*/) {
    throw new ... //бросить исключение
  }
  entities.put(coordinates, entity);
}

```
Сейчас можно поместить существо на координату, которая находится за пределами карты.

- Никогда не возвращай null
```
private final Map<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) { 
  return entities.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- В целом, класс не содержит в себе ничего лишнего- и в этом смысле не нарушает принципа единой ответственности. 
Но он нарушает SRP по-другому- часть своей ответственности он передал другим классам.

Карта должна хранить в себе и сообщать свои размеры- без этой информации клиентскому коду будет невозможно пользоваться картой, а значит эти методы составляют часть ее SRP.  
Карта должна содержать в себе публичный метод, который будет проверять координату на ее вхождение в размеры карты.  
Карта должна содержать в себе приватный метод валидации координаты- если координата не находится в пределах размеров карты, нужно кидать исключение. 
Валидацию нужно проводить во всех методах, во входящие аргументы которых приходит координата.

**6. class BFS**, поиск пути

- Внезапный нейминг. Никто, кроме заядлых шахматистов, не знает, что такое "rank" и "file" применительно к координатам. 
Остальным слово "file" в этом контексте может поломать мозг
```
int newRank = current.getLine() + d[0];
int newFile = current.getColumn() + d[1];

Coordinates neighbors = new Coordinates(newRank, newFile);

//ПРАВИЛЬНО:
int line = current.getLine() + d[0];
int column = current.getColumn() + d[1];

Coordinates neighbors = new Coordinates(line, column);
```

- Смещения нужно вынести из метода и сделать константами.  
И, что такое пара х/у? Правильно, координата
```
public class BFS {
  //
  public List<Coordinates> pathFinder(...) {
    int[][] dir = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
    //...
  }
}

//ПРАВИЛЬНО:
public class BFS {
  private final static List<Coordinates> SHIFT_COORDINATES = List.of{
    new Coordinates(0, 1),
    new Coordinates(1, 0),
    //...
  };  
  //
  public List<Coordinates> pathFinder(...) {
    //...
  }
}
```

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем опроса Creature о его еде
```
public List<Coordinates> pathFinder(Location location, Class<? extends Creature> hunterClass, Coordinates start) {
  //...
  Class<? extends Entity> targetClass = Creature.getTarget(hunterClass);
  //..
}

//ПРАВИЛЬНО:
public List<Coordinates> pathFinder(Location location, Coordinates start, Class<? extends Entity> target) {
  //...
}
```

- Нарушение инкапсуляции. Публичным должен быть только один метод- тот, который ищет путь.

- Как я уже писал, никогда не возвращай null
```
public List<Coordinates> pathFinder(Location location, Class<? extends Creature> hunterClass, Coordinates start) {
  if (start == null) {
    return null;
  }
  //...
}
```

**7. class Factory**

- Название просто "Factory" может быть у интерфейса или какого-то абстрактного класса, от которых будут реализации конкретных файбрик
```
class Factory

//ТУТ ПРАВИЛЬНО:
class КонкретнаяПродукцияFactory
```

- Нарушение инкапсуляции. Публичным должен быть только один метод- тот, который производит продукцию.
Те методы, которые являются вспомогательными, должны быть private. 

- Другое дело, что непонятно, что эта фабрика производит. Потому что она ничего не производит:
```
public class Factory {
  public void setEntities(Location location) {...}
  public void spawnPredator(Location location) {...}
  //...
} 
```
Фабрика должна производить продукцию, либо это не фабрика.

Вот пример фабрики, которая делает Карту и первоначально заселяет ее:
```
public class LocationFactory {
  private final static WIDTH = 20;
  private final static HEIGHT = 15;

  private final static PREDATOR_AMOUNT = 3;
  private final static TREE_AMOUNT = 5;
  private final static PREDATOR_AMOUNT = 3;
  //...

  public Location get() {
    Location location = new Location(WIDTH, HEIGHT);

    spawn(location, (coordinates) -> new Tree(), TREE_AMOUNT);
    
    spawn(location, 
       (coordinates) -> new Predator(
          coordinates, 
          PREDATOR_SPEED, 
          PREDATOR_HP, 
          PREDATOR_POWER
        ), 
        PREDATOR_AMOUNT);
    //...
    return location;
  }

  private void spawn(Location location, Function<Coordinates, Entity> mapper, int amount) {
    for(int i = 0; i < amount; i++) {
      Coordinates coordinates = getRandomEmptyCoordinates(location);
      Entity entity = mapper.apply(coordinates);
      location.addEntity(coordinates, entity);
    }
  }
  //...
} 
```

- Нарушение DRY, опосредованное дублирование кода
```
for (int i = 0; i < 2; i++) {
  spawnTree(location);
}
for (int i = 0; i < 2; i++) {
  spawnRock(location);
}
//еще миллион таких же циклов

public void spawnTree(Location location) {
  Coordinates coordinates = randomCoordinates(location);
  location.addEntity(coordinates, new Tree());
}

public void spawnRock(Location location) {
  Coordinates coordinates = randomCoordinates(location);
  location.addEntity(coordinates, new Rock());
}
//еще миллион таких же методов  
```
В примере фабрики я показал, как при использовании стандартных интерфейсов java, в данном случае Function, можно избавиться от такого дублирования.

**8. class SpawnEntities implements Action**

Нарушение DRY, дублирование кода как в прошлом примере.

**9. abstract class Entity и его простые наследники Tree/Rock/Grass**

👍 Идеально
```
public abstract class Entity {
}
public class Rock extends Entity {
}
```

**10. abstract class Creature extends Entity**

- Неизменяемые поля объекта(не константы) делай final.
Посмотри, там IDE подсказывает- подчеркивает желтым. 
Поставь курсор на это подчеркивание-> слева появится желтая лампочка-> нажи на нее ЛКМ-> "Make 'название поля' final".
```
private int speed;
private int hp;

//ПРАВИЛЬНО:
private final int speed;
private final int hp;
```

+ 👍 Только с этого уровня иерархии Entity хранится координата, это хорошо.

- Нарушение инкапсуляции. Публичными должны быть только те методы, которые предназначены для использования клиентским кодом. 
Внутренние вспомогательные методы должны быть `private` и `protected`.
Например, метод `moveEntity(...)` должен быть private потому что является вспомогательным для публичного `makeMove(...)`, который предназначен для использования клиентом.

- Нарушение LSP. Предок не должен знать своих потомков и реализовывать за них их логику
```
public abstract class Creature extends Entity {

  public static Class<? extends Entity> getTarget(Class<? extends Creature> hunterClass) {
    if (hunterClass.equals(Predator.class)) {
      return Herbivore.class;
    }
    if (hunterClass.equals(Herbivore.class)) {
      return Grass.class;
    }
    return null; <-- НИКОГДА НЕ ВОЗВРАЩАЙ NULL!!!!11
  }
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {

  public abstract Class<? extends Entity> getTarget();
  //...
}

class Herbivore extends Creature {
  private final static Class<? extends Entity> TARGET = Grass.class;

  @Override
  public abstract Class<? extends Entity> getTarget() {
     return TARGET;
  } 
  //... 
}
```

- Сразу кастуй
```
if (entity instanceof Creature) {
  ((Creature) entity).setCoordinates(newPosition);
}

//ПРАВИЛЬНО:
if (entity instanceof Creature creature) {
  creature.setCoordinates(newPosition);
}
```

**11. class Herbivore/Predator extends Creature**

- Нарушение OCP. Каждый раз при добавлении новых типов Entity, придется добавлять их в это условие: ввел в проект Чебурашку- добавил в это условие "&& !!(entity instanceof Чебурашка)"
```
public class Predator extends Creature {
  @Override
  public boolean isPossibleCell(Coordinates coordinates, Location location){
    Entity entity = location.getEntity(coordinates);
    return entity==null|| !(entity instanceof Rock) && !(entity instanceof Tree) && !(entity instanceof Predator) && !(entity instanceof Grass);
  }
}

//ПРАВИЛЬНО:
return entity==null || entity instanceof Herbivore;
```

**12. class Renderer**

+ 👍 Спрайты существ находятся здесь, а не в самих существах. Это хорошо- принцип разделения данных и представления.

- Правило "счетчик в цикле должен быть однобуквенным" не всегда является оптимальным
```
for (int i = Coordinates.MAX_LINE - 1; i >= 0; i--) {
  for (int j = 0; j <= Coordinates.MAX_COLUMN - 1; j++) {
    Coordinates coordinates = new Coordinates(i, j);
  }
}

//ЛУЧШЕ:
for (int line = Coordinates.MAX_LINE - 1; line >= 0; line--) {
  for (int column = 0; column <= Coordinates.MAX_COLUMN - 1; column++) {
    Coordinates coordinates = new Coordinates(line, column);
  }
}
```

- В каждом switch-case должен быть default. Здесь в default нужно кидать исключение. 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- существо просто не распечатается и этот глюк будет выявлен далеко не сразу.

- Геттер должен что-то возвращать
```
public void renderInConsole(Location location) {
  //...  
  if (location.getEntity(coordinates) == null) {
    System.out.print(" . ");
  } else {
    getSprite(location, coordinates);
  }
  //...
}

private void getSprite(Location location, Coordinates coordinates) {
  Entity entity = location.getEntity(coordinates);
  //...

  switch (entity.getClass().getSimpleName()) {
    case "Herbivore":
      System.out.print(" \uD83D\uDC07");
      break;
    case "Predator":
      System.out.print(" \uD83D\uDC3A");
      break;
    //...  
  }
}

//ПРАВИЛЬНО:
private final static String HERBIVORE = "\uD83D\uDC07";
//oths const's

public void renderInConsole(Location location) {
  //...  
  if (location.isEmpty(coordinates)) {
    System.out.print(" . ");
  } else {
    Entity entity = location.getEntity(coordinates);
    String sprite = toSprite(entity);
    System.out.print(" " + sprite);
  }
  //...
}

private String toSprite(Entity entity) {
  return switch (entity.getClass().getSimpleName()) { <-- НОВАЯ УДОБНАЯ ФОРМА ЗАПИСИ
    case "Herbivore" -> HERBIVORE;
    case "Predator" -> PREDATOR;
    default -> throw new ... //неизвестное существо- бросить исключение
  };
}
```

**13. class Simulation**

- Для движения существ нужно использовать специальный Action, а не метод в этом классе
```
private void creatureMove() {...}
```

- Сейчас в классе используется только один Action
```
private final SpawnEntities spawnEntities = new SpawnEntities();
```
Согласно ТЗ, в этом классе должны содержаться экшены, которые вызываются в начале старта симуляции и во время каждого хода.  
Должно быть примерно так:
```
private final List<Action> initActions = List.of(new ПервоначальноеЗаселениеКартыСуществамиAction());
private final List<Action> turnActions = List.of(new ВозобновлениеРесурсовAction(), new MoveAction());
```

В зависимости от алгоритма, в качестве Первоначального заполнения можно использовать Возобновление ресурсов.

- Симуляция должна принимать необходимые зависимости в конструктор. Здесь- как минимум экземпляр карты.
Сейчас карта создается с фиксированными размерами, которые хранятся в константах.  
Когда ты переделаешь карту по-нормальному, то есть когда Карта будет принимать свои размеры в конструктор, то экземпляр карты нужно будет создавать в майне и далее инжектить эту карту в конструктор Симуляции.

- Избыточно
```
class Simulation {
  //...
  List<Entity> entities = new ArrayList<>(location.getEntities()); <-- ЛИШНЯЯ ПЕРЕСТРАХОВКА
  //...
}

public class Location {
  //...
  public List<Entity> getEntities() {
    return new ArrayList<>(entities.values()); <-- ЭТО УЖЕ БЕЗОПАСНО ДЛЯ ВНУТРЕННОСТЕЙ КАРТЫ
  }
}

//ПРАВИЛЬНО:
class Simulation {
  //...
  List<Entity> entities = location.getEntities();
  //...
}
```

**14. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

Сделать Action'ы по ТЗ.  
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы.
Посмотреть ролики Немчинского про SOLID.

n.93(210)  
#ревью #симуляция 