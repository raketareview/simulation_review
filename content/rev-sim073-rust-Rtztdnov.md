https://github.com/Rtztdnov/Simulation  
[Rust]

Есть над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(просто мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию
```
HashMap<Coordinates, Entity> entitiesMap = new HashMap<>();

//ПРАВИЛЬНО:
HashMap<Coordinates, Entity> entities = new HashMap<>();
```

- Метод обещает вернуть ход, но возвращает координату
```
Coordinates getMove(Coordinates coordinates, WorldMap worldMap)
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Entity> entitiesMap = new HashMap<>();
ArrayList<Coordinates> getneighbours(Coordinates coordinates) 


//ПРАВИЛЬНО:
Map<Coordinates, Entity> entitiesMap = new HashMap<>();
List<Coordinates> getneighbours(Coordinates coordinates) 
```

**3. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется - в этом случае неважно, будет else или нет, так как программа будет работать одинаково и код выглядит читабельней
```
if (...) {
  attack(way, worldMap);
  return coordinates;
} else {
  return way.get(getSpeed());
}

//ПРАВИЛЬНО:
if (...) {
  attack(way, worldMap);
  return coordinates;
} 
return way.get(getSpeed());
```

**4. В таких случаях сразу возвращай значение**
```
boolean inRange;
if (...) {
  inRange = true;
} else {
  inRange = false;
}
return inRange;

//ПРАВИЛЬНО:
if (...) {
  return true;
} 
return false;
```

**5. Не старайся засунуть все в одну строку в ущерб читаемости кода**. Что происходит в этом коде- совершенно непонятно, пока его не разложишь в нормальном виде
```
if (worldMap.getEntitiesMap().get(target) instanceof Prey) {
  ((Prey) worldMap.getEntitiesMap().get(target)).takeDamage(getPower());
}

//ПРАВИЛЬНО:
Entity entity = worldMap.getEntitiesMap().get(target);
if (entity instanceof Prey prey) {
  prey.takeDamage(getPower());
}
```

**6. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```
InitAction initAction = new InitAction(worldMap, 5, 10, 15, 8);
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"* 

**7. class Coordinates**

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x]

- Неправильное использование метода. В таких случаях метод должен быть static
```
Coordinates coordinates = new Coordinates().getRandomCoordinates(column, row);

//ПРАВИЛЬНО:
Coordinates coordinates = Coordinates.createRandomCoordinates(column, row);
```

Поэтому в координате не должно быть пустого конструктора
```
public Coordinates(int column, int row) {
  this.column = column;
  this.row = row;
}

public Coordinates() { <-- ЭТОТ КОНСТРУКТОР ЛИШНИЙ И БЕССМЫСЛЕННЫЙ
}
```

- Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: row,column.
В принципе, этот класс даже может быть record'ом.
Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия, тем самым нарушая SRP: `Coordinates getRandomCoordinates(int maxColumn, int maxRow)`

Создать рандомную координату это наверное очень полезно для проекта, но это не имеет никакого отношения к единой ответственности Координаты по идентификации точки в пространстве.

**8. class WorldMap**

- Эти поля не должны быть static
```
public static int column;
public static int row;
```

Статические поля в ООП программе нужно применять очень осторожно. 
Потому что сама идея классов состоит в том, что у класса может быть несколько экземпляров одновременно. И при этом все они должны корректно работать.
В данном случае если в программе будет существовать несколько экземпляров карты одновременно, они будут работать некорректно, потому что будут одновременно вносить изменения в одно и то же статическое поле.

И тут вопрос не в том, должно ли в проекте быть два экземпляра одного класса, или не должно. 
Если класс не синглтон, то по умолчанию мы признаем, что его экземпляров в проекте может быть более одного.

Пока не разобрался с ООП, лучше не делай статических полей в классах вообще, кроме констант.

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
private HashMap<Coordinates, Entity> entitiesMap = new HashMap<>();

public HashMap<Coordinates, Entity> getEntitiesMap() {
  return entitiesMap;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта(100, 100);
//заселить карту существами
карта.getEntitiesMap().clear(); //геноцид минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

- Никогда не возвращай null
```
public Entity getEntity(Coordinates coordinates) {
  return entitiesMap.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  

- Нарушение DRY, повторяющиеся куски кода выноси во вспомогательные методы
```
 public void addEntity(Entity entity, Coordinates coordinates) {
  if (entity != null && coordinates != null) {...}
  //...  
 }

public void addEntityRandomCoordinates(Entity entity) {
  if (entity != null && coordinates != null) {...}
  //...
}
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение.
```
public void removeEntity(Coordinates coordinates) {
  entitiesMap.remove(coordinates);
}
```

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
Creature creature = new Заяц();
карта.putEntity(new Coordinates(0, 0), creature);
карта.moveEntity(creature, new Coordinates(0, 0), new Coordinates(99, 99));

/* class WorldMap */
moveEntity(Entity entity, Coordinates from, Coordinates to)
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.

Здесь методы чужих ответственностей: `int getNumberOfCreatures()`, `void addEntityRandomCoordinates(Entity entity)`.  
Наверное, подсчет количества креатур это важно для проекта, но это не имеет никакого отношения к единой ответственности Карты по хранению существ. 
Этот метод использует `class Movement`, значит, метод должен находиться где-то там.

**9. class BFSAlgorithm**

- Нарушение инкапсуляции. В данном случае, поле должно быть private: `WorldMap worldMap;`

- Метод `findTheWay(...)` абсолютно нечитаем из-за чрезмерной вложенности блоков for-if-if
```
for (...) {
  if (...) {
    if (...) {
      if (...) {
        if (...) {
```
Не должно быть более 2-3 уровней вложенности. Если получается больше, вводи вспомогательные методы.

- Очень странно, что корректными в карте считаются координаты, начиная с 1,1 а не с 0,0
```
if (1 <= column && column <= maxColumn && 1 <= row && row <= maxRow) {
  inRange = true;
}
```
Википедия по этому поводу пишет:
```
В декартовой системе координат, начало координат — это точка, в которой пересекаются все оси координат. 
Это означает, что все координаты этой точки равны нулю. 
Например, на плоскости она имеет координаты (0,0)
```

- Индусский код
```
ArrayList<Coordinates> neighbours = new ArrayList<>();

Coordinates neighbour1 = new Coordinates(column - 1, row - 1);
neighbours.add(neighbour1);
Coordinates neighbour2 = new Coordinates(column, row - 1);
neighbours.add(neighbour2);
//еще миллион таких пар

//ПРАВИЛЬНО:
private final static List<Coordinates> SHIFT_COORDINATES = List.of{ new Coordinates(0, 1), new Coordinates(1, 1), ...};
  //...
  List<Coordinates> neighbours = new ArrayList<>();

  for(Coordinates shift: SHIFT_COORDINATES) {
    Coordinates neighbour = new Coordinates(column + shift.column, row + shift.row);
    neighbours.add
  }
```

**10. abstract class Creature extends Entity implements Damageable**

- Конструктор по конвенции всегда должен стоять выше остальных методов.

- Что будет, если после получения повреждения, hp будет отрицательным? Что будет, если в поле метода передадут отрицательный damage?
```
  public void takeDamage(int damage) {
    hp -= damage;
  }
```

**11. class Predator extends Creature**

Не пересоздавай обект каждый раз при вызове метода, создай его один раз
```
public Coordinates getMove(Coordinates coordinates, WorldMap worldMap) {
  BFSAlgorithm bfsAlgorithm = new BFSAlgorithm(worldMap);
  //...
}

//ПРАВИЛЬНО:
private final BFSAlgorithm bfsAlgorithm = new BFSAlgorithm(worldMap);
```

**12. class Prey extends Creature**

- Почему Травоядное называется Добычей? 
Суть класса состоит не в том, что он является добычей для кого-то другого. По этой логике Трава тоже добыча добычи.  
Суть класса в том, что это травоядное.

- Дублирование кода в классах зайца и волка
```
@Override
public void attack(LinkedList<Coordinates> way, WorldMap worldMap) {
  Coordinates target = way.getLast();
  if (worldMap.getEntitiesMap().get(target) instanceof Prey) {
    ((Prey) worldMap.getEntitiesMap().get(target)).takeDamage(getPower());
  }
}

@Override
public void attack(LinkedList<Coordinates> way, WorldMap worldMap) {
  Coordinates target = way.getLast();
  if (worldMap.getEntitiesMap().get(target) instanceof Grass) {
    ((Grass) worldMap.getEntitiesMap().get(target)).takeDamage(getPower());
  }
}
```

 Общий код выноси в предка
 ```
 public void attack(LinkedList<Coordinates> way, WorldMap worldMap) {
  Coordinates target = way.getLast();
  Entity entity = worldMap.getEntitiesMap().get(target);
  if (entity instanceof Damageable damageable) {
    if(isFood(damageable)) {
      damageable.takeDamage(getPower());  
    }
  }
}

protected boolean isFood(Damageabley damageable);
```

**13. Action'ы**

- Идея Actions здесь не осмыслена. Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.
Actions, изложенный в ТЗ, это вариант реализации паттерна Command.

Должен быть общий класс/интерфейс Action и его наследники. 
Это своего рода вариация паттерна Command.
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

В Action'ах должен быть только один публичный метод, а не куча, как сейчас.
Отдельно рассматривать каждый из представленных тут экшенов не имеет смысла- каждый из них плох, делает много разного, а не одно дело.

- Нарушение DRY
```
public void grassFactory(WorldMap worldMap, int amount) {
  for (int i = 0; i < amount; i++) {
    Grass grass = new Grass();
    worldMap.addEntityRandomCoordinates(grass);
  }
}

public void rockFactory(WorldMap worldMap, int amount) {
  for (int i = 0; i < amount; i++) {
    Rock rock = new Rock();
    worldMap.addEntityRandomCoordinates(rock);
  }
}
//еще миллион дублей

//ПРАВИЛЬНО:
spawn(worldMap, ()->new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ);

public void spawn(WorldMap worldMap, Supplier<Entity> entitySupplier, int amount) {
  for (int i = 0; i < amount; i++) {
    Entity entity = entitySupplier.get();
    worldMap.addEntityRandomCoordinates(entity);
  }
}
```

**14. class Renderer**

Не нужно получать отдельно размеры карты в конструктор. Размеры карты бери непосредственно из карты
```
public Renderer(WorldMap worldMap) {
  this.worldMap = worldMap;
  this.maxColumn = worldMap.getColumn();
  this.maxRow = worldMap.getRow();
}

  //...
  for (int row = 1; row <= maxRow; row++) {...}

//ПРАВИЛЬНО:
public Renderer(WorldMap worldMap) {
  this.worldMap = worldMap;
}

  //...
  for (int row = 1; row <= worldMap.getRow(); row++) {...}
```

**15. class Simulation**

- Нарушение инкапсуляции. Всегда явно указывай область видимости переменных и методов.

- Нарушение DI. Класс не должен инициализировать сам себя. 
В данном случае класс инициализирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```
WorldMap worldMap = new WorldMap(20, 10);
```
Таким образом нельзя создать несколько игровых конфигураций с разными картами.  
*Мартин, "ЧК", гл.11.2*

## ВЫВОД

Сделать нормальные Action'ы.  
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.73(161)  
#ревью #симуляция