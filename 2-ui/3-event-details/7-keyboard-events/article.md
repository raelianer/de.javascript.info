# Keyboard: keydown und keyup

Bevor wir zur Tastatur kommen, beachten Sie bitte, dass es auf modernen Geräten andere Möglichkeiten gibt, "etwas einzugeben". Zum Beispiel verwenden Menschen Spracherkennung (besonders auf mobilen Geräten) oder Kopieren/Einfügen mit der Maus.

Wenn wir also jegliche Eingaben in ein `<input>` Feld erfassen wollen, dann reichen Tastatur-Events nicht aus. Es gibt ein weiteres Event namens `input`, welches beliebige Änderungen eines `<input>` Feldes verfolgt. Und wahrscheinlich ist es die bessere Wahl für eine solche Aufgabe. Wir werden es später im Kapitel <info:events-change-input> behandeln.

Tastatur-Events sollten verwendet werden, wenn wir mit Tastatur-Aktionen umgehen wollen (die virtuelle Tastatur zählt auch dazu). Zum Beispiel, um auf die Pfeiltasten `key:Up` und `key:Down` oder auf Hotkeys (einschließlich Tastenkombinationen) zu reagieren.


## Prüfstand [#keyboard-test-stand]

```offline
Um Tastatur-Events besser zu verstehen, können Sie den [Prüfstand](sandbox:keyboard-dump) benutzen.
```

```online
Um Tastatur-Events besser zu verstehen, können Sie den Prüfstand unten verwenden.

Probieren Sie verschiedene Tastenkombinationen im Textfeld aus.

[codetabs src="keyboard-dump" height=480]
```


## Keydown und keyup

Das `Key-Down`-Event tritt ein, wenn eine Taste gedrückt wird, und `Key-Up` -- wenn sie losgelassen wird.

### event.code und event.key

Die Eigenschaft `key` des Event-Objekts erlaubt es, das Schriftzeichen zu erhalten, während die Eigenschaft `code` des Event-Objekts es erlaubt, den "physikalischen Tastencode" zu erhalten.

Zum Beispiel kann die gleiche Taste `key:Z` mit oder ohne `key:Shift` gedrückt werden. Dadurch erhalten wir zwei verschiedene Zeichen: Kleinbuchstaben `z` und Großbuchstaben `Z`.

Der `event.key` wird für das gleiche Zeichen unterschiedlich sein, doch der `event.code` bleibt derselbe:

| Taste          | `event.key` | `event.code` |
|--------------|-------------|--------------|
| `key:Z`      |`z` (Kleinschreibung)         |`KeyZ`        |
| `key:Shift+Z`|`Z` (Großschreibung)          |`KeyZ`        |


Wenn ein Benutzer mit verschiedenen Sprachen arbeitet, dann würde der Wechsel zu einer anderen Sprache ein ganz anderes Zeichen als `"Z"` ergeben. Das neue Zeichen würde zum Wert von `event.key`, während der `event.code` immer gleich bleibt: `"KeyZ"`.

```smart header="\"KeyZ\" und andere Tastencodes"
Jede Taste hat einen Code, der von ihrer Position auf der Tastatur abhängt. Die Tastencodes sind in der [UI Event Code-Spezifikation](https://www.w3.org/TR/uievents-code/) beschrieben.

Zum Beispiel:
- Buchstabentasten haben die Codes `"Key<letter>"`: `"KeyA"`, `"KeyB"` usw.
- Zifferntasten haben die codes: `"Digit<number>"`: `"Digit0"`, `"Digit1"` usw.
- Sondertasten werden durch ihre Namen kodiert: `"Enter"`, `"Backspace"`, `"Tab"` usw.

Es gibt diverse weit verbreitete Tastaturlayouts, und die Spezifikation gibt für jedes dieser Layouts Tastencodes an.

Lesen Sie den [alphanumerischen Abschnitt der Spezifikation](https://www.w3.org/TR/uievents-code/#key-alphanumeric-section) für weitere Codes, oder drücken Sie einfach eine Taste im [Prüfstand](#Tastatur-Prüfstand) oben.
```

```warn header="Groß- und Kleinschreibung zählt: `\"KeyZ\"`, nicht `\"keyZ\"`"
Scheint offensichtlich, aber Menschen machen Fehler.

Bitte vermeiden Sie Tippfehler: Es ist `KeyZ`, nicht `keyZ`. Eine Prüfung wie `event.code=="keyZ"` wird nicht funktionieren: der erste Buchstabe von `"Key"` muss großgeschrieben werden.
```

Was ist, wenn eine Taste kein Zeichen ausgibt? Zum Beispiel `key:Shift` oder `key:F1` oder andere. Für diese Tasten ist `event.key` ungefähr das Gleiche wie `event.code`:

| Taste          | `event.key` | `event.code` |
|--------------|-------------|--------------|
| `key:F1`      |`F1`          |`F1`        |
| `key:Backspace`      |`Backspace`          |`Backspace`        |
| `key:Shift`|`Shift`          |`ShiftRight` or `ShiftLeft`        |

Bitte beachten Sie, dass `event.code` genau angibt, welche Taste gedrückt wird. Zum Beispiel haben die meisten Tastaturen zwei `key:Shift`: auf der linken und auf der rechten Seite. Der `event.code` sagt uns genau, welche Taste gedrückt wurde, und `event.key` ist verantwortlich für die "Bedeutung" der Taste: was sie ist (eine "Umschalttaste").

Nehmen wir an, wir wollen einen Hotkey bedienen: `key:Ctrl+Z` (oder `key:Cmd+Z` für Mac). Die meisten Texteditoren hängen die Aktion "Rückgängig" daran. Wir können einen Listener auf `keydown` setzen und überprüfen, welche Taste gedrückt wird.

Hier gibt es ein Dilemma: Sollten wir bei einem solchen Zuhörer den Wert von `event.key` oder `event.code` überprüfen?

Einerseits ist der Wert von `event.key` ein Zeichen, er ändert sich je nach Sprache. Wenn der Besucher mehrere Sprachen im Betriebssystem hat und zwischen ihnen wechselt, liefert dieselbe Taste unterschiedliche Zeichen. Es ergibt also Sinn, `event.code` zu überprüfen, er ist immer derselbe.

Etwa so:

```js run
document.addEventListener('keydown', function(event) {
  if (event.code == 'KeyZ' && (event.ctrlKey || event.metaKey)) {
    alert('Undo!')
  }
});
```

Andererseits gibt es ein Problem mit `event.code`. Bei verschiedenen Tastaturlayouts kann dieselbe Taste unterschiedliche Zeichen haben.

Darunter befinden sich zum Beispiel das US-amerikanische Layout ("QWERTY") und das deutsche Layout ("QWERTZ") (aus Wikipedia):

![](us-layout.svg)

![](german-layout.svg)

Die gleiche Taste ist im US-Layout ein "Z" und im deutschen Layout ein "Y" (die Buchstaben sind vertauscht).

Für Personen mit deutschem Layout ist der `event.code` gleich `KeyZ`, wenn sie die Taste `Y` drücken.

Falls wir in unserem Code `event.code == 'KeyZ'` testen, dann wird für Leute mit deutschem Layout ein solcher Test bestanden, wenn sie `key:Y` drücken.

Das klingt wirklich seltsam, aber so ist es. In der [Spezifikation](https://www.w3.org/TR/uievents-code/#table-key-code-alphanumeric-writing-system) wird ein solches Verhalten ausdrücklich erwähnt.

Daher kann `event.code` bei unerwartetem Layout ein falsches Zeichen enthalten. Dieselben Buchstaben in unterschiedlichen Layouts können auf unterschiedliche physikalische Tasten abgebildet werden, was zu unterschiedlichen Codes führen kann. Glücklicherweise passiert das nur bei einigen Codes, z.B. `keyA`, `keyQ`, `keyZ` (wie wir gesehen haben), und nicht bei Sondertasten wie `Shift`. Sie finden die Liste in der [Spezifikation](https://www.w3.org/TR/uievents-code/#table-key-code-alphanumeric-writing-system).

Um layout-abhängige Zeichen zuverlässig zu erfassen, könnte `event.key` ein besserer Weg sein.

Auf der anderen Seite hat der `event.code` den Vorteil, dass er immer derselbe bleibt, gebunden an den physischen Ort der Taste, auch wenn der Besucher die Sprache wechselt. Hotkeys, die darauf angewiesen sind, funktionieren also auch im Falle eines Sprachwechsels gut.

Wollen wir mit layout-abhängigen Tasten zurechtkommen? Dann ist `event.key` der richtige Weg.

Oder wir wollen, dass ein Hotkey auch nach einem Sprachwechsel funktioniert? Dann ist `event.code` vielleicht besser.

## Automatische Wiederholung

Wenn eine Taste lange genug gedrückt wird, fängt sie an, sich "automatisch zu wiederholen": Der `keydown` wird immer wieder ausgelöst, und wenn sie losgelassen wird, bekommen wir schließlich `keyup`. Es ist also ganz normal, viele `keydown` zu haben und ein einziges `keyup`.

Bei Ereignissen, die durch automatische Wiederholung ausgelöst werden, hat das Event-Objekt die Eigenschaft `event.repeat` auf `true` gesetzt.


## Standard-Aktionen

Die Standardaktionen variieren, da es viele mögliche Dinge gibt, die über die Tastatur ausgelöst werden können.

Zum Beispiel:

- Ein Zeichen erscheint auf dem Bildschirm (das offensichtlichste Ergebnis).
- Ein Zeichen wird gelöscht (Taste `key:Delete`).
- Die Seite wird gescrollt (`key:PageDown` Taste).
- Der Browser öffnet den "Seite speichern"-Dialog (`key:Ctrl+S`)
- ...und so weiter.

Das Unterbinden der Standardaktion bei `keydown` kann die meisten Aktionen verhindern, mit Ausnahme der Betriebssystem-basierten Sondertasten. Zum Beispiel schließt unter Windows `key:Alt+F4` das aktuelle Browserfenster. Und es gibt keine Möglichkeit, dies durch Unterbinden der Standardaktion in JavaScript zu verhindern.

Zum Beispiel erwartet das `<input>` unten eine Telefonnummer, daher akzeptiert es keine Tasten außer Ziffern, `+`, `()` oder `-`:

```html autorun height=60 run
<script>
function checkPhoneKey(key) {
  return (key >= '0' && key <= '9') || key == '+' || key == '(' || key == ')' || key == '-';
}
</script>
<input *!*onkeydown="return checkPhoneKey(event.key)"*/!* placeholder="Telefonnummer bitte" type="tel">
```

Bitte beachten Sie, dass Sondertasten, wie z.B. `key:Backspace`, `key:Left`, `key:Right`, `key:Ctrl+V`, bei der Eingabe nicht funktionieren. Das ist ein Nebeneffekt des strengen Filters `checkPhoneKey`.

Lockern wir es ein wenig:


```html autorun height=60 run
<script>
function checkPhoneKey(key) {
  return (key >= '0' && key <= '9') || key == '+' || key == '(' || key == ')' || key == '-' ||
    key == 'ArrowLeft' || key == 'ArrowRight' || key == 'Delete' || key == 'Backspace';
}
</script>
<input onkeydown="return checkPhoneKey(event.key)" placeholder="Telefonnummer bitte" type="tel">
```

Jetzt funktionieren Pfeiltasten und Löschen gut.

...Aber wir können immer noch alles eingeben, indem wir mit der Maus und der rechten Maustaste + Einfügen klicken. Der Filter ist also nicht 100% zuverlässig. Wir können es einfach so belassen, denn meistens funktioniert er. Oder ein alternativer Ansatz wäre, das `input`-Event zu verfolgen - es wird nach jeder Änderung ausgelöst. Dort können wir den neuen Wert überprüfen und ihn hervorheben/modifizieren, wenn er ungültig ist.

## Altlasten

In der Vergangenheit gab es ein `keypress`-Event und auch die Eigenschaften `keyCode`, `charCode`, `which` des Event-Objekts.

Es gab so viele Browser-Inkompatibilitäten beim Umgang mit ihnen, dass den Entwicklern der Spezifikation nichts anderes übrig blieb, als sie alle zu verwerfen und neue, moderne Events zu erstellen (oben in diesem Kapitel beschrieben). Der alte Code funktioniert noch immer, da die Browser sie weiterhin unterstützen, aber es besteht überhaupt keinen Grund mehr, sie zu verwenden.

## Zusammenfassung

Das Drücken einer Taste erzeugt immer ein Tastaturereignis, seien es Symboltasten oder Sondertasten wie `key:Shift` oder `key:Ctrl` und so weiter. Die einzige Ausnahme ist die Taste `key:Fn`, die manchmal auf einer Laptop-Tastatur erscheint. Dafür gibt es kein Tastatur-Ereignis, weil es oft auf einer niedrigeren Ebene als dem Betriebssystem implementiert ist.

Tastatur events:

- `keydown` -- beim Drücken der Taste (automatische Wiederholung, wenn die Taste lange gedrückt wird),
- `keyup` -- beim Loslassen der Taste.

Haupteigenschaften des Tastatur-Events:

- `code` -- der "Tastencode" (`"KeyA"`, `"ArrowLeft"` usw.), der spezifisch für die physische Position der Taste auf der Tastatur ist.
- `key` -- das Zeichen (`"A"`, `"a"` und so weiter), für zeichenlose Tasten, wie z.B. `key:Esc`, ist es normalerweise der gleiche Wert wie `code`.

In der Vergangenheit wurden Tastatur-Events manchmal verwendet, um Benutzereingaben in Formularfeldern zu erfassen. Das ist nicht zuverlässig, da die Eingaben aus verschiedenen Quellen stammen können. Wir haben `input`- und `change`-Events, um jede Eingabe zu verarbeiten (später im Kapitel <info:events-change-input> behandelt). Sie lösen nach jeder Art von Eingabe aus, einschließlich Kopieren und Einfügen oder Spracherkennung.

Wir sollten Tastatur-Events verwenden, wenn wir wirklich die Tastatur wollen. Zum Beispiel, um auf Hotkeys oder Sondertasten zu reagieren.
