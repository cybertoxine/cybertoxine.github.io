---
layout: post
title:  "Mobilny Farmaceuta spotyka Speeduino."
author: MF
categories: [ energoelektronika, opensoftware, openhardware, sterownik silnika spalinowego, tuning, kogeneracja, agregat prądotwórczy, sport motorowy ]
image: assets/images/wmo.jpg
tags: [sticky]
---


<p style="color:green;">Ostatnia aktualizacja: 04.11.2025</p>

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
   - [5c] Prawidłowe wysterowanie wtryskiwaczy niskoimpedancyjnych. 
   - [5d] Możliwość udawania fabrycznego ECU po podłączeniu do czytnika OBD.
   - [5e] ???   
- [6] Przebieg realizacji.
- [7] Efekty i wnioski.


<p><span style="border: 2px solid red; border-radius: 10px; background-color: white;">&nbsp &nbsp[4a]&nbsp &nbsp</span></p>

![walking]({{ site.baseurl }}/assets/images/sicignition.png)

U1, U2 - Niezbyt szybkie transoptory. Szybkość podczas załączania nie jest potrzebna, prąd płynący w uzwojeniu pierwotnym cewki zapłonowej i tak narasta powoli.
Dałem dwie sztuki bo koszt prawie zerowy a mamy większą wydajność prądową. Układ przez to jest pewniejszy - czasem przy częściowym uszkodzeniu bramki MOSFET-a potrafi płynąć niewielki prąd. Poza tym wilgoć itd.

U3 - Szybki transoptor, powoduje momentalne załączenie T2 i prawie natychmiastowe sprowadzenie potencjału bramki MOSFET-a do 0V. Tu szybkość jest ważna bo cewka ma wygenerować stromy pik napięcia po stronie wtórnej.

T1 - MOSFET z węglika krzemu. Podczas przeglądania for na temat speeduino kilka razy natknąłem się na wpisy opisujące problem zakłucania pracy mikrokontrolera przez moduł zapłonowy.
Postanowiłem więc maksymalnie odizolować obwód wysokiego napięcia od obwodów mikrokontrolera. T1 ma malutką pojemność dren-bramka i dren-źródło co skutkuje niewielkim przedostawaniem się zakłuceń ze strony pierwotnej cewki zapłonowej w stronę mikrokontrolera. Dodatkową izolację z apewniają U1,U2 i U3. Tranzystor jest dość drogi (około 20zł) ale ze względu na wspomniane wyżej małe pojemności wewnętrzne, małą rezystancję w stanie załączenia, dopuszczalne bardzo wysokie napięcie pracy, prawie niezmieniające się wraz ze zmianą temperatury parametry, szybkość wyłączania (w przeciwieństwie do IGBT brak "ogona prądowego") i odporność na wysokie temperatury wart swojej ceny biorąc pod uwagę warunki panujące w okolicach sportowo eksploatowanego silnika spalinowego.

T2 - Rozładowuje bramkę T1 przez R6 po zadziałaniu U3.

T3 - Element obwodu ograniczającego prąd płynący przez cewkę. Gdy napięcie na R8 przekroczy 0.6V T3 zaczyna rozładowywać bramkę T1 działając w kontrze do U1 i U2.

D1 - Dioda Shottky z węglika krzemu. Element obwodu ograniczającego piki wysokiego napięcia po stronie wtórnej, zabezpieczającego T1 i cewkę przed uszkodzeniem. Kombinacja dużej szybkości i wysokiego dopuszczalnego napięcia wstecznego.
.
D2-D8 - Diody Zenera. Odkłada się na nich moc przepięć ze strony pierwotnej. Każda z nich ma napięcie przebicia lawinowego wynoszące 200V więc zabezpieczenie zaczyna działać "miękko" (dzięki R9) przy 600V i "twardo" przy 800V.

D9 - Dioda Zenera. Zabezpieczenie przed pikami napięcia mogącymi pojawić się na bramce T1.

C1 - Niskoimpedancyjny elekrolit 105st. C w obwodzie 12V.

C2 - Dodatkowa "przycięcie" szybkozmiennych sygnałów.

C3 - Stabilizacja napięcia na D2-D8 oraz zniwelowanie indukcyjności obwodu zawierającego diody Zenera. 

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












Tymczasem w Debrznie:

Pod Debrzeńskim lasem stało raz Seicento.\
Wieśniaczkom się nie podobało. Bez butów panienkom.\
Kierowca też chyba nie trafiał w ich gusta.\
"Na buty mi nie da!" Mówiła raz Tłusta.\
\
Chuda powiedziała "tylko samochody\
u tego jegomościa powodują wzwody".\
\
"Gdybym tak na ciuchy, szpilki i zegarki\
dostawała tyle ile kosztują go turbo te sprężarki...\
...pokochałabym" - Pomyślała Ruda -\
"Same tu pijaki a w robocie nuda." 







