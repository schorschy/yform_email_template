# yform_email_template

Ein Workflow zum Erstellen von HTML-E-Mails für Kontakt-Formulare

## Vorbereitung in CampaignMonitor

[CampaignMonitor](http://campaignmonitor.com) dient uns als Template-Generator, um mit möglichst wenig Aufwand eine HTML-Mail zu designen. Dort legen wir unter Templates ein Template an und gestalten es nach unseren Wünschen. Alternativ befindet sich in diesem Repository auch eine [HTML-Vorlage](template_allgemein.html).

Das in CampaignMonitor entworfene Tempalte befüllen wir mit Platzhalter-Text an den Stellen, an denen später der Nachrichteninhalt platziert wird. Anschließend lassen wir uns über eine Dummy-Kampagne zusenden und extrahieren den HTML-Code. 

Zusätzlich fügen wir für das Logo in der Kopfzeile der Mail einen Platzhalter anstelle der von CampaignMonitor vorgesehenen URL ein, vgl. mit der [HTML-Vorlage](template_allgemein.html).

Aus Campaign Monitor entfernt werden muss das Tracking-Bild am Ende des HTML-Codes. Außerdem zu ersetzen:
* `=3D` in `=`
* `=20` in ` `
* `=$\n` (regex) entfernen
* src-Pfad des Logos im IMG-Tag ersetzen durch eine URL auf dem Webserver, ggf. auch bei den Social Icons
* href-Pfad des Logo-Ankers ersetzen durch die URL zur Website
* "Einstellungen bearbeiten"-Link und "Abmelden"-Link entfernen, indem `<div class="... email-footer"...>` gelöscht wird.

Der E-Mail-HTML-Code wird dann in Redaxo als Template angelegt.

## Konfiguration des YForm-Formulars im Table Manager

Wir erstellen im Table Manager eine Tabelle `rex_yform_messages`, die die eingehenden Anfragen zusätzlich zum E-Mail-Versand aufzeichnen. Die Felder name, company, etc. anlegen.

## Konfiguration des YForm-Formulars im YForm-Modul

Wenn das YForm-Modul benutzt wird, kann die Pipe-Schreibweise verwendet werden, andernfalls müssen die entsprechenden Objparams, Values und Actions in der PHP-Variante verwendet werden.

```
objparams|form_skin|bootstrap
objparams|form_showformafterupdate|0
objparams|real_field_names|true

select|request|Anliegen|Auswahl 1,Auswahl 2,Auswahl 3|||0|
text|name|Name|
validate|empty|name|Bitte geben Sie einen Namen an.|
text|company|Firma|
text|adress|Adresse|
text|zip|PLZ|
text|city|Stadt|
text|phone|Telefonnummer|
text|email|E-Mail-Adresse|
validate|email|email|Bitte geben Sie eine gültige E-Mail-Adresse an.|
textarea|message|Nachricht|

action|tpl2email|contact_de||kunde@domain.de
action|tpl2email|contact_de_confirm|email|
action|db|rex_yf_messages|
```

## Konfiguration der E-Mail-Templates

Wir legen 2 Profile pro Sprache an:

* `contact_de` = Mail, die an die Mail-Adresse des Website-Betreiber geht
* `contact_de_confirm` = Mail, die an den Verfasser geht

Im BODY (HTML) holen wir uns das E-Mail-Template, befüllen die Platzhalter im Template mit den Variablen des Formulars und zusätzlichen Angaben.
```
<?

$template = new rex_template(9); // Hier Template ID eintragen

$emailValues['###template:email:color_hex###'] = '#cc007a';
$emailValues['###template:email:website_url###'] = 'http://domain.de/';
$emailValues['###template:email:logo_url###'] = 'http://domain.de/email_logo.png';
$emailValues['###template:email:client###'] = 'firma gmbh';
$emailValues['###template:email:headline###'] = 'Eine neue Anfrage auf domain.de';
$emailValues['###template:email:text###'] = 'Ein Interessent hat sich soeben bei Ihnen gemeldet.';
$emailValues['###template:email:details:headline###'] = 'Angaben';
$emailValues['###template:email:details:text###'] = '
REX_YFORM_DATA[field="name"]<br />
REX_YFORM_DATA[field="company"]<br />
REX_YFORM_DATA[field="adress"]<br />
REX_YFORM_DATA[field="zip"] REX_YFORM_DATA[field="city"]<br />
REX_YFORM_DATA[field="phone"]<br />
REX_YFORM_DATA[field="email"]<br />
REX_YFORM_DATA[field="request"]
';

$emailValues['###template:email:message:headline###'] = 'Nachricht';
$emailValues['###template:email:message:text###'] = 'REX_YFORM_DATA[field="message"]';

$emailValues['###template:email:imprint###'] = '';

echo str_replace(array_keys($emailValues), array_values($emailValues), $template->getTemplate());
?>
```

Für die normale Text-Mail ohne HTML fügen wir den gleichen Code aus der HTML-Mail hinzu, jedoch in der letzten Zeile mit `strip_tags()` behandelt:

```
echo strip_tags(str_replace(array_keys($emailValues), array_values($emailValues), $template->getTemplate()));
```

So stellen wir sicher, dass auch Personen ohne aktivierte HTML-Maildarstellung die Mail darstellen können.
