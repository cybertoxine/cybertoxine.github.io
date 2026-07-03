---
layout: post
title:  "Ucieczka z Arduino na drodze wywoływania Ducha Polskiego Aptekarstwa."
author: MF
categories: [przemiany fazowe, elektromechanika, mistycyzm, GCBASIC ]
image: assets/images/htszczyt.jpg
tags: [featured]
---

Wstęp:

Po co komu w domu destylator pracujący pod zmniejszonym ciśnieniem? Jak dla mnie to potrzebne AGD. Mam zamiar używać maszynki do zagęszczania soku z czarnego bzu i ekstraktu z owoców dzikiej róży (mniej miejsca w zamrażarce zajmie) oraz do suszenia mięsa (alternatywa dla wędlin). Zmniejszenie ciśnienia pozwala na obniżenie temperatury wrzenia cieczy. Już przy temperaturze wsadu wynoszącej 60°C maszynka będzie bardzo szybko i praktycznie do końca usuwać wodę powodując wrzenie (soczki, ekstrakty) albo przyspieszone parowanie (mięso, owoce).

Ponieważ Qualcomm kupił Arduino i rości sobie różne prawa do twórczości ludzi działających na tej platformie Arduino nie dotykam.
Szukałem alternatyw i znalazłem GCBASIC. GCBASIC ze wstawkami asemblerowymi wydaje mi się na ten moment rozwiązaniem optymalnym dla programowania 8-bitowych mikrokontrolerów.

https://gcbasic.com/

Więc przy okazji poznam (i przedstawię Wam) nowe przydatne narzędzie programistyczne.

### 03.07.2026

Skraplacz uszczelniony. Maszynka chodzi. Jeszcze tylko umieścić czujniki temperatury w docelowych miejscach, podłączyć pompę próżniową, podeprzeć pokrywę od środka (żeby podciśnienie nie spowodowało jej pęknięcia) i można suszyć mięso. Implozja pojemnika tej wielkości potrafi zrobić krzywdę szczególnie gdy elementy pojemnika są zrobione ze szkła, jak w moim projekcie, dlatego nie polecam kopiowania pomysłu bo można skończyć ze szkłem w pupie albo w oku.

Na filmiku otrzymuję filiżankę wody destylowanej.

https://www.youtube.com/watch?v=qicAs3IY6lc


### 01.07.2026

Sterownik zrobiłem na Attiny44A i PT6964-S. Sterownik wyświetla wynik pomiaru temperatury z czterech czujników (diody krzemowe spolaryzowane w kierunku przewodzenia),
daje możliwość regulacji mocy pompy wody, wentylatora chłodnicy i grzałki w kotle. Później jeszcze "dokoduję" jakieś funkcję poprawiające bezpieczeństwo pracy przy maszynce.
Użycie mikrokontrolera w perspektywie pozwoli na ustalenie programów (jak w pralce) dzięki czemu obsługa maszynki będzie ograniczała się do załadowania pojemnika, wybrania programu i naciśnięcia przycisku start. Nie trzeba będzie jej pilnować i co chwilę regulować parametrów żeby np. nie przypalić kawałka wołowiny.
Płytka z wyświetlaczem, przyciskami i układem PT6964-S pochodzi z elektrośmieciowego odtwarzacza DVD w którym znajdowała się płyta z filmem o Królewnie Śnieżce. Do PT6964-S bardzo podobny jest STLED316 z tym, że ten drugi łatwiej dostępny.

Dokumentacja PT6964: https://www.electronicsdatasheets.com/download/52dfa14fe34e24792f7c543e.pdf%3Fformat%3Dpdf

Dokumentacja STLED316: https://www.st.com/resource/en/datasheet/stled316s.pdf

Te banalne w obsłudze układziki bardzo ułatwiają tworzenie różnych prostych urządzonek. Sterują segmentowymi ledowymi wyświetlaczami, odczytują stany przycisków a z mikrokontrolerem komunikują się magistralą szeregową. "Zacni kompani" prostszych 8-bitowców.

Na filmiku widać o co chodzi w maszynce:

https://www.youtube.com/shorts/HMj3wD3re7k

Nie podłączałem jeszcze kotła i pompy próżniowej, czujniki temperatury wiszą sobie na kablach. Nieszczelny płaszcz wodny - końcówka skraplacza z której sączy się woda jest do poprawy.

Poniżej kod napisany w BASCOM-AVR DEMO 2.0.7.9. Później napiszę to samo w GCBASIC i okaże się który bejzikowy kompilator produkuje lepszy kod wynikowy.

```
$regfile = "attiny44.dat"
$hwstack = 32
$swstack = 16
$framesize = 64
$eepromhex
$crystal = 16500000

'----stale----

'### cyfry z ledow
const cimno = &B00000000
const dig0 = &B00111111
const dig1 = &B00000110
const dig2 = &B01011011
const dig3 = &B01001111
const dig4 = &B01100110
const dig5 = &B01101101
const dig6 = &B01111101
const dig7 = &B00000111
const dig8 = &B01111111
const dig9 = &B01101111

'### litery z ledow
const letb = &B01111100 'b jak boiler
const letc = &B01011000 'c jak coolant
const leti = &B00010000 'i jak in
const leto = &B01011100 'o jak out
const letf = &B01110001 'f jak flow
const leth = &B01110110 'h jak heater

'### do konfigurowania displaju:
const dimode0 = &B00000000 '4 cyfry 13 seg
const dimode1 = &B00000001 '5 cyfr 12 seg
const dimode2 = &B00000010 '6 cyfr 11 seg
const dimode3 = &B00000011 '7 cyfr 10 seg
const didataset0 = &B01000100 ' normal mode, write do display, fixed adress
const didataset1 = &B01000010 ' normal mode, read key data,
const dictrl0 = &B10001111 ' ON, 14/16 duty

'----zmienne w SRAM----
Dim bytetosend as byte
Dim bytetosendpointer as byte
Dim nadawaj as bit
Dim skip as bit
Dim aktywator as bit 'setter wysylania
Dim dezaktywator as bit
Dim papa as bit
Dim zrobione0 as bit
Dim kontyn as bit
Dim kounter as word
Dim wartoscdz As String * 4
Dim asci as byte
Dim leddot as byte
Dim tysiace as byte
Dim setki as  byte
Dim dziesiatki as byte
Dim jednosci as byte
Dim karuzela as byte
Dim zo as bit ' przelacznik zapis/odczyt do pt6964
Dim pakiet0 as byte
Dim pakiet1 as byte
Dim pakiet2 as byte
Dim pakiet3 as byte
Dim pakiet4 as byte
Dim pakietpointer as byte
Dim pakietnumber as byte
Dim stanbutonowa as byte
Dim stanbutonowb as byte
Dim bumelant as bit 'znacznik czasu w jakim dzialaja petle delayowe
Dim wynikzadc as word
Dim temperatura as word
Dim oznaczenie as byte
Dim starystanbutonowa as byte
Dim starystanbutonowb as byte
Dim nochange as bit
Dim butzero as bit
Dim buzic as byte
Dim korektadropu as byte
Dim heatpower as byte
Dim coolerpower as byte
Dim heatpowerpwm as word
Dim heatpowerpwm8 as byte
Dim temporpower as bit
Dim floworheat as bit
Dim flowpower as byte
Dim powershow as byte
Dim flowpowerh as byte
Dim flowpowerl as byte



'----konfiguracja peripherials----
Config Timer0 = Timer , Prescale = 8 '8 to tik co okolo 110 us przy RC max speed czyli przy togglowaniu to okres SCK wyniesie 220 us czyli jakies 4 KHz, 2048 cykli rdzenia od tiku do tiku
Enable Interrupts
Enable Timer0
Timer0 = 0
Start Timer0
On Timer0 dispandbutt
Config Adc = Free , Prescaler = 128 , Reference = Internal
Admux = &B10000000 '&B10000000 -t0, &B10000001 -t1, &B10000010 -t2, &B10000011 - t3
Adcsrb = &B00000000  'ADLAR = 0, ADCH - lewa czesc worda, ADCL - prawa czesc worda
Enable Adc
On Adc pomiar

'### 16 KHz 10 bit fast pwm na OC1A (PA6) bez przerwan
TIMSK1 = &B00000000
TCCR1A = &B10000011
TCCR1B = &B00001001
TCCR1C = &B00000000

' przy zmienianiu wartoci pwm najpierw do OCR1AH a nastepnie do OCR1AL wpisywac wartosci bo dostep jest zrobiony z uzyciem TEMP

'### we/wy
DDRB = &B00000111 'DI/O - pb0, CLK - pb1, STB - pb2
DDRA = &B11100000 'Te0(wsad) - pa0, Te1(wej. skr.) - pa1, Te2(kon. skr.) - pa2, Te3(zb. wyr.) - pa3, pom. cisn. w kotle - pa4, PWMheater - pa5, PWMpump+blower - pa6, glosnik - pa7

'### gladkie, stopniowe ustawienie zegara na MAX (okolo 16.5 MHz)
osccal=  128
While osccal < 255
incr osccal
Wend

'---kod przed mainloopem wykonywany jednokrotnie na starcie---
zo = 0
PORTB.1 = 1
PORTB.2 = 1
gosub delay5
bytetosend = dimode0
aktywator =  1
gosub delay5
bytetosend = didataset0
aktywator =  1
gosub delay5
bytetosend = dictrl0
aktywator =  1
kounter =   1000
oznaczenie = letb 'po resecie pokazuje temperature boilera
korektadropu = 12
heatpower = 0 'na wszelki wypadek po resecie zero grzania

'---mainloop---'
Do
gosub disprender 'generowanie obrazu dla wyswietlacza
gosub sendtodisp 'wysylanie obrazu do PT
gosub przyciski  'odczyt stanu przyciskow i zapisanie go w 2 bajtach
gosub buttexe 'reakcja na nacisniecie przycisku
Loop

pomiar:
Out &H007B , ADCL     'wrzucenie wyniku pomiaru na adres zmiennej wynikzadc
Out &H007C , ADCH
If buzic < 255 then
Toggle PORTA.7
incr buzic
else
PORTA.7 = 0
Endif
Return

buttexe:
If starystanbutonowa = stanbutonowa AND starystanbutonowb = stanbutonowb Then
nochange = 1
Else
nochange = 0
Endif
If stanbutonowa = 0 AND stanbutonowb = 0 Then
butzero = 1
Else
butzero = 0
Endif
If butzero = 1 OR nochange = 1 Then
starystanbutonowa = stanbutonowa
starystanbutonowb = stanbutonowb
Return
Endif
incr buzic
If stanbutonowa.1 = 1 Then
oznaczenie = letb
Admux = &B10000000
korektadropu = 12
temporpower = 0
Elseif stanbutonowb.1 = 1 Then
oznaczenie = leti
Admux = &B10000001
korektadropu = 12
temporpower = 0
Elseif stanbutonowa.3 = 1 Then
oznaczenie = leto
Admux = &B10000010
korektadropu = 16
temporpower = 0
Elseif stanbutonowa.7 = 1 Then
oznaczenie = letc
Admux = &B10000011
korektadropu = 21
temporpower = 0
Elseif stanbutonowb.2 = 1 Then  'select pompki
oznaczenie = letf
floworheat = 0
temporpower = 1
powershow = flowpower
Elseif stanbutonowb.4 = 1 Then  'select heater
oznaczenie = leth
floworheat = 1
temporpower = 1
powershow = heatpower
Endif
If temporpower = 1 AND floworheat = 0 AND stanbutonowa.6 = 1 Then  'przysp. pompek
flowpower =  flowpower + 16
powershow = flowpower
flowpowerh.1 = flowpower.7       'wbicie danych do compare A timera 1 (10-bit PWM)
flowpowerh.0 = flowpower.6
flowpowerl = flowpower
Shift flowpowerl, Left, 2
OCR1AH = flowpowerh
OCR1AL = flowpowerl
Endif
If temporpower = 1 AND floworheat = 0 AND stanbutonowb.0 = 1 Then 'spowol. pompek
flowpower  =  flowpower - 16
powershow = flowpower
flowpowerh.1 = flowpower.7      'wbicie danych do compare A timera 1 (10-bit PWM)
flowpowerh.0 = flowpower.6
flowpowerl = flowpower
Shift flowpowerl, Left, 2
OCR1AH = flowpowerh
OCR1AL = flowpowerl
Endif
If temporpower = 1 AND floworheat = 1 AND stanbutonowa.6 = 1 Then 'dawaj wiecej ciepla
heatpower  =  heatpower + 16
powershow = heatpower
Endif
If temporpower = 1 AND floworheat = 1 AND stanbutonowb.0 = 1 Then 'dawaj mniej ciepla
heatpower = heatpower - 16
powershow = heatpower
Endif
starystanbutonowa = stanbutonowa
starystanbutonowb = stanbutonowb
Return

przyciski:
gosub delay5 'wlacenie trybu odczytywania PT
bytetosend = didataset1
kontyn = 1
aktywator =  1
gosub delay5
DDRB.0 = 0 ' kierunek transmisji danych PT --> MCU
pakietnumber = 0 'zerowanie pointerow subrutyny readpt6964S
pakietpointer = 0
zo = 1 ' przelaczenie dispandbutt w tryb odczytywania
gosub delay5
stanbutonowa.0 = pakiet0.0 'MENU
stanbutonowa.1 = pakiet0.1 'PLAY
stanbutonowa.2 = pakiet0.3 'EXIT
stanbutonowa.3 = pakiet0.4 'STOP
stanbutonowa.4 = pakiet1.0 'BACK
stanbutonowa.5 = pakiet1.1 'RECORD
stanbutonowa.6 = pakiet1.3 'UP
stanbutonowa.7 = pakiet1.4 'REWIND
stanbutonowb.0 = pakiet2.0 'DOWN
stanbutonowb.1 = pakiet2.1 'FORWARD
stanbutonowb.2 = pakiet2.3 'LEFT
stanbutonowb.3 = pakiet2.4
stanbutonowb.4 = pakiet3.0 'RIGHT
stanbutonowb.5 = pakiet3.1
stanbutonowb.6 = pakiet3.3
stanbutonowb.7 = pakiet3.4 'OK
Return

sendtodisp:
gosub delay5 'wlacenie trybu wysylania do PT
bytetosend = didataset0
kontyn = 0
aktywator =  1
gosub delay5 ' wyslanie cyferek na odpowiedni adres
bytetosend = &B11000000
kontyn = 1
aktywator = 1
gosub delay5
bytetosend = tysiace
kontyn = 0
aktywator = 1
gosub delay5 '---
bytetosend = &B11000010
kontyn = 1
aktywator = 1
gosub delay5
bytetosend = setki
kontyn = 0
aktywator = 1
gosub delay5 '---
bytetosend = &B11000100
kontyn = 1
aktywator = 1
gosub delay5
bytetosend = dziesiatki
kontyn = 0
aktywator = 1
gosub delay5 '---
bytetosend = &B11000110
kontyn = 1
aktywator = 1
gosub delay5
bytetosend = jednosci
kontyn = 0
aktywator = 1
Return

disprender:
Out &H0065, 0 'zerowanie bajtow ktore funkcja str nadpisuje
Out &H0066 , 0
Out &H0067 , 0
Out &H0068 , 0
temperatura = wynikzadc
temperatura = 680 - temperatura
temperatura = temperatura/2
temperatura = temperatura + korektadropu
kounter = 1000 'trick niwelujacy left adjust funkcji str
kounter = temperatura + kounter
If temporpower = 1 Then
kounter = 1000 + powershow
Endif
wartoscdz = Str(kounter)
tysiace = oznaczenie
asci = Inp(&H0066) ' konwersja asci do ukladu ledow w wyswietlaczu
gosub ascitoled
setki = leddot
asci = Inp(&H0067)
gosub ascitoled
dziesiatki = leddot
asci = Inp(&H0068)
gosub ascitoled
jednosci = leddot
Return

ascitoled:
If asci = 48 Then
leddot = dig0                                                   '0
Elseif asci = 49 Then
leddot = dig1                                                    '1
Elseif asci = 50 Then
leddot = dig2                                                   '2
Elseif asci = 51 Then
leddot = dig3                                                   '3
Elseif asci = 52 Then
leddot = dig4                                                    '4
Elseif asci = 53 Then
leddot = dig5                                                    '5
Elseif asci = 54 Then
leddot = dig6                                                    '6
Elseif asci = 55 Then
leddot = dig7                                                   '7
Elseif asci = 56 Then
leddot = dig8                                                    '8
Elseif asci = 57 Then
leddot = dig9
Else
leddot = cimno
Endif

delay5:
bumelant = 1
Waitms 5 'PT6964-S ponizej 2ms nie kontaktowal juz
bumelant = 0
Return

dispandbutt: 'przerwanie timera0, 256X8, co 0.124 ms, jakies 8 KHz
If zo = 0 then
gosub writept6964s
else
gosub readpt6964s
Endif
incr heatpowerpwm
heatpowerpwm8 = Inp(&H0087)
if heatpowerpwm8 < heatpower Then 'software PWM grzalki, okres okolo 10 sekund, rozdzielczosc 256
PORTA.5 = 1
Else
PORTA.5 = 0
Endif
Return

writept6964S: 'zapisywanie do PT6964-S, wyzwalane w rytm timera0
If papa = 1 and kontyn = 0 Then
PORTB.2 = 1 ' strobe w gore
papa = 0
zrobione0 = 1
Endif
If papa = 1 and kontyn = 1 Then
PORTB.2 = 0 ' strobe dalej na dole
papa = 0
zrobione0 = 1
Endif
If dezaktywator = 1 Then
dezaktywator = 0
PORTB.1 = 1 'clk w gore
papa = 1
Endif
If skip = 1 then
nadawaj = 1
aktywator = 0
skip = 0
Endif
if aktywator = 1 then
skip = 1
PORTB.2 = 0 ' strobe w dol
Endif
If nadawaj = 1 then
Toggle PORTB.1
Endif
If nadawaj = 1 and PORTB.1 = 0 Then
PORTB.0 = bytetosend.bytetosendpointer
incr bytetosendpointer
Endif
if nadawaj = 1 and bytetosendpointer = 8 Then
bytetosendpointer = 0
nadawaj = 0
dezaktywator = 1
Endif
Return

readpt6964S: 'odczyt z PT, wyzwalane w rytm timera0
Toggle PORTB.1
If PORTB.1 = 1 and pakietnumber = 0 Then
pakiet0.pakietpointer = PINB.0
incr pakietpointer
Elseif PORTB.1 = 1 and pakietnumber = 1 Then
pakiet1.pakietpointer = PINB.0
incr pakietpointer
Elseif PORTB.1 = 1 and pakietnumber = 2 Then
pakiet2.pakietpointer = PINB.0
incr pakietpointer
Elseif PORTB.1 = 1 and pakietnumber = 3 Then
pakiet3.pakietpointer = PINB.0
incr pakietpointer
Elseif PORTB.1 = 1 and pakietnumber = 4 Then
pakiet4.pakietpointer = PINB.0
incr pakietpointer
Endif
If pakietpointer = 8 Then
pakietpointer = 0
incr pakietnumber
Endif
If pakietnumber = 5 Then
pakietpointer = 9
pakietnumber = 6
kontyn = 0
PORTB.2 = 1 ' strobe w gore
DDRB.0 = 1 ' przywrocenie kierunku DI/O na MCU --> PT
zo = 0 'teraz dispandbutt bedzie robic wysylanie
Endif
Return

$eeprom                                                     'uwaga na autokasowanie epromu
Dta:
Data 0 , dig0 , dig1, dig2, dig3, dig4, dig5, dig6, dig7, dig8, dig9

```
