**name**
ue_storchenwetter

**Description**
Ein R-Skript zum einlesen und verarbeiten von Daten über das Brutverhalten von Störchen in Wesenberg (Meckl.Vorp.) und Klimadaten einer Station in Waren (Müritz)

**Usage**
Die Funktion muss beim ersten Durchlauf entweder "Chunk" für "Chunk" (Achtung Markdown-Dokument) ausgeführt werden oder komplett ausgeführt werden (CHUNKS->run all ), um die entsprechenden Variablen im Arbeitsspeicher (Workspace) zu erzeugen. 
!!!Vor der Ausführung Arbeitspfade anpassen (setwd())!!!

**Input**
Wetterdaten -> storch_wetterdaten.txt 
Objektname: wetter
Quelle: https://www.ncdc.noaa.gov/cgi-bin/res40.pl?page=gsod.html
Strochdaten bei Wesenberg (Mecklenburg-Vorpommern) -> storchdaten_1.csv
Objektname: pop
Quelle: Eigene Recherche M. Otto Juli 2013

**Output**
verschieden Diagramme und Konsolenausgaben (Mittelwert etc.)

**author**
C. V. 
M. O. 

**Last modification** 

29.04.2013 - Kleine Aenderungen und Umstellung auf Markdown-Format für Github
22.11.2013 - Korrektur einiger Fehler und Umstrukturierung (C.Vick)
19.11.2013 - Umstrukturierung und Dokumentation (M.Otto)

**TO DO**
Test durch Projektteilnehmer
Ausgabe der vorprozessierten Daten in eine Datei 

====================================================================================


### Setup R Umgebung
set working directory 
```{r}
setwd("/users/blahblahblah/documents/storchtraining")
```

```{r}
getwd() 
```

```{r}
wetter <- read.table (file="storch_wetterdaten.txt", header=TRUE)
```

einlesen der Populationsdaten
```{r}
pop<-read.table(file="storchdaten_1.csv", header=TRUE, sep=";", dec=",", fill=TRUE)
```

Übersicht über die Strukturierung der Klimadaten
anzeigen des Kopfes eines Datenblattes: Spalten mit Namen und die ersten 6 Einträge(Zeilen)

```{r}
head(wetter)
```

anzeigen der Struktur des Objekts 

```{r}
str(wetter)
```

Umformatieren der Daten fuer besseres Handling 

Namen der Spalten ändern für bessere Handhabbarkeit? 'names(wetter)' zeigt die derzeitigen Namen, damit wir keine Probleme mit Groß- und Kleinschreibung bekommen, wird alles klein geschrieben der folgenede Befehl wird die names(wetter) überschreiben!

```{r}
names(wetter) <- c ("date","temp","dewp","slp","stp","wdsp","mxspd","tmax","tmin","prcp")
```
The database is attached to the R search path

```{r}
attach(wetter)
```

So sieht eure Tabelle jetzt aus; mit 'date', 'temp', usw.
```{r}
str(wetter)
```
Ihr könnt die einzelnen Spalten aufrufen oder mit dem Zuordnungspfeil 
auf neue Variablen übertragen
bsp.:
```{r}
temp_dewp <- cbind(temp,dewp)
```
'temp_dewp' ist jetzt eine Matrix mit zwei Spalten (cbind=column-bind) namens temp und dewp und den jeweiligen Einträgen. So sehen die aus:

```{r}
head(temp_dewp)
```

rm()' für ReMove aus dem Arbeitsspeicher oder Workspace (was ist ein Arbeitsspeicher? siehe: http://de.wikipedia.org/wiki/Arbeitsspeicher) 
```{r}
rm(temp_dewp) 
```
entfernt 'temp_dewp' (schau mal im Reiter Workspace sollte nun nicht mehr aufgelistet sein) 

Ok, weiter gehts:
```{r}
str(date)
```


```{r}
wetter$date <- strptime(date, format="%Y%m%d")
```

wir haben nun Grundlegendes geändert, deswegen 'attach()' zum überschreiben der "alten" Variablennamen im Arbeitsspeicher
```{r}
attach(wetter)
```

```{r}
str(date) 
```

Zusammenfassen der Daten, erster Überblick über Datenstruktur/Qualität mit:
```{r}
summary(wetter)
```

```{r}
temp[4]
```

indiziere wo dewp genau gleich 9999.9" und schreibe NA rein.

```{r}
wetter$dewp [dewp==9999.9] <- NA

wetter$stp [stp==9999.9] <- NA

wetter$wdsp [wdsp==999.9] <- NA
wetter$mxspd [mxspd==999.9] <- NA

wetter$tmax [tmax==999.9] <- NA

wetter$prcp [prcp==99.99] <- NA
```



```{r}
plot(date,wetter$prcp, xlab="Zeit [Jahre]", ylab="Niederschlag [inch/Tag]")
```


zeigt, dass ab einer bestimmten Stelle die Aufzeichnungen gestoppt wurden und die Werte auf 0 gesetzt worden sind.
Vermutung: am 01.August 2003 wurde die Niederschlagsmessung eingestellt.
'0' ist aber ein normaler Wert ... Wir brauchen 'NA'. Deswegen werden Werte ab dem 01.08.2003 auf NA gesetzt:

```{r}
wetter$prcp [date>=strptime(20030801,format="%Y%m%d")] <- NA
```

alle Stellen in $prcp, für die gilt, dass das Datum größer-gleich als 01.08.2003 ist, werden auf 'NA' geändert

und chic?
```{r}
summary(wetter)
```

Naja 3597 mal "NA" bei prcp ist nicht super, soll uns aber auch nicht weiter interessieren

Wir haben Grundlegendes geändert, deswegen 'attach()' zum überschreiben der "alten" Variablennamen im Arbeitsspeicher
```{r}
attach(wetter)
```

nochmal summary()

```{r}
summary(wetter$temp)
```

Obwohl wir eine Station in Waren an der Müritz haben ist der Maximalwert der Temperatur der letzten 20Jahre bei **83.4 C°**???
Die Daten dieser Station haben wir von einer us-amerikanischen Einrichtung (GSOD).
http://www.ncdc.noaa.gov/cgi-bin/res40.pl
Leider werden dort sehr viele Größen nicht SI-konform benutzt (siehe Webseite).

Die Temperaturen sind in Fahrenheit angegeben, die Windgeschwindigkeit in Knoten und der Niederschlag in inch Säule pro squareinch (WHAT?).
Wir verwenden die Celciusskala, m/s für Geschwindigkeiten 
und mm/m² als Säulenhöhe Niederschlag (genau!) 
Macht aber alles nichts wir haben ja R und hier lernen wir gleich mal wie man eigene 
Funktionen in R schreibt!

**EIGENE FUNKTION IN R SCHREIBEN**
Das Umrechnen in R ist recht simpel
und das geht so:
'Name der Funktion' <- function('Argument'){
        'Was mit dem Argument gemacht wird'
} Ende

Für unser Beispiel: Umrechnung von Fahrenheit in Celsius
```{r}
f_in_c <- function(x){
  (x-32)*5/9 # das ist die Umrechnungsformel 
}
```


Anwendung auf die jeweiligen Spalten

```{r}
wetter$temp <- f_in_c (temp) # wieder wird überschrieben
wetter$dewp <- f_in_c (dewp)
wetter$tmax <- f_in_c (tmax)
wetter$tmin <- f_in_c (tmin)
```

Anwendung auf Umrechnung von Knoten in m/s
```{r}
kn_in_ms<-function(x){
  x*0.51444 
}
```

Anwendung auf die jeweiligen Spalten in unserem Datenframe

```{r}
wetter$wdsp <- kn_in_ms (wdsp)
wetter$mxspd <- kn_in_ms (mxspd)
```

Anwendung auf die Umrechnung von inch in mm
```{r}
inch_in_mm <- function(x){
  x*25.4 
}
```

Anwendung auf die jeweiligen Spalten

```{r}
wetter$prcp <- inch_in_mm(prcp)
```

Wir haben  Grundlegendes geändert, deswegen 'attach()' zum überschreiben der "alten" 
Variablennamen im Arbeitsspeicher

```{r}
attach(wetter)
```
<<<<<<< HEAD
Speichern von "wetter" auf der Festplatte im Arbeitsverzeichnis (wd)



```{r}
save(wetter, file="wetter.saved")
```


### 2. Analyse der Daten (Datenv(V)erabeitung)     

2.1 mittels Methoden der beschreibenden Statistik 
Mittelwert:

```{r}
mean(temp)
```
Ihr könnt auf jede einzelne Spalte 'mean()' anwenden, oder:
für Fortgeschrittene:
apply-Funktionen wenden eine bestimmte Funktion auf eine Liste oder Tabelle an, so wird Zeit gespart und Schreibplatz
```{r}
sapply(wetter[,2:10],mean, na.rm=TRUE)
```
Auf die Spalten 2 bis 10 wird die Funktion 'mean' angewendet. NA werden entfernt mit Schlüsselwort/Argument 'na.rm = TRUE').

weitere Parameter um die Daten statistisch zu "beschreiben":
Median (ein sog. Lageparameter siehe http://de.wikipedia.org/wiki/Median)
```{r}
median(temp)
```

Maximalwert (schon klar oder ;-) )
```{r}
max(temp)
```

no comment
```{r}
min(temp)
```

5 % Quantil 
```{r}
quantile(temp,0.05)
```
95% Quantil
```{r}
quantile(temp,0.95)
```
eine andere Definition des Medians
```{r}
quantile(temp,0.5)==median(temp)
```
mehr zu Quatilen unter  http://de.wikipedia.org/wiki/Quantil

Varianz (Wie sark shwankt die Temperatur im Mittel (zum Quadrat)?)
```{r}
var(temp)
```
Standardabweichung (gleiche Aussage wie die Varianz nur einheitentreu 
es gibt ja kein "°C zum Qudrat" daher Wurzel aus Var)
```{r}
sd(temp)
```
**Übung:** Was ergibt sd(temp)*sd(temp)?

Es gibt keine vorinstallierte Funktion für den Modus. 
Da er aber durchaus benutzt wird (auch von uns) ist mittlerweile eine 
Funktion von Usern  geschrieben worden.
Wir haben bisher nur Funktionen aus den Paketen 'base', 'stats' und 'graphics' benutzt
es gibt aber noch viele weitere, einige mehr verbreitet, andere weniger
Modus aus dem bereits geladenen Paket "modeest"
```{r}
mfv(temp)
```

### Erste Diagramme (OUTPUT)

**Histogramm**
Was ein Histogramm macht, am besten selber nachschauen!
mit ?hist könnt ihr noch weitere Argumente sehen, die das Histogramm noch 
schicker machen können. Im Allgemeinen ist das 'graphics' package dafür ausgelegt den plot
nach und nach zu professionalisieren mit befehlen wie 'lines' und 'points' und 'legend' usw.
! library(help = "graphics") 
**Übung: Bitte mal selbständig herrausfinden, wie man die Achsen richtig beschrifftet**
```{r}
hist(temp, xlab="Temperatur in ºC", ylab="Häufigkeit",main="Mittlere Temperatur in Waren Müritz")
```

Boxplot
Was ein Boxplot macht, am besten selber nachschauen!
**Übung: Bitte mal selbständig herrausfinden, wie man die Achsen richtig beschrifftet**
```{r}
boxplot(temp)
```

Zeitreihe im Diagramm
**Übung: Bitte mal selbständig herrausfinden, wie man die Achsen richtig beschrifftet**
```{r}
plot(date,temp,type="l")
```

Eine ein wenig unübersichtliche Zeitreihe wenn ihr "ranzoomen" wollt, müsst ihr 'date' und 'temp' mit eckigen Klammern einengen. **Aber Achtung! müssen gleich lang sein!**

**Subset von Daten:**
```{r}
plot(date[date>=strptime(20070101,format="%Y%m%d")& date<=strptime(20071231,format="%Y%m%d")],temp[date>=strptime(20070101,format="%Y%m%d")& date<=strptime(20071231,format="%Y%m%d")],type="l", xlab="Monate (in 2007)", ylab="Temperatur [°C]",main="Jahresübersicht Waren(Müritz) 2007")
```
Hier habe ich das datum zwischen 01.01. und 31.12. 2007 genommen
Sieht auf den ersten Blick kryptisch aus, aber im Grunde ist es nur eine Abfrage, ob nach dem 01.01.2007 (date>=) und (&) vor dem 31.12.2007 (date<=). und zwar jeweils bei date und temp (damit date und temp nach der Abfrage gleich lang bleiben)

**Erweiterte Indizierung**
Für dieses zeitliche "ranzoomen" könnt ihr am besten eigene Variablen kreieren, 
mit denen ihr dann die anderen Spalten genauso indizieren könnt
```{r}
tage2007<-date>=strptime(20070101,format="%Y%m%d")& date<=strptime(20071231,format="%Y%m%d")
```

genau das selbe wie oben
```{r}
str(tage2007) 
```
Wie ihr seht ist das jetzt ein "logical"-Vektor ('logi'), Besonderheit: er zeigt nur bei 2007 'TRUE'. 

Wenn ihr jetzt temp[tage2007] nehmt, habt ihr nur die werte von 2007
bsp.:

indiziert die temp auf das Jahr 2007 und bildet dann den Mittelwert
```{r}
mean(temp[tage2007]) 
```

Die 19zeilige Tafel der Storchenpopulationsentwicklung ist ja schon eingelesen
Das Zeitformat der Strochdaten müssen aber noch R-konform umformatiert werden (wie auch bei den wetterdaten)
```{r}
head(pop)
str(pop)
names(pop)<-c("year","arr","doy_a","hatch","doy_h","fledg","lost","dep","doy_d")
pop$arr<-strptime(pop$arr,format="%d.%m.%Y")
pop$hatch<-strptime(pop$hatch,format="%d.%m.%Y")
pop$dep<-strptime(pop$dep,format="%d.%m.%Y")
attach(pop)
str(pop)
```
So schnell kanns manchmal gehen :-)


### 2.2 Datenanalyse mit Methoden der explorativen Statistik (Wo gibt es Muster, Trends, Auffälligkeiten)

Frage die von Interesse sein könnte:
z.B. Welchen Einfluss hat das Klima auf die Storchpopulation?
 
Technisches Problem 
wir haben 19 Datenzeilen in 'pop'. die 'plot(x,y)'Funktion nimmt aber immer gleich viele x und y werte.
Wir müssen also unsere Wetterdaten (6765 Zeilen) auf die jeweiligen Jahre "zusammenkürzen". 
Wir brauchen pro Jahr einen Wert.
Wie kann aber ein ganzen Jahr auf einen Wert gekürzt werden? statistische Maße!

Wenn wir zB die Jungtiere betrachten brauchen wir nicht das ganze Jahr;
hier z.B. die Brutzeit und die Nistzeit: Wir nehmen der Einfachheit mal an, dass Störche durchschnittlich 33 Tage brüten und dann 61 Tage Nesthocker sind. (wikipedia)
```{r}
hatchtime<-as.difftime(33,format="%d",units="days")
nesttime<-as.difftime(61,format="%d",units="days")
```

Minimaltemperaturen während des Brütens:

Minimum der Minimaltemperaturen zwischen dem Beginn des Brütens und dem Ende der Brutzeit für das erste Jahr
Hypothese: die besonders kalten Nächte erschweren den Erfolg des Ausbrütens

```{r}
mintemp_hatch<-min(tmin[date>=hatch[1]& date<=(hatch[1]+hatchtime)])
```

Minimum der Minimaltemperaturen zwischen dem Beginn des Brütens und dem Ende der Brutzeit nun für die fehlenden Jahre über eine sog. **While-Loop (Schleife** Mach es, solange bis ich stop sage, hier: Minimumtemperaturen finden über 19 Jahre in der Brutzeit)

```{r}
i<-1          # laufvariable für die Schleife also ein Zähler für den Rechner
while(i<19) { # beginn schleife - solange i kleiner 19
  i<-i+1      # i "läuft", wird also um 1 erhöht bei jedem Durchgang
              # die Variable mintemp_hatch wird nicht überschrieben, 
              # sondern der neue Wert an den alten angehängt "(c)
              # ombine"oder concatenate ('?c')
mintemp_hatch<-c(mintemp_hatch,min(tmin[date>=hatch[i] & date<=(hatch[i]+hatchtime)]))  
}#ende der Schleife - springt nach oben zu 'while' - ist i immernoch kleiner 19? dann alles nochmal!
```

am Ende haben wir einen mintemp_hatch-Vektor der die Minimaltemperaturen während des Brütens festgehalten hat. **HURRA!!!**

Oder wie wäre es mit Minimaltemperaturen während der Nestlingzeit?

Minimum der Minimaltemperaturen zwischen dem Ende der Brutzeit und dem Ende der Nesthockerzeit
Hypothese: die besoders kalten Nächte erschweren die Aufzucht der Jungvögel

```{r}
mintemp_nest<-min(tmin[date>=(hatch[1]+hatchtime) & date<=(hatch[1]+hatchtime+nesttime)])
i<-1 # wieder die Laufvariable
while(i<19) { # beginn der Schleife
  i<-i+1 # sie läuft
  mintemp_nest<-c(mintemp_nest,min(tmin[date>=(hatch[i]+hatchtime) & date<=(hatch[i]+hatchtime+nesttime)]))
  # siehe oben
}
```
zur Überprüfung der Daten:

```{r}
length(mintemp_hatch)
length(mintemp_nest)
```
Länge 19? dann alles ok!

Scatterplots auch Streudiagramme genannt:
achtet bitte mal auf die Achsenbeschrifftung - so und nicht anders :-) 

```{r}
plot(mintemp_hatch, fledg-lost, xlab="Minimaltemperatur während der Brutzeit [°C]", ylab="Anzahl der überlebenden Jungtiere", main="Beispiel für Korrelation?")
plot(mintemp_nest, fledg-lost, xlab="Minimaltemperatur während der Nistzeit [°C]", ylab="Anzahl der überlebenden Jungtiere", main="Beispiel für Korrelation?")
```

### 2.3.Datenanalyse mit Methoden der schließende Statistik (Was sind meine Erkenntnisse?)

Ok wir kennen die Daten jetzt ganz gut oder?
Gibt es erkennbare Zusammenhänge? Erkennbare Muster im Scatterplot?
Hier kommt die Korrelation zum Einsatz, zur Erinnerung Korrelation is das Verhältnis der
Varianz zwischen den Variablen zur Varianz unabhängig von den Variablen!
Klinkt kompliziert? Macht nichts R rechnet das für euch aus.
Korrelationskoeffizient: ohne Indizierung gibt er NA aus.... man muss die NA werte rausstreichen: '?is.na' 
Dies ist eigentlich Teil der Vorprozessierung wir machen es aber hier der Einfacheit halber!

```{r}
cor(mintemp_hatch[!is.na(mintemp_hatch)],(fledg-lost)[!is.na(mintemp_hatch)])
cor(mintemp_nest[!is.na(mintemp_nest)],(fledg-lost)[!is.na(mintemp_nest)])
```

keine Korrelation oder? Es gab ja auch kein wirklich erkennbares Muster im scatterplot oben.
Der Korrelationsfaktor oder auch Bestimmtheitsmaß sagt auch nochmal: nein zumindest kein klassisch linearer Zusammenhang zwischen den Variablen 
**Aber Achtung Korrelation ist keine Kausalität per se!!!**
Hier mal ein hübsches Beispiel und schlechte Nachrichten fuer kleine Maenner:
http://www.spiegel.de/unispiegel/jobundberuf/10-cm-2000-euro-grosse-maenner-verdienen-mehr-a-296853.html

Zurück zu den Störchen: Vielleicht die Aufenthaltsdauer? Im Zusammenhang mit der durchschnittlichen Temperatur?


```{r}
meantemp_stay<-mean(temp[date>=arr[1] & date<=dep[1]])
```

Mittelwert der Temperaturen, für die gilt, dass Datum größer-gleich Ankunftsdatum UND Datum
kleiner-gleich Abflugdatum wieder eine Schleife:


```{r}
i<-1 # wieder die Laufvariable
while(i<19) { # beginn der Schleife
  i<-i+1 # Laufvariable läuft
  meantemp_stay<-c(meantemp_stay,mean(temp[date>=arr[i] & date<=dep[i]]))
  # siehe oben
}
```

```{r}
length(meantemp_stay)
```
Auch 19?

```{r}
plot(meantemp_stay,dep-arr , xlab="Durchschnittstemperaturen während des Aufenthaltes [°C]", ylab="Aufenthaltsdauer [in Tage]", main="Korrelation?")
```

die NA-Values müssen wieder gestrichen werden durch Indizierung
dep-arr muss als Zahl berechnet werden (as.numeric = als Zahl), sonst ist es eine Zeitangabe, mit der R nicht rechnen kann

```{r}
cor(meantemp_stay[!is.na(meantemp_stay)],as.numeric(dep-arr)[!is.na(meantemp_stay)])
```

**Übung: Was ist der Wert für den Korrelationskoeffizienten? Was bedeutet er?** 
(Hinweis nutze die Funktion print()) **und diskutiert im ISIS-Forum**

ziehen der 'lm()' Linie in dem bestehenden Plot. Geht auch mit 'lines()'
'lm()' modelliert eine Regressionsgerade. 'y~x' heißt y ist abhängig von x

```{r}
abline(lm((dep-arr) ~ meantemp_stay),col="blue")
```

Wir brauchen aber noch die Werte dieser Regression.
Regression ehm??? schau mal unter: http://de.wikipedia.org/wiki/Lineare_Regression

```{r}
co<-coef(lm((dep-arr) ~ meantemp_stay))
```

co ist mein Name der Variable, die folgendes bekommt:
'?coef' ist eine Funktion die Koeffizienten sucht und zwar innerhalb der 'lm()'-funktion
co enthält jetzt also die beiden Koeffizienten der linearen Regression

```{r}
print(co)
```
'text(x,y,text)' schreibt einen Text an die x,y-Koordinaten. 
'cex=' stellt die Schriftgröße ein.

```{r}
text(17.3,129,"lineare Regression",cex=0.8)
text(17.3,127,paste("y =",round(co[1],2),round(co[2],2), "x "),cex=0.8,col="blue")
```

schreibt die Formel für die lineare Regression in den plot. 'round(Zahl,2)' rundet die Zahl auf zwei Nachkommastellen.

Fertig!
**Und was sagen euch die Ergenisse? Diskutiert es im Forum (ISIS). Postet eure plot als png!**

## zum Aufräumen

mit ls() könnt ihr euch euren workspace einmal anschauen. Kram, den ihr nicht mehr braucht, könnt ihr rm() entfernen

rm(list=ls()) # entfernt alles. ALLES!

Und wie wars? 

```{r}
ihr_sagt<-'super! Danke ihr seit einfach die .....'
print(ihr_sagt)
wir_darauf <- 'Schon gut ;-)'
print(wir_darauf)
rm(ihr_sagt,wir_darauf)
```
