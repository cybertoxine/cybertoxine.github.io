---
layout: post
title:  "Mobilny Farmaceuta spotyka Speeduino."
author: MF
categories: [ energoelektronika, opensoftware, openhardware, sterownik silnika spalinowego, tuning, kogeneracja, agregat prądotwórczy, sport motorowy ]
image: assets/images/wmo.jpg
tags: [sticky]
---


<p style="color:green;">Ostatnia aktualizacja: 13.11.2025</p>

<b>Spis treści:</b>

- [1] Co to jest Speeduino? Kto jest autorem i co mówi licencja na której ten projekt jest dystrybuowany? Link do repo.
- [2] Po co mi Speeduino?
- [3] Co postanowiłem zmienić/uzupełnić w Speeduino i dlaczego?
- [4] Konkretne rozwiązania etapu pierwszego i ich opis (część teoretyczna).
   - [4a] Dodanie modułu zapłonowowego na SiC MOSFETach. 
   - [4b] Dodanie kondycjonera sygnału z czujnika reluktancyjnego położenia wału korbowego. 
   - [4c] ???
- [5] Konkretne rozwiązania etapu drugiego i ich opis (część teoretyczna).
   - [5a] Użycie AVR128DB64 zamiast ATmega2560. 
   - [5b] Wykrywanie spalania stukowego na drodze DSP. 
   - [5c] Prawidłowe wysterowanie benzynowych wtryskiwaczy niskoimpedancyjnych. 
   - [5d] Możliwość udawania fabrycznego ECU po podłączeniu do czytnika OBD.
   - [5e] Współpraca z sensorami oraz możliwość pełnosprawnego wysterowania aktuatorów systemu wtrysku Common Rail.
   - [5f] Dodanie nadrzędnej jednostki sterującej opartej o architekturę RISC-V (np ESP32-C3), interfejs do konfiguracji i diagnozy typu client(przeglądarka internetowa)-serwer(odpalony na RISC-V). Komunikacja Wi-Fi. Na AVRxm kod przepisany w asemblerze, na RISC-V pisany w Pythonie.
- [6] Przebieg realizacji.
- [7] Efekty i wnioski.


<p><span style="border: 2px solid red; border-radius: 10px; background-color: red;color:white">&nbsp; &nbsp;[4a]&nbsp; &nbsp;</span></p>

## Moduł zapłonowy dla jednej cewki:

Dla większej ilości cewek potrzebna wielokrotność poniższego układu.

![walking]({{ site.baseurl }}/assets/images/sicignition.png)

U1, U2 - Niezbyt szybkie transoptory. Szybkość podczas załączania nie jest potrzebna, prąd płynący w uzwojeniu pierwotnym cewki zapłonowej i tak narasta powoli.
Dałem dwie sztuki bo koszt prawie zerowy a mamy większą wydajność prądową. Układ przez to jest pewniejszy - czasem przy częściowym uszkodzeniu bramki MOSFET-a potrafi płynąć niewielki prąd. Poza tym wilgoć itd.

U3 - Szybki transoptor, powoduje momentalne załączenie T2 i prawie natychmiastowe sprowadzenie potencjału bramki MOSFET-a do 0V. Tu szybkość jest ważna bo cewka ma wygenerować stromy pik napięcia po stronie wtórnej.

T1 - MOSFET z węglika krzemu. Podczas przeglądania for na temat speeduino kilka razy natknąłem się na wpisy opisujące problem zakłucania pracy mikrokontrolera przez moduł zapłonowy.
Postanowiłem więc maksymalnie odizolować obwód wysokiego napięcia od obwodów mikrokontrolera. T1 ma malutką pojemność dren-bramka i dren-źródło co skutkuje niewielkim przedostawaniem się zakłuceń ze strony pierwotnej cewki zapłonowej w stronę mikrokontrolera. Dodatkową izolację zapewniają U1, U2 i U3. Tranzystor jest dość drogi (około 20zł) ale ze względu na wspomniane wyżej małe pojemności wewnętrzne, małą rezystancję w stanie załączenia, dopuszczalne bardzo wysokie napięcie pracy, prawie niezmieniające się wraz ze zmianą temperatury parametry, szybkość wyłączania (w przeciwieństwie do IGBT brak "ogona prądowego") i odporność na wysokie temperatury wart swojej ceny biorąc pod uwagę warunki panujące w okolicach sportowo eksploatowanego silnika spalinowego.

T2 - Rozładowuje bramkę T1 przez R6 po zadziałaniu U3.

T3 - Element obwodu ograniczającego prąd płynący przez cewkę. Gdy napięcie na R8 przekroczy 0.6V T3 zaczyna rozładowywać bramkę T1 działając w kontrze do U1 i U2.

D1 - Dioda Shottky z węglika krzemu. Element obwodu ograniczającego piki wysokiego napięcia po stronie wtórnej, zabezpieczającego T1 i cewkę przed uszkodzeniem. Kombinacja dużej szybkości i wysokiego dopuszczalnego napięcia wstecznego.

D2-D8 - Diody Zenera. Odkłada się na nich moc przepięć ze strony pierwotnej. Każda z nich ma napięcie przebicia lawinowego wynoszące 200V więc zabezpieczenie zaczyna działać "miękko" (dzięki R9) przy 600V i "twardo" przy 800V.

D9 - Dioda Zenera. Zabezpieczenie przed pikami napięcia mogącymi pojawić się na bramce T1.

C1 - Niskoimpedancyjny elekrolit 105st. C w obwodzie 12V.

C2 - Dodatkowe "przycięcie" szybkozmiennych sygnałów.

C3 - Stabilizacja napięcia na D2-D8 oraz niwelowanie wpływu indukcyjności obwodu zawierającego diody Zenera. 

C4 - Wiadomo. Na szynie 20V.

R1 - Ograniczenie prądu płynącego przez U3 i bramkę T2.

R2 - Przyspieszenie wyłączania T2.

R3, R4 - Ograniczenie prądu płynącego przez U1 i U2.

R5 - Opór na tyle niewielki, że "przeważa" nad R3 i R4 w momencie zadziałania ograniczenia prądu płynącego przez cewkę.

R6 - Ograniczenie prądu płynącego przez T2. 

R7 - Ograniczenie prądu płynącego przez bramkę T3 w momencie zadziałania ograniczenia prądu cewki.

R8 - Ustala maksymalną wartość prądu cewki. Pełni rolę bezpiecznika w razie uszkodzenia T1.

R9 - Punkt w którym można nawet bez oscyloskopu sprawdzić, czy napięcie w obwodzie pierwotnym cewki nie jest zbyt wysokie.

R10 - Ograniczenie prądu płynącego przez LED-a w U3.

R11 - Ograniczenie prądu płynącego przez LED-y w U1 i U2.

## Przetwornica napięcia do zasilania driverów bramek SiC-MOSFET-ów:

![walking]({{ site.baseurl }}/assets/images/boostt.png)

C1  - Niskoimpedancyjny elektrolityczny.

C2 - Niskoimpedancyjny elektrolityczny. Chociaż chyba lepiej tu tantalowy 25V.

U1 - Dobry, tani, nieco przestarzały już układ scalony przetwornicy. 52KHz, cycle by cycle current limit przy 5A. Tutaj w nietypowej konfiguracji buck-boost, nieopisanej w nocie katalogowej.
Przewymiarowane bo IC jest w stanie "przepompować" 20W mocy w tej topologii i przy tym napięciu zasilania. Zdecydowałem się jednak na ten układ bo modyfikując L1 można łatwo uzyskać szereg wyjść o różnych napięciach. Oprócz 20V potrzebne jest 5V. Prawdopodobnie przyda się też 3.3V. Prawdopodobnie zajdzie potrzeba zasilenia bramek high side N-MOSFET-ów z izolowanego, "pływającego" uzwojenia. Narazie niech tak zostanie (prosto, tanio, wystarczająco dobrze i elastycznie).

T1 - MOSFET z pojemnością bramki tylko 300pF.

D1 - Zaczyna przewodzić gdy górny klucz wbudowany w U1 przestaje przewodzić.

D2, D3 - Zabezpieczenie bramki T1 przed przepięciami.

D4 - Jak D1. Na zmianę przewodzą U1 + T1 i D1 + D4. Kondensator wyjściowy jest ładowany tylko w czasie zanikania pola magnetycznego w dławiku więc łatwo "doczepić" wiele wyjść, jak w flybacku.

D5 - Jakiś LED o znamionowym ciągłym prądzie pracy większym bądź równym 20mA.

R1 - Szeregowo z bramką T1.

R2 - Ograniczenie prądu płynącego przez LED, niewielkie wstępne obciążenie przetwornicy.

R3, R4 - Dzielnik napięcia, ustala napięcie wyjściowe.

L1 - Tu zależy. Gdy ma być zasilany tylko moduł zapłonowy dławik może być mały, stratny i o większej indukcyjności.


<p><span style="border: 2px solid red; border-radius: 10px; background-color: red;color:white">&nbsp; &nbsp;[4b]&nbsp; &nbsp;</span></p>

### Dwa różne podejścia do sprawy. Analogowe i cyfrowe.

Najprostsze z sensownych analogowe z izolacją galwaniczną:

![walking]({{ site.baseurl }}/assets/images/signal3.png)

Ten sam układ z dodaną kompensacją "falowania" sygnału z czujnika. Falowania będącego skutkiem bicia na kole z wieńcem zębatym:

![walking]({{ site.baseurl }}/assets/images/signal3c.png)

Różnica polega na tym, że teraz układ "stara się" utrzymać stały, równy stosunek czasów trwania stanów wysokich i niskich na wyjściu U2.





#
#
#
#
#
#
#
#
#
#
#









