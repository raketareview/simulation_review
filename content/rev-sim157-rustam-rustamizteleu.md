https://github.com/rustamizteleu/Simulation  
[Rustam]

Есть над чем поработать.  
Нужно сделать многопоточность. 

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim157/img0.png)

2. В программе не реализована работа с потоками и выполнение старт/стоп во время работы симуляции.  
Хотя это требование есть в ТЗ.

## ХОРОШО

+ 👍 Координаты в существах хранятся начиная с Creature
+ 👍 Существа размножаются 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Это не сеттер.

Сеттер должен устанавливать значение какого-то конкретного поля.  
А этот метод не устанавливает конкретное существ, он ДОБАВЛЯЕТ существо в группу других существ:
```java
private HashMap<Coordinates, Entity> entities;

public void setEntity(Coordinates coords, Entity entity) {
  entities.put(coords, entity);
}

//ПРАВИЛЬНО:
public void putEntity(Coordinates coords, Entity entity)
```

Когда есть какое-то одно существо, то сеттер корректен. Условный пример:
```java
private Entity entity;

void setEntity(Entity entity) {
  this.entity = entity;  <-- Правильно, сеттер устанавливает конкретное значение
}
```

Когда существо добавляется к другим, это не сеттер. Условный пример:
```java
private List<Entity> entities;

void addEntity(Entity entity) {
  entities.add(entity);  <-- Это не сеттер, потому что добавляет новый объект в список других
}
```

- Не сокращай названия без особой необходимости
```java
public Entity getEntity(Coordinates coords) 

//ПРАВИЛЬНО:
public Entity getEntity(Coordinates coordinates) 
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```java
HashMap<Coordinates, Coordinates> cameFrom = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Coordinates> cameFrom = new HashMap<>();
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**3. Нарушение конвенции кода**

- Скобочки.

В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки.  
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```java
if (nextStep == null) return;

//ПРАВИЛЬНО:
if (nextStep == null) {
  return;
}
```
*"Oracle Java code conventions"*  

**4. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо.

- Класс может быть преобразован в record без потери функционала.

Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**5. class Map**

- Не называй свои классы так же, как называются стандартные класс и интерфейсы java core.  
Класс не должен называться "Map" - это название стандартного интерфейса.

Следовать ТЗ это хорошо. 
Называть свои пользовательские классы так же, как стандартные классы и интерфейсы- плохо.  
Второе перевешивает, потому что это приводит к постоянной путанице в коде.

Например, такой код будет всё время сбивать с толку, потому что придется всё время на нем останавливаться и думать, 
какой именно из Map'ов имеется в виду- стандартный или кастомный
```java
Map map = new Map(30, 15);
```

Для наименования классов придерживайся терминологии предметной области и среды разработки.  
Если они вступают в противоречие, жертвуй терминологией предметной области(в данном случае, это ТЗ).

Конкретно в этом случае вместо "Map", назови свой кастомный класс как-либо иначе: `Board`, `GameMap` etc. 

- Никогда не возвращай null
```java
private HashMap<Coordinates, Entity> entities;


public Entity getEntity(Coordinates coords) {
  return entities.get(coords); <-- может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Сейчас можно вставить существо на координату, которая находится за пределами карты
```java
public void setEntity(Coordinates coords, Entity entity) {
  entities.put(coords, entity);
}

//ПРАВИЛЬНО:
public void setEntity(Coordinates coordinates, Entity entity) {
  validate(coordinates);  <-- Бросает исключение, если координата за пределами карты
  entities.put(coords, entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

+ 👍 С точки зрения SRP класс хороший, в нем нет лишних ответственностей.  
Но не хватает метода `Coordinates getCoordinates(Entity entity)`.

**6. class PathFinder**

- Этот класс не ищет путь.

Путь это последовательность точек от точки старта до точки финиша.  
Этот класс не ищет путь. Его единственный публичный метод возвращает одиночную координату
```java
public Coordinates findNextStep(Coordinates from, Class targetType, Map map)
```

Этот метод ищет не путь, а один шаг пути. Тоже хорошо, но не то.

Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритма BFS или A*.  
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
    //потом прокладывает путь по алгоритму A* между точкой стартой и точкой цели
  }
}
```
Плюсы- унификация сигнатуры разных алгоритмов поиска для полиморфизма.  
То есть, можно сделать семейство родственных классов поиска, объединенных общим интерфейсом.

- Никогда не возвращай null
```java
public Coordinates findNextStep(Coordinates from, Class targetType, Map map) {
  //...
  return null;
}
```

- Используй константы
```java
public Coordinates findNextStep(...) {
  //...
  Coordinates right = new Coordinates(current.getX() + 1, current.getY());
  Coordinates left = new Coordinates(current.getX() - 1, current.getY());
  Coordinates down = new Coordinates(current.getX(), current.getY() + 1);
  Coordinates up = new Coordinates(current.getX(), current.getY() - 1);

  for (Coordinates neighbor : List.of(right, left, down, up)) {...}
}

//ПРАВИЛЬНО:
private static final List<Coordinates> SHIFT_COORDINATES = List.of(new Coordinates(0, 1), new Coordinates(0, -1), ...);

public Coordinates findNextStep(...) {
  //...

  for (Coordinates shift : SHIFT_COORDINATES) {
   Coordinates coordinates = //current + shift
   //...
  }
}
```

- Не делай таких ацких условий в if'ах.  
Вводи или вспомогательные методы или поясняющие переменные
```java
if (neighbor.getX() >= 0 && neighbor.getX() < map.getWidth()
  && neighbor.getY() >= 0 && neighbor.getY() < map.getHeight()
  && !visited.contains(neighbor)) {...}

//ПРАВИЛЬНО:
if(isGameMapCoordinates(neighbor) && !visited.contains(neighbor)) {...}
```

Тут тоже, шо это? Никто сходу не поймёт это условие
```java
while (cameFrom.containsKey(step) && !cameFrom.get(step).equals(from)) 
```
Объясняй свои намерения через нейминг поясняющих переменных или вспомогательных методов.

**7. class Entity**

- Класс должен быть абстрактным. Не должно быть возможности создать "просто Entity"
```java
public class Entity 

//ПРАВИЛЬНО:
public abstract class Entity
```

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```java
public class Entity {
  protected String sprite;

  public String getSprite() {...}
}
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

+ 👍 Вот таким должен быть идеальный Entity и его простые неходячие наследники в этом проекте:
```java
public abstract class Entity {
  //да, тут совсем пусто
}

public class Tree extends Entity {
}
```

**8. class Grass extends Entity**

- Нарушение дизайна ООП в наследовании. 

При создании экземпляра класса, значения обязательных полей класса должны устанавливаться через конструктор, а не сетиться
```java
public class Grass extends Entity {
  public Grass() {
    this.sprite = "🌿";
  }
}

//ПРАВИЛЬНО:
public class Grass extends Entity {
  private static final String SPRITE = "🌿";

  public Grass() {
    this(SPRITE);
  }
}
```
*Эккель "Философия Java", гл.5, "Конструктор гарантирует инициализацию"*

Да, в моделях нельзя хранить спрайты, потому что из-за этого модели перестают быть моделями.  
Но я сейчас говорю про другое- про порядок инициализации данных в предке.

**9. class Herbivore extends Creature**

- Неправильная инициализация данных в предке
```java
public class Herbivore extends Creature {
  public Herbivore(Coordinates coordinates) {
    this.sprite = "🐑";
    this.coordinates = coordinates;
  }
  //...
}

//ПРАВИЛЬНО:
public class Herbivore extends Creature {
  //...

  public Herbivore(Coordinates coordinates) {
    this(SPRITE, coordinates);
  }
  //...
}
```

- Нарушение SRP. Заяц не имеет права рожать траву
```java
public class Herbivore extends Creature {
 
  @Override
  public void makeMove(Map map) {
    //...
    map.setEntity(new Coordinates(x, y), new Grass());
  }
}
```

- Используй константы
```java
public abstract class Creature extends Entity {
  //...
  protected Class<? extends Entity> food;

  public Creature(Class<? extends Entity> food, ...) {...}
  //...
}

public class Herbivore extends Creature {
  //...

  public void makeMove(Map map) {
    Coordinates nextStep = findTarget(map, food);
    //...

    if (food.isInstance(target)) {...}
  }
}
```

- Магические числа
```java
if (this.hp >= 20) {
  this.hp = 10;
  //...
}
```

**10. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(Map map);
}
```

**11. abstract class InitAction implements Action**

- Главные публичные методы должны стоять выше непубличных вспомогательных
```java
public abstract class InitAction implements Action {

  protected abstract Entity createEntity(Coordinates coords);

  public void execute(Map map) {...}
}

//ПРАВИЛЬНО:
public abstract class InitAction implements Action {

  public void execute(Map map) {...}

  protected abstract Entity createEntity(Coordinates coords);
}
```

- Этот параметр нужно принимать в конструктор
```java
public abstract class InitAction implements Action {
  public void execute(Map map) {
    for (int i = 0; i < 3; i++) {...}
  }
}

//ПРАВИЛЬНО:
public abstract class InitAction implements Action {
  //...

  public InitAction(int count) {...}

  public void execute(Map map) {
    for (int i = 0; i < count; i++) {...}
  }
}
```

**12. class Renderer**

- Спрайты существ должны храниться здесь, а не браться из самих существ.

Ответственность этого класса- печатать карту игры.  
Поэтому класс должен знать, как это делается. В том числе, как выглядят существа на карте.

**13. class Simulation**

- Нарушение DI. Класс не должен сам себя конструировать.  
В данном случае класс конструирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```java
public class Simulation {
  private Map map;
  //...


  public Simulation() {
    this.map = new Map(10, 10);
    //...
  }
}

//ПРАВИЛЬНО:
public class Simulation {
  private final GameMap gameMap;
  //...

  public Simulation(GameMap gameMap) {
    this.gameMap = gameMap;
    //...
  }
}
```
Таким образом нельзя без изменения кода в классе `Simulation` создать несколько игровых конфигураций с разными картами.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Метод должен делать только одно дело
```java
while (true) {
  nextTurn();
}

//ПРАВИЛЬНО:
while (true) {
  nextTurn();
  pause();
}
```
Делай маленькие методы, каждый из которых должен делать одно дело на одном уровне абстракции.  
*"ЧК", гл.3, "Правило одной операции", "Один уровень абстракции"*

- Согласно ТЗ, должен быть отдельный экшен для совершения хода существами
```java
public class Simulation {
  private List<Action> initActions;
  //...

  public Simulation(...) {
    //...
    this.initActions = new ArrayList<>();

    initActions.add(new InitActionRock());
    initActions.add(new InitActionGrass());
    //...
  }

  public void startSimulation() {
    for (Action action : initActions) {
      action.execute(map);
    }
    renderer.render(map);
    while (true) {
      nextTurn();
    }
  }


  public void nextTurn() {
    List<Entity> entities = new ArrayList<>(map.getEntities());
    for (Entity entity : entities) {
      if (entity instanceof Creature) {
        Creature creature = (Creature) entity;
        creature.makeMove(map);
      }
    }
    turnCount++;
    //...
  }
}

//ПРАВИЛЬНО:
public class Simulation {
  private final List<Action> initActions;
  private final List<Action> turnActions;
  //...

  public Simulation(...) {
    //...
    this.initActions = new ArrayList<>();
    this.turnActions = new ArrayList<>();

    initActions.add(new InitActionRock());
    initActions.add(new InitActionGrass());
    //...

    turnActions.add(new MoveAction());

    executeActions(initActions);
  }

  public void executeActions(List<Action> actions) {
    for (Action action : actions) {
      action.execute(map);
    }
  }


  public void nextTurn() {
    executeActions(turnActions);
    turnCount++;
    //...
  }
}
```
- Нарушение ТЗ, нет указанных для этого класса методов 
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

**14. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

С SOLID всё в целом более-менее норм.

Программа сделана частично- не реализована возможность сделать паузу во время работы симуляции.  
Во втором проекте важно поработать с потоками, а для этого нужно придумать, как делать паузу/пуск.

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.157(334)  
#ревью #симуляция 