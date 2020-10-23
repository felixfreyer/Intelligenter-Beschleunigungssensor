# Intelligenter Beschleunigungssensor
Dec 2nd, 2013

Ziel dieses Projektes ist es die Beschleunigung in 3 Achsen zu messen und numerisch, sowie grafisch auszugeben. Die Beschleunigung ist eine physikalische Größe die in vielen Bereichen der Technik eine wichtige Rolle spielt, sodass sich etliche denkbare Anwendungsszenarien ergeben.

### Hardware
Das Arduino Board, welches für dieses Projekt verwendet wird, arbeitet auf der Basis des Atmega1280. Der große Vorteil des Arduino Boards liegt darin, dass die Sensordaten einfach über die USB Schnittstelle auf dem PC empfangen werden können. Zunächst wird per Software die Kommunikation zwischen Controller und Sensor realisiert, dabei kommt der I²C - Bus zum Einsatz. Diese so empfangen Sensormesswerte werden im nächsten Schritt mit Hilfe der UART - Schnittstelle des Controllers per USB an den PC gesendet. Der schematische Kommunikationsaufbau ist in Abbildung 1 dargestellt.

![](https://github.com/felixfreyer/Intelligenter-Beschleunigungssensor/raw/main/adxl.png)
<br />
**Abbildung 1: ADXL345 Kommunikationsaufbau**


Der ADXL345 ist ein dreiachsiger Beschleunigungssensor, der einen Messbereich von +/- 16 g umfasst. Der Sensor besteht aus einer mikromechanischen Polysiliziumstruktur, welche auf einen Siliziumwafer aufgebracht ist. Das Messprinzip basiert auf der Messungen eines differentiellen Kondensators. Der Sensor und das Arduino Board sind folgendermaßen verschaltet:

![](https://github.com/felixfreyer/Intelligenter-Beschleunigungssensor/raw/main/adxl_arduino.png)
<br />
**Abbildung 2: ADXL345 Verbindung mit Arduino**

Jeweils die Datenleitung (SDA) und die Taktleitung (SCL) des Sensors und der entsprechende Pin der Schnittstelle des Atmega1280 sind miteinander verbunden. Die SDO Leitung wird lediglich für die Kommunikation per SPI verwendet und wird deshalb auf Masse gezogen. Die beiden Interrupt Leitungen INT1 und INT2 werden verwendet und sind deshlab hochohmig geschaltet. Da nur ein Beschleunigungssensor verwendet wird, wird die Chip Select Leitung (CS) zusammen mit der VCC Leitung auf 3,3V gelegt.

### Software
Nachdem nun die Hardware sowie das Messprinzip beschrieben wurde, wird in diesem Kapitel das Lösungskonzept und die softwaretechnische Realisierung erläutert. Das Projekt besteht aus zwei Programmen: Der Firmware des Arduinos, sowie der LabVIEW Applikation am PC. Die Firmware des Arduinos ist bereits im LabVIEW VI Package Manager Addon „LabVIEW Interface for Arduino“ enthalten. Mithilfe dieses Addons kann der Mikrocontroller direkt über LabVIEW programmiert und dessen GPIO-Ports sowie Peripherie angesteuert werden. Nach der Installation des Labview Addons befindet sich die Arduino Firmware unter Windows in folgendem Verzeichnis: C:\Program Files\National Instruments\LabVIEW 2010\vi.lib\LabVIEW Interface for Arduino\Firmware\LVIFA_Base\LVIFA_Base.pde

Bevor der Arduino via LabVIEW angesteuert werden kann muss jedoch zunächst die Firmware mittels der Arduino IDE auf dem Mikrocontroller programmiert werden.

![](https://github.com/felixfreyer/Intelligenter-Beschleunigungssensor/raw/main/adxl_frontend.png)
<br />
**Abbildung 3: LabVIEW VI - Frontend**

Das Frontend verfügt lediglich über ein Bedienelement, nämlich über einen Stop Button. Die restlichen Elemente sind allesamt Anzeigeelemente und dienen zur Visualisierung der Messwerte. Das Frontend ist in Abbildung 3 dargestellt. Der Stop Button links unten im Frontend, blau umrandet, stoppt das laufende Programm und die USB Verbindung zum Arudino Board, zudem werden die Anzeigeelemente zurückgesetzt. Die Legende, schwarz markiert, zeigt welche Kurveder beiden Graphen welcher Achse des Beschleunigungssensor zugeordnet ist. Die Visualisierung der Beschleunigungswerte geschiet in den verbleibenden drei Anzeigeelementen. Zunächst werden die unbearbeiteten Sensordaten in dem Graph dargestellt der grün umrandet ist. In Abbildung 3 kann man bereits erkennen, dass es während der Messung immer wieder zu unerwünschten Peaks kommt. Aus diesem Grund ist eine zusätzliche Verarbeitung der Sensordaten notwendig. Die geglätteten Signale werden im rot markierten Graphen dargestellt. Schließlich wird im letzten Graphen, hier gelb hervorgehoben, eine 3D Grafik aus den Sensordaten erzeugt. Diese gibt die relative Lage zur Startposition des Sensors an. Alle Graphen zeigen die Messwerte in Echtzeit an.

Im Folgenden wird das Blockdiagramm der LabVIEW VI erläutert:

![](https://github.com/felixfreyer/Intelligenter-Beschleunigungssensor/raw/main/adxl_blockdiagramm.png)
<br />
**Abbildung 4: LabVIEW VI - Blockdiagramm**

Grundsätzlich hat das Blockdiagramm ein Top-Down Design. Oben links beginnt die Hardwareinitialisierung (HW-Init), welche sich außerhalb der Haupt-While-Schleife befindet und nur einmal beim Start des Programms ausgeführt wird. Bei der HW-Init wird zunächst die COM-Schnittstelle für den Arduino geöffnet. Dabei werden Baudrate und COM-Schnittstelle ausgewählt sowie I²C Kommandos an den Arduino gesendet um später Sensordaten zu empfangen. In der Sub VI für die Datenerfassung werden die Daten aus dem FIFO des Beschleunigungssensors gelesen und anschließend in einem Array gespeichert. Die empfangenen Daten müssen nun zur Anschaulichkeit für die grafische Darstellung skaliert werden. Dabei entspricht ein Messwert von 255 einem Beschleunigungswert von 1 g. Zur besseren grafischen Darstellung der Beschleunigung mittels 3D Würfel müssen die Messwerte zunächst geglättet werden. Dafür ist eine Mittelwertbildung der letzten 5 Messwerte ausreichend, wie sich in der Praxis gezeigt hat. Des Weiteren ist für die Transformation des 3D Würfels eine Winkelberechnung notwendig.

