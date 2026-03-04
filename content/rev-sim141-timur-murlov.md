https://github.com/murlov/simulation  
[Тимур]

Есть много над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Реализованы пауза/пуск во время работы
+ 👍 Используется паттерн Listener/Callback для сообщений о происходящих событиях
+ 🚀 Возможность конфигурирования параметров симуляции через меню

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Если это константа, то она должна быть final static
```java
private final int NUMBER_OF_ENTITY_TYPES = 5;

//ПРАВИЛЬНО:
private final static int NUMBER_OF_ENTITY_TYPES = 5;
```

- Название полей класса нужно писать стилем camelCase
```java
record Coordinates(int X, int Y)

//ПРАВИЛЬНО:
record Coordinates(int x, int y)
```

- Обычно эта пара называется width/height
```java
Size(int width, int length)

//ПРАВИЛЬНО:
Size(int width, int height)
```

- Формулируй точнее. 

Старайся, чтобы название соответствовало правилам английского языка.  
В английском языке сначала сказуемое(глагол), потом- дополнение.

Поэтому если первым стоит "count", значит метод обещает что-то считать.  
Но этот метод на самом деле делает декремент, а "count"(counter) здесь это не действие, а название счетчика
```java
void countForEntityTypeDecrement(Class<? extends Entity> entityType) {
  //уменьшает на единицу количество существ заданного типа
}

//ПРАВИЛЬНО:
void decrementEntityCount(Class<? extends Entity> entityType)
```

То же самое:
```java
void satietyDecrement()

//ПРАВИЛЬНО:
void decrementSafety()
```

- Название метода неправильно описывает выполняемое действие.

Слово "set" означает "установить", а не "добавить".  
То есть, установить значение какой-то конкретной переменной(сеттер) или установить значение переменной по конкретному адресу.  
Пример из стандартной библиотеки: `List.set(int index, E element)`.

Здесь больше подойдет "add"(добавить) или "put"(поместить): 
```java
public class SimulationMap {
  public void setEntity(Coordinates coordinates, Entity entity) {
    //добавляет новое существо в список существ
  }
  //...
}  

//ПРАВИЛЬНО:
public void putEntity(Coordinates coordinates, Entity entity) 
```
То есть, установится не конкретное поле, а добавится новый элемент в в группу уже существующих.  
Примеры из стандартной библиотеки: `List.add(E e)`, `Set.put(K key, V value)`.

- Делай названия более однозначными, чтобы избежать разных толкований.

Рендерер печатает на экран. Под логгированием мы обычно понимаем немного другое- запись информации в лог-файл
```java
public interface Renderer {
  void logMove(...);
  void logAttack(...);
  //...
}

//ПРАВИЛЬНО:
public interface Renderer {
  void printMove(...);  //showMove(...)
  void printAttack(...);  //showAttack(...)
  //...
}
```

- Название метода должно быть глаголом. 

"View" это существительное.  
По крайней мере, в java это слово используется как существительное в качестве названия класса или слоя(группы классов) представления.  
А не как, например, глагол "узреть"- хотя это тоже одно из значений слова "View" в английском языке.

Если нужно что-то распечатать, используй стандартные слова
```java
public void viewMap(SimulationMap simulationMap) {...}

//ПРАВИЛЬНО: print, show, draw, render
```

- При использовании консоли ввод данных уже подразумевает нажатие энтера
```java
System.out.println("Для выхода введите q и прожмите Enter.");

//ПРАВИЛЬНО:
System.out.println("Для выхода введите 'q'.");

//ПРАВИЛЬНО, ХОТЬ И НЕ ИЗЯЩНО:
System.out.println("Для выхода нажмите 'q' и нажмите Enter.");
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется.**
 
В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без `else` будет выглядеть читабельней
```java
if (type == Rabbit.class) {
  return Grass.class;
} else {
  throw new IllegalArgumentException("Unknown entity type: " + type);
}

//ПРАВИЛЬНО:
if (type == Rabbit.class) {
  return Grass.class;
} 
throw new IllegalArgumentException("Unknown entity type: " + type);
```

**3. Форматирование строк**

Если нужно печатать или создавать строку с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматирование- тогда сразу будет виден весь шаблон
```java
System.out.println(creatureType.getSimpleName() + " has moved from " + from + " to " + to + "\n");

//ПРАВИЛЬНО:
System.out.printf("%s has moved from %s to %s  %n", creatureType.getSimpleName(), from, to);
```

**4. record Coordinates(int X, int Y)**

+ 👍 Record для координаты- идеально.

- Нарушение SRP

Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: x,y или row,column.
Лучше даже, если этот класс будет record'ом.  
Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия- гененрирует случайную координату в заданных пределах.  
Тем самым нарушая SRP: 
```java
public static Coordinates getRandom(Size size)
```
Нужно ли координате для идентификации точки в пространстве, искать случайную координату, тем более в пределах, заданных объектом `Size`? Нет.

Получение случайной координаты нужно не для идентификации точки в пространстве, а для начального размещения существ на случайных координатах.  
А значит, генерация случайной координаты это часть ответственности начального размещения существ.  

Доказательство:
Идентификация точки в пространстве возможна без создания случайных координат.  
Начальное размещение существ- невозможно без создания случайных координат.

Еще довод:  
Одна из главных идей ООП- делать классы универсальными для переиспользования в других проектах, где используются одинаковые концепции.  
Например, концепция координаты.

Можно представить программу, которая использует координату, но которой не нужен процесс создания случайной координаты.  
Скажем, математическая программа, которая будет строить кривую функции.  
Там не нужно будет генерировать случайные координаты, а нужно будет рассчитывать конкретные координаты по формуле. 

А значит, создание случайной координаты нужно в контексте проекта "Симуляция".  
Но не обязательно будет нужна в контексте другой задачи, которая будет использовать ту же концепцию(сущность) координаты.

- Нарушение Low Coupling.

Нарушение SRP потянуло за собой высокую связность классов.
Координата вынуждена знать про классы, которые не имеют никакого отношения к идентификации точки в пространстве: `Size` и `RandomProvider`:
```java
public static Coordinates getRandom(Size size) {
  Random random = RandomProvider.getInstance();
  //...
}
```
Из-за этого координата не может быть переиспользована в других проектах сама по себе, она вынуждена везде ходить в обнимку с этими двумя классами.

**5. record Size(int width, int length)**

👍 Объект, который хранит размеры прямоугольной области, почему бы и нет.  
Используется как объект-аргумент для Карты- тоже ок.  
*"ЧК", гл.3, "Объекты как аргументы"*

**6. class SimulationMap**

- Неправильная перегрузка конструктора
```java
public SimulationMap(Size size) {
  this.size = size;
  //...
}

public SimulationMap(SimulationMap simulationMap) {
  this.size = simulationMap.size;
  //...
}

//ПРАВИЛЬНО:
public SimulationMap(Size size) {
  this.size = size;
  //...
}

public SimulationMap(SimulationMap simulationMap) {
  this(simulationMap.size);
}
```
*Эккель "Философия Java", гл.5 "Вызов конструктора из конструктора"*

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретным классом представителей и не должна даже знать по именам наследников Entity.  
Здесь карта для каких-то своих целей ведет подсчет существ определенного типа
```java
private int wolfsCount;
private int rabbitsCount;
private int grassCount;
```
Если карта будет знать по именам наследников Entity и иметь для них персональные поля-счетчики, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод счетчик `birdCount`.

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
Подсчитать количество существ определенного вида,  
уменьшением счетчик существ на единицу,  
увеличить счетчик существ на единицу,  
вернуть случайную пустую координату.

Наверное, для проекта в целом полезно иметь метод, который уменьшает счетчик существ на единицу.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно уменьшать счетчик существ на единицу.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```java
private final Map<Coordinates, Entity> entities;

public Map<Coordinates, Entity> getEntities() {
  return entities;
}
```

В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```java
Карта карта = new Карта();
<заселить карту существами>
карта.getEntities().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

Сейчас можно вставить существ в карту двумя способами:  
Конвенционный способ- через использование метода `public void setEntity(Coordinates coordinates, Entity entity)`.  
И "нелегальный" способ- через `карта.getEntities().put(coordinates, entity)`.

Это плохо уже само по себе, но здесь есть еще одна неочевидная проблема.  
Хотя оба эти способа позволят записать в карту существ и их координаты, но при этом будут происходить разные процессы.  
При записи через `setEntity(...)` происходит синхронизация координаты в карте с той, которая хранится в самом существе
```java
public void setEntity(Coordinates coordinates, Entity entity) {
  entity.setCoordinates(coordinates); <-- запись координаты в существо
  entities.put(coordinates, entity); <-- запись координаты в карту
}
```
При нелегальной записи через `карта.getEntities().put(coordinates, entity)` не произойдет синхронизация координат в существе и карте.

Тут есть два варианта, как сделать правильно.  
Либо вернуть не оригинал мапы, а ее копию.  
Тогда клиент не сможет волюнтаристски изменять данные в хешмапе Карты:  
```java
public Map<Coordinates, Entity> getEntities() {
  return new HashMap(entities);
}
```

Либо возвращать существ персонально:
```java
public Entity get(Coordinates coordinates)
public Coordinates getCoordinates(Entity entity)
```
Лично мне больше нравится второй вариант, когда существ возвращают персонально. 

- Нарушение SRP.

Нет метода, который возвращает существо по ее координате.  
Значит клиент эту составляющую ответственность карты берет на себя и вытягивает существо через заднюю дверь- используя `getEntities()`.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 

Если координата некорректна(находится вне пределов карты), нужно бросать исключение.  
Сейчас можно вставить существа на координату, которая находится вне пределов карты
```java
public void setEntity(Coordinates coordinates, Entity entity) {  
  entity.setCoordinates(coordinates);  <-- можно вставить координату (-100500, +100500)
  entities.put(coordinates, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Неочевидная мотивация. 

Класс содержит поле типа `Size`. В том классе хранится ширина, высота и площадь.  
При этом есть геттер для всего Size и геттер для площади, но нет геттеров для размеров.  
Приведи к какому-то единому стандарту
```java
public Size getSize() {
  return size;
}

public int getNumberOfCells() {
  return size.getArea();
}

//ПРАВИЛЬНО ТАК:
public Size getSize() { <-- только геттер для Size
  return size;
}

//ТАК ТОЖЕ ПРАВИЛЬНО - геттеры всех величин, кроме Size:
public int getNumberOfCells() {
  return size.getArea();
}

public int getWidth() { 
  return size.width(); 
}

public int getHeight() { 
  return size.height(); 
}
```

**7. interface PathFinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

- Из сигнатуры неясно, как искать путь с помощью этого поиска
```java
public interface PathFinder {
  List<Coordinates> execute(SimulationMap simulationMap, Creature creature);
}
```

Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритма BFS.  
В данном случае- до точки, в которой находится существо нужного класса
```java
public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```

Или поиск должен искать путь между точками старта и финиша, согласно алгоритма A*
```java
public List<Coordinates> find(Карта карта, Coordinates from, Coordinates to) {...}
```

Так же, поиск пути по алгоритму A* может использовать ту же сигнатуру, что и BFS.  
Тогда поиск пути в нем интерпритируется не как поиск пути между двумя точками, а как поиск цели и прокладывание пути к ней.  
Например: 
```java
public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
  //ищет координату цели
  //ищет путь по алгоритму A* между точкой стартой и точкой цели
}
```
Плюсы- унификация сигнатуры разных алгоритмов поиска для полиморфизма.  
То есть, можно сделать семейство родственных классов поиска, объединенных общим интерфейсом.

Сейчас из сигнауры метода понятно, что поиск пути не только ищет путь.  
Но при этом еще обладает какими-то дополнительными знаниями о правилах игры- раз принимает в себя `Creature`.  
Наверное, прием креатуры нужен для того, чтобы как-то потому ее проанализировать и на основе этой информации принимать какие-то решения.  
Но в любом случае, эти дествия по анализу полученной креатуры не имеют никакого отношения к единой ответственности по поиску карты.

Еще раз: с точки зрения А*, поиск пути на карте проводится между двумя точками.  
С точки зрения BFS, поиск пути на карте проводится от точки старта, до точки, где находится объект отвечающий заданным условиям.  
В любом случае, все необходимые для этого данные поиск должен получать в готовом виде, а не высчитывать самостоятельно.

**8. class BfsPathFinder implements PathFinder**

- Не нужно хранить размеры карты в виде полей в этом классе.  
Потому что в метод поиска передается карта целиком и из нее можно получить размеры непосредственно в этом методе
```java
public class BfsPathFinder implements PathFinder {
  private final int numberOfCells;
  private final Size sizeOfMap;

  public BfsPathFinder(Size sizeOfMap, int numberOfCells) {...}

  public List<Coordinates> execute(SimulationMap simulationMap, Creature creature) {...} 
}

//ПРАВИЛЬНО:
public class BfsPathFinder implements PathFinder {
  public List<Coordinates> execute(SimulationMap simulationMap, ...) {
    int numberOfCells = simulationMap.getБлабла();
    Size sizeOfMap = simulationMap.getБлаблабла();
    //...
  }  
}
```
Среди прочего, сейчас это приводит к потенциальным багам.  
Например, можно создать экземпляр поиска с размерами, которые будут не совпадать с размерами карты, в которой нужно выполнить поиск:
```java
public class Main {
  public static void main(String[] args) {
    Size firstSize = new Size(5, 5);
    BfsPathFinder pathFinder = new BfsPathFinder(firstSize, 3);

    Size secondSize = new Size(100, 100);
    SimulationMap simulationMap = new SimulationMap(secondSize);

    Rabbit rabbit =  new Rabbit(1, 1, 1);
    simulationMap.setEntity(new Coordinates(20, 20), rabbit);
    simulationMap.setEntity(new Coordinates(30, 30), new Grass());

    List<Coordinates> path = pathFinder.execute(simulationMap, rabbit);
  }
}

//РЕЗУЛЬТАТ:
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 120 out of bounds for length 3
	at com.murlov.action.BfsPathFinder.execute(BfsPathFinder.java:46)
	at com.murlov.temp.Main.main(Main.java:25)
```

- При переопределении методов всегда ставь аннотацию. Это полезная привычка, которая повышает читаемость кода
```java
public List<Coordinates> execute(SimulationMap simulationMap, Creature creature)

//ПРАВИЛЬНО:
@Override
public List<Coordinates> execute(SimulationMap simulationMap, Creature creature)
```

- Нарушение SRP.

Сейчас поиск берет на себя чужую ответственность- определяет за животных, какую еду они едят
```java
if (type == Wolf.class) {
  return Rabbit.class;
} else if (type == Rabbit.class) {
  return Grass.class;
}
```

Как я писал выше, поиск должен получать все необходимые данные во входящие аргументы метода, а не опрашивать `Creture's` самостоятельно.  
Если этот класс реализует алгоритм BFS, то принимать в себя он должен те данные, которые нужны для этого алгоритма:
```java
public List<Coordinates> execute(SimulationMap simulationMap, Creature creature)

//ПРАВИЛЬНО:
public List<Coordinates> find(SimulationMap simulationMap, Coordinates start, Class<? extends Entity> target)
```
Поиск не должен хранить в себе меню для животных. Каждое существо само должно знать, что оно ест.  
В конце концов, каждое существо тут, даже камень, знает, где оно стоит на карте.  
Почему бы ему не знать свои пищевые пристрастия.

- Нарушение OCP.

Из-за того, что класс сам анализирует креатур на принаждлежность к типам, класс открыт для изменений.  
Например, при добавлении нового существа `Bird`, придется вносить изменения в алгоритм поиска:
```java
else if (type == Rabbit.class) {
  return Grass.class;
} else if(type = Bird.class) {...}
```

- Смещения нужно вынести из метода и сделать константами.  

Потому что сейчас при вызове метода `execute(...)` каждый раз пересоздается объект(а массив это объект) со смещениями.  
Пары x/y это координаты. Направление можно представить как координату, смещенную относительно начала движения
```java
public List<Coordinates> execute(...) {
  int[][] directions = {
    {0, -1},
    {1, 0},
    //...
  };
  //...
}

//ПРАВИЛЬНО:

private static final List<Coordinate> SHIFT_COORDINATES = List.of(
    new Coordinates(0, -1),
    new Coordinates(1, 0),
    //oth's coord's
);

public List<Coordinates> execute(...) {
  for(Coordinates shift : SHIFT_COORDINATES) {...}
  //...
}
```

**9. Пакет model**

Если в проекте есть пакет "model", то это заявка на то, что все модели проекта лежат в этом пакете.  
Следовательно, во всех других пакетах должны лежать не модели.

В этом проекте есть модели, которые не находятся в этом пакете: `Coordinates`, `SimulationMap` etc.  
Экшены- тоже модели.

С точки зрения ООП, модель это класс, который не использует представление.  
Поэтому `SimulationMap` это модель, а `Simulation` и `ConsoleRenderer`- нет.

Пакет нужно переименовать в `entity`.

**10. interface MoveEventListener**, паттерн Callback 

- Нарушение ISP.

Много методов из разных ответственностей, которые объедененены в одном интерфейсе
```java
public interface MoveEventListener {
  void onMove(Class<? extends Creature> creatureType, Coordinates from, Coordinates to);
  void onAttack(Class<? extends Creature> attackerType, Coordinates from, Class<? extends Creature> victimType, Coordinates to);
  void onEat(Class<? extends Creature> creatureType, Coordinates from, Class<? extends Entity> victimType, Coordinates to);
  void onSpawn(Class<? extends Entity> entityType, Coordinates coordinates);
  void onDeath(Class<? extends Creature> creatureType, Coordinates coordinates);
  void onMoveEnd(SimulationMap simulationMap);
  void onMoveStart();
}
```
Нелогичность общего интерфейса видна даже на уровне нейминга.  
Интерфейс называется "Движение", но некоторые методы в нем не соответствуют этому термину.  
Например, атаку и поедание можно считать движением.  
Но умирание существа, хотя и является процессом, но это не движение.  
Помещение существа в карту это конечно некое перемещение. Но это не движение, которое выполняет само существо. 

ISP: "Клиенты не должны зависеть от методов, которые они не используют".  
Тот клиент, который должен дергаться при помещении существа в карту(метод `onSpawn(...)`) не должен знать про методы атаки хищника `onAttack(...)`.  
Даже Травоядное не должно знать про атаку, потому что травоядное хотя и родственник Хищника, но не умеет атаковать.  
Хищник не ест, а атакует, поэтому тоже не должен знать про метод поедания.  
Хищник, который дергается при атаке, не должен знать про методы помещения существ на карту и т.д.

Каждый метод здесь должен быть представлен отдельным интерфейсом.  
Методы с одинаковой сигнатурой, могут быть одним и тем же методом одного и того же интерфейса
```java
public interface MoveEventListener {
  void onSpawn(Class<? extends Entity> entityType, Coordinates coordinates);
  void onDeath(Class<? extends Creature> creatureType, Coordinates coordinates);
  //...
}

//ПРАВИЛЬНО:
public interface ОбщееНазваниеДляДействийСвязанныхСсуществомИееКоординатойListener {
  void onДействие(Entity entity, Coordinates coordinates);
}
```

- Передается недостаточно информации.

Нет причин передавать в листнеры класс существа вместо самого экземпляра существа
```java
void onAttack(Class<? extends Creature> attackerType, Coordinates from, Class<? extends Creature> victimType, Coordinates to);

//ПРАВИЛЬНО:
void onAttack(Creature attacker, Coordinates from, Creature victim, Coordinates to);
```
Тогда при необходимости, из экземпляра существа клиент сможет извлечь намного больше информации.  
Например, он может написать не только, что некий кот напал на некую мышь.  
А более конкретно: "Кот Васька напал на мышь в точке (5, 1)"

Аналогия- использование стандартных листнеров/колбеков в Swing и Android.  
Там в подобных случаях передаются сами объекты, а не их классы.

**11. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`
```java
private Coordinates coordinates;
```

**12. abstract class Creature extends Entity**

- Нарушение конвенции кода.  
Константы должны стоять выше остальных методов.

- Магические числа
```java
satiety = min(10, ++satiety);
```

- Делай свои намерения более очевидными.

На таких командах можно надолго зависнуть
```java
satiety = min(10, ++satiety);
```
Возникают вопросы:  
Произойдет ли увеличение значения satiety на единицу до применения команды `min`?  
Произойдет ли увеличение значения satiety на единицу после применения команды `min`?  
При префиксной форме инкремента, сначала увеличивается значение поля, а потом к нему применяется операция или наоборот?

Что вообще тут происходит? Через время тебе самому будет трудно разобраться в этом коде.

Обозначь намерения более явно:
```java
satiety++;
if(satiety > MAX_SATIETY) {
  satiety = MAX_SATIETY;
}
```

- Нарушение инкапсуляции. Это поле не должно быть публичным 
```java
public boolean isDead;
```

- Класс должен хранить тип своей еды. Примерно так:
```java
private final int speed;
private final Class<? extends Entity> food;

public Creature(Class<? extends Entity> food, int speed, ...)
```

- Нарушение инкапсуляции. 

Публичными должны быть только те методы, которые предназначены для использования клиентом.  
Вспомогательные методы, которые используются только в самом классе или его потомках, должны быть private/protect
```java
public boolean makeMove(SimulationMap simulationMap, Coordinates oldCoordinates, PathFinder pathFinder) {
  //...
  satietyDecrement();
}

public void satietyDecrement() {...}

//ПРАВИЛЬНО:
public boolean makeMove(SimulationMap simulationMap, Coordinates oldCoordinates, PathFinder pathFinder) {
  //...
  satietyDecrement();
}

private void satietyDecrement() {...}
```
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

- Совмещение команды и запроса.

Метод должен или выполнить действие или ответить на запрос.  
Но не то и другое вместе.

Этот метод выполняет действие- совершает ход.  
И отвечает на запрос- надо полагать, говорит, был ли выполнен ход или нет
```java
public boolean makeMove(SimulationMap simulationMap, Coordinates oldCoordinates, PathFinder pathFinder)
```

Мотивация такого совмещения неясна- тут используется слушатель, который и так информирует клиента о совершаемых моделью действиях:
```java
private MoveEventListener listener;
```

Поэтому нет необходимости возвращать результат выполнения команды в виде булевого значения
```java
public boolean makeMove(...)

//ПРАВИЛЬНО:
public void makeMove(...)
```
*Мартин, "Чистый код", гл.3, "Разделение команд и запросов"*  

- Нарушение OCP.

Класс не должен знать своих потомков и как-то учитывать их в логике своей работы
```java
case "Wolf" -> Rabbit.class;
case "Rabbit" -> Grass.class;
```
Потому что при добавлении новых потомков, придется переписывать предка и тем самым нарушать правило 
"класс должен быть закрыт для изменений".

Общий для всех креатур функционал совершения хода должен находиться здесь.  
А отличия- в потомках, в их переопределенных методах `makeMove(...)`.

- Креатура должна сама искать свои текущие координаты
```java
public boolean makeMove(SimulationMap simulationMap, Coordinates oldCoordinates, PathFinder pathFinder) {...}

//ПРАВИЛЬНО:
public boolean makeMove(SimulationMap simulationMap, PathFinder pathFinder) {...}
```
Креатура все равно в метод хода принимает `SimulationMap`, поэтому может спросить у карты свое текущее местоположение.  

Если креатура будет принимать свое текущее местоположение извне, а не получать самостоятельно из карты, то это потенциально может привести к багам.  
Потому что креатура не может гарантировать, что пришедшая извне координата *действительно* соответствует ее текущему местоположению.  
Слепая вера клиенту тут вредна.

- Не делай сложных условий в if'ах.  
Вводи вспомогательные методы или поясняющие переменные, которые своим названием будут объяснять, что происходит
```java
if (path.size() == NEIGHBOR_PATH_LENGTH && hasResourceNearby(oldCoordinates, simulationMap)) {...}

//ПРАВИЛЬНО:
boolean isНазваниеКотороеВсеОбъясняет = path.size() == NEIGHBOR_PATH_LENGTH && hasResourceNearby(oldCoordinates, simulationMap);
if (isНазваниеКотороеВсеОбъясняет) {...}
```
*Фаулер, "Рефакторинг", гл.6, "Введение поясняющей переменной"*

Вот это вообще не читается:
```java
if (simulationMap.getEntities().containsKey(targetCoordinates) && simulationMap.getEntities().get(targetCoordinates).getClass() == targetEntityType) {...}
```

**13. Пакет model.factory (entity.factory)**

+ 👍 Фабрика это хорошо.

- Расположение фабрик.

Фабрики существ не являются существами и поэтому не должны находиться внутри пакета существ.  
Должно быть два равноценных пакета, находящихся на одном уровне: `entity` и `factory`(или `entityfactory` или `factory.entityfactory`).

- Неправильное использование паттерна Фабрика.

Фабрики здесь не упрощают процесс создания существ, а просто дублируют их конструкторы:
```java
public class TreeFactory implements EntityFactory {
  @Override
  public Entity create() {
    return new Tree();
  }
}

public class WolfFactory implements EntityFactory {
  private final int health;
  //...
  public WolfFactory(int health, int speed, int satiety, int damage) {...}

  @Override
  public Entity create() {
    return new Wolf(health, speed, satiety, damage);
  }
}
```
Сейчас это похоже на антипаттерн "Полтергейст".

Фактически, ты можешь выбросить все эти фабрики.  
И, немного переделав, оставить только `EntityFactoryProvider` в качестве простой параметризированной фабрики:
```java
public class EntityFactoryProvider {
  public static EntityFactory getFactory(String name, SimulationSettings settings) {
    return switch (name) {
      case "Grass" -> new GrassFactory();
      case "Wolf" ->
          new WolfFactory(settings.getPredatorHealth(), settings.getPredatorSpeed(), settings.getPredatorSatiety(), settings.getPredatorDamage());
      //...
    };
  }
}

//ПРАВИЛЬНО:
public class EntityFactory {
  private final SimulationSettings settings;

  public EntityFactory(SimulationSettings settings) {...}

  public Entity get(Class<? extends Entity> type) {
    String name = type.getClass().getSimpleName();
    return switch (name) {
      case "Grass" -> new Grass();
      case "Wolf" ->
          new Wolf(settings.getPredatorHealth(), 
             settings.getPredatorSpeed(), 
             settings.getPredatorSatiety(), 
             settings.getPredatorDamage());
      //...
    };
  }
}
```

**14. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(SimulationMap simulationMap);
}
```

**15. class EntitiesMoveAction implements Action**

- Не соответствует концепции Action.

По своей сути, это не экшен, а что-то иное.  
Все экшены должны должны иметь только один публичный метод- `execute(SimulationMap simulationMap)`.  
Непубличных методов они могут иметь сколько угодно.  
Экшены это разновидность паттерна `Command` и должны использоваться соответственно идее этого паттерна.

Этот же класс имеет много разных публичных методов, кроме `execute(...)`:
```java
public class EntitiesMoveAction implements Action {
  //...
  public interface PauseCallback {
    void waitIfPaused();
  }

  public interface ExitCallback {
    boolean shouldExit();
  }

  public interface SpawnCallback {
    void spawn(SimulationMap simulationMap);
  }

  public void setPauseCallback(PauseCallback pauseCallback) {...}
  public void setExitCallback(ExitCallback exitCallback) {...}
  public void setSpawnCallback(SpawnCallback spawnCallback) {...}

  @Override
  public void execute(SimulationMap simulationMap) {...}
}
```
Не совсем даже понятно, в чем состоит единая ответственность этого класса- потому он в себе хранит много всякого.

- Сложная логика совершения хода.

В соответствии с ТЗ, у тебя в Creature есть метод, который совершает ход:
```java
public abstract class Creature extends Entity {
  //...
  public boolean makeMove(SimulationMap simulationMap, Coordinates oldCoordinates, PathFinder pathFinder) {
    //логика совершения хода
  }
}
```

Вся логика по передвижению существа должна находиться именно там.  
Иначе единая ответственность по совершению хода размазывается на несколько классов.  
Здесь, в экшене хода, должна быть только логика инициирования движения существа, а не логика управления движением существа.

Экшен должен только дать креатуре пинка, а дальше она должна бежать сама:  
```java
public class EntitiesMoveAction implements Action {
  //миллион каких-то публичных методов

  @Override
  public void execute(SimulationMap simulationMap) {
    //миллион строк кода- управление движением существа
  }
}

//ПРАВИЛЬНО:
class MoveAction реализует Action {

  @Override
  public void execute(Карта карта) {  <-- ТОЛЬКО ОДИН ПУБЛИЧНЫЙ МЕТОД В КЛАССЕ
    List<Creature> creatures = getCreatures(карта);
    for(Creature creature: creatures) {
      creature.makeMove(карта);  //только даёт пинка
    }
  }

  private static List<Creature> getCreatures(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

**16. class Spawner**

- В метод передается огромное количество аргументов
```java
public static void execute(SimulationMap simulationMap, String entityName, MoveListenerRegistry listenerRegistry, int count, boolean isLoggingRequired, MoveEventListener listener)
```
Если количество аргументов больше 3-4, то нужно как-то упрощать сигнатуру.  
Например, передавать объект-аргумент.  
*"ЧК", гл.3, "Объекты как аргументы"*

**17. class SimulationSettings**

- Нет оснований делать этот класс синглтоном.  

Вообще, синглетоны сейчас считаются антипаттерном.  
Но конкретно здесь использвание синглтона выглядит тем более неправильно, потому что возможно создание двух разных наборов настроек- 
через два разных фабричных метода.  
И конкретный синглтон будет создан в зависимости от того, какой из этих методов будет запущен первым
```java
public static SimulationSettings getInstance(int width, int length, int fillPercentage, Map<Class<? extends Entity>, ...) {
  if (instance == null) {
    instance = new SimulationSettings(...);
  }
  return instance;
}

public static SimulationSettings getInstance() {
  if (instance == null) {
    instance = new SimulationSettings();
  }
  return instance;
}
```

Вот к чему это приведет на практике.  
Я создам экземпляр дефолтных настроек с помощью первого фабричного метода.  
Потом создам экземпляр кастомных настроек с помощью второго фабричного метода.  
И это будет один и тот же екземпляр
```java
SimulationSettings firstSettings = SimulationSettings.getInstance();
System.out.println("firstSettings: " + firstSettings);

int width = 11;
int length = 22;
SimulationSettings secondSettings = SimulationSettings.getInstance( width, length, 3, new HashMap<>(), 1, 1, 1, 1, 1, 1, 1);
System.out.println("secondSettings: " + secondSettings);

System.out.println("Width: " + secondSettings.getSizeOfMap().width());
System.out.println("Length: " + secondSettings.getSizeOfMap().length());

//РЕЗУЛЬТАТ:
firstSettings: com.murlov.settings.SimulationSettings@6f496d9f
secondSettings: com.murlov.settings.SimulationSettings@6f496d9f
Width: 0
Length: 0
```

Ты скажешь, что так все и задумано- это же синглтон.  
А я скажу, что метод с параметрами `getInstance(int width, int length, ...)` не выполнил свой контракт- не создал экземпляр настроек с заданными мною значениями.  

- Просто адовое количество аргументов
```java
private SimulationSettings(int width, int length, int fillPercentage, Map<Class<? extends Entity>, Integer> minNumbersForEntityTypes, int herbivoreHealth,
    int herbivoreSpeed, int herbivoreSatiety, int predatorHealth, int predatorSpeed, int predatorSatiety, int predatorDamage)
```

Большое количество аргументов в методах потенциально опасно багами при использовании таких методов.  
Например, здесь много подряд идущих аргументов одного типа int.  
При создании объекта, можно легко перепутать передаваемые в конструктор аргументы местами.  
И тогда в поле `int herbivoreSatiety` вставится `int predatorHealth` и наоборот.

Вместо такого большого количества входяных параметров, нужно использовать либо объект-аргумент:
```java
SimulationSettings.Param param = new SimulationSettings.Param();
param.width = 1;
param.length = 2;
//...
SimulationSettings settings = new SimulationSettings(param);

//...
public class SimulationSettings {
  public SimulationSettings(Param param) {...}

  public class Param {
   public int width;
   public int length;
   //...
  }
}  
```

Либо паттерн Билдер:
```java
SimulationSettingsBuilder builder = new SimulationSettingsBuilder();  //или new SimulationSettings.Builder();
SimulationSettings settings = builder.width(1)
                                  .length(2)
                                  .fillPercentage(3)
                                  //oth's
                                  .build();
```
Если количество аргументов в методе больше 3-4, то нужно с этим что-то делать.  
*"ЧК", гл.3, "Объекты как аргументы"*  
*"ЧК", гл.17, F1*

- Нарушение SRP.

Объект не должен сам себя инициализировать.  
В данном случае класс настроек не только хранит настройки, но и самостоятельно инициализирует их с помощью каких-то сложных алгоритмов:
```java
private SimulationSettings(int width, int length, int fillPercentage, Map<Class<? extends Entity>, Integer> minNumbersForEntityTypes, int herbivoreHealth,
                             int herbivoreSpeed, int herbivoreSatiety, int predatorHealth, int predatorSpeed, int predatorSatiety, int predatorDamage) {
  this.fillPercentage = fillPercentage * 0.01;  <-- СЛОЖНЫЙ АЛГОРИТМ
  numberOfEntitiesPerEntityType = (int) (getNumberOfCells() * this.fillPercentage) / NUMBER_OF_ENTITY_TYPES;  <-- СЛОЖНЫЙ АЛГОРИТМ
  numberOfRemainingEntities = (int) (getNumberOfCells() * this.fillPercentage) - numberOfEntitiesPerEntityType * NUMBER_OF_ENTITY_TYPES;  <-- СЛОЖНЫЙ АЛГОРИТМ
  //...
}
```
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

**Переделай этот класс таким образом, чтобы:**  
Класс не должен быть синглтоном.  
Этот класс должен только хранить настройки.  
Заполнять его данными должен кто-то другой, а не он сам.  
Методы класса не должны принимать во входящие такое большое количество аргументов.

**18. class SimulationSettingsFactoryProvider**

Магические числа должны быть синхронизированы с теми классами, где прописаны размеры карты, скорость, сытость и т.д.
```java
System.out.println("""
  1. Использовать настройки по-умолчанию:
  - Размер карты — 10x10
  - Травоядные:
    - Здоровье — 10
    - Скорость передвижения  — 1
    - Сытость — 10
    ...
  """;
```

**19. class InputSimulationSettingsFactory implements SimulationSettingsFactory**

👍  Хороший универсальный метод для вода чисел в заданных диапазонах:
```java
private int input(String title, String failMessage, int min, int max)
```

**20. class RandomProvider**

Мотивация существования этого класса мне непонятна
```java
public class RandomProvider {
  private static final Random instance = new Random(1);

  public static Random getInstance() {
    return instance;
  }
}
```
Это сделано для экономии ресурсов, чтобы разные классы могли пользоваться одним и тем же экземпляром `Random instance`?  
Если да, то так стало хуже- усложнение проекта не окупилось такой экономией.

**21. interface Renderer**

👍 Интерфейс рендерера это хорошо.

Теперь можно делать разные рендереры для разных визуальных сред(консоль, интерфейс виндовс, http, матричный принтер etc)  
и разного отображения информации(цветной, черно-белый и пр.)

**22. class ConsoleRenderer implements Renderer**

+ 👍 Хорошо, что спрайты существ хранятся здесь, а не берутся из самих существ.

+ 👍 Правильное использование нестандартных имен x,y в качестве имен счетчиков вместо стандартных i,j.  
Это мелочь, но полезная- сразу видно, какие значения вставляются в координату.
```java
for (int y = 0; y < simulationMap.getSize().length(); y++) {
  for (int x = 0; x < simulationMap.getSize().width(); x++) {
    Coordinates coordinates = new Coordinates(x, y);
  }
}
```

- Нарушение SRP.

Класс должен только распечатывать информацию.  
Реализация паузы не имеет к этому процессу никакого отношения
```java
try {
  Thread.sleep(500);
} catch (InterruptedException e) {
  Thread.currentThread().interrupt();
}
```

Зачем нужна эта пауза?  
Чтобы распечатывать карту через заданные промежутки времени.  
За порядок выполнения действий общей игровой логики отвечает класс `Simulation`, 
вон там и должна лежать пауза.

**23. class Simulation**

+ 👍 Принимает достаточное количество зависимостей в конструктор.

- Если по ходу работы программы не планируется добавлять экшены в список, а это не планируется, то можно создать сразу неизменяемый список
```java
turnActions = new ArrayList<>()
turnActions.add(entitiesSpawnAction);
turnActions.add(entitiesMoveAction);

//ПРАВИЛЬНО:
turnActions = List.of(entitiesSpawnAction, entitiesMoveAction);
```

- Нарушение DRY, магические буквы, числа, слова. Вводи константы 
```java
if (input.equals("q")) {...}
System.out.println("Для начала/паузы симуляции используйте Enter. Для выхода введите q и прожмите Enter.");

//ПРАВИЛЬНО:
private final static String QUIT = "q";

if (input.equals(QUIT)) {...}
System.out.printf("Для начала/паузы симуляции используйте Enter. Для выхода введите '%s'.  %n", QUIT);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

- Нарушене DRY, дублирование кода
```java
executeInitActions();
executeTurnActions();

private void executeInitActions() {
  for (Action action : initActions) {
    action.execute(simulationMap);
  }
}

private void executeTurnActions() {
  for (Action action : turnActions) {
    action.execute(simulationMap);
  }
}

//ПРАВИЛЬНО (1):
executeActions(initActions);
executeActions(turnActions);

private void executeActions(List<Action> actions) {
  for (Action action : actions) {
    action.execute(actions);
  }
}

//ПРАВИЛЬНО (2):
executeInitActions();
executeTurnActions();

private void executeInitActions() {
  executeActions(initActions);
}

private void executeTurnActions() {
  executeActions(turnActions);
}

private void executeActions(List<Action> actions) {...}
```

- Два разных термина при описании одного и того же взаимодействия с энтером. Придерживайся единообразия
```java
System.out.println("Для начала/паузы симуляции используйте Enter. Для выхода введите q и прожмите Enter.");

//ПРАВИЛЬНО:
System.out.println("Для начала/паузы симуляции нажмите Enter. Для выхода нажмите 'q' и нажмите Enter.");
```

**24. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, предварительно подготовив для нее все енобходимые зависимости. Это хорошо.

## ВЫВОД

Архитектура немного поплыла, когда ты усложнил функционал и ввел в программу листнеры и возможность пользовательского конфигурирования при запуске программы.  
Но усложнения это не плохо, подобные эксперименты я привествую.

Тем не менее, такие усложнения предъявляют более высокие требования к архитектуре.  
Поэтому перераспредели методы и поля таким образом, чтобы они лучше соответствовали SOLID.
Особенно SRP и OCP.

В остальном, все более-менее норм.

n.141(302)  
#ревью #симуляция 