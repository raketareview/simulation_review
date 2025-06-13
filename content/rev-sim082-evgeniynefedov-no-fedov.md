https://github.com/no-fedov/simulation  
[Evgeniy Nefedov]

В целом, неплохо.

## ХОРОШО

👍 Координаты существ не хранятся в самих существах(мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Это не счетчики, это ширина и высота
```
public class Field {
  private final int rowCount;
  private final int columnCount;
  //...
}

//ПРАВИЛЬНО:
public class Field {
  private final int height; //rows
  private final int width;  //columns
  //...
}
```

- Избыточный конекст. Из сигнатуры и так видим, что получаем Entity по Position
```
Optional<Entity> getEntityByPosition(Position position) 

//ЛУЧШЕ:
Optional<Entity> getEntity(Position position) 
```

- В таких случаях принято писать `Class clazz`
```
Entity generate(Class<?> classZ)

//ПРИНЯТО:
Entity generate(Class<?> clazz)
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"* 

**2. Текст в exception** всегда должен быть на английском языке
```
throw new RuntimeException("вышли за рамки поля");
```
Исключение это не просто телеграмма, которая летит сквозь слои. 
У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты. 
А значит, сообщение должно быть на английском. 

Интерпритация исключения и перевод его на локальный язык должно происходить там, где это соответствует архитектуре программы или не происходить вовсе, если исключение не планируется перехватывать

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы.

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов. 
Либо перенеси ее в третий класс- эти данные должны быть синхронизированы между собой
```
public class Simulation {
  private void nextTurn() {
   //...
    System.out.println("""
        Введите:
        S - для продолжения
        P - для паузы
        Q - для выхода
        """);
  }
}

public class ConsoleManagementSimulation {
  private static final String START = "S";
  private static final String PAUSE = "P";
  private static final String QUIT = "Q";
  //...
}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**4. class Position**, координата

+ 👍 Нет ничего лишнего, это хорошо. 

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

- Класс может быть преобразован в record без потери функционала.

**5. class Field**, координата

- Сложная система хранения существ- используется две мапы. Зачем- непонятно. Обычно для этого вполне достаточно одной
```
private final Map<Position, Entity> positionToEntity = new HashMap<>();
private final Map<Entity, Position> entityToPosition = new HashMap<>();
```

+ 👍 Геттеры не возвращают null
```
public Optional<Entity> getEntityByPosition(Position position)
public Optional<Position> getPositionByEntity(Entity entity)
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void remove(Position position) {
  Entity entity = positionToEntity.remove(position);
  if (entity != null) {
    entityToPosition.remove(entity);
  }
}
```

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: 
```
public boolean canFill() {
  return positionToEntity.size() / (double) (rowCount * columnCount) < 0.4d;
}
```

Наверное, для проекта в целом полезно иметь метод, который определяет... а что собственно определяет этот метод?  
Судя по названию, он определяет, можно ли заполнять карту. Если бы метод определял, что карта заполнена(`isFull()` или типа того), то ладно.

Но как я понял, метод учитывает некий крайний лимит пустых клеток, которые не должны быть заполнены. 
То есть, этот метод реализует какую-то логику, которая не имеет отношение к хранению данных в карте, но имеет отношение к логике игрового процесса в целом.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

**6. class PositionUtil**

- По сути методов, которые хранит этот класс, это не утилита позиции(координаты), а утилита поля(карты)
```
class PositionUtil

//ПРАВИЛЬНО:
class FieldUtil
```

- Если класс содержит только статические методы, то класс должен быть `final` и иметь приватный конструктор. 
Не должно быть возможности создать экземпляр такого класса или унаследоваться от него.

**7. class PathSearcherUtil**

Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям. 
В данном случае- до точки, в которой находится существо нужного класса.  
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно.
Здесь же поиск принимает в себя `entity`, в интересах которого он производит поиск, и анализирует этот entity для выявления критериев поиска
```
public static <T extends Entity & Movable> List<Position> searchPath(T entity, Position currentPosition, Field field) {
    //...
    if (entityForCheck.getClass().equals(entity.moveCondition())) {...}
    //...
}
```
Все необходимые данные для поиска пути класс должен принимать в себя, например так
```
public List<Coordinates> getPatch(Карта карта, Координата start, Class<? extends Entity> target) {...}
```

**8. class Grass/Rock/... extends Entity**

Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами.

Спрайты всех существ должны храниться в классе, который распечатывает карту.
Здесь спрайт изображения существа, в данном случае в виде буквы, хранится в самом существе
```
public class Grass extends Entity {
  @Override
  public String toString() {
    return "G";
  }
}
```

**9. class Herbivore/Predator extends Creature**

- toString() должен быть стандартным(как делает IDE по Alt+Ins) и использоваться только для отладки. 
toString() должен содержать значение всех значимых полей класса.
toString() не должен использоваться для представления, за исключением предельно простых классов, вроде PhoneNumber.
```
@Override
public String toString() {
  return "P";
}

//ПРАВИЛЬНО:
@Override
public String toString() {
  return "Predator{" +
    "attackPower=" + attackPower +
    ", speed=" + speed +
    ", health=" + health +
    '}';
}
```
*Блох, "Java эффективное программирование" гл.3.3*  

- Сигнатура метода определения еды выглядит странно
```
@Override
public Class<Grass> moveCondition() {
  return Grass.class;
}
```
Чтобы у метода появился смысл, он должен иметь такую сигнатуру
```
@Override
public Class<? extends Entity> moveCondition() {
  return Grass.class;
}
```
Тогда клиентский код может работать с любыми наследниками Creature и через полиморфизм получать от них любой класс еды, не привязываясь при этом к конкретному (напр. `Class<Grass>`).

**10. interface Action**

Класс(интерфейс) и метод в классе не могут называться одинаково. Название класса должно быть существительным, а метода- глаголом
```
public interface Action {
  void action(Field field);
}

//ПРАВИЛЬНО:
public interface Action {
  void execute(Field field);
}
```

**11. class SpawnEntityAction<T extends Entity> implements Action**

- Создавай вспомогательные методы, делай программу более простой и понятной.

Таких ацких условий не должно быть в условных операторах
```
if (field.getEntities().stream().filter(e -> e.getClass().equals(entityType)).count() > 3) {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(field)) {...}

private boolean isНазваниеКотороеВсеОбъясняет(Field field) {
  return field.getEntities().stream().filter(e -> e.getClass().equals(entityType)).count() > 3;
}
```

- Магическое число: 3.

**12. class InitEntityAction implements Action**

- Магические числа: 3, 5, 10.

- Нарушение DRY, дублирование кода. Вводи вспомогательные методы
```
for (int i = 0; i < 10; i++) {
  generateEntityOnMap(field, Tree.class);
}
for (int i = 0; i < 5; i++) {
  generateEntityOnMap(field, Grass.class);
}

//...
private void generateEntityOnMap(Field field, Class<?> classZ) {...}

//ПРАВИЛЬНО:
generateEntityOnMap(field, Tree.class, TREE_AMOUNT);
generateEntityOnMap(field, Grass.class, GRASS_AMOUNT);

//...
private void generateEntityOnMap(Field field, Class<?> clazz, int amount) {...}
```

**13. class FieldRender**

- Нарушение SRP. Рендерер не должен определять, в какой ОС он будет запускаться и подстраиваться под эту операционную систему.
Эти зависимости класс должен принимать в конструктор. Определять ОС и конфигурировать зависимости исходя из этого в данном случае нужно в `main()`
```
public class FieldRender {
  private static final ProcessBuilder processBuilder;

  static {
    String nameOS = System.getProperty("os.name").toLowerCase();
    if (nameOS.contains("linux") || nameOS.contains("mac")) {
      processBuilder = new ProcessBuilder("/bin/bash", "-c", "clear").inheritIO();
    } else {
      processBuilder = new ProcessBuilder("cmd", "/c", "cls").inheritIO();
    }
  }
  public FieldRender(Field field) {...}
  //...
}

//ПРАВИЛЬНО:
public class FieldRender {
  private final ProcessBuilder processBuilder;

  public FieldRender(Field field, ProcessBuilder processBuilder) {...}
  //...
}
```

**14. class Simulation**

+ 👍 Принимает достаточное количество зависимостей в конструктор
```
public Simulation(Field field, FieldRender fieldRender, EntityFactory entityFactory, Random random)
```

**15. class Runner**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

В целом, неплохо.

n.82(184)  
#ревью #симуляция 