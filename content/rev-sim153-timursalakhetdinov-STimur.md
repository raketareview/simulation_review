Timur Salakhetdinov  
[https://github.com/STimur/Simulation]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Юзабилити.

При старте программы начинает быстро фигачить распечатку.  
Причем так быстро, что клянусь, я сначала не смог увидеть надпись
```java
(Press p - for pause/continue or q - to exit)
```
И только стопнув работу, заметил это.  
Поэтому сначала подумал, что в программе нет паузы/пуск.

При запуске игры должно появиться меню с перечнем команд.  
После ввода команды должно выполниться соответствующее действие. 

2. Нет возможности сделать только один ход, хотя это требование есть в ТЗ. 

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализована пауза/пуск во время работы(но нет сделать один ход)
+ 👍 Карта распечатывается ровно

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для избежания путаницы не называй пакеты, классы, интерфейсы так же, как называются классы и интерфейсы стандартной библиотеки.  
Есть стандартный интерфейс `Map`, не должно быть кастомного пакета с таким же названием
```java
class Map
```

- Точнее объясняй свои намерения.

Если метод называется "найти(существо)", то метод должен найти существо, а не его координату
```java
Position find(Entity entity)

//ПРАВИЛЬНО:
Position getPosition(Entity entity)
```

- Избыточный контекст.

При нейминге учитывай принятую практику.  
```java
void addIfValid(List<Position> positions, Position position)

//ПРАВИЛЬНО:
void add(List<Position> positions, Position position)
```

Нигде в стандартной библиотеке нет методов типа "addIfValid", "removeIfValid()" etc.  
Хотя, например, метод `ArrayList.remove(int index)` тоже удаляет данные, только если `int index` валиден.

Если в названии метода есть союзы "And", "Or", то это указание, что с методом что-то не так.  
Может быть, это просто неудачное название.  
А может быть, он делает несколько дел сразу, которые в названии метода объединены через эти союзы.

- Для стандартных действий используй стандартные названия.  

Слово "place" неоднозначное и его главное значение это "место", то есть, существительное.  
Глагол "помещать" для этого слова является второстепенным значением.  
В стандартной библиотеке java слово "place" не используется для методов, которые что-то куда-то помещают.

Если нужно что-то куда-то добавить или поместить, используй слова "put" или "add".  
При прочих равных, употребляй стандартные названия, чтобы всем были понятны твои намерения
```java
void place(Entity entity, Position position)

//ПРАВИЛЬНО: put, add
void put(Entity entity, Position position) //или add(...)
```

- Придерживайся единообразия
```java
public void removeEntity(Position p)
public Entity getEntity(Position position) 

//ПРАВИЛЬНО:
public void removeEntity(Position position)
public Entity getEntity(Position position) 
```

- В названия не нужно вставлять частицы "Of", "The", "Are" и т.д.

Это только делает названия более громоздкими.  
Если конечно там "Of" не используется в контексте valueOf()
```java
int numberOfTrees;
int numberOfRocks;

//ПРАВИЛЬНО (В КОНТЕКСТЕ ЗАДАЧИ):
int treeCount;
int rockCount;
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки.  
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```java
if ((x >= 0 && x < width) && (y >= 0 && y < height))
  positions.add(position);

for (Action action : initActions)
  action.execute(map);

//ПРАВИЛЬНО:
if ((x >= 0 && x < width) && (y >= 0 && y < height)) {
 positions.add(position);
}

for (Action action : initActions) {
  action.execute(map);
}
```
*"Oracle Java code conventions"*

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы. 

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов.  
Либо перенеси ее в третий класс и из первых двух классов обращайся к этим константам- эти данные должны быть синхронизированы между собой
```java
public class Main {

  public static void main(String[] args) throws InterruptedException {
    //...
    case "p" -> {...}  
    case "q" -> {...}  
  }
}

public class Simulation {

  public void nextTurn() {
    System.out.printf("Simulation turn %d (Press p - for pause/continue or q - to exit) \n", turns); 
    //...
  }
  //...
}   
```

*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**4. record Position(int x, int y)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```java
public record Position(int x, int y) {
}
```

**5. class Map**

- Не называй свои классы так же, как называются стандартные класс и интерфейсы java core.

Следовать ТЗ это хорошо. 
Называть свои пользовательские классы так же, как стандартные классы и интерфейсы- плохо.  
Второе перевешивает, потому что это приводит к постоянной путанице в коде.

Например, такой код будет всё время сбивать с толку, потому что придется всё время на нем останавливаться и думать, 
какой именно из Map'ов имеется в виду- стандартный или кастомный
```java
Map map = new Map(30, 15);
```

А сюда что передается- стандартный интерфейс `Map` или кастомный класc `Map`?
```java
public void makeMove(Map map) {...}
```

Чтобы компилятор понимал, про какой именно `Map` идет речь, приходится писать полный путь к стандартному интерфейсу Map через прописывание полного пути к этому интерфейсу
```java
public class Map {
  private final java.util.Map<Entity, Position> positions = new HashMap<>();  <-- Полный путь к интерфейсу "Map" чтобы отличать от кастомного класса "Map"
  //...
}
```

Из-за этого в том числе ацкий ад в сигнатуре методов
```java
private List<Position> buildPath(Position start, Position target, java.util.Map<Position, Position> parent) 
```

Для наименования классов придерживайся терминологии предметной области и среды разработки.  
Если они вступают в противоречие, жертвуй терминологией предметной области(в данном случае, это ТЗ).

Конкретно в этом случае вместо "Map", назови свой кастомный класс как-либо иначе: `Board`, `GameMap` etc. 

- Две коллекции- избыточно.

Две коллекции для хранения записей "существо-координата" и "координата-существо" это избыточно и делает код сложнее.  
Теперь при всех операциях с существом, нужно вносить изменения в два этих списка и следить за тем, чтобы информация в них была синхронизирована между собой.  
Для всех операций с существами в карте достаточно иметь только один список
```java
private final java.util.Map<Entity, Position> positions = new HashMap<>();  <-- ЭТО ЛИШНЕЕ 
private final java.util.Map<Position, Entity> entities = new HashMap<>();  <-- ЭТОГО ДОСТАТОЧНО
```
Наличие двух синхронизированных списков(прямой и обратный) могут сделать выполнение некоторых операций быстрее.  
Например, получение координаты существа:
```java
public Position find(Entity entity) {
  return positions.get(entity); <-- КООРДИНАТА ВОЗВРАЩАЕТСЯ МАКСИМАЛЬНО БЫСТРО
}
```
Но за это приходится заплатить более громоздким алгоритмом, дублированием хранимой информации, необходимостью дублировать данные в двух коллекциях.  

При этом на *реальном быстродействии* такая оптимизация никак не скажется- 
подавляющую часть времени программа будет пребывать в паузе. 
А выигрыш в 0.000000001 секунды при выполнении некоторых операций тоже невозможно заметить человеческим глазом.

**Преждевременная оптимизация это Антипаттерн.**

Оптимизировать алгоритмы для увеличения их быстродействия за счет ухудшения архитектуры нужно тогда, когда в этом есть реальная необходимость.
В противном случае, отдавай приоритет простоте алгоритма.

- Нарушение SRP, божественный класс, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
Найти все соседние точки, найти путь, определить ходящие существо по координате etc.

Наверное, для проекта в целом полезно иметь метод, который определяет по координате, находится ли там ходящее существо.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно знать, находится ли по координате ходящее существо или неходящее.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Creature> getCreatures()

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда креатур

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getBirds()`.

- Никогда не возвращай null
```java
private final java.util.Map<Position, Entity> entities = new HashMap<>();

public Entity getEntity(Position position) {
  return entities.get(position);  <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*


- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность
```java
public void removeEntity(Position p) {
  Entity entityToRemove = entities.remove(p);
  positions.remove(entityToRemove);
}

//ПРАВИЛЬНО:
public void removeEntity(Position position) {
  validate(position);  <-- Если координата не в пределах карты, то бросает исключение  
  Entity entityToRemove = entities.remove(p);
  positions.remove(entityToRemove);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Метод совершения хода в карте- нарушение SRP.  
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
Карта карта = new Карта(100, 100);
Заяц заяц = new Заяц();
карта.place(заяц, new Координата(0, 0));
карта.move(заяц, new Координата (99, 99));

/* class Карта */
public void move(Creature creature, Position newPosition) {
  Position oldPosition = positions.remove(creature);
  entities.remove(oldPosition);

  positions.put(creature, newPosition);
  entities.put(newPosition, creature);
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как?  
Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Это не объект, это контейнер функций.

В ООП каждый каждый класс должен из себя представлять какую-то внятную сущность.  
Этот класс таковым не является, это просто контейнер для функций в стиле процедурного программирования.

**6. Отсутствуют необходимые по ТЗ классы**

Отсутствуют необходимые по ТЗ классы: 
```java
Чеклист для самопроверки #
Проблемы и ошибки в коде:
Поиск пути
...Следует вынести это(поиск пути) в отдельный класс
```

Сейчас из-за того, что ответственность поиска пути находится в классе Карта, 
это ухудшает ООП проекта.  
И в том числе из-за этого делает Карту божественным классом.

Придерживайся требований ТЗ.  

**7. abstract class Entity и его простые наследники Tree/Rock/Grass**

👍 Идеально
```java
public abstract class Entity {
}

public class Tree extends Entity {
}
```

**8. abstract class Creature extends Entity**

- Не кешируй статус жизни существа, определяй его динамически
```java
public abstract class Creature extends Entity {
  private boolean isAlive;
  private int health;
  //...

  protected Creature(int speed, int health) {
    this.speed = speed;
    this.health = health;
    this.isAlive = true;
  }

  public void decreaseHealth(int units) {
    this.health -= units;
    if (health <= 0)
      this.isAlive = false;
  }

  public boolean isAlive() {
    return isAlive;
  }
  //...
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  private int health;
  //...

  protected Creature(int speed, int health) {
    this.speed = speed;
    this.health = health;
  }

  public void decreaseHealth(int units) {
    this.health -= units;
    if (health < 0)
      health = 0;  <-- Потому что существо не может быть мертвее мертвого
  }

  public boolean isAlive() {
    return health == 0;
  }
  //...
}
```

- Вся механика совершения хода должна находиться только в этом классе,
а не разделена между этим классом и Картой
```java
public abstract class Creature extends Entity {
  //...
  public void makeMove(Map map) {...}  <-- Механика совершения хода
}

public class Map {
  //...
  public void move(Creature creature, Position newPosition) {...}  <-- Механика совершения хода
}
```

**9. class Predator extends Creature**

Неправильная перегрузка конструктора
```java
public Predator() {
  super(1, 1);
  this.attack = 1;
}

public Predator(int speed, int health, int attack) {
  super(speed, health);
  this.attack = attack;
}

//ПРАВИЛЬНО:
private static final int DEFAULT_SPEAD = 1;
private static final int DEFAULT_HEALTH = 1;
private static final int DEFAULT_ATTACK = 1;

public Predator() {
  super(DEFAULT_SPEAD, DEFAULT_HEALTH, DEFAULT_ATTACK);
}

public Predator(int speed, int health, int attack) {
  super(speed, health);
  this.attack = attack;
}
```

**10. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(Map map);
}
```

**11. abstract class SpawnAction**

- Это не Экшен.

Этот класс не соответствует идее `Action` из ТЗ- он даже не имплементирует `interface Action`.  
Поэтому он в своем названии не должен содержать слово "Action".  
Экшеном может являться только тот класс, который реализует `interface Action` напрямую или через своего предка.

- Нарушение инкапсуляции.

Если методы предназначены только для использования потомками, они должны быть protected
```java
public int getNumberOfTrees() 
public int getNumberOfRocks() 
public int getNumberOfPredators() 
public int getNumberOfHerbivores() 
public int getNumberOfGrass() 
```

- По той же причине константы здесь должны быть protected
```java
public static final double PERCENT_OF_TREES = 0.02;
public static final int HERBIVORES_TO_PREDATORS_FACTOR = 2;
public static final int GRASS_TO_HERBIVORES_FACTOR = 2;
```

- По факту, этот метод или спавнит или не спавнит. Оч. странная логика работы
```java
private boolean spawnEntity(Map map, Supplier<? extends Entity> factory) {
  int x = random.nextInt(map.getWidth());
  int y = random.nextInt(map.getHeight());
  Position spawnPosition = new Position(x, y);

  if (map.getEntity(spawnPosition) == null) {
    map.place(factory.get(), spawnPosition);
    return true;
  }
  return false;
}

//ПРАВИЛЬНО:
private void spawnEntity(Map gameMap, Supplier<? extends Entity> factory) {
  Position position = getRandomFreePosition(map);
  Entity entity = factory.get();
  gameMap.place(entity, position);
}

private Position getRandomFreePosition(Map gameMap) {
  for(int i = 0; i < MAX_ATTEMPT; i++)  {
    //создать случайную координату и вернуть её, если она свободна
  }
  //бросить исключение- пустая координата в карте не найдена
}
```

**10. class Renderer**

+ 👍 Спрайты существ хранятся здесь, а не берутся из самих существ, это хорошо.

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < height; i++) {
  for (int j = 0; j < width; j++) {
    printPosition(j, i, map);
  }
}

//ЛУЧШЕ:
for (int y = 0; y < height; y++) {
  for (int x = 0; x < width; x++) {
    printPosition(x, y, map);
  }
}
```

- Избыточно
```java
case Rock r -> System.out.print("🪨");

//ПРАВИЛЬНО:
case Rock -> System.out.print("🪨");
```

**12. class Simulation**

👍 Норм.

**13. class Main**

- Нарушение SRP.

Main должен только сконфигурировать зависимости и запустить программу.  
Управлять работой программы этот класс не должен.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

Определи сущности, которые сейчас сидят в этом классе и вынеси их в отдельные классы.  
Например, сущность, которая управляет симуляцией, получая от юзера команды пауза/пуск.

- Не бросай исключения в космос.

Метод main - последняя точка в программе, после которой исключение аварийно закрывает работу программы.  
Если ты знаешь, что в программе могут возникнуть исключения, то обработай их внутри программы
```java
public static void main(String[] args) throws InterruptedException
```
В крайнем случае хотя бы просто перехвати, напечатай сообщение "Критическая ошибка, работа программы будет прекращена" и корректно закрой программу.  
Сообщение без конкретики- не самый лучший вариант.  
Но это лучше, чем если программа вылетит с красными эксепшенами и непонятными сообщениями.

## ВЫВОД

На примере божественноой Карты не видно понимания принципа единой ответственности.  
Посмотреть на ютубе ролики Немчинского про SOLID и SRP в частности.  

При написании проектов соблюдай ТЗ.

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.153(326)  
#ревью #симуляция 