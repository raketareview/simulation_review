https://github.com/Rinvel/simulationv2  
[Люба Белик]

Есть над чем поработать.

## ХОРОШО

1. Координаты существ не хранятся в самих существах(мне так больше нравится)
2. Можно делать карту произвольных размеров

## ЗАМЕЧАНИЯ

**1. Нейминг**

-UPPER_SNAKE только для констант, а это не константа
```
private final int WIDTH;
```

-Название класса должно быть существительным
```
class CreatureMove

//ПРАВИЛЬНО:
class CreatureПередвигатель
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*

**2. Нарушение DRY, магические буквы, числа, слова**  
Вводи константы. А если они уже есть- пользуйся 
```
public static final int NEXT_TURN = 1;
public static final int INFINITE_SIMULATION = 2;
public static final int EXIT = 3;

//...
System.out.println("1 - следующий ход");
System.out.println("2 - бесконечная симуляция");
System.out.println("3 - выход");

//ПРАВИЛЬНО:
public static final int NEXT_TURN = 1;
public static final int INFINITE_SIMULATION = 2;
public static final int EXIT = 3;

//...
System.out.println(NEXT_TURN + " - следующий ход");
System.out.println(INFINITE_SIMULATION + " - бесконечная симуляция");
System.out.println(EXIT + " - выход");
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"*

**3. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется**  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково
```
if (isDead()) {
  return new FieldChange(cell, null);
} else {
   return new CreatureMove(field, this, cell, targetMap.get(this.getClass())).findMove();
}

//ПРАВИЛЬНО:
if (isDead()) {
  return new FieldChange(cell, null);
} 
return new CreatureMove(field, this, cell, targetMap.get(this.getClass())).findMove();
```

**4. Используй форматированный вывод**   
Если нужно печатать текст с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматированный вывод- тогда сразу будет виден весь шаблон
```
System.out.println("Создание симуляции с размерами: " + width + " x " + height);

//ПРАВИЛЬНО:
System.out.printf("Создание симуляции с размерами: %d x %d \n", width, height);
```

**5. Не передавай тернарники во входящие аргументы метода**  
Это читается просто ужасно
```
System.out.print("|" + (entity != null ? entity : "⬛"));

//ПРАВИЛЬНО:
String sprite = entity != null ? entity : "⬛";
System.out.print("|" + sprite);
```

**6. record Cell(int x, int y), координата**

+Координата в виде рекорда это прекрасно. Плохо то, что на простом рекорде все не закончилось.

-toString выводит ошибочную информацию про координату. Какие бы ни были причины инкрементировать оси при распечатке, так делать неправильно
```
public String toString() {
  return "[x=" + (x + 1) +
      ", y=" + (y + 1) + "]";
}
```

Потому что toString() обманывает
```
Cell cell = new Cell(9, 9);
System.out.println("Координаты по версии toString(): " + cell.toString());
System.out.printf("Координаты на самом деле: x=%d, y=%d", cell.x(), cell.y());

//РЕЗУЛЬТАТ:
Координаты по версии toString(): [x=10, y=10]
Координаты на самом деле: x=9, y=9
```

-Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: x,y или row,column.
Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия, тем самым нарушая SRP: высчитывает дистанцию для каждой координаты из списка
```
public int getHeuristicDistance(List<Cell> targetCells) {...}
```
Без этого метода класс Координаты прекрасно обойдется- наличие или отсутствие этого метода никак не влияет на способность класса хранить и выдавать информацию для идентификации точки в пространстве(x, y).
Но без этого метода не может работать класс PathToTarget, значит именно там он и должен находиться.

-В данной реализации опорочена сама идея рекорда- навесив дополнительные методы, рекорд по сути превратился в обычный класс. 
Делай классы классами, а рекорды рекордами.
Единственно нормальный record для координаты выглядел бы так:
```
public record Coordinate(int x, int y) {
  //тело- пустое
  //в крайнем случае тут можно было бы toString() переопределить, и все.
}
```

-Зачем у рекорда переопределять хешкод, совершено непонятно. 
Рекорд умеет сам считать хешкод, тем более алгоритм твоего переопределенного хешкода соответствует дефолному.
Вот мой рекорд
```
public static void main(String[] args) {
  List<Coordinate> coordinates = List.of(
      new Coordinate(0, 0),
      new Coordinate(0, 1),
      new Coordinate(1, 0),
      new Coordinate(9, 10)
  );
  for(Coordinate c : coordinates) {
    System.out.printf("%s, hashcode: %d \n", c.toString(),  c.hashCode());
  }
}

//РЕЗУЛЬТАТ:
Coordinate[x=0, y=0], hashcode: 0
Coordinate[x=0, y=1], hashcode: 1
Coordinate[x=1, y=0], hashcode: 31
Coordinate[x=9, y=10], hashcode: 289
```

А вот твой рекорд(помним про неправильный toString())
```
public static void main(String[] args) {
  List<Cell> cells = List.of(
      new Cell(0, 0),
      new Cell(0, 1),
      new Cell(1, 0),
      new Cell(9, 10)
  );
  for (Cell c : cells) {
    System.out.printf("%s, hashcode: %d \n", c.toString(),  c.hashCode());
  }
}

//РЕЗУЛЬТАТ:
[x=1, y=1], hashcode: 0
[x=1, y=2], hashcode: 1
[x=2, y=1], hashcode: 31
[x=10, y=11], hashcode: 289
```

**7. class Field**

-Никогда не возвращай null
```
private final Map<Cell, Entity> entities = new HashMap<>();

public Entity getEntity(Cell cell) {
  return entities.get(cell); //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

-При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void removeEntity(Cell cell) {
  entities.remove(cell);
}
```

-Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
карта.addEntity(new Cell(0, 0), new Заяц());
карта.moveEntity(new Coordinates(0, 0), new Coordinates(99, 99));

/* class Field */
public void moveEntity(Cell from, Cell to) {
  if (!isEmpty(to)) {
    throw new IllegalArgumentException("Cell is not empty");
  }
  Entity entity = entities.remove(from);
  if (entity == null) {
    throw new IllegalArgumentException("Cell is empty");
  }
  entities.put(to, entity);
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Правильный ответ- карта вообще не должна иметь в себе метод телепортации, это не ее ответственность.

-Нарушение SRP. Карта не должна решать, можно ли вставлять существо в ячейку, если там уже находится другое существо, или нельзя
```
public void addEntity(Cell cell, Entity entity) {
  if (isEmpty(cell)) {
    entities.put(cell, entity);
  } else {
    throw new IllegalArgumentException("Cell is not empty");
  }
}
```
Это ответственность логики работы приложения в целом, а не логики работы карты. 
Например, логика работы приложения может измениться и волки будут сразу съедать зайцев, сразу становясь на ячейку с зайцем, по аналогии с шахматами.
Тогда придется менять логику работы addEntity() в Карте, а значит это в классе Карта дополнительная причина для изменения.

**8. abstract class Entity**

-Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением. 
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами.
Спрайты всех существ должны храниться в классе, который распечатывает карту
```
private final String symbolToPrint;
```

-toString() должен быть стандартным(как делает IDE по Alt+Ins) и использоваться только для отладки. 
toString() должен содержать значение всех значимых полей класса.
toString() не должен использоваться для представления, за исключением предельно простых классов, вроде PhoneNumber.
В данном случае все наследники Entity будут иметь переопределенный toString(), который невозможно использовать для отладки. 
Например, для Овечки toString() будет только выдавать картинку с овечкой, и не будет сообщать других значимых характеристик экземпляра: здоровье, скорость и т.д.
```
@Override
public String toString() {
  return symbolToPrint;
}
```
*Блох, "Java эффективное программирование" гл.3.3*

**9. abstract class Creature extends Entity**

-Нарушение OCP, класс знает своих потомков. При добавлении новых классов потомков, будет вызываться каскадное изменение в классе-предке 
```
public static Map<Class<? extends Creature>, Class<? extends Entity>> targetMap = new HashMap<>();

static {
  targetMap.put(Predator.class, Herbivore.class);
  targetMap.put(Herbivore.class, Grass.class);
}
```
Например, при добавлении в проект классов Черепаха и Жаба, придется добавлять пару черепаха-жаба в targetMap их предка, класс Creature.
Предок не должен знать своих потомков.

Алтернатива: ввести в Creature поле Class food и пусть потомки креатуры в своем конструкторе инжектят в супер предка класс своей еды.

**10. class PathToTarget**

+Поиск должен искать путь в Карте от точки старта до точки, соответствующей условиям. В данном случае- до точки, содержащей съедобное существо
```
public static List<Cell> findPath(Field field, Cell start, Cell targetCell) {...}

//ПРАВИЛЬНО:
public static List<Cell> findPath(Field field, Cell start, Class target) {...}
```
Если алгоритм ищет путь через список координат всех съедобных существ в карте(алгоритм А*?), то он все равно должен принимать класс цели, на основе этого класса готовить список координат и скармливать этот список самому себе.

**11. class EntityFactory**

-Избыточно
```
entities.put(Rock.class, EntityFactory::createRock);

public static Rock createRock() {
  return new Rock();
}

//ЛУЧШЕ:
entities.put(Rock.class, Rock::new);
```

**12. class SpawnAction extends Action**

-Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники. 
Это своего рода вариация паттерна Command.
Т.е. акции должны быть родственны и одинаково использоваться через полиморфизм. 
Поэтому все наследники Action должны иметь один публичный метод- perform().

Здесь SpawnAction имеет и другие публичные методы, а значит этот класс по своей сути не Action, а что-то другое. 

Вот так вкратце должны работать классы семейства Action
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

**13. class Launcher** 

-Магические цифры: 1, 99, 100.

-(А)Нарушение DRY, опосредованное дублирование кода. (Б)Использование try-catch для ветвления бизнес логики
```
while (true) {
  try {
    System.out.print("Введите ширину: ");
    width = Integer.parseInt(scanner.nextLine());
    System.out.print("Введите высоту: ");
    height = Integer.parseInt(scanner.nextLine());

    if (isValidNumber(width) && isValidNumber(height)) {
       break;
    } else {
      System.out.println("Оба числа должны быть в диапазоне от 1 до 99. Повторите ввод.");
    }
  } catch (NumberFormatException e) {
     System.out.println("Ошибка ввода! Введите целые числа.");
  }
}

new Simulation(width, height, scanner);
```

Вводи вспомогательные методы:
```
int width = inputFieldSide(scanner, "Введите ширину: ", MIN, MAX);
int height = inputFieldSide(scanner, "Введите высоту: ", MIN, MAX);

new Simulation(width, height, scanner);

//...
  private static int inputFieldSide(Scanner scanner, String title, int min, int max) {
    while (true) {
      System.out.println(title);
      String s = scanner.nextLine();
      if (!isInteger(s)) {
        System.out.println("Ошибка ввода! Введите целое число.");
        continue; 
      }   

      int value = Integer.parseInt(s);
      if (value >= min && value <= max) {
         return value;
      }
      System.out.printf("Число должно быть в диапазоне от %d до %d. Повторите ввод. \n", min, max); 
    }
  }

  private static boolean isInteger(String s) {
    try {
      Integer.parseInt(s);
      return true;
    } catch (NumberFormatException e) {
      return false;
    }
  }
```

-Та же проблема, что была в хангмане- запуск методов класса прямо из конструктора класса
```
new Simulation(width, height, scanner);

//ПРАВИЛЬНО:
Simulation simulation = new Simulation(width, height, scanner);
simulation.start();
```

## ВЫВОД

Для лучшего понимания декомпозиции посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID- по одному ролику на каждую букву.
Перестать запускать методы класса прямо из конструктора класса.

#ревью #симуляция