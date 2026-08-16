https://github.com/motr3x/simulation  
[Matvey]

Есть многое, над чем нужно вдумчиво поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Программа не запускается.

## ХОРОШО

+ 🤷 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относятся.  
И вообще не употребляй венгерскую нотацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
Map<Coordination, Entity> map = new LinkedHashMap<>();

//ПРАВИЛЬНО:
Map<Coordination, Entity> entities = new LinkedHashMap<>();
```

- Не используй синонимы для одной и той же концепции
```java
Optional<Coordination> getPosition(Entity entity) 
Coordination creatureCoordinate = ...

//ПРАВИЛЬНО:
Optional<Coordination> getCoordination(Entity entity) 
Coordination creatureCoordination = ...
```

- Здесь тоже одна и та же концепция названа двумя разными словами: поле и позиция
```java
public Optional<Coordination> getPosition(Entity entity) {...}
public boolean fieldIsEmpty(Coordination coordination) {...}
```

- В проекте нужны "координаты" а не "координации"
```java
record Coordination

//ПРАВИЛЬНО:
record Coordinates
```

- Это не сеттер.

Этот метод не устанавливает значение определенного поля.  
Он добавляет новый экземпляр в группу других 
```java
public final class GameMap {

  private final Map<Coordination, Entity> map = new LinkedHashMap<>();
  //...

  public void set(Coordination coordination, Entity entity) {
    map.put(coordination, entity);
  }
}

//ПРАВИЛЬНО:
1. void add(Coordination coordination, Entity entity)
2. void put(Coordination coordination, Entity entity)
```

- Не называй класс просто `Helper`, `Util`, `Manager` etc.  
Указывай в названии класса, для чего предназначен этот конкретный хелпер
```java
class Helper

//ПРАВИЛЬНО:
class КакойтоКонкретныйHelper
```

- Это не константа
```java
private final EntityFactory FACTORY = new EntityFactory();

//ПРАВИЛЬНО:
private final EntityFactory factory = new EntityFactory();
```
Только `static final` поля являются константами.

- Название должно объяснять, что делает метод.  
В данном случае метод ищет путь
```java
public final class PathFinder {
  public static Optional<Deque<Coordination>> useBfsAlgorithm(...) {...}
}

//ПРАВИЛЬНО:
public final class BfsPathFinder {
  public static Optional<Deque<Coordination>> find(...) {...}
}
```

- Названия пакетов нужно писать стилем alllowercase 
```java
package entity.staticObject;

//ПРАВИЛЬНО:
package entity.staticobject;

```

- Несколько ошибок в названии.
  - В java всё- объекты, поэтому не используй это слово в названиях.  
  - Слово "static" тоже не используй в названиях классов и пакетов, чтобы не было ложных ассоциаций с зарезервированным словом `static`
```java
package entity.staticObject;

//ПРАВИЛЬНО:
package entity.stationaryentity
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Комментарии**

Комментарии здесь в основном не несут полезной нагрузки.  
Здесь комментарии вводят в заблуждение- один и тот же комментарий для разных перегруженных методов
```java
// create default creature
public Creature() {
  this(DEFAULT_CREATURE_HP, DEFAULT_CREATURE_SPEED);
}

// create default creature
public Creature(int hp, int speed) {
  this.hp = hp;
  this.speed = speed;
}
```
Когда в проекте много каментов, это плохо- пользы от них практически нет, они только забивают пространство и мешают читать код.  
В идеале, комментариев вообще не должно быть(кроме `TODO`), код должен объяснять сам себя через правильный нейминг и лаконичный код.  
*Мартин, "ЧК", гл.4*

**3. record Coordination**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

- Нет смысла в этом record'e переопределять `hashCode()` и `equals()`.

Эти переопределения нужно убрать:
```java

public record Coordination(int x, int y) {

  @Override
  public boolean equals(Object o) {...}

  @Override
  public int hashCode() {...}
}

//ПРАВИЛЬНО:
public record Coordination(int x, int y) {
}    
```

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**4. class GameMap**

- Ненужное ограничение наследования.

От этого класса нельзя унаследоваться- он final
```java
public final class GameMap {...}
```
Не вижу причин, почему этот класс нужно ограничить в наследовании.

- Не знаю, зачем нужен этот ад с дженериками
```java
public <T extends Entity> Optional<T> get(Coordination coordination, Class<T> type) {...}
```

Когда есть вот этот нормальный метод:
```java
public Optional<Entity> get(Coordination coordination) {...}
```

Сейчас этот метод нарушает SRP: `<T extends Entity> Optional<T> get(Coordination coordination, Class<T> type)`.  
Потому что какие бы задачи он не решал, они не имеют отношения к единой ответственности Карты по хранению в себе существ и обеспечению базового доступа к ним. 

- Нарушение SRP. 

Сейчас Карта не хранит в себе свои размеры.  
Значит их хранит кто-то другой и забирает на себя, таким образом, часть ответственности Карты.

+ 👍 А этот метод пусть будет, почему бы и нет
```java
public <T extends Entity> Queue<Coordination> getPositions(GameMap gameMap, Class<T> type) 
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность
```java
public void set(Coordination coordination, Entity entity) {
  map.put(coordination, entity);
}

//ПРАВИЛЬНО:
public void add(Coordinates coordinates, Entity entity) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение  
  map.put(coordinates, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

**5. abstract class Entity и его простые наследники Tree/Rock/Grass**

+ 👍 Идеально
```java
public abstract class Entity {
}

public class Rock extends Entity {
}
```

**6. final class CreatureConfig**

- Соблюдай требования константных классов. 

Константные классы, так же, как и утилитные, должны иметь приватный конструктор.  
Не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.  
*Блох, "Java. Эффективное программирование", изд.3, гл.2.4*

**7. abstract class Creature extends Entity**

- Не используй статические импорты. 

При использовании статических импортов становится непонятно, что используемый в классе метод или константа не его собственные, а чьи-то чужие
```java
import static config.CreatureConfig.DEFAULT_CREATURE_HP;
import static config.CreatureConfig.DEFAULT_CREATURE_SPEED;
//...

public Creature() {
  this(DEFAULT_CREATURE_HP, DEFAULT_CREATURE_SPEED);
}

//ПРАВИЛЬНО:
public Creature() {
  this(CreatureConfig.DEFAULT_CREATURE_HP, CreatureConfig.DEFAULT_CREATURE_SPEED);
}
```
*"ЧК", гл.17, G18*

- Константный класс в программе используется неправильно.

Класс `Creature` напрямую лезет в константный класс, а должен *только* получать необходимые данные в конструктор
```java
public Creature() {  <-- Этот конструктор нужно УДАЛИТЬ
  this(DEFAULT_CREATURE_HP, DEFAULT_CREATURE_SPEED);
}

public Creature(int hp, int speed) {  <-- Этот конструктор нужно ОСТАВИТЬ
  this.hp = hp;
  this.speed = speed;
}
```
Сейчас из-за конструктора, который напрямую лезет в константы, класс становится неуниверсальными и зависимым от константного класса.  

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
Про классы констант, конфигурации и их использование я писал [ТУТ](https://t.me/zhukovsd_it_chat/53243/176984)

Да, класс может получать в конструктор не отдельные поля, а целый объект с набором этих полей.  
Но там должна быть только нужная классу информация.  
*"ЧК", гл.3, "Объекты как аргументы"*

- Нарушение SRP.

Если Креатура должна узнать, является ли объект едой или барьером, то Креатура должна анализировать сам объект, а не брать этот объект из карты и потом анализировать 
```java
public abstract boolean checkBarrier(GameMap gameMap, Coordination followCoordinate);

public abstract boolean isGoal(GameMap gameMap, Coordination followCoordinate);

//ПРАВИЛЬНО:
public abstract boolean checkBarrier(Entity entity); 

public abstract boolean isGoal(Entity entity);
```

На мой взгляд, в этом классе вообще не нужен специальный метод, который проверяет барьер.  
Достаточно только метода, который проверяет еду.

Потому что барьером можно считать всё, что не является едой.

- Нарушение принципа минимального открытого интерфейса.

В классах публичными должны быть только те методы, которые предназначены для использования клиентами.  
Если метод предназначен только для использования потомками, то метод должен быть `protected`.  
Остальные методы должны быть `private`.
```java
public abstract class Creature extends Entity {
  //...

  public void makeMove(GameMap gameMap, Graph graph) {
    //...
    reproduce(gameMap);
  }

  public abstract void reproduce(GameMap gameMap);
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //...

  public void makeMove(GameMap gameMap, Graph graph) {
    //...
    reproduce(gameMap);
  }

  protected abstract void reproduce(GameMap gameMap);
}
```
По умолчанию все методы должны быть приватными, если нет причин для того, чтобы сделать их не приватными.  
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

**8. enum EntityType**

Енам существует для дополнительной типизации классов
```java
public enum EntityType {
  PREDATOR, HERBIVORE, GRASS, TREE, ROCK;
}
```
Не используй дополнительную типизацию классов, потому что это дублирует стандартные способы.  
Используй стандартные способы определять принадлежность объектов к классам.

Условный пример:
```java
//❌️ НЕПРАВИЛЬНО:
EntityType type = entity.getType();

switch (type) {
  case HERBIVORE: //...
  case PREDATOR:  //...
}

//✅ ПРАВИЛЬНО:
String name = entity.getClass().getSimpleName();

switch (name) {
  case "Herbivore": //...
  case "Predator":  //...
}
```

**9. class Herbivore/Predator extends Creature**

- Ссылка на несуществующий в проекте класс
```java
import static actions.InitActions.SPAWNER;
```
В репе нет пакета "actions".

- Неправильная перегрузка конструктора
```java
public Herbivore() {
  super(DEFAULT_HERBIVORE_HP, DEFAULT_HERBIVORE_SPEED);
}

public Herbivore(int hp, int speed) {
  super(hp, speed);
}

//ПРАВИЛЬНО:
public Herbivore() {
  this(DEFAULT_HERBIVORE_HP, DEFAULT_HERBIVORE_SPEED);
}

public Herbivore(int hp, int speed) {
  super(hp, speed);
}
```
*Эккель "Философия Java", гл.5 "Вызов конструктора из конструктора"*

- Нарушение SRP, нарушение Low Coupling.

Этот класс не должен зависеть от экшенов.  
Наоборот, это экшены должны зависеть от этого класса.

То есть только экшены должны вызывать методы из `Creature` и его потомков.  
`Creature` и его потомки не должны вызывать никаких методов из экшенов
```java
import static actions.InitActions.SPAWNER;

public class Herbivore extends Creature {
  //...

  @Override
  public void reproduce(GameMap gameMap) {
    //...
    SPAWNER.spawnToMap(gameMap, HERBIVORE);  <-- Зависимость Herbivore от InitActions
  }
}
```

- Неправильное распределение кода по уровням иерархии классов.

Все наследники `Creature` (то есть классы `Herbivore` и `Predator`) имеют общую характеристику- еду, которую они едят.  
Так как они едят по одному виду еды, этот параметр нужно вынести в их общего предка.  
Сейчас так:
```java
public abstract class Creature extends Entity {
  //...

  public abstract boolean isGoal(GameMap gameMap, Coordination followCoordinate);
}

public class Herbivore extends Creature {
  //...
 
  @Override
  public boolean isGoal(GameMap gameMap, Coordination followCoordinate) {
    return (checkClassType(gameMap, followCoordinate, Grass.class));
  }
}

public class Predator extends Creature {
  //...
  @Override
  public boolean isGoal(GameMap gameMap, Coordination followCoordinate) {
    return (checkClassType(gameMap, followCoordinate, Herbivore.class));
  }
}
```

Правильно так:
```java
public abstract class Creature extends Entity {
  private final Class<? extends Entity> goal;  
  //...

  public Creature(Class<? extends Entity> goal, ...) {
    //...
    this.goal = goal;
  }

  public boolean isGoal(Entity entity) {
    return entity.getClass() = goal;
  }
}

public class Herbivore extends Creature {
  private static final Class<? extends Entity> GOAL = Grass.class;
  //...

  public Herbivore(...) {
    super(GOAL, ...);
  }
}

public class Predator extends Creature {
  private static final Class<? extends Entity> GOAL = Herbivore.class;
  //...

  public Predator(...) {
    super(GOAL, ...);
  }
}
```

- Нарушение Low Coupling.

Какие-то мутные зависимости от непонятных классов
```java
public class Herbivore extends Creature {
  //...

  @Override
  public boolean checkBarrier(GameMap gameMap, Coordination followCoordinate) {
    return (checkClassType(gameMap, followCoordinate, Rock.class)  <-- Вызывает метод из класса Helper
    //...
  }
}
```

**10. Пакет utility**

Не все классы в пакете являются утилитами.  
`class EntitySpawner` не является утилитой. 

**11. class Helper**

- Нейминг.

По своей внутренней структуре, этот класс не хелпер, а утилита.

Класс c названием "Хелпер" не может находиться в папке "утилита" и наоборот.  
Это не одно и то же, почитай, какие требования предъявляют к классам названия "Helper", "Utility", "Manager" etc.

- Непонятное.

В классе находятся два метода, которые используются для разных ответственностей.  
Класс удалить, его методы перенести в те классы, в интересах которых они используются. 

**12. class PathFinder**

- Возвращай `Optional` только тогда, когда это необходимо.

Здесь не нужно возвращать `Optional`.  
Если метод не найдет путь, пусть вернет пустую коллекцию
```java
public static Optional<Deque<Coordination>> useBfsAlgorithm(...) {...}

//ПРАВИЛЬНО:
public static Deque<Coordination> find(...) {...}
```
*Блох "Java. Эффективное программирование", изд.3, 8.6.*

- Не используй экзотические типы данных для возвращаемых значений
```java
public static Deque<Coordination> find(...) {...}

//ПРАВИЛЬНО:
public static List<Coordination> find(...) {...}
```

- Из сигнатуры метода неясно, как пользоваться поиском пути
```java
public static Optional<Deque<Coordination>> useBfsAlgorithm(GameMap gameMap, Map<Coordination, List<Coordination>> graph, Coordination start) {...}
```

Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или A*.  

В случае алгоритма BFS, поиск должен искать путь до точки, в которой находится существо нужного класса.  
Например так:
```java
public class BfsPathFinder {

  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
    //ищет путь на карте от точки start
    //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

- Нарушение SRP, OCP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и НЕ ДОЛЖЕН определять эти условия самостоятельно, например путем САМОСТОЯТЕЛЬНОГО анализа принадлежности `Entity` тому или иному виду существ
```java
if (entity.get() instanceof Grass || entity.get() instanceof Herbivore) {...}
```

Нарушение OCP здесь состоит в том, что при добавлении в проект нового существа, например, Птицы, 
придется изменить код класса. То есть, сейчас класс открыт для изменений:
```java
if (entity.get() instanceof Grass || entity.get() instanceof Herbivore || entity.get() instanceof Bird) {...}
```

Должно быть примерно так:
```java
public class BfsPathFinder {

  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
    //...
    if(entity.getClass() == target) {
      //этот объект- еда    
    } else {
      //этот объект- не еда  
    }
  }
}
```

- Антипаттерн "Стрела".

Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
while (!queue.isEmpty()) {
  if (!path.getLast().equals(start)) {
    if (entity.isPresent()) {
      if (entity.get() instanceof Grass || entity.get() instanceof Herbivore) {
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

- Сложные условия в if.

Непонятно, что происходит в этом if-e, его условие вообще не читается
```java
if (!path.getLast().equals(start)) {...}
```
Вводи вспомогательные методы или поясняющие переменные.

**13. class Menu**

- Стрела.

- Нейминг. 

Название метода обманывает.  
Метод не только печатает, но еще и делает много всякого другого
```java
void printMenu() {
  //45 строк кода    
}
```

- Код класса нечитабелен.

Код свален в один большой божественный метод.  
Такой код нечитабелен, вводи вспомогательные методы.

- Класс сделан в процедурном стиле.

Про меню в ООП стиле я писал [ТУТ](https://t.me/zhukovsd_it_chat/53243/114908).

**14. Отсутствуют указанные в ТЗ классы Action's**

**15. class Simulation**

- Свалка.

Половина кода в классе закомментирована.

Закомментированный код это антипаттерн "Лодочный якорь", такой код никогда уже не понадобится.  
Не превращай программу в свалку. 

Если что-то хочешь сохранить на память, сделай коммит в гите с соответствующим комментарием.  
Правильное использование комментариев: *"Чистый код", гл.4*

- Не используй аргументы-флаги. Тем более в конструкторах
```java
public Simulation(..., Boolean stopFlag) {...}
```
*Мартин "ЧК", гл.3, "Аргументы-флаги"*
```
"Аргументы-флаги уродливы... функция выполняет более одной операции" - Мартин.
```

- Нарушение SRP.

Распечатку карты нужно вынести в отдельный класс, например:
```java
public class GameMapRenderer {
  //...

  public void render(GameMap gameMap) {
    //печатает карту
  }    
}
```

ТЗ:
```
Рендерер #
Рендерер ответственен за визуализацию состояния поля, его отрисовку. 
По желанию студента интерфейс приложения может быть консольным, либо графическим.
```

- Неправильное использование экшенов.

Хотя экшены ты вообще не закомитил на репу, всё равно можно сделать вывод, что используются они в проекте неправильно.

Экшены должны быть сделаны по ТЗ:
```java
Actions #
Action - действие, совершаемое над миром. Например - сходить всеми существами. 
Это действие итерировало бы существ и вызывало каждому makeMove(). 
Каждое действие описывается отдельным классом и совершает операции над картой. 
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

Использоваться экшены в классе Симуляция  должны примерно так:
```java
public class Simulation {
  private final List<Action> initActions = List.of(new UnoAction());
  private final List<Action> turnActions = List.of(new DosAction(), new TresAction(), new MoveAction());
  //...

  public Simulation(...) {
    //...
    executeActions(initActions);  //выполнение экшенов на старте
  }

  public void nextTurn() {
    //...
    executeActions(turnActions);  //выполнение экшенов на каждом ходе
  }

  private void executeActions(List<Action> actions) {
    for(Action a : actions) {
      a.execute(карта);
    }
  }

 //...
}
```

**16. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

Сделать программу, чтобы работала.

Придерживайся ТЗ.  
Если в нем указано, что нужно сделать класс- делай его.  
Соблюдение ТЗ это важный навык для работы. 

Подробнее разберись с Action'ами, посмотри ролики на ютубе про паттерн "Command", чтобы получить о нем общее представление.

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы.  
Посмотреть на ютубе ролики Немчинского про SOLID- по одному ролику на каждый принцип.

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.164(348)  
#ревью #симуляция 