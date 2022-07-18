## Наследование

**Наследование** — это *механизм*, посредством которого один или несколько классов можно получить из **некоторого базового класса**. Класс, унаследованный от другого класса, называется его **подклассом**.Эта связь обычно
описывается с помощью терминов **родительский и дочерний**. В частности,
дочерний класс происходит от родительского и наследует его характеристики, состоящие из свойств и методов. Обычно функциональные возможности родительского класса, который называется также **суперклассом**, дополняются в дочернем классе новыми функциональными возможностями
Поэтому говорят, что **дочерний класс расширяет родительский**. Дальше подбробнее расскажу зачем этот механизм нужен и какие проблемы он решает.

Давайте рассмотрим класс `ShopProduct`:
```php
class ShopProduct {
    public $numPages;
    public $playLength;
    public $title;
    public $producerMainName;
    public $producerFirstName;
    public $price;
    public function __construct(
        string $title, 
        string $firstName, 
        string $mainName, 
        float $price,
        int $numPages = 0,
        int $playLength = 0)
    {
        $this->title = $title;
        $this->producerFirstName = $firstName;
        $this->producerMainName = $mainName;
        $this->price = $price;
        $this->numPages = $numPages;
        $this->playLength = $playLength;
    }
    public function getNumberOfPages(): int
    {
    return $this->numPages;
    }
    public function getPlayLength(): int
    {
    return $this->playLength;
    }
    public function getProducer(): string
    {
    return $this->producerFirstName . $this->producerMainName;
    }
}
```

Как видно по примеру, у нас есть **один класс**, который отвечает за отображение информации о товаре. Скажем, мы хотим показать 
книгу и песню. Так как у книги и песни есть **свои особенности**, для песни это её длительность, а в книге количество страниц.
В итоге объект, экземпляр которого создается с помощью
приведенного выше класса, **будет всегда содержать избыточные методы, и придется передавать лишний аргумент в конструктор**. А что, если у каждого товара есть больше особенностей? А что если видов
товара будет гораздо больше? Со временем, наш класс станет крайне **громадным и неудобным для поддержки**.
Мы можем создавать отдельные классы для каждого товара, но тогда у нас получится много дубликата, нарушается приницип **DRY(Don't Repeat Yourself)**. Для того, чтобы сохранить общие методы и свойства, но для каждого вида товара добавить свои, не касающиеся других, нам надо использовать **наследование**.

Давайте перепишем класс `ShopProduct`, и унаследуем два класса для книг и песен.
```php
class ShopProduct { // Родительский класс
    public $numPages;
    public $playLength;
    public $title;
    public $producerMainName;
    public $producerFirstName;
    public $price;
    public function __construct(string $title, string $firstName, string $mainName, float $price,
        int $numPages = 0,
        int $playLength = 0)
    {
        $this->title = $title;
        $this->producerFirstName = $firstName;
        $this->producerMainName = $mainName;
        $this->price = $price;
        $this->numPages = $numPages;
        $this->playLength = $playLength;
    }
    public function getProducer(): string
    {
        return $this->producerFirstName . $this->producerMainName;
    }
    public function getSummaryLine(): string
    {
        $base = $this->title . $this->producerMainName;
        $base .= $this->producerFirstName;
        return $base;
    }
}

class SongProduct extends ShopProduct // Дочерний класс для песен
{
    public function getPlayLength(): int
    {
        return $this->playLength;
    }
    public function getSummaryLine(): string
    {
        $base = $this->title . $this->producerMainName . $this->producerFirstName;
        $base .= "Время звучания -" . $this->playLength;
        return $base;
    }
}

class BookProduct extends ShopProduct // Дочерний класс для книг
{
    public function getNumberOfPages(): int
    {
        return $this->numPages;
    }

    public function getSummaryLine(): string
    {
        $base = $this->title . $this->producerMainName . $this->producerFirstName;
        $base .= ":" . $this->numPages . "стр.";
        return $base;
    }
}
```

Чтобы создать производный (дочерний) класс, достаточно указать в его
объявлении ключевое слово `extends`. В данном примере мы создали два
новых класса — `BookProduct` и `SongProduct`. Оба они расширяют класс
`ShopProduct`. Поскольку в производных классах конструкторы не определяются,
при получении экземпляров этих классов будет **автоматически вызываться конструктор родительского класса**. Дочерние классы наследуют
доступ ко всем методам типа `public` и `protected` из родительского
класса, но не к методам и свойствам типа `private`.Это означает, что метод `getProducer()` можно вызвать для созданного экземпляра класса
`SongProduct`, как показано ниже, несмотря на то, что данный метод определен в классе `ShopProduct`

```php
$product2 = new SongProduct(
    "Классическая музыка. Лучшее",
    "Антонио",
    "Вивальди",
    10.99,
    0 ,
    60.33);
print "Композитор: {$product2->getProducer()}\п";
```

Таким образом, оба дочерних класса **наследуют поведение общего родительского класса**. И мы можем обращаться с объектом типа `SongProduct`
так, как будто это объект типа `ShopProduct`.

### Конструкторы и наследование
Чтобы вызвать нужный метод из родительского класса, придется обратиться непосредственно к этому классу через дескриптор. Для этой цели в
РНР предусмотрено ключевое слово `parent`.
Чтобы обратиться к методу в контексте класса, а не объекта, следует использовать символы `::` , а не `->`. Синтаксическая конструкция `parent::__construct()` означает следующее: “Вызвать метод `__construct()` из родительского класса”.
Если мы решим переопределить конструктор в дочернем классе, вот как он будет выглядеть:
```php
class SongProduct extends ShopProduct{
    //...
    public function __construct(string $title, string $firstName, string $mainName, float $price, int $numPages)
    {
        parent::__construct($title,$firstName,$mainName,$price); // Вызываем конструктор родителя
        $this->numPages = $numPages;
    }
    //...
}
```