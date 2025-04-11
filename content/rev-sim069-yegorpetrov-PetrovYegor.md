https://github.com/PetrovYegor/Simulation  
[Егор Петров]

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Слишком длинные команды
```
Select one of the simulation commands:
start  - begin simulation
pause  - pause simulation
resume - resume paused simulation
stop   - stop simulation completely
exit   - quit program
```
Здесь консоль является общим механизмом для печати информации и ввода команд. Поэтому пока наберешь "pause", консоль обновится и нужно вводить все заново. 
У меня например, с трех попыток не получилось успеть ввести команду во время работы симуляции.   
При тестировании программы ставь себя на место юзера и думай, насколько удобно пользоваться программой.  
Сделай длинной в 1 символ хотя бы те команды, которые нужно быстро вводить, например "pause" -> "p".

## ХОРОШО

+ 👍 Координаты в существах хранятся начиная с Creature
+ 👍 Простой понятный алгоритм

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Пакет "models" должен содержать в себе либо все классы-модели, либо называться иначе. Сейчас в проекте есть модели, которые находятся вне этого класса: Coordinates, GameBoard etc.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
System.out.println("start  - begin simulation");
System.out.println("pause  - pause simulation");
      
case "start":  //...
case "pause":  //...

//ПРАВИЛЬНО:
private final static String START = "start";
private final static String PAUSE = "pause";

System.out.println(START + " - begin simulation");
System.out.println(PAUSE + "  - pause simulation");
      
case START:  //...
case PAUSE:  //...
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"*  

**3. Сразу кастуй**
```
if (entity instanceof Creature) {
  Creature creature = (Creature) entity;
  creature.setCoordinates(coordinates);
}

//ПРАВИЛЬНО:
if (entity instanceof Creature creature) {
  creature.setCoordinates(coordinates);
}
```

**4. Комментарии**  

По возможности, заменяй комментарии поясняющими переменными или вспомогательными методами, которые своим названием будут объяснять, что происходит
```
if (steps == coordinatesForMoving.size() - 1) {//если creature уже находится на расстоянии одной клетки от еды
  //...
}

//ЛУЧШЕ:
if (isNear(coordinatesForMoving, steps)) {
  //...
}
```
Когда в проекте много каментов, это плохо- пользы от них практически нет, они только забивают пространство и мешают читать код.
В идеале, комментариев вообще не должно быть, код должен объяснять сам себя через правильный нейминг и лаконичный код.  
*Мартин, "ЧК", гл.4*  

**5. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется - в этом случае неважно, будет else или нет, так как программа будет работать одинаково и код выглядит читабельней
```
if (entity instanceof Herbivore) {
  resultSptite = Sprite.HERBIVORE;
} else if (entity instanceof Grass) {
  resultSptite = Sprite.GRASS;
} 

//ПРАВИЛЬНО:
if (entity instanceof Herbivore) {
  resultSptite = Sprite.HERBIVORE;
} 
if (entity instanceof Grass) {
  resultSptite = Sprite.GRASS;
} 
```

**6. record Coordinates**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**7. class GameBoard**

+ 👍 Валидация координаты при всех операциях с участием координаты.

- В метод валидации координаты не нужно передавать имя метода, из которого вызывается валидация. 
В исключения не нужно передавать имя метода, который вызвал исключение
```
public void setEntity(Coordinates coordinates, Entity entity) {
  validateCoordinates(coordinates, "setEntity");
  //...
}  

public void validateCoordinates(Coordinates c, String methodName) {
  //...
  throw new InvalidCoordinateException(
    String.format("[%s] Invalid coordinates (%d, %d). Field size: %dx%d",
      methodName,  <-- в исключение передается имя метода, который вызвал валидацию
      //...
}
```
Потому что произойдет исключение и оно не будет перехвачено, в Stack Trace будет распечатана последовательность вызовов методов, которые привели к появлению исключения- для этого трассировка стека и придумана.
```
Exception in thread "main" simulation.exceptions.InvalidCoordinateException: [setEntity] Invalid coordinates (100, 100). Field size: 10x10
	at simulation.GameBoard.validateCoordinates(GameBoard.java:111)
	at simulation.GameBoard.setEntity(GameBoard.java:28)   <--- ТУТ ВИДНО ОТКУДА ПРИЛЕТЕЛО
```

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: getRandomFreeCoordinates(), getAllGameBoardCoordinates(), isFood(...) etc.

Методы чужих ответственностей должны находиться в тех классах, в интересх которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Нарушение SRP. 
Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретным классом представителей и не должна даже знать по именам наследников Entity.  
Тем более Карта не должна быть в курсе проблем, связанных этими наследниками- достаточно этих наследников хранимого класса для игровой логики или нет. 
Все эти проблемы не имеют отношения к единой ответственности карты- хранению/выдаче существ
```
  public boolean isGrassEnough() {
     return getCertainEntities(Grass.class).size() > ActionUtils.HERBIVORE_LIMIT;  
  }
  public boolean isHerbivoreEnough() {...}
```

- Нарушение Low Coupling. Карта не должна знать о существовании класса ActionUtils, потому что ответственность по хранению/выдаче существ к экшенам не имеет никакого отношения
```
return getCertainEntities(Herbivore.class).size() > ActionUtils.PREDATOR_LIMIT;
```

+ 👍 Этот метод пусть тут будет, но я бы его переименовал например в `getEntitiesBy(Class<T> entityType)`
```
public <T extends Entity> List<T> getCertainEntities(Class<T> entityType)
```

❌ В целом, класс карты здесь это божественный объект(антипаттерн), который занимается совершенно разными вещами.

**8. class PathFinder**

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar. 
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно. 
Например путем анализа существа, которое находится на стартовой координате: поиск знает где в карте брать информацию о еде этого существа и на основе этой информации ведет поиск пути
```
public List<Coordinates> searchFood(Coordinates start) {
  //...
  if (board.isFood(current, currentCreature)) {...}
  //...
}
```
Таким образом, класс поиска в курсе логики более высокого порядка, знает зачем осуществляется поиск и где брать данные для этого.
Все эти проблемы не должны волновать этот класс. Класс должен все необходимые данные для поиска получать в аргументы метода поиска.  
Например, так
```
class PathFinder {
  private final GameBoard board;
  //...

  public List<Coordinates> getPatch(Coordinates start, Class<? extends Entity> target) {...}
}
```

**9. Пакет models**, хранит Entity и его потомки

+ 👍 В целом, особых зачечаний нет.

**10. class ActionUtils**

Гибрид класса констант и утилитного класса. Одни классы используют его как класс констант, другие- как утилитный.  
Для начала стоит разделить этот класс на два: на константный и на утилитный, а дальше будет видно, что с ними делать.  
Про классы констант, конфигурации и их использование я писал тут: https://t.me/zhukovsd_it_chat/53243/176984  
Еще про константные классы: *https://www.baeldung.com/java-constants-good-practices*  

**11. Пакет setup_actions**

Содержит группу класcов, которые делают одно и то же- заселяют карту конкретным видом существ в заданном количестве.
Вместо отдельного класса на каждый вид существ, нужно сделать универсальный класс, который будет принимать в в себя количество создаваемых существ и способ их создания.
Способ создания можно передавать через стандартный интерфейс Supplier. Например, так
```
SpawnAction заяцSpawnAction = new SpawnAction(() -> new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ);
SpawnAction волкSpawnAction = new SpawnAction(() -> new Волк(), КОЛИЧЕСТВО_ВОЛКОВ);
заяцSpawnAction.perform(worldMap); 
волкSpawnAction.perform(worldMap); 
//...

public class SpawnAction extends Action {
  private final Supplier<Entity> entitySupplier;
  private final int amount;

  public SpawnAction(Supplier<Entity> entitySupplier, int amount) {
    this.entitySupplier = entitySupplier;
    this.amount = amount;
  }

  @Override
  public void execute(Map map) {
    for (int i = 0; i < amount; i++) {
      Entity entity = entitySupplier.get();
      //добавить существо в карту
    }
  }
}
```

**12. class SetupPredator(Rock/Tree/...)Action extends Action**

Неправильное использование класса констант
```
board.setEntity(randomFreeCoordinates, new Predator(randomFreeCoordinates, ActionUtils.getRandomSpeed(), ActionUtils.getRandomHealth(), ActionUtils.getRandomAttackPower()));
```
Класс не должен лезть напрямую в класс констант за данными и таким образом иметь зависимость на этот класс. Класс должен принимать необходимые ему данные в конструктор. 

Условный пример
```
public final class Settings {
  //приватный конструктор
  public static final int HOUSE_ROOMS = 5;
}

//ПЛОХО:
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

**13. class AddGrass(Herbivore)Action extends Action**

- Дублирование в классах, как и в предыдущем случае. Именно этот класс должен выссчитывать, когда ресурса(травы или зайцев) в карте недостаточно, а не заставлять это знать карту 
```
@Override
public void execute() {
  if (!board.isGrassEnough()) {  <-- ТУТ
    new SetupGrassAction(board).execute();
  }
}
```
Тут принцип тот же, что и с Setup-Action: сделать универсальный класс Пополнятор, который в себя будет принимать необходимые данные для пополнения ресурсов.  
Общий принцип мне видится таким
```
public class Пополнятор<T extends Entity> extends Action {
  //...
  public Пополнятор(GameBoard board, Class<T> entityClazz, Supplier<T> entitySupplier, int upperLimit) {...}

  @Override
  public void execute() {
    List<T> entities = board.getCertainEntities(entityClazz);
    if(!isEnough(entities.size())) {  //зная текущее количество и верхний лимит легко понять, достаточно существ или нет
      int amont = //рассчитать необходимое количество существ для пополнения 
      SpawnAction spawnAction = new SpawnAction(entitySupplier, amount);
      spawnAction.execute();
    }
  }
  //...
}
```

**14. class Sprite**

Константы спрайтов существ. В контексте этой реализации не вижу смысла в отдельном классе с константами спрайтов. 
Выглядит, как искуственное разделение единой сущности Рендерера на два класса: собственно рендерер и константы для рендерера.

**15. class GameBoardRenderer**

- Нужно перенести сюда спарйты из константного класса.

- Плюсование строк в цикле это расточительно- постоянно пересоздаются объекты. 
Посмотри, idea подчеркивает здесь плюсование желтым цветом, это она предлагает заменить складывание строк использованием стрингбилдера.
Поставь курсор на желтое подчеркивание, слева появится желтая лампочка, клацни на нее, выбери пункт "Convert ..." и идея автоматически заменит плюсование на стрингбилдер
```
String line = "";
for (...) {
  line += getSpriteForEmptyCell();
}

//ПРАВИЛЬНО:
StringBuilder line = new StringBuilder();
for (...) {
  line.append(getSpriteForEmptyCell());
}  
```

- Избыточно. В таких случаях сразу ретурняй
```
String resultSptite = null;
if (entity instanceof Herbivore) {
  resultSptite = Sprite.HERBIVORE;
} else if (entity instanceof Grass) {
  resultSptite = Sprite.GRASS;
} else if (entity instanceof Rock) {...}

if (resultSptite == null) {
  throw new IllegalArgumentException("The entity is null or there is no sprite for current entity");
}
return resultSptite;

//ПРАВИЛЬНО:
if (entity instanceof Herbivore) {
  return Sprite.HERBIVORE;
} 
if (entity instanceof Grass) {
  return Sprite.GRASS;
} 
if (entity instanceof Rock) {...}

throw new IllegalArgumentException("The entity is null or there is no sprite for current entity");
```

**16. class Simulation**

+ 👍 Достаточное количество констант.

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий. 
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе симуляции
```
public Simulation(GameBoard board, List<Action> initActions, List<Action> turnActions)
```

**17. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

В принципе, не так плохо. Но нужно разобраться с принципом единой ответственности(SRP) при создании классов.  
Для этого посмотреть ролики Cергея про шахматы и ролики Немчинкого про SOLID.

n.69(155)  
#ревью #симуляция