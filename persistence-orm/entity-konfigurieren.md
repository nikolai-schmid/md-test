# Entity konfigurieren

Hier werden die Konfigurationsmöglichkeiten von Entities beschrieben.

## Id

Ist nichts weiter definiert, wird angenommen, dass die Eigenschaft mit Namen \"id\" die Id (Primary Key) repräsentiert und das der Wert der zugehörigen Spalte von der Datenbank automatisch generiert wird (in MySQL `AUTO_INCREMENT`). Du kannst auch eine andere Eigenschaft als Id definieren, indem du sie mit `n2n\\persistence\\orm\\annotation\\AnnoId` annotierst.

```php
class User extends EntityAdapter {
    private static function _annos(AnnoInit $as) {
        $as->p('name', new AnnoId());
    }
    
    private $name;
    private $password;
    // getters / setters
}
```

Über die Parameter von `AnnoId` kannst du dem ORM ausserdem mitteilen, ob der Wert von der Datenbank oder der Applikation generiert wird und ob eine Sequenz zum Generieren dieses Wertes benötigt wird (z. B. in Oracle).

## Tabellen- und Spaltennamen

Wird nichts weiter konfiguriert, werden standardmässig Klassen- und Eigenschaftsnamen \"hyphenated\" und diese als Tabellen- und Spaltennamen verwendet. Für eine Entity mit Klassennamen `BlogArticle` wird also eine Tabelle `blog_article` und für eine Eigenschaft `$orderIndex` eine Spalte `order_index` gesucht. Du kannst die Tabellen- und Spaltennamen aber auch direkt in der Entity über die Annotationen `n2n\\persistence\\orm\\annotation\\AnnoTable` und `n2n\\persistence\\orm\\annotation\\AnnoColumn` festlegen.

```php
class BlogUser extends EntityAdapter {
    private static function _annos(AnnoInit $ai) {
        $ai->c(new AnnoTable('bloguser'));
        $ai->p('firstName', new AnnoColumn('firstname'));
    }
    
    private $id;
    private $firstName;
    private $lastName;
    // getters / setters
}
```

Für die Entity `User` im oben stehenden Beispiel wäre also folgende Tabelle passend:

```sql
CREATE TABLE `bloguser` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `firstname` VARCHAR(50) NULL DEFAULT NULL,
  `last_name` VARCHAR(50) NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) COLLATE='utf8_general_ci' ENGINE=InnoDB;
```

### Naming Strategy

Du kannst den Algorithmus zum Generieren von Tabellen- und Spaltennamen überschreiben, indem du eine eigene \"Naming Strategy\" definierst. Hierzu musst du eine Klasse erstellen, die `n2n\\persistence\\orm\\model\
amingStrategy` implementiert und diese in der app.ini unter der Gruppe `[orm]` registrieren.

```ini
[orm]
naming_strategy = \"atusch\\config\\CustomNamingStrategy\"
```

Du kannst auch eine der Implementationen verwenden, die n2n zur Verfügung stellt:

- `n2n\\persistence\\orm\\model\\HyphenatedNamingStrategy` (Standard)
- `n2n\\persistence\\orm\\model\\IdenticalNamingStrategy`
- `n2n\\persistence\\orm\\model\\LowercasedNamingStrategy`

Über die Annotation `n2n\\persistence\\orm\\annotation\\AnnoNamingStrategy` kannst du die \"Naming Strategy\" auch für einzelne Entities überschreiben.

```php
class User extends EntityAdapter {
    private static function _anno(AnnoInit $ai) {
        $ai->c(new AnnoNamingStrategy(new LowercasedNamingStrategy()));
    }
    // class body
}
```

**Achtung:** Definierst du eine eigene \"Naming Strategy\" in der `app.ini`, beeinflusst dies die Entities aller installierten Module. Dies kann dazu führen, dass Module, die zum Beispiel über den Store installiert wurden, ihre Tabellen und Spalten nicht mehr finden. Es ist also zu empfehlen die \"Naming Strategy\" nur für eigene Entities zu überschreiben (über `AnnoNamingStrategy`).

## Mapped Supperclass

Du kannst Eigenschaften in sogenannte \"Mapped Superclasses\" auslagern, die von Entities erweitert werden können. Die Eigenschaften werden dabei in der Tabelle der Entity gespeichert, als wären sie in ihr selbst definiert. \"Mapped Superclasses\" müssen mit `n2n\\persistence\\orm\\annotation\\AnnoMappedSuperclass` annotiert werden. Es ist **nicht** nötig sie im `app.ini` zu registrieren.

```php
class Teaser extends ObjectAdapter {
    private static function _annos(AnnoInit $ai) {
        $ai->c(new AnnoMappedSuperclass());
    }
    
    private $title;
    private $description;
    
    public function getTitle() {
        return $this->title;
    }
    
    public function setTitle($title) {
        $this->title = $title;
    }
    
    public function getDescription() {
        return $this->description;
    }
    
    public function setDescription($description) {
        $this->description = $description;
    }
}
```

`Teaser` kann nun von beliebigen Entities erweitert werden.

```php
class News extends Teaser {
    private $id;
    private $text;
    
    public function getId() {
        return $this->id;
    }
    
    public function setId($id) {
        $this->id = $id;
    }
    
    public function getText() {
        return $this->text;
    }
    
    public function setText($text) {
        $this->text = $text;
    }
}
```

Zur Entity `News` passt folgende Tabelle:

```sql
CREATE TABLE IF NOT EXISTS `news` (
  `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `title` VARCHAR(50),
  `description` VARCHAR(255),
  `text` TEXT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## Embedded

Du kannst Eigenschaften auch in eine Extraklasse auslagern und deren Objekte per Eigenschaft referenzieren. Solche Eigenschaften müssen mit `n2n\\persistence\\orm\\annotation\\AnnoEmbedded` annotiert werden.

Gehen wir davon aus, dass wir eine Entity `Order` benötigen, um Bestellungen eines Shops zu erfassen. Die Bestellung soll eine Rechnungs- und wenn gewünscht eine separate Lieferadresse enthalten. Dies bedeutet, dass wir Eigenschaften wie `$forename` und `$surname` jeweils für die Rechnungs- sowie Lieferadresse benötigen. Anstatt diese Adress-Eigenschaften doppelt zu deklarieren, ist es einfacher diese in eine Extraklasse `Address` auszulagern und in der Entity `Order` zwei \"embedded\" Eigenschaften vom Typ `Address` zu deklarieren. `Address` könnte später auch von anderen Entities genutzt werden, um Adress-Eigenschaften auszulagern. Extraklassen müssen **nicht** im `app.ini` registriert werden.

```php
class Order extends ObjectAdapter {
    private static function _annos(AnnoInit $ai) {
        $ai->p('address', new AnnoEmbedded(Address::getClass()));
        $ai->p('deliveryAddress', new AnnoEmbedded(Address::getClass(), 'delivery_'));
    }
    
    private $id;
    private $productName;
    private $address;
    private $deliveryAddress;
    
    public function getProductName() {
        return $this->productName;
    }
    
    public function setProductName($productName) {
        $this->productName = $productName;
    }
    
    /**
    * @return Address
    */
    public function getAddress() {
        return $this->address;
    }
    
    public function setAddress(Address $address) {
        $this->address = $address;
    }
    
    public function hasDeliveryAddress() {
        return $this->deliveryAddress !== null;
    }
    
    /**
    * @return Address
    */
    public function getDeliveryAddress() {
        return $this->deliveryAddress;
    }
    
    public function setDeliveryAddress(Address $deliveryAddress = null) {
        $this->deliveryAddress = $deliveryAddress;
    }
}
```

Damit die Eigenschaften der Extraklasse `Address` für die Rechnungs- und Lieferadresse nicht die selben Spalten in der Tabelle \"order\" verwenden, bietet `AnnoEmbedded` über den zweiten und dritter Parameter die Möglichkeit einen Prefix sowie Suffix für die Spalten der jeweiligen Extraklasse zu definieren.

**Hinweis:** Der angegebene Prefix und Suffix wird bei [Beziehungs](http://support.web.n2n.ch/de/n2n/documentation/beziehungen)-Eigenschaften auch für die Namen der Join-Spalten und Zwischentabellen verwendet.

`AnnoEmbedded` erwartet als ersten Parameter die `ReflectionClass` der jeweiligen Extraklasse, darum ist es auch hier sinnvoll, dass die Extraklasse `n2n\\reflection\\ObjectAdapter` erweitert.

```php
class Address extends ObjectAdapter {
    private $forename;
    private $surname;
    // more properties
    
    public function __construct($forename = null, $surname = null) {
        $this->forename = $forename;
        $this->surname = $surname;
    }
    
    public function getForename() {
        return $this->forename;
    }
    
    public function setForename($forename) {
        $this->forename = $forename;
    }
    
    public function getSurname() {
        return $this->surname;
    }
    
    public function setSurname($surname) {
        $this->surname = $surname;
    }
}
```

Zur Entity `Order` passt folgende Tabelle:

```sql
CREATE TABLE `order` (
  `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `product_name` VARCHAR(50) NOT NULL,
  `forename` VARCHAR(50) NOT NULL,
  `surname` VARCHAR(50) NOT NULL,
  `delivery_forename` VARCHAR(50) DEFAULT NULL,
  `delivery_surname` VARCHAR(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Ist `Order::$deliveryAddress` beim Speichern `null`, so werden in die beiden Felder \"delivery_forename\" und \"delivery_surname\" *NULL* geschrieben. Genauso wird `Order::$deliveryAddress` beim Lesen `null` enthalten, wenn die beiden Felder \"delivery_forename\" und \"delivery_surname\" *NULL* sind.

In folgendem Beispiel erstellen wir eine Bestellung mit und eine ohne Lieferadresse:

```php
public function doOrm(EntityManager $em) {
    $this->beginTransaction();
    
    $order = new Order();
    $order->setProductName('Broccoli');
    $order->setAddress(new Address('Peter', 'Awesome'));
    $order->setDeliveryAddress(new Address('Amanda', 'Pretty'));
    $em->persist($order);
    
    $order = new Order();
    $order->setProductName('ZSC Playoff Tickets');
    $order->setAddress(new Address('Peter', 'Awesome'));
    $em->persist($order);
    
    $this->commit();
}
```

Das oben stehende Beispiel schreibt folgende zwei Zeilen in die Tabelle \"order\":

| id | product_name        | forename | surname | delivery_forename | delivery_surname |
|----|---------------------|----------|---------|-------------------|------------------|
| 1  | Broccoli            | Peter    | Awesome | Amanda            | Pretty           |
| 2  | ZSC Playoff Tickets | Peter    | Awesome | NULL              | NULL             |

### Abfragen

Folgende Beispiele zeigen, wie du \"embedded\" Eigenschaften in Abfragen verwenden kannst.

#### Beispiel 1

**Criteria API**

```php
$criteria = $em->createSimpleCriteria(Order::getClass(), ['address.forename' => 'Peter']);
```

**NQL**

```php
$criteria = $em->createNqlCriteria('SELECT o FROM \"Order\" o WHERE o.address.forename = :forename', ['forename' => 'Peter']);
```

#### Beispiel 2

**Criteria API**

```php
$address = new Address('Peter', 'Awesome');
$criteria = $em->createSimpleCriteria(Order::getClass(), ['address' => $address]);
```

**NQL**

```php
$address = new Address('Peter', 'Awesome');
$criteria = $em->createNqlCriteria('SELECT o FROM \"Order\" o WHERE o.address = :address', ['address' => $address]);
```

#### Beispiel 3

**Criteria API**

```php
$criteria = $em->createSimpleCriteria(Order::getClass(), ['deliveryAddress' => null]);
```

**NQL**

```php
$criteria = $em->createNqlCriteria('SELECT o FROM \"Order\" o WHERE o.deliveryAddress = :deliveryAddress', ['deliveryAddress' => null]);
```

## Attribute Overrides

Über die Annotation `n2n\\persistence\\orm\\annotation\\AnnoAttributeOverrides` kannst du für alle Eigenschaften einer Entity die assoziierten Spaltennamen überschreiben. `AnnoAttributeOverrides` ist somit eine Alternative zu `n2n\\persistence\\orm\\annotation\\AnnoColumn`. `AnnoAttributeOverrides` erlaubt es dir auch die assoziierten Spaltennamen der Eigenschaften einer [Mapped Supperclass](http://support.web.n2n.ch/de/n2n/documentation/entity-konfigurieren#mapped-supperclass) zu überschreiben.

```php
class News extends Teaser {
    private static function _annos(AnnoInit $ai) {
        $ai->c(new AnnoAttributeOverrides([
            'id' => 'news_id',
            'title' => 'news_title',
            'description' => 'news_description',
            'text' => 'news_text'
        ]));
    }
    
    private $id;
    private $text;
    // getters / setters
}
```

In diesem Beispiel haben wir das Beispiel aus dem Abschnitt [Mapped Supperclass](http://support.web.n2n.ch/de/n2n/documentation/entity-konfigurieren#mapped-supperclass) angepasst und über `AnnoAttributeOverrides` alle Spaltennamen der Entity `News` und ihrer [Mapped Supperclass](http://support.web.n2n.ch/de/n2n/documentation/entity-konfigurieren#mapped-supperclass) `Teaser` überschrieben.

Die Tabelle von `News` sieht neu folgendermassen aus:

```sql
CREATE TABLE `news` (
  `news_id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `news_title` VARCHAR(50),
  `news_description` VARCHAR(255),
  `news_text` TEXT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Du kannst `AnnoAttributeOverrides` auch auf Extraklassen anwenden, indem du die entsprechende [Embedded](http://support.web.n2n.ch/de/n2n/documentation/entity-konfigurieren#embedded)-Eigenschaften annotierst.

```php
class Order extends ObjectAdapter {
    private static function _annos(AnnoInit $ai) {
        $ai->p('address', new AnnoEmbedded(Address::getClass()), new AnnoAttributeOverrides([
            'forename' => 'bill_forename',
            'surname' => 'bill_surname'
        ]));
        $ai->p('deliveryAddress', new AnnoEmbedded(Address::getClass(), 'delivery_'));
    }
    
    private $id;
    private $productName;
    private $address;
    private $deliveryAddress;
    // getter / setter
}
```

Hier haben wir das Beispiel aus dem Abschnitt [Embedded](http://support.web.n2n.ch/de/n2n/documentation/entity-konfigurieren#embedded) angepasst und über `AnnoAttributeOverrides` die Spaltennamen der Extraklasse `Address` überschrieben.

Die Tabelle von `Order` sieht neu folgendermassen aus:

```sql
CREATE TABLE `order` (
  `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `product_name` VARCHAR(50) NOT NULL,
  `bill_forename` VARCHAR(50) NOT NULL,
  `bill_surname` VARCHAR(50) NOT NULL,
  `delivery_forename` VARCHAR(50) DEFAULT NULL,
  `delivery_surname` VARCHAR(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
