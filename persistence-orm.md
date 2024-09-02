# Persistence / ORM

n2n Persistence bildet die Schnittstelle zur Datenbank und bietet eine umfangreiche ORM (object-relational mapping) API, welche die Spezifikation 2.1 der JPA (Java Persistence API) nahezu vollständig übernimmt. Ein ORM bietet dir eine objektorientierte Abstraktionsebene der Datenbank. Die Basis aller Datenbank-Verbindungen in n2n bildet PDO (PHP Data Objects).

## Persistence Unit

In der `app.ini` unter der Gruppe `[database]` kannst du Datenbank-Verbindungen (Persistence Units) für deine Applikation registrieren. Das Schema für die Eigenschaften einer Verbindung ist `{persistence_unit_name}.{property}`.

\[database : development\]

default.dsn_uri = "mysql:server=localhost;dbname=testdb"
default.user = "root"
default.password = ""
default.transaction_isolation_level = "SERIALIZABLE"
default.dialect = "n2n\\\\persistence\\\\meta\\\\dialect\\\\mysql\\\\MysqlDialect"

\[database : live\]

default.dsn_uri = "mysql:server=localhost;dbname=livedb"
default.user = "liveuser"
default.password = "livepw"
default.transaction_isolation_level = "SERIALIZABLE"
default.dialect = "n2n\\\\persistence\\\\meta\\\\dialect\\\\mysql\\\\MysqlDialect"

In diesem Beispiel registrieren wir eine Persistence Unit mit Namen "default" für den Development- und Live-Modus. Die Eigenschaft `{persistence_unit_name}.dsn_uri` entspricht dem Parameter `$dsn` von [`PDO::__construct()`](http://php.net/manual/de/pdo.construct.php). Über `{persistence_unit_name}.transaction_isolation_level` wählst du das [Isolations-Level](http://de.wikipedia.org/wiki/Isolation_%28Datenbank%29#Serializable), das für Transaktionen dieser Persistence Unit gelten soll. Als Werte sind [`"READ UNCOMMITTED"`](http://de.wikipedia.org/wiki/Isolation_%28Datenbank%29#Read_Uncommitted), [`"READ COMMITTED"`](http://de.wikipedia.org/wiki/Isolation_%28Datenbank%29#Read_Committed), [`"REPEATABLE READ"`](http://de.wikipedia.org/wiki/Isolation_%28Datenbank%29#Repeatable_Read) und [`"SERIALIZABLE"`](http://de.wikipedia.org/wiki/Isolation_%28Datenbank%29#Serializable) möglich. Der Standard-Wert ist [`"SERIALIZABLE"`.](http://de.wikipedia.org/wiki/Isolation_%28Datenbank%29#Serializable)

Mit `{persistence_unit_name}.dialect` definierst du den, zur Datenbank passenden, `Dialect`. Der `Dialect` bildet eine Abstraktions-Ebene, über welche Meta-Informationen zur Datenbank einheitlich abgefragt werden können. Diese werden sowohl von der ORM API wie auch der Meta API benötigt.

n2n bietet standardmäßig folgende Implementationen von `Dialect`.

Datenbank | Dialect Klasse
--- | ---
MySQL | `n2n\\persistence\\meta\\dialect\\mysql\\MysqlDialect`
PostgreSQL | `n2n\\persistence\\meta\\dialect\\pgsql\\PgsqlDialect`
SQLite | `n2n\\persistence\\meta\\dialect\\sqlite\\SqliteDialect`
Microsoft SQL | `n2n\\persistence\\meta\\dialect\\mssql\\MssqlDialect`
Oracle | `n2n\\persistence\\meta\\dialect\\oracle\\OracleDialect`

### PDO-Objekt anfordern

Wie bereits erwähnt, werden Datenbank-Verbindungen in n2n über [PDO](http://php.net/manual/de/book.pdo.php) realisiert. In [magischen Methoden](http://support.web.n2n.ch/de/n2n/documentation/dependency-injections#magische-methoden) kannst du dir ein `n2n\\persistence\\Pdo`-Objekt der Persistence Unit "default" übergeben lassen.

```php showLineNumbers
private function _init(Pdo $pdo) {
    $this->pdo = $pdo;
}
```

`n2n\\persistence\\Pdo`-Objekte anderer Persistence Units musst du über den `n2n\\persistence\\PdoPool` anfordern.

```php showLineNumbers
private function _init(DbhPool $dbhPool) {
    $this->pdo = $dbhPool->getPdo('db2');
}
```

In diesem Beispiel fordern wir ein `n2n\\persistence\\Pdo`-Objekt der Persistence Unit mit dem Namen "db2" an.

`n2n\\persistence\\Pdo` erweitert `PDO` und bietet zusätzliche Methoden.

Entity
------

Eine Entity ist eine PHP-Klasse, die auf eine einzelne Tabelle in einer relationalen Datenbank abgebildet wird. Instanzen dieser Klasse entsprechen hierbei den Zeilen der Tabelle und die Eigenschaften den Spalten. Entities sind der Kern des ORM's und bilden normalerweise die Business-Logik deiner Applikation.

```php showLineNumbers
namespace atusch\\bo;

use n2n\\reflection\\ObjectAdapter;

class Article extends ObjectAdapter {
    const TYPE_NEWS = 'news';
    const TYPE_COMMENT = 'comment';

    private $id;
    private $title;
    private $text;
    private $type = self::TYPE_NEWS;
    private $online = false;

    public function __construct($title) {
        $this->title = $title;
    }

    public function getId() {
        return $this->id;
    }

    public function setId($id) {
        $this->id = $id;
    }

    public function getTitle() {
        return $this->title;
    }

    public function setTitle($title) {
        $this->title = $title;
    }

    public function getText() {
        return $this->text;
    }

    public function setText($text) {
        $this->text = $text;
    }

    public function getType() {
        return $this->type;
    }

    public function setType($type) {
        $this->type = $type;
    }

    public function isOnline() {
        return $this->online;
    }

    public function setOnline($online) {
        $this->online = $online;
    }
}
```

Es werden alle Eigenschaften unabhängig von ihrer Sichtbarkeit erkannt. Du kannst aber einzelne Eigenschaften ignorieren lassen, indem du sie mit `n2n\\persistence\\orm\\annotation\\AnnoTransient` annotierst. Wir empfehlen dir, Entities zu erstellen, die den `n2n\\reflection\\ObjectAdapter` erweitern. Der `ObjectAdapter` implementiert die statische Methode `ObjectAdapter::getClass()`, die dich beim Arbeiten mit dem [EntityManager](http://support.web.n2n.ch/de/n2n/documentation/persistence-orm#entitymanager) unterstützt.

Für die Entity `Article` im oben stehenden Beispiel wäre folgende Tabelle denkbar:

```sql
CREATE TABLE `article` (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `title` VARCHAR(50) DEFAULT NULL,
  `text` TEXT,
  `type` ENUM('news','comment') DEFAULT 'news',
  `online` TINYINT(4) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Wie du siehst, nimmt das ORM automatisch an, dass die Eigenschaft mit Namen "id" den Primary Key repräsentiert und die Werte von der Datenbank generiert werden. Wie du dies ändern kannst, erfährst du im Abschnitt [Id](http://support.web.n2n.ch/de/n2n/documentation/entity-konfigurieren#id) des Artikels [Entity konfigurieren](http://support.web.n2n.ch/de/n2n/documentation/entity-konfigurieren).

Alle Entity-Klassen müssen in der `app.ini` unter der Gruppe `[orm]` registriert werden.

\[orm\]
entities\[\] = "atusch\\\\bo\\\\Article"
entities\[\] = "atusch\\\\bo\\\\User"

EntityManager
-------------

Über den `EntityManager` kannst du Entities in die Datenbank speichern (persistieren) und auch wieder abfragen oder löschen. Der `EntityManager` für die Persistence Unit "default" lässt sich in allen [magischen Methoden](http://support.web.n2n.ch/de/n2n/documentation/dependency-injections#magische-methoden) übergeben. Du kannst zum Beispiel ein [Lookupable](http://support.web.n2n.ch/de/n2n/documentation/lookupables) `ArticleDao` implementieren, das den `EntityManager` über [`_init()`](http://support.web.n2n.ch/documentation/n2n/de/dependency-injections#magische-methoden) holt und verschiedene Methoden zum Lesen und Verwalten von `Article`-Entities zur Verfügung stellt.

```php showLineNumbers
class ArticleDao implements RequestScoped {
    private $em;

    private function _init(EntityManager $em) {
        $this->em = $em;
    }

    /**
     * @param int $id
     * @return atusch\\bo\\Article
     */
    public function getArticleById($id) {
        return $this->em->find(Article::getClass(), $id);
    }

    public function saveArticle(Article $article) {
        $this->em->persist($article);
    }
}
```

Dieses `ArticleDao` steht nun in jeder magischen Methode zur Verfügung und kann zum Beispiel in einem `ArticleController` verwendet werden.

```php showLineNumbers
public function doArticle(ArticleDao $articleDao, $id) {
    $article = $articleDao->getArticleById($id);
    if ($article === null) {
        throw new PageNotFoundException();
    }

    $this->forward('view\\\\article.html', array('article' => $article));
}
```

`EntityManager` für andere Persistence Units kannst du, wie bei den `Pdo`-Objekten, über den `PdoPool` anfordern.

```php showLineNumbers
private function _init(PdoPool $pdoPool) {
    $this->em = $pdoPool->getEntityManagerFactory("db2")->getExtended();
}
```

In diesem Beispiel holen wir zuerst die `EntityManagerFactory` der Persistence Unit "db2" und fordern dann über `EntityManagerFactory::getExtended()` einen `EntityManager` für diese Persistence Unit an.

Mit `EntityManagerFactory::getExtended()` forderst du einen "extended" `EntityManager` an, der mit der ganzen Applikation geteilt wird. Du kannst über `EntityManagerFactory::create()` auch einen neuen erstellen oder über `EntityManagerFactory::getTransactional()` einen `EntityManager` anfordern, der nur für die aktuelle Transaktion gültig ist. In den Artikeln [Lifecycle](http://support.web.n2n.ch/de/n2n/documentation/lifecycle) und [Transaktionen](http://support.web.n2n.ch/de/n2n/documentation/transaktionen) kannst du mehr über dieses Konzept erfahren.

### Abfragen

Wie im oben stehenden Beispiel ersichtlich, kannst du über `EntityManager::find()` eine einzelne Entity über ihren Primary Key abfragen.

```php showLineNumbers
/**
 * @param int $id
 * @return atusch\\bo\\Article
 */
public function getArticleById($id) {
    return $this->em->find(Article::getClass(), $id);
}
```

`find()` erwartet als ersten Parameter ein `ReflectionClass`-Objekt der abzufragenden Entity. Erweitert die gewünschte Entity `n2n\\persistence\\orm\\ObjectAdapter`, kannst du auf das jeweilige ReflectionClass-Objekt über `ObjectAdapter::getClass()` zugreifen.

Möchtest du komplexere Abfragen ausführen, kannst du wahlweise auf die Criteria API oder NQL (n2n Query Language) zurückgreifen.

#### Criteria API

Ein `Criteria`-Objekt beschreibt eine Abfrage. Es beschreibt, was, unter welchen Bedingungen und in welcher Reihenfolge selektiert werden soll. Die Beschreibungen beziehen sich dabei nicht auf Datenbanktabellen und Spalten, sondern auf Entities und ihre Eigenschaften. Mit `Criteria::toQuery()` lässt sich ein `Query`-Objekt bilden, über welches die Abfrage ausgeführt und die jeweiligen Daten ausgelesen werden können.

Um ein einfaches `Criteria` zusammenzustellen, das die Abfrage eines einzelnen Entity-Typs unter einfachen Vergleichsbedingungen beschreibt, eignet sich `EntityManager::createSimpleCriteria()`.

```php showLineNumbers
/**
 * @param string $type
 * @param int $limit
 * @param int $num
 * @return Article[]
 */
public function getArticlesByType($type, $limit, $num) {
    $criteria = $this->em->createSimpleCriteria(Article::getClass(),
            array('online' => true, 'type' => $type),
            array('id' => 'DESC'), $limit, $num);
    return $criteria->toQuery()->fetchArray();
}
```

`createSimpleCriteria()` erwartet als ersten Parameter die `ReflectionClass` der Entity, die du abfragen möchtest (beeinflusst den `SELECT`- und `FROM`-Abschnitt im SQL-Statement).

Über den zweiten Parameter kannst du einfache Vergleichsbedingungen in der Form _Eigenschaft = Wert_ bestimmen, wobei als Array-Schlüssel der Name der Eigenschaft und als Array-Wert der Vergleichswert erwartet (beeinflusst den `WHERE`-Abschnitt im SQL-Statement).

Über den dritten Parameter definierst du die Sortierreihenfolge (beeinflusst den `ORDER BY`-Abschnitt im SQL-Statement). Über die letzten beiden Parameter kannst du definieren, wie viele Article-Objekte maximal ausgeliefert werden sollen (beeinflusst `LIMIT`-Abschnitt im SQL-Statement).

Sobald du das Criteria in ein `Query` umgewandelt hast, kannst du mit `Query::fetchArray()` die Abfrage ausführen und das Resultat als `array` zurückgeben lassen. Im oben stehenden Beispiel gibt `Query::fetchArray()` ein `array` von `Article`-Objekten zurück. `ArticleDao::getArticlesByType()` könntest du also mit `$articleDao->getArticlesByType('news', 0, 30)` aufrufen, um die ersten 30 Artikel umgekehrt sortiert nach `Article::$id` abzufragen, bei denen `Article::$online` gleich `true` und `Article::$type` gleich "news" ist.

Erwartest du als Resultat nur eine einzelne Zeile, kannst du die Abfrage auch mit `Query::fetchSingle()` ausführen. `Query::fetchSingle()` gibt dir nur eine einzelne Zeile zurück, während `Query::fetchArray()` ein `array` mit einer Zeile zurückgeben würde.

```php showLineNumbers
/**
 * @param string $userName
 * @return User
 */
public function getUserByUserName($userName) {
    return $this->em->createSimpleCriteria(User::getClass(), array('userName' => $userName))
            ->toQuery()->fetchSingle();
}
```

In diesem Beispiel gibt `fetchSingle()` den User mit dem übergebenen Benutzername zurück. Ist das Resultat leer, wird `null` zurückgegeben. Umfasst das Resultat mehrere Zeilen, wird eine `QueryConflictException` geworfen.

Mit `EntityManager::createCriteria()` kannst du auch ein "leeres" Criteria erstellen und die Abfrage selbst beschreiben. Dies ermöglicht dir komplexere Abfragen zu formulieren.

```php showLineNumbers
/**
 * @param array $types
 * @return Article[]
 */
public function getArticlesByTypes(array $types) {
    $criteria = $this->em->createCriteria();
    $criteria->select('a')
            ->from(Article::getClass(), 'a')
            ->where()->match('a.online', '=', true)->andMatch('a.type', 'IN', $types);
    return $criteria->toQuery()->fetchArray();
}
```

`ArticleDao::getArticleDao()` des oben stehenden Beispiels könntest du mit `$articleDao->getArticlesByTypes(array('news', 'comment'))` aufrufen, um alle Article-Objekte abzufragen, bei denen `Article::$online` gleich `true` und `Article::$type` gleich "news" oder "comment" ist.

Dieser Artikel bietet nur eine kurze Einführung in die Criteria API. Ausführliche Informationen findest du im Artikel [Criteria API / NQL](http://support.web.n2n.ch/de/n2n/documentation/criteria-api-nql).

#### NQL (n2n Query Language)

Die n2n Query Language ähnelt syntaktisch SQL-Statements, beziehen sich aber, wie die Criteria API, auf Entities und dessen Eigenschaften statt auf Datenbanktabellen und Spalten. Mit `EntityManager::createNqlCriteria()` kannst du ein `Criteria`-Objekt auf Basis von NQL erstellen.

```php showLineNumbers
/**
 * @param string $type
 * @param int $limit
 * @param int $num
 * @return Article[]
 */
public function getArticlesByType($type, $limit, $num) {
    $criteria = $this->em->createNqlCriteria(
        'SELECT a FROM Article a WHERE a.online = :online AND a.type = :type',
        array('online' => true, 'type' => $type));
    return $criteria->limit(0, 30)->toQuery()->fetchAll();
}
```

In diesem Beispiel haben wir die Methode `ArticleDao::getArticlesByType()`, aus dem Beispiel weiter oben, auf Basis von NQL implementiert. `EntityManager::createNqlCriteria()` erwartet als ersten Parameter den NQL-String und als zweiten Parameter die Werte der Placeholder, die du in deinem NQL-Statement verwendet hast.

Placeholder-Werte kannst du auch über `Query::setParameter()` setzen. Der Unterschied dieser zwei Varianten wird im Abschnitt [Placeholders](http://support.web.n2n.ch/de/n2n/documentation/criteria-api-nql#placeholders) des Artikels [Criteria API / NQL](http://support.web.n2n.ch/de/n2n/documentation/criteria-api-nql) erläutert.

### Schreiben

Über den `EntityManager::persist()` kannst du Entities in die Datenbank speichern und mit `EntityManager::remove()` löschen. Operationen, die zu Datenbank-Mutationen führen, können nur innerhalb einer Transaktion ausgeführt werden. Am einfachsten ist es, Transaktionen im Controller zu verwalten. Über `ControllerAdapter::beginTransaction()` kannst du eine Transaktion starten und mit `ControllerAdapter::commit()` abschließen.

```php showLineNumbers
public function doCreateArticle(ArticleDao $articleDao) {
    $this->beginTransaction();

    $article = new Article('Lorem ipsum');
    $article->setText('Lorem ipsum dolor...');
    $article->setType(Article::TYPE_COMMENT);

    $articleDao->saveArticle($article);

    $this->commit();
}
```

`ArticleDao::saveArticle()` kann folgendermaßen aussehen:

```php showLineNumbers
public function saveArticle(Article $article) {
    $this->em->persist($article);
}
```

Die `INSERT`- und `DELETE`-Statements werden ausgeführt, sobald die Transaktion abgeschlossen wird (COMMIT). Beim Abschließen einer Transaktion werden auch alle Änderungen, die du an Entities vorgenommen hast, in der Datenbank aktualisiert (`UPDATE`-Statement).

```php showLineNumbers
public function doExample(ArticleDao $articleDao) {
    $this->beginTransaction();
    
    $article = $articleDao->getArticleById(1);
    $article->setTitle('New title');
    
    $this->commit();
}
```

In diesem Beispiel wird der Titel in der Datenbank automatisch aktualisiert, sobald `ControllerAdapter::commit()` aufgerufen wird (ein `UPDATE`-Statement wird abgesetzt).

Logger
------

Interessiert es dich, welche SQL-Statements der EntityManager generiert und absendet, kannst du sie über den `n2n\\persistence\\PdoLogger` ausgeben lassen. Der `PdoLogger` loggt alle Datenbankzugriffe eines `Pdo`-Objekts. Auf den jeweiligen `PdoLogger` kannst du über `Pdo::getLogger()` zugreifen.

```php showLineNumbers
public function saveArticle(Article $article) {
    $this->em->persist($article);
    
    var_dump($this->em->getPdo()->getLogger()->getEntries());
}
```

In diesem Beispiel holen wir über `EntityManager::getPdo()` das vom `EntityManager` verwendete `Pdo` und greifen über `PdoLogger::getEntries()` auf die gewünschten Log-Informationen zu.