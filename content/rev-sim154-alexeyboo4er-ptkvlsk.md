https://github.com/ptkvlsk/Simulation  
[Алексей Boo4er]

Есть много над чем поработать. 

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Совершенно адовые вертикальные зазоры между распечаткой кадров- 50 строк.

2. В программе не реализована работа с потоками и выполнение старт/стоп во время работы симуляции.  
Хотя это требование есть в ТЗ.

## ХОРОШО

+ 👍 Карта распечатывается ровно. 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относится.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
private Map<Point, Entity> map = new HashMap<>();

//ПРАВИЛЬНО:
private Map<Point, Entity> entities = new HashMap<>(); //или pointWithEntities
```

- Название класса должно объяснять его суть.  
Этот класс не проверяет здоровье существ. Он удаляет мертвых существ из карты
```java
class CheckHealthAction implements Action 
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Первыми в классе должны стоять все поля, за ними- все методы.

Сейчас во многих классах методы и поля написаны вперемешку.  
Это сильно дизориентирует при чтении кода.  
Первым среди методов должен быть конструктор
```java
public abstract class Creature extends Entity {

  public int getSpeed() {...}
  public int getHp() {...}

  private final int speed;
  private int hp;

  public Creature(int speed, int hp, int maxHp, Point position) {...}

  public abstract void makeMove(GameMap map);
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {

  private final int speed;
  private int hp;

  public Creature(int speed, int hp, int maxHp, Point position) {...} 

  public int getSpeed() {...}
  public int getHp() {...}

  public abstract void makeMove(GameMap map);
}
```
*"Oracle Java code conventions"* 

**3. class GameMap**

- Нарушение конвенции кода. Поля должны быть вверху, методы- внизу
```java
public class GameMap {
  private final int width;
  private final int height;

  public GameMap(int width, int height) {...}

  private Map<Point, Entity> map = new HashMap<>();
  //...
}

//ПРАВИЛЬНО:
public class GameMap {
  private final int width;
  private final int height;
  private Map<Point, Entity> map = new HashMap<>();

  public GameMap(int width, int height) {...}
  //...
}
```

- Неправильная реакция на ошибки.

Есть три основных реакции на ошибки: игнорирование, исправление, падение.  
Здесь выбрано игнорирование- при получении некорректной координаты, метод просто ничего не делает
```java
public void addEntity(Point position, Entity entity) {  
  if (isCellEmpty(position)) {  <-- ИГНОРИРОВАНИЕ
    map.put(position, entity);
  }
}
```
Это потенциально может привести к глюкам.

Представь, что ты делаешь ход существом.  
Для этого нужно существо удалить со старой координаты и поместить его на новую координату.  
Но если ты в качестве новой укажешь недопустимую координату, то существо будет удалено со старого места и не помещено на новое место.  
То есть, существа будут тупо исчезать из карты без всяких причин и замечено это будет не сразу(если вообще будет).

Потому попытка сделать ход на некорректную координату является багом алгоритма.  
Этот баг не нужно маскировать, его нужно максимально быстро выявить и устранить.

Правильная реакция должна быть другой, нужно сделать падение:
```java
public void addEntity(Point position, Entity entity) {  
  validate(position);  <-- Бросает исключение если координата не находится в пределах карты
  map.put(position, entity);
}
```

- Проверяй, находится ли координата в пределах карты.

При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что она свободна.  
А правильный ответ- ячейки с такой координатой в карте нет вообще
```java
public boolean isCellEmpty(Point position) {
  return !map.containsKey(position);
}

//ПРАВИЛЬНО:
public boolean isCellEmpty(Point position) {
  validate(position);  <-- Бросает исключение если координата не находится в пределах карты
  return !map.containsKey(position);
}
```

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Herbivore> getAllHerbivores() {...}
public List<Predator> getAllPredators() {...}
public Set<Point> getAllGrassPositions() {...}

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда креатур

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getBirds()`.

- Никогда не возвращай null 
```java
private Map<Point, Entity> map = new HashMap<>();

public Entity getEntityAt(Point position) {
  return map.get(position);   <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

+ 👍 В принципе, с точки зрения SOLID, класс более-менее норм.  
Нарушения есть, но их меньше, чем бывает обычно.

**4. class PathFinder**

- Этот класс не ищет путь.

Путь это последовательность точек от точки старта до точки финиша.  
Этот класс не ищет путь. Его единственный публичный метод возвращает одиночную координату
```java
public Point findNextStep(GameMap map, Point start, Class<? extends Entity> targetType) 
```
Как с помощью этого класса искать путь, совершенно неясно.

Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритма BFS или A*.  
В случае алгоритма BFS- до точки, в которой находится существо нужного класса
```java
public class PathFinder {
  public List<Coordinates> find(Карта карта, Point start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

- Нарушение SRP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```java
private boolean isPassible(GameMap map, Point position, Class<?> targetType) {
  Entity entity = map.getEntityAt(position);
  if (entity instanceof Predator) {
    return false;
  }
  if (entity instanceof Grass) {
    return false;
  }
  //...
}
```

Чтобы найти путь по алгоритму BFS, классу поиска достаточно знать, что существо может ходить только по пустым(незанятым) ячейкам карты, 
координату начала пути и тип искомого существа.

Эту информацию поиск должен получать от клиента и не определять самостоятельно путем опроса существ:
```java
public List<Coordinates> find(
    Карта карта,  <-- ТУТ ИЩЕМ ПУТЬ ПО НЕЗАНЯТЫМ ЯЧЕЙКАМ
    Point start,  <-- НАЧИНАЕМ ПОИСК С ЭТОЙ КООРДИНАТЫ 
    Class<? extends Entity> target  <-- ТИП ИСКОМОГО СУЩЕСТВА
    ) {...}
```

- Совершенно адовое условие, что оно определяет- вообще ни разу не понятно.  
Вводи вспомогательные методы или поясняющие переменные
```java
if (nx >= 0 && nx < width && ny >= 0 && ny < height && !visited.contains(neighbor) && isPassible(map, neighbor, targetType)) {...}

//ПРАВИЛЬНО:
boolean isНазваниеКотороеВсеОбъясняет = nx >= 0 && nx < width && ny >= 0 && ny < height && !visited.contains(neighbor) && isPassible(map, neighbor, targetType);
if (isНазваниеКотороеВсеОбъясняет) {...}
```

- Используй координаты сразу как объект, а не по запчастям
```java
public Point findNextStep(GameMap map, Point start, Class<? extends Entity> targetType) {
  Point current = queue.poll();
  int[] dx = {-1, 1, 0, 0};
  int[] dy = {0, 0, -1, 1};
  for (int i = 0; i < 4; i++) { // magic
    int nx = current.x + dx[i];
    int ny = current.y + dy[i];
    Point neighbor = new Point(nx, ny);
    //...
  }
}

//ПРАВИЛЬНО:
private static final List<Point> SHIFT_POINTS = List.of(new Point(-1, 0), new Point(1, 0), ...);

public Point findNextStep(GameMap map, Point start, Class<? extends Entity> targetType) {
  Point current = queue.poll();
  for (Point shift : SHIFT_POINTS) { // больше никакой магии- просто нормальный алгоритм
    int nx = current.x + shift.x;
    int ny = current.y + shift.y;
    Point neighbor = new Point(nx, ny);
    //...
  }
}
```

- Никогда не возвращай null
```java
public Point findNextStep(GameMap map, Point start, Class<? extends Entity> targetType) {
  //...
  return null;
}
```

- Избыточно
```java
Queue<Point> queue = new LinkedList<Point>();
Set<Point> visited = new HashSet<Point>();

//ПРАВИЛЬНО:
Queue<Point> queue = new LinkedList<>();
Set<Point> visited = new HashSet<>();
```

**5. abstract class Entity и его простые наследники Tree/Rock/Grass**

+ 👍 Идеально
```java
public abstract class Entity {
}

public class Tree extends Entity {
}
```

**6. abstract class Creature extends Entity**

- Та же тема с использованием координат по запчастям
```java
protected void move(GameMap map) {
  int[] x = {-1, 1, 0, 0};
  int[] y = {0, 0, -1, 1};
  //...
}
```

- Нарушение SRP.

В этом методе куча кода, который вроде как имеет отношение к проблематике поиска пути
```java
protected void move(GameMap map) 
```

Поиск пути должен происходить в классе поиска пути.  
А здесь нужно только использовать этот поиск, примерно так:
```java
protected void move(GameMap gameMap) {
  PathFinder pathFinder = new PathFinder();

  Point currentPos = this.getPosition();
  List<Point> path = pathFinder.find(gameMap, currentPos, getTargetClass());
  //Пройти по координатам path
  //если прошел до конца пути- съесть цель
}
```

- Всегда явно указывай область видимости.  
Потому что неясно- то ли ты забыл её указать(а однажды забудешь), то ли шо
```java
abstract boolean isTarget(Entity entity);

//ПРАВИЛЬНО:
protected abstract boolean isTarget(Entity entity);
```

- Не делай заглушки
```java
protected void onNoPath(GameMap map) {
}

//ПРАВИЛЬНО:
protected abstract void onNoPath(GameMap map);
```

**7. class Herbivore extends Creature**

- Всегда явно указывай область видимости
```java
public class Herbivore extends Creature {

  boolean isTarget(Entity entity) {...}

  Class<? extends Entity> getTargetClass() {...}

  void interactWithTarget(Entity target, GameMap map, Point position) {...}
  //...
}
```

- Магическое число. Вводи константы
```java
void interactWithTarget(Entity target, GameMap map, Point position) {
  map.removeEntity(position);
  setHp(getHp() + 2);
}
```

**8. class Herbivore extends Creature**

- Нарушение LSP.

Если вместо реализации наследуемых методов делаешь заглушки, значит иерархия классов построена неправильно.  
В предке должны быть только те методы, которые должны быть реализованы у ВСЕХ потомков
```java
public abstract class Creature extends Entity {
  protected void onNoPath(GameMap map) {
  }
  //...
}

public class Herbivore extends Creature {
  @Override
  protected void onNoPath(GameMap map) {
    setHp(getHp() - 1);
  }
  //...
}

public class Predator extends Creature {
  @Override
  protected void onNoPath(GameMap map) {  <-- ЗАГЛУШКА
  }
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //Нет метода onNoPath(GameMap map)
  //...
}

public class Herbivore extends Creature {
  protected void onNoPath(GameMap map) {
    setHp(getHp() - 1);
  }
  //...
}

public class Predator extends Creature {
  //Нет метода onNoPath(GameMap map)
  //...
}
```

**9. Избыточный код в наследниках Creature**

Herbivore и Predator совершают ход совершенно одинаково
```java
public abstract class Creature extends Entity {
  //...
  public abstract void makeMove(GameMap map);

  protected void move(GameMap map) {
    //куча кода
  }
}

public class Herbivore extends Creature {
  //...
  @Override
  public void makeMove(GameMap map) {
    move(map);  <-- ОДИНАКОВО С PREDATOR
  }
}

public class Predator extends Creature {
  //...
  @Override
  public void makeMove(GameMap map) {
    move(map); <-- ОДИНАКОВО С HERBIVORE
  }
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //...
  public void makeMove(GameMap map) {
    //куча кода
  }
}

public class Herbivore extends Creature {
  //makeMove(GameMap map) тут не переопределяется- он наследуется от предка
}

public class Predator extends Creature {
  //makeMove(GameMap map) тут не переопределяется- он наследуется от предка
}
```

**10. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(GameMap map);
}
```

**11. Экшены хода**

Должен быть один класс, который ходит всеми существами, а не отдельные классы для передвижения каждого типа существ
```java
public class MoveHerbivoreAction implements Action  <-- Двигает травоядных
public class MovePredatorAction implements Action  <-- Двигает хищников

//ПРАВИЛЬНО:
public class MoveAction implements Action <-- Двигает Creature, а значит всех его наследников
```

**12. Экшены спавна**

Множество классов, которые делают одно то же- заселяют карту определенным видом существ
```java
class SpawnGrassAction implements Action
class SpawnHerbivoresAction implements Action
class SpawnPredatorsAction implements Action
and others
```
Как следствие- куча одинакового кода в этих классах.

Вместо отдельного класса на каждый вид существ, нужно сделать универсальный класс, который будет принимать в конструктор количество создаваемых существ и способ их создания.  
Если при создании существ в них (или в некоторые из них) нужно передавать координату, cпособ создания можно передавать через стандартный интерфейс Function
```java
public class SpawnAction extends Action {
  private final Function<Point, Entity> entityCreator;
  private final int count;

  public SpawnAction(Function<Point, Entity> entityCreator, int count) {
    this.entityCreator = entityCreator;
    this.amount = count;
  }

  @Override
  public void execute(Карта карта) {
    for (int i = 0; i < count; i++) {
      Point point = getRandomEmptyCoordinates(карта);
      Entity entity = entityCreator.apply(point);
      карта.вставить(entity, point);
    }
  }

  private Point getRandomEmptyCoordinates(Карта карта) {
    //находит и возвращает случайную пустую координату
  }
}
```

Использование:
```java
SpawnAction заяцSpawnAction = new SpawnAction((point) -> new Заяц(point), КОЛИЧЕСТВО_ЗАЙЦЕВ);
SpawnAction траваSpawnAction = new SpawnAction((point) -> new Трава(), КОЛИЧЕСТВО_ТРАВЫ);
заяцSpawnAction.execute(карта); 
траваSpawnAction.execute(карта); 
```

**13. Отсутствует рендерер карты**

Нарушение ТЗ
```java
Дизайн классов
Рендерер
Рендерер ответственен за визуализацию состояния поля, его отрисовку. 
```
Сейчас распечаткой занимается класс `Simulation`, что нарушает его SRP.

**14. class Simulation**

- Придерживайся единообразия
```java
public Simulation(int width, int height) {
  map = new GameMap(width, height);
  this.initActions = new ArrayList<>();
  this.turnActions = new ArrayList<>();
}

//ПРАВИЛЬНО:
public Simulation(int width, int height) {
  this.map = new GameMap(width, height);
  this.initActions = new ArrayList<>();
  this.turnActions = new ArrayList<>();
}
```

- Нарушение ТЗ, нет указанных для этого класса методов 
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга  <-- НЕТ ЭТОГО МЕТОДА
```

**15. class Main**

+ 👍 Только создает и запускает `Simulation`, это хорошо.

## ВЫВОД

В программе есть простейшие нарушения конвенции кода.  
Внимательно изучи *Oracle Java code conventions*.

Придерживайся ТЗ.  
Если в нем указано, что нужно сделать класс- делай его.  
Соблюдение ТЗ это важный навык для работы. 

Программа сделана частично- не реализована возможность сделать паузу во время работы симуляции.  
Для второго проекта важно поработать с потоками. Для этого нужно придумать, как делать паузу/пуск.

Переделать класс поиска пути, чтобы он искал путь.  
Убрать дублирование кода в экшенах.

Посмотреть ролики Немчинского про SOLID- по одному ролику на каждую букву.

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.154(327)  
#ревью #симуляция 