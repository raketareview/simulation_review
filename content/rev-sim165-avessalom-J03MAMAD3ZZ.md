https://github.com/J03MAMAD3ZZ/Simulation  
[Авессалом Владимирович Изнурёнков]

Нет старт/стоп во время работы симуляции.  
В остальном неплохо.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. В программе не реализована работа с потоками и выполнение старт/стоп во время работы симуляции.  
Хотя это требование есть в ТЗ.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Широко применяются интерфейсы.
+ 👍 Меню для создания карт разного размера

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название метода должно быть глаголом в повелительном наклонении.

По правилам английского языка, в предложении сначала пишется глагол, потом дополнение 
```java
public Coordinate coordinateShift(Coordinate coordinate) <-- Перевод: "Сдвиг координаты"; "сдвиг" это существ.

//ПРАВИЛЬНО:
public Coordinate shiftCoordinate(Coordinate coordinate) <-- Перевод: "Сдвинуть координату"; "сдвинуть" это глаг.
```

- Не дублируй имя класса в названии метода класса
```java
public record Coordinate(int x, int y) {
  public Coordinate coordinateShift(Coordinate coordinate) {
}

//ПРАВИЛЬНО:
public record Coordinate(int x, int y) {
  public Coordinate shift(Coordinate coordinate) {
}
```

- Если в проекте есть класс/интерфейс `Direction`, то все переменные с именем, включающим это название, должны быть экземплярами этого класса.  

Когда разные концепции называются одним и тем же именем, это приводит к путанице
```java
public interface Direction {
  public List<Coordinate> getDirection(); <-- Судя по названию метода, он должен вернуть имплементацию интерфейса Direction
}

//ПРАВИЛЬНО:
public interface Direction {
  public List<Coordinate> getCoordinates();
}
```

- Названия пакетов нужно писать стилем alllowercase 
```java
package com.simulation.entity.inanimateEntity;

//ПРАВИЛЬНО:
package com.simulation.entity.inanimateentity;
```

- Аббревиатуры в названиях лучше писать так же, как и другие слова
```java
class BFSPathFinder

//ЛУЧШЕ:
class BfsPathFinder
```
Например, какое название понятнее: HTTPURL или HttpUrl?  
*Блох "Java. Эффективное программирование", изд.3, гл.9.12.*

- Избыточно. Мы и так понимаем, что метод интерфейса Поиска Пути найдет путь 
```java
public interface PathFinder {
  List<Coordinate> findPath(...);
}

//ПРАВИЛЬНО:
public interface PathFinder {
  List<Coordinate> find(...);
}
```

- Избыточно
```java
public interface FieldRenderer {
  void renderField(Field field);
}

//ПРАВИЛЬНО:
public interface FieldRenderer {
  void render(Field field);
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. record Coordinate(int x, int y)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

+ 👍 Метод сдвига не нарушает SRP, пусть будет
```java
public record Coordinate(int x, int y) {
  public Coordinate coordinateShift(Coordinate coordinate) {...}
}
```

**3. interface Direction**

+ 👍 Интерфейс, который возвращает координаты направлений
```java
public interface Direction {
  public List<Coordinate> getDirection();
}
```
Благодаря этому интерфейсу, можно использовать в программе разные направления движения существ: на 4 или 8 сторон.

- Писать `public` в методах интерфейсов- избыточно.  
Методы в интерфейсах по умолчанию и так `public`. 

**4. class SquareDirection implements Direction**

Имплементация для движения по 4 сторонам, как ладья в шахматах.

- Куча мусора- закомментированный код.

- Если используется только конструктор по умолчанию, не объявляй его явно.

Если в классе используется только конструктор по умолчанию, то не нужно его прописывать. 
```java
public class SquareDirection implements Direction {

  public SquareDirection() {  <-- Это явно объявленный конструктор по умолчанию
  }
}

//ПРАВИЛЬНО:
public class SquareDirection implements Direction {

  //нет явно объявленного конструктора- используется конструктор по умолчанию 
  //...
}
```

**5. class Field**

+ 👍 Самое лучшее название и реализация метода, который возвращает мапу из Карты 
```java
public Map<Coordinate, Entity> toMap() {
  return new HashMap<>(entities);
}
```
Нужен ли в Карте такой метод в принципе- это дискуссионно.  
Но сделан метод идеально. Так что пусть будет.

Из плюсов: благодаря этому методу алгоритм класса-мувера работает быстрее. 

+ 👍 Полезный метод
```java
public boolean isInBounds(Coordinate coordinate) {
  //проверяет вхождение координаты в Карту    
}
```

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.

Сейчас в карту можно вставить существо на координату, выходящую за размер карты.  
Если координата некорректна (находится вне пределов карты), нужно бросать исключение:
```java
public void addEntity(Coordinate coordinate, Entity entity) {
  entities.put(coordinate, entity);
}

//ПРАВИЛЬНО:
public void addEntity(Coordinate coordinate, Entity entity) {
  validate(coordinates);  <-- Если координата вне пределов карты, бросает исключение
  entities.put(coordinate, entity);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, то бросается исключение.

+ 👍 Геттеры не возвращают null, они возвращают Optional, это хорошо.

+ 👍 С точки зрения SRP класс хороший, в нем нет лишних ответственностей.

**6. interface PathFinder**

+ 👍 Интерфейс поиска пути- это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
public interface PathFinder {
  List<Coordinate> findPath(Field field, Coordinate start, Class<? extends Entity> target);
}
```

**7. class BFSPathFinder implements PathFinder**

- Нарушение SRP, DI.

Если в проекте есть интерфейс `Direction`, который предоставляет направления движения, то эта зависимость должна внедряться сверху.  
Если есть отдельный интерфейс с направлениями, то класс поиска пути не должен самостоятельно принимать решение, по каким направлениям будет происходить движение.  
Эти данные должны прилететь ему в конструктор
```java
public class BFSPathFinder implements PathFinder {
  private final SquareDirection direction = new SquareDirection();
  //...
}

//ПРАВИЛЬНО:
public class BFSPathFinder implements PathFinder {
  private final Direction direction;

  public BFSPathFinder(Direction direction) {
    this.direction = direction;
  }
  //...
}
```

Внедрять эту зависимость нужно примерно с уровня Main/  
В простейшем случае без использования потоков это должно быть примерно так:
```java
public class Main {
  void main() {
    Direction direction = new SquareDirection();
    PathFinder pathFinder = new BfsPathFinder(direction);
    //...
    Game game = new Game(pathFinder, ...);
    game.start();
  }
}
```
Это даст возможность делать разные Main-конфигурации, не меняя код в остальных классах:
```java
public class SecondMain {
  void main() {
    Direction direction = new EightDirection();
    PathFinder pathFinder = new AStarPathFinder(direction);
    //...
    Game game = new Game(pathFinder, ...);
    game.start();
  }
}
В проекте может быть сколько угодно разных Main-классов.

- Нарушение принципа минимального открытого интерфейса.

В классе поиска пути должен быть только один публичный метод- тот, который ищет путь
```java
public List<Coordinate> findPath(...) {
  //...
  return pathReconstruction(parentMap, start, current);
}

public List<Coordinate> pathReconstruction(...) {...}

//ПРАВИЛЬНО:
public List<Coordinate> findPath(...) {
  //...
  return pathReconstruction(parentMap, start, current);
}

private List<Coordinate> pathReconstruction(...) {...}
```
Клиент должен видеть только те методы, которые ему нужны для выполнения поставленной задачи.  
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

+ 👍 В целом класс хороший.

**8. abstract class Entity и его простые наследники Tree/Rock/Grass**

+ 👍 В целом норм
```java
public abstract class Entity {
}

public class InanimateEntity extends Entity {
}

public class Tree extends InanimateEntity {
}
```

Хотя непонятно, зачем нужен промежуточный уровень абстракции `InanimateEntity`
```java
Entity
   △
   |
   +-------------------+
   |                   |
LivingEntity      InanimateEntity   
   △                   △
   |                    |
   |                    +---------+---------+
   |                    |         |         |
   +----------+       Rock      Tree      Grass
   |          |
Herbivore   Predator
```

`InanimateEntity` не дает своим потомкам никакого особого поведения:
```java
public class InanimateEntity extends Entity {
}
```

Если классы `Rock`, `Tree` и `Grass` нужно пометить каким-то общим типом, то для этого применяются [маркерные интерфейсы](https://javarush.com/groups/posts/1541-interfeysih---markerih):
```java
public interface InanimateEntity {
}

public class Tree extends Entity implements InanimateEntity {
}
```
Преимущества: нет лишнего уровня в иерархии классов.

**9. abstract class LivingEntity extends Entity**

Аналог класса `Herbivore`.  
Переименовано ради дихотомии `LivingEntity`/`InanimateEntity`, почему бы и нет.

- Нарушение DIP.

Если в проекте есть интерфейс `PathFinder`, то классы должны зависеть от этого интерфейса, а не от его конкретных реализаций
```java
public abstract class LivingEntity extends Entity {
  //...  
  
  public void makeMove(Field field, BFSPathFinder finder) {
    BFSPathFinder pathFinder = new BFSPathFinder();
    List<Coordinate> path = pathFinder.findPath(field, start, target);
    //...
  }
}

//ПРАВИЛЬНО ТАК:
public abstract class LivingEntity extends Entity {
  //...  

  public void makeMove(Field field, PathFinder finder) {
    List<Coordinate> path = pathFinder.findPath(field, start, target);
    //...
  }
}

//ТАК ТОЖЕ ПРАВИЛЬНО:
public abstract class LivingEntity extends Entity {
  private final PathFinder finder;
  //...

  public LivingEntity(PathFinder finder, ...) {
    this.finder = finder;
    //...
  }
     
  public void makeMove(Field field) {
    List<Coordinate> path = pathFinder.findPath(field, start, target);
    //...
  }
}
```

**10. Нарушение принципа минимального открытого интерфейса в LivingEntity и его наследниках**

В классах публичными должны быть только те методы, которые предназначены для использования клиентами.  
Если метод предназначен только для использования потомками, то метод должен быть `protected`.  
Остальные методы должны быть `private`.

Сейчас в LivingEntity и его наследниках все методы публичные. Даже те, которые не предназначены для использования клиентами. 

По умолчанию все методы должны быть приватными, если нет причин для того, чтобы сделать их не приватными.  
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

**11. interface Action**

👍 Идеально
```java
public interface Action {
  void makeAction(Field field);
}
```

**12. class MoveAction implements Action**

- Нарушение SRP, DIP.

Если муверу нужен Поиск пути, то он должен принять его в конструктор
```java
public class MoveAction implements Action {
  private final BFSPathFinder pathFinder = new BFSPathFinder();
  //...
}

//ПРАВИЛЬНО:
public class MoveAction implements Action {
  private final PathFinder pathFinder;

  public MoveAction(PathFinder pathFinder) {...}
  //...
}
```

**13. interface FieldRenderer**

👍 Интерфейс рендерера это хорошо
```java
public interface FieldRenderer {
  void renderField(Field field);
}
```

Теперь можно делать разные рендереры для разных визуальных сред (консоль, интерфейс виндовс, http, матричный принтер etc)  
и разного отображения информации(цветной, черно-белый и пр.)


**14. class ConsoleFieldRender implements FieldRenderer**

👍 В целом ок.

**15. public record SimulationConfig**

- Неправильное использование record'a.

Классы типа `record` предназначены для простого хранения данных.  
В них не должно быть такого сложного поведения, как в этом record'e.  
Если тебе нужен класс со сложным поведением, сделай обычный класс, а не record. 

- Какой-то мутный метод, который идёт вразрез с идеей рекорда как простого контейнера данных
```java
public record SimulationConfig(
    Field field,
    Map<EntityType, Integer> entitySpawnAmount,
    FieldRenderer renderer
) {

  public Map<EntityType, Integer> mapCopy() {  <-- ?
    return Map.copyOf(entitySpawnAmount);
  }

  //...
}
```

- Нарушение SRP.

Класс совмещает в себе контейнер с данными и Фабрику.  

Класс не должен сам себя инициализировать конкретными значениями по какой-то сложной логике
```java
public record SimulationConfig(
    Field field,
    Map<EntityType, Integer> entitySpawnAmount,
    FieldRenderer renderer
) {

  public Map<EntityType, Integer> mapCopy() {
    return Map.copyOf(entitySpawnAmount);
  }

  public static SimulationConfig fromPreset(FieldPreset preset) {
    Map<EntityType, Integer> amount = new EnumMap<>(EntityType.class);
    int area = preset.getHeight() * preset.getWidth();

    amount.put(EntityType.PREDATOR, Math.max(1, area / 50));
    amount.put(EntityType.HERBIVORE, Math.max(1, area / 25));
    //...
        return new SimulationConfig(
        new Field(preset.getHeight(), preset.getWidth()),
        amount,
        new ConsoleFieldRender()
    );
  }
}
```
Сейчас фабричный метод `fromPreset(...)` знает хитрые формулы с магическими значениями для `Predator`, `Herbivore` etc.  
Этот фабричный метод использует конкретную реализацию `ConsoleFieldRender`, поэтому метод зависит от конкретного представления- текстовой консоли.

То есть, при использовании в других визуальных средах (Андроид, Windows UI, аркадный автомат с лампочками), нельзя будет использовать этот фабричный метод.  

Класс нужно разделить на Фабрику и простой контейнер с данными
```java
//ПРАВИЛЬНО:
public record SimulationConfig(Field field, Map<EntityType, Integer> entitySpawnAmount, FieldRenderer renderer) {
}

public final class ConsoleSimulationConfigFactory {
  public static SimulationConfig create(FieldPreset preset) {...}    
}
```

**16. class Simulation**

- Экшены должны быть в списках для того, чтобы можно было быстро менять их количество в этих списках
```java
private final Action initAction;
private final List<Action> turnActions = List.of(new MoveAction());

//ПРАВИЛЬНО:
private final List<Action> initActions;
private final List<Action> turnActions = List.of(new MoveAction());

public Simulation(SimulationConfig config) {
  this.config = config;
  this.initActions = List.of(new WorldInitializeAction(config));
}
```

+ 👍 Симуляция принимает в себя объект-аргумент `SimulationConfig`.  
В принципе, почему бы и нет
```java
public Simulation(SimulationConfig config) {...}
```
Главное, чтобы в этом конфиге были все необходимые для Симуляции зависимости, в том числе `PathFinder`.  
И конфиг должен лежать в одном пакете с `Simulation`, а не где-то у чёрта на куличках.

Конфиг должен быть простым контейнером данных, поэтому для удобства его можно даже положить внутрь `Simulation`:
```java
public class Simulation {
  
  public Simulation(SimulationConfig config) {...}

  //...

  public record SimulationConfig(...) { 
  }
}
```

## ВЫВОД

В целом неплохо. Если не считать того, что ты скипнул работу с потоками и не сделал старт/стоп во время работы программы.   
Поэтому результат не соответствует ТЗ.   

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.165(350)  
#ревью #симуляция 