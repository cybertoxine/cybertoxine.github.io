---
layout: post
title:  "Mobilny Farmaceuta spotyka Speeduino."
author: MF
categories: [ energoelektronika, opensoftware, openhardware, sterownik silnika spalinowego, tuning, kogeneracja, agregat prądotwórczy, sport motorowy ]
image: assets/images/wmo.jpg
tags: [sticky]
---


<p style="color:green;">Ostatnia aktualizacja: 03.11.2025</p>

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


[4a]

![walking]({{ site.baseurl }}/assets/images/sicignition.png)

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







