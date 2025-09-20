https://github.com/writer366/project-2  
[writer366]

Есть над чем поработать. 

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Когда я ввожу команду запустить симуляцию, программа некотрое время тупит, а только потом распечатывает карту.  
Должно быть наоборот: после запуска сразу напечатать карту, пауза и все по кругу.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализовано пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для избежания путаницы не называй пакеты, классы, интерфейсы так же, как называются классы и интерфейсы стандартной библиотеки.
Есть стандартный интерфейс `Map`, не должно быть кастомного пакета с таким же названием
```
package map;

//ПРАВИЛЬНО:
package entity_map;
```

- Есть стандартный интерфейс `List`. не должно быть кастомных пакетов с таким же названием
```
package lists;
```

- Не называй класс "List", если он не реализует стандартный интерфейс `List`. Здесь не реализует
```
abstract class EntityList<E extends Entity> {...}
```

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```
Map<Coordinate, Entity> map;

//ПРАВИЛЬНО:
Map<Coordinate, Entity> entities;
```

- Не дублируй имя класса в навании полей класса
```
public class Node {
  private Node parentNode;
  private Coordinate coordinateNode;
  //...
}

//ПРАВИЛЬНО:
public class Node {
  private Node parent;
  private Coordinate coordinate;
  //...
}
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**. Систематически константы объявляются ниже полей класса
```
private Map<Coordinate, Boolean> visited;
private final static int SAME_AXIS = 0;
private final static int ADJACENT_AXIS = 1;
```

**3. Нарушение конвенции кода**. В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (!way.isEmpty())
  return way.pop();
else
  return comeFrom();

//ПРАВИЛЬНО:
if (!way.isEmpty()) {
  return way.pop();
} else {
  return comeFrom();
}
```

**4. Нарушение DRY**, магические буквы, числа, слова. Вводи константы.

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов.  
И ОБРАЩАЙСЯ к этим константам
```
public class Renderer {
  //...
  public void printMenu() {
    System.out.print(""" 
        Выберите вариант симуляции:
        1 - Просимулировать один ход 
        2 - Запустить бесконечный цикл симуляции (4 - Приостановить бесконечный цикл симуляции)
        3 - Выйти из программы
        """);
  }
}

public class Simulation {
  private final static String STEP = "1";
  private final static String ENDLESS_CYCLE = "2";
  private final static String EXIT = "3";
  //...
}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**5. class Coordinate**

- Нарушение конвенции кода- константа ниже полей класса.

- Нарушение SRP. Это не должно здесь находиться
```
public static final int COORDINATE_NOT_FOUND = -1;
```
Класс Координата просто хранит информацию о положении точки в двумерном пространстве. 
Для выполнения этого действия, координате не нужн хранить в себе коды чьих-то посторонних операций.

+ 👍 Хорошо, что поля координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

- Если убрать ненужную для класса константу, то класс можно преобразовать в record без потери им функционала.
В этом случае хешкод, иквалс и геттеры в рекорде создадутся автоматически и неявно. 

**6. class EntityMap**

- Карта содержит в себе фиксированные размеры, поэтому становится не универсальной
```
public static final int WIDTH = 14;
public static final int HEIGHT = 14;
```
Фиксированные размеры могут быть у шахматной доски(8x8) или игры крестики-нолики(3x3).  
Здесь размеры должны приниматься в конструктор, чтобы можно было создавать карту произвольных размеров.

- Карта не должна хранить запись координата-null, то есть пустые ячейки. 
Иначе теряется  все преимущество, которое дает HashMap для хранения сущностей
```
for (int i = 0; i < WIDTH; i++) {
  for (int j = 0; j < HEIGHT; j++) {
    map.put(new Coordinate(i, j), null);  <-- ЗАПОЛНЯЕТ КАРТУ ПУСТЫМИ ЯЧЕЙКАМИ
  }
}
```
Фактически, тут HashMap становится аналогом обычного массива, только хуже, бесполезно забивая память.
Например, есть игровая доска 100*100 и в ней единомоментно находится общим числом 150 разных существ: камней, травы, зайцев и т.д.
Тогда при нормальном хранении этих данных в мапе с помощью ключ-значение, потребуется 150 записей.
Если в этой же мапе кроме этих значимых данных хранить еще пустые сущности, то потребуется уже 10.000 записей.

Для определения того, что в точке карты никого нет, достаточно ввести метод 
```
private boolean isПустаяЯчейка(Координата координата) {...}
```

+ 👍 Хорошо, что геттер возвращает Optional, а не null
```
public Optional<Entity> getEntity(Coordinate coordinate) {
  return Optional.ofNullable(map.get(coordinate));
}
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.

Если координата некорректна(находится вне пределов карты), нужно бросать исключение.
Сейчас можно поселить сущетсво на координату, выходящую за пределы карты
```
public void putEntity(Coordinate coordinate, Entity entity) {
  map.put(coordinate, entity);
}
```

**7. class BreadthFirstSearch**

- Нарушение SRP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar. 
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```
if (entityMap.getEntity(coordinateStartNode).isPresent() && entityMap.getEntity(coordinateStartNode).get() instanceof Herbivore) {...}
if (entityMap.getEntity(coordinateStartNode).isPresent() && entityMap.getEntity(coordinateStartNode).get() instanceof Predator) {...}
```

Класс не должен ковыряться в Зайцах с Волками и выпытывать у них чего они хотят. 
Всё необходимое для поиска метод должен принимать во входящие аргументы.  
Например, так:
```
public List<Coordinates> getPatch(World world, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте world от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```

- Как правильно пользоваться поиском, совершенно неясно- в классе куча публичных методов. 
В этом классе должен быть только один публичный, то есть предназначенный для клиентских классов, метод- тот, который найдет и вернет путь.

- Ацки длинное, а потому нечитаемое условие. Вводи вспомогательные методы, которые своим названием будут объяснять, что здесь происходит
```
if (((difRow == ADJACENT_AXIS && difColumn == SAME_AXIS) || (difColumn == ADJACENT_AXIS && difRow == SAME_AXIS)) && (isHerbivore || isPredator)) {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(...)) {...}
```

- Каждый раз при поиске не нужно создавать мапу со всеми существующими координатами
```
private Map<Coordinate, Boolean> visited

for (int i = 0; i < EntityMap.WIDTH; i++) {
  for (int j = 0; j < EntityMap.HEIGHT; j++) {
    this.visited.put(new Coordinate(i, j), false);
  }
}
```
Ты тут явно делаешь что-то не то, перечень посещенных точек нужно пополнять только по мере движения.


**8. class Node**

👍 Нода, то есть узел однонаправленного связного списка, это полезный вспомогательный класс для поиска BFS.

**9. abstract class Entity и его простые наследники**

👍 Идеально
```
public abstract class Entity {
}

public class Grass extends Entity {
}
```

**10. abstract class Creature extends Entity**

- Класс не должен принимать Карту в конструктор. 

Карта не является частью внутреннего устройства Креатуры. 
Карта нужна только для совершения хода 
```
public abstract class Creature extends Entity {
  //...
  public Creature(EntityMap entityMap) {...}
  public void makeMove() {...}
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //...
  public Creature() {...}
  public void makeMove(EntityMap entityMap) {...}
}
```

- Нарушение ООП дизайна в наследовании. Эти поля нужно не сетить, а инжектить в конструктор
```
public abstract class Creature extends Entity {
  protected int speed;
  protected int hp;
  //...

  public Creature(EntityMap entityMap) {
    this.entityMap = entityMap;
  }
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  protected int speed;
  protected int hp;
  //...

  public Creature(int speed, int hp) {
    this.speed = speed;
    this.hp = hp;
  }
}
```

- Нарушение инкапсуляции. 

Классы не должны светить наружу всеми своими методами. 
Публичными должны быть только те методы, которые предназначены для использования клиентскими классами.
Остальные должны быть ptivate или protected.  
Например, этот метод используется только в своем классе и поэтому должен быть private
```
public Coordinate getCoordinate()
```

- Нарушение SRP. 

Класс забирает себе часть ответственности Карты- реализует поиск себя во внутренностях Карты
```
public Coordinate getCoordinate() {
  //миллион строк
  //в которых метод 
  //копается в классе EntityMap
}  
```
Сделай в Карте метод `public Coordinate getCoordinate(Entity entity)` и вызывай его в этом классе.

- Не используй исключения в качестве условных операторов для ветвления кода. 
Исключения придуманы не для этого
```
try {
  for (Coordinate coordinate : entityMap.getCoordinates()) {
    if (entityMap.getEntity(coordinate).isPresent() && entityMap.getEntity(coordinate).get().equals(this)) {
      return coordinate;
    }
  }
  throw new IllegalStateException();
} catch (IllegalStateException e) {
  //действия  
}
```

Вот равнозначный нормальный код:
```
for (Coordinate coordinate : entityMap.getCoordinates()) {
  if (entityMap.getEntity(coordinate).isPresent() && entityMap.getEntity(coordinate).get().equals(this)) {
    return coordinate;
  }
}
//действия  
```

- Нарушение SRP, чужая ответственность, зависимость модели от представления.
Модель(а это модель) не должна ничего печатать в консоль
```
System.out.println("Creature not found on the map");
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

- Не возвращай коды или объекты ошибок.

Если метод не может найти свое сущетсво в карте, он возвращает координату с кодами ошибок
```
public Coordinate getCoordinate() {
  //...
  return new Coordinate(Coordinate.COORDINATE_NOT_FOUND, Coordinate.COORDINATE_NOT_FOUND);
}
```
Типа, если координата содержит в себе недопустимые значения, то существо не найдено.

Подход неправильный. Если существо пытается найти в карте себя и не находит, это баг алгоритма. 
В этом случае баг нужно максимально быстро выявить и устранить в алгоритме те причины, которые приводят к этому багу. 

То есть, нужно не возвращать фейковую координату, а бросать исключение.  
*"ЧК", гл. 7, "...вместо кодов ошибок"* 

**11. Пакет lists, class `EntityList<E extends Entity>` и его наследники**

Содержит очень странные классы
```
abstract class EntityList<E extends Entity>
class GrassList extends EntityList<Grass>
lass HerbivoreList extends EntityList<Herbivore>
class PredatorList extends EntityList<Predator>
```
Шо оно такое, а главное зачем, я пытался разобраться минут пять. Так и не понял
```
public abstract class EntityList<E extends Entity> {
  protected EntityMap entityMap;
  protected List<E> list;

  public EntityList(EntityMap entityMap) {...}

  protected abstract Class<E> getEntityType();

  private void add() {
    for (Coordinate coordinate : entityMap.getCoordinates()) {
      Optional<Entity> optionalEntity = entityMap.getEntity(coordinate);
      Entity entity = optionalEntity.orElse(null);
      if (getEntityType().isInstance(entity)) {
        list.add(getEntityType().cast(entity));
      }
    }
  }

  public List<E> getList() {
    this.add();
    return Collections.unmodifiableList(list);
  }
}
```
Одно ясно наверняка- несмотря на свое название, этот класс не имеет никакого отношения к стандартному интерфейсу `List`.

Нужно уничтожить весь этот ужас без всякого сожаления. 

**12. abstract class Move implements Action и его наследники**

Нарушение DRY. Три класса, которые делают одно и то же- передвигают существ по карте
```
public abstract class Move implements Action 
public class MoveHerbivore extends Move
public class MovePredator extends Move
```

Не должно быть отдельных классов для передвижения каждого вида существ. 
Должен быть один класс, который будет двигать все Креатуры одинаково через полиморфизм.

**13. abstract class Placement implements Action и его наследники**

Та же самая ситуация. Куча классов, которые делают одно и то же- заселяют карту существами
```
public abstract class Placement implements Action
public class GrassPlacement extends Placement
public class HerbivorePlacement extends Placement
public class PredatorPlacement extends Placement
public class RockPlacement extends Placement
public class TreePlacement extends Placement
```

Вместо отдельного класса на каждый вид существ, нужно сделать универсальный класс, который будет принимать в конструктор количество создаваемых существ и способ их создания.
Способ создания можно передавать через стандартный интерфейс Supplier
```
SpawnAction заяцSpawnAction = new SpawnAction(() -> new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ);
SpawnAction волкSpawnAction = new SpawnAction(() -> new Волк(), КОЛИЧЕСТВО_ВОЛКОВ);
заяцSpawnAction.execute(карта); 
волкSpawnAction.execute(карта); 
//...

public class SpawnAction extends Action {
  private final Supplier<Entity> entitySupplier;
  private final int amount;

  public SpawnAction(Supplier<Entity> entitySupplier, int amount) {
    this.entitySupplier = entitySupplier;
    this.amount = amount;
  }

  @Override
  public void execute(Карта карта) {
    for (int i = 0; i < amount; i++) {
      Entity entity = entitySupplier.get();
      Coordinates coordinates = getRandomEmptyCoordinates(карта);
      карта.put(entity, coordinates);
    }
  }

  private Coordinates getRandomEmptyCoordinates(Карта карта) {
    //находит и возвращает случайную пустую координату
  }
}
```

**14. public class Renderer**

- Снова странное использование исключения
```
default:
  try {
    throw new IllegalStateException();
  } catch (IllegalStateException e) {
    System.out.println("Unexpected value: " + entity);
  }

//ЭТО РАВНОЦЕННО ТАКОМУ КОДУ:
default: System.out.println("Unexpected value: " + entity);
```

- В switch-case в default нужно кидать исключение, а не писать "Unexpected value: ".

Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
Такую ситуацию нужно сразу выявить и устранить.

- Вместо огромного свич-кейса в методе распечатки, сделай вспомогательный метод: `private String toSprite(Entity entity)`.

**15. class Simulation**

Нарушение DI. Класс не должен инициализировать сам себя. 
В данном случае класс инициализирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```
public class Simulation {
  public Simulation() {
    //...
    this.entityMap = new EntityMap();
    //...
  }
}
```
Таким образом нельзя создать несколько игровых конфигураций с разными картами.  
*Мартин, "ЧК", гл.11.2*

**16. class Main**, содержит точку входа main

👍 Только создает и запускает Simulation, это хорошо.

## ВЫВОД

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. 
Посмотреть ролики Немчинского про SOLID.

Видно, что есть проблемы с пониманием базы ООП: наследование и полиморфизм.  
Например, хотя все наследники Creature имеют полиморфный метод движения, их тут двигают не полиморфно, а персонально.

Почитай статьи, посмотри видосики на ютубе про три принципа ООП. 
Возможно, сделай пару простых упражнений на наследование и полиморфизм.

n.102(231)  
#ревью #симуляция 