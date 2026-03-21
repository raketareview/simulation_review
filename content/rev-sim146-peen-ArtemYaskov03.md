https://github.com/ArtemYaskov03/simulation2  
[Peen]

Стоит всё переделать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Длинные команды: "play", "pause" etc 
```
Запустить симуляцию: play; Сделать один ход: one; Cелать паузу: pause; Возобновить симуляцию: resume; Завершить: stop;
```
Ввод команд и рисование карты используют одну и ту же консоль.  
Поэтому игрок между циклами распечатки карты не всегда успевает набрать и ввести длинные команды.  
Делай короткие команды, например: 0/1. 

2. Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim146/img0.png) 

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализованы пауза/пуск во время работы(но не факт- не удалось проверить из-за длинных команд)
+ 👍 Алгоритм поиска A*
+ 👍 Меню для создания карт разного размера

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов нужно писать стилем all lowercase
```java
package Simulation;

//ПРАВИЛЬНО:
package simulation;
```

- Нарушение конвенции кода.  
С большой буквы пишутся только классы.  
Методы пишутся стилем camelcase
```java
public List<Coordinates> A()

//ПРАВИЛЬНО:
public List<Coordinates> a()
```

- Метод не возвращает координату существа, он возвращает существ с их координатами в виде мапы
```java
private final Map<Entity, Coordinates> entityCoordinates = new HashMap<>();

public Map<Entity, Coordinates> getEntityCoordinates() {
  return this.entityCoordinates;
}

//ПРАВИЛЬНО:
private final Map<Entity, Coordinates> entitiesWithCoordinates = new HashMap<>();

public Map<Entity, Coordinates> getEntitiesWithCoordinates() {
  return this.entitiesWithCoordinates;
}
```

- Венгерская нотация это плохо.  
Венгерская нотация, которая обманывает- еще хуже.  
Эта переменная не принадлежит к типу List
```java
Map<Coordinates, Way> closeList = new HashMap<>();
```

- Это не константы, это обычные поля класса
```java
public class Astar {
  final int HEIGHT;
  final int WIDTH;
  //...
}

//ПРАВИЛЬНО:
final int height;
final int width;
```

- Название метода должно быть глаголом в повелительном наклонении.  
Эти названия- существительные
```java
void commanding()
void startingGame()
void oneStepSimulation()
//и много других
```

- Транслит это ужасно.  
Если не знаешь, как слово будет на английском, используй переводчик.  
Название метода должно быть глаголом, это название- существительное
```java
void statistika() {
  //печатает статистику  
}
```

- Назвать так переменную- все равно, что никак её не назвать.  
```java
boolean a = true;
```
Однобуквенные названия допустимы в качестве названий счетчиков циклов. В остальных случаях- нет.  
Давай переменным осмысленные имена, которые будут объяснять назначение переменной.

- Это дважды плохо
```java
List<Coordinates> steps = a.A();
```

- Геттеры называй геттерами
```java
public Entity whoHere(Coordinates entityCoordinates)

//ПРАВИЛЬНО:
public Entity getEntity(Coordinates coordinates)
```

- Сеттеры называй сеттерами
```java
public void newHp(int hp) {
  this.hp = hp;
}

//ПРАВИЛЬНО:
public void setHp(int hp) {
  this.hp = hp;
}
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```java
ArrayList<Coordinates> getNeighbors(Coordinates coordinates)

//ПРАВИЛЬНО:
List<Coordinates> getNeighbors(Coordinates coordinates)
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```java
System.out.print("Введите высоту поля (мин 10 макс 100):");
if (h < 10 || h > 100) {...}

//ПРАВИЛЬНО:
private final static int MIN_HEIGHT = 10;
private final static int MAX_HEIGHT = 100;

System.out.printf("Введите высоту поля (мин %s макс %s):", MIN_HEIGHT, MAX_HEIGHT);
if (h < MIN_HEIGHT || h > MAX_HEIGHT) {...}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**4. Шутейки в коде**

При командной работе шутейки в коде раздражают.  
Пользователь программы тоже может не разделять твое чувство юмора
```java
System.out.println("hz");  <-- печатается при вводе неизвестной команды
```

**5. Упрощай сложные условия в if'ах**

Вводи вспомогательные методы или поясняющие переменные, которые своим названием будут объяснять, что тут происходит
```java
if (worldMap.whoHere(newCoord) == null && steps.size() > this.speed) {...}

//ПРАВИЛЬНО, ПОЯСНЯЮЩАЯ ПЕРЕМЕННАЯ:
boolean isВсеОбъясняет = worldMap.whoHere(newCoord) == null && steps.size() > this.speed;
if (isВсеОбъясняет) {...}

//ПРАВИЛЬНО, ВСПОМОГАТЕЛЬНЫЕ МЕТОД:
if (isВсеОбъясняет(...)) {...}

private boolean isВсеОбъясняет(...) {
  return worldMap.whoHere(newCoord) == null && steps.size() > this.speed;
}
```

**6. Приоритет- простой код и хорошая читаемость**

Не соединяй несколько инструкций в одну просто потому, что можешь.  
Приоритет- простота и читаемость кода, а не экономия строчек
```java
worldMap.getEntityCoordinates().remove(worldMap.whoHere(steps.getLast()));  <-- понять это почти невозможно

//ПРАВИЛЬНО:
Coordinates coordinates = steps.getLast();
Entity entity = worldMap.whoHere(coordinates);

Map<Entity, Coordinates> entityWithCoordinates = worldMap.getEntityCoordinates();
entityWithCoordinates.remove(entity);
```

**7. В проекте системно нарушается инкапсуляция**

В некоторых классах все методы публичные. 
В других- все поля публичные.

Это делает классы уязвимыми для повреждения их внутреннего состояния через неправильное использование клиентами тех методов,
которые не прредназначены для пользования клиентами.  

Публичными должны быть только те методы и поля, которые предназначены для пользования клиентами.  
Доступ `protected` должен быть у тех методов и полей, которые предназначены только для пользования классами-потомками.  
Все остальные методы и поля должны быть `private`.

**8. record Coordinates(int high, int width)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

- Темрмины "ширина" и "высота" означают размеры, а не положение точки в пространстве
```Java
record Coordinates(int high, int width) 

//ПРАВИЛЬНО:
record Coordinates(int x, int y) //или row/column
```

**9. class WorldMap**

- Это карта.  

В географических картах первичны координаты- именно они выступают в роли ключа для поиска объектов на карте
```java
private final Map<Entity, Coordinates> entityCoordinates = new HashMap<>();

//ПРАВИЛЬНО:
private final Map<Coordinates, Entity> coordinatesWithEntities = new HashMap<>();
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```java
public Entity whoHere(Coordinates entityCoordinates) 
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Очень авангардное название
```java
public Entity whoHere(Coordinates entityCoordinates) {
  //по координате возвращает entity или null  
}

//ПРАВИЛЬНО:
public Entity getEntity(Coordinates entityCoordinates) 
```

- Никогда не возвращай null
```java
public Entity whoHere(Coordinates entityCoordinates) {
  //...  
  return null; 
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Рекурсия
```java
public void put(Entity entity) {
  //...
  if (entityCoordinates.containsValue(coordinatesEntity)) {
    put(entity);
  } 
}    
```
Рекурсия должна использоваться только там, где она оптимальна алгоритмически.
Например, при обходе бинарного дерева.

В других ситуациях, где рекурсия может быть замененена циклом, она должна быть им замененена.  
Рекурсия тяжело читается и при определенных условиях может привести к переполнению стека.  
Убери рекурсию, замени ее использование на цикл while().

Условный пример
```java
//❌ ПЛОХО:
void start() {
  printMessage("Введите команду: 0-выйти, 1-играть");  
  int command = inputCommand();

  if(command == 0) {
    return;
  } 

  if(command == 1) {
    playGame();
  } 

  start();  <-- РЕКУРСИЯ
}

//✔️ ХОРОШО:
void start() {
  while(true) {
    printMessage("Введите команду: 0-выйти, 1-играть");  
    int command = inputCommand();

    if(command == 0) {
      return;
    } 

    if(command == 1) {
      playGame();
    } 
  }  
}
```

- Этот метод не просто помещает существо на координату.  
Он помещает существо на случайную пустую координату.  
Помещать существо нужно на конкретную координату
```Java
public void put(Entity entity) {
  Random random = new Random();
  //помещает entity на случайную пустую координату
}

//ПРАВИЛЬНО:
public void put(Entity entity, Coordinates coordinates) {
  //помещает entity на coordinates
}
```
Если клиенту карты нужно поместить существо на случайную координату, то клиент должен самостоятельно найти случайную пустую кординату на карте.   
И поместить на нее существо с помощью метода `put(Entity entity, Coordinates coordinates)`.

- Инкапсуляция протекает- внутренности карты открыты для произвола
```java
private final Map<Entity, Coordinates> entityCoordinates = new HashMap<>();

public Map<Entity, Coordinates> getEntityCoordinates() {
  return this.entityCoordinates;
}
```

Нарушение чеклиста ТЗ:
```java
Класс Map
Проблемы и ошибки в коде:
Недостаточная инкапсуляция - “протекающее” наружу внутреннее устройство класса Map (например, прямой доступ к коллекции ячеек) 
вместо набора методов с говорящими названиями (AddEntity, RemoveEntity, и так далее)
```

В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```java
Карта карта = new Карта();
<заселить карту существами>
карта.getEntities().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

- Нарушения SRP.

Карта должна хранить существа по координатам и обеспичивать к ним базовые операции доступа:  
поместить существо на координату, удалить существо по координате, вернуть существо по координате, вернуть координату по существу etc.

Сейчас большинства из этих операций в Карте нет, а значит, клиенты юзают Карту через задний ход.  
А именно- в нарушение инкапсуляции, получают ее внутренности методом `Map<Entity, Coordinates> getEntityCoordinates()`.

**10. class Astar**

- Нарушение инкапсуляции.  
Всегда явно указывай область видимости и final, если поле не изменяется
```java
WorldMap worldMap;
Entity entity;

//ПРАВИЛЬНО:
private final WorldMap worldMap;
private final Entity entity;
```

- В классе много публичных методов. Поэтому неясно, как пользоваться этим поиском.

Если этот класс ищет путь, то в нем должен быть только один публичный метод(непубличных- сколько угодно).  
Этот единственный публичный метод должен искать путь.  
Для алгоритма А* это должно быть примерно так:
```java
public List<Coordinates> find(Карта карта, Coordinates from, Coordinates to) {...}
```

- Нарушение SRP.

Чтобы алгоритму А* искать путь между двумя точками, ему не нужно хранить в себе меню для существ
```java
if (entity instanceof Predator) {
  for (Entity e : worldMap.getEntityCoordinates().keySet()) {
    if (e instanceof Herbivore) {  
      typeFoodList.add(e);
    }
  }
  return typeFoodList;
}
```
Если поиск пути анализирует существ и подбирает для них еду, значит поиск занят чем-то не тем, чем должен по своей единой ответственности.  
Все, что должен делать класс поиска А*- искать путь на карте между двумя точками.

- Такие длинные строки нечитаемы в принципе, тут 177 символов:
```java
closeList.put(worldMap.getEntityCoordinates().get(e), new Way(worldMap.getEntityCoordinates().get(e), null, 0, searchH(worldMap.getEntityCoordinates().get(e), finish)));
```
А тут их 244, реально строка не влазит в монитор
```java
int steps = Math.abs(worldMap.getEntityCoordinates().get(entity).high() - worldMap.getEntityCoordinates().get(e).high()) + Math.abs(worldMap.getEntityCoordinates().get(entity).width() - worldMap.getEntityCoordinates().get(e).width());
```
Вводи поясняющие переменные.  
*Фаулер, "Рефакторинг", гл.6, "Введение поясняющей переменной".*

**11. abstract class Entity и его простые наследники Tree/Rock/Grass**

+ 👍 Идеально
```java
public abstract class Entity {
}

public class Grass extends Entity {
}
```

**12. abstract class Creature extends Entity**

- Нарушение инкапсуляции

Эти поля не должны быть публичными
```java
public abstract class Creature extends Entity {
  public int speed;
  public int hp;

  public abstract void makeMove(WorldMap worldMap);
}
```
Все поля должны быть приватными, а доступ к ним- только через геттеры и сеттеры.  
Для прямого использования потомками, некоторые поля могут быть `protected`.  
*Блох "Java. Эффективное программирование", изд.3, гл.4.2*

С точки зрения ООП, сейчас это не класс, это- структура.  
*"ЧК", гл.6*

- Значения необходимых полей в классе при создании объекта должно устанавливаться через его конструктор.  
Тем более, если этот класс задумывается, как часть сложной иерархии
```java
public abstract class Creature extends Entity {
  public int speed;
  public int hp;

  public abstract void makeMove(WorldMap worldMap);
}

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  protected final int speed;
  protected int hp;

  public Creature(int speed, int hp) {
    this.speed = speed;
    this.hp = hp;
  }

  public abstract void makeMove(WorldMap worldMap);
  //...
}
```
*Эккель "Философия Java", гл.5, "Конструктор гарантирует инициализацию"*

**13. class Predator extends Creature**

Несмотря на то, что кода в классе очень мало, читается он очень плохо.  
Здесь 8(ВОСЕМЬ) инструкций скручены в одну. Понять, что здесь происходит, почти невозможно
```java
((Herbivore) worldMap.whoHere(steps.getLast())).newHp(hit(((Creature) worldMap.whoHere(steps.getLast()))));
```

Понять это условие почти невозможно
```java
if (worldMap.whoHere(steps.getLast()) instanceof Herbivore && worldMap.whoHere(newCoord) == null) {...}
```

**14. class Herbivore extends Creature**

- Нарушение DRY.

Код в методе `makeMove(...)` частично повторяет аналогичный метод в Predator.  
Это нарушение чеклиста ТЗ:
```java
Проблемы и ошибки в коде:
Дублирование кода между классами Herbivore и Predator
```
Общий код выноси в общего предка.

**15. class Wolf extends Predator**

Нарушение дизайна ООП. 

При создании экземпляра класса, значения обязательных полей предка должны устанавливаться через конструктор, а не сетиться  
```java
public class Wolf extends Predator {
  public Wolf() {
    speed = 3;
    hp = 15;
    setDamage(5);
  }
}

//ПРАВИЛЬНО:
public class Wolf extends Predator {
  private final static int SPEED = 3;
  //...

  public Wolf() {
    super(SPEED, HP, DAMAGE);
  }
}
```
*Эккель "Философия Java", гл.5, "Конструктор гарантирует инициализацию"*


**16. class Actions**

- Нарушение конвенции кода.  
Поля вперемешку с методами, вспомогательные приватные поля стоят вышет главных публичных. 

- Идея `Action` здесь не осмыслена. 

Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.  
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.  
Action'ы, изложенные в ТЗ, это вариант реализации паттерна Command. 

Сейчас `Actions` у тебя это просто сборник функций, сделанный в стиле процедурного программирования
```java
public class Actions {
  public void startingGame() {...}
  public void oneStepSimulation() {...}
  //и еще миллион публичных методов
}
```

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.  
В каждом экшене должен быть только один публичный метод(не публичных может быть сколько угодно).  
Это вариация паттерна Command- экшены должны быть родственны и одинаково использоваться через полиморфизм.  
Примерно так:
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

- Сейчас это божественный класс, который делает абсолютно разные вещи.

Он устанавливает размеры карты, выполняет ход симуляции, делает паузу и стоп симуляции и бог знает что еще.  
В ТЗ написано:
```java
Action - действие, совершаемое над миром. Например - сходить всеми существами. 
Это действие итерировало бы существ и вызывало каждому makeMove(). 
Каждое действие описывается отдельным классом и совершает операции над картой. 
```
Сейчас класс Actions не соответствует ТЗ.

Из ТЗ следует, что:  
🔹 Action'ы должны производить действия над Картой. А не над Симуляцией, диалогами с пользователем или чем-то иным.  
🔹 Каждое действие над Картой должен выполнять ОДИН класс Action.

- Нарушение DRY.

Куча дублированного кода. Например, эти два метода делают одно и то же
```java
private int enterHeight() {
  //получает от юзера число в заданном диапазоне
}

private int enterWidth() {
  //получает от юзера число в заданном диапазоне
}
```
Общий код выноси во вспомогательные методы.

**17. class Simulation**

Класс не соответствует ТЗ
```java
public class Simulation {
  static void main(String[] args) {
    Actions actions = new Actions();
    actions.commanding();
  }
}
```

## ВЫВОД

Стиль программирования- процедурный.  
ТЗ игнорируется.

Не видно понимания ООП на базовом уровне: зачем нужны геттеры, сеттеры, конструкторы.  
Что такое инкапсуляция, полиморфизм, наследование и как их использовать в программе.  
Фактически, вся архитектура программы построена на нарушении инкапсуляции.

Почитай статьи, посмотри видео на ютубе по этим основам ООП.  
Может быть, сделай пару простых упражнений на наследование и полиморфизм.

Подробнее разберись с Action'ами, посмотри ролики на ютубе про паттерн "Command".

Не видно и понимания декомпозиции- как делить программу на классы.  
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы.  
Посмотреть ролики Немчинского про SOLID.

Сравнить различия между ООП и процедурным стилями:  
Стрим Сергея [Крестики-нолики в процедурном стиле](https://www.youtube.com/watch?v=PPikj1qHxrA)  
Мой стрим [Крестики-нолики в ООП стиле](https://t.me/zhukovsd_it_chat/53243/187097)

Всё переделать с нуля, рефакторинг тут не поможет.  
Делай по ТЗ.

Начать стоит с изучения "Oracle Java code conventions", есть русские переводы.  
Возможно, для начала стоит написать первый проект "Виселица".

n.146(308)  
#ревью #симуляция 