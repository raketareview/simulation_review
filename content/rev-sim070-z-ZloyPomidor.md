https://github.com/ZloyPomidor/simulationJavaProject  
[.Z.]

Есть много над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(просто мне так больше нравится)
+ 👍 Алгоритм поиска AStar 

## ЗАМЕЧАНИЯ

**0. Мусор в репозитории**, разберись, какие файлы и папки не нужно добавлять в комиты, добавь их в список игнорируемых.

**1. Нейминг**

- Если в проекте есть класс WorldMap то все переменные с именем, включающим это название, должны быть экземплярами этого класса
```
public class WorldMap {
  private final Map<Coordinates, Entity> worldWap;
} 

//ПРАВИЛЬНО:
public class WorldMap {
  private final jMap<Coordinates, Entity> entities;
}
```

- Избыточный контекст. И так ясно, что внутри Карты метод "поместить" означает помещение существа в карту
```
/* class WorldMap */
void setEntityOnMap(Coordinates coordinates, Entity entity)
void removeEntity(Coordinates coordinates)

//ПРАВИЛЬНО:
void set(Coordinates coordinates, Entity entity)
void remove(Coordinates coordinates)
```

- Методы с префиксами `is-`, `has-` должны возвращать boolean 
```
void isCoordinatesValid(Coordinates coordinates)
```

- Название обманывает. Говорит, что вернет существа, но возвращает координаты
```
List<Coordinates> getEntitiesOfType(Class<? extends Entity> searchingType)
```

- UPPER_SNAKE только для констант, а это не константа
```
private final int MAX_HP_VALUE;
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"* 

**2. Нарушение конвенции кода**. Переменные и методы располагаются черте-как: константы под переменными, конструкторы ниже других методов, публичные методы ниже вспомогательных приватных. 

**3. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (!worldMap.coordinatesIsEmpty(cord)) {
  return getTheSelectedSmiley(worldMap.getEntity(cord));
} else
  return BACKGROUND;

//ПРАВИЛЬНО:
if (!worldMap.coordinatesIsEmpty(cord)) {
  return getTheSelectedSmiley(worldMap.getEntity(cord));
} else {
  return BACKGROUND;
}
```

**4. Нарушение DRY**, магические буквы, числа, слова. Вводи константы.  
Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов, либо перенеси ее в третий класс
```
public class ConsoleMessagesRenderer {
  public static final String STRING_MESSAGE = "Please select an action: \"S\" - (START) or \"E\" - (EXIT)"; <-- МАГИЧЕСКИЕ БУКВЫ
  //...
} 

public class Simulation {
  private static final String EXIT = "E";  <-- КОНСТАНТЫ КОТОРЫЕ ДОЛЖНЫ БЫТЬ СИНХРОНИЗИРОВАНЫ С МАГИЧЕСКИМИ ЧИСЛАМИ В ConsoleMessagesRenderer
  private static final String START_STOP = "S";
  //...
}
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**5. class Coordinates**

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

- Нарушение SRP. Координата хранит данные, которые к ней не имеют никакого отношения: размеры карты
```
public static final int MAX_COORDINATES_OF_COLUMN = 20;
public static final int MAX_COORDINATES_OF_ROW = 50;
public static final int MIN_COORDINATES_OF_THE_WORLD = 0;
```

- Нарушение конвенции кода- конструктор должен стоять выше, чем все остальные методы.

- Нарушение SRP и Low Coupling. Координата не должна определять, можно ли ее сдвинуть в интересах какого-то другого класса или нельзя.
Потому что это не касается единой ответственности координаты- хранение информации для идентификации точки в пространстве
```
public boolean canShift(Coordinates shift)
```

- Нарушение SRP. Этот метод опредеделяет не корректность координаты как таковой, а корректность координаты в смысле вхождение этой координаты в Карту
```
public boolean isCoordinatesValid() {
  return row <= MAX_COORDINATES_OF_ROW && row >= MIN_COORDINATES_OF_THE_WORLD &&
      column <= MAX_COORDINATES_OF_COLUMN && column >= MIN_COORDINATES_OF_THE_WORLD;
}
```
Валидность координаты для карты должна определять сама карта, а не класс координаты. 

- Нарушение SRP. Этот метод не имеет никакого отношения к единой ответстсвенности по идентификации точки в пространстве.  
Нарушение Low Couopling. Координата не должна знать про класс WorldMap
```
Coordinates getRandomAvailableCoordinate(WorldMap worldMap)
```

❌ В целом, здесь собраны худшие практики для класса координаты.

**6. class WorldMap**

- Карта должна сама знать свои размеры и по ним валидировать, а не запрашивать свою валидацию из класса координаты (1). Для этого карта должна принимать свои размеры в конструктор.  
Карта не должна знать про существования Рендерера и нет более брать из него текст сообщения своего исключения (2)
```
private void isCoordinatesValid(Coordinates coordinates) {
  if (!coordinates.isCoordinatesValid()) {  <-- 1
    throw new IllegalArgumentException(ConsoleMessagesRenderer.COORDINATES_IS_NOT_VALID_MESSAGE);  <-- 2
  }
}
```

- Что, если ты попытаешься удалить существо по координате, где нет существа? Подумай, как корректно в этом случае должна себя программа
```
public void removeEntity(Coordinates coordinates) {
  isCoordinatesValid(coordinates);
  worldWap.remove(coordinates);
}
```

- Карта не должна заставлять креатур атаковать, это часть игровой логики, которая не имеет никакого отношения к задачам карты
```
private void interactionOnEntity(Coordinates from, Coordinates to) {
  Entity entityTo = getEntity(to);
  Creature creatureFrom = (Creature) getEntity(from);

  if (creatureFrom.isTarget(entityTo)) {
    creatureFrom.attack(this, to);   <-- ?!
  }
}
```

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта();
карта.setEntityOnMap(new Coordinates(0, 0), new Заяц());
карта.moveEntity(new Coordinates(0, 0), new Coordinates(99, 99));

/* class GameMap */
 public void moveEntity(Coordinates from, Coordinates to)
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Неправильное использование констант из другого класса  
```
public static final int MAP_SIZE = Coordinates.MAX_COORDINATES_OF_ROW * Coordinates.MAX_COORDINATES_OF_COLUMN;
```
Свои размеры Карта должна получать в конструктор, а не вытягивать из констант другого класса.

Условный пример
```
public final class Settings {
  //приватный конструктор
  public static final int HOUSE_ROOMS = 5;
}

/ПЛОХО:
public static void main(String[] args) {
  House house = new House();
  //oth code
}

public class House {
  private final int rooms;

  public House() {
    this.rooms = Settings.HOUSE_ROOMS;
  }

  public int getRooms() {
    return rooms;
  }
}

//ХОРОШО:
public static void main(String[] args) {
  House house = new House(Settings.HOUSE_ROOMS);
  //oth code
}

public class House {
  private final int rooms;

  public House(int rooms) {
    this.rooms = rooms;
  }

  public int getRooms() {
    return rooms;
  }
}
```

+ 👍 Этото метод норм, а его название- нет
```
public List<Coordinates> getEntitiesOfType(Class<? extends Entity> searchingType)
```

- Не нужно два раза читать из хешмапы
```
Entity entity = worldWap.get(coordinates);
if (entity == null) {...}
return worldWap.get(coordinates);

//ПРАВИЛЬНО:
Entity entity = worldWap.get(coordinates);
if (entity == null) {...}
return entity;
```

**7. Пакет utils**

Не содержит утилиты. Гугли, что понимается под утилитным классом в яве.

**8. class AStar**

- Как пользоваться этим поиском, совршенно не ясно- поиск возвращает не путь, то есть последовательность точек, а одиночную точку
```
public Coordinates search(Coordinates start)
```
Метод поиска должен в качестве пути возвращать последовательность точек, например в виде List'a.

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем анализаданных в Creature, которая находится в стартовой координате
```
public Coordinates search(Coordinates start) {
  Creature currentCreature = (Creature) worldMap.getEntity(start);
  Node startNode = new Node(start);
  Coordinates target = getTarget(start, currentCreature.getTarget().getClass());
  //...
}  
```

- У этого класса должен быть один публичный метод, который просто ищет путь от точки старта до точки, соответствующей условиям.  
Например, так:
```
public List<Coordinates> getPatch(World world, Coordinates start, Class<? extends Entity> target) {...}
```

- Класс не должен возвращать объект, который принимает во входящие. 
Потому что клиент ожидает, что объект, который в результате своей работы вернет метод, будет *новым*, а не измененным старым.
На этом логичном допущении клиент будет строить логику своей работы, а подобные фокусы с возвратом входящих могут привести к неожиданным багам в программе
```
public Coordinates search(Coordinates start) {
  //...
  return start;
}

//ПРАВИЛЬНО:
public Coordinates search(Coordinates start) {
  Coordinates coordinates = new Coordinates(start.getRow(), start.getColumn());  
  //...
  return coordinates;
}
```

**9. class PathFinder**

Абсолютно бесполезный класс, простейшая обертка над AStar. Причина существования этого класса мне неясна. Антипаттерн "Полтергейст"
```
public class PathFinder {
  public Coordinates findPath(WorldMap worldMap, Coordinates current) {
    AStar searcher = new AStar(worldMap);
    return searcher.search(current);
  }
}
```

**10. class MoveType**

Класс содержит один статический метод- значит это утилитный класс, а значит должен быть final и иметь приватный конструктор.

**11. public class Entity**

Класс должен быть абстрактным. Не должно быть возможности создать "просто Entity".

**12. abstract class Creature extends Entity**

- Нарушение SRP. Что бы тут ни происходило, креатура не должна ничего знать про Actions
```
private boolean isTimeToWalk(Actions actions) {
  return actions.getStepCounter() % speed == 0;
}
```
На самом деле я понимаю, что здесь происходит: кто-то извне дергает креатуру за хвост и кретура считает эти дерганья. 
Когда дерганья становятся кратными `скорости`, креатура мяукает(`isTimeToWalk(...) == true`) и этот кто-то дает ей пинка, чтобы она побежала.

Здесь логика в корне неправильна. Должно быть наоборот: тот кто, сидит в кустах, сам должен дергать себя за хвост и считать количество дерганий. 
Когда количество самодерганий будет кратно скорости креатуры, тогда из кустов нужно давать пинка под зад креатуре, чтобы она побежала.

- Креатура в качестве еды хранит не класс, а экземпляр класса
```
private final Entity target;
Entity getTarget()
```
Представль волка, который в качестве образца еды носит в кармане живого зайца и для опредления еды достает из кармана и смотрит на этого зайца, сравнивая его со встреченным объектом. Это непрактично.

На самом деле волк должен носить не живого зайца на кармане, а его фотографию и потенциальную еду сравнивать с этой фотографией.  
Примерно так
```
private final Class<? extends Entity> food;
Class<? extends Entity> getFood()
```

**13. class Predator/Herbivore extends Creature**

👍 Тут норм.

**14. Пакет actions**

- Идея Actions здесь не осмыслена. Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.
Actions, изложенный в ТЗ, это вариант реализации паттерна Command.

То есть акции должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
```
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

List<Action> actions = List.of{new ДатьСигаретуВсемЗайцамAction(), new ХодитьAction()};
for(Action a: actions) {
  a.execute(карта);
}

/*
Результат: программа обойдет всю карту, найдет всех креатур и даст им пинка, чтобы они побежали.
А каждому зайцу предварительно даст сигарету.
*/
```

**15. class WorldConsoleRenderer**

В switch-case в default нужно кидать исключение. Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт.

**16. class Simulation**

Симуляция должна принимать необходимые зависимости в конструктор. Здесь- как минимум экземпляр карты.
Сейчас карта создается с фиксированными размерами, которые хранятся в константах.  
Когда ты переделаешь карту по-нормальному, то есть когда Карта будет принимать свои размеры в конструктор, то экземпляр карты нужно будет создавать в майне и далее инжектить эту карту в конструктор Симуляции.

**17. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

Сделать Action's по ТЗ.  
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

P.S. Зря пропустил виселицу, нужно было бы с нее начинать.

n.70(156)  
#ревью #симуляция