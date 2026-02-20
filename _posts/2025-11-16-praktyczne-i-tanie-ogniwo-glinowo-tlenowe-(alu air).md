---
layout: post
title:  "Droga do praktycznego wykorzystania ogniwa glinowo-tlenowego. Pół-bateria-pół-ogniwo paliwowe w akompaniamencie popiskujących niskorezystancyjnych kluczy."
author: MF
categories: [ elektrochemia, energoelektronika ]
image: assets/images/alulogofin.jpg
tags: [ featured ]
---

<a name="zero"></a>
Wstęp:

Dobrze zaprojektowane ogniwo glinowo-tlenowe "mieści w sobie" więcej energii w przeliczeniu na jednostkę masy niż najlepszy pod tym względem aktualnie masowo produkowany akumulator litowo-jonowy.
Atom glinu może oddać aż 3 elektrony a sam materiał jest lekki. Katoda "spożywa" tlen z powietrza więc masa tego reagenta nie musi być brana pod uwagę.
Gdyby alu-air było zdolne do dostarczenia tak dużej mocy w przeliczeniu na wagę jak li-poly to automatycznie i natychmiastowo znalazłoby zastosowanie w dronach bojowych no bo tam przecież nikogo nie obchodzi ładowanie baterii ogniw po rozpadzie maszyny. Niestety wąskim gardłem pozostaje tempo reakcji redukcji tlenu w temperaturze pokojowej na katodzie.

Czy więc konstruowanie alu-air to sztuka dla sztuki? Niekoniecznie. Czasami znajduję się w sytuacji braku możliwości korzystania z prądu z sieci przez kilka dni przy czym nie chcę hałasować spalinowym agregatem prądotwórczym. Cena agregatów prądotwóczych jest z resztą teraz niemała. Bardzo zadowolony byłbym z urządzenia wielkości wiaderka potrafiącego cichutko i bez smrodu oddać kilka kilowatogodzin w ciągu powiedzmy trzech dni.

Praca przedstawiona w tym arcie będzie polegała na skonstruowaniu takiego ogniwa z dostępnych materiałów i w cenie niższej niż najtańszy spalinowy agregat prądotwórczy.
Planuję wykonać tylko jedną celę o dużej powierzchni ponieważ wtedy łatwo wymieniać materiał anodowy (wsadzić będzie trzeba tylko jeden kawał blachy).
Konsekwencją zastosowania jednej celi jest konieczność zaprojektowania wysokosprawnej przetwornicy podwyższającej napięcie co przy napięciu na ogniwie wynoszącym około 1V z pewnością nie będzie zadaniem trywialnym. Sądzę jednak, że przy użyciu topologii push-pull z odczepem i prostownikiem na tranzystorach polowych jest możliwe przekroczenie 80% sprawności (przy maksymalnym obciążeniu) bez używania egzotycznych materiałów ani drogich półprzewodników.

Spis treści:

16.11.2025 - Wstęp. Co, po co i jak. [skocz](#zero) 

02.02.2026 - Postawy teoretyczne i wyniki dotychczasowych badań eksperymentalnych. Pochwalenie się tranzystorem. [skocz](#jeden) 

11.02.2026 - Suszenie Adblue i zarys planu konstrukcji elektrody dodatniej. [skocz](#dwa) 


<a name="dwa"></a>
### 11.02.2026

Odparowywanie wody z płynu Adblue do diesli w celu otrzymania krystalicznego mocznika:

![walking]({{ site.baseurl }}/assets/images/alu0.jpg)

Mocznik potrzebny do otrzymania dopingowanej azotem grafitopodobnej substancji z osadzonym na niej tlenkiem kobaltu, pełniącej funkcję katalizatora reakcji redukcji tlenu.

Mój plan to utworzenie związku kompleksowego metforminy z kobaltem, dalej hydrotermalna obróbka mieszaniny tego związku z mocznikiem i kwasem cytrynowym. Następnie, po zmieszaniu z sproszkowanym grafitem, piroliza w temperaturze 700 st. C.

Podobnie jak to co opisane tutaj:

https://www.sciencedirect.com/science/article/abs/pii/S0013468615302590#:~:text=Preparation%20of%20N%2Ddoped%20porous%20carbons%20In%20a,filtration%2C%20washed%20with%20distilled%20water%2C%20and%20dried.

czyli:

"In a typical process, 8.75 g (41.7 mmol) citric acid and 7.5 g (125 mmol) urea were dissolved in 20 mL of water. Subsequently, 20 mL of the solution was sealed in a 35 mL microwave tube and heated with a maximum microwave irradiation power of 800 W for 5 min (optimized conditions) using a microwave system (CEM MARS6). The resulting carbonaceous solid, denoted as hydrochar, was recovered by filtration, washed with distilled water, and dried.
The dried hydrochar (2 g) was then carbonized at 700 °C "

W kuchni mam niezły microwave system. ;-)

Produkt po zmieleniu zmieszam z minimalną ilością polichloroprenu rozpuszczonego w toluenie.
Powstałym płynnym jeszcze materiałem elektrodowym powlekę drobny miedziany poniklowany drut i wysuszę. Po odparowaniu toulenu powstanie stała masa zaciśnięta na drucie (bo materiał zmniejszy objętość po ewaporacji rozpuszczalnika).
Powierzchnia tej masy zostanie dodatkowo poddana obróbce ciernej.
Takie druciki ułożone jeden przy drugim utworzą płaską elektrodę o ogromnej powierzchni czynnej i ekstremalnie niskiej rezystancji wewnętrznej.
Oba końce każdego drucika będą połączone poprzez lutowanie z current collectorem. Więc w sumie będą 2 current collectory po przeciwległych stronach drucikowego sheet-a.
Drucikowy sheet będzie dociśnięty do perforowanej (liczne bardzo drobne otwory) silikonowej błony.
To combo z jednej strony będzie stykać się z powietrzem a z drugiej strony z elektrolitem.

Plan jest w miarę prosty. Zastanawiam się jak to zrobić by się nie przepracować. Mogę niklować wiele odcinków drutu jednocześnie, mogę przeciągać drut przez roztwór do galwanizowania...
"Farbę" mogę nakładać pędzelkiem na już ułożone i przylutowane druty...Obróbka cierna mogłaby odbywać się fluidalnie...

Ale jak wykonać perforowaną silikonową błonę? A gdybym zaimpregnował silikonem drobną tkaninę?

<a name="jeden"></a>
### 02.02.2026

Dzisiaj tylko kilka linków z materiałami wiążącymi się z tematem projektu i jedna fotka: 

## Elektrochemia:

https://advanced.onlinelibrary.wiley.com/doi/full/10.1002/advs.202304214
https://backend.orbit.dtu.dk/ws/portalfiles/portal/247862975/1_s2.0_S0013468621010811_main.pdf
https://www.cell.com/cell-reports-physical-science/fulltext/S2666-3864(25)00288-7
https://www.nature.com/articles/s41467-020-15416-4
https://www.ustl.edu.cn/__local/4/9A/6A/8F738AD20C0087F58DE2B92BFF9_52F96293_FF7E5.pdf?e=.pdf
https://www.mdpi.com/1420-3049/28/24/8136
https://www.researchgate.net/publication/368970006_Precipitation-free_aluminum-air_batteries_with_high_capacity_and_durable_service_life

## Energoelektronika:

Przetwornica rozruchowa:

https://web.archive.org/web/20171030234220/http://flagiusz.republika.pl/joulethief/sim.html

Dokumentacja AD136 VI: https://www.mgelectronic.rs/ProductFilesDownload?Id=7821

Fotka zdobycznego AD136 VI:

![walking]({{ site.baseurl }}/assets/images/ad136.jpg)


Cieszę się z pokazanego germanowca ponieważ chcę by wstępna przetwornica startowała samoczynnie po dołączeniu ogniwa i by jej wydajność po starcie była wystarczająca do wysterowania drivera bramek FET-ów i zasilenia mikrokontrolera. AD136 VI powinien z zapasem spełnić oczekiwania już przy napięciu zasilania wynoszącym 0.3V. Nie dość, że germanowy to jeszcze niemiecki :).

Przetwornica właściwa:

https://www.researchgate.net/publication/319367432_Low_voltage_single_fuel_cell_interface_by_Push-Pull_converter_A_case_of_study

