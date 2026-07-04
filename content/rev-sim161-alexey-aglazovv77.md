https://github.com/aglazovv77-svg/Simulation  
[Alexey]

Есть много над чем поработать.

## ХОРОШО

+ 👍 Есть пауза/пуск во время работы
+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Подсветка ходящих существ  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim112/img0.png) 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Без необходимости не сокращай названия, особенно названия классов
```java
public class Coords {

  private final Integer row;
  private final Integer col;
  //...
}

//ПРАВИЛЬНО:
public class Coordinates {

  private final Integer row;
  private final Integer column;
  //...
}
```
Сокращать имена было круто при Иване Грозном, когда IDE работали в текстовом режиме 80x25 символов.

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относится.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
Coords getRandomCoords(Set<Coords> set)

//ПРАВИЛЬНО:
Coords getRandomCoords(Set<Coords> allCoordinates)
```

- Одной концепции- одно название
```java
public Entity getEntity(Coords coords) {...}
public void highlight(Coords cell, String type) {...}

//ПРАВИЛЬНО:
public Entity getEntity(Coords coords) {...}
public void highlight(Coords coords, String type) {...}
```

- Избыточное уточнение
```java
public class PathFinder {
  public Coords findPath(...);
}

//ПРАВИЛЬНО:
public class PathFinder {
  public Coords find(...);
}
```

- В яве названия интерфейсов не пишут с префиксом "I"
```java
interface IActionEngine
interface IRenderer
interface IInputHandler
etc
```
Так требуется делать в других языках, например, в C#.    
В яве названия интерфейсов пишут либо с постфиксом "able", например Runnable.  
Либо просто существительное, например List.

- Название обманывает.

Метод не совершает ход по направлению к цели, хотя метод буквально так и называется
```java
// получение первого шага к цели
Coords moveTowardsTarget(...)

//ПРАВИЛЬНО:
Coords getStep(...)
```

Название метода должно быть глаголом в настоящем времени в повелительном наклонении.  
Название должно соответствовать правилам английского языка: сначала глагол, потом дополнение: `глаголДополнение()`.    

- Название обманывает.

Этот метод не делает ход, он возвращает какую-то координату
```java
abstract Coords makeMove(Set<Coords> set);
```

- Не используй слова в экзотических смыслах.

Ну *сириусли*, многие ли знают, что вторым значением слова "close" является "близко, рядом"?
```java
Coords findClosestEntityByClass(Set<Coords> availableCells, Class<? extends Entity> targetClass)

//ПРАВИЛЬНО:
Coords findNearestEntityBy(Set<Coords> availableCells, Class<? extends Entity> target)
```

- Название путает.

- Не называй метод "run" если он не переопределяет `Runnable.run()`- название этого метода ассоциируется с этим стандартным интерфейсом и применением потоков.  
Здесь этот метод не имеет отношения к `Runnable.run()`
```java
public class Simulation {
  public void run() {...}
}

//ПРАВИЛЬНО:
start(){...} //go(), execute(), perform(), поехали() 
```

- Это не конфиг, это Простая Фабрика
```java
public class ApplicationConfig {
  public static Simulation createSimulation() {...}
}

//ПРАВИЛЬНО:
public class SimulationFactory {
  public static Simulation create() {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Комментарии**

Комментарии здесь в основном не несут полезной нагрузки, а констатируют очевидное.  
Например, просто дублируют на русском названия методов:
```java
// шаг сдвига
public Coords shift(CoordsShift shift) {...} 

// Удаляет сущность
public void removeEntity(Coords coords) {...}
```
Кажется, ты считаешь, что чем больше каментов, тем лучше. На самом деле всё наоборот.

Когда в проекте много каментов, это плохо- пользы от них практически нет, они только забивают пространство и мешают читать код.  
В идеале, комментариев вообще не должно быть(кроме `TODO`), код должен объяснять сам себя через правильный нейминг и лаконичный код.  
*Мартин, "ЧК", гл.4*

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки.  
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```java
if ((r < 1) || (r > GameConfig.WORLD_HEIGHT)) return false;

//ПРАВИЛЬНО:
if ((r < 1) || (r > GameConfig.WORLD_HEIGHT)) {
  return false;
}
```
*"Oracle Java code conventions"* 

**3. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется**  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без else будет выглядеть читабельней
```java
if (inputHandler.isQuit(command)) {
  //...
  return;
} else if (inputHandler.isResume(command)) {
  running = true;
  System.out.println("Симуляция продолжена!");
}

//ПРАВИЛЬНО:
if (inputHandler.isQuit(command)) {
  //...
  return;
} 
if (inputHandler.isResume(command)) {
  running = true;
  System.out.println("Симуляция продолжена!");
}
```

**4. Форматирование**

- Избыточно
```java
System.out.print("Нажмите [R] для продолжения\n");

//ПРАВИЛЬНО:
System.out.println("Нажмите [R] для продолжения");
```

**5. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```java
System.out.print("""
    Вас приветствует мир Симуляции!\s
    Нажмите [N] для рендеринга одного хода,
    либо [I] для старта бесконечного цикла!\s
    """);
}

System.out.print("Неверная команда! Введите [N] или [I]\n");

//ПРАВИЛЬНО:
private static final String STEP = "N"; 
private static final String START = "I"; 

System.out.printf("""
    Вас приветствует мир Симуляции!
    Нажмите [%s] для рендеринга одного хода,
    либо [%s] для старта бесконечного цикла!
    """, STEP, START);
}

System.out.print("Неверная команда! Введите [%s] или [%s] %n", STEP, START);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**6. class CoordsShif**

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates.  
Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```java
Coordinates shifDownRightCoordinates = new Coordinates(-1, 1);
```

**7. class Coords**

- При прочих равных, нужно использовать примитивный тип, а не класс-обертку
```java
public class Coords {

  private final Integer row;
  private final Integer col;
  //...
}

//ПРАВИЛЬНО:
public class Coordinates {

  private final int row;
  private final int column;
  //...
}
```
Для применения класса-обертки над примитивным типом данных, должна быть какая-то причина.  
Здесь использование обертки не дает ничего большего по сравнению с примитивом.  
*Блох "Java. Эффективное программирование", изд.3, гл.9.5*
```java
"Предпочитайте примитивные типы упакованным примитивным типам" - Блох.
```

- Нарушение SRP 

Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: x,y или row,column.  
Лучше даже, если этот класс будет record'ом.

Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия, тем самым нарушая SRP:  
🔸 Создает случайную координату  
🔸 Проверяет, можно ли сделать сдвиг  

+ 👍 Этот метод тут можно оставить, если хочется, он не нарушает SRP.  
Но сдвигаться он должен при взаимодействии с экземпляром того же класса
```java
public Coords shift(CoordsShift shift) {
  return new Coords(this.row + shift.rowShift, this.col + shift.colShift);
}

//ПРАВИЛЬНО:
public Coords shift(Coords shift) {...}
```

- Никогда не возвращай null
```java
public static Coords getRandomCoords(Set<Coords> set) {
  //...
  if (size == 0) return null;
  //...
  return null;
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Избыточно
```java
public static Coords getRandomCoords(Set<Coords> set) {
  //...  
  int item = new Random().nextInt(size);
  for (Coords coords : set) {
    if (i == item) return coords;
    i++;
  }
  //...
}

//ПРАВИЛЬНО: 
public static Coords getRandomCoords(Set<Coords> allCoordinates) {
  int index = new Random().nextInt(size);
  return allCoordinates.toArray()[index];
}
```

- Идеальная координата для этого и подобных ему проектов- простой record типа такого
```java
public record Coordinates (int row, int column) {
  public Coordinates shift(Coordinates shiftCoordinates) {...}    
}
```

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

+ 👍 Из хорошего в этом классе отмечу то, что он хранит только необходимые поля: row, column.  А методы да, зоопарк.

**8. class World**

- Нарушение SRP, божественный класс, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Делает первоначальную расстановку сущностей   
🔸 Возвращает подсвеченные координаты   
🔸 Возвращает случайную пустую координату  
etc

Наверное, для проекта в целом полезно иметь метод, который хранит подсвеченные координаты.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно знать, какие координаты подсвечены, а какие нет.  

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".


- Нарушение SRP, зависимость модели от представления
```java
private final Map<Coords, String> highlightedCells = new HashMap<>();

// Добавляет клетку в список подсвеченных с указанием типа подсветки
public void highlight(Coords cell, String type) {...}

// Очищает все подсветки
public void clearHighlights() {...}

// Возвращает неизменяемую карту подсвеченных клеток
public Map<Coords, String> getHighlightedCells() {...}
```

Карта это модель, она должна только хранить существа и обеспечивать базовые операции для этого: вставить, удалить и т.д.  
Карту, как модель, не должно волновать, как ее будут показывать юзеру- это ответственность тех классов, которые будут заниматься представлением.

- Карта не должна сама себя заселять существами.

Иначе она становится неуниверсальной и нельзя будет создать несколько конфигураций игры с разными начальными комбинациями существ на карте
```java
public void setupEntitiesPosition(EntityFactory entityFactory) {...}
```
Объект не должен сам себя конструировать.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Нарушение SRP, дублирование кода.

Повторяющиеся действия выноси во вспомогательные методы
```java
for (int i = 0; i < GameConfig.INITIAL_GRASS; i++) {
  Coords coords = getRandomEmptyCoords();
  setEntity(coords, entityFactory.createGrass(coords));
}

for (int i = 0; i < GameConfig.INITIAL_ROCK; i++) {
  Coords coords = getRandomEmptyCoords();
  setEntity(coords, entityFactory.createRock(coords));
}

//ПРАВИЛЬНО:
spawn(entityFactory::createGrass, GameConfig.INITIAL_GRASS);
spawn(entityFactory::createRock, GameConfig.INITIAL_ROCK);

private void spawn(Function<Coordinates, Entity> mapper, int count) {
  for (int i = 0; i < count; i++) {
    Coords coords = getRandomEmptyCoords();
    Entity entity = mapper.apply(coords);
    setEntity(coords, entity);
  }
}
```
Тема: стандартные интерфейсы java core: `Function`, `Supplier` etc.

- Никогда не возвращай null
```java
private final Map<Coords, Entity> entities = new HashMap<>();
 
public Entity getEntity(Coords coords) {
  return entities.get(coords);  <-- вернет null если coords нет в entities
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Проверяй, находится ли координата в пределах карты. 

При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что она свободна.  
А правильный ответ- ячейки с такой координатой в карте нет вообще
```java
public boolean isCellEmpty(Coords coords) {
  return !entities.containsKey(coords);
}

//ПРАВИЛЬНО:
public boolean isCellEmpty(Coords coords) {
    validate(coords);  <-- Бросает исключение если координата не находится в пределах карты
  return !entities.containsKey(coords);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

**9. class PathFinder**

+ 👍 Лайк за константу
```java
private static final List<CoordsShift> DIRECTIONS = List.of(...);
```

- Этот класс не ищет путь.

Путь это последовательность точек от точки старта до точки финиша.  
Этот класс не ищет путь. Его единственный публичный метод возвращает одиночную координату
```java
public Coords findPath(Coords start, Coords target, World world, PassabilityStrategy passabilityStrategy) {...}
```
Этот метод ищет не путь, а один шаг пути.  
Это тоже хорошо, но не то, что требуется от класса, который должен искать путь. 

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

**10. interface PassabilityStrategy**

Просто ацкий ад, удоли.

Логика движения к цели должна быть такой:
Класс поиска пути по алгоритму BFS(а заявлен именно он) должен прокладывать путь по доступным(то есть пустым) клеткам от точки старта, до точки с соответствующими условиями: 
```java
public class BfsPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

Поэтому нет необходимости в интерфейсе `PassabilityStrategy`- он не упрощает код, а делает его только сложнее и запутаннее.

**11. abstract class Entity**

- Содержит координату
```java
public abstract class Entity {
  private Coords coords;
  //...
}
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`

+ 👍 Вот таким должен быть идеальный Entity и его простые неходящие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**12. abstract class Creature extends Entity**

- Циклическая зависимость.

Карта хранит Креатур, а Креатуры хранят карту
```java
public abstract class Creature extends Entity {
  protected final World world;
  //..
}
```

Креатура может использовать карту в качестве входящего аргумента в методы, но не хранить её в качестве части себя.  
Например, можно делать вот так:
```java
class Creature {
  //...

  public void makeMove(Карта карта) {...}     
}
```

- Сложные правила пользования классом, нарушение SRP.

У существа должен быть только один (публичный) метод для совершения хода: `makeMove(...)`.  
Только этот метод другие классы должны вызывать у существа, чтобы оно сходило 
```java
public abstract class Creature extends Entity {

  // множество клеток куда можно пойти
  public Set<Coords> getAvailableMoveCells(World world) {
    //куча какого-то хода
  }

  abstract Coords makeMove(Set<Coords> set);
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  private final Class<? extends Entity> food;   
  //...

  public void makeMove(World world) {
    Coordinates current = getCoords();
    List<Coordinates> path = pathFinder.find(world, current, food);
    //пройти по пути path
    //если подошел к еде- съесть её
  }
}
```

**13. class Predator/Herbivore extends Creature**

- Нарушение DRY. Дублирование кода в методах наследников
```java

public class Predator extends Creature {
  //...

  public Coords makeMove(Set<Coords> availableCells) {

    Coords bestCoordsHerbivore = findClosestEntityByClass(availableCells, Herbivore.class);

    return moveTowardsTarget(
        bestCoordsHerbivore,
        availableCells,
        (neighbor, entity, target) ->
            neighbor.equals(target) ||
                entity == null ||
                entity instanceof Herbivore
    );
  }
}

public class Herbivore extends Creature {
  //...

  public Coords makeMove(Set<Coords> availableCells) {

    if (wasAttackedLastTurn) {
      wasAttackedLastTurn = false; // Сбрасываем флаг (один раз убежали)
      return findEscapeMove(availableCells);
    }

    Coords bestCoordsGrass = findClosestEntityByClass(availableCells, Grass.class);

    return moveTowardsTarget(
      //то же самое
      entity instanceof Grass
    );
  }
}
```

Общий код потомков выноси в методы потомков:
```java
public class Herbivore extends Creature {
  //...

  public Coords makeMove(Set<Coords> availableCells) {

    if (wasAttackedLastTurn) {
      wasAttackedLastTurn = false; // Сбрасываем флаг (один раз убежали)
      return findEscapeMove(availableCells);
    }

    return makeMove(availableCells, Grass.class);
  }
}

public abstract class Creature extends Entity {
  //...

  protected Coords makeMove(Set<Coords> availableCells, Class<? extends Entity> target) {
    Coords bestCoordsGrass = findClosestEntityByClass(availableCells, target);

    return moveTowardsTarget(
        //то же самое
        target.isInstance(entity)
    );
  }
}
```

**14. interface IActionEngine**

Нарушение ISP
```java
public interface IActionEngine {
  void initActions();

  void turnActions();

  int getTurnCount();
}
```

**15. class Actions implements IActionEngine**

- Идея `Action` здесь не осмыслена. 

Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.  
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.  
Action'ы, изложенные в ТЗ, это вариант реализации паттерна Command. 

Сейчас `Actions` у тебя это просто сборник функций, сделанный в стиле процедурного программирования
```java
public class Actions implements IActionEngine {
  public void initActions() {...}
  public void turnActions() {...}
  public int getTurnCount() {...}
}
```

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.  
В каждом экшене должен быть только один публичный метод(не публичных может быть сколько угодно).  
Это вариация паттерна Command- экшены должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
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

Детально анализировать `class Actions` сейчас не имеет смысла.  
Сначала нужно привести его к тому виду, который указан в ТЗ:

"Action - действие, совершаемое над миром. Например - сходить всеми существами.  
Это действие итерировало бы существ и вызывало каждому makeMove().  
КАЖДОЕ действие описывается ОТДЕЛЬНЫМ КЛАССОМ и совершает операции над картой."

**16. Пакет logic**

Кроме класса `Actions` в этом пакете много еще каких-то непонятных классов, которые содержат обрывки чужих ответственностей.  
Классы эти сделаны в стиле процедурного программирования и являются не объектами в понимании ООП, а контейнерами для функций. 

Например, класс `class MovementHandler`.  
Это класс, который выполняет перемещение существ, причем дублируя код для передвижения Хищника и Травоядного.  

На самом деле код ВЫПОЛНЕНИЯ передвижения Хищника и Травоядного должен находиться в их(`Creature`, `Herbivore`, `Predator`)  методах `void makeMove(World world)`. 

Экшен выполнения хода должен выглядеть примерно так:
```java
class MoveAction реализует Action {

  @Override
  public void execute(Карта карта) {
    List<Creature> creatures = getCreatures(карта);
    for(Creature creature: creatures) {
      creature.makeMove(карта);  //даёт пинка чтобы креатура побежала
    }
  }

  private static List<Creature> getCreatures(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

Этот экшен должен только инициировать движение креатур(давать им пинка), но не должен сам осуществлять процесс перемещения(переставлять им ноги). 

**17. interface IRenderer**

+ 👍 Интерфейс рендерера это хорошо.

Теперь можно делать разные рендереры для разных визуальных сред(консоль, интерфейс виндовс, http, матричный принтер etc)  
и разного отображения информации(цветной, черно-белый и пр.)
```java
public interface IRenderer {
  void render();
}
```

**18. Использование конфигурационных классов**

В программе используются конфигурационные/константные классы.  
Используются они неправильно- классы напрямую лезут в эти конфиги, хотя должны только получать из них данные в свой конструктор.  

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

Конфиги это дополнительное усложнение проекта, которое ты решил сделать добровольно.

**НЕ УСЛОЖНЯЙ ПРОЕКТЫ, ДЕЛАЙ ИХ СТРОГО ПО ТЗ.**  
Пока не совсем умеешь строить грамотную архитектуру программ, усложнения заданий принесут больше вреда, чем пользы.

Например, возьмём фабрику существ.  
Фабрика неправильно использует конфиг- лезет в него напрямую.  
Даём пинка конфигу: пока-пока! Код стал чище:
```java
//БЫЛО:
public class EntityFactory {

  private final PathFinder pathFinder;
  private final World world;

  public EntityFactory(PathFinder pathFinder, World world) {
    this.pathFinder = pathFinder;
    this.world = world;
  }

  public Tree createTree(Coords coords) {...}
  public Rock createRock(Coords coords) {...}
  public Grass createGrass(Coords coords) {...}
  public Herbivore createHerbivore(Coords coords) {...}

  public Predator createPredator(Coords coords) {
    return  new Predator(coords, GameConfig.PREDATOR_HP, GameConfig.PREDATOR_SPEED, pathFinder, world, GameConfig.PREDATOR_ATTACK);
  }
}

//СТАНЕТ:
public class EntityFactory {
  private static final int PREDATOR_HP = 3;
  private static final int PREDATOR_SPEED = 2;
  private static final int PREDATOR_ATTACK = 1;
  //...

  private final PathFinder pathFinder;
  private final World world;

  public EntityFactory(PathFinder pathFinder, World world) {
    this.pathFinder = pathFinder;
    this.world = world;
  }

  public Tree createTree(Coords coords) {...}
  public Rock createRock(Coords coords) {...}
  public Grass createGrass(Coords coords) {...}
  public Herbivore createHerbivore(Coords coords) {...}

  public Predator createPredator(Coords coords) {
    return  new Predator(coords, PREDATOR_HP, PREDATOR_SPEED, pathFinder, world, PREDATOR_ATTACK);
  }
}
```

С конфигом прикольнее? Возможно.  
Но неправильное использование конфига делает код более запутанным, а архитектуру еще хуже.  
И разгрести архитектуру программы до нормального состояния становится намного сложнее.

Для начала удали конфигурационные/константные классы и перенеси константы в те классы, которые используют эти константы.

**19. class Simulation**

- Не выкидывай члены из цикла `for`.

В java можно делать разные извращения с циклом `for`. Но не нужно этого делать.  
Если хочется выкинуть один-два-три члена из `for`, то просто используй вместо него другой вид цикла
```java
for (; ; ) {...}

//ПРАВИЛЬНО:
while(true) {...} //вечный цикл
```

- Нарушение SRP.

Этот класс не должен вести с юзером диалоги "Старт/Стоп".  
Этот класс должен иметь три публичных метода, за которые его будет дёргать КЛИЕНТ, а не он сам себя.  
ТЗ:
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

Это должен быть такой класс, который может работать без команд старт/стоп в бесконечном режиме:
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

Чтобы добавить в проект управление командами старт/стоп, то нужно ввести дополнительный класс С ПОТОКАМИ.  
Этот класс должен в одном потоке принимать от юзера команды, а в другом потоке дергать `Simulation` за её публичные методы:   
`nextTurn()`, `startSimulation()` и `pauseSimulation()`.

Тогда `Main` с такой конфигурацией должен выглядеть примерно так:
```java
public class MainWithThreads {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);
    SimulationManager simulationManager = new SimulationManager(simulation);  //в этом классе многопоточность
    simulationManager.execute(); 
  }
}
```

**20. class Main**, содержит точку входа main

+ 👍 Только создает и запускает `Simulation`, это хорошо.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

Прежде всего хочу тебя поздравить с **большим прогрессом** в программировании по сравнению с тем, как ты написал свой первый проект 😎🤝😎  
Есть еще куда стремиться, главное не останавливаться.

Делай только указанный в ТЗ функционал.  
Дополнительные усложнения, при недостаточном умении строить архитектуру программы, приносят больше вреда, чем пользы.

Внимательно читай и соблюдай ТЗ.  
Переделай по ТЗ классы Action's: посмотри ролики на ютубе про паттерн "Command", чтобы получить об экшенах-командах общее представление.

Введи в проект многопоточность.

Посмотри на ютубе видео Немчинского про SOLID- по одному ролику на каждый принцип.

n.161(340)  
#ревью #симуляция 