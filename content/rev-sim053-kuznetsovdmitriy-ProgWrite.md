https://github.com/ProgWrite/Simulation.git  
[Кузнецов Дмитрий]

После рефакторинга. Не все прошлые замечания устранены.

## ХОРОШО

1. Все хорошее, что было хорошо в предыдущей версии
2. Симуляция принимает в конструктор список действий- можно делать разные конфигурации

## ЗАМЕЧАНИЯ

**1. Нейминг**

-В яве нзвания интерфейсов не пишут с префиксом "I", так принято делать в C#
```
interface IGetTargetCoordinates
```
В яве названия интерфейсов пишут либо с постфиксом "able", например Runnable. Либо просто существительное, например List.

-В ТЗ указано название в единственном числе: Action
```
class Actions
```

-Название "существа" больше подходит для списка существ или массива существ. Здесь карту мира лучше назвать картой мира.
В данном случае это не венгерская ноттация, потому что слово "gameMap" объясняет не способ хранения сущности, а концепцию сущности
```
GameMap entities

//ПРАВИЛЬНО:
GameMap gameMap
```

Венгерская ноттация: Entity[] arrayEntities, List<Entity> listEntities, int iAge, String stringName и т.д.  
Не венгерская ноттация: GameMap gameMap, Entity entity, Action action и т.д.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  

**2. Текст в exception** всегда должен быть на английском языке. Исключение это не просто телеграмма, которая летит сквозь слои. 
У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты. 
А значит, сообщение должно быть на английском. 

Интерпритация исключения и перевод его на локальный язык должно происходить там, где это соответствует архитектуре программы или не происходить вовсе, если исключение не планируется перехватывать
```
throw new IllegalArgumentException("Высота карты не может быть равна нулю");
```

**3. class Coordinates**

+Хорошо, что координаты называются row и column а не x и y. 
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

-Класс можно было бы сделать рекордом- для такого простого класса, как Координата, это идеальный вариант. 
К тому же рекорды умеют по умолчанию делать хешкод и equals, и их не нужно будет переопределять, как это сейчас сделано в Coordinates.

-Лучше, если Координаты будут immutable
```
public class Coordinates {
  public int row;
  public int column;
  //...
}

//ЛУЧШЕ ТАК:
public class Coordinates {
  public final int row;
  public final int column;
  //...
}

//ИЛИ ТАК:
public class Coordinates {
  private final int row;
  private final int column;
  //...
  //геттеры для row и column
}
```

**4. class GameMap**

+В классе имеются только нужные методы, то есть те, которые используются в единой ответственности класса. 
Хотя я бы еще сюда добавил public Coordinates getCoordinates(Entity entity)

-Валидация значения должна происходить при вводе данных, а не при выдаче данных. То есть в данном случае- в конструкторе, а не в геттере
```
public int getHeight() {
 if (height == 0) {
   throw new IllegalArgumentException("Высота карты не может быть равна нулю");
 }
 return height;
}
```

-Карта должна разрешить вставить существо по координате в том случае, если координата находится в пределах ширины и высоты карты.
При этом неважно, будет эта ячейка занята или нет. Разрешать вставлять существо на занятую ячейку или не разрешать- это ответственность не карты, а игровой логики и должна находиться в том классе, который эту логику реализует.
Здесь карта должна проверять только те условия, которые касаются непосредственно ее ответственности, в данном случае- находится ли координата в допустимом диапазоне
```
public void setEntity(Coordinates coordinates, Entity entity) {
  if (isCoordinatesValid(coordinates) && isSquareEmpty(coordinates)) {
    entities.put(coordinates, entity);
  } 
  //...
}

//ПРАВИЛЬНО:
public void setEntity(Coordinates coordinates, Entity entity) {
  if (isCoordinatesValid(coordinates)) {
    entities.put(coordinates, entity);
  } 
  //...
}
```

-Не вижу смысла в этом перехвате исключения. Этот перехват ничего не перехватит, потому что при вставке существа в хэшмап никогда не вылетит exception.
Хэшмап не бросает исключение при вставке пары ключ-значение, если уже есть пара с таким ключом
```
private final Map<Coordinates, Entity> entities = new HashMap<>();

public void setEntity(Coordinates coordinates, Entity entity) {
  //...
  try {
    entities.put(coordinates, entity);
  } catch (IllegalArgumentException e) {
    System.out.println("Coordinates isn't empty or out of bounds");
  }
  //...
}
```

Проверяем
```
public static void main(String[] args) {
  Map<Coordinates, Entity> map = new HashMap<>();
  Coordinates coordinates = new Coordinates(+100500, -10050);
  map.put(coordinates, new Tree(coordinates));
  map.put(coordinates, new Grass(coordinates));

  map.put(new Coordinates(1, 1), null);
  map.put(null, new Tree(null));
  map.put(null, null);

  System.out.println("Все ок");
}

//РЕЗУЛЬТАТ:
Все ок
```

-Здесь тоже
```
public void removeEntity(Coordinates coordinates) {
  //...
  try {
    entities.remove(coordinates);
  } catch (IllegalArgumentException e) {
    System.out.println("Entity not found" + e.getMessage());
  }
}
```

-В замечаниях к прошлой верссии программы я писал про другое. Методы сами должны кидать исключения в случае некорректной координаты, 
а не пытаться перехватывать какие-то другие исключения.
Нужно делать вот так
```
public void removeEntity(Coordinates coordinates) {
  if(!isCoordinatesValid(coordinates)) {
    throw new IllegalArgumentException(КООРДИНАТА_ВНЕ_ДОСКИ_MESSAGE);
  }
  entities.remove(coordinates); 
}
```
Логика здесь простая- если программа где-то в своих недрах генерирует некорректную координату и пытается ее всунуть в карту, 
значит это баг алгоритма и нужно аварийно приостановить работу программы путем выброса исключения. Для того, чтобы сразу узнать про баг и оперативно его устранить.

-Нарушение SRP, чужая ответственность, зависимость модели от представления.  
Модель(а это модель) не должна ничего печатать в консоль
```
System.out.println("Coordinates isn't empty or out of bounds");
``` 
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

-Очевидно, ты не так понял мои прошлые замечания - сейчас геттер все равно может вернуть null. Методы не должны возвращать null
```
public Entity getEntity(Coordinates coordinates) {
  if (isCoordinatesValid(coordinates)) {
    try {
     entities.get(coordinates);
    } catch (NullPointerException e) {
      System.out.println("Entity not found" + e.getMessage());
    }
  }
  return entities.get(coordinates);
}
```

Проверяем
```
public static void main(String[] args) {
  GameMap gameMap = new GameMap(10, 10);
  Entity entity = gameMap.getEntity(new Coordinates(0, 0));
  System.out.println("Entity= " + entity);
}

//РЕЗУЛЬТАТ:
Entity= null
```

Вот реализация, которую я имел ввиду и которая не возвращает null:
```
public Entity getEntity(Coordinates coordinates) {
  if(!isCoordinatesValid(coordinates)) {
    throw new IllegalArgumentException(КООРДИНАТА_ВНЕ_ДОСКИ_MESSAGE);
  }
  Entity entity = entities.get(coordinates);
  if(entity == null) {
    throw new IllegalArgumentException(НЕТ_ENTITY_ПО_КООРДИНАТЕ_MESSAGE);
  }
  
  return entity;
}
```

Так как во многих методах тут нужно проверять координату на корректность и бросать исключение если что не так, то нужно сделать вспомогательный метод
```
public void removeEntity(Coordinates coordinates) {
  validate(coordinates);
  entities.remove(coordinates); 
}

private void validate(Coordinates coordinates) {
  if(!isCoordinatesValid(coordinates)) {
    throw new IllegalArgumentException(КООРДИНАТА_ВНЕ_ДОСКИ_MESSAGE);
  }
}
```

**5. interface Eatable**  
Интерфейс, который сообщает, что его носитель Съедобен
```
public interface Eatable {
  void setEaten(boolean eaten);
}
```

-Непонятно, зачем интерфейс существует в таком виде. У него только один метод- ввести режим съеденности. 
То есть, setEaten(true) - существо съедено, setEaten(false) - существо не съедено. 
Ок, а где метод "сообщить, съеден ли он"?

Какую бы проблему мог решать такой интерфейс?  
Например я мог бы из списка всех наличных ентити смапить список всех съедобных существ:
из `List<Entity>` в `List<Eatable>`. Далее прошелся бы по списку съедобных и спросил у них, съедены ли они. 
Например, через метод boolean isСъеден().
И каждый из этих съеденных существ удалил бы из Карты
```
List<Entity> всеСущества = карта.getВсеСущества();
List<Eatable> съедобныеСущества = toCъедобныеСущества(всеСущества);
for(Eatable съедобный: съедобныеСущества) {
  if(съедобный.isСъеден()) {
    карта.удалить((Entity)съедобный);
  }
}
```

И тогда Заяц бы имел такой код
```
класс Заяц() реализует Съедобность{
  private final int hp;

  //...
  @Переопределить 
  public boolean isСъеден() {
    return hp <= 0;
  }
}
```

**6. public class Grass extends Entity**

-Содержит координату. Но координата нужна только тому существу, которое ходит, а трава тут не ходит.
Поэтому entity должен хранить координату только начиная с уровня Creature.

-Нарушение инкапсуляции, поле Координата не должно быть публичным
```
public class Grass extends Entity implements IGetTargetCoordinates, Eatable {
  public Coordinates coordinates;
  //...
}
```

-Класс реализует Cъедобность, но к ему приватному полю isEaten никто извне не обращается, а геттера нет. Зачем в таком случае здесь это нужно- неясно
```
public class Grass extends Entity implements IGetTargetCoordinates, Eatable {
  private boolean isEaten = false;
  //...
}
```

-Если класс переопределяет методы предка либо реализует методы интерфейса, всегда ставь соответствующую аннотацию. Это радикально улучшает читабельность кода
```
public class Grass extends Entity implements IGetTargetCoordinates, Eatable {
  //...
  public void setEaten(boolean isEaten) {
    this.isEaten = isEaten;
  }

  public Coordinates getCoordinates() {
    return coordinates;
  }
}

//ПРАВИЛЬНО
public class Grass extends Entity implements IGetTargetCoordinates, Eatable {
  //...
  @Override
  public void setEaten(boolean isEaten) {
    this.isEaten = isEaten;
  }

  @Override
  public Coordinates getCoordinates() {
    return coordinates;
  }
}
```

**7. class BreadthFirstSearch**

-Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям.
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно и не выгружать эти условия из Creture самостоятельно.
Поиск должен сам искать путь до цели, а не спрашивать у Creature, где находится цель
```
public List<Coordinates> findPath(Coordinates start, GameMap entities, Creature creature) {
  Coordinates target = creature.takeTargetCoordinates(creature, entities);
  //...
}

//ПРАВИЛЬНО:
public List<Coordinates> findPath(GameMap gameMap, Coordinates start, Class<T extends Entity> target) {
  //Алгоритм BFS: 
  //Перебирать все координаты начиная от start
  //как только найдется координата ячейки, где сидит существо класса target
  //то вернуть список координат от start до target
}
```

-Поиск не должен как-то по особенному искать путь для разных наследников Entity и знать наследников Entity по именам
```
if (... && checkForGrass) {...}
if (... && checkForHerbivore) {...}
if (creature instanceof Herbivore) {...}
if (creature instanceof Predator) {...}
```
Поиску должно быть достаточно знать класс цели- Class<T extends Entity> target. 
Поиск пути до цели должен происходить одинаково вне зависимости от интересанта поиска- будь то заяц, черепашка или колобок.

-Отмечать посещенные координаты в массиве- слишком избыточно
```
boolean[][] visitedCells = new boolean[entities.getHeight()][entities.getWidth()];
```
Представь, что у карты будет размер 100x100. Тогда каждый раз при поиске пути нужно будет создавать массив на 10.000 элементов.
Посещенные координаты можно хранить, например, в Set.

**8. abstract class Creature<T extends Entity> extends Entity**

-Нарушение инкапсуляции, поля не должны быть публичные
```
public abstract class Creature<T extends Entity> extends Entity implements Eatable {
  public boolean isEaten = false;
  public Coordinates coordinates;
  //...
}
```

-Нарушение конвенции кода, первым среди методов должен стоять конструктор.

-Бесполезный toString()- такой же, как у предка, а у предка его нет. То есть реализация по умолчанию- от класса Object. Этот код нужно убрать, разницы никакой не будет
```
@Override
public String toString() {
  return super.toString();
}
```

-Реализация поиска координат целей в карте это не отвественность Креатуры
```
public abstract Coordinates takeTargetCoordinates(Creature creature, GameMap entities);
```

Креатура только должна хранить информацию о том, каких существ оно ест, например так
```
private final Class<? extends Entity> food;
```

Далее класс цели нужно передавать в метод, который ищет путь на карте. Например, в такой
```
public interface PatchSearch {
  public List<Coordinates> findPath(Карта карта, Coordinates start, Class<? extends Entity> target);
}
```

И тогда если метод ходьбы делать в Creature, как указано в ТЗ, то это будет примерно так
```
class Creature бла-бла-бла {
  private final Class<? extends Entity> food;
  private final PatchSearch patchSearch;
  private Coordinates coordinates;

  public Creature(PatchSearch patchSearch, бла-бла) {
    this.patchSearch = patchSearch;
    //...
  }  
 
  public void makeMove(GameMap gameMap) {
    List<Coordinates> patch = patchSearch(gameMap, coordinates, food);
    //пройти нужное количество шагов по пути patch
    //если дошел до конца пути- значит пришел к еде и нужно ее есть 
  }
}
```

-Нарушение SRP, существо не должно реализовывать поиски каких-то координат
```
protected Coordinates findNearestCoordinates(Class<T> entityTarget, GameMap entities) {...}
```
Реализовывать поиск кординат здесь должен только класс поиска пути.

**9.class Herbivore extends Creature<Grass>**

-Нарушение SRP, класс не должен хранить и считать количество своих экземпляров
```
public static int startingHerbivoreCount = generateStartingCreatureCount(STARTING_MINIMUM_HERBIVORE, STARTING_MAXIMUM_HERBIVORE);

protected static int generateStartingCreatureCount(int minimumCreature, int maximumCreature) {
  Random random = new Random();
  return random.nextInt(maximumCreature - minimumCreature + 1) + minimumCreature;
}
protected void isCreatureDiedOfHunger() {
  startingHerbivoreCount--;
}
```
Зайцы в лесу не устраивают перекличку каждый раз, когда кто-то из них рождается или умирает. Пересчетом зайцев может заниматься лесник.

Так и здесь подсчитывать количество зайцев и волков нужно после каждого цикла ходов(Turn): взять из карты список всех существ и посчитать, сколько из них зайцев или волков. 

Понимаю, это кажется не настолько "оптимизированным" по сравнению со статическим счетчиком. Но в этом проекте приоритет состоит в создании правильной и понятной архитектуры. А не в оптимизации скорости выполнения гейм-процессов, что в большинстве случаев означает ухудшение архитектуры.
Сейчас статический учет зайцев/волков это сложный и неочевидный процесс: счетчик хранится в зайце, уменьшение счетчика тоже в зайце, а увеличение- в EntitySpawnerAction.

-Магические числа: 10, 5.

**10. class InitialSpawnAction extends Actions**

-Магические числа: 15, 7, 2.

**11. class MoveAllCreaturesAction extends Actions**

-Нарушение SRP. Кроме совершения хода, класс еще ведет учет креатур
```
creature.decreaseHealthByOne(startHealth)
```
 
**12. class MapConsoleRenderer**

-В switch-case в default нужно кидать исключение. 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
private String selectPictureForEntity(Entity entity) {
  switch (entity.getClass().getSimpleName()) {
    case "Grass":
      return "\uD83C\uDF3F ";
    //...
    default:
      return "?";
  }
}
```

**13. class Simulation**

+Список действий симуляция принимает в конструктор- можно делать разные игровые конфигурации с разными действиями, не меняя код Simulation. Это хорошо
```
public Simulation(GameMap entities, List<Actions> turnActions) 
```

## ВЫВОД

Некоторые прошлые замечания были поняты ошибочно и не устранены. Но после рефакторинга стало лучше.

n.53(121)   
#ревью #симуляция