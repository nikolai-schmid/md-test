# Meta

## Cmd

```php
<?php
namespace atusch\\controller;

use n2n\\http\\controller\\ControllerAdapter;
use n2n\\util\\uri\\Path;

class ExampleController extends ControllerAdapter {

    public function index(Path $cmdPath, Path $cmdContextPath) {
        echo 'cmdPath: ' . $cmdPath . ' cmdContextPath: ' . $cmdContextPath;
    }

    public function doDetail(array $cmdPathParts, array $cmdContextPathParts) {
        echo 'cmdPath: ' . implode('/', $cmdPathParts) . ' cmdContextPath: ' . implode('/', $cmdContextPathParts);
    }
}
```

## Request

```
n2n\\http\\controller\\ControllerAdapter::getRequest()
```

php-doc von n2n\\http\\Request. Zwei mal schreiben macht eigentlich keinen Sinn. Mit Thomas besprechen, wie lösen.

## Response

```
n2n\\http\\controller\\ControllerAdapter::getResponse()
```

php-doc von n2n\\http\\Response. Zwei mal schreiben macht eigentlich keinen Sinn. Mit Thomas besprechen, wie lösen.

## Redirect

```
n2n\\http\\controller\\ControllerAdapter::redirect()
```

php-doc von n2n\\http\\controller\\ControllerAdapter. Zwei mal schreiben macht eigentlich keinen Sinn. Mit Thomas besprechen, wie lösen.
