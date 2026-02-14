---
layout: post
title:  "Mobilny Farmaceuta spotyka Speeduino."
author: MF
categories: [ energoelektronika, opensoftware, openhardware, sterownik silnika spalinowego, tuning, kogeneracja, agregat prądotwórczy, sport motorowy ]
image: assets/images/wmo.jpg
tags: [sticky]
---


<p style="color:green;">Ostatnia aktualizacja: 14.02.2026</p>
........................................................
........................................................
........................................................
<a name="spis"></a>
   
## Spis treści:

- [1] Co to jest Speeduino? Kto jest autorem i co mówi licencja na której ten projekt jest dystrybuowany? Link do repo.
- [2] Po co mi Speeduino?
- [3] Co postanowiłem zmienić/uzupełnić w Speeduino i dlaczego?
- [4] Konkretne rozwiązania etapu pierwszego i ich opis (część teoretyczna).
   - [4a] Dodanie modułu zapłonowowego na SiC MOSFETach.[skocz](#sik) 
   - [4b] Przetwornica. [skocz](#przet)
   - [4c] Dodanie kondycjonera sygnału z czujnika reluktancyjnego położenia wału korbowego. [skocz](#kond)
- [5] Konkretne rozwiązania etapu drugiego i ich opis (część teoretyczna).
   - [5a] Użycie AVR128DB64 zamiast ATmega2560. 
   - [5b] Wykrywanie spalania stukowego na drodze DSP. 
   - [5c] Prawidłowe wysterowanie benzynowych wtryskiwaczy niskoimpedancyjnych. 
   - [5d] CAN.
   - [5e] Współpraca z sensorami oraz możliwość pełnosprawnego wysterowania aktuatorów systemu wtrysku Common Rail.
   - [5f] Dodanie nadrzędnej jednostki sterującej opartej o architekturę RISC-V (np ESP32-C3), interfejs do konfiguracji i diagnozy typu client(przeglądarka internetowa)-serwer(odpalony na RISC-V). Komunikacja Wi-Fi. Na AVRxm kod przepisany w asemblerze, na RISC-V pisany w Pythonie.
- [6] Przebieg realizacji, dziennik prac.
   - [6a] Etapu pierwszego. (kliknij datę by do niej skoczyć) [07.01.2026](#etap1a)•[09.01.2026](#etap1b)•[15.01.2026](#etap1c)•[19.01.2026](#etap1d)•[21.01.2026](#etap1e)•[14.02.2026](#etap1f) 
   - [6b] Etapu drugiego.
- [7] Efekty i wnioski.
   - [7a] Etap pierwszy.
   - [7b] Etap drugi.

<a name="sik"></a>
<p><span style="border: 2px solid red; border-radius: 10px; background-color: red;color:white">&nbsp; &nbsp;[4a]&nbsp; &nbsp;</span></p>

## Moduł zapłonowy dla jednej cewki:

Dla większej ilości cewek potrzebna wielokrotność poniższego układu.

![walking]({{ site.baseurl }}/assets/images/sicignition.png)

**U1, U2** - Niezbyt szybkie transoptory. Szybkość podczas załączania nie jest potrzebna, prąd płynący w uzwojeniu pierwotnym cewki zapłonowej i tak narasta powoli.
Dałem dwie sztuki bo koszt prawie zerowy a mamy większą wydajność prądową. Układ przez to jest pewniejszy - czasem przy częściowym uszkodzeniu bramki MOSFET-a potrafi płynąć niewielki prąd. Poza tym wilgoć itd.
**U3** - Szybki transoptor, powoduje momentalne załączenie T2 i prawie natychmiastowe sprowadzenie potencjału bramki MOSFET-a do 0V. Tu szybkość jest ważna bo cewka ma wygenerować stromy pik napięcia po stronie wtórnej.
**T1** - MOSFET z węglika krzemu. Podczas przeglądania for na temat speeduino kilka razy natknąłem się na wpisy opisujące problem zakłucania pracy mikrokontrolera przez moduł zapłonowy.
Postanowiłem więc maksymalnie odizolować obwód wysokiego napięcia od obwodów mikrokontrolera. T1 ma malutką pojemność dren-bramka i dren-źródło co skutkuje niewielkim przedostawaniem się zakłuceń ze strony pierwotnej cewki zapłonowej w stronę mikrokontrolera. Dodatkową izolację zapewniają U1, U2 i U3. Tranzystor jest dość drogi (około 20zł) ale ze względu na wspomniane wyżej małe pojemności wewnętrzne, małą rezystancję w stanie załączenia, dopuszczalne bardzo wysokie napięcie pracy, prawie niezmieniające się wraz ze zmianą temperatury parametry, szybkość wyłączania (w przeciwieństwie do IGBT brak "ogona prądowego") i odporność na wysokie temperatury wart swojej ceny biorąc pod uwagę warunki panujące w okolicach sportowo eksploatowanego silnika spalinowego.
**T2** - Rozładowuje bramkę T1 przez R6 po zadziałaniu U3.
**T3** - Element obwodu ograniczającego prąd płynący przez cewkę. Gdy napięcie na R8 przekroczy 0.6V T3 zaczyna rozładowywać bramkę T1 działając w kontrze do U1 i U2.
**D1** - Dioda Shottky z węglika krzemu. Element obwodu ograniczającego piki wysokiego napięcia po stronie wtórnej, zabezpieczającego T1 i cewkę przed uszkodzeniem. Kombinacja dużej szybkości i wysokiego dopuszczalnego napięcia wstecznego.
**D2-D8** - Diody Zenera. Odkłada się na nich moc przepięć ze strony pierwotnej. Każda z nich ma napięcie przebicia lawinowego wynoszące 200V więc zabezpieczenie zaczyna działać "miękko" (dzięki R9) przy 600V i "twardo" przy 800V.
**D9** - Dioda Zenera. Zabezpieczenie przed pikami napięcia mogącymi pojawić się na bramce T1.
**C1** - Niskoimpedancyjny elekrolit 105st. C w obwodzie 12V.
**C2** - Dodatkowe "przycięcie" szybkozmiennych sygnałów.
**C3** - Stabilizacja napięcia na D2-D8 oraz niwelowanie wpływu indukcyjności obwodu zawierającego diody Zenera. 
**C4** - Wiadomo. Na szynie 20V.
**R1** - Ograniczenie prądu płynącego przez U3 i bramkę T2.
**R2** - Przyspieszenie wyłączania T2.
**R3, R4** - Ograniczenie prądu płynącego przez U1 i U2.
**R5** - Opór na tyle niewielki, że "przeważa" nad R3 i R4 w momencie zadziałania ograniczenia prądu płynącego przez cewkę.
**R6** - Ograniczenie prądu płynącego przez T2. 
**R7** - Ograniczenie prądu płynącego przez bramkę T3 w momencie zadziałania ograniczenia prądu cewki.
**R8** - Ustala maksymalną wartość prądu cewki. Pełni rolę bezpiecznika w razie uszkodzenia T1.
**R9** - Punkt w którym można nawet bez oscyloskopu sprawdzić, czy napięcie w obwodzie pierwotnym cewki nie jest zbyt wysokie.
**R10** - Ograniczenie prądu płynącego przez LED-a w U3.
**R11** - Ograniczenie prądu płynącego przez LED-y w U1 i U2.

[skocz do spisu treści](#spis)

<a name="przet"></a>
<p><span style="border: 2px solid red; border-radius: 10px; background-color: red;color:white">&nbsp; &nbsp;[4b]&nbsp; &nbsp;</span></p>

## Przetwornica napięcia. (zasilanie m.in. driverów bramek SiC-MOSFET-ów):

![walking]({{ site.baseurl }}/assets/images/boostt.png)

**C1**  - Niskoimpedancyjny elektrolityczny.
**C2** - Niskoimpedancyjny elektrolityczny. Chociaż chyba lepiej tu tantalowy 25V.
**U1** - Dobry, tani, nieco przestarzały już układ scalony przetwornicy. 52KHz, cycle by cycle current limit przy 5A. Tutaj w nietypowej konfiguracji buck-boost, nieopisanej w nocie katalogowej.
Przewymiarowane bo IC jest w stanie "przepompować" 20W mocy w tej topologii i przy tym napięciu zasilania. Zdecydowałem się jednak na ten układ bo modyfikując L1 można łatwo uzyskać szereg wyjść o różnych napięciach. Oprócz 20V potrzebne jest 5V. Prawdopodobnie przyda się też 3.3V. Prawdopodobnie zajdzie potrzeba zasilenia bramek high side N-MOSFET-ów z izolowanego, "pływającego" uzwojenia. Narazie niech tak zostanie (prosto, tanio, wystarczająco dobrze i elastycznie).
**T1** - MOSFET z pojemnością bramki tylko 300pF.
**D1** - Zaczyna przewodzić gdy górny klucz wbudowany w U1 przestaje przewodzić.
**D2, D3** - Zabezpieczenie bramki T1 przed przepięciami.
**D4** - Jak D1. Na zmianę przewodzą U1 + T1 i D1 + D4. Kondensator wyjściowy jest ładowany tylko w czasie zanikania pola magnetycznego w dławiku więc łatwo "doczepić" wiele wyjść, jak w flybacku.
**D5** - Jakiś LED o znamionowym ciągłym prądzie pracy większym bądź równym 20mA.
**R1** - Szeregowo z bramką T1.
**R2** - Ograniczenie prądu płynącego przez LED, niewielkie wstępne obciążenie przetwornicy.
**R3, R4** - Dzielnik napięcia, ustala napięcie wyjściowe.
**L1** - Tu zależy. Gdy ma być zasilany tylko moduł zapłonowy dławik może być mały, stratny i o większej indukcyjności.

[skocz do spisu treści](#spis)

<a name="kond"></a>
<p><span style="border: 2px solid red; border-radius: 10px; background-color: red;color:white">&nbsp; &nbsp;[4c]&nbsp; &nbsp;</span></p>

## Kondycjoner sygnału z czujnika reluktancyjnego położenia wału korbowego. Trzy różne podejścia do sprawy. Analogowe, analogowe uproszczone oraz DSP.

### Analogowe:

Regulacja wzmocnienia, przesuwanie przebiegu w stronę zera, ukształtowanie prostokątnego sygnału wyściowego, bez izolacji galwanicznej:

![walking]({{ site.baseurl }}/assets/images/signal4.png)

Koszt elementów elektronicznych potrzebnych do zbudowania powyższego układu nie przekracza 10zł. LM324, 3 tranzystory, 3 optoizolatory, kilka diod, kondensatorów i rezystorów.
Po zmontowaniu na dwustronnej płytce rozmiar całości bardzo mały więc nie wiem czy stosowanie scalonych obwodów dedykowanych dla czujników VR ma sens. Nie podaję tutaj wartości elementów gdyż nie potrafię do końca przewidzieć jak układ się zachowa. Niby LM324B powinien "się wyrabiać" przy 10kHz na wyjściu czujnika VR, niby charakterystyka optoizolatorów nie musi odzwierciedlać zachowania się rezystora. Musiałbym nad tym posiedzieć z oscyloskopem. Przedstawione wyżej rozwiązanie ma charakter koncepcyjny. Moim zdaniem w dobie mikrokontrolerów w cenie batonika nie ma sensu iść tą drogą.

### Analogowe uproszczone:

![walking]({{ site.baseurl }}/assets/images/simplean3.png)

Zamiast regulacji wzmocnienia "miękkie przycięcie" szczytów przebiegu sygnału z czujnika przy użyciu pomarańczowych LEDów. Zamiast sprzężenia zwrotnego utrzymjącego 50% wypełnienie sygnału wyjściowego jest tutaj przemieszczanie się (w takt obrotu wału korbowego, chodzi o zmniejszenie wpływu zmian odległości czujnika od wieńca zębatego) punktu odniesienia dla komparatora przy wejściu odwracającym wzmacniacza. Komparator jest bez histerezy, pozostaje jedynie histereza portu wejściowego mikrokontolera. Mimo swojej prostoty układ ma duże szanse na zadowalające działanie w rzeczywistych warunkach więc może "dla jaj" go kiedyś wykonam.


### Cel stosowania DSP i jego algorytm. Wstęp do grupy rozwiązań analogowo-cyfrowych.

Na poniższym rysunku przedtawiono 3 różne, podobne sygnały z reluktancyjnego czujnika położenia wału korbowego, operacje na nich wykonywane i ostateczny sygnał wyjściowy nadający się do podania
do Speeduino.

![walking]({{ site.baseurl }}/assets/images/vrsc2.png)

Celem stosowania DSP w układzie kondycjonera jest zmniejszenie wpływu deformacji sygnału z czujnika na proces wyznaczania aktualnej pozycji wału korbowego przez software odpalony na ATmega2560.

Drugorzędny ale istotny dla mnie cel zaimplementowania obróbki cyfrowej sygnału to nadanie projektowi maksymalnej elastyczności. Wgranie innego software pozwoli na współpracę z innymi tego samego typu czujnikami i/albo pracującymi w innym otoczeniu.

Co do algorytmów to trzeba zadbać o małą złożoność obliczeniową wszystkich procedur. Np przy 6000 RPM na 1 ząbek z rowkiem (koło foniczne 60-2) przypada 0.166 milisekundy czyli (dla kondycjonera DSP w opcji drugiej) około 20 sampli na okres przebiegu. Pomiędzy momentami w których pojawia się wynik konwersji jest 208 cykli rdzenia.

Potrzebne będą 3 funkcje. Pierwsza to dynamiczne określanie poziomu odniesienia dla detektora przejścia przez zero. Druga to sam detektor przejścia przez zero. Trzecia to automatyczna regulacja tłumienia (właściwie to sterowanie poziomem rezystancji wejściowej kondycjonera + kompensowanie skokowych zmian rezystancji krótkookresowym podładowywaniem albo rozładowywaniem kondenstorów podwajacza przy AREF).

Określanie poziomu odniesienia:

Przesunięcie całego przebiegu w górę lub w dół jest spowodowane cyklicznymi zmianami odległości wieńca zębatego od czujnika. Przyczynami są drgania rezonansowe wału korbowego i kadłuba silnika, zużyte łożyska, wyjątkowo niedbałe osadzenie koła z wieńcem zębatym przez pijanego serwisanta albo też wklejenie czujnika na gumę do żucia.

Algorytm powinien zapamiętywać wartości poziomu sygnału z dwóch poprzednich skrajnych wychyleń sinusoidy i wyliczać średnią. Średnia będzie poziomem odniesienia.
Detekcja szkrajnych wychyleń jest prosta. Jeśli wartości sygnału zaczynają maleć to wcześniejsza próbka była poziomem MAX. Jeśli wartości sygnału zaczynają rosnąć to poprzednia próbka była poziomem MIN.
W praktyce by zapewnić odporność na szum potrzeba brać pod uwagę kilka kolejnych próbek. "Kły" w otoczeniu "wybitych zębów" powiny być ignorowane.

Detektor przejścia przez zero:

Myk polega na ustaleniu momentu w którym próbka zaczyna być większa albo mniejsza od wartości uznawanej przez zero zależnie od kierunku zmiany napięcia sygnału. Wtedy ma nastąpić zmiana stanu wyjścia kondycjonera. Odporność na szum uzyskać tutaj trudniej ponieważ potrzeba dokonać decyzji (przełączenia wyjścia) jak najbliżej chwili w której poziom zera zostaje przekroczony.A jak się przełączy to się później już nie odprzełączy...

Automatyczna regulacja tłumienia:

Takie wysterowywanie transoptorów aby wartość AREF oscylowała blisko 3.75V. Wartości próbek nie mogą stanowić odniesienia dla ART (bo są zależne od AREF) dlatego informacji takiej musi dostarczyć
sprzętowy komparator.


### DSP opcja pierwsza (użyty optoFET):

Funkcje opisane przy okazji przedstawienia rozwiązania analogowego są możliwe do zaimplementowania za pomocą mikrokontrolera w cenie batonika.
Przykładem niech będzie ATtiny44A. Nota katalogowa podaje, że przetwornik analogowo-cyfrowy wbudowany w ten mikrokontroler pracuje z szybkością 16ksps przy 10-cio bitowej rozdzielczości.
Tor analogowy tego mikrokontrolera posiada pasmo ograniczone od góry ze spadkiem o 3dB w punkcie 38.5 KHz (dla konwersji różnicowych jest to 4KHz).

W kilku miejscach w sieci ludzie raportowali, że przetwornik AD w ATtiny44A potrafi pracować z precyzją bliską 8 bitów gdy taktowany jest zegarem 2MHz.
Daje to częstotliwość próbkowania 2 mln/13/s = 153KHz i 104 cykle (przy zegarze 16MHz z wewnętrznego oscylatora) szybkiego, krótkopotokowego rdzenia na 1 próbkę. Wystarczająco dobrze.
Najlepszym ustawieniem wydaje się być 1.5 mln/13/s = 115KHz i 208 cykli rdzenia na próbkę (lekkie przetaktowanie do 24MHz), jednak wtedy należałoby dołożyć zewnętrzny oscylator.
Niestety rdzeń wspomnianego ATtiny pozbawiono możliwości wykonywania instrukcji MUL, a szkoda. Utrudnia to albo uniemożliwia implementację bardziej złożonych algorytmów.

![walking]({{ site.baseurl }}/assets/images/tiny44k2.png)

Układ jest izolowany galwanicznie. Zasilany z osobnego uzwojenia nawiniętego na rdzeniu wcześniej przedstawianej przetwornicy. Na wyjściu znajduje się transoptor. 
Dzięki takiemu manewrowi całość zachowuje się jak wzmacniacz operacyjny nie posiadający (praktycznie) limitu co do wysokości napięcia na wejściach, układ reaguje tylko na różnicę napięć przy R12 i R13.
Zakres dynamiczny ADC pracującego z podwyższoną częstotliwością jest mały. W tym mikrokontrolerze można liczyć na max 8 bitów dlatego też płynnie i w zależności od poziomu sygnału na wejściu zmienia się napięcie odniesienia dla przetwornika (PWM na nóżce 5) oraz zmianom ulega rezystancja U4 (PWM na nóżce 6) czyli jest tutaj sterowana programowo automatyczna regulacja wzmocnienia.

Rozwiązanie z DSP na pierwszy rzut oka wygląda na overkill jednak jego niski koszt i elastyczność czyni go sensownym wyborem dla asemblerowego magika. BTW Gdy zastosuje się ATtiny44A pozostaje wolne 6 pinów (z resetem 7). Wystarczy na przykład do wysterowania 3-cyfrowego wyświetlacza i odbioru danych UARTem...albo co tam do głowy przyjdzie.

### DSP opcja druga (użyty transoptorowy Digital Controlled Resistor):

Mój faworyt.

![walking]({{ site.baseurl }}/assets/images/tiny44kombo.png)

Przetaktowywanie układów scalonych nie jest dobrą praktyką w przypadku konstruowania profesjonalnych, komercyjnych urządzeń. Jednakże "indywiduum" składające samemu sobie potrzebne mu urządzenie stoi na uprzywilejowanej pozycji. Można przeprowadzić selekcję układów i wytypować te, które pracują stabilnie z zegarem nieco wyższym od dopuszczalnego, przedstawionego w nocie katalogowej.
W praktyce wygląda to tak, że prawie każdy egzemplarz Attiny44A z serii o maksymalnej dopuszczalnej temperaturze pracy do 125 st. C w temperaturze pokojowej, przy dobrze odfiltrowanym napięciu zasilania wynoszącym 5V i przy dostarczeniu sygnału zegara z zewnętrznego oscylatora pracuje stabilnie z taktowaniem 24MHz. Wspominam o tym by podkreślić potrzebę wyboru odpowiedniej serii układu oraz potrzebę przeprowadzenia selekcji pod kątem odpowiedniego marginesu bezpieczeństwa, innymi słowy "zapasu stabilności". Normalnie powinno się po prostu dokonać wyboru któregoś z szybszych mikrokontrolerów, których na rynku nie brakuje. Mój wybór jest podyktowany lenistwem, przyzwyczajeniem, dobrą znajomością dokumentacji układu i doświadczeniem w "męczeniu" go. Ponadto 8 bitowe AVR-y mają prosty i zwięzły asembler co przy kombinowaniu przy DSP okazuje się wielką zaletą.

Zegar: 24 MHz, częstotliwość taktowania ADC: 1.5 MHz, napięcie zasilania mikrokontrolera: 5V. Napięcie referencyjne dla ADC zmienia się wraz ze zmianą amplitudy napięcia z czujnika.
AREF jest dołączony do + podwajacza napięcia utworzonego przez C6, C7, D4, D5. Podwajacz jest zasilany z gałęzi utworzonej przez cyfrowo regulowany rezystor połączony równolegle (przez R10) z czujnikiem reluktancyjnym. Rozwiązanie owe skutkuje tym, że amplituda sygnału dla ADC na PA3 jest zawsze wysoka, bardzo bliska napięciu na AREF. Przebieg sygnału wypełnia przedział między poziomem GND i AREF z minimalnym albo zerowym clippingiem. Powyższym trickiem udało się zmniejszyć negatywny wpływ ograniczonego do 8-bitów zakresu dynamicznego ADC.
Poprzez R7 można dokonywać niewielkiej korekty napięcia na podwajaczu minimalizując ryzyko clippingu albo też zwiększyć czułość ADC gdy nastąpi gwałtowny spadek amplitudy sygnału. Wątpię jednak by w realu zachodziła taka potrzeba. R8 i R9 "pilnują" by napięcia na kondensatorach były identyczne.

Mikrokontroler załącza optoizolatory U6-U15 tak by modulować poziom sygnału na wejściu podwajacza. Są 32 wartości oporu do wyboru.
W sumie daje to wysoką czułość i dokładność przy niskich obrotach wieńca zębatego i jednocześnie poprawne próbkowanie sygnału przy wysokich obrotach. 
Nie jestem pewien czy R12, R13, R15, R16, R17 doprowadzą do nasycenia tranzystory w optoizolatorach. Ich dokładne wartości powinny być ustalone eksperymentalnie.
W CTR optoizolatorów bywają znaczne różnice pomiędzy egzemplarzami, występuje też problem stażenia.
Programowe przełączanie optoizolatorów odbywa się w momencie mijania dwóch "wybitych" zębów i z zachowaniem odpowiednich deadtime-ów. Moment ten przedstawia poniższa grafika:

![walking]({{ site.baseurl }}/assets/images/optoc.png)

Chodzi o to by zniekształcenia sygnału powstające w chwili zmiany oporu były jak najmniejsze i powstawały w chwili, która nie ma znaczenia dla odczytu położenia wału korbowego. 
Przy wysokich obrotach zmniejszona impedancja regulowanego rezystora powoduje zwiększenie prądu płynącego przez uzwojenie czujnika i zmniejszenie indukcji w jego rdzeniu. 
Poprawia to (nieznacznie) liniowość odpowiedzi czujnika na zmianę natężenia pola magnetycznego w jego otoczeniu.
Komparator wbudowany w Attiny przez PA1 wyczuwa moment w którym napięcie podwajacza osiąga 3.75V albo spada poniżej tej wartości uruchamiając procedurę zwiększania albo zmniejszania rezystancji regulowanego rezystora. Zaciski do podłączenia czujnika reluktancyjnego zachowują się jak wejścia wzmacniacza operacyjnego o regulowanej impedancji wejściowej, poprawiono tym samym odporność na zakłócenia.

Wyjście kondycjonera jest izolowane przez U2. D1 sobie mryga gdy trzeba, pełni rolę diagnostyczną.

### Inne sposoby regulacji poziomu sygnału w torze analogowym.

1. Kluczowanie stałą rezystancją z wielką częstotliwością ze zmiennym wypełnieniem, za niewielkim kondensatorem. (wada: "sianie RF-em")
2. Fotorezystor CdS albo PbS i źródło światła (PbS ma czas reakcji do zaakceptowania, CdS jest strasznie powolny).(wada: CdS powolny, gotowe detektory PbS drogie)
3. Obciążenie wejścia układem generującym cykliczne zmiany pola magnetycznego w materiale rdzenia ferromagnetycznego (NIE chodzi o rdzeń czujnika) by wywoływać w nim kontrolowane częstotliwością przełączania straty. (warto kiedyś sprawdzić, dzielnik rezystancyjny z jednym rezystorem sterowanym częstotliwością - ciekawa koncepcja)
4. JFET/MOSFET w pracujący w obszarze omowym czyli z napięciem bramki niewiele większym od napięcia progowego. (wada: charakterystyka podobna do charakterystyki rezystora ale jednak inna - zniekształcenia sygnału, w pobliżu napięcia progowego wzmocnienie jest bardzo duże - dużo szumu, układ będzie "zbierał szpile" z otoczenia)


[skocz do spisu treści](#spis)

<a name="etap1a"></a>
<p><span style="border: 2px solid red; border-radius: 10px; background-color: red;color:white">&nbsp; &nbsp;[6a]&nbsp; &nbsp;</span></p>

### 07.01.2026

Wykonano generator sygnału reluktancyjnego czujnika położenia wału korbowego w celu testowania różnych rozwiązań układów elektronicznych kondycjonujących ten sygnał.

Przy okazji testom podlega kompozytowy pas napędowy (sieciowany silikon-włókno szklane).  

![walking]({{ site.baseurl }}/assets/images/spinner.jpg)

Poniższy film przedstawia konstrukcję generatora:

https://www.youtube.com/watch?v=yFezR9aNbO8

Na kolejnym materiale widać sygnał generatora na ekranie oscyloskopu. Pomiar odbywa się przy różnych prędkościach obrotowych (od kilkuset RPM do 4500 RPM).

https://www.youtube.com/watch?v=T8YZQ1_v4hg

Maszynka potrafi chwilowo dobić do 10000 RPM. Użytkowana jest z należytą ostrożnością - uderzenie fragmentem (w razie rozerwania) żeliwnego wirującego elementu może zabić. Zastosowano modelarski szczotkowy silnik SPEED 600 o nominalnym napięciu pracy 7.2V który dobrze znosi krótkie przeciążenia.
Przekraczanie 100W mocy na wale (przy napięciu 12V) odbywa się nieniszcząco. Po ukończeniu kondycjonera maszynka znajdzie zastosowanie jako bajerancka garażowa wysokoobrotowa szlifierka :). 

[skocz do spisu treści](#spis)

<a name="etap1b"></a>
### 09.01.2026 

Materiał paska wymaga udoskonalenia.

![walking]({{ site.baseurl }}/assets/images/pasekbum.jpg)

Trwa kompletowanie hardware-u.

![walking]({{ site.baseurl }}/assets/images/kompletowanie.jpg)

[skocz do spisu treści](#spis)

<a name="etap1c"></a>
### 15.01.2026

Namalowałem prototypową płytkę do kondycjonera DSP z DCR. Widok z góry. Płytka jest dwustronna. Z drugiej strony znajdą się U3 i U4 otoczone polem miedzi.

![walking]({{ site.baseurl }}/assets/images/dspvrsc.png)

Z elektrośmieciowego Sound Blaster-a Live! wylutowałem pasujący do projektu generator kwarcowy.

![walking]({{ site.baseurl }}/assets/images/q24576.jpg)

[skocz do spisu treści](#spis)

<a name="etap1d"></a>
### 19.01.2026

Robiąc dwustronnie wyszłoby mniejsze. Narazie to prototyp, póżniej "ścisnę". Nawet teraz wymiary są do zaakceptowania.

![walking]({{ site.baseurl }}/assets/images/vrscplytka.jpg)


[skocz do spisu treści](#spis)

<a name="etap1e"></a>
### 21.01.2026

Mikrokontroler i transoptory gotowe do przylutowania:

![walking]({{ site.baseurl }}/assets/images/vrscq.jpg)

ATtiny44A opcja N, industrial, -40 do 105 st.C. 
Przy overclockingu lepsza byłaby opcja F czyli od -40 do 125 st.C. 

TCXO 24.576 MHz:

![walking]({{ site.baseurl }}/assets/images/vrscchip.jpg)

Wbudowany w ATtiny44A układ współpracujący z zewnętrznymi kwarcami "nie lubi" częstotliwości wyższych od 20 MHz. Dlatego dodałem zewnętrzny kompletny generator.
Pod ręką miałem tylko taki wielki. :)

[skocz do spisu treści](#spis)

<a name="etap1f"></a>
### 14.02.2026

Sprawdziłem jakie przebiegi napięcia wyjściowego uzyskam obciążając czujnik położenia wału różnymi wartościami rezystancji.

![walking]({{ site.baseurl }}/assets/images/sroshgmbh.jpg)

Okazuje się, że obwód regulacji tłumienia nie musi mieć tak dużej rozdzielczości jak myślałem. Zależność obroty-napięcie w obrębie gęstego zgryzu przypomina wykres funkcji logarytmicznej.
32 wartości rezystancji zupełnie wystarczą. Rezystory w cyfrowo regulowanym tłumiku nie powinny iść śladem potęgi dwójki jak wcześniej zakładałem. Ich wartości idą do poprawki.
Najbardziej podobała mi się praca czujnika pod obciążeniem 470 Ohm. Piki wokół wybitych zębów będą musiały być w torze analogowym miękko obcinane. Chyba najlepiej będzie ciąć po przejściu przez DCR. 

[skocz do spisu treści](#spis)










