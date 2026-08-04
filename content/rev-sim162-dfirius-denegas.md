https://github.com/denegas/Simulation  
[dfirius]

Есть много над чем поработать.  
Попытка написать архитектуру MVC.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim162/img0.png) 

2. В программе не реализована работа с потоками и выполнение старт/стоп во время работы симуляции.  
Хотя это требование тоже есть в ТЗ.

## ХОРОШО

+ 👍 Можно задавать размеры карты через меню. 
+ 👍 Попытка написать архитектуру MVC. 

## ЗАМЕЧАНИЯ

Проект разбит на пакеты `Model`, `View`, `Controller`.  
Поэтому критика проекта будет с точки зрения архитектуры `MVC`.  

**1. Нейминг**

- Названия пакетов нужно писать стилем alllowercase 
```java
package Controller;

//ПРАВИЛЬНО:
package controller;
```

- Не называй пакет именем "map", это название стандартного интерфейса "Map". 

Не называй пакеты так же, как называются классы, интерфейсы, пакеты библиотеки java core.  
В данном случае- стандартного интерфейса Map
```java
package Model.map;

//ПРАВИЛЬНО:
package model.entitymap;
```

- Не называй поля "переменная1", "переменная2", это совсем плохой тон.

Всегда можно найти более осмысленные названия
```java
public boolean equals(Object o) {
  Coordinates cord1 = (Coordinates) o;
  return (this.getCoordinateX() == cord1.getCoordinateX()) && (this.getCoordinateY() == cord1.getCoordinateY());
}

//ПРАВИЛЬНО:
public boolean equals(Object o) {
  Coordinates other = (Coordinates) o;
  return (this.getCoordinateX() == other.getCoordinateX()) && (this.getCoordinateY() == other.getCoordinateY());
}
```

- Тавтология
```java
public final class Coordinates {
  private final int coordinateX;
  private final int coordinateY;
  //...
}

//ПРАВИЛЬНО:
public final class Coordinates {
  private final int x;
  private final int y;
  //...
}
```

Вот так 
```java
... = coordinates.getX();
```
смотрится естественнее, чем вот так
```java
... = coordinates.getCoordinateX();
```
А вот так было бы совсем хорошо:
```java
... = coordinates.x();
```

- В названии переменной не указывай количество элементов, которое хранит эта переменная.  
Сегодня четыре, завтра пять- это вообще неважно, думай абстракциями
```java
public static final int[][] FOUR_NEAR_DIRECTIONS = {...};

//ПРАВИЛЬНО:
public static final int[][] NEAR_DIRECTIONS = {...};
```

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относятся.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
private final Map<Coordinates, Entity> map = new HashMap<>();

//ПРАВИЛЬНО:
private final Map<Coordinates, Entity> entities = new HashMap<>(); //или coordinatesWithEntities
```

- Это не константа
```java
private final int SIZE;

//ПРАВИЛЬНО:
private final int size;
```

- Придерживайся единообразия, не используй синонимы для одной и той же концепции
```java
boolean mapHasNoCell(Coordinates cell)

//ПРАВИЛЬНО:
boolean mapHasNoCell(Coordinates coordinates)
```

- В названиях `boolean` методов слова "is" или "has" должны стоять в начале названия
```java
boolean mapHasNoCell(Coordinates cell)
```

- Не используй в назывании пакетов слово "object".

Если пакет называется "object", это значит, что там хранятся классы типа Object.  
Но это бессмысленно, потому что в java все классы неявно наследуются от класса Object,
поэтому вообще все классы относятся к типу Object и являются объектами
```java
package Model.entities.objects;
```

- Не используй слово "new".

Во-первых `new` это зарезервированное слово и не нужно его везде использовать всуе.  
Во-вторых, если ты один раз его где-то использовал в качестве префикса передаваемого аргумента, 
то для единообразия наименований нужно будет делать эту приписку в названиях аргументов вообще всех методов в проекте.  
Иначе неясно, почему в одном случае `newCoordinates`, а в другом случае просто `coordinates` 
```java
public void makeMove(Coordinates newCoordinates) {
  setCoordinates(newCoordinates);
}

protected void setCoordinates(Coordinates coordinates) {
  this.coordinates = coordinates;
}

//ПРАВИЛЬНО:
 public void makeMove(Coordinates coordinates) {
  setCoordinates(coordinates);
}

protected void setCoordinates(Coordinates coordinates) {
  this.coordinates = coordinates;
}
```

- Название обманывает.

Этот класс не создает карту. Он заполняет существами уже существующую карту
```java
class MapCreator implements Action 
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется**

В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без else будет выглядеть читабельней
```java
if (mapHasNoCell(nextCell) || (!CellUtils.isCellVoid(nextCell, map) && !CellUtils.isCellTarget(nextCell, target, map))) {
  continue;
} else if (visitedDirections.contains(nextCell)) {
  continue;
} else if (CellUtils.isCellVoid(nextCell, map)) {
  //...  
  continue;
} 

//ПРАВИЛЬНО:
if (mapHasNoCell(nextCell) || (!CellUtils.isCellVoid(nextCell, map) && !CellUtils.isCellTarget(nextCell, target, map))) {
  continue;
} 
if (visitedDirections.contains(nextCell)) {
  continue;
} 
if (CellUtils.isCellVoid(nextCell, map)) {
  //...  
  continue;
} 
```

### МОДЕЛИ (MODEL)

Пакет: `Model`.  
В архитектуре MVC модели должны хранить данные и алгоритмы бизнес-логики, которые не взаимодействуют с представлением. 

**3. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. Класс только хранит значения x,y.

- Класс может быть преобразован в `record` без потери функционала.

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**4. class Directions**

+ 👍 Класс соответствует требованиям для утилитного/константного класса.  
Класс `final` и его конструктор- приватный.

- Храни координаты как сущности, а не по запчастям.

Пара значений x,y это координата.  
Направление можно рассматривать как координату относительно точки 0,0
```java
public static final int[][] FOUR_NEAR_DIRECTIONS = {
    {0, 1},
    {0, -1},
    //...
};

//ПРАВИЛЬНО:
public static final int[][] NEAR_DIRECTIONS = {
    new Coordinates(0, 1),
    new Coordinates(0, -1),
    //...
};
```

**5. class EntityMap**

- Карта должна иметь возможность быть созданной с произвольными размерами.

Не вижу причин, почему геометрию карты нужно ограничивать только квадратом
```java
public class EntityMap {
  private final int SIZE;
  //...

  public EntityMap(int size) {
    this.SIZE = size;
  }
}

//ПРАВИЛЬНО:
public class EntityMap {
  public final int width;
  public final int height;
  //...

  public EntityMap(int width, int height) {...}
  //...
}
```

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Например, здесь методы чужой ответственности:  
```java
public List<Coordinates> getVoidCells() {
```

Наверное, для проекта в целом полезно иметь метод, который создает список всех пустых координат карты.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно создавать список пустых координат.  

А что действительно нужно, так это иметь метод, который определяет, что координата в карте не занята, например:
```java
public boolean isEmpty(Coordinates coordinates)
```

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

- Никогда не возвращай null
```java
private final Map<Coordinates, Entity> map = new HashMap<>();

public Entity get(Coordinates coordinates) {
  return map.get(coordinates);  <-- вернет null если coordinates нет в map
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Вредный метод.

Не должно быть возможности вставить в карту значение null
```java
public void add(Coordinates coordinates) {
  map.put(coordinates, null);
}
```

Другой класс неправильно работает с картой- с помощью этого метода он забивает карту бесполезным мусором:
```java
public class MapCreator implements Action {
  //...

  private static void fillMap(EntityMap entityMap) {
    int mapSize = entityMap.size();
    for (int i = 0; i < mapSize; i++) {
      for (int j = 0; j < mapSize; j++) {
        entityMap.add(new Coordinates(i, j));  <-- Заполняет карту записями координата-null
      }
    }
  }
}
```

Карта не должна хранить запись координата-null, то есть пустые ячейки, иначе теряется  все преимущество, которое дает `HashMap` для хранения сущностей.

Фактически, тут `Map` становится аналогом обычного массива, только хуже, бесполезно забивая память.  
Например, есть игровая доска 100x100 и в ней одномоментно находится общим числом 150 разных существ: камней, травы, зайцев и т.д.
Тогда при нормальном хранении этих данных в мапе с помощью ключ-значение, потребуется 150 записей.
Если в этой же мапе кроме этих значимых данных хранить еще пустые сущности, то потребуется уже 10.000 записей.  
То есть, эта карта будет хранить 98.5% ненужной информации.

`Map` должна хранить только значимые записи координата-entity.  
Пустое место на карте(земля) должно кодироваться отсутствием координаты в мапе.  
То есть, если какой-то координаты нет в Map'e, то это должно значить, что в Карте по этой координате не находится никаких существ.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность
```java
public void add(Coordinates coordinates, Entity entity) {
  map.put(coordinates, entity);
}

//ПРАВИЛЬНО:
public void add(Coordinates coordinates, Entity entity) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение  
  map.put(coordinates, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public Map<Coordinates, Creature> getCellsWithCreatures() {
  //возвращает мапу креатур  
}

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда креатур

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getCellsWithBirds()`.

- В классе не хватает необходимых методов, например:
```java
public Coordinates getCoordinates(Entity entity)
```

**6. final class PathFinder**

- Нарушение SRP, OCP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритма BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```java
return switch (creature.getType()) {
  case EntityType.PREDATOR -> EntityType.HERBIVORE;
  case EntityType.HERBIVORE -> EntityType.GRASS;
  default -> throw new IllegalArgumentException("Unexpected Creature: " + creature.getType());
};
```

Нарушение OCP здесь состоит в том, что при добавлении в проект нового существа, например, Птицы, 
придется изменить код класса. То есть, класс открыт для изменений:
```java
return switch (creature.getType()) {
  case EntityType.PREDATOR -> EntityType.HERBIVORE;
  case EntityType.HERBIVORE -> EntityType.GRASS;
  case EntityType.BIRD -> EntityType.ЧЕРВЯЧОК;
  default -> throw new IllegalArgumentException("Unexpected Creature: " + creature.getType());
};
```

Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или A*.  
В случае алгоритма BFS- до точки, в которой находится существо нужного класса
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
Тогда поиск пути в нем интерпретируется не как поиск пути между двумя точками, а как поиск цели и прокладывание пути к ней.  
Например: 
```java
public class AstarPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
    //сначала находит координату цели
    //потом прокладывает путь по алгоритму A* между точкой старта и точкой цели
  }
}
```
Плюсы- унификация сигнатуры разных алгоритмов поиска для полиморфизма.  
То есть, можно сделать семейство родственных классов поиска, объединенных общим интерфейсом.

**7. abstract class Entity**

- Содержит координату
```java
public abstract class Entity {
  protected Coordinates coordinates;
  //...
}
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

- Не нужно использовать дополнительную типизацию для определения принадлежности объекта к разным типам существ
```java
public abstract OccupantType getType();
```
Используй стандартные способы определения принадлежности объектов к типам: `instanceof`, `getClass()`.

+ 👍 Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

- Зависимость модели от представления.

Существа хранят в себе поле
```java
protected EntityType entityType;
```

В свою очередь `EntityType` хранит в себе спрайты существ
```java
public enum EntityType {
  PREDATOR("🦁"), HERBIVORE("🦓"), TREE("🌴"), ROCK("🌑"), GRASS("🍀"), VOID("--");
  //...
}
```

Следовательно, каждое существо хранит в себе свое изображение и тем самым модель(а `Entity` это модель) зависит от представления.

Модель не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

**8. abstract class Creature extends Entity**

- Класс не реализует логики своего передвижения по карте.

Его метод `makeMove(Coordinates newCoordinates)` просто перемещает существо на указанную координату
```java
public void makeMove(Coordinates newCoordinates) {
  setCoordinates(newCoordinates);
}
```
В принципе, почему бы и нет, пока просто факт.

Минус такого подхода состоит в том, что `Creature` не имеет собственной воли.  
А значит ей нельзя управлять через команду типа "просто сделай ход, как посчитаешь нужным".  
И придётся, условно говоря, переставлять за неё каждую ногу.

Условный пример, Собака, которая сама управляет своим движением:
```java
class Dog {
  public void makeMove(Карта карта) {
    //собака сама переставляет лапы,
    //перемещаясь по карте по своему алгоритму
  }    
}
```
Собака, которой нужно переставлять её лапы:
```java
class Dog {
  public void переставитьПереднююЛевуюНогу(Карта карта, Координата координата) {...}
  public void переставитьПереднююПравуюНогу(Карта карта, Координата координата) {...}
  public void переставитьЗаднююЛевуюНогу(Карта карта, Координата координата) {...}
  public void переставитьЗаднююПравуюНогу(Карта карта, Координата координата) {...}
}
```
Очевидно, что первая собака проще в использовании и больше соответствует идеям ООП.

- Лишняя путаница
```java
protected boolean isAlive = true;

public boolean isDead() {
  return !isAlive;
}

//ПРАВИЛЬНО ТАК:
protected boolean isAlive = true;

public boolean isAlive() {
  return isAlive;
}

//ТАК ТОЖЕ ПРАВИЛЬНО:
protected boolean isDead = false;

public boolean isDead() {
  return isDead;
}
```

- Нарушение SRP.

Какая-то чужая логика, которая никак не характеризует сущность `Creature`.  
А характеризует то, как кто-то другой чего-то от неё хочет
```java
protected boolean canMultiply = true;

public void setCanMultiply(boolean canMultiply) {
  this.canMultiply = canMultiply;
}
```
Кто это и шо оно хочет, `Creature` без понятия.

**9. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(EntityMap map);
}
```

**10. class InitializeEntityCreator extends EntityCreator implements Action**

Нарушение SRP.
```java
public final class InitializeEntityCreator extends EntityCreator implements Action {

  @Override
  public void execute(EntityMap map) {
    //...
    Simulation.setMap(map); <-- Изменяет состояние не Карты, а класса Симуляция
  }
  //...
}
```

Экшены должны изменять только состояние карты.  
ТЗ:
```java
Action - действие, совершаемое над миром... 
Каждое действие описывается отдельным классом и совершает операции НАД КАРТОЙ. 
```

То же самое касается и других классов-экшенов, они не соответствуют этому пункту ТЗ.

- Смешивание слоёв.

Если экшены у тебя модели, а класс `Simulation` это контроллер, то изменяя в экшенах состояние Симуляции,
ты тем самым из модели управляешь контроллером.  
Модель не должна управлять контроллером.


**11. class CreatureMoveService**

Процедурный код, это вообще не читаемо.  
Класс содержит логику движения существ, причем, для каждого типа существ имеются отдельные методы.  

Лучше надели мозгами саму Креатуру и её наследников- пусть они сами управляют своим движением через собственные методы.

**12. class MoverAndRendererEachCreature implements Action**

Смешивание слоёв.

Из названия видно, что этот экшен что-то куда-то печатает.  
В данном случае, дёргая `Simulation`, он печатает в консоль.  

То есть, модель напрямую работает с представлением.  
Модели не должны ничего знать о представлении.

Печатать Карту должен контроллер.  
Суть контроллеров состоит в том, чтобы связывать модели и представления.  
В данном случае контроллер должен взять модель(Карту) и распечатать её(Представление).

**13. Множество других классов в пакете**

В пакете моделей есть большое количество других классов, но рассматривать каждый из них я не вижу смысла.  
Во-первых, их просто чересчур много и их количество не соответствует явленным возможностям программы.  
Попросту говоря, классов много, а программа делает мало.

Во-вторых недостатки этих классов примерно те же: спагетти-код, нарушение SOLID и процедурный подход при решении задач.

Отмечу только, что сервисы здесь не соответствуют критериям сервисов.  
Здесь сервисы на самом деле являются утилитами- то есть, состоят из статических методов.

### ПРЕДСТАВЛЕНИЕ (VIEW)

Пакет: `View`.  
В архитектуре MVC вью должны распечатывать данные, в том числе модели.  
Также вью должны принимать ввод от юзера.  
Вью не должны ничего знать про контроллеры.

**14. interface Renderer**

👍 Интерфейс рендерера это хорошо.

Теперь можно делать разные рендереры для разных визуальных сред(консоль, интерфейс виндовс, http, матричный принтер etc)  
и разного отображения информации(цветной, черно-белый и пр.)
```java
public interface Renderer {
  void render(EntityMap map);
}
```

**15. class ConsoleRenderer implements Renderer**

- Спрайты существ должны храниться здесь, а не браться из самих существ.

### КОНТРОЛЛЕРЫ (CONTROLLER)

Пакет: `Controller`.  
Контроллеры должны знать всё про слои view и model.  
Задача контроллеров состоит в том, чтобы связывать модели и вью в единую программу.  
Контроллер должен быть тонким.

**16. class ConsoleInput**

- Нечитаемая процедурщина.

- Консольный ввод это не контроллер, а представление.

**17. class Simulation**

- Этот класс не должен быть утилитой.

Сейчас все его поля и методы- `static`.  
Поэтому класс не имеет отношения не только к архитектуре MVC, но и к ООП в целом.

- Неправильное использование потоков в проекте.

Внутри класса `Simulation` не должно быть никакой работы с потоками
```java
public static void startSimulation(RenderMode renderMode) {
  Thread simulationThread = new Thread(() -> {
    isRunning = true;
    shouldStop = false;

      while (isRunning && !shouldStop) {
        renderMode.run(1);
      }
      isRunning = false;
  });

  simulationThread.setDaemon(false);
  simulationThread.start();
}
```

Этот класс не должен управлять потоками.  
Наоборот, какие-то потоки должны управлять этим классом и дергать за его публичные методы, которые указаны в ТЗ:
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

Это должен быть такой класс, который можно будет не только запускать из потоков.  
Но и запустить без потоков просто в бесконечном цикле и он должен(бесконечно) работать:
```java
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);
    simulation.startSimulation(); //бесконечная симуляция без потоков 
  }
}
```

Для этого класс должен выглядеть примерно так:
```java
public class Simulation {
  //...

  public Simulation(...) {
    //выполняет все initActions
  }

  public void nextTurn() {
    //выполняет все turnActions
    //печатает карту
    //печатает сопроводительную информацию: n.хода и прочее
  }

  public void startSimulation() {
    isRunning = true;
    while(isRunning) {
      nextTurn();
      sleep();
    }
  }

  private void sleep() {
    try {
      Thread.sleep(SLEEP_TIME);
    } catch (InterruptedException e) {...}
  }

  public void pauseSimulation() {
    //останавливает бесконечный цикл
  }
}
```

Когда мы добавляем в программу потоки, то поток должен принимать команды от юзера и дергать `Simulation` за ее методы:   
`nextTurn()`, `startSimulation()` и `pauseSimulation()`.

**18. Main**, содержит точку входа `main()`.

- Отсутствует Dependency injection.

Main должен сконфигурировать зависимости и запустить программу. Особенно, если дело касается MVC.  
Это дает гибкость- не меняя код в других классах, можно будет делать разные конфигурации-main'ы.

Сейчас никакой гибкости нет, потому что `Dependency injection` здесь отсутствует в принципе
```java
class Main {
  public static void main(String[] args) {
    App app = new App();
    app.start();
  }
}
```

В MVC программе это должно выглядеть примерно так:
```java
class FirstMain {
  
  //Симуляция без потоков, рендерер- эмоджи   
  public static void main(String[] args) {

    GameMap gameMap = new GameMap(10, 10);
    Renderer renderer = new EmodjiRenderer();
    Simulation simulation = new Simulation(gameMap, renderer);
    simulation.start();
  }
}

class SecondMain {
  
  //Симуляция без потоков, рендерер- текстовый(буквы)   
  public static void main(String[] args) {

    GameMap gameMap = new GameMap(20, 12);
    Renderer renderer = new TextRenderer();
    Simulation simulation = new Simulation(gameMap, renderer);
    simulation.start();
  }
}
```

Если нужно логику создания контроллера `Simulation` инкапсулировать в отдельном классе, тогда так:
```java
//❌️ НЕПРАВИЛЬНО:
class ThirdMain {
  public static void main(String[] args) {
    App app = new App();
    app.start();
  }
}

//✅ ПРАВИЛЬНО:
class Main {
  public static void main(String[] args) {
    Simulation simulation = SimulationFactory.get();
    simulation.start();
  }
}
```

Если симуляция работает в потоке "старт/стоп", должно быть примерно так:
```java
class SuperMain {
  
  //Симуляция c потоком "старт/стоп"
  public static void main(String[] args) {

    GameMap gameMap = new GameMap(15, 15);
    Renderer renderer = new EmodjiRenderer();
    Simulation simulation = new Simulation(gameMap, renderer);
    SimulationManager manager = new SimulationManager(simulation);  //поток с командами старт/стоп
    manager.start();
  }
}
```
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

Программа сделана частично- не реализована возможность сделать паузу во время работы симуляции.  
Для второго проекта важно поработать с потоками. Для этого нужно придумать, как делать паузу/пуск.

Не видно понимания сути MVC: ответственности слоёв перемешаны, отсутствует `Dependency injection`, контроллеры сделаны утилитами.  
В коде часто встречаются процедурные приёмы.

**Делай только указанный в ТЗ функционал.**  

Дополнительные усложнения, при недостаточном умении строить архитектуру программы, приносят больше вреда, чем пользы.  
Сейчас архитектура программы рухнула под её тяжестью.  
На этапе учёбы лучше простая программа с хорошей архитектурой, чем сложная с плохой.

Убери лишнюю сложность- начальное конфигурирование размеров карты, стопитсот непонятных классов и т.д.  
Упрости поведение существ до минимума, пусть они будут просто идти к еде и есть её, а если еда закончилась- просто стоять на карте. 

Посмотреть ролики Немчинского про SOLID- по одному ролику на каждый принцип.

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.162(346)  
#ревью #симуляция 