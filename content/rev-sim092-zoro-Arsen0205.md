https://github.com/Arsen0205/simulation  
[Zoro]

Удручающе- базовые ошибки ООП. Существа ходят неправильно.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Существа плохо ищут еду  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim092/img0.png)

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Реализовано пауза/пуск во время работы 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Сеттеры должны устанавливать *уже существующее*. А этот метод не устанавливает уже существующее, а добавляет то, чего еще не было
```
void setGrass(Coordinates coordinates, Grass herb)

//ПРАВИЛЬНО:
void addGrass(Coordinates coordinates, Grass herb)
```

- Название должно объяснять суть явления. "Сделать все"- не объясняет ничего
```
void makeAll() 
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Tree> tree = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Tree> tree = new HashMap<>();
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д. 
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. Но это уже нюансы.

**3. Текст в exception** всегда должен быть на английском языке
```
throw new IllegalStateException("Невозможно переместиться на занятую клетку");
```
Исключение это не просто телеграмма, которая летит сквозь слои. 
У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты. 
А значит, сообщение должно быть на английском. 
Интерпритация исключения и перевод его на локальный язык должно происходить там, где это соответствует архитектуре программы или не происходить вовсе, если исключение не планируется перехватывать

**4. Нарушение инкапсуляции**.

Системная проблема во всем проекте- поля объявляются публичными или их уровень доступа явно не объявляется, и доступ к ним происходит напрямую. 
Доступ к полям должен происходить только через геттеры и сеттеры
```
public final int vision;
public final int speed;
public Coordinates coordinates;
```

**5. Комментарии**

Коментарии в основном не несут полезной нагрузки, а констатируют очевидное
```
// Вариант 1: Запуск одного хода
public void nextTurn()
```
Когда в проекте много каментов, это плохо- пользы от них практически нет, они только забивают пространство и мешают читать код.
В идеале, комментариев вообще не должно быть, код должен объяснять сам себя через правильный нейминг и лаконичный код.  
*Мартин, "ЧК", гл.4* 

**6. Если нужно печатать или создавать строку** с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматированный вывод- тогда сразу будет виден весь шаблон
```
System.out.println("Заяц съел траву на " + herbCoord + ". Здоровье: " + health);

//ПРАВИЛЬНО:
System.out.printf("Заяц съел траву на %s. Здоровье: %d \n", herbCoord, health);
```

**7. class Coordinates**

Нарушение инкапсуляции. Поля не должны быть публичными, доступ к ним болжен быть только через геттеры и сеттеры
```
public class Coordinates {
  public int x;
  public int y;
  //...
}
```
Поля могут быть публичными у классов, которые являются структурами. Но этот класс имеет развитое поведение, а значит не является структурой. 
*Мартин "ЧК", гл.6*

- Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: x,y или row,column.
Лучше даже, если этот класс будет record'ом.
Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия, тем самым нарушая SRP: рассчитывает дистанцию между точками, проверяет на соседние координаты.

**8. class World**

- Нарушение инкапсуляции. Всегда явно указывай область видимости и final, если переменная final
```
HashMap<Coordinates, Predator> predators = new HashMap<>();

//ПРАВИЛЬНО:
private final Map<Coordinates, Predator> predators = new HashMap<>();
```

- Нарушение SRP. 
Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретными классами-наследниками Entity. 
Здесь карта хранит каждый вид существ в отдельной мапе
```
HashMap<Coordinates, Herbivores> herbivores = new HashMap<>();
HashMap<Coordinates, Predator> predators = new HashMap<>();
HashMap<Coordinates, Grass> grass = new HashMap<>();
HashMap<Coordinates, Tree> tree = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Entity> enties = new HashMap<>();
```
Читай ТЗ, там все написано:
```
Проблемы и ошибки в коде: ...параллельные коллекции для разных типов существ (для каждого типа своя коллекция)
```

- Нарушение SRP, которое проистекает из предыдущего нарушения SRP. Отдельные методы для каждого типа существ
```
public void setGrass(Coordinates coordinates, Grass herb) {...}
public void setHerbivores(Coordinates coordinates, Herbivores herbivore) {...} 
public void setPredators(Coordinates coordinates, Predator predator) {...}
public void setTree(Coordinates coordinates, Tree wood) {...}

//ПРАВИЛЬНО:
public void add(Coordinates coordinates, Entity entity) {...}
```

- Нарушение SRP, божественный класс, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: начальная инициализация карты существами, сделать ход существами, найти случайную пустую координату и все такое.

Наверное, для проекта в целом полезно иметь метод, который находит случайную пустую координату. 
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно искать в себе случайную пустую координату.  

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Карта не должна сама себя заселять существами, иначе она становится не универсальной и нельзя будет создать несколько конфигураций игры с разными комбинациями существ на карте
```
void setupDefaultPiecesPositions()
```
Объект не должен сам себя инициализировать.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования".*

- Нарушение SRP. Метод, который инициирует движение креатур
```
public void makeAll() {
  //...  
  for (Herbivores herb : herbivoresList) {
    herb.makeMove(this);
  }
  for (Predator predator : predatorList) {
    predator.makeMove(this);
  }
}
```
Инициация движения в креатурах не имеет никакого отношения к единой ответственности карты по хранению в себе существ. 
Вообще-то для этого в ТЗ предусмотрены Action'ы.

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
карта.setHerbivore(new Coordinates(0, 0), new Заяц());
карта.moveCreature(new Coordinates(0, 0), new Coordinates(99, 99));

/* class World */
public void moveCreature(Creature creature, Coordinates newCoord) {...}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
HashMap<Coordinates, Herbivores> herbivores = new HashMap<>();

public HashMap<Coordinates, Herbivores> getHerbivores() {
  return herbivores;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта(100, 100);
<заселить карту существами>
карта.getHerbivores().clear(); //геноцид- удаление из карты всех травоядных, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void setPredators(Coordinates coordinates, Predator predator) {
  predator.coordinates = coordinates;
  predators.put(coordinates, predator);
}
```

❌ В целом, класс собрал в себе худшие практики.

**9. class Pathfinding**, поиск пути

- Как я показал в самом начале, существа ходят неправильно. А значит, поиск пути работает неправильно.

- Утилитные классы должны быть `final` и иметь приватный конструктор- не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.

- Класс не ищет путь, он ищет первый шаг пути. Нужно искать путь, то есть последовательность координат
```
public static Coordinates getNextStep(...)

//ПРАВИЛЬНО:
public static List<Coordinates> find(...) 
```

- Смещения нужно вынести из метода и сделать константами.  
И, что такое пара х/у? Правильно, координата
```
public class Pathfinding {
  public static Coordinates getNextStep(Coordinates start, Coordinates goal, World world) {
    int[][] directions = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
   //...
  }
}

//ПРАВИЛЬНО:
public class Pathfinding {
  private final static List<Coordinates> SHIFTS = List.of{
    new Coordinates(0, 1),
    new Coordinates(1, 0),
    //...
  };

  public static Coordinates getNextStep(Coordinates start, Coordinates goal, World world) {
   //...
  }
}
```

- Нарушение SRP.

Поиск должен просто искать путь от точки старта до точки, соответствующей заданным условиям.
В данном случае- до точки, в которой находится существо нужного класса.
Если поиск получает во входящие конкретный экземпляр цели, значит часть работы поиска уже выполнил кто-то другой. 
То есть, кто-то другой уже нашел цель и передал ее в поиск пути
```
public static Coordinates getNextStep(Coordinates start, Coordinates goal, World world)
```
На самом деле поиск пути должен сам искать цель и прокладывать к ней путь. Сигнатура метода для этого должна выглядеть примерно так
```
public static List<Coordinates> find(World world, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте world от точки start
  //до точки, где находится существо нужного класса(напр. Grass.class)
}
```

**10. public class Entity**

- Нарушение инкапсуляции, доступ к полям должен быть только через геттеры и сеттеры
```
public Coordinates coordinates;
```

- Содержит координату. Но координата нужна только тому существу, которое ходит. 
Поэтому entities должны хранить координату только начиная с уровня `Creature`

**11. public class Creature**

- Нарушение ООП дизайна и принципов наследования- класс должен наследоваться от Entity
```
public class Creature

//ПРАВИЛЬНО:
public class Creature extends Entity
```

**12. class Herbivores/Predator extends Creature**

- Нарушение DRY, дублирование кода в классах Herbivores и Predator. 
Например, в `findNearestTarget(World world)`:
```
public class Herbivores extends Creature implements Detection, Eating {
  //...
  @Override
  public Coordinates findNearestTarget(World world) {
    Coordinates nearest = null;
    int minDistance = Integer.MAX_VALUE;

    for (Coordinates grassCoord : world.getGrass().keySet()) {
      int distance = coordinates.distanceTo(grassCoord);
      if (distance <= this.vision && distance < minDistance) {
         nearest = grassCoord;
         minDistance = distance;
      }
    }
    return nearest;
  }
}

public class Predator ... {
  //...
  @Override
  public Coordinates findNearestTarget(World world) {
    // то же самое
    for (Coordinates grassCoord : world.getHerbivores().keySet()) {
      // то же самое
    }
    return nearest;
  }
}
```

Общий код выноси в предка:
```
public class Creature ... {
  //...
  public Coordinates findNearestTarget(World world) {
    Coordinates nearest = null;
    int minDistance = Integer.MAX_VALUE;

    for (Coordinates targetCoordinates : getAllTargetCoordinates()) {
      int distance = coordinates.distanceTo(targetCoordinates);
      if (distance <= this.vision && distance < minDistance) {
         nearest = targetCoordinates;
         minDistance = distance;
      }
    }
    return nearest;
  }

  protected abstract List<Coordinates> getAllTargetCoordinates();
}

public class Herbivore ... {
  //...
  @Override
  protected List<Coordinates> getAllTargetCoordinates() {
    return world.getGrass().keySet();
  }
}

public class Predator ... {
  //...
  @Override
  protected List<Coordinates> getAllTargetCoordinates() {
    return world.getHerbivore().keySet();
  }
}
```

- Нарушение SRP, чужая ответственность, зависимость модели от представления.

Модель(а это модель) не должна ничего печатать в консоль
```
if (newCoordinates != null){
  System.out.println("Заяц спрыгнул с клетки: " + oldCoordinates);
  System.out.println("Заяц прыгнул на клетку: " + newCoordinates);
  //...
}
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах нужно будет менять код в этом классе, чтобы вывод осуществлялся по правилам этой среды, то есть, это лишняя причина для изменения класса.
В других средах(напр. Андроид) эту модель нельзя будет использовать- она там просто не скомпилируется. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

Если модель должна что-то сообщить миру, она может это сделать через паттерн CallBack: https://t.me/zhukovsd_it_chat/53243/139594

Вот к чему приводит зависимость модели от представления, карта рисуется в виндовый интерфейс, а модель пишет в консоль:  
https://github.com/raketareview/simulation_review/blob/master/content/rev-sim064-davidtagirov-DavidTagirov.md

Если модель должна что-то сообщить миру, она может это сделать через паттерн CallBack.  
[CallBack луноход](https://t.me/zhukovsd_it_chat/53243/139594)  
[CallBack разрушение](https://t.me/zhukovsd_it_chat/53243/184749)

**13. Action'ы, которых нет**

В ТЗ указаны Action'ы, которые должны быть в проекте. 
Здесь их нет и весь тот функционал, который по ТЗ должны реализовывать экшены, перенесен в божественный класс Карта.

Нарушать ТЗ можно, когда это улучшает архитектуру проекта. 
Отсутствие Action'ов здесь ухудшает архитектуру, поэтому сделай Action'ы так, как указано в ТЗ.

Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.  
Каждый из этих классов должен делать что-то свое с картой: один экшен должен заселять карту существами, другой делать ходы и т.д.  
Actions, изложенный в ТЗ, это вариант реализации паттерна Command.  

То есть экшены должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
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

**14. class WorldConsoleRenderer**

+ 👍 Спрайты существ хранятся здесь, а не в самих существах, это хорошо.

- В конце цепочки if-else нужно кидать исключение- значит в карте есть существо, спрайт которого мы не знаем. 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
private String getSprite(Coordinates coordinates) {
  //...
  if (world.getTree().containsKey(coordinates)) {
    symbol = formatSymbol(SYMBOL_TREES);
  } else ...;

  return ANSI_GREEN_BACKGROUND + symbol;  <-- ВМЕСТО ЭТОГО НУЖНО КИНУТЬ ИСКЛЮЧЕНИЕ
}
```

**15. class Simulation**

- Нарушение DRY, магические буквы, числа, слова. Вводи константы. А если они есть- пользуйся
```
public class Simulation {
  private final static int ONE_MOVE = 1;
  private final static int THE_ENDLESS_LOOP = 2;
  private final static int QUIT = 3;
  private final static String PAUSE = "P";
 
  if (input.equalsIgnoreCase("P")) {...}
  if (input.equals("1")) {...}
  if (input.equals("2")) {...}
  if (input.equalsIgnoreCase("3")) {...}

//ПРАВИЛЬНО:
  private final static String ONE_MOVE = "1";
  private final static String THE_ENDLESS_LOOP = "2";
  private final static String QUIT = "3";
  private final static String PAUSE = "P";
 
  if (input.equalsIgnoreCase(PAUSE)) {...}
  if (input.equals(ONE_MOVE)) {...}
  if (input.equals(THE_ENDLESS_LOOP)) {...}
  if (input.equalsIgnoreCase(PAUSE)) {...}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**16. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## АРХИТЕКТУРА

Нет понимания ООП даже на самом базовом уровне- инкапсуляция, наследование, полиморфизм.  
Здесь одно цепляется за другое. Не понимая, что такое полиморфизм, зачем он нужен и как используется в ООП программе, невозможно сделать грамотное наследование.

Тем более нет понимания ООП и на более глубоком уровне- SOLID.

## ВЫВОД

Сделать Action по ТЗ. 

Изучить базовые принципы ООП- инкапсуляция, наследование, полиморфизм. Посмотреть ролики Немчинского про эти принципы.

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.92(209)  
#ревью #симуляция 