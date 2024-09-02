# Coding Standards

Bei der Entwicklung von N2N haben wir uns an die folgenden Code-Richtlinien gehalten. Es bleibt dem Entwickler aber natürlich selbst überlassen, ob er sich daran halten will.

## Dateiformatierungen

### PHP

Bei Dateien, die nur PHP-Code enthalten, ist der Abschlusstag (`?>`) wegzulassen. So werden unerwünschte Leerzeichen beim finalen Output verhindert.

PHP-Blöcke sollen immer mit `<?php` eröffnet werden. `<?` sowie `<?=` sind nicht erlaubt.

### CSS

Bei CSS-Dateien ist auf der ersten Zeile die Zeichencodierung anzugeben:

```css
@charset \"UTF-8\";
```

## Kontrollstrukturen

Bei Kontrollstrukturen soll die eröffnende Klammer vom Schlüsselwort durch einen Abstand getrennt werden. Nach einer geöffneten und vor einer geschlossenen geschweiften Klammer wird die Zeile umgebrochen. Zwischen geschlossenen und geöffneten Klammern sowie vor und nach Operatoren wird ein Abstand eingefügt.

```php
// if example
if ($status == $otherStatus || $atusch == true) {
    $text = 'Hello';
} elseif ($condition3 && $condition4) {
    $sum += 2;
} else {
    $sum = $otherSum + $num;
}

// foreach example
foreach ($dsAdvertisements as $key => $rowAdvertisement) {
    echo $rowAdvertisement['teig'];
}

// switch example
switch ($sum) {
    case 1:
        // code here
        break;
    case 2:
        // code here
        break;
    default:
        // code here
        break;
}
```

### Längere Statements

Längere Statements sind auf mehrere Zeilen aufzuteilen. Beachte die doppelte Einrückung. $conditionX steht für eine längere Bedingung.

```php
if ($condition1
        && $condition2
        && $condition3) {
    // code here
}

if ($condition1
        && ($condition2
                || $condition3)
        && $condition4
        && $condition5) {
    // code here
}

if (($condition1
                || $condition2)
        && $condition3
        && $condition4) {
    // code here
}
```

## Variablen

Für die Benennung von Variablen wird ausschließlich die Lowercase-Variante von CamelCase verwendet (z.B. camelCaseVariable). Eine Ausnahme bilden allerdings Konstanten.

### Konstanten

Bei Konstanten werden alle Buchstaben groß geschrieben und die einzelnen Worte durch Unterstriche getrennt.

```php
define('CFG_APP_NAME', 'advertisementstest');

class Test {
    const LIST_SIZE = 20;
}
```

## Strings

Ein String sollte grundsätzlich mit Hochkommas (single quotes) abgegrenzt werden. Entstehen dadurch aber Mehraufwände oder werden Zeilenumbrüche benötigt, darf auch auf die Doppeltenanführungszeichen (double quotes) zurückgegriffen werden. Ein String wird mit dem '.'-Operator zusammen gesetzt. Vor und hinter '.' ist ein Abstand einzufügen.

```php
// Sample Strings
$sampleString = 'This is a sample string.';
$sampleString2 = 'sampleString contains: ' . $sampleString;
$sampleString3 = \"sampleString2 contains: '\" . $sampleString2 . \"'\";
$sampleString4 = 'This string is followed by a new line' . \"\
\";
```

Um die Leserlichkeit zu erhöhen, sollten längere Strings auf mehrere Zeilen aufgeteilt werden. Je nach Verwendung, müssen Strings beim Zeilenumbruch nicht unbedingt aufgesplittet werden. Beinhaltet der String eine spezielle Kodierung z.B. SQL oder HTML, ist der Variablenname mit einem entsprechenden Prefix zu versehen.

```php
$exampleString = 'n2n ist ein Content Management System welches trotz ' .
        'umfangreichem Funktionsumfang sehr einfach zu bedienen ist.';

$htmlExample = '
        <div class=\"message\">
            <strong>n2n</strong> ist ein <strong>Content Management
            System</strong> welches trotz umfangreichem Funktionsumfang
            sehr einfach zu bedienen ist.
        </div>';

$sqlWhere = ' `id` = 1';
$sql = 'SELECT * FROM `blog_post` WHERE' . $sqlWhere;
```

## SQL

Schlüsselwörter in SQL-Statements sind immer groß zu schreiben. Handelt es sich um ein größeres Statement, sind Zeilenumbrüche zu verwenden. Aus Performance- und Sicherheits-Gründen sollte bei Datenbank-Abfragen das Prepared-Statement dem normalen Statement vorgezogen werden. Sollen trotzdem PHP-Variablen direkt ins Statement eingefügt werden, muss jede Variable, unabhängig ihres Ursprungs, im Statement \"escaped\" werden.

```php
$type = 'sell';
$userId = 12;

$stmtAdvertisements = $dbh->query('
        SELECT advertisement_id, name, type
        FROM `advertisement` AS a
        INNER JOIN `user` AS u ON a.user_id = u.user_id
        WHERE type = ' . $dbh->quote($type) . '
          AND user_id = ' . $dbh->quote($userId));
```

## Arrays

```php
$arrTest = array(
        'foo'  => 'bar',
        'spam' => 'ham');

$arrTest['foo'] = 'atusch';
$arrTest[4] = 'atusch';
```

## Klassen und Methoden

Für Klassen und Methoden gelten die selben Regeln wie für Kontrollstrukturen. Für Klassennamen ist die Uppercase- und für Methodennamen die Lowercase-Variante von CamelCase zu verwenden. Weitere Regeln können dem fogenden Beispiel entnommen werden:

```php
<?php

class Test implements Testable {
    private $test;
    private $test2;
    
    public function __construct($test, $test2 = null) {
        $this->test = $test;
        $this->test2 = $test2;
    }
    
    public function getTest() {
        return $this->test;
    }
}
```

Wird ein Aufruf einer Methode oder Funktion (z.B. auf Grund von vielen Parametern) zu lang, ist der Aufruf folgendemassen auf mehrere Zeilen aufzuteilen. Beachte die doppelte Einrückung:

```php
$numLikes = OrmUtils::createCountCriteria($this->em, Rating::getClass(),
        array('suggestion' => $suggestion, 'like' => true))->fetchSingle();

return $this->em->createSimpleCriteria(Suggestion::getClass(), array('online' => true,
                'client' => $this->getCurrentClient()),
        array('numComments' => 'DESC'), $limit, $num)->fetchArray();
```

## Views

Views beginnen mit einem großen PHP-Block, in dem einfacher Code wie Parameter initialisieren, Template setzen, CSS laden usw. erlaubt ist. Nach diesem Block sollten nur einzelne Anweisungen, die in separate PHP-Code-Blöcke aufgeteilt sind, vorkommen. Lagere komplexere Logik in ein Model aus. Nutze für Kontrollstrukturen in Views den alternativen Syntax (z.B. if ($x):). Mit $view->assert() kannst du dich versichern, dass der View gültige Parameter übergeben wurden. Die Verwendung von instanceof in $view->assert() ermöglicht deinem Editor den Typ des Parameters zu erkennen und dir Autovervollständigung anzubieten.

Weiteres kannst du folgendem Beispiel entnehmen:

```php
<?php
use com\\example\\model\\FooModel;

$fooModel = $view->getParam('fooModel');
$view->assert($fooModel instanceof FooModel);

$view->useTemplate('view\\template.html');
$html->meta()->addCss('css/foo.css');
?>

<div>
    <h2>Bars overview</h2>
    <?php if (!$fooModel->hasBars()): ?>
        <p>No bars available.</p>
    <?php else: ?>
        <ul>
            <?php foreach ($fooModel->getBars() as $bar): ?>
                <li><?php $html->out($bar->getName()) ?></li>
            <?php endforeach ?>
        </ul>
    <?php endif ?>
</div>
```

## Annotationen

Die _annotations()-Methode muss in der Klasse an erster Stelle stehen ohne führenden Zeilenumbruch.

```php
class Post extends EntityAdapter {
    private static function _annos(AnnoInit $ai) {
        $ai->c(new AnnoTable('blog_post'));
    }

    // class body
}
```

## Closures

Die geschweifte Klammer soll auf der selben Ebene geöffnet und geschlossen werden. Beachte auch den Abstand nach function.

```php
$closure = function () {
    // code
};

$obj->onEvent(function () {
    // code
});

$obj->onOtherEvent($param1, $param2,
        function () {
            // code
        }, $param4);
```

