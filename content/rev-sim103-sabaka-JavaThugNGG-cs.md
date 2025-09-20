https://github.com/JavaThugNGG/simulation-game-csharp  
[Сабака]

Проект на C#, ревью на правах общей эрудиции.

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Почему-то не видно картинок  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim099/img0.png)

## ХОРОШО

+ 👍 Реализовано пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Это не менеджер 
```
class SimulationManager

//ТУТ ПРАВИЛЬНО:
class Simulation
```
Менеджером этот класс был бы, если бы в проекте существовал класс "Simulation". Тогда отдельный класс "SimulationManager", который содержал бы некие методы для работы с Simulation.
Здесь такой пары классов нет, поэтому постфикс "Manager" не имеет никакого смысла.

Корректность использования приписок типа "Manager" и "Helper" много обсуждается в сети, [например тут](https://gist.github.com/hurricane-voronin/9ceccace0fd530bbf17c83b059c86eb7).
*.NET Общие соглашения об именовании*  
*Правила и соглашения об именовании идентификаторов C#*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
Console.WriteLine("1 - Запуск симуляции");
Console.WriteLine("2 - Пауза");
Console.WriteLine("3 - Продолжить");
case 1:  //...
case 2:  //...
case 3:  //...
if (choice is >= 1 and <= 3)
Console.WriteLine("Неверный выбор. Введите число от 1 до 3.");

//ПРАВИЛЬНО:
private const int Start = 1;
private const int Pause = 2;
private const int Continue = 3;

Console.WriteLine($"{Start} - Запуск симуляции");
Console.WriteLine($"{Pause} - Пауза");
Console.WriteLine($"{Continue} - Продолжить");
case Start:  //...
case Pause:  //...
case Continue:  //...
if (choice is >= Start and <= Continue)
Console.WriteLine($"Неверный выбор. Введите число от {Start} до {Continue}.");
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"* 

**3. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется.  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково и код без `else` будет выглядеть читабельней
```
if (_map.TryGetValue(neighbor, out Entity? entity))
{
  //...  
  return false;
}
else
{
  return true;
}

//ПРАВИЛЬНО:
if (_map.TryGetValue(neighbor, out Entity? entity))
{
  //...  
  return false;
}
return true;
```

**4. record Coordinates(int Row, int Column)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.
```
internal record Coordinates(int Row, int Column) : IComparable<Coordinates>
{
  public override string ToString() => $"{Row} {Column}";
  public int CompareTo(Coordinates? other) => (Row, Column).CompareTo((other!.Row, other.Column));
}
```

**5. Класс, которого нет- Карта**

В ТЗ указан класс `Map`, который является воплощением сущности Карта(Доска) для хранения существ.  
В этой реализации такого класса не сделано. 
Вместо этого класса, для хранения существ используется `IDictionary<Coordinates, Entity>`, который перебрасывается между методами других классов.
Например, часть методов для работы с этим `IDictionary` находится в классе `MapController`.

Проблема, однако, состояит в том, что от этого сущность Карта никуда не делась из проекта.  
Просто она теперь разбросана в виде запчастей по всему проекту:
- Некоторые методы для работы с этой сущностью находятся в классе `MapController`
- Размеры в виде фиксированных в константах размеров хранятся в классе `SimulationManager` etc.

**6. class MapController**, некоторые методы для работы с записями координата-существо из `IDictionary<Coordinates, Entity>`

- Побочный эффект. 

Судя по названию метод должен только заполнить карту существами из списка
```
private void FillMap(IList<Entity> generatedEntities)
{
  foreach (Entity entity in generatedEntities)
  {
    Coordinates entityCoordinates = entity.Coordinates;
    _map[entityCoordinates] = entity;
    if (entity is Creature creature)
    {
     creature.Moved += OnCreatureMoved;
    }
  }
}
```
Но кроме этого метод устанавливает событие(event) креатурам из этого списка.
Метод нужно разделить на несколько, каждый из которых будет выполнять только одну задачу.  
*Мартин "Чистый код", гл.3, "Избавьтесь от побочных эффектов"*  
*Фаулер "Рефакторинг", гл.6, "Извлечение метода"*  

- Если бы это был просто класс-воплощение сущности "Карта", я бы сказал, что этот метод нарушает SRP
```
private void OnCreatureMoved(object sender, Coordinates oldCoordinates, Coordinates newCoordinates)
{
  _map[newCoordinates] = (Entity)sender;
  _map.Remove(oldCoordinates);
}
```
А так даже и не знаю- настолько этот класс загадочен.

**7. class Menu**

- Нарушение DRY. Две одинаковых строки кода
```
Console.WriteLine("Неверный выбор. Введите цифру от 1 до 3!");
```
Вынеси этот код во вспомогательный метод, например `void writeFailCommandMessage()` и в этих двух строках вызывай данный метод.

- Нет смысла делать две одинаковых проверки на диапазон числа 1-3 в методах `int GetValidChoice()` и `void RunMenu()`.
Оставь эту проверку только в `RunMenu()`.

**8. class PathFinder**

- Нарушение SRP. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar. 
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```
internal List<Coordinates> FindPathToVictim(Creature creature)
{
  if (creature is Predator)
  {
    return FindPathToGoal(typeof(Herbivore));
  }
  if (creature is Herbivore)
  {
    return FindPathToGoal(typeof(Grass));
  }
  return new List<Coordinates>();
}
```
Метод поиска должен принимать не креатуру, в интересах которой будет вестись поиск.
А набор данных, которые необходимы для процесса поиска.  
Например, так:
```
internal List<Coordinates> Find(IDictionary<Coordinates, Entity> map, Coordinates start, Type target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(entity.GetType() == target)
}
```

- Нарушение SRP, чужая ответственность, зависимость модели от представления.

Модель(а бизнес-логика по поиску пути это модель) не должна ничего печатать в консоль
```
Console.WriteLine($"При определении существа в методе {nameof(IsValidMove)} произошла ошибка!");
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах нужно будет менять код в этом классе, чтобы вывод осуществлялся по правилам этой среды, то есть, это лишняя причина для изменения класса.
В других средах(напр. Андроид) эту модель нельзя будет использовать- она там просто не скомпилируется. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

Давай разбиремся по сути кейса.
Итак если в метод `IsValidMove(Coordinates neighbor, Type goalType)` передан некорректный `Type goalType` то это является ошибкой алгоритма.
Потому что в  

**9. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. 
Поэтому entities должны хранить координату только начиная с уровня `Creature`.

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```
internal string Figure { get; init; }
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.

- ToString() должен быть стандартным, то есть перечислять значения всех полей, и использоваться только для отладки.
Здесь все семейство `Entity` обречено через `ToString()` выводить значение только одного поля- Figure
```
public override string ToString() => Figure;
```
ToString() не должен использоваться для представления, за исключением предельно простых классов, вроде PhoneNumber.
У наследников энтити, таких как `Creature` и его потомки, много значимых полей, которые стоило бы выводить в ToString().

**10. abstract class Creature : Entity**

- Вот эти все хендлеры только усложнают алгоритм и перемешивают ответственности между Креатурой и Картой.
Просто напиши нормальную последовательность команд для совершения хода
```
internal delegate void MoveHandler(object sender, Coordinates oldCoordinates, Coordinates newCoordinates);
internal event MoveHandler Moved;
//...
internal void MakeMove(Coordinates newCoordinates)
{
  Coordinates oldCoordinates = Coordinates;
  Coordinates = newCoordinates;
  OnMove(oldCoordinates, newCoordinates);
}

//ПРАВИЛЬНО:
{
  Coordinates oldCoordinates = Coordinates;
  Coordinates = newCoordinates;
  //удалить себя из карты по координате oldCoordinates
  //вставить себя в карту по координате newCoordinates
}
```

- Метод совершения хода не должен принимать координату, чтобы потом на нее перейти
```
void MakeMove(Coordinates newCoordinates)
```

Алгоритм совершения хода может находиться либо в Креатуре в методе `MakeMove(...)`, либо находиться в стороннем классе, например, Мувере.

Если алгоритм движения находится в стороннем Мувере, тогда Креатура интерпритируется как сущность, лишенная своей воли- ее перемещает внешняя сила.
Типа, фигура в шахматах, которую двигает игрок.  
Это вполне норм, но тогда `Creature` вообще не должен иметь в себе метод `MakeMove(...)`.

Если же Креатура интерпритируется как сущность с собственной волей, тогда принимать решение о том, куда и как ходить, должна принимать только она.
И вся эта логика должна находиться в методе `MakeMove(...)`. Сторонние классы могут только вызывать этот метод, то есть давать пинка креатуре, чтобы она побежала.

И в этом случае метод `MakeMove(...)` должен не принимать в себя конечную координату хода. 
А сам в себе вызывать метод поиска и найти путь к еде, а потом пройти по этому пути.

**11. Пакет Commands**

Реализация паттерна "Команда": интерфейс `ICommand` и его реализации.
Вроде, все ок 👍 

**12. Пакет Actions**

- Реализация экшенов в этом пакете не соответствует ТЗ.

Должен быть интерфейс `IAction` и его реализации. В интерфейсе и реализациях должно быть только по одному публичному методу.
Все эти экшены должны использоваться через полиморфизм. А это невозможно, если между ними не будет родства по линии общего наследования или реализации общего интерфейса.

Вот например два "экшена", один из которых двигает существ, а другой заселяет существ на карту. Они никоим образом не родственники
```
internal class MoveCreaturesAction {...}
internal abstract class PlantSpawnAction {...}
```

- Нарушение DRY. Куча классов, которые делают одно и то же- заселяют на карту существ определенного типа
```
abstract class CreatureSpawnAction
class GrassSpawnAction : PlantSpawnAction
class HerbivoreSpawnAction : CreatureSpawnAction
class PredatorSpawnAction : CreatureSpawnAction
class TreeSpawnAction : PlantSpawnAction
```
Внутри этих классов- фактически одинаковый код.

Нужно сделать универсальный спавнер, который будет спавнить в карту любых существ.

**13. class Launcher**

Откуда этот класс знает про цифры 1-3? 
```
Console.WriteLine("Симуляция закончена! Введите цифру от 1 до 3 для выхода");
```
Эти цифры знает и использует только `class Menu`.

**14. class WorldPrinter**

- В циклах рекомендуют использовать однобуквенные индексы. Но иногда это объективно хуже
```
for (int i = 0; i < SimulationManager.WorldRows; i++)
{
  for (int j = 0; j < SimulationManager.WorldColumns; j++)
  {
    Coordinates coordinate = new Coordinates(i, j);
  }
}

//ЛУЧШЕ:
for (int row = 0; row < SimulationManager.WorldRows; row++)
{
  for (int column = 0; column < SimulationManager.WorldColumns; column++)
  {
    Coordinates coordinate = new Coordinates(row, column);
  }
}
```

- Спрайты существ этот класс должен хранить в себе, а не брать из Entity.

**15. class SimulationManager**

- Нет никаких оснований делать карту фиксированного размера
```
internal static readonly int WorldRows = 10;
internal static readonly int WorldColumns = 20;
```
Фиксированные размеры могут быть у шахматной доски(8x8) или игры крестики-нолики(3x3).  
Здесь размеры должны приниматься в конструктор, чтобы можно было создавать карту произвольных размеров.

- Так как в реализации нарушена идея `Action` из ТЗ, то все т.н. "экшены" используются не через полиморфизм, а персонально
```
private readonly IList<CreatureSpawnAction> _creatureInitActions = new List<CreatureSpawnAction>();
private readonly IList<PlantSpawnAction> _plantInitActions = new List<PlantSpawnAction>();
private readonly IList<MoveCreaturesAction> _turnActions = new List<MoveCreaturesAction>();

foreach (PlantSpawnAction action in _plantInitActions)
{
  action.Perform(_map, _generatedEntities);
}

foreach (CreatureSpawnAction action in _creatureInitActions)
{
  action.Perform(_map, _generatedEntities);
}
```
Все экшены должны иметь общий интерфейс `ISimAction`(название `IAction` не подойдет- будет путаница со станд. C# классом `Action`) и использоваться одинаково через полиморфизм.  
Примерно так:
```
private readonly IList<ISimAction> _initActions = new List<ISimAction> {
  new UnoAction(),
};

private readonly IList<ISimAction> _turnActions = new List<ISimAction> {
  new DosAction(),
  new TresAction(),
  new MoveAction(),
};

//...
performActions(_initActions);  //ВЫПОЛНИТЬ ДЕЙСТВИЯ СТАРТОВЫХ ЭКШЕНОВ
performActions(_turnActions);  //ВЫПОЛНИТЬ ДЕЙСТВИЯ ЭКШЕНОВ НА КАЖДОМ ХОДЕ

internal void performActions(IList<ISimAction> actions) { 
 foreach(ISimAction a in actions) {
   a.perform( /*карта*/ );
 }
}
```

## ВЫВОД

Не видно понимания полиморфизма- экшены лишены родства.

С декомпозицией тоже не все ок- отсутствует класс Карты.    
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы.  
Посмотреть ролики Немчинского про SOLID.

Эта симуляция на шарпе концептуально не отличается от [версии на java](https://github.com/JavaThugNGG/simulation-game).  
Видимо, либо я тебя не ревьюрил, либо ты не провел работу над ошибками.

n.103(232)  
#ревью #симуляция #csharp 