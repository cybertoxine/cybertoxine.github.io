---
layout: post
title:  "Mobilny Farmaceuta spotyka Speeduino."
author: MF
categories: [ energoelektronika, opensoftware, openhardware, sterownik silnika spalinowego, tuning, kogeneracja, agregat prądotwórczy, sport motorowy ]
image: assets/images/speedlogofin.jpg
tags: [sticky]
---


<p style="color:green;">Ostatnia aktualizacja:&nbsp;02.05.2026&nbsp;</p>
<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p>

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
   - [6a] Etapu pierwszego. (kliknij datę by do niej skoczyć)

[07.01.2026](#etap1a) <span style="color:crimson">Ostre rozkręcanie koła fonicznego - wykonanie generatora sygnał czujnika p.w.k.</style>

[09.01.2026](#etap1b) <span style="color:crimson">Catastrophic failure paska napędowego + sesja photo.</style>

[15.01.2026](#etap1c) <span style="color:crimson">Proto 1 obrazu ścieżek miedzianych kondycjonera sygnału czujnika p.w.k.</style>

[19.01.2026](#etap1d) <span style="color:crimson">Trawienie i oględziny proto 1 obrazu ścieżek.</style>

[21.01.2026](#etap1e) <span style="color:crimson">Fotki ze składania proto 1 k.s.c.p.w.k.</style>

[14.02.2026](#etap1f) <span style="color:crimson">C.p.w.k i różne rezystory pod oscyloskopem.</style>

[25.02.2026](#etap1g) <span style="color:crimson">Kilka refleksji toważyszących składaniu k.s.c.p.w.k i rozruch DCR.</style>

[27.02.2026](#etap1h) <span style="color:crimson">Badanie DCR pod kątem występowania progu napięciowego.</style>

[06.03.2026](#etap1i) <span style="color:crimson">Przerzutnik Schmitta.</style>

[08.03.2026](#etap1j) <span style="color:crimson">Uruchomienie shit triggera.</style>

[15.03.2026](#etap1k) <span style="color:crimson">Ostateczny schemat elektryczny k.s.c.p.w.k.</style>

[17.03.2026](#etap1l) <span style="color:crimson">Proto 2 obrazu ścieżek miedzianych k.s.c.p.w.k.</style>

[30.03.2026](#etap1m) <span style="color:crimson">Rysowanie ścieżek miedzianych SMPS-a.</style>

[08.04.2026](#etap1n) <span style="color:crimson">Wytrawione proto 2 płytki kondycjonera.</style>

[13.04.2026](#etap1o) <span style="color:crimson">Wytrawiona płytka przetwornicy.</style>

[19.04.2026](#etap1p) <span style="color:crimson">Piotras Converter odpalił.</style>

[02.05.2026](#etap1r) <span style="color:crimson">Pomiary przetwornicy + schemat i fotki złożonego układu.</style>

   - [6b] Etapu drugiego.
- [7] Efekty i wnioski.
   - [7a] Etap pierwszy.
   - [7b] Etap drugi.

<a name="sik"></a>
<p><span style="border: 2px solid red; border-radius: 10px; background-color: red;color:white">&nbsp; &nbsp;[4a]&nbsp; &nbsp;</span></p>

## Moduł zapłonowy dla jednej cewki:

![SiC ignitor]({{ site.baseurl }}/assets/images/sicignition.png)

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

![boost converter]({{ site.baseurl }}/assets/images/boostt.png)

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

![signal conditioner schematic]({{ site.baseurl }}/assets/images/signal4.png)

Koszt elementów elektronicznych potrzebnych do zbudowania powyższego układu nie przekracza 10zł. LM324, 3 tranzystory, 3 optoizolatory, kilka diod, kondensatorów i rezystorów.
Po zmontowaniu na dwustronnej płytce rozmiar całości bardzo mały więc nie wiem czy stosowanie scalonych obwodów dedykowanych dla czujników VR ma sens. Nie podaję tutaj wartości elementów gdyż nie potrafię do końca przewidzieć jak układ się zachowa. Niby LM324B powinien "się wyrabiać" przy 10kHz na wyjściu czujnika VR, niby charakterystyka optoizolatorów nie musi odzwierciedlać zachowania się rezystora. Musiałbym nad tym posiedzieć z oscyloskopem. Przedstawione wyżej rozwiązanie ma charakter koncepcyjny. Moim zdaniem w dobie mikrokontrolerów w cenie batonika nie ma sensu iść tą drogą.

### Analogowe uproszczone:

![analog signal conditioner schematic]({{ site.baseurl }}/assets/images/simplean3.png)

Zamiast regulacji wzmocnienia "miękkie przycięcie" szczytów przebiegu sygnału z czujnika przy użyciu pomarańczowych LEDów. Zamiast sprzężenia zwrotnego utrzymjącego 50% wypełnienie sygnału wyjściowego jest tutaj przemieszczanie się (w takt obrotu wału korbowego, chodzi o zmniejszenie wpływu zmian odległości czujnika od wieńca zębatego) punktu odniesienia dla komparatora przy wejściu odwracającym wzmacniacza. Komparator jest bez histerezy, pozostaje jedynie histereza portu wejściowego mikrokontolera. Mimo swojej prostoty układ ma duże szanse na zadowalające działanie w rzeczywistych warunkach więc może "dla jaj" go kiedyś wykonam.


### Cel stosowania DSP i jego algorytm. Wstęp do grupy rozwiązań analogowo-cyfrowych.

Na poniższym rysunku przedtawiono 3 różne, podobne sygnały z reluktancyjnego czujnika położenia wału korbowego, operacje na nich wykonywane i ostateczny sygnał wyjściowy nadający się do podania
do Speeduino.

![DSP algorithm ilustration]({{ site.baseurl }}/assets/images/vrsc2.png)

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

![digital signal conditioner schematic]({{ site.baseurl }}/assets/images/tiny44k2.png)

Układ jest izolowany galwanicznie. Zasilany z osobnego uzwojenia nawiniętego na rdzeniu wcześniej przedstawianej przetwornicy. Na wyjściu znajduje się transoptor. 
Dzięki takiemu manewrowi całość zachowuje się jak wzmacniacz operacyjny nie posiadający (praktycznie) limitu co do wysokości napięcia na wejściach, układ reaguje tylko na różnicę napięć przy R12 i R13.
Zakres dynamiczny ADC pracującego z podwyższoną częstotliwością jest mały. W tym mikrokontrolerze można liczyć na max 8 bitów dlatego też płynnie i w zależności od poziomu sygnału na wejściu zmienia się napięcie odniesienia dla przetwornika (PWM na nóżce 5) oraz zmianom ulega rezystancja U4 (PWM na nóżce 6) czyli jest tutaj sterowana programowo automatyczna regulacja wzmocnienia.

Rozwiązanie z DSP na pierwszy rzut oka wygląda na overkill jednak jego niski koszt i elastyczność czyni go sensownym wyborem dla asemblerowego magika. BTW Gdy zastosuje się ATtiny44A pozostaje wolne 6 pinów (z resetem 7). Wystarczy na przykład do wysterowania 3-cyfrowego wyświetlacza i odbioru danych UARTem...albo co tam do głowy przyjdzie.

### DSP opcja druga (użyty transoptorowy Digital Controlled Resistor):

Mój faworyt.

![digital signal conditioner schematic]({{ site.baseurl }}/assets/images/tiny44kombo.png)

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

![crancshaft angle]({{ site.baseurl }}/assets/images/optoc.png)

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

![signal generator]({{ site.baseurl }}/assets/images/spinner.jpg)

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

![transmission belt failure]({{ site.baseurl }}/assets/images/pasekbum.jpg)

Trwa kompletowanie hardware-u.

![electronic parts]({{ site.baseurl }}/assets/images/kompletowanie.jpg)

[skocz do spisu treści](#spis)

<a name="etap1c"></a>
### 15.01.2026

Namalowałem prototypową płytkę do kondycjonera DSP z DCR. Widok z góry. Płytka jest dwustronna. Z drugiej strony znajdą się U3 i U4 otoczone polem miedzi.

![copper picture]({{ site.baseurl }}/assets/images/dspvrsc.png)

Z elektrośmieciowego Sound Blaster-a Live! wylutowałem pasujący do projektu generator kwarcowy.

![quartz generator]({{ site.baseurl }}/assets/images/q24576.jpg)

[skocz do spisu treści](#spis)

<a name="etap1d"></a>
### 19.01.2026

Robiąc dwustronnie wyszłoby mniejsze. Narazie to prototyp, póżniej "ścisnę". Nawet teraz wymiary są do zaakceptowania.

![laminate]({{ site.baseurl }}/assets/images/vrscplytka.jpg)


[skocz do spisu treści](#spis)

<a name="etap1e"></a>
### 21.01.2026

Mikrokontroler i transoptory gotowe do przylutowania:

![parts on laminate]({{ site.baseurl }}/assets/images/vrscq.jpg)

ATtiny44A opcja N, industrial, -40 do 105 st.C. 
Przy overclockingu lepsza byłaby opcja F czyli od -40 do 125 st.C. 

TCXO 24.576 MHz:

![quartz generator]({{ site.baseurl }}/assets/images/vrscchip.jpg)

Wbudowany w ATtiny44A układ współpracujący z zewnętrznymi kwarcami "nie lubi" częstotliwości wyższych od 20 MHz. Dlatego dodałem zewnętrzny kompletny generator.
Pod ręką miałem tylko taki wielki. :)

[skocz do spisu treści](#spis)

<a name="etap1f"></a>
### 14.02.2026

Sprawdziłem jakie przebiegi napięcia wyjściowego uzyskam obciążając czujnik położenia wału różnymi wartościami rezystancji.

![test equipment]({{ site.baseurl }}/assets/images/sroshgmbh.jpg)

Okazuje się, że obwód regulacji tłumienia nie musi mieć tak dużej rozdzielczości jak myślałem. Zależność obroty-napięcie w obrębie gęstego zgryzu przypomina wykres funkcji logarytmicznej.
32 wartości rezystancji zupełnie wystarczą. Rezystory w cyfrowo regulowanym tłumiku nie powinny iść śladem potęgi dwójki jak wcześniej zakładałem. Ich wartości idą do poprawki.
Najbardziej podobała mi się praca czujnika pod obciążeniem 470 Ohm. Piki wokół wybitych zębów będą musiały być w torze analogowym miękko obcinane. Chyba najlepiej będzie ciąć po przejściu przez DCR. 

[skocz do spisu treści](#spis)

<a name="etap1g"></a>
### 25.02.2026
Poskładałem prototyp 1. Zbyt trudny w montażu, płytkę ostatecznie zrobię większą:

![completed prototype]({{ site.baseurl }}/assets/images/dcrimg0.jpg)

Attiny44A ruszyło przy zegarze 24.576 MHz. Rejestry, SRAM, FLASH, EEPROM wszystko elegancko, żadnych glitchy na portach.

Przetestowałem DCR (Digital Controlled Resistor) i wszystko oki. Do scharakteryzowania mam szybkość przełączania transoptorów oraz spadek napięcia na nich w bliskich okolicach zerowego napięcia na wyjściu z czujnika (czyli zachowanie przy niskim poziomie sygnału wejściowego, zniekształcenia przy przejściu przez zero).

![test circuit]({{ site.baseurl }}/assets/images/dcrimg1.jpg)

Przeprojektuję nieco tor analogowy. Mikrokontroler zajmie się sterowaniem impedancją wejścia oraz ustalaniem stałej czasowej ART (Automatycznej Regulacji Tłumienia). Wykrywaniem przejścia przejścia przez zero zajmie się sprzętowy komparator ze stałą histerezą, który "dostanie" na wejściu przebieg o stałej amplitudzie. Tak będzie lepiej. Układ dzięki takiej modyfikacji dobrze "ogarnie" również bardzo duże prędkości obrotowe koła fonicznego. 

Podczas obserwowania na oscyloskopie przebiegu z generatora sygnału czyjnika położenia wału zauważyłem, że bicie koła fonicznego skutkuje prawie wyłącznie zmianą amplitudy sygnału natomiast zmiany położenia sygnału względem zera nie były zauważalne. Wniosek: wystarczy DCR i ART, algorytm symetryzujący niepotrzebny. No może gdyby komuś zależało na dokładnościach rzędu 0.1 stopnia...

[skocz do spisu treści](#spis)

<a name="etap1h"></a>
### 28.02.2026

Byłem ciekaw czy transoptory w stanie załączonym będzie charakteryzowało jakieś progowe napięcie przy którym tranzystory wewnątrz nich zaczynają przewodzić.
Zestawiono obwód pomiarowy taki jak pokazano poniżej:

![circuit diagram]({{ site.baseurl }}/assets/images/dcr910.jpg)

Nagrałem szorta z obrazem z oscyloskopu:

https://www.youtube.com/shorts/h2ulkzRZBKs

Nie widzę objawów występowania progu. Fajnie.

Następnie sprawdziłem jak szybki jest DCR tzn. przy jakiej częstotliwości zmian rezystancji deformacje sygnału staną się istotne.
Ważna dla mnie jest też szerokość szpilek powstających przy przełączaniu ponieważ w założeniu DCR ma się wyrobić w czasie "przelotu dwóch wybitych zębów".
W prawym dolnym rogu zestawienia zamieściłem schemat obwodu pomiarowego.

![osciloscope screen picture]({{ site.baseurl }}/assets/images/dcrwyniki.jpg)

Wnioski:

1. Ustalenie się stanu rezystancji następuje maksymalnie po 40 mikrosekundach. Czas przelotu jednego ząbka z jednym rowkiem przy prędkości obrotowej wału wynoszącej 6000 obr/minutę, dla koła fonicznego 58-2, wynosi 166 mikrosekund. Czyli szybkość DCR-a jest wystarczająca dla prędkości obrotowych wynoszących nawet 12000 obr/min co jest wynikiem zupełnie wystarczającym w zastosowaniach w których projektowany kondycjoner sygnału ma się znaleźć.

2. Brak zauważalnego progu napięciowego pozwala sądzić, że DCR nie zaburzy w żaden sposób działania sprzętowego komparatora z histerezą.

3. Prezentowany układ DCR z racji liniowej charakterystyki przenoszenia, niewielkiej sumarycznej pojemności tranzystorów w transoptorach i dobremu odseparowaniu sygnałów sterujących od sygnału modulowanego może znaleźć zastosowanie w modulacji amplitudowej sygnałów wielkiej częstotliwości.

4. Tanie i wszechobecne EL817C robią robotę.

[skocz do spisu treści](#spis)

<a name="etap1i"></a>
### 06.03.2026

Zamiast programowego wykrywania przejścia przez 0 zastosuję jednak przerzutnik Schmitta na tranzystorach. Ogranicznikiem napięcia na podwajaczu będzie biała LED co da napięcie około 3V na pinie AREF. Napięcie na LEDzie będzie wraz z temperaturą dryfowało razem z napięciami progowymi przerzutnika Schmitta więc układ będzie mniej wrażliwy na zmiany temperatury. Alternatywnie można zastosować diodę Zenera i wzmacniacz operacyjny. LED jednak podoba mi się bardziej bo 1. obcina "miękko", 2. będzie sygnalizował jasnością świecenia moment mijania dwóch "wybitych zębów". LED będzie "podciągnięta" do plusa rezystorem o wartości kilku kiloohmów żeby napięcie na AREF od początku (tzn. przy zatrzymanym wale) było zbliżone do 3V.

Tak więc mikrokontroler sterując DCR-em (Digital Controlled Resistor) będzie starał się utrzymać stałą amplitudę sygnału na wejściu przerzutnika mimo zmieniającej się szybkości obrotowej wału. Zmiana oporu DCR-a będzie następowała raz na obrót wału, w szczelinie czasowej wyznaczonej przez przelot dwóch "wybitych zębów" czyli wraz ze wzrostem szybkości obrotowej będzie następowało zmniejszenie stałej czasowej ART (Automatycznej Regulacji Tłumienia).

Zielony - sygnał wejściowy przerzutnika (3V p-p), niebieski - sygnał wyjściowy przerzutnika (5V p-p).

![schmitt trigger]({{ site.baseurl }}/assets/images/szmit.jpg)

EL817 jest powolny więc lepiej będzie go mocniej wysterować i dać niższą rezystancję na wyjściu:

![schmitt trigger fast]({{ site.baseurl }}/assets/images/szmitf.jpg)

[skocz do spisu treści](#spis)

<a name="etap1j"></a>
### 08.03.2026

Poskładałem shit trigger. ;)

Q1 - SS8050, Q2 - 2SC3330, Q3 - BD136-16.

![schmitt trigger]({{ site.baseurl }}/assets/images/schittr.jpg)

Układ nie wymaga żadnych poprawek poza zmianą BD136-16 na coś o mniejszych gabarytach. Przełącza się przy napięciach 1.8V i 1.2V czyli symetrycznie wokół 1.5V z odstępem 0.3V. Ideolo.

Filmik z pomiaru:

https://www.youtube.com/shorts/BL7iWC0DEsE

[skocz do spisu treści](#spis)

<a name="etap1k"></a>
### 15.03.2026

Wzbogacam domenę publiczną.

![schemat kondycjonera]({{ site.baseurl }}/assets/images/ksrcpwk12026.jpg)

[skocz do spisu treści](#spis)

<a name="etap1l"></a>
### 17.03.2026

Obraz miedzianych ścieżek kondycjonera. Tym razem brałem pod uwagę wygodę montażu.

![kondycjoner obraz ścieżek]({{ site.baseurl }}/assets/images/m12026.png)

[skocz do spisu treści](#spis)

<a name="etap1m"></a>
### 30.03.2026

Rysuję sobie ścieżki przetwornicy. Zdecydowałem się na 3 izolowane wyjścia. 8V dla kondycjonera, 12V dla górnych N-FETów i 20V dla bramek tranzystorów z węglika krzemu w module zapłonowym.
5V będzie nieizolowane (defacto jest izolowane od 3 wymienionych wcześniej...). Napięcie na izolowanych wyjściach jest stabilizowane na tej samej zasadzie jek to się dzieje w wielowyjściowych flybackach.
Przy 12V i 20V zastosuję dodatkowo stabilizatory liniowe na płycie przetwornicy. Dławik będzie miał 4 oddzielne uzwojenia nawinięte na toroidalnym rdzeniu proszkowym.
Z powodu niewielkiej mocy przenoszonej przez przetwornicę rolę radiatora spełni pole miedzi do którego będzie przylutowany thermal pad układu HTC2576. 

![smps obraz ścieżek]({{ site.baseurl }}/assets/images/przetwopis.jpg)

[skocz do spisu treści](#spis)

<a name="etap1n"></a>
### 08.04.2026

Nowa (proto 2) płytka kondycjonera po trawieniu i cynowaniu.

![kondycjoner obraz ścieżek]({{ site.baseurl }}/assets/images/kondp.jpg)

Różnice w stosunku do poprzedniej wersji to 1 - dodany sprzętowy przerzutnik, 2 - dodana jedna para transoptorowa (czyli DCR jest sześciobitowy), 3 - znacznie lepsze rozłożenie przestrzenne elementów.

Proto 2 jest bardzo bliskie wersji ostatecznej.

[skocz do spisu treści](#spis)

<a name="etap1o"></a>
### 13.04.2026

Płytka (proto 1) przetwornicy po trawieniu i cynowaniu.

![flyboost]({{ site.baseurl }}/assets/images/flyboost.jpg)

Zastanawiałem się nad nazwą topologii tej przetwornicy. To będzie chyba jakiś flyback.

[skocz do spisu treści](#spis)

<a name="etap1p"></a>
### 19.04.2026

Częściowo złożyłem, uruchomiłem i wstępnie przetestowałem flyback-a, którego zabawowo nazwałem Piotras Converter-em. Wygląda na to, że działa miodzio. Zamieniłem niektóre elementy np. wsadziłem STP16NF06 zamiast staromodnego IRFZ14 i wyliczyłem, że optymalną indukcyjnością dławika będzie jednak 250 µH.

Dławik typu "ferrytowa klepsydra" zapewne nieźle sieje ale kogo tam obchodzi kompatybilność elektromagnetyczna ;)

![piotras converter]({{ site.baseurl }}/assets/images/pconv0.jpg)

![piotras converter]({{ site.baseurl }}/assets/images/pconv1.jpg)

![piotras converter]({{ site.baseurl }}/assets/images/pconv2.jpg)

![piotras converter]({{ site.baseurl }}/assets/images/pconv3.jpg)

W następnych wpisiach przedstawię pomiary już kompletnego układu oraz uaktualniony schemat.

[skocz do spisu treści](#spis)

<a name="etap1r"></a>
### 02.05.2026

Aktualny schemat elektryczny proto 1 przetwornicy:

![przetwornica schemat]({{ site.baseurl }}/assets/images/boosttp1c.png)

Uwzględniłem w nim już wnioski po dzisiejszych pomiarach czyli nieizolowane napięcie wyjściowe zostało podniesione do 14.6V (R5, R6, R7) oraz zwiększono wartość rezystorów przy ledach.
Z pojemnością C7 i C6 można spokojnie zejść do odpowiednio 22uF i 47uF, będzie wtedy mniej "bulky".

![przetwornica bok]({{ site.baseurl }}/assets/images/pcdlaw.jpg)

Większe dławiki łatwo się nawija - to ważne w produkcji jednostkowej, dlatego też w przetwornicy siedzi LM2576 o częstotliwości pracy zaledwie 52 KHz. Uzwojenia odizolowano papierową taśmą. L2 i L3 nawinięte są obok siebie. Nawinięcie jednego na drugim byłoby błędem. L1 jest pod L2 i L3 i jest nawinięte na całej szerokości. Nawinięty dławik oczywiście idzie do suszenia i próżniowego zalania masą epoxy. Ferrytowa klepsydra wzięta została z elektrośmieci. Niestety nie wiem co to za materiał. Rzeń robi się delikatnie ciepły podczas pracy, na obrazie z oscyloskopu nie widać objawów nasycania się więc jest ok. Póżniej jeszcze pomierzę ten dławik w -20 i w + 80 st. C.

![przetwornica bok]({{ site.baseurl }}/assets/images/pcbok1.jpg)

T1 mógłby leżeć. Nie nagrzewa się więc niepotrzebnie tak sterczy.

![przetwornica bok]({{ site.baseurl }}/assets/images/pcbok2.jpg)

![przetwornica bok]({{ site.baseurl }}/assets/images/pcbok3.jpg)

"Bomble" C6 i C7 do wymiany na mniejsze. Sorci. Podczas składania miałem ograniczony dostęp do elektrośmieci. :)

![przetwornica bok]({{ site.baseurl }}/assets/images/pcbok4.jpg)

![przetwornica dol]({{ site.baseurl }}/assets/images/pcdol.jpg)

Będą zmiany w obrazie ścieżek miedzianych.

![przetwornica gora]({{ site.baseurl }}/assets/images/pcgora.jpg)

Przyznacie, że z góry wygląda stylowo. ;) ;) ;).

Plus i minus sondy przyłączone do L1. Przebieg jak z baśni o idealnym flybacku:

![przetwornica oscyloskop pod obciążeniem]({{ site.baseurl }}/assets/images/pcload.jpg)

Plus i minus sondy przyłączone do L1. Przetworniczka musi pracować z pewnym minimalnym obicążeniem na wyjściu nieizolowanym inaczej wydajność prądowa wyjść izolowanych robi się bardzo mała.
Ładowanie kondensatorów wyjść izolowanych zachodzi wraz z zanikaniem pola magnetycznego w dławiku (ujemna część przebiegu napięcia na L1):

![przetwornica oscyloskop bez obciążenia]({{ site.baseurl }}/assets/images/pcnoload.jpg)

No i jeszcze stanowisko pomiarowe:

![blat z gratami]({{ site.baseurl }}/assets/images/pcstol.jpg)

Ciekawa topologia. Najbardziej nadaje się do podnoszenia napięcia ponieważ w fazie zanikania pola magnetycznego prąd płynie przez aż dwie diody (czyli są duże straty mocy przy niskich napięciach wyjściowych) z których tylko D4 musi wytrzymywać napięcie wyjściowe. Oczywiście przy wysokich napięciach na wyjściu T1 musi być wysokonapięciowy.

Zrobione pomiary:

(WE - napięcie wejściowe, WY - napięcie wyjściowe, SiC - napięcie zasilania gate driver-a w module zapłonowym, SSC - napięcie zasilania kondycjonera sygnału) 

Pomiar 1:

Obiążony tylko WY.

WE: 12.3 V

WY: 11.62 V @ 140 mA

SiC: 19.51 V @ 0 mA

SSC: 7.84 V @ 0 mA

Pomiar 2:

Obciążono WY i SiC.

WE: 12.3 V

WY: 11.63 V @ 141 mA

SiC: 14.54 V @ 110 mA

SSC: 6.45 V @ 0 mA

Pomiar 3:

Obciążono WY i SSC.

WE: 12.3 V

WY: 11.62 V @ 141 mA

SiC: 18.49 V @ 0 mA

SSC: 6.19 V @ 40 mA

Wnioski z pomiarów: 

Podnieść WY z 11.62V do 14.6V (za czym polecą w górę napięcia na SiC i SSC) i będzie dobrze. Wydajność prądowa wystarczająca. Przetwornica pracuje stabilnie. Nie nagrzewa się.
Oscylogramy ładne.

Radio FM w odległości kilku metrów od urządzenia działa normalnie. W zakresie SW sprawdziłem oscyloskopem i może być (dopuki ktoś zza ściany nie kontaktuje się radiotelegrafią z Murzynami z południowej Afryki). LW i MW nikt nie słucha więc po co sprawdzać ;) ;).


Rafleksja na boczku: 

Tak BTW to LM2576 ma maksymalne wypełnienie bardzo blisko 100%. Załóżmy, że max duty cycle to 95%. W takim wypadku przy 50 V na wejściu otrzymamy 1 KV na wyjściu. Current limit w LM2567 to około 5 A. Przeniesiona moc to będzie jakieś 200W. 1 KV @ 200W przy niezłej sprawności, niskim koszcie i "przerażającej" prostocie układu to cechy nadające sens zastosowaniu go w amatorskiej robotyce do zasilania siłowników opartych o np. elastomeryczne dielektryki. Przy niższych napięciach dostępnych na platformie dławik nawinąłbym bi albo nawet tri-filarnie po czym szeregowo połączył uzwojenia przy czym dren T1 podłączyłbym do końcówki uzwojenia, którego początek dotyka nóżki U1. Ogólnie przedstawiona topologia przy użyciu taniutkiego LM2576 daje dużo możliwości w uzyskiwaniu wysokich, stabilnych napięć wyjściowych zapewniając jednocześnie zabezpieczenie przed zwarciem i przeciążeniem. 

[skocz do spisu treści](#spis)
















