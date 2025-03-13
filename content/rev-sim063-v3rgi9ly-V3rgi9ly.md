https://github.com/V3rgi9ly/Simulation  
[V3rgi9ly]

Есть над чем поработать.

## ХОРОШО

+ 👍 Существа не хранят в себе свои спрайты
+ 👍 Красивая распечатка карты

## ЗАМЕЧАНИЯ

**1. Нейминг**

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию
```
HashMap<Coordinates, Entity> map

//ПРАВИЛЬНО:
HashMap<Coordinates, Entity> entities
```

- Название должно как можно лучше объяснять суть явления
```
GameMap.x
GameMap.y

//ЛУЧШЕ:
GameMap.height
GameMap.width
```

- Излишний контекст
```
void setStaticObjects(Coordinates coordinates, Entity entity)

//ПРАВИЛЬНО:
void setEntity(Coordinates coordinates, Entity entity)
```

- Не старт а звезда
```
class AStart

//ПРАВИЛЬНО:
class AStar
```

- Так все-таки BFS или AStar?
```
final AStart bfs;
```

- Определись с принципом наименования пакетов, в единственном числе или во множественном: models, service, action, enums.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Тотально везде** использование оберток над примитивами(Integer) вместо примитивов(int) там, где это вообще не оправдано.  
Используй обертки осознано, а не доверяйся автоподстановке редактора кода в интелидж идее.

**3. Используй классы через их интерфейсы**
```
private final HashMap<Coordinates, Entity> map;

//ПРАВИЛЬНО:
private final Map<Coordinates, Entity> map;
```

**4. Избыточно, сразу кастуй**
```
if (entity instanceof Creature) {
  ((Creature) entity).makeMove(gameMap);
}

//ПРАВИЛЬНО:
if (entity instanceof Creature creature) {
  creature.makeMove(gameMap);
}
```

**5. class AppConf**, класс конфигурации

- Про классы констант, конфигурации и их использование я писал тут: https://t.me/zhukovsd_it_chat/53243/176984

**6. class Coordinates**

- При прочих равных используй примитивные типы, а не классы обертки
```
public Coordinates(Integer x, Integer y)

//ПРАВИЛЬНО:
public Coordinates(int x, int y)
```
Для использования обертки должна быть причина и выгода, которую дает применение этой обертки.
Тут выгоды никакой нет, но возможны баги
```
Coordinates coordinates = new Coordinates(null, null);
```

- Сдвиг(shift) или складывание(add) координаты должно происходить с экземпляром своего класса. Заводить для этого специальный класс сдвиговой координаты не надо
```
public Coordinates shift(CoordinatesShift coordinatesShift) 

//ПРАВИЛЬНО:
public Coordinates shift(Coordinates shiftCoordinates) 
```
Обычную координату можно использовать как координату сдвига, например так
```
Coordinates coordinates = new Coordinates(10, 10);
Coordinates shiftCoordinates = new Coordinates(-1, -1);  //сдвиг влево-вниз
coordinates = coordinates.shift(shiftCoordinates);
```

- Нарушение SRP и Low Coupling. Координата не должна определять, можно или нельзя ее сдвинуть в интересах какого-то другого класса.
Потому что это не касается единой ответственности координаты- хранение информации для идентификации точки в пространстве.  
Сейчас координата вынуждена знать про совершенно посторонние для себя вещи- про существование класса Карта, классов Камень и Хищник(?!) и правил игровой логики
```
public boolean canShift(CoordinatesShift coordinatesShift, GameMap gameMap) {
  //...
  Entity entity = gameMap.getEntity(newCoordinates);
  if (entity instanceof Rock || entity instanceof Predator) {
    return false; // Клетка занята препятствием
  }
}
```
Метод отсюда убрать. 

- toString() не должен быть на русском. Этот метод должен использоваться для отладки, а потому информация его тустринга должна быть ясна программисту, который понимает не только русский язык
```
public String toString() {
  return "Координата x = " + x + ", y = " + y;
}
```
*Блох, "Java эффективное программирование" гл.3.3*

❌ В целом, класс собрал в себе худшие практики для класса координаты. Идеальная координата здесь- это простой record, состоящий из двух полей x, y.

**7. class CoordinatesShift**

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates.

**8. class GameMap**

- Нарушение SRP и Low Coupling. Если класс получает конфишурационные данные из специального класса конфигурации, то он должен получать экземпляр конфигурации в конструктор, а не добывать самостоятельно
```
public GameMap() {
  this.x = AppConf.StartCoordinates.horizontal;
  this.y = AppConf.StartCoordinates.vertical;
  //...
}

//ПРАВИЛЬНО:
public GameMap(AppConf conig) {
  this.x = conig.StartCoordinates.horizontal;
  this.y = conig.StartCoordinates.vertical;
  //...
}

//НО ТАК КАК ИНТЕРЕСУЕТ ТОЛЬКО КЛАСС StartCoordinates, ТО:
public GameMap(StartCoordinates startCoordinates) {
  this.x = startCoordinates.horizontal;
  this.y = startCoordinates.vertical;
  //...
}
```

- Никогда не возвращай null
```
private final HashMap<Coordinates, Entity> map;

public Entity getEntity(Coordinates coordinates) {
  return map.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
private final HashMap<Coordinates, Entity> map;

public HashMap<Coordinates, Entity> getGameMap() {
  return map;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта();
//заселить карту существами
карта.getGameMap().clear(); //геноцид минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

- Почему "удалить ентити", но при этом "вставить статический объект", хотя в обоих случаях мы соершаем операции с ентити?
```
void deleteEntity(Entity entity) 
void setStaticObjects(Coordinates coordinates, Entity entity)
```

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними- вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: определение живых существ на карте, хранение координатного сервиса.

Будет знать карта про живых существ в себе или не будет этого знать- это никак не повлияет на выполнение Катой своей единой ответственности.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение.
```
public boolean isSquareEmpty(Coordinates coordinates) {
  return !map.containsKey(coordinates);
}

public Entity getEntity(Coordinates coordinates) {
  return map.get(coordinates);
}
```

**9. class TargetAwareCoordinateService**

- Нарушение DRY, дублилование кода в методах
```
Grass findAvailableGrass(Entity creature, Map<Coordinates, Entity> map)
Herbivore findAvailableHerbivore(Entity creature, Map<Coordinates, Entity> map)

//ПРАВИЛЬНО:
public <T extends Creature>  T findAvailable(Map<Coordinates, Entity> map,  Class<T> clazz)
```

**10. class AStart**, поиск по алгоритму astar

- Нарушение конвенции кода. Публичные методы должны находиться выше приватных. Здесь публичный метод пришлось реально искать.

+ 👍 В целом, к классу претензий нет, кроме длинного метода `aStarSearch(...)`, который явно нужно разделить на несколько методов поменьше.

**11. MapField**

- Так и не понял, зачем это. Возможно только для того, чтобы делать программу сложнее
```
public enum MapField {
  EMPTY,
  FILLED;
}
```

**12. Пакет models**, хранит классы ентити

- Моделями в данной реализации являются не только ентити, но и классы в других пакетах: Coordinates, GameMap etc. Поэтому папку надо назвать entities.

**13. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня Creature.
- Нарушение принципов наследования. Содержит другие поля, которые не нужны на этом уровне иерархии, а нужны только потомкам, и то не всем: 
```
protected MapField mapField;
protected TargetAwareCoordinateService coordinateService;
```

**14. class Herbivore/Predator extends Creature**

- Магическое: 1, 2.

- Дублирование кода. Общий код выноси в предка
```
//Herbivore.class
if (target != null) {
  if (isAdjacent(target.getCoordinates())) {
    if (target instanceof Grass) {
      eatGrass((Grass) target, gameMap);
    }
  } else {
    moveTowardsTarget(target, gameMap);
  }
}

//Predator.class
if (target != null) {
  if (isAdjacent(target.getCoordinates())) {
    if (target instanceof Herbivore) {
      attackHerbivore((Herbivore) target, gameMap);
    }
  } else {
    moveTowardsTarget(target, gameMap);
  }
}
```

**15. class MoveCreatures implements Action**

+ 👍 Идеальный мувер, просто каждой креатуре на карте дает пинка, чтобы она побежала. То есть, инициирует движение.

**16. class SpawEntity implements Action**

- Заселяет карту существами на фиксированные позиции, это несерьезно
```
gameMap.setStaticObjects(new Coordinates(31, 6), new Predator(2, 3, MapField.FILLED, coordinateService));
```
Нужно заселять по случайным пустым координатам на карте.

**17. class MapConsoleRenderer**

- В switch-case в default нужно кидать исключение. Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
switch (entity.getClass().getSimpleName()) {
  case "Predator":
    return AppConf.Image.lionEmoji;
  //...
  case "Grass":
    return AppConf.Image.grass;
}
return "";
```

**18. class Simulation**

+ 👍 Отдельные листы для разных видов экшенов это хорошо
```
List<Action> iniActions;
List<Action> turnActions;
```

- Это не сеттер. Метод здесь не устанавливает значение, а добавляет
```
public void setIniActions(Action action) {
  iniActions.add(action);
}

//ПРАВИЛЬНО: 
public void addInitAction(Action action)
```

**19. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Simulation, это хорошо.
+ 👍 Здесь добавляются экшены в Симуляцию
```
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap();
    Simulation simulation = new Simulation(gameMap);
    simulation.setTurnActions(new MoveCreatures());
    simulation.setIniActions(new SpawEntity());
    simulation.startSimulation();
  }
}
```
Тем самым на уровне Майн можно делать разные игровые конфигурации, не меняя код в Game
```
Action pandemic = new Action() {
  int turn;
  @Override
  public void perform(GameMap gameMap) {
    turn++;
    if (turn % 14 == 0) {
      //убить каждого второго
      System.out.println("Объявляется неделя чумы! %nКоличество существ уменьшается вдвое.");
    }
  }
};
simulation.addTurnActions(pandemic);
simulation.startSimulation();
```

## ВЫВОД

Разобраться в каких случаях нужно применять примитивные типы(int), а в каких обертки примитивов(Integer).  
Для лучшего понимания декомпозиции посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.63(142)  
#ревью #симуляция