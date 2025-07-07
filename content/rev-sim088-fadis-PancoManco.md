https://github.com/PancoManco/SimulationConsole_secondProject  
[FADIS]

В целом, неплохо.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

- UPPER_SNAKE только для констант, а это не константы
```
private final int ROW = 12;
private final int COLUMN = 12;

//ПРАВИЛЬНО ТАК:
private final static int ROW = 12;
private final static int COLUMN = 12;

//ИЛИ ТАК:
private final int row = 12;
private final int column = 12;
```

- Название должно как можно лучше объяснять суть явления. Это ширина и высота по умолчанию
```
public GameMap() {
  this.height = ROW;
  this.width = COLUMN;
}

//ПРАВИЛЬНО:
public GameMap() {
  this.height = DEFAULT_HEIGHT;
  this.width = DEFAULT_WIDTH;
}
```

- Если переменные принимают в себя ширину и высоту, почему бы их не назвать "ширина" и "высота"?
```
int rows = map.getHeight();
int cols = map.getWidth();

//ПРАВИЛЬНО:
int height = map.getHeight();
int width = map.getWidth();
```

- Названия пакетов нужно писать стилем lower_snake
```
Entities
Utils

//ПРАВИЛЬНО:
entities
utils
```

- Это не рендерер. Рендерер должен рисовать, а этот класс не рисует, а просто хранит спрайты
```
class EntityRenderer
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Entity> entities = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Entity> entities = new HashMap<>();
```
Общее правило: ArrayList нужно использовать через List, HashSet- через Set и т.д. 
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. Но это уже нюансы.

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
System.out.println("- Введите 'q' и нажмите Enter для выхода.");
if (input.equals("q")) {...}
System.out.println("Неизвестная команда. Для выхода нажмите 'q'.");

//ПРАВИЛЬНО:
private final static String QUIT = "q";

System.out.printf("- Введите '%s' и нажмите Enter для выхода. \n", QUIT);
if (input.equals(QUIT)) {...}
System.out.printf("Неизвестная команда. Для выхода нажмите '%s'. \n", QUIT);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**4. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. 

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

- Класс может быть преобразован в record без потери функционала.

**5. class GameMap**

- Карта не должна создаваться только одного размера- это делает карту неуниверсальной. 
Нужно ввести перегруженый конструктор, который позволит создавать карту произвольного размера
```
public GameMap() {
  this.height = ROW;
  this.width = COLUMN;
}

//ПРАВИЛЬНО:
public GameMap() {
  this(DEFAULT_HEIGHT, DEFAULT_WIDTH);
}

public GameMap(int height, int width) {
  this.height = height;
  this.width = width;
}
```

+ 👍 Геттеры не возвращают null, а возвращают Optional.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public boolean isEmptySquare(Coordinates coordinates) {
  return !entities.containsKey(coordinates);
}
```
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что свободна. А правильный ответ- такой координаты нет вообще.

- Надо искать не по любому классу, а по любойму наследнику Entity
```
public Collection<Entity> getEntitiesByType(Class<?> type)

//ПРАВИЛЬНО:
public Collection<Entity> getEntitiesByType(Class<? extends Entity> type)
```

+ 👍 С точки зрения SRP, класс хороший.

**6. class PathFinderBFS**

- Пара x,y это что? Правильно, координата
```
private static final int[][] DIRECTIONS = { 
  { 1, 0 },  // движение вправо
  {-1, 0 }, // движение влево
  //...
};

//ПРАВИЛЬНО:
private static final Coordinates[] DIRECTIONS = { 
  new Coordinates(1, 0),  // движение вправо
  //...
};
```

- Нарушение SRP.

Поиск должен просто искать путь от точки старта до точки, соответствующей заданным условиям.  
В данном случае- до точки, в которой находится существо нужного класса.
Если поиск получает во входящие координату цели, значит часть работы поиска уже выполнил кто-то другой. 
То есть, кто-то другой уже нашел цель и передал ее координату в поиск пути
```
public List<Coordinates> findPath(GameMap map, Coordinates start, Coordinates goal, Set<Class<?>> obstacles) 
```
На самом деле поиск пути должен сам искать цель и прокладывать к ней путь. Сигнатура метода для этого(вместе с припятствиями) должна выглядеть примерно так
```
public List<Coordinates> getPatch(Карта карта, Coordinates start, Class<? extends Entity> target,  Set<Class<?>> obstacles) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. Grass.class)
}
```

**7. Два отдельных пакета: Entities и Creatures**

Не должно быть два отдельных пакета Entities и Creatures- потому что Creatures это и есть Entities. 
Потому что Creature наследуется от Entity.
Креатуры могут находиться в отдельном от всех остальных энтити пакете, но этот пакет должен быть вложенным в пакет "entities"

**8. abstract class Entity и его "простые" потомки Grass/Tree/Rock**

Не имет смысла тут явно указывать конструктор. Не бойся ["пустых" классов](https://t.me/zhukovsd_it_chat/53243/241961)
```
public abstract class Entity {
  public Entity() {
  }
}

public class Grass extends Entity {
  public Grass() {
  }
}

//ПРАВИЛЬНО:
public abstract class Entity {
}

public class Grass extends Entity {
}
```

**9. Наследники Creature: class Herbivore/Predator extends Creature**

- Используй константы
```
public class Herbivore extends Creature {
  public Herbivore() {
    this.health = 10;
    this.speed = 1;
  }
  //...
}

public class Predator extends Creature {
  public Predator() {
    this.health = 20;
    this.speed = 2; 
  }
  //...
}

//ПРАВИЛЬНО:
public class Herbivore extends Creature {
  private final static HEALTH = 10;
  private final static SPEED = 1;

  public Herbivore() {
    this.health = HEALTH;
    this.speed = SPEED;
  }
  //...
}

public class Predator extends Creature {
  private final static HEALTH = 20;
  private final static SPEED = 2;
  
  public Predator() {...}
  //...
}
```
Это позволит легче добавлять наследников- копируешь класс, вставляешь, меняешь константы и всё. 
Характеристики существ в классах тоже будут выглядеть нагляднее
```
public class Kolobok extends Creature {
  private final static HEALTH = 30;
  private final static SPEED = 3;
  
  public Kolobok() {
    this.health = HEALTH;
    this.speed = SPEED;
  }
}
```

- Нарушение DRY. Почти идентичный код у Зайца и Волка в методе `makeMove(GameMap map)`. Общий код выноси в предка, в данном случае- в Creature.

- Большой метод `makeMove(GameMap map)`- 50 строк. Если метод больше 10-20 строк, значит скорее всего он делает много разного. 
Постоянные комментарии того, что происходит в классе- маркер именно этого. Каждый комментарий тут как будто разделяет разные смысловые блоки.

Раздели метод на несколько, каждый из которых будет делать свое. Например:
```
public void makeMove(GameMap map) {
  //...
  // Если добрались до травы — съедаем её  <-- ОРИГИНАЛЬНЫЙ КОММЕНТАРИЙ
  if (grassPositions.contains(nextPosition) && optionalEntityAtNext.filter(entity -> entity instanceof Grass).isPresent()) {
    optionalEntityAtNext.ifPresent(entityAtNext -> map.removeEntity(nextPosition, entityAtNext));
    this.health += 5;
  }
  //...
}

//ПРАВИЛЬНО:
public void makeMove(GameMap map) {
  //...
  if (isТрава(nextPosition)) {  <-- ТЕПЕРЬ НЕ НУЖЕН КОММЕНТАРИЙ, НЕЙМИНГ САМ ВСЕ ОБЪЯСНЯЕТ
    съестьТраву();
  }
  //...
}
```

- Магические числа: 5 (у Зайца), 10 (у Волка).

**10. Пакет Utils**

👍 Содержит только утилитные классы: final, имеют только статические методы и приватный конструктор.

**11. Пакет Actions**

Внезапно: содержит файл ".gitignore". Этот файл не явлется экшеном и не должен находиться в данном пакете.

**12. class InitActions implements Action**

👍 В целом, норм.

- Количество существ вынеси в константы.

**13. class TurnActions implements Action**

Не должно быть два публичных метода. Или публичный метод здесь будет один, или это фактически не Action. 
Потому что концепция Action- один класс с одним публичным методом для одинакового использования всех экшенов через полиморфизм- см. паттерн "Команда".

В данном случае, ты просто забыл заприватить этот метод
```
public class TurnActions implements Action {
  //...
  @Override
  public void execute() {
    makeIteration(map);
  }

  public void makeIteration(GameMap map) {...}
}

//ПРАВИЛЬНО:
private void makeIteration(GameMap map) {...}
```

**14. class EntityRenderer**

- Это не рендерер, это хранилище спрайтов существ.

- Если пришел запрос на спрайт неизвестного существа- нужно бросать исключение
```
public String getSymbol(Entity entity) {
  if (entity == null)
    return " "; <-- НУЖНО БРОСИТЬ ИСКЛЮЧЕНИЕ
  String symbol = entitySymbols.get(entity.getClass());
  return symbol != null ? symbol : "?"; <-- ЕСЛИ NULL- НУЖНО БРОСИТЬ ИСКЛЮЧЕНИЕ
}
```
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт.

- Лучше сразу
```
entitySymbols.put(Herbivore.class, "\uD83D\uDC0F"); // 🐏 Овца (Herbivore)

//ПРАВИЛЬНО:
entitySymbols.put(Herbivore.class, "🐏");
```

**15. class MapRenderer**

+ 👍 Форматированный вывод это хорошо.

- Внезапная экономия букв на полуслове
```
Coordinates coord = new Coordinates(row, col);

//ПРАВИЛЬНО ТАК:
Coordinates coordinates = new Coordinates(row, col);

//ИЛИ ТАК- РАЗ УЖ ОБЛАСТЬ ВИДИМОСТИ ПЕРЕМЕННОЙ ВСЕГО 3 СТРОКИ
Coordinates c = new Coordinates(row, col);
```

- Вынеси в константы: "%-3d", "_".

**16. class Simulation**

👍 Принимает в конструктор достаточное количество зависимостей, в том числе действия. Это конечно хуже, чем если бы принимало *списки* действий, ну пусть так.
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе Simulation
```
public Simulation(GameMap map, MapRenderer renderer, Action initAction, Action turnAction)
```

**17. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

+ 👍 Создает и запускает Симуляцию через фабрику, это хорошо.

## ВЫВОД

В целом, неплохо.

n.88(197)  
#ревью #симуляция 