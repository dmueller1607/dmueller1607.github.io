---
layout: post
title: Hello World mit AdoptOpenJDK 15, JavaFX 15 und Eclipse IDE
author: Daniel Müller
date: '2020-11-05'
category: guides
tags: javafx
summary: Hello World mit AdoptOpenJDK 15, JavaFX 15 und Eclipse IDE
thumbnail: post2_javafx/logo.png
---

# Hello World mit AdoptOpenJDK 15, JavaFX 15 und Eclipse IDE

Das letzte Mal als ich mit JavaFX gearbeitet habe, ist bereits mehrere Jahre her. Damals verwendete ich das Oracle JDK 1.8, welches bereits die JavaFX Module enthielt. 

Etwas komplizierter gestaltet sich die Verwendung von JavaFX bei neueren Versionen. Sowohl bei neueren Java Versionen von Oracle (seit Java 11), als auch bei sämtlichen Varianten von OpenJDK, wird JavaFX als eigenständiges Modul ausgeliefert und muss deshalb separat in das Java-Projekt eingebunden werden.

Da ich nun wieder in einem Projekt mit JavaFX zu tun habe, wollte ich mir dessen Verwendung in Kombination mit dem **AdoptOpenJDK 15** und der **Eclipse IDE** (Version 2020-09) unter **Windows** ansehen. Ich möchte mit diesem Blog-Post mein Vorgehen beim Einrichten dieser Entwicklungsumgebung dokumentieren und mit euch teilen.

Hier also nochmals meine Konstellation:

* Windows 10 x64
* AdoptOpenJDK 15
* JavaFX 15
* Eclipse 2020-09
* (Scene Builder)

### 1. Installation von AdoptOpenJDK 15

Zunächst begann ich damit, das unter OpenSource-Lizenzen stehende AdoptOpenJDK in der aktuellsten Version (jdk-15.0.1+9) herunterzuladen und auf meinem Windows 10 Rechner (64-bit) zu installieren. 

Downloadlink für AdoptOpenJDK: [https://adoptopenjdk.net/](https://adoptopenjdk.net/)

Der Installations-Wizard begleitet einen durch die Installation. Es empfiehlt sich, auch die *JAVA_HOME* Systemvariable über den Wizard setzen zu lassen.

Alternativ zum JDK 15 lässt sich auch die Long-Term-Support (LTS) Version 11 von AdoptOpenJDK installieren.

### 2. Installation der Eclipse IDE

Nachdem Java installiert war, folgte der Download der aktuellen Eclipse IDE 2020-09. 

Downloadlink für Eclipse: [https://www.eclipse.org/downloads](https://www.eclipse.org/downloads)

Hier kann die Standardinstallation (Eclipse IDE for Java Developers) heruntergeladen werden. Das zip-Archiv kann in einen beliebigen Ordner auf dem Rechner extrahiert werden, z.B. *C:\Program Files\eclipse* (evtl. werden hierfür Administrator-Rechte benötigt). Eclipse lässt sich nun über die gleichnamige *.exe* Datei starten. Zum Starten der IDE wird eine Java Installation benötigt, sie sollte aber automatisch gefunden werden und Eclipse sollte sich ohne Fehlermeldung öffnen.

Hinweis: Aus Erfahrung macht es Sinn, die Eclipse IDE zukünftig als Administrator zu starten (Rechtsklick auf *eclipse.exe* > Eigenschaften > Kompatibilität > Programm als Administrator ausführen).

### 3. Download von JavaFX 15

Das von der Java-Installation entkoppelte JavaFX Modul lässt sich über eine eigene Website herunterladen.

Downloadlink für JavaFX: [https://gluonhq.com/products/javafx/](https://gluonhq.com/products/javafx/)

Neben der LTS Version 11 kann auch das aktuellste Release (momentan Version 15) auf dieser Seite heruntergeladen werden. Ich habe für dieses Tutorial JavaFX Version 15 verwendet. Benötigt habe ich lediglich die *JavaFX Windows x64 SDK*, die sich als zip-Archiv herunterladen lässt. Auch dieses zip-Archiv muss entpackt werden, z.B. nach *C:\Program Files\OpenJFX\javafx-sdk-15.0.1*.

### 4. Installation des JavaFX Plugins für Eclipse

Damit das JavaFX Modul von der Eclipse IDE gefunden und für zukünftige Java-Projekte verwendet werden kann, muss das Eclipse-Plugin *e(fx)clipse* nachinstalliert werden.

Über den Link [https://download.eclipse.org/efxclipse/updates-released/](https://download.eclipse.org/efxclipse/updates-released/) kann die aktuellste Version abgerufen werden (aktuell 3.7.0).

Dieser Link muss nun als neues Repository der Eclipse Software Sites Liste hinzugefügt werden (Menü Help > Install new Software > Add...):

* Name = e(fx)clipse
* Location = [https://download.eclipse.org/efxclipse/updates-released/3.7.0/site/](https://download.eclipse.org/efxclipse/updates-released/3.7.0/site/)

Per Klick auf den Button *Add* wird das Repository der Softwareliste von Eclipse hinzugefügt und es werden zwei Module gelistet: *install* und *single components*. Diese müssen beide ausgewählt und per Klick auf den Button *Next* installiert werden (Lizenzbedingungen und Zertifikat müssen akzeptiert werden). Die Software-Installation dauert dann einen Moment und die IDE muss abschließend neugestartet werden.

Nach erfolgreicher Installation sollte im Auswahlmenü für neue Projekte (Menü File > New > Project...) die Kategorie *JavaFX > JavaFX Project* gelistet sein.

Hinweis: Sollte es bei der Installation zu Problemen kommen, die IDE als Administrator starten und die Installation nochmals versuchen.

### 5. Download von Scene Builder für Windows

Mit dem Scene Builder können UIs im FXML-Format erstellt und bearbeitet werden. Die Software vereinfacht die Erstellung von komplexeren Oberflächen. Mit folgendem Link kann der *Scene Builder for Java 11* heruntergeladen werden. Ein Wizard unterstützt die Installation des Tools.

Link zum Download: [https://gluonhq.com/products/scene-builder/#download](https://gluonhq.com/products/scene-builder/#download)

Hinweis: Der SceneBuilder funktioniert auch mit Java-Installationen, die neuer als Version 11 sind.

### 6. Einrichten von JavaFX und Scene Builder in Eclipse IDE

Wenn die Installation des e(fx)clipse Plugins funktioniert hat, findet sich nun auch ein neuer Menüpunkt *JavaFX* in den IDE-Eigenschaften (Menü Window > Preferences > JavaFX). Dort muss der Pfad zum entpackten JavaFX SDK Verzeichnis angegeben werden:

* JavaFX 11+ SDK = [z.B.] *C:\Program Files\OpenJFX\javafx-sdk-15.0.1\lib*
* Scene Builder Executable = [z.B.] *C:\Program Files\SceneBuilder\SceneBuilder.exe*

Es empfiehlt sich ein abschließender Neustart der Eclipse IDE.

### 7. Anlegen eines neuen JavaFX Projekts

Ist soweit alles installiert, kann ein neues JavaFX Projekt in Eclipse angelegt werden (Menü File > New > Project... > JavaFX > JavaFX Project). 

* Project Name = [z.B.] JavaFXHelloWorld
* JRE = Auswählen des zuvor installierten AdoptOpenJDK 15

Ein neues Projekt wird angelegt, welches einen kleinen Beispielcode für JavaFX enthält. Durch das Hinterlegen des JavaFX Installationspfads im vorherigen Schritt 6 sollte dem Projekt automatisch das Modul *JavaFX SDK* hinzugefügt werden und das Projekt sollte sich fehlerfrei bauen und ausführen lassen.

Falls dies nicht der Fall sein sollte, hat der Verweis auf das JavaFX nicht korrekt funktioniert, es empfiehlt sich, Schritt 8 durchzuführen.

### 8. (optional) Einrichten einer User Library für JavaFX

Das Einrichten einer User Library sollte, wie bereits oben erwähnt, durch den vorherigen Schritt (Hinterlegen des Installationspfades von JavaFX in den Eclipse Eigenschaften) übersprungen werden können. Sollte es Probleme beim Erstellen eines neuen JavaFX Projekts geben (z.B. dass das Projekt die JavaFX Abhängigkeiten nicht findet), dürfte dieser Schritt Abhilfe schaffen. Hierüber wird JavaFX manuell als sogenannte *User Library* in Eclipse eingebunden (was eigentlich automatisch nach Schritt 6 passieren sollte).

Per Eclipse Menü (Window > Preferences > Java > Build Path > User Libraries > New...) kann eine neue User Library für JavaFX hinzugefügt werden, die auf die JARs der JavaFX Installation verweist.

* Name = OpenJFX15

Nach einem Klick auf *OK* müssen der Bibliothek anschließend per Button *Add External JARs* die JAR Dateien, die im JavaFX Pfad (z.B. C:\Program Files\OpenJFX\javafx-sdk-15.0.1\lib) enthalten sind, hinzugefügt werden. 

Dem Java Projekt kann nun diese User Library manuell hinzugefügt werden (Rechtsklick auf das Projekt > Properties > Java Build Path > Modulepath > Add Library... > User Library > Next > OpenJFX15 > Finish). Das Projekt sollte nun die JavaFX Klassen finden und sich bauen lassen.

### 9. HelloWorld mit JavaFX

Abschließend noch ein kleines Beispiel eines JavaFX Projekts. Ein neues Projekt kann per Menü (File > New > Project... > JavaFX > JavaFX Project) angelegt werden (siehe Schritt 7).

Die Datei *module-info.java* kann verwendet werden, um benötigte JavaFX Module zu definieren. Wir benötigen für dieses Beispiel das *javafx.graphics* Modul, wodurch die Definitionsdatei wie folgt aussehen muss:

``````java
// Datei "module-info.java"
// ========================

module JavaFXHelloWorld {
	requires javafx.controls;
	opens application to javafx.graphics;
}
``````

Folgender Quellcode ließt die Java Version aus und zeigt sie in einem neuen Fenster an: 

``````java
// Datei "Main.java"
// =================

package application;
	
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Label;
import javafx.scene.layout.StackPane;
import javafx.stage.Stage;

public class Main extends Application {

    @Override
    public void start(Stage stage) {
        String javaVersion = System.getProperty("java.runtime.version");
        String javafxVersion = System.getProperty("javafx.runtime.version");
        
        Label labelVersions = new Label ("Java version: " + javaVersion + "\n" + 
        								 "JavaFX version: " + javafxVersion);
        
        Scene scene = new Scene (new StackPane(labelVersions), 300, 200);
        stage.setScene(scene);
        stage.show();
    }

    public static void main(String[] args) {
        launch();
    }
}
``````

Die Datei *application.css* kann aus dem erstellen Beispielprojekt gelöscht werden. Das Projekt sollte sich nun starten lassen (über die main Methode der Klasse Main).

### Notizen

* Alternativ zur Datei *module-info.java* können auch die VM Argumente der Run Configuration (Rechtsklick auf die Klasse *Main.java* > Run as > Run Configurations > Main Klasse auswählen > Tab *Arguments* > Abschnitt *VM Arguments*) verwendet werden, um benötigte JavaFX Module zu definieren:

  ``````shell
  --module-path "C:\Program Files\OpenJFX\javafx-sdk-15.0.1\lib" --add-modules=javafx.controls
  ``````