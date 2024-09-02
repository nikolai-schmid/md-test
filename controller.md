# Controller

Ein Controller definiert den Einsprungspunkt deiner Applikation. Hierzu interpretiert er die URL des aktuellen HTTP-Requests.

## Controller definieren

Ein Controller wird als Klasse definiert, die üblicherweise n2n\\http\\controller\\ControllerAdapter erweitert. Wir erstellen im folgenden Beispiel einen Controller im Package atusch\\controller.

```php
<?php
namespace atusch\\controller;

use n2n\\http\\controller\\ControllerAdapter;

class ExampleController extends ControllerAdapter {   
    public function index() {
        echo 'hallo atusch';
    }
}
```

Um den Controller direkt aufrufen zu können, müssen wir in der app.ini unter der Gruppe [routing] einen Kontextpfad definieren.

```ini
[routing]

controllers[] = \"/example > atusch\\\\controller\\\\ExampleController\"
```

Wenn du mit deinem Browser nun http://[url-to-n2n-public]/example aufrufst, wird die index-Methode ausgeführt und \"hallo atusch\" ausgegeben.

Alternativ kannst du den Kontextpfad auch wie folgt definieren:

```ini
[routing]

controllers[/example] = \"atusch\\\\controller\\\\ExampleController\"
```

### Action-Methoden

Typische Action-Methoden in Controllern sind Do-Methoden. Diese werden mit dem Prefix \"do\" gekennzeichnet und ausgeführt, wenn der Pfad des aktuellen Requests mit der Methoden-Signatur übereinstimmt. Definieren wir im ExampleController vom oberen Beispiel z.B. eine Methode doDetail(), ist diese unter http://[url-to-n2n-public]/example/detail aufrufbar.

```php
<?php
namespace atusch\\controller;

use n2n\\http\\controller\\ControllerAdapter;

class ExampleController extends ControllerAdapter {   
    public function index() {
        echo 'hallo atusch';
    }
     
    public function doDetail() {
        echo 'hallo atusch detail';
    }
}
```

Der Name der Methode (abzüglich des Präfixes \"do\") muss mit dem ersten Teil des Pfades nach dem Kontextpfad übereinstimmen. Dieser Teil muss klein geschrieben sein, auch wenn der Methoden-Name Großbuchstaben enthält. Die Methode doAdvancedDetail() würde im oberen Beispiel also nur über http://[url-to-n2n-public]/example/advanceddetail aufgerufen werden können.

### Pfad-Parameter

Du kannst Pfadteile der Methode als Parameter übergeben lassen:

```php
public function doDetail($param1, $param2, $param3 = null) {
    echo 'param1: ' . $param1 . ' param2: ' . $param2 . 'param3: ' . $param3;
}
```

Die Methode ist nun zum Beispiel über http://[url-to-n2n-public]/example/detail/foo/bar/baz oder, da $param3 optional ist, auch über http://[url-to-n2n-public]/example/detail/foo/bar aufrufbar.

Die Pfad-Parameter kannst du dir auch als n2n\\http\\controller\\ParamPath-Objekte übergeben lassen:

```php
public function doDetail(ParamPath $param1, ParamPath $param2, ParamPath $param3 = null) {
    echo 'param1: ' . $param1 . ' param2: ' . $param2 . 'param3: ' . $param3;
}
```

### Query-Parameter

Auf die gleiche Weise kannst du dir Query-Parameter als n2n\\http\\controller\\ParamQuery-Objekte übergeben lassen:

```php
public function doDetail(ParamQuery $param1, ParamQuery $param2 = null) {
    echo 'param1: ' . $param1 . ' param2: ' . $param2;
}
```

Diese Methode ist über http://[url-to-n2n-public]/example/detail?param1=foo&param2=bar oder http://[url-to-n2n-public]/example/detail?param1=foo aufrufbar.

### Pfaderweiterung

Du kannst außerdem eine Pfaderweiterung erzwingen, indem du die Methode um einen Parameter $extension erweiterst:

```php
public function doDetail($param1, $extension) {
    echo 'param1: ' . $param1 . ' extension: ' . $extension;
}
```

Diese Methode ist z.B. über http://[url-to-n2n-public]/example/detail/foo.txt aufrufbar.

### Pfadteile zusammenfassen

Definierst du einen Pfad-Parameter als array, kannst du Pfadteile zusammenfassen:

```php
public function doDetail($param1, array $params) {
    echo 'param1: ' . $param1 . ' params: ' . implode(', ', $params);
}
```

$params erlaubt in diesem Beispiel einen oder mehrere Pfadteile. Diese Methode ist z.B. über http://[url-to-n2n-public]/example/detail/foo/bar/baz/... aufrufbar. Natürlich kannst du auch einen array-Parameter optional machen:

```php
public function doDetail($param1, array $params = array()) {
    echo 'param1: ' . $param1 . ' params: ' . implode(', ', $params);
}
```

doDetail() wäre jetzt auch über http://[url-to-n2n-public]/example/detail/foo aufrufbar.

Alle oben gezeigten Beispiele funktionieren auch mit der index()-Methode, nur dass dort beim Aufruf der Pfadteil \"detail\" natürlich weggelassen werden muss.

## Annotationen

Über Annotationen lassen sich Controller-Aufrufe noch genauer steuern. Zum Beispiel kannst du über n2n\\http\\annotation\\AnnoPath für Controller-Methoden ein Pfad-Muster annotieren:

```php
<?php
namespace atusch\\controller;

use n2n\\http\\controller\\ControllerAdapter;
use n2n\\reflection\\annotation\\AnnoInit;
use n2n\\http\\annotation\\AnnoPath;

class ExampleController extends ControllerAdapter {
    private static function _annos(AnnoInit $ai) {
        $ai->m('detail', new AnnoPath('param1:*/const/param2:*'));
    }
     
    public function detail($param1, $param2) {
        echo 'param1: ' . $param1 . ' params2: ' . $param2;
    }
}
```

detail() ist in diesem Beispiel über http://[url-to-n2n-public]/example/foo/const/bar aufrufbar. Enthält das Muster einen \":\", wird vor dem \":\" der Name des zugehörigen Methoden-Parameters (optional) und nach dem \":\" ein regulärer Ausdruck (mit Delimiter #) oder \"*\" für einen beliebigen Pfadteil erwartet. Bei :* kann der \":\" auch weggelassen werden.

Auch Methoden, für die ein Muster annotiert wurde, können mit Query-Parametern kombiniert werden.

Konstante Pfadteile im Muster müssen nach RFC 1738 kodiert sein. Z.B. foo/b%26r/baz

## Modifikatoren

Mit Modifikatoren kannst du im Muster bestimmen, wie viele Male ein Pfadteil nacheinander vorkommen darf. Folgende Modifikatoren stehen zur Auswahl:

? optional; 0 oder 1
* 0 oder mehrere
+ 1 oder mehrere

Der Modifikator muss im Muster am Schluss eines Pfadteils, aber vor einem allfälligen \":\" platziert werden.

### Beispiele:

Pattern | Beispiele passender Pfade
--- | ---
test/foo? | test, test/foo
test/foo* | test, test/foo, test/foo/foo, test/foo/foo/foo, ...
test/foo+ | test/foo, test/foo/foo, test/foo/foo/foo, ...
test/*? | test, test/foo
test/** | test, test/foo, test/foo/bar, test/foo/bar/baz, ...
test/*+ | test/foo, test/foo/bar, test/foo/bar/baz, ...
test/param?:#^[a-z]+$# | test, test/foo
test/param*:#^[a-z]+$# | test, test/foo, test/foo/bar, test/foo/bar/baz, ...
test/param+:#^[a-z]+$# | test/foo, test/foo/bar, test/foo/bar/baz, ...

Folgendes Beispiel demonstriert die Anwendung von Modifikatoren:

```php
class ExampleController extends ControllerAdapter {   
    private static function _annos(AnnoInit $ai) {
        $ai->m('detail', new AnnoPath('const?/params+:#^[0-9]+$#'));
    }
     
    public function detail(array $params) {
        echo 'params: ' . implode(', ', $params);
    }
}
```

detail() ist z.B. aufrufbar über http://[url-to-n2n-public]/example/const/1 oder http://[url-to-n2n-public]/example/1/2/33.

## Pfaderweiterung

Du kannst über die Annotation n2n\\http\\annotation\\AnnoExt die Pfaderweiterungen einschränken.

```php
class ExampleController extends ControllerAdapter {    
    private static function _annos(AnnoInit $ai) {
        $ai->m('doDetail', new AnnoExt('txt'));
        $ai->m('number', new AnnoPath('param:#^[0-9]$#'), new AnnoExt('txt', 'html'));
    }
     
    public function doDetail($param, $extension) {
        echo 'param: ' . $param . ' extension: ' . $extension;
    }
     
    public function number($param, $extension) {
        echo 'param: ' . $param . ' extension: ' . $extension;
    }
}
```

doDetail() ist z.B. über http://[url-to-n2n-public]/example/detail/foo.txt und number() über http://[url-to-n2n-public]/example/1.txt oder http://[url-to-n2n-public]/example/1.html aufrufbar.

Pfaderweiterungen können auch für den ganzen Controller definiert werden.

```php
class ExampleController extends ControllerAdapter {   
    private static function _annos(AnnoInit $ai) {
        $ai->c(new AnnoExt('txt', 'html'));
    }
     
    // methods
}
```

Pfaderweiterungen, die für den ganzen Controller definiert wurden, gelten für alle Methoden, die nicht ihrerseits mit AnnoExt annotiert wurden.

## Vordefinierte Methoden

prepare() Wird als erstes aufgerufen, sobald der Controller ausgeführt wird. Diese Methode wird auch ausgeführt, wenn keine Controller-Methode zum Request-Pfad passt.
notFound() Wird ausgeführt, wenn keine Controller-Methode zum Request-Pfad passt oder in einer solchen Methode eine HttpStatusException mit Status 404 geworfen wurde.

## Dependency Injections

Alle Action-Methoden sind \"magisch\" und erlauben Dependency Injections. Zudem ist im Controller die _init()-Methode implementierbar. Die _init()-Methode wird aufgerufen, sobald der Controller instanziert wird. Da dies nicht bedeuten muss, dass der Controller auch ausgeführt wird, ist es sinnvoll die prepare()-Methode als Alternative zu nutzen.

## Kontextpfad

Für das Muster der Kontextpfade, die in der app.ini registriert werden, gelten dieselben Regeln wie für die Muster, die über n2n\\http\\controller\\AnnosController::PATH annotiert werden. Im Muster des Kontextpfades kann aber zusätzlich der Platzhalter \{locale\} verwendet werden.

```ini
[routing]

controllers[] = \"/{locale}/example > atusch\\controller\\ExampleController\"

locales[] = \"de_CH\"
locales[] = \"fr\"
locales[] = \"en_US\"
```

Beim Aufruf werden für diesen Platzhalter alle Locales akzeptiert, die unter der Eigenschaft \"locales\" angegeben wurden. Ist zum Beispiel das Locale \"de_CH\" angegeben, wird für diesen Pfadteil \"de-ch\" akzeptiert. Gefällt dir diese Darstellung nicht, kannst du in der app.ini in der Gruppe [web] unter der Eigenschaft \"locale_aliases\" einen Alias definieren:

```ini
[web]

locale_aliases[de_CH] = \"de\"
```

In diesem Beispiel ist der ExampleController über http://[url-to-n2n-public]/de/example, http://[url-to-n2n-public]/fr/example oder http://[url-to-n2n-public]/en-us/example aufrufbar. Das entsprechende Locale wird direkt dem Request zugewiesen und kann bei Bedarf im Controller über $this->getRequest()->getLocale() ausgelesen werden.

### Kontextparameter

Parameter, die im Muster eines Kontextpfades definiert sind, können Controller-Methoden übergeben werden.

```ini
controllers[] = \"/cParam1:#^(foo|bar)$#/cParam2:* > atusch\\controller\\ExampleController\"
```

Folgendes Beispiel soll zeigen, wie du im Controller auf \"cParam1\" und \"cParam2\" zugreifen kannst:

```php
class ExampleController extends ControllerAdapter {
    private static function _annos(AnnoInit $ai) {
        $ai->m('number', new AnnoPath('param:#^[0-9]+$#'));
    }
     
    private $cParam1;
     
    public function prepare($cParam1) {
        $this->cParam1 = $cParam1;
    }
     
    public function index($cParam2, $param) {
        echo 'index (cParam1: ' . $this->cParam1 . '; cParam2: ' . $cParam2 . '; param: ' . $param . ')';
    }
     
    public function doDetail($cParam2, $param) {
        echo 'doDetail (cParam1: ' . $this->cParam1 . '; cParam2: ' . $cParam2 . '; param: ' . $param . ')';
    }
     
    public function number($cParam2, $param) {
        echo 'number (cParam1: ' . $this->cParam1 . '; cParam2: ' . $cParam2 . '; param: ' . $param . ')';
    }
}
```

### Aufrufbeispiele:

http://localhost/php-hnm-zend/default/n2n-7/public/foo/baz/qux

Ausgabe: index (cParam1: foo; cParam2: baz; param: qux)

http://localhost/php-hnm-zend/default/n2n-7/public/foo/baz/detail/qux

Ausgabe: doDetail (cParam1: foo; cParam2: baz; param: qux)

http://localhost/php-hnm-zend/default/n2n-7/public/foo/baz/24

Ausgabe: number (cParam1: foo; cParam2: baz; param: 24)
