https://github.com/fynkoR/simulation  
[распознаётся графически]

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неправильно- с поворотом на 90 градусов

2. Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim169/img0.png) 

## ХОРОШО

+ 👍 Есть пауза/пуск во время работы
+ 👍 Координаты существ не хранятся в самих существах (мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Придерживайся единообразия
```java
public int compareTo(Position o) 
public double distanceTo(Position other)

//ПРАВИЛЬНО:
public int compareTo(Position other) 
public double distanceTo(Position other)
```

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относятся.  
И вообще не употребляй венгерскую нотацию. 
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
private HashMap<Position, Entity> map;

//ПРАВИЛЬНО:
private HashMap<Position, Entity> entities;  // или positionWithEntity
```

- Хуже венгерской нотации только венгерская нотация, которая обманывает. 

Здесь в названии переменной имеется слово "arr(ay)." - то есть, массив.  
Но эта переменная не является массивом, это List
```java
List<Position> arr = new ArrayList<>();

//ПРАВИЛЬНО:
List<Position> positions = new ArrayList<>();
```

- Этот метод возвращает не ключ, он возвращает позицию.  
И возвращает он её не по `value`, а по `entity`
```java
Position getKeyByValue(Entity entity)

//ПРАВИЛЬНО:
Position getPosition(Entity entity)
```

- Избыточно. Приписка "obj" тут ничего не даёт
```java
getEntitiesByClass(..., Class<T> objClass)
```

Принятое название в таких случаях это "clazz" или "type"
```java
getEntitiesBy(..., Class<T> clazz)
getEntitiesBy(..., Class<T> type)
```

- Избыточно.  

Названия методов всегда рассматриваются в контексте их классов, поэтому тавтология не нужна 
```java
public class MoveCounter {
  public void incrementCounter() {...}
}

//ПРАВИЛЬНО:
public class MoveCounter {
  public void increment() {...}
}
```

- Геттер должен возвращать только одноименное поле.

Либо так
```java
private int counter;

public String getCounter() {
  return "Текущий ход симуляции: " + counter;
}

//ПРАВИЛЬНО ТАК:
private int counter;

public int getCounter() {
  return counter;
}

//ТАК ТОЖЕ ПРАВИЛЬНО:
private int counter;

public String getCounterMessage() {
  return "Текущий ход симуляции: " + counter;
}
```

- В Java всё- объекты, поэтому не называй свои кастомные пакеты, классы, поля таким именем
```java
package entity.objects;

//ПРАВИЛЬНО:
package entity.basic;
```

- Константы должны быть `static final`
```java
private final int SPEED = 2;

//ПРАВИЛЬНО:
private static final int SPEED = 2;
```

- Класс не может называться на "able"- так называют только интерфейсы
```java
class SimulationRunnable implements Runnable 

//ПРАВИЛЬНО:
class SimulationRunner implements Runnable 
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```java
HashMap<Position, Entity> map;
HashMap<Position, Entity> getMap()

//ПРАВИЛЬНО:
Map<Position, Entity> entities;
Map<Position, Entity> getMap()
```
Общее правило: ArrayList нужно использовать через List, HashMap через Map, HashSet через Set и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**3. class Position implements Comparable<Position>**

+ 👍 Нет ничего лишнего, это хорошо. Класс только хранит значения x,y. 

- Поля x,y тут должны быть final.

При прочих равных простые делай классы `immutable`.
```java
"Классы должны быть неизменяемыми, если только нет очень важной причины, чтобы сделать их изменяемыми",
"Вы всегда должны делать объекты с небольшими значениями, такими как PhoneNumber или Complex, неизменяемыми" - Блох. 
```
*Блох "Java. Эффективное программирование", изд.3, гл.4.3.*

Собственно, ты сейчас этот класс и используешь как неизменяемый- его сеттеры нигде не вызываются. 

- Класс может быть преобразован в `record` без потери функционала.

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**4. class GameMap**

- Необоснованные ограничения формы карты.

Не вижу причин для ограничивания карты формой квадрата
```java
public class GameMap {
  //...

  public GameMap(int size) {...}
  //...
}

//ПРАВИЛЬНО:
public class GameMap {
  //...

  public GameMap(int width, int height) {...}
  //...
}
```
Ограничивать игровое поле какой-то формой можно только тогда, когда это определяется устоявшимися правилами игры.  
Например, шашки или "Крестики-нолики". 

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Вернуть список соседних координат  
🔸 Сделать ход существами   

Наверное, для проекта в целом полезно иметь метод, который возвращает все соседние координаты.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно искать соседние координаты.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

- Метод совершения хода в карте- нарушение SRP.

Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
Карта карта = new Карта(100, 100);
Заяц заяц = new Заяц();
карта.addEntity(new Координата(0, 0), заяц);
карта.moveEntity(new Координата(99, 99), заяц);

/* class Карта */
public void moveEntity(Position position, Creature creature) {
  map.remove(getKeyByValue(creature));
  map.put(position, creature);
  RenderMap.render(this);
}
```
Должна ли карта учитывать правила логистики зайцев? Если да, то как?  
Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Нарушение SRP, зависимость модели от представления. 

Модель(а это модель) не должна ничего печатать в консоль.  
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду. 
И конкретное представление себя в этой среде или какое-либо сообщение о себе в этой среде- в данном случае в консоли.
В других средах(напр. Андроид) эту модель нельзя будет использовать- там ее откажутся компилировать потому что не поддерживается консольный вывод информации
```java
public void addEntity(Position position, Entity entity) {
  //...
  System.out.println("Данная клетка занята !");  <-- ПЕЧАТАЕТ В КОНСОЛЬ
}

public void moveEntity(Position position, Creature creature) {
  //...
  RenderMap.render(this);  <-- ПЕЧАТАЕТ В КОНСОЛЬ
}
```

- Дублирующиеся методы
```java
public Entity findEntity(Position position)
public Entity getEntityByPostion(Position position)
```

- Никогда не возвращай null
```java
public Position getKeyByValue(Entity entity) {
  //...
  return null;
}
```

Тут тоже метод возвращает null, хоть это и не так очевидно
```java
private HashMap<Position, Entity> map;
 
public Entity findEntity(Position position) {
  return map.get(position);  <-- вернет null если position нет в map
}
```

Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность  
```java
public Entity findEntity(Position position) {
  return map.get(position);  
}

//ПРАВИЛЬНО:
public Entity findEntity(Position position) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение 
  return map.get(position);  
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

+ 👍 Хороший метод, пусть будет
```java
public <T extends Entity> List<Position> getEntitiesByClass(Entity entity, Class<T> objClass)
```

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```java
private HashMap<Position, Entity> map;

public HashMap<Position, Entity> getMap() {
  return map;
}
```

Чеклист ТЗ:
```java
Проблемы и ошибки в коде:
...
Недостаточная инкапсуляция - “протекающее” наружу внутреннее устройство класса Map (например, прямой доступ к коллекции ячеек) 
вместо набора методов с говорящими названиями (AddEntity, RemoveEntity, и так далее)
```

В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```java
Карта карта = new Карта();
<заселить карту существами>
карта.getMap().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

В таких случаях нужно возвращать не оригинал объекта, а его копию: 
```java
public Map<Position, Entity> getMap() {
  return new HashMap(map);
}
```

- Неочевидное поведение, нарушение правила одной операции.  

Метод не просто возвращает список координат, но и предварительно сортирует его
```java
public <T extends Entity> List<Position> getEntitiesByClass(Entity entity, Class<T> objClass) {
  //...
  if (position != null) {
    entities.sort(Comparator.comparingDouble(p -> p.distanceTo(position))); <-- Сортирует список перед тем, как его выдать
  }
  return entities;
}
```
Это можно интерпретировать как нарушение правила одной операции.  

Откуда вообще карта знает, что клиенту нужен не просто список координат, но именно отсортированный список координат?  
Карта этого знать не должна- это её вообще не касается. Это ответственность более общей игровой логики.

*"ЧК", гл.3, "Правило одной операции", "Один уровень абстракции"*

**5. class MoveCounter**

- Нарушение SRP, зависимость модели от представления.  

Конкретный язык текстового сообщения ограничивает использование этого класса-модели только русскоязычной программой 
```java
public class MoveCounter {
  private int counter;

  public MoveCounter() {
    counter = 0;
  }

  public void incrementCounter() {
    this.counter++;
  }

  public String getCounter() {
    return "Текущий ход симуляции: " + counter;
  }
}

//ПРАВИЛЬНО:
public class MoveCounter {
  private int counter;

  public MoveCounter() {
    counter = 0;
  }

  public void incrementCounter() {
    this.counter++;
  }

  public int getCounter() {
    return counter;
  }
}
```

- Класс знает слишком много.

Откуда класс вообще знает про какую-то симуляцию?
```java
return "Текущий ход симуляции: " + counter;
```

Класс называется "Счётчик ходов".  
Бог с ним, что согласно названия он счетчик именно ходов, хотя может считать что угодно, хоть селёдку поштучно.  

Но почему он знает про симуляцию?  
Это просто счетчик, который можно использовать для подсчёта шагов в шагоходе, партии в шахматах или где-то ещё.  
Внутри этого класса я не вижу какой-то специфической логики именно для Симуляции.

- Если убрать из класса нарушение SRP, то получится класс, который не делает ничего больше, чем обычный int-примитив:
```java
public class Counter {
  private int counter;

  public void increment() {
    this.counter++;
  }

  public int getCounter() {
    return counter;
  }
}

Counter counter = new Counter();
counter.increment();

//ТО ЖЕ САМОЕ ЧЕРЕЗ ПРИМИТИВ:
int counter = 0;
counter++;
```

Если такой класс решает какую-то осознанную проблему, тогда он имеет право на существование.  
Но пока я не вижу, что даст проекту эта новая абстракция.  

**6. class PathFinder**

- Нейминг.

Названия методов должны быть глаголом в настоящем времени в повелительном наклонении.  
"BFS" это существительное- название алгоритма
```java
public class PathFinder {

  public static List<Position> bfs(....) {...}
}

//ПРАВИЛЬНОЕ:
public class BfsPathFinder {

  public static List<Position> find(....) {...}
}
```

- Соблюдай требования утилитных классов. 

Это утилитный класс, он содержит только статические методы.  
Утилитные классы должны быть `final` и иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.

*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

- Сигнатура метода поиска не соответствует алгоритму BFS:
```java
public static List<Position> bfs(GameMap gameMap, Position start, Position target) 
```

Поиск по алгоритму BFS должен просто искать путь от точки старта до точки, соответствующей заданным условиям. 
В данном случае- до точки, в которой находится существо нужного класса.

Если поиск BFS получает во входящие аргументы метода координаты цели, значит часть работы поиска уже выполнил кто-то другой. 
То есть, кто-то другой уже нашел цель и передал ее координату в поиск пути.

На самом деле BFS должен сам искать цель и прокладывать к ней путь. Сигнатура метода для этого должна выглядеть примерно так
```java
List<Position> bfs(GameMap gameMap, Position start, Position target) 

//ПРАВИЛЬНО:
List<Coordinate> find(GameMap gameMap, Position start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```

**7. abstract class Entity**

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```java
private String icon;
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах (консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

+ 👍 Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**8. abstract class Creature extends Entity**

- Нарушение инкапсуляции.  

Всегда явно указывай область видимости и final, если переменная final 
```java
public abstract class Creature extends Entity {
  int speed;
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  private final int speed;
  //...
}
```

- Циклическая зависимость.

`GameMap` хранит в себе `Creature's`, а `Creature's` в себе хранят `GameMap`
```java
public abstract class Creature extends Entity {
  GameMap map;
  //...
}
```

Креатура может использовать Карту в своей работе, но она не должна её в себе хранить:
```java
public abstract class Creature extends Entity {
  GameMap map;
  //....

  public void makeMove() {
    List<Position> targets = map.getEntitiesByClass(this, this.getTargetType());
    //...
  }
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //....

  public void makeMove(GameMap gameMap) {
    List<Position> targets = gameMap.getEntitiesByClass(this, this.getTargetType());
    //...
  }
}
```

- Антипаттерн "Стрела".

Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
if (isAlive && !targets.isEmpty()) {
  if (!path.isEmpty()) {
    if (path.size() == 1) {
      for (int i = 0; i < this.getSpeed(); i++) {
        if (i < path.size() - 1) {
          //наконечник стрелы
        }
      }
    }
  }
}
```
"Стрела" значит, что метод делает несколько дел сразу и его нужно разделить на несколько вспомогательных.
```java
"Если вам нужно более трех уровней вложенности, вы все равно запутались, так что исправьте программу" - Линус Торвальдс.
```

- Антипаттерн "Лодочный якорь"
```java
//System.out.println(this + " attacked(1): " + targets.stream().toList().getFirst());
```

- Не нужно использовать флаги там, где можно обойтись методом
```java
int hp;
boolean isAlive;

//ПРАВИЛЬНО:
private int hp;

public boolean isAlive() {
  return hp > 0;    
}
```

Флаги плохи тем, что требуют к себе больше внимания в коде- их нужно где-то поднимать, где-то снимать.

- Нарушение инкапсуляции. 

Публичными должны быть только те методы, которые предназначены для использования клиентом.  
Вспомогательные методы, которые используются только в самом классе или его потомках, должны быть private/protected
```java
public abstract void attack(Position position, int damage);
abstract Class<? extends Entity> getTargetType();

//ПРАВИЛЬНО:
protected abstract void attack(Position position, int damage);
protected abstract Class<? extends Entity> getTargetType();
```
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

**9. class Herbivore extends Creature**

- Не нужно переопределять метод, если в нём нет нового поведения по сравнению с поведением предка
```java
@Override
public void makeMove() {  <-- ЛИШНИЙ МЕТОД В КЛАССЕ
  super.makeMove();
}
```

- Используй константы
```java
private final int DAMAGE = 2;
//...

@Override
Class<? extends Entity> getTargetType() {
  return Grass.class;
}

//ПРАВИЛЬНО:
private static final int DAMAGE = 2;
private static final Class<? extends Entity> TARGET_TYPE = Grass.class;
//...

@Override
Class<? extends Entity> getTargetType() {
  return TARGET_TYPE;
}
```

**10. abstract class Action**

- Не абстрактные методы не должны быть пустыми- такая практика провоцирует создание неправильной иерархии классов
```java
public abstract class Action {
  public void perform(GameMap map) {
  }
}

//ПРАВИЛЬНО:
public abstract class Action {
  public abstract void perform(GameMap map);
}
```

+ 👍 К функционалу класса претензий нет, всё ок.

**11. abstract class SpawnAction extends Action**

- Класс не соответствует описанию паттерна "Command".

Сейчас в `SpawnAction` много публичных методов
```java
public abstract class SpawnAction extends Action {
  public Random random;

  @Override
  public void perform(GameMap map) {...}  <-- ТОЛЬКО ЭТОТ МЕТОД ДОЛЖЕН БЫТЬ ПУБЛИЧНЫМ

  public Position getEmptyRandomPosition(GameMap map) {...}
  public abstract int getCount();
  public abstract Entity createEntity();
}
```

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.  
В каждом экшене должен быть только ОДИН публичный метод(не публичных может быть сколько угодно).  
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

По этой же причине все наследники `SpawnAction` тоже не являются экшенами из ТЗ. 

**12. class MoveCreaturesAction extends Action**

+ 👍 Норм.

**13. class RenderMap**

- Нарушение SRP.

Рендерер должен только распечатать карту, больше ничего.  
Рендерер не должен делать паузу- это не относится к проблематике распечатывания карты
```java
try {
  Thread.sleep(1000);
} catch (InterruptedException exception) {
  exception.getStackTrace();
}
```

- Спрайты существ должны храниться здесь, а не браться из самих существ.

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < map.getSize(); i++) {
  for (int j = 0; j < map.getSize(); j++) {
    Position position = new Position(i, j);
  }
}

//ЛУЧШЕ:
for (int x = 0; x < map.getSize(); x++) {
  for (int y = 0; y < map.getSize(); y++) {
    Position position = new Position(x, y);
  }
}
```

- Карта распечатывается неправильно
```java
public class MainTest {
  static void main() {
    GameMap gameMap = new GameMap(6);

    int x = 0;
    int y = 5;

    gameMap.addEntity(new Position(x, y), new Tree("\uD83C\uDF33"));

    RenderMap.render(gameMap);
  }
}

//РЕЗУЛЬТАТ:
** ** ** ** ** 🌳  
** ** ** ** ** **  
** ** ** ** ** **  
** ** ** ** ** **  
** ** ** ** ** **  
** ** ** ** ** **  
```

**14. class Simulation**

+ 👍 Принимает достаточное количество зависимостей в конструктор.

- Нарушение паттерна GRASP "Creator"(Создатель)

Здесь класс принимает в конструктор зависимость, которую должен создавать самостоятельно:
```java
public Simulation(GameMap map, MoveCounter moveCounter) {...}

//ПРАВИЛЬНО:
public Simulation(GameMap map) {
  //...
  this.moveCounter = new MoveCounter();
}
```
Creator гласит, что создавать объект должен тот, кто его использует. 

Рассмотрим, что это значит.

В данном случае `Simulation` принимает в конструктор `GameMap`- это правильно.  
Потому что карта может быть создана разных размеров и Simulation не знает, какая именно карта нужна в этот раз.  
Поэтому ее создает клиент, а потом инжектит в этот класс.

Но принимать в конструктор `MoveCounter`- уже неправильно.  
Потому что `MoveCounter` это конкретный класс, а не интерфейс и этот класс не параметризируется. 

Экземпляр `MoveCounter` всегда одинаковый.  
Поэтому, согласно паттерну Creator, объект `MoveCounter` *здесь* должен создавать сам класс Simulation.

Технически, можно создать наследника `MoveCounter`, переопределить в нем методы и передавать в конструктор Simulation.  
Но если программист хочет использовать в своем классе другие классы через полиморфизм, то он должен обозначить свои намерения более явно.  
Например, передавать в конструктор интерфейс или абстрактный класс.

- Сложные правила пользования классом.

Если в классе нужно инициализировать значения при его создании, то делай это в конструкторе, а не в публичном инициализаторе
```java
Simulation simulation = new Simulation(map, moveCounter);
simulation.createInitActions();

public class Simulation {
  //...

  public Simulation(GameMap map, MoveCounter moveCounter) {
    this.map = map;
    this.moveCounter = moveCounter;
  }


  public void createInitActions() {
    initActions = new ArrayList<>(List.of(
        new GrassSpawnAction(map.getSize()),
        new RockSpawnAction(map.getSize()),
        //...
    ));
  }
  //...
}

//ПРАВИЛЬНО:
Simulation simulation = new Simulation(map, moveCounter);

public class Simulation {
  //...

  public Simulation(GameMap map, MoveCounter moveCounter) {
    this.map = map;
    this.moveCounter = moveCounter;
    createInitActions();
  }

  private void createInitActions() {
    initActions = new ArrayList<>(List.of(
        new GrassSpawnAction(map.getSize()),
        new RockSpawnAction(map.getSize()),
        //...
    ));
  }
  //...
}
```

- Класс не соответствует ТЗ
```java
public class Simulation {
  private final GameMap map;
  private final MoveCounter moveCounter;
  private List<Action> initActions;
  private List<Action> turnActions;

  public Simulation(GameMap map, MoveCounter moveCounter) {
    this.map = map;
    this.moveCounter = moveCounter;
  }

  public void nextTurn() {...}

  public void createInitActions() {...}

  public void createTurnActions() {...}
}
```

В этом классе нет указанных для него публичных методов 
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

Сейчас этот класс похож на гибрид- он наполовину структура данных, наполовину объект.  
От объекта в нем только поведение `nextTurn()`. В остальном это структура, которая просто хранит в себе поля и отдаёт их клиенту.  
*"Чистый код", гл.6, "Гибриды"*

Сделай этот класс по ТЗ, оставь в нём только три публичных метода, за которые этот класс должен дёргать клиент:
```java
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

- Метод нарушает ТЗ. Я не вижу, чтобы он рендерил ход
```java
//ТЗ: nextTurn() - просимулировать и отрендерить один ход
public void nextTurn() {
  moveCounter.incrementCounter();
  System.out.println(moveCounter.getCounter());
  for (Action action : turnActions) {
    action.perform(map);
  }
}
```

**15. class SimulationRunnable implements Runnable**

Нужно перераспределить код между этим классом, `Simulation` и `Main`.

В этом классе нужно оставить только работу с потоками.  
Класс должен принимать команды от юзера и вызывать методы объекта `Simulation`: запустить бесконечную симуляцию, пауза, сделать один ход.

Поэтому класс в себя должен принимать готовый объект Simulation, больше ничего:
```java
public class SimulationRunnable implements Runnable {
  //...

  public SimulationRunnable(GameMap map, List<Action> turnActions, MoveCounter moveCounter) {
    this.map = map;
    //...
  }
  //...
}

//ПРАВИЛЬНО:
public class SimulationRunner implements Runnable {
  private final Simulation simulation;

  public SimulationRunnable(Simulation simulation) {
    this.simulation = simulation;
  }

  //...
}
```

**16. class Main**, содержит точку входа main

- Нарушение SRP.

Main должен только сконфигурировать зависимости и запустить программу.  
Управлять работой программы этот класс не должен.  

Сейчас `Main` не только конструирует систему и запускает её, но и управляет ходом её работы.

Код в этом мейне должен выглядеть примерно так:
```java
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    Simulation simulation = new Simulation(gameMap);
    
    SimulationRunner runner = new SimulationRunner(simulation);
    runner.start();
  }
}
```

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

Подробнее разберись с Action'ми.  

Посмотреть ролики Немчинского про SOLID- по одному ролику на каждый принцип.

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.169(364)  
#ревью #симуляция 