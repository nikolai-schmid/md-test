# Installation

Zurzeit kann n2n nur über Composer installiert werden. Weitere Varianten sind in Arbeit.

## Composer

Wir gehen davon aus, dass du Composer bereits installiert hast. Wenn nicht, dann findest du hier eine Anleitung.

Mit folgendem Befehl erstellst du ein n2n-Projekt mit der letzten stabilen n2n-Version.

```bash
composer create-project n2n/n2n-skeleton your-project-name
```

Alternativ kannst du mit nachstehendem Befehl auch ein n2n-Projekt mit der allerneusten n2n-Version erstellen. Dies kann eine stabile, aber auch eine Alpha-, Beta- oder RC-Version sein.

```bash
composer create-project n2n/n2n-skeleton=7.1.x your-project-name --prefer-dist --stability=dev
```

Anschliessend wirst du gefragt, ob du die Module n2n/rocket und n2n/page installieren möchtest. Für den n2n-Quickstart benötigst du diese zwei Module nicht, jedoch für die Fortsetzungen Rocket-Quickstart und Page-Quickstart.

Du kannst diese zwei Module auch zu einem späteren Zeitpunkt hinzufügen, indem du composer.json um die zwei \"require\":-Einträge n2n/rocket und n2n/page ergänzt.

```json
\"require\": {
        ...
        \"n2n/rocket\": \"1.1.*\",
        \"n2n/page\": \"1.3.*\"
}
```

Hast du das Projekt mit n2n/n2n-skeleton=7.1.x (dem zweiten Befehl in diesem Abschnitt) erstellt, dann verwende folgende \"require\":-Einträge:

```json
\"require\": {
        ...
        \"n2n/rocket\": \"1.1.x-dev\",
        \"n2n/page\": \"1.3.x-dev\"
}
```

Führe anschliessend composer update aus, um diese zwei Module zu installieren.
