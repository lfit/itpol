========================================
Linux-Workstation Sicherheits-Checkliste
========================================

Zielgruppe
==========

Dieses Dokument richtet sich an Teams von Systemadministratoren, die 
Linux-Workstations verwenden, um in ihrem Projekt auf IT-Infrastruktur
zuzugreifen und diese zu verwalten.

Systemadministratoren können diese Richtlinien verwenden, um sicherzustellen,
dass ihre Arbeitsplätze grundlegenden Sicherheitsanforderungen entsprechen, die
das Risiko, ein Angriffsvektor gegen die gesamte IT-Infrastruktur zu sein,
reduzieren.

Einschränkungen
===============

Dies ist kein umfassendes Dokument zum Härten von Workstations, sondern der
Versuch, mit einer Reihe von Empfehlungen die gravierensten Sicherheitsfehler zu
vermeiden, ohne zu viele Unannehmlichkeiten einzuführen. Manche werden dieses
Dokument lesen und denken, wir sind paranoid; während wir für andere kaum an
der Oberfläche kratzen. Diese Richtlinien sollen lediglich eine Reihe
grundlegender Fehler vermeiden helfen, ohne ein Ersatz für Erfahrung,
Wachsamkeit und gesunden Menschenverstand zu sein.

Wir teilen dieses Dokument, um die Vorteile von Open-Source auf die
Dokumentation von IT-Richtlinien zu übertragen. Gerne können Sie dieses Dokument
für Ihre Organisation kopieren und uns Ihre Verbesserungen mitteilen.

Struktur
========

Jeder Abschnitt ist in zwei verschiedene Bereiche unterteilt:

#. Die Checkliste, die Ihren Projektanforderungen angepasst werden kann.
#. Überlegungen, die erklären, wie es zu diesen Entscheidungen kam.

Die Checkliste unterscheidet zwischen den folgenden drei Prioritätsstufen:

wesentlich
    Diese Aufgaben sollten auf jeden Fall umgesetzt werden.

    Falls sie nicht umgesetzt sind, besteht ein hohes Risiko bzgl. der
    Sicherheit Ihrer Workstation.

wünschenswert
    Aufgaben, die die allgemeine Sicherheit Ihrer Workstation verbessern, die
    jedoch neue Arbeitsweisen erfordern.

paranoid
    Aufgaben, die die Sicherheit Ihrer Workstation verbessern, jedoch mit
    erheblichem Aufwand bei der Umsetzung verbunden sind.

Denken Sie bitte daran, dass es sich im Folgenden nur um Richtlinien handelt,
die Sie entsprechend Ihrem Sicherheitsniveau anpassen sollten, genau so wie Sie
für richtig halten.

1. Die Wahl der richtigen Hardware
==================================

Checkliste
----------

Wir präferieren keinen bestimmten Anbieter oder bestimmte Modelle, so dass der
folgende Abschnitt nur die wesentlichen Überlegungen für die Wahl einer
Workstation enthalten.

* ☐ System unterstützt SecureBoot (wesentlich)
* ☐ System hat keine Firewire-, Thunderbolt- oder Express-Ports (wünschenswert)
* ☐ System verfügt über einen TPM-Chip (wünschenswert)

Überlegungen
------------

SecureBoot
~~~~~~~~~~

Obwohl umstritten, beugt SecureBoot gut gegen viele Angriffe auf Workstations
vor, wie z.B. gegen Rootkits, *Evil Maid* und andere Bootkits etc. Es wird
keinen hinreichenden Schutz vor versierten Angreifern und staatlichen
Sicherheitsbehörden bieten, es ist jedoch besser als gar nichts.

Alternativ können Sie auch `Anti Evil Maid
<https://github.com/QubesOS/qubes-antievilmaid>`_ einrichten, das einen besseren
Schutz gegen diese Art von Angriffen bietet. Es erfordert jedoch einen höheren
Aufwand bei der Einrichtung und Pflege.

Firewire, Thunderbolt und ExpressCard-Ports
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Firewire ist ein Standard, der vom Design her in jeder Verbindungseinrichtung
direkten Speicherzugriff (DMA) auf Ihr System ermöglicht.

Für Thunderbolt und ExpressCard gilt das Gleiche, obwohl einige spätere
Implementierungen von Thunderbolt den Versuch unternehmen, den Umfang des
Speicherzugriffs zu begrenzen. Am besten ist es, wenn die Hardware keine dieser
Ports bereitstellt, alternativ können diese jedoch üblicherweise via UEFI
deaktiviert oder im Kernel selbst ausgeschaltet werden.

Trusted Platform Module (TPM)-Chip
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Das *Trusted Platform Module* (TPM) ist ein Kryptochip, der mit dem Motherboard
verbunden, jedoch getrennt vom Core-Prozessor ist. Er soll für zusätzliche
Sicherheit dienen (wie die Speicherung der Schlüssel für die
Festplattenverschlüsselung). Sofern kein besonderer Bedarf besteht, ist der Chip
wünschenswert, jedoch nicht unbedingt erforderlich, um die Sicherheit einer
Workstation zu erhöhen.

2. Vor dem Booten
=================

Checkliste
----------

* ☐ Verwenden des UEFI-Boot-Modus, nicht eines veralteten BIOS (wesentlich)
* ☐ Ein Passwort ist erforderlich zur Verwendung der UEFI-Konfiguration
  (wesentlich)
* ☐ SecureBoot ist aktiviert (wesentlich)
* ☐ UEFI-Level-Passwort ist erforderlich um das System zu booten (wünschenswert)

Überlegungen
------------

UEFI und SecureBoot
~~~~~~~~~~~~~~~~~~~

UEFI bietet eine Menge Vorteile, die ein älterer BIOS nicht bietet. Die meisten
modernen Systeme haben einen aktivierten UEFI-Modus.

Stellen Sie sicher, dass ein starkes Passwort verwendet wird, um den UEFI-
Konfigurationsmodus aufzurufen. Beachten Sie, dass manche Hersteller
stillschweigend die Länge des Passworts begrenzen, sodass Sie gezwungen sein
könnten, kürzere Passwörter mit hoher Entropie zu verwenden (s.u. für weitere
Informationen über Passphrasen).

Je nach verwendeter Linux-Distribution ist es unterschiedlich aufwändig, in die
Distribution einen SecureBoot-Schlüssel zu importieren, der dann erst das Booten
der Distribution erlaubt. Viele Distributionen sind eine Partnerschaft mit
Microsoft eingegangen, um ihre Kernel mit einem Schlüssel zu signieren, der von
den meisten Systemherstellern erkannt wird. Dies erspart Ihnen dann die Mühe,
sich selbst mit dem Schlüsselimport zu beschäftigen.

Als zusätzliche Maßnahme kann ein Passwort für den Zugriff auf die
Boot-Partition vergeben werden. Dieses Passwort sollte sich von Ihrem
UEFI-Management-Passwort unterscheiden, um *shoulder-surfing* zu verhindern.
Wenn Sie Ihre Workstation häufiger booten, können Sie auch eine LUKS-Passphrase wählen, die zusätzliche Eingaben überflüssig macht.

3. Wahl der Distribution
========================

Vermutlich werden Sie bei einer der weit verbreiteten Distributionen wie Fedora
Ubuntu, Arch, Debian, oder einer ihrer Ableger landen. In jedem Fall sollten
Sie das Folgende beachten, bevor Sie sich für eine Distribution entscheiden.

Checkliste
----------

* ☐ verfügt über eine robuste MAC/RBAC-Implementierung
  (SELinux/AppArmor/grsecurity). (wesentlich)
* ☐ publiziert Sicherheitsmitteilungen. (wesentlich)
* ☐ bietet rechtzeitige Sicherheits-Patches. (wesentlich)
* ☐ bietet die kryptographische Verifizierung von Paketen. (wesentlich)
* ☐ bietet volle Unterstützung für UEFI und SecureBoot. (wesentlich)
* ☐ bietet robuste, native und vollständige Festplattenverschlüsselung.
  (wesentlich)

Überlegungen
------------

SELinux, AppArmor und grsecurity/PaX
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mandatory Access Control (MAC) oder Role Based Access Controls (RBAC) sind
Erweiterungen des grundlegenden Benutzer-Gruppen-Sicherheitsmechanismus
von POSIX-Systemen. Die meisten aktuellen Distributionen kommen entweder
bereits mit einer MAC/RBAC-Implementierung (Fedora, Ubuntu) oder stellen einen
anderen Mechanismus bereit, um sie in einem späteren Schritt hinzufügen zu
können (Gentoo, Arch, Debian).

Von Distributionen, die keine MAC/RBAC-Mechanismen vorsehen, wird dringend
abgeraten, da die übliche POSIX-Benutzer- und Gruppenbasierten
Sicherheitsmechanismen heutzutage nicht mehr ausreichend sind. Wenn Sie mit
MAC/RBAC beginnen möchten, so sind AppArmor und grsecurity/PaX in der Regel
leichter zu erlernen als SELinux. Zudem dürfte auf einer Workstation mit keinen
oder wenigen auf öffentlichen Ports lauschenden Daemons die Ausführung von
Benutzer-Anwendungen das höchste Risiko darstellen und damit grsecurity/PaX mehr
Sicherheit bieten als SELinux.

Sicherheitsmitteilungen der Distribution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Die meisten der weit verbreiteten Distributionen teilen ihren Nutzern
zuverlässig sicherheitsrelevante Informationen mit. Bei etwas exotischeren
Installationen sollte jedoch überprüft werden, ob die Distribution hierfür eine
zuverlässige Alarmierung der Benutzer über Sicherheitslücken und Patches
umgesetzt hat. Fehlt ein solcher Mechanismus, ist das ein wichtiges
Warnsignal, dass die Distribution noch nicht ausgereift ist. 

Rechtzeitige und vertrauenswürdige Sicherheits-Updates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Die meisten der weit verbreiteten Distributionen liefern regelmäßige
Sicherheits-Updates. Sie unterscheiden sich jedoch deutlich in der Art und
Weise, wie diese Pakete bereitgestellt werden. Vermeiden Sie daher Ableger und
*Community Rebuilds*, da sich dort üblicherweise die Sicherheitsupdates
verzögern.

Die meisten Distributionen signieren auch ihre Pakete und Metadaten. 

UEFI- und SecureBoot-Unterstützung
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Überprüfen Sie, ob die Distribution UEFI und SecureBoot unterstützt. Finden Sie
heraus, ob zusätzliche Schlüssel importiert werden müssen oder ob die
Distribution den Kernel mit einem Schlüssel signiert, dem die Systemhersteller
(z.B. über eine Vereinbarung mit Microsoft) vertrauen. Einige Distributionen
nutzen nicht UEFI/SecureBoot, sondern verwenden manipulationssicherere
Boot-Umgebungen. So nutzt z.B. `Qubes-OS <https://qubes-os.org/>`_ das oben
bereits erwähnte `Anti Evil Maid`_.
Unterstützt eine Distribution jedoch weder SecureBoot, noch einen anderen
Mechanismus, der Angriffe auf den Boot-Prozess verhindert, so sollten Sie sich
eine andere Distribution suchen.

Festplattenverschlüsselung
~~~~~~~~~~~~~~~~~~~~~~~~~~

Festplattenverschlüsselung ist eine Voraussetzung, um gespeicherte Daten zu
sichern, und sie wird von den meisten Distributionen unterstützt. Als
Alternative können selbstverschlüsselnde Festplatten verwendet werden, die dies
in der Regel über den On-Board-TPM-Chip implementieren. Sie bieten ein
vergleichbares Sicherheitsniveau bei schnellerem Betrieb, jedoch auch bei
höheren Kosten.

4. Installation
===============

Alle Distributionen sind unterschiedlich, aber hier sind allgemeine
Richtlinien:

Checkliste
----------

* ☐ Verwenden Sie vollständige Festplattenverschlüsselung (LUKS) mit einem
  robusten Passwort (wesentlich)
* ☐ Stellen Sie sicher, dass die Swap-Partition ebenfalls verschlüsselt wird
  (wesentlich)
* ☐ Zum Bearbeiten des Bootloader muss ein Kennwort erforderlich sein. Es
  kann dasselbe wie LUKS sein. (wesentlich)
* ☐ Verwenden Sie ein robustes Root-Passwort. Es kann dasselbe wie LUKS
  sein. (wesentlich)
* ☐ Verwenden Sie ein unprivilegiertes Konto, das Teil der Gruppe
  *administrators* ist (wesentlich)
* ☐ Verwenden Sie für dieses Konto ein robustes Passwort, das sich vom Root-
  Passwort unterscheidet (wesentlich)

Überlegungen
------------

Festplattenverschlüsselung
~~~~~~~~~~~~~~~~~~~~~~~~~~

Vorausgesetzt Sie verwenden keine selbstverschlüsselnden Festplatten, müssen
Sie Ihr Installationsprogramm so konfigurieren, dass es vollständig alle
Laufwerke verschlüsselt, die für die Speicherung Ihrer Daten und Systemdateien
verwendet werden sollen. Es ist nicht hinreichend, einfach nur das
Benutzerverzeichnis über auto-mounting cryptfs loop-Dateien zu verschlüsseln,
wie dies z.B. bei älteren Versionen von Ubuntu der Fall war. Dies bietet keinen
Schutz der Systemdateien und des Swap, die eine ganze Reihe sensibler Daten
enthalten. Wir empfehlen die LVM-Laufwerke zu verschlüsseln. so dass nur ein
Passwort während des Bootvorgangs erforderlich ist. 

Die ``/boot``-Partition wird in der Regel unverschlüsselt bleiben, da der
Bootloader den Kernel vor dem Aufruf von LUKS/dm-crypt booten muss. Einige
Distributionen unterstützen die Verschlüsselung der ``/boot``-Partition (z.B.
`Arch <http://www.pavelkogan.com/2014/05/23/luks-full-disk-encryption/>`_),
und es ist auch möglich, dies auf andere Distributionen zu übertragen, jedoch
voraussichtlich auf Kosten der Komplexität von System-Updates. Es erscheint uns
nicht erforderlich, die ``/boot``-Partition zu verschlüsseln, wenn Ihre
Distribution dies nicht nativ unterstützt, da das Kernel-Image selbst keine
privaten Daten enthält und gegen Manipulation mit einer SecureBoot-Signatur
geschützt wird.

Die Wahl guter Passphrasen
~~~~~~~~~~~~~~~~~~~~~~~~~~

Moderne Linux-Systeme haben keine Begrenzung der Passwort/Passphrasen-
Länge, so dass die einzige wirkliche Einschränkung Ihr Paranoia-Niveau ist.

Wenn Sie Ihr System häufig booten, werden Sie wahrscheinlich mindestens zwei
verschiedene Passwörter eingeben müssen: eins, um LUKS zu entsperren und ein
anderes, um sich anzumelden. In diesem Fall werden Sie vermutlich mit langen
Passphrasen nicht glücklich werden. Vermutlich empfehlen sich hier Passwörter
mit höherer Entropie. Jedoch sollten auch diese nie weniger als 12 Zeichen lang
sein.

Root, Benutzerkennwörter und die Admin-Gruppe
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sie können problemlos dieselbe Passphrase als Root-Passwort und für die
LUKS-Verschlüsselung verwenden, sofern nicht andere vertrauenswürdige
Personen die Laufwerke entschlüsseln sollen, ohne Root werden zu dürfen. Im
Allgemeinen können Sie die gleichen Passphrasen für Ihre UEFI-Administration,
Festplattenverschlüsselung und Root-Konto verwenden – da ein Angreifer mit
jedem der Zugänge volle Kontrolle über Ihr System gewinnen kann.

Für Ihr normales Benutzerkonto, mit dem Sie Ihre täglichen Aufgaben erledigen
können, sollten Sie ein anderes, aber ebenso starkes Passwort verwenden.
Dieses Benutzerkonto sollte Mitglied der ``admin``-Gruppe (oder ``wheel`` o.ä.
je nach Distribution) sein und Ihnen die Erweiterung der Privilegien mit
``sudo`` erlauben.

Mit anderen Worten: Auch wenn Sie der einzige Benutzer auf Ihrer Workstation
sind, sollten Sie zwei verschiedene, ebenso starke Passwörter haben, an die Sie
sich ggf. erinnern müssen als:

* **Admin** für

  * die Verwaltung von UEFI
  * den Bootloader (GRUB)
  * die Festplattenverschlüsselung (LUKS)
  * und Root auf der Workstation

* **Benutzer** für:

  * das Benutzerkonto und ``sudo``
  * das Master-Passwort des Passwort-Managers

``Rkhunter`` und IDS
~~~~~~~~~~~~~~~~~~~~

Die Installation von ``rkhunter`` und einem Intrusion Detection System (IDS) wie
``aide`` oder ``tripwire`` wird nicht wirklich nützlich sein, wenn Sie nicht
wirklich verstehen, wie diese funktionieren. Nur dann werden Sie die
notwendigen Schritte unternehmen können, um sie richtig einzurichten wie z.B.

* die Datenbanken auf externen Medien zu halten
* regelmäßige Überprüfungen von vertrauenswürdigen Umgebungen
* Aktualisieren der Hash-Datenbanken nach der Durchführung von System-Updates
  und Konfigurationsänderungen
* etc.

Wenn Sie nicht bereit sind, diese Maßnahmen zu ergreifen, und Ihre Workstation
entsprechend einstellen, werden diese Werkzeuge ohne greifbaren
Sicherheitsnutzen bleiben. Wir empfehlen die Installation und Konfiguration von
``rkhunter`` sodass es jede Nacht läuft. Auch ist es ziemlich einfach zu
erlernen und zu benutzen. Und selbst wenn es versierte Angreifer nicht davon
abhalten kann, so wird es Ihnen dennoch helfen, einige Ihrer eigenen Fehler zu
erkennen.

5. Härtung
==========

Die Härtung der Sicherheit nach der Installation ist stark von der Distribution
abhängig weswegen wir an dieser Stelle keine detaillierten Anweisungen geben
können sondern nur allgemeine. Hier einige Schritte, die Sie beachten sollten:

* ☐ Firewire und Thunderbolt global deaktivieren (wesentlich)
* ☐ Überprüfen Sie alle eingehenden Ports in Ihrer Ihre Firewall um
  sicherzustellen, dass diese gefiltert werden (wesentlich)
* ☐ Stellen Sie sicher, dass ``root``-Mails an ein Konto weitergeleitet werden,
  das regelmäßig überprüft wird (wesentlich)
* ☐ Richten Sie einen Zeitplan für automatische OS-Updates oder Update-
  Erinnerungen ein. (wesentlich)
* ☐ Überprüfen Sie, dass der ``sshd``-Service standardmäßig deaktiviert ist
  (wünschenswert)
* ☐ Konfigurieren Sie den Bildschirmschoner so, dass er automatisch nach
  einer gewissen Zeit der Inaktivität die Eingabe sperrt (wünschenswert)
* ☐ Richten Sie ``logwatch`` ein (wünschenswert)
* ☐ Installieren und verwenden Sie ``rkhunter`` (Rootkit Hunter) (wünschenswert)
* ☐ Installieren Sie ein Intrusion Detection System (wünschenswert)

Überlegungen
------------

Blacklisting
~~~~~~~~~~~~

Um FireWire und Thunderbolt-Module auf die Backlist zu setzen, fügen Sie die
folgenden Zeilen in die Datei ``/etc/modprobe.d/blacklist-dma.conf`` ein::

    blacklist firewire-core
    blacklist thunderbolt

Beide Module werden bei einem anschließenden Neustart auf die Schwarze Liste
gesetzt. 

Root-Mail
~~~~~~~~~

Standardmäßig werden Root-Mails auf dem System nur gespeichert und man neigt
dazu, diese nie zu lesen. Stellen Sie in ``/etc/aliases`` sicher, dass
Root-Mails an eine Mailbox weitergeleitet werden, die tatsächlich gelesen wird,
damit Sie wichtige Systemmeldungen und Berichte nicht verpassen::

    # Person who should get root’s mail
    root:          sue@cusy.io

Führen Sie nach dieser Änderung ``sudo newaliases`` aus und testen Sie
anschließend, ob die Mails auch tatsächlich ausgeliefert werden, da einige E-
Mail-Provider Mails aus nicht vorhandenen oder nicht routebaren Domain-
Namen ablehnen. Wenn dies der Fall ist, müssen Sie Ihre E-Mail-Konfiguration
anpassen.

Firewalls, sshd und listening Dienste
Firewalls, SSH- und andere Dienste
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Die Standardeinstellungen der Firewall vieler Distributionen lässt eingehende
SSH-Verbindungen zu. Sofern Sie keinen zwingenden Grund für eingehende
SSH-Verbindungen haben, sollten Sie den SSH-Dienst deaktivieren::

    $ sudo systemctl disable sshd.service
    $ sudo systemctl stop sshd.service

Dies hindert Sie nicht daran, vorübergehend den SSH-Dienst zu starten, wenn Sie
ihn benötigen.

Allgemeiner sollte Ihr System keine offenen Ports haben, an denen ein Dienst
auf Anfragen lauscht. Dies schützt Sie besser vor Zero-Day-Exploits.

Automatische Updates oder Benachrichtigungen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wir empfehlen automatische Updates, da die eigenen Abläufe selten besser sind
als diejenigen der jeweiligen Distribution. Zumindest sollten Sie jedoch
automatische Benachrichtigungen über verfügbare Updates aktivieren. Und auch
dann sollten Sie alle ausstehenden Updates so schnell wie möglich anwenden,
auch wenn etwas nicht speziell als *Sicherheitsupdate* markiert ist oder keinen
zugehörigen CVE-Code aufweist. Alle Bugs haben das Potenzial,
Sicherheitslücken zu sein.

Logs beobachten
~~~~~~~~~~~~~~~

Sie sollten ein großes Interesse an dem haben, was auf Ihrem System passiert.
Aus diesem Grund sollten Sie ``logwatch`` installieren und konfigurieren.
Sie sollten sich tägliche Berichte über alle Aktivitäten zusenden lassen, um
informiert zu sein, was auf Ihrem System passiert. Dies wird zwar keinen
dedizierten Angriff verhindern, erhöht aber dennoch die Sicherheit auf Ihrem
System.

Beachten Sie, dass viele ``systemd``-Distributionen nicht mehr automatisch
einen Syslog-Server installieren, so dass Sie ggf. einen eigenen ``rsyslog``-
Server installieren und aktivieren müssen, um sicherzustellen, dass ``/var/log``
nicht leer ist, bevor ``logwatch`` von Nutzen sein kann.

6. Workstation-Backups
======================

Checkliste
----------

* ☐ Richten Sie verschlüsselte Backups auf externen Speichern ein (wesentlich)
* ☐ Verwenden Sie Zero-Knowledge-Backup-Werkzeuge für Remote-Backups
  (wünschenswert)

Überlegungen
------------

Voll verschlüsselte Daten auf externen Speichern
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Es ist praktisch, eine externe Festplatte für Backups zu haben, da man sich
keine Sorgen machen muss über Bandbreite, Upstream etc. Selbstverständlich
muss diese Festplatte ebenfalls über LUKS verschlüsselt werden oder, sollten
Sie ein Backup-Tool (wie *Duplicity* oder *Déjà Dup*) verwenden, die Backups
selbst verschlüsselt werden. Wir empfehlen letzteres mit einem guten zufällig
generierten Passwort zu verwenden und an einem sicheren Ort offline zu
speichern. 

Zusätzlich zu Ihrem *Home*-Verzeichnis, sollten Sie auch die Verzeichnisse 
``/etc`` und ``/var/log`` für forensische Zwecke sichern.

Auch sollten Sie vermeiden, dass Ihr *Home*-Verzeichnis auf einen
unverschlüsselten externen Speicher kopiert wird. Dies mag zwar zunächst als
einfache und schnelle Möglichkeit erscheinen, Ihre Dateien zwischen
verschiedenen Systemen kopieren zu können, aber auch wenn Sie nicht
vergessen, die Daten anschließend wieder zu löschen, so erlaubt es doch
Schnüfflern ggf. an Ihre sensiblen Daten heranzukommen.

Selektive Off-Site-Backups
~~~~~~~~~~~~~~~~~~~~~~~~~~

Off-Site-Backups sind extrem wichtig und sollten von Ihrem Arbeitgeber
bereitgestellt werden. Sie können ein separates *Duplicity*/*Déjà Dup*-Profil
einrichten, das nur die wichtigsten Dateien kopiert und große Datenmengen
vermeidet wie z.B. Browser-Cache, Downloads etc.

Alternativ können Sie auch ein Zero-Knowledge-Backup-Tool verwenden, wie z.B.
*SpiderOak*, das über zusätzliche nützliche Funktionen, wie zum Beispiel die
Synchronisation von Daten zwischen mehreren Systemen und Plattformen, verfügt.

7. Best Practices
=================

Die folgenden Best Practices sind sicher nicht umfassend. Sie versuchen
vielmehr praktische Ratschläge zu geben, die unseres Erachtens eine
tragfähige Balance zwischen Sicherheit und Benutzerfreundlichkeit halten.

Web-Browser
-----------

Es ist keine Frage, dass Web-Browser die Software mit der größten und am
stärksten exponierten Angriffsfläche auf Ihrem System sind. Sie sind speziell
geschrieben, um beliebige Dateien herunterzuladen und nicht 
vertrauenswürdigen, häufig feindlichen Code auszuführen. Browser versuchen,
Sie von dieser Gefahr durch verschiedene Mechanismen wie Sandboxes und
Code Sanitization zu schützen, aber dies kann keinen 100%igen Schutz bieten.
Sie sollten lernen, dass das Browsen von Websites die unsicherste Aktivität ist,
die Sie ausführen können.

Es gibt nun mehrere Möglichkeiten, wie Sie die Auswirkungen kompromittierter
Browser reduzieren können, aber eine wirklich effektive Lösung erfordert
erhebliche Veränderungen in der Art und Weise, wie Sie auf Ihrer Workstation
arbeiten.

#. Verwenden Sie zwei verschiedene Browser (wesentlich)

   Dies ist die einfachste Möglichkeit, bietet jedoch auch nur einen geringen
   Schutz. Nicht alle Angriffe auf einen Browser führen zu einem vollen
   ungehinderten Zugang zu Ihrem System – manchmal sind sie beschränkt
   auf den lokalen Browser-Storage, die aktive Sitzung anderer Tabs etc. Die
   Verwendung von zwei verschiedenen Browsern, einen für die Arbeit und den
   anderen für alles andere, verringert ein wenig die Auswirkungen
   kompromittierter Browser, stört jedoch auch durch erhöhten
   Speicherverbrauch.

   #. Firefox für die Arbeit und vertrauenswürdige Websites

      Firefox lässt sich für die Arbeit durch zusätzliche Add-ons noch weiter
      absichern:

      NoScript (wesentlich)
          NoScript verhindert das Nachladen von Inhalten mit Ausnahme der in
          einer Whitelist gepflegten Domains. Der Aufwand wäre für den
          Standardbrowser zu hoch, bietet jedoch eine deutlich verbesserten
          Schutz vor Angriffen.
      Privacy Badger (wesentlich)
          EFF’s Privacy Badger verhindert, dass die meisten externen Tracker
          und Werbeplattformen geladen werden. Dies verhindert, dass Ihr
          Browser durch einen dieser Dienste kompromitiert werden kann
          (diese werden häufig genutzt um schnell tausende von Systemen zu
          infizieren.)
      HTTPS Everywhere (wesentlich)
          Dieses von der EFF entwickelte Add-on sorgt dafür, dass auf die
          meisten Ihrer Websites über eine sichere Verbindung zugegriffen
          wird, auch wenn ein Link als Protokoll ``http://`` angibt. Dies hilft um
          eine Reihe von Angriffen zu vermeiden wie z.B. `SSL-strip
          <http://www.thoughtcrime.org/software/sslstrip/>`_.
      Certificate Patrol (wünschenswert)
          Dieses Tool warnt Sie, wenn sich das TLS-Zertifikat der Website, auf
          die Sie gerade zugreifen, geändert hat oder demnächst ändert – z.B.,
          wenn sich das Verfallsdatum nähert oder eine andere
          Zertifizierungsstelle verwendet wird. Es alarmiert Sie bei dem Versuch
          einer Man-in-the-Middle-Attacke, aber erzeugt auch eine Menge
          Fehlalarme.

      Sie sollten als Standardbrowser für das Öffnen von Links Firefox als
      Standard-Browser verwenden, da NoScript das Nachladen oder
      Ausführen von Inhalten meist zuverlässig verhindert.

   #. Chrome/Chromium für alles andere

      Die Chromium-Entwickler haben ihrem Browser vor Firefox viele nette
      Sicherheits-Features hinzugefügt wie Seccomp-Sandkästen, Kernel-
      User-Namespaces usw., die als zusätzliche Isolationsschicht zwischen
      den von Ihnen besuchten Websites und dem Rest Ihres Systems wirken.
      Chromium ist das Upstream-Open-Source-Projekt, und Chrome ist
      Googles darauf basierender proprietärer binärer Build (sie sollten
      Chrome also nicht für Aufgaben einsetzen, von denen Google nichts wissen
      sollte).

      Wir empfehlen, dass Sie auch in Chrome/Chromium die Erweiterungen
      *Privacy Badger* und *HTTPS Everywhere* installieren. Zudem sollten
      Sie ihm ein deutlich anderes Theme geben als Firefox um Ihnen
      anzuzeigen, dass dies nicht der Browser für vertrauenswürdige Sites
      ist.

#. Verwenden Sie zwei verschiedene Browser, davon einen innerhalb einer
   dedizierten VM (wünschenswert)

   Dies ist eine ähnliche Empfehlung wie oben, erfordert jedoch einen
   zusätzlichen Schritt zur Ausführung des anderen Browsers. Die
   dedizierte VM sollte über ein schnelles Protokoll zugreifen und den
   Austausch über die Zwischenablage ermöglichen. Dies erlaubt eine
   deutlich bessre Isolation zwischen nicht vertrauenswürdigen
   Websites und dem Rest Ihrer Arbeitsumgebung.

   Dies erfordert jedoch einen deutlich erhöhten Aufwand, da nun auch
   die VM gepflegt werden muss. Zudem wird deutlich mehr RAM und
   schnelle Prozessoren erwartet, um die erhöhte Last zu bewältigen.

#. Volle Trennung der Arbeitsumgebung durch Virtualisierung (paranoid)

   Siehe hierzu das `Qubes-OS`_-Projekt, das über eine Kapselung der
   Anwendungen in separate, komplett isolierte VMs eine
   hochsichere Umgebung schaffen möchte.

Passwort-Manager
----------------

Checkliste
~~~~~~~~~~

* ☐ Verwenden Sie einen Passwort-Manager (wesentlich)
* ☐ Verwenden Sie einzigartige Passwörter auf unabhängigen Websites
  (wesentlich)
* ☐ Verwenden Sie einen Passwort-Manager, der Team-Sharing unterstützt
  (wünschenswert)
* ☐ Verwenden Sie einen separaten Passwort-Manager für Website-Konten
  (wünschenswert)

Überlegungen
~~~~~~~~~~~~

Die Verwendung von guten, einzigartigen Passwörtern ist eine entscheidende
Voraussetzung für jedes Team-Mitglied. Laufend werden Credentials gestohlen –
entweder über infizierte Computer, über gestohlene Datenbank-Dumps, Remote-
Site-Exploits oder fast beliebige andere Szenarien. Um den Schaden in einem
solchen Fall gering zu halten, sollten Anmeldeinformationen niemals für andere
Anwendungen wiederverwendet werden.

In-Browser-Passwort-Manager
```````````````````````````

Jeder Browser hat einen Mechanismus um Passwörter zu speichern, der ziemlich
sicher ist. Diese Daten können mit einem vom jeweiligen Hersteller
bereitgestellten Cloud-Storage synchronisiert werden, wobei die Daten mit einem
vom Benutzer bereitgestellten Kennwort verschlüsselt werden. Dieser
Mechanismus hat jedoch erhebliche Nachteile:

* Er funktioniert nicht über unterschiedliche Browser hinweg
* Er bietet keine Möglichkeit, Anmeldeinformationen mit den Teammitgliedern zu
  teilen

Es gibt mehrere freie oder billige Passwort-Manager, die in mehrere Browsern gut
integriert sind und auch auf verschiedenen Plattformen arbeiten. Zudem bieten
Sie Gruppenaustausch (in der Regel jedoch als kostenpflichtigen Service).

Standalone-Password-Manager
```````````````````````````

Einer der größten Nachteile von Passwort-Managern ist, dass Sie mit
Browserintegration daherkommen und damit als Teil einer Anwendung, die
höchstwahrscheinlich von Eindringlingen angegriffen wird. Daher sollten Sie
wählen zwischen zwei verschiedenen Passwort-Managern – einem für Websites,
der in Ihren Browser integriert ist, und einen, der als eigenständige Anwendung
läuft. Letzterer kann verwendet werden, um hochsensible Anmeldeinformationen
zu speichern wie Root-und Datenbank-Passwörter, shell account credentials
usw.

Dabei kann ein Werkzeug nützlich sein, um z.B. die folgenden Passwörter mit
den anderen Teammitgliedern zu teilen:

* Server Root-Passwörter
* LOM-Kennwörter
* Datenbank-Admin-Passwörter
* Bootloader-Passwörter
* etc. 

Folgende Tools können Ihnen dabei helfen:

`KeePassX <https://keepassx.org/>`_
    In Version 2 wurde das Team-Sharing deutlich verbessert
`Pass <http://www.passwordstore.org/>`_
    Es nutzt Textdateien und PGP zur Integration in git
`Django-Pstore <https://pypi.python.org/pypi/django-pstore>`_
    GPG wird verwendet um Anmeldeinformationen zwischen Administratoren zu
    teilen
`Hiera-Eyaml <https://github.com/TomPoulton/hiera-eyaml>`_
    Wenn Sie bereits Puppet für Ihre Infrastruktur verwenden, kann dies eine
    praktische Möglichkeit sein, um Ihre Server/Service-Credentials im
    verschlüsselten Hiera-Datenspeichers zu speichern

Sichern der SSH- und PGP-Schlüssel
----------------------------------

Persönliche Schlüssel einschließlich SSH- und PGP-Schlüssel, werden die
schützenswertesten Objekte auf der Workstation sein – etwas, das für Angreifer
von höchstem Interesse sein dürfte, da es ihnen weiter erlauben würde, Ihre
Infrastruktur anzugreifen oder Ihre Identität anzunehmen. Daher sollten Sie
zusätzliche Maßnahmen ergreifen um sicherzustellen, dass Ihre privaten
Schlüssel gut gegen Diebstahl geschützt sind.

Checkliste
~~~~~~~~~~

* ☐ Starke Passwörter werden verwendet um Ihre privaten Schlüssel zu schützen
  (wesentlich)
* ☐ Der PGP-Master-Schlüssel wird auf einem Wechselspeicher gespeichert
  (wünschenswert)
* ☐ Unterschlüssel zum Authentifizieren, Signieren und Verschlüsseln werden auf
  einer Smartcard gespeichert (wünschenswert)
* ☐ SSH ist so konfiguriert, dass PGP-Auth-Schlüssel als SSH-private-key
  verwendet werden (wünschenswert)

Überlegungen
~~~~~~~~~~~~

Der beste Weg, einen Diebstahl privater Schlüssel zu verhindern ist, ihn auf
einer Smartcard zu speichern und ihn niemals auf eine Workstation zu kopieren.
Es gibt mehrere Hersteller, die OpenPGP-fähige Geräte anbieten:

`Kernel Concepts <http://shop.kernelconcepts.de/>`_
    bieten sowohl OpenPGP-kompatible Smartcards als auch ein USB-
    Kartenleser, falls Sie eines benötigen
`Yubikey NEO <https://www.yubico.com/products/yubikey-hardware/yubikey-neo/>`_
    bietet OpenPGP-Smartcard-Funktionalität neben vielen coolen Features
    (U2F, PIV, HOTP, etc.)

Es ist auch wichtig, dass der Master-PGP-Schlüssel nicht auf der Workstation
gespeichert wird und nur Subkeys verwendet werden. Der Hauptschlüssel wird
nur dann benötigt, wenn Schlüssel anderer Personen signiert oder neue
Unterschlüssel erstellt werden sollen – Operationen, die nicht sehr häufig
vorkommen. Wie ein Hauptschlüssel auf dem Wechselspeicher erstellt und
Unterschlüssel erstellt werden ist gut in `Using OpenPGP subkeys in Debian
development <https://wiki.debian.org/Subkeys>`_ beschrieben.

Anschließend sollten Sie dann Ihren GnuPG-Agenten als SSH-Agenten
konfigurieren und den Smartcard-basierten-PGP-Auth-Schlüssel als privaten
SSH-Schlüssel verwenden. In `Wie konfiguriere ich eine GPG-Smartcard zur
SSH-Authentifizierung?
<wie-konfiguriere-ich-eine-gpg-smartcard-zur-ssh-authentifizierung>`_ finden
Sie eine detaillierte Anleitung.

Hibernate
---------

Bei *suspend* verbleiben die Inhalte des RAM auf den Speicherchips und
können von Angreifern gelesen werden (Cold Boot Attack). Wenn Sie also für
längere Zeit Ihre Workstation verlassen wie z.B. am Ende des Arbeitstages,
empfiehlt es sich, die Maschine herunterzufahren oder in den Ruhezustand zu
wechseln (hibernate).

SELinux konfigurieren
---------------------

Wenn Sie eine Distribution verwenden, die mit SELinux geliefert wird, machen
wir einige Empfehlungen, um die Sicherheit Ihres Arbeitsplatzes zu erhöhen.

Checkliste
~~~~~~~~~~

* ☐ Stellen Sie sicher, dass SELinux zwingend auf Ihrer Workstation installiert
  ist (wesentlich)
* ☐ Nur nach Überprüfung ``audit2allow -M`` zustimmen (wesentlich)
* ☐ Nie ``setenforce 0`` verwenden (wünschenswert)
* ☐ Ändern Sie Ihr Konto in SELinux User (wünschenswert)

Überlegungen
~~~~~~~~~~~~

SELinux ist eine Mandatory Access Control (MAC)-Erweiterung der POSIX-
Berechtigungen. Es ist ausgereift und robust. Dennoch empfehlen viele
Sysadmins bis heute, »es einfach abzuschalten«.

Davon abgesehen hat SELinux nur begrenzte Sicherheitsvorteile auf einer
Workstation, da die meisten Anwendungen von Ihnen als Benutzer ausgeführt
werden wird und daher uneingeschränkt laufen werden. Dennoch kann es
voraussichtlich verhindern, dass ein Angreifer die errungenen Privilegien
eskalieren und Root-Level-Zugriff über einen verwundbaren Daemon Service
gewinnen kann.

Unsere Empfehlung ist, sich darauf zu velassen und es zwingend anzulassen.

Nie ``setenforce 0`` verwenden
``````````````````````````````

Zwar mag es verlockend sein, ``setenforce 0`` zu verwenden um den
Freigabemodus von SELinux zu verlassen, aber das sollten Sie tunlichst
vermeiden, da hierdurch SELinux für das gesamte System im Wesentlichen
abgeschaltet wird. Meist wollen Sie hingegen nur eine bestimmte Anwendung
oder einen Daemon ausnehmen.

Statt ``setenforce 0`` sollten Sie also vielmehr ``semanage permissive -a
[somedomain_t]`` verwenden um eine bestimmte Domäne freizugeben. Um
nun diejenige Domäne herauszufinden, welche Probleme verursacht, wählen
Sie ``ausearch``::

    ausearch -ts recent -m avc

Anschließend suchen Sie nach ``scontext=`` und halten Ausschau nach
einer Zeile mit (Quelle SELinux Kontext), etwa::

    scontext=staff_u:staff_r:gpg_pinentry_t:s0-s0:c0.c1023
                             ^^^^^^^^^^^^^^

Dies zeigt Ihnen, dass die Domäne ``gpg_pinentry_t`` verweigert wird, so
dass Sie diese Anwendung freigeben wollen mit::

    semanage permissive -a gpg_pinentry_t

Dies ermöglicht Ihnen, die Anwendung zu verwenden und den Rest der AVCs
zu sammeln, die Sie dann zusammen mit ``audit2allow`` verwenden können,
um eine lokale Richtlinie zu schreiben. Sobald dies geschehen ist und keine
neuen AVC-Denials entdeckt werden, können Sie diese Domäne aus den
Freigaben entfernen mit::

    semanage permissive -d gpg_pinentry_t

Verwenden Sie Ihre Workstation als SELinux-Rolle ``staff_r``
````````````````````````````````````````````````````````````

SELinux kommt mit einer nativen Implementierung von Rollen, die bestimmte
Privilegien gewähren oder verbieten, basierend auf der Rolle, die dem
Benutzerkonto zugeordnet ist. Als Administrator sollten Sie die Rolle ``staff_r``
verwenden, die Ihnen den Zugriff auf viele Konfigurations- und
sicherheitsrelevante Dateien beschränkt, bis Sie zum ersten Mal ``sudo``
aufrufen.

Üblicherweise werden Konten erstellt als ``unconfined_r`` und die meisten
Anwendungen werden ohne Einschränkungen ausgeführt. Um nun den Account
der ``staff_r``-Rolle zuzuordnen, führen Sie den folgenden Befehl aus::

    usermod -Z staff_u [username]

Sie sollten sich ab- und wieder anmelden, um die neue Rolle zu erhalten. Wenn
Sie nun ``id -Z`` aufrufen werden Sie folgende Ausgabe erhalten::

    staff_u:staff_r:staff_t:s0-s0:c0.c1023

Beim Aufruf von ``sudo`` sollten Sie ein zusätzliches Flag hinzuzufügen, um
SELinux mitzuteilen, dass die *Sysadmin*-Rolle angenommen werden soll. Der
Befehl hierzu ist::

    sudo -i -r sysadm_r

``id -Z`` zeigt nun::

    staff_u:sysadm_r:sysadm_t:s0-s0:c0.c1023

.. warning::
   Sie sollten vertraut sein mit ``ausearch`` und ``audit2allow``, bevor Sie
   diesen Schritt machen, da einige Ihrer Anwendungen möglicherweise nicht
   mehr funktionieren werden, wenn Sie als Rolle ``staff_r`` laufen. Dies betrifft zum Beispiel die folgenden gängigen Anwendungen:

   * Chrome/Chromium
   * Skype
   * VirtualBox

Weiterführende Literatur
========================

* `Fedora Security Guide <https://docs.fedoraproject.org/en-US/Fedora/19/html/Security_Guide/index.html>`_
* `CESG Ubuntu Security Guide <https://www.cesg.gov.uk/guidance/end-user-devices-security-guidance-ubuntu-1404-lts>`_
* `Debian Security Manual <https://www.debian.org/doc/manuals/securing-debian-howto/index.en.html>`_
* `Arch Linux Security Wiki <https://wiki.archlinux.org/index.php/Security>`_
* `Mac OSX Security <https://www.apple.com/support/security/guides/>`_

Lizenz
======

Diese Arbeit ist lizenziert unter der `Creative Commons Attribution-ShareAlike
4.0 International License <http://creativecommons.org/licenses/by-sa/4.0/>`_.

