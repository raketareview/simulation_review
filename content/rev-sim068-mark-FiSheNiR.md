https://github.com/FiSheNiR/Simulation  
[Марк]

Есть над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Используются стримы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Названия пакетов пишутся стилем lower_snake: "Actions" -> "actions".

- Название должно быть информативным, но лакончиным
```
Coordinates.horizontalPosition
Coordinates.verticalPosition

//ПРАВИЛЬНО:
Coordinates.row
Coordinates.column

//ИЛИ:
Coordinates.x
Coordinates.y
```

- Название должно как можно лучше объяснять, что хранит переменная
```
Settings.HORIZONTAL_BOTTOM_BOUND
Settings.VERTICAL_BOTTOM_BOUND

//ЛУЧШЕ:
Settings.GAME_MAP_HEIGHT
Settings.GAME_MAP_WIDTH
```

- Если в проекте есть класс GameMap то все переменные с именем, включающим это название, должны быть экземплярами этого класса
```
/*class GameMap*/
public HashMap<Coordinates, Entity> getCurrentGameMap()

//ПРАВИЛЬНО:
/*class GameMap*/
public HashMap<Coordinates, Entity> GameMap.getEntities()
```

- Почему удалитьСущество но получитьСуществоПоКоординате, хотя по координате происходит и удаление и получение существа? 
Избыточный контекст
```
public removeEntity(Coordinates coordinates)
public Entity getEntityByCoordinate(Coordinate coordinate)

//ПРАВИЛЬНО:
public void remove(Coordinate coordinate) 
public Entity get(Coordinate coordinate)
```

- Название метода вводит в заблуждение. Этот метод не преобразует поля в проверки. Метод создает сдвиговые координаты
```
Set<CoordinateShift> fieldsToCheck()

//ПРАВИЛЬНО:
Set<CoordinateShift> createShiftCoordinates()
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

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```
System.out.println("Press 'p' to pause or 'r' to resume:");
if (input.equalsIgnoreCase("p")) {
  //...        
} else if (input.equalsIgnoreCase("r")) {
  //...   
}
          
//ПРАВИЛЬНО:
private final static String PAUSE = "p";
private final static String RESUME = r";

System.out.printf("Press '%s' to pause or '%s' to resume: ", PAUSE, RESUME);
if (input.equalsIgnoreCase(PAUSE)) {
  //...        
} else if (input.equalsIgnoreCase(RESUME)) {
  //...   
}
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"*  

**4. class Settings**, константный класс

- Константный класс должен быть final и иметь приватный конструктор. Это нужно для того, чтобы не было возможности унасдоваться от константного класса или сделать экземпляр этого класса.

- Константный класс здесь используется неправильно- другие классы напрямую лезут в константы вместо того, чтобы получать необходимые этим классам данные в свой конструктор.  
Это делает классы неуниверсальными и зависимыми от класса констант.  

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
Про классы констант, конфигурации и их использование я писал тут: https://t.me/zhukovsd_it_chat/53243/176984

**5. class Coordinates**

- При прочих равных используй примитивные типы, а не классы обертки
```
public final Integer horizontalPosition;
public final Integer verticalPosition

//ПРАВИЛЬНО:
public final int horizontalPosition;
public final int verticalPosition
```
Для использования обертки должна быть причина и выгода, которую дает применение этой обертки.
Тут выгоды никакой нет.

- Сдвиг(shift) или складывание(add) координаты должно происходить с экземпляром своего класса. Заводить для этого специальный класс сдвиговой координаты не надо
```
public Coordinates shift(CoordinateShift shift)

//ПРАВИЛЬНО:
public Coordinates shift(Coordinates shiftCoordinates) 
```
Обычную координату можно использовать как координату сдвига, например так
```
Coordinates coordinates = new Coordinates(10, 10);
Coordinates shiftCoordinates = new Coordinates(-1, -1);  //сдвиг влево-вниз
coordinates = coordinates.shift(shiftCoordinates);
```

- Нарушение SRP и Low Coupling. Координата не должна определять, можно ли ее сдвинуть в интересах какого-то другого класса или нельзя.
Потому что это не касается единой ответственности координаты- хранение информации для идентификации точки в пространстве
```
public boolean canShift(CoordinateShift shift) {  <-- проверка "возможноси сдвига" - чужая ответственность
  //...
  v >= Settings.VERTICAL_BOTTOM_BOUND <-- лишняя зависимость на Settings - нарушение Low Coupling
  //...
  }
```

**6. class CoordinateShift**, координата для сдвига

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates. Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```
Coordinates shifDownRightCoordinates = new Coordinates(-1, 1);
```

**7. class GameMap**

- Никогда не возвращай null
```
public Entity getEntityByCoordinates(Coordinates coordinates) {
  return entities.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
private final HashMap<Coordinates, Entity> entities = new HashMap<>();

public HashMap<Coordinates, Entity> getCurrentGameMap() {
  return entities;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта();
//заселить карту существами
карта.getCurrentGameMap().clear(); //геноцид минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность
```
private void removeEntity(Coordinates coordinates) {
  entities.remove(coordinates);
}
```
Если координата некорректна(находится вне пределов карты), нужно бросать исключение.

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта();
карта.setEntities(new Coordinates(0, 0), new Заяц());
карта.moveEntity(new Coordinates(0, 0), new Coordinates(99, 99));

/* class GameMap */
public void moveEntity(Coordinates from, Coordinates to)
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Свои собственные размеры по ширине и высоте карта не должна хранить в другом классе, например в константном. Карта должна свои размеры получать в конструктор. 
Даже если в проекте есть класс с константами(напр. Settings), то третий класс должен прочитать эти константы из константного класса и передать в конструктор Карты.

Сейчас Карта в принципе не знает собственных размеров, а ее размеры хранит кто-то другой.  
Должно быть ровно наоборот, только карта должна хранить в себе свои размеры. А все остальные классы должны эти данные получать только из самой карты.

**8. class BFS**, поиск пути

- Нарушение SRP. Класс поиска пути должен заниматься поиском пути. А здесь есть какие-то левые методы, типа выдать случайную координату сдвига
```
public Coordinates getRandomShiftCoordinates(Coordinates coordinates)
```
Наверное, выдача случайной сдвиговой координаты это очень полезно. Но это не имеет отношения к процессу поиска пути. Внутри класса этот метод не используется.  
А если бы использовалься, то должен был бы быть private.
С целесообразностью нахождения других публичных методов тут тоже нужно разбираться. Единственный публичный метод, который точно должен быть тут, это `findPathToTarget(...)`.

- Избыточно и сложно. Просто сделай константы, а не пересоздавай объекты каждый раз
```
private Set<CoordinateShift> fieldsToCheck() {
  Set<CoordinateShift> result = new HashSet<>();
  for (int fileShift = -1; fileShift <= 1; fileShift++) {
    for (int verticalShift = -1; verticalShift <= 1; verticalShift++) {
      if ((fileShift == 0) && (verticalShift == 0)) {
        continue;
      }
      result.add(new CoordinateShift(fileShift, verticalShift));
    }
  }
  return result;
}

//...
for (CoordinateShift shift : fieldsToCheck()) {...}

//ПРАВИЛЬНО:
private static final List<CoordinateShift> SHIFT_COORDINATES = List.of(
    new CoordinateShift(1, 0),
    new CoordinateShift(1, 1),
    //oth's
);

//...
for (CoordinateShift shift : SHIFT_COORDINATES) {...}
```

**9. abstract class Entity**

Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня Creature.

**10. class Herbivore/Predator extends Creature**

- Тоже напрямую лезет в класс констант за данными, которые класс должен получать в конструктор
```
public Predator(Coordinates coordinates) {
  super(coordinates, Settings.BASE_PREDATOR_SPEED_RATE, Settings.BASE_HEALTH);
  this.attackRate = Settings.BASE_ATTACK_RATE;
}
```

- Дублирование кода. Одинаковый метод `makeMove(...)` в Зайце и Волке
```
public class Herbivore extends Creature {
  //...

  @Override
  public void makeMove(GameMap gameMap) {
    for (int i = 0; i < speed; i++) {
      move(this.coordinates, gameMap, Herbivore.class);  <-- так в зайце
      //move(this.coordinates, gameMap, Predator.class);  <-- так в волке
    }
  }
}
```
Общий код потомков выноси в их предка. Например, так
```
public abstract class Creature extends Entity implements Movable {
  //...  
  public void makeMove(GameMap gameMap) {
    for (int i = 0; i < speed; i++) {
      move(this.coordinates, gameMap, this.getClass()); 
    }
  }
}
```

**11. class MoveEntityAction implements Action**

👍 Только вызывает движение во всех креатурах, это хорошо.

**12. class SpawnEntityAction implements Action**

- Применение рефлексии в прикладных классах обычно не оправдано. В этом классе- уж точно
```
private <T extends Entity> T spawnEntity(Class<T> clazz, Coordinates coordinates) {
  try {
    Constructor<T> constructor = clazz.getConstructor(Coordinates.class);
    return constructor.newInstance(coordinates);
  } catch (Exception e) {...}
}
```
Рефлексия не является корректным способом строить бизнес логику программы.  
Рефлексия может использоваться в юнит тестах для тестирования приватных методов классов. 
В остальных случаях ее применение нужно избегать, если ты конечно не строишь свой фреймворк а-ля Спринг. 

Если нужно создать универсальный спавнер, используй стандартные интерфейсы явы
```
SpawnAction заяцSpawnAction = new SpawnAction(() -> new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ);
заяцSpawnAction.perform(worldMap); 
//...

public class SpawnAction extends Action {
  private final Supplier<Entity> entitySupplier;
  private final int amount;

  public SpawnAction(Supplier<Entity> entitySupplier, int amount) {...}

  @Override
  public void execute(Map map) {
    for (int i = 0; i < amount; i++) {
      Entity entity = entitySupplier.get();
      //добавить существо в карту
    }
  }
}
```

**13. class MapConsoleRenderer**

В switch-case в default нужно кидать исключение. Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
return switch (entity.getClass().getSimpleName()) {
  case "Herbivore" -> Settings.HERBIVORE_EMOJI;
  case "Predator" -> Settings.PREDATOR_EMOJI;
  //...
  default -> ""; <-- если в проект добавим Мамонта, а в распечатку нет, то вместо мамонта напечатает пустоту
};
```

**14. class Simulation**

Класс должен принимать необходимые зависимости в конструктор- как минимум, экземпляр карты. Сейчас у карты есть только конструктор по умолчанию.  
Как только ты сделаешь нормальную карту, которая будет принимать свои размеры в конструктор, то эту карту нужно будет создавать в майне, там же создавать экземпляр Симуляции и инжектить ей в конструктор эту карту.

**15. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

P.S. У кого вы все друг за другом подсматриваете `class CoordinateShift` и `boolean canShift(CoordinateShift shift)`? Не подсматривайте больше.

n.68(154)  
#ревью #симуляция