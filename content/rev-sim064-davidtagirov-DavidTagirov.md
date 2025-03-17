https://github.com/DavidTagirov/simulation-cats-and-mice  
[DavidTagirov]

Симуляция на Swing.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. В редакторе карты ввожу x = 20, y = 4, а создается карта с размерами по x = 4, y = 20.
2. Размер окна карты автоматически не подстраивается под размеры карты.
3. Сопроводительная информация печатается не в виндовое окно, а в консоль
```
Кот поел
Кот поел
Мышь была съедена
```
![UI](https://github.com/raketareview/simulation_review/blob/master/content/resources/res-sim064/img0.jpg)

## ХОРОШО

+ 🚀 Есть редактор карты
+ 🚀 Windows UI, Swing 
+ 👍 Спрайты существ хранятся не в самих существах

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название должно как можно лучше объяснять суть
```
World.x
World.y

//ЛУЧШЕ:
World.width
World.height
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

**3. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется - в этом случае неважно, будет else или нет, так как программа будет работать одинаково
```
if (creature instanceof Cat) {
  return !(entity instanceof Barrier || entity instanceof Cheese || entity instanceof Cat);
} else if (creature instanceof Mouse) {...}

//ПРАВИЛЬНО:
if (creature instanceof Cat) {
  return !(entity instanceof Barrier || entity instanceof Cheese || entity instanceof Cat);
} 
if (creature instanceof Mouse) {...}
```

**4. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками. Исключение- метод equals()
```
if (targets.isEmpty()) return null;

//ПРАВИЛЬНО:
if (targets.isEmpty()) {
  return null;
}  
```

**5. Текст в exception** всегда должен быть на английском языке  
```
throw new RuntimeException("Не удалось загрузить изображения");
```
Исключение это не просто телеграмма, которая летит сквозь слои. 
У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты. 
А значит, сообщение должно быть на английском. 
Интерпритация исключения и перевод его на локальный язык должно происходить там, где это соответствует архитектуре программы или не происходить вовсе, если исключение не планируется перехватывать

**6. Длинные нечитаемые конструкции**, вводи поясняющие переменные
```
Coordinates nearestMouse = pathFinder.getNearestTarget(pathFinder.findTargets("Mouse"), this);
```

**7. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if (world.getEntities().get(coords).getClass().getSimpleName().equals("Barrier")) {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет("Barrier", /*args or empty*/)) {...}

private boolean isНазваниеКотороеВсеОбъясняет(String someText, /*args or empty*/) {
  return world.getEntities().get(coords).getClass().getSimpleName().equals(someText);
}
```

**8. record Coordinates(int x, int y)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**9. class World**

- Класс должен принимать размеры карты в конструктор. Здесь класс вообще не имеет конструктора.  
Сейчас Карта создается так
```
World world = new World();
world.generate(x, y);
```
А должна создаваться так
```
World world = new World(width, height);
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void setEntities(Coordinates coordinates, Entity entity) {
  entity.setCoordinates(coordinates);
  entities.put(coordinates, entity);
}
```

- Два метода, которые не имеют отношения к функции карты: первоначальное заполнение карты существами и обновление существ в карте.

- Миллион магических цифр.

- Дублирование кода в `generate(int x, int y)` и `update()`
```
setEntities(coordinates, Mouse.builder()
  .speed(1)
  .health(100)
  .build());
//etc  
```

- Все поля имеют неявные геттеры, заданные аннотацией ломбока `@Getter`, здесь это приводит к нарушению инкапсуляции.  
Класс предоставляет клиентскому коду подробности внутреннего устройства и доступа к хешмапе `entities`
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта();
//заселить карту существами
карта.getEntities().clear(); //геноцид минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

❌ Совершенно удивительный класс в реверсном относительно положительного смысле. Содерджит всего 3 метода. Из них к ответстсвенности карты имеет отношение только `setEntities(...)`.

**10. class PathFinder**, поиск пути

- Как пользоваться классом, совершенно неясно- в нем несколько публичных методов.  
Публичным должен быть только один метод, который будет искать путь от точки старта до точки, соответствующей заданным условиям.
В данном случае- до точки, в которой находится существо нужного класса. Типа такого
```
public List<Coordinates> getPatch(World world, Coordinates start, Class target) {...}
```

- Нарушение SRP. Класс должен принимать условия поиска в конструктор, а не определять их самостоятельно. 
Здесь же класс определяет условия путем анализа креатур
```
if (creature instanceof Cat) {
  return !(entity instanceof Barrier || entity instanceof Cheese || entity instanceof Cat);
} else if (creature instanceof Mouse) {
  return !(entity instanceof Barrier || entity instanceof Cat || entity instanceof Mouse);
} else return false;
```

- Смещения сразу сделай координатами и вынеси в константы- тогда их не придется пересоздавать каждый раз
```
public class PathFinder {
  public List<Coordinates> findPath(Coordinates start, Coordinates goal, Creature creature) {
    //...
    int[][] directions = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    for (int[] dir : directions) {
      Coordinates next = new Coordinates(current.x() + dir[0], current.y() + dir[1]);
      //...
   }
  }
}

//ПРАВИЛЬНО:
public class PathFinder {

  private final ststic Coordinates[] DIRECTIONS = {
    new Coordinates(1, 0),
    new Coordinates(-1, 0),
    //oths
  };

  public List<Coordinates> findPath(Coordinates start, Coordinates goal, Creature creature) {
    //...
    for (Coordinates direction : DIRECTIONS) {
      Coordinates next = new Coordinates(current.x() + direction.x(), current.y() + direction.y());
      //...
   }
  }
}
```

**11. Никогда не возвращай null**
```
public Coordinates getNearestTarget(List<Coordinates> targets, Creature creature) {
  if (targets.isEmpty()) return null;
  //...
}  
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

**12. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня Creature.

**13. abstract class Creature extends**

- При прочих равных используй примитивные типы, а не классы обертки
```
protected Integer speed;
protected Integer health;

//ПРАВИЛЬНО:
protected int speed;
protected int health;
```
Для использования обертки должна быть причина и выгода, которую дает применение этой обертки.
Тут выгоды никакой нет.

**14. class Mouse/Cat extends Creature**

- Нарушение SRP, зависимость модели от представления. 
Модель(а это модель) не должна ничего печатать в консоль или куда бы то ни было еще напрямую
```
System.out.println("Сыр не найден — мышь не двигается");
System.out.println("Кот умер от голода");
```
Если модель во время выполнения каких-либо действий должна проинформировать об этом систему, то можно использовать паттерн Callback.  
CallBack луноход: https://t.me/zhukovsd_it_chat/53243/139594  
CallBack разрушение: https://t.me/zhukovsd_it_chat/53243/184749  

- Магические цифры: 10, 100.

**15. class Action**

- Вместо семейства классов `Action`, как указано в ТЗ, здесь только один класс, который инициирует движение креатур.  
Сделай Action'ы по ТЗ. В том числе, отдельный экшен для первоначального заселения существ- сейчас этот функционал находится в классе Карта.

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники. 
Это своего рода вариация паттерна Command.
Т.е. акции должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
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
Более подробно про экшены я писал тут: https://t.me/zhukovsd_it_chat/53243/147969 п."Архитектура"

**16. class Simulation**

- Редактор карты нужно вынести из этого класса, создавать карту- не ответственность Симуляции.

- Не возвращай логически связанные друг с другом данные в виде массива параметров. Возвращай объекты, даже если они предельно простые
```
int[] worldParams = getWorldParameters();
int x = worldParams[0];
int y = worldParams[1];
int worldSpeed = worldParams[2];

World world = new World();
world.generate(x, y);

//ПРАВИЛЬНО:
WorldParameters parameters = getWorldParameters();
World world = new World(parameters.width, parameters.height);
```
- Нарушение SRP. Метод не должен завершать работу программы через exit()
```
private int[] getWorldParameters() {
  //...  
  System.exit(0);
}  
```
Каждый метод и класс имеют право завершать только свою работу через return. 
Потому что методы и классы не должны знать логику работу более высоких слоев программы, у которых могут быть свои мысли на тему того, когда и почему нужно завершать работу программы.

**17. class Renderer extends JPanel**

+ 👍 Вью виндового интерфейса на Swing.
+ 👍 Только отрисовка, не содержит бизнес логику, за исключением незначительного нюанса.

- Нарушение SRP, рендерер должен только распечатывать данные, но не формировать данные, в данном случае рендерер ведет подсчет хотов
```
private int moveCount = 0;
```

**18. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.
- Сдесь нужно запускать редактор карты, получать ее и передавать в конструктор Симуляции.

## ВЫВОД

Сделать Action'ы по ТЗ. Разобраться с единой ответственностью классов в проекте.  
Реализация понравилась. Но даже не тем, что виндовый интерфейс.  
А тем, что виндовый интерфейс реализован в рендерере, который только делает отрисовку.

n.64(144)  
#ревью #симуляция #swing