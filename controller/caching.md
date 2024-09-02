# Caching

Mit "Caching" erreichst du, dass generierte Seiten wiederverwendet und nicht bei jedem Request neu generiert werden. Damit lässt sich die Performance verbessern sowie den Ressourcen-Verbrauch verringern. Schnelle Ladezeiten wirken sich positiv auf die "User Experience" sowie SEO aus.

## Caching in Controllern anwenden

Nutze die Methoden `ControllerAdapter::assignResponseCacheControl()` und `ControllerAdapter::assignHttpCacheControl()`, um den Response und Http Cache für deine Controller-Methoden steueren. Lese die Artikel Response Cache und Http Cache, um mehr zu diesem Thema zu erfahren.

Du solltest möglichst darauf achten, dass für alle deine Seiten ein Caching in irgend einer Form aktiv ist. Am besten verwendest du Response und Http Cache zusammen.

```php
class ExampleController extends ControllerAdapter {
    public function doFoo() {
        $this->assignHttpCacheControl(new \\DateInterval('PT30M'));
        $this->assignResponseCacheControl(new \\DateInterval('P1D'));

        $this->forward('view\\bar.html');
    }
}
```

## Cache Control

Cache Control-Eigenschaften, die über `ControllerAdapter::assignResponseCacheControl()` oder `ControllerAdapter::assignHttpCacheControl()` dem Controller zugewiesen werden, funktionieren nur im Zusammenspiel mit `ControllerAdapter::forward..()`, `ControllerAdapter::redirect..()` oder `ControllerAdapter::send()`. Nutzt du keine dieser Methoden und möchtest trotzdem, dass der Response zwischengespeichert wird, musst du die Cache-Control-Eigenschaften direkt dem Response übergeben. Hierzu benötigst du die Typen `n2n\\http\\ResponseCacheControl` und `n2n\\http\\HttpCacheControl`.

```php
public function doFoo() {
    $this->getResponse()->setHttpCacheControl(
            new HttpCacheControl(new \\DateInterval('PT30M')));
    $this->getResponse()->setResponseCacheControl(
            new ResponseCacheControl(new \\DateInterval('PT30M')));

    $this->getResponse()->setHeader('Content-Type: text/plain; charset=utf-8');

    echo 'text without view';
}
```

Modifiziere den Response-Header ausschliesslich über `n2n\\http\\Response` und verzichte auf PHP-Funktionen wie `header()`. Diese könnten das Exception-Handling stören.

**KommentareFragen**

Du musst eingeloggt sein, damit du Beiträge erstellen kannst.