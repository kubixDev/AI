
# 📗 Projekt 2 - prolog


## ⭐ Zadanie wstępne
Zapisz reguły dla:
* nieprzyjazn(X,Y)
* niby_przyjazn(X,Y)
* loves(X,Y)
* true_love(X,Y) (dodaj fakt płeć)

<br>

**Rozwiązanie:**

```
% ==============================
% BAZA WIEDZY
% ==============================
mezczyzna(jan).
mezczyzna(pawel).
mezczyzna(krzysztof).
mezczyzna(michal).
mezczyzna(adam).

kobieta(maria).
kobieta(anna).
kobieta(julia).


lubi(pawel, krzysztof).
lubi(pawel, jan).
lubi(krzysztof, pawel).

lubi(jan, krzysztof).

lubi(adam, jan).
lubi(jan, adam).
lubi(adam, anna).
lubi(anna, adam).

lubi(michal, julia).
lubi(julia, michal).



% ==============================
% REGULY
% ==============================

% Przyjazn jest wtedy, gdy 'X' lubi 'Y' oraz 'Y' lubi 'X'
przyjazn(X, Y) :-
    lubi(X, Y),
    lubi(Y, X).

% Niby przyjazn (jednostronna sympatia) jest wtedy, gdy 'X' lubi 'Y' lub 'Y' lubi 'X'
niby_przyjazn(X, Y) :-
    lubi(X, Y);
    lubi(Y, X).

% Nieprzyjazn jest wtedy, gdy nie ma przyjazni miedzy osobami, 'X' nie lubi 'Y' oraz 'Y' nie lubi 'X'
nieprzyjazn(X, Y) :-
    \+ lubi(X, Y),
    \+ lubi(Y, X).

% Loves jest wtedy, gdy istnieje przyjazn między 'X' a 'Y' i nie ma innej osoby 'Z', z ktora "X" jest w przyjazni
loves(X, Y) :-
    przyjazn(X, Y),
    \+ (przyjazn(X, Z), Y \= Z).

% True love jest wtedy, gdy 'X' i 'Y' sie wzajemnie kochaja, nie ma innej osoby ktora kazde z nich lubi oraz gdy maja przeciwna plec (warunek 'or' zabezpiecza obie kombinacje wpisania)
true_love(X, Y) :-
    loves(X, Y),
    loves(Y, X),
    ((kobieta(X), mezczyzna(Y));
    (mezczyzna(X), kobieta(Y))).



% ==============================
% DOBRE PRZYKLADY
% ==============================

% Przyjazn:         pawel i krzysztof   (obaj sie lubia)
% Niby przyjazn:    jan i krzysztof     (jan lubi jednostronnie krzysztofa)
% Nieprzyjazn:      jan i maria         (sa dla siebie obcy)
% Loves:            anna i adam         (anna nikogo innego nie lubi, ale adam moze)
% True love:        michal i julia      (nie lubia nikogo poza soba, rozna plec)
```

<br>
<br>

## ⭐ Zadanie 1
Kim są dla siebie osoby x oraz y w przypadkach A-G (bazując na rysunkach)

<br>

**Rozwiązanie:**

<ol type="A">
  <li> <b> "x" oraz "y" są rodzeństwem </b> (wspólni rodzice) </li>
  <li> <b> "x" oraz "y" są kuzynami </b> (wspólny dziadek, ale inni rodzice) </li>
  <li> <b> "x" oraz "y" są dziadkami </b> (niepowiązani ze sobą, z dwóch rodzin) </li>
  <li> <b> "y" jest przybranym rodzicem dla "x" </b> (nie ma bezpośredniego powiązania) </li>
  <li> <b> "x" oraz "y" są przyrodnim rodzeństwem </b> (jeden z rodziców wspólny, np. wspólna matka i inni ojcowie) </li>
  <li> <b> "x" oraz "y" są szwagrami </b> (czyli są dla siebie jak np. mąż siostry albo brat żony) </li>
  <li> <b> "x" oraz "y" są kazirodztwem </b> </li>
</ol>

<br>
<br>

## ⭐ Zadanie 2
Załóżmy, że w pewnej bazie zdefiniowano wyłącznie predykaty: rodzic oraz mężczyzna.
Załóżmy ponadto, że zakresem rozważań jest zbiór osób – jako dodatkowy, trzeci predykat.
Opierając się na tych trzech predyklatach zdefiniuj w języku Prolog kolejno każdą z poniższych reguł.

Podczas tworzenia reguł nie można korzystać z definicji reguł, które nie zostały jeszcze w bazie podane.
Oznacza to, że:
* definiując regułę kobieta możemy korzystać wyłącznie z predykatów osoba, mezczyzna, rodzic
* definiując regułę ojciec możemy korzystać wyłącznie z predykatów osoba, mezczyzna, rodzic oraz reguły kobieta
* definiując regułę matka możemy korzystać wyłącznie z predykatów osoba, mezczyzna, rodzic oraz reguł kobieta, ojciec
itd.

<br>

*Jeżeli w przykładzie brakowało nazw konkretnych osób, to nazywałem je pierwszą literą dla skrócenia zapisu (np. O = ojciec)*

<br>

**Rozwiązanie:**

**kobieta(X)**
```
kobieta(X) :-
    \+ mezczyzna(X).
```

<br>

**ojciec(X,Y) – X jest ojcem Y**
```
ojciec(X, Y) :-
    mezczyzna(X),
    rodzic(X, Y).
```

<br>

**matka(X,Y) – X jest matką Y**
```
matka(X, Y) :-
    kobieta(X),
    rodzic(X, Y).
```

<br>

**corka(X,Y) – X jest córką Y**
```
corka(X, Y) :-
    kobieta(X),
    rodzic(Y, X).
```

<br>

**brat_rodzony(X,Y) – X jest rodzonym bratem Y**
```
brat_rodzony(X, Y) :-
    mezczyzna(X),
    ojciec(O, X),
    matka(M, X),
    ojciec(O, Y),
    matka(M, Y).
```

<br>

**brat_przyrodni(X,Y) – X jest przyrodnim bratem Y**
```
brat_przyrodni(X, Y) :-
    mezczyzna(X),
    (ojciec(O1, X),
    matka(M, X),
    ojciec(O2, Y),
    matka(M, Y),
    O1 \= O2);
    (ojciec(O, X),
    matka(M1, X),
    ojciec(O, Y),
    matka(M2, Y),
    M1 \= M2).
```

<br>

**kuzyn(X,Y) – X jest kuzynem Y**
```
kuzyn(X, Y) :-
    mezczyzna(X),
    rodzic(R1, X),
    rodzic(R2, Y),
    (brat_rodzony(R1, R2);
    brat_przyrodni(R1, R2)).
```

<br>

**dziadek_od_strony_ojca(X,Y) – X jest dziadkiem od strony ojca dla Y**
```
dziadek_od_strony_ojca(X, Y) :-
    mezczyzna(X),
    ojciec(O, Y),
    ojciec(X, O).
```

<br>

**dziadek_od_strony_matki(X,Y) – X jest dziadkiem od strony matki dla Y**
```
dziadek_od_strony_matki(X, Y) :-
    mezczyzna(X),
    matka(M, Y),
    ojciec(X, M).
```

<br>

**dziadek(X,Y) – X jest dziadkiem Y**
```
dziadek(X, Y) :-
    mezczyzna(X),
    (dziadek_od_strony_ojca(X, Y);
    dziadek_od_strony_matki(X, Y)).
```

<br>

**babcia(X,Y) – X jest babcią Y**
```
babcia(X, Y) :-
    kobieta(X),
    rodzic(R, Y),
    rodzic(X, R).
```

<br>

**wnuczka(X,Y) – Y jest wnuczką X**
```
wnuczka(X, Y) :-
    kobieta(Y),
    (dziadek(X, Y);
    babcia(X, Y)).
```

<br>

**przodek_do2pokolenia_wstecz(X,Y) – X jest przodkiem Y do drugiego pokolenia wstecz**
```
przodek_do2pokolenia_wstecz(X, Y) :-
    rodzic(X, R),
    rodzic(R, Y).
```

<br>

**przodek_do3pokolenia_wstecz(X,Y) - X jest przodkiem Y do trzeciego pokolenia wstecz**
```
przodek_do3pokolenia_wstecz(X, Y) :-
    rodzic(X, R),
    przodek_do2pokolenia_wstecz(R, Y).
```


<br>
<br>

## ⭐ Zadanie dla chętnych - labirynt
Pod postacią bazy danych zadany jest budynek który należy opuścić:
* Poszukiwania należy zawsze rozpocząć z pokoju wskazanego w poleceniu jako początkowy
* Przechodzić można pomiędzy pomieszczeniami jedynie gdy łączą je drzwi, czyli istnieje w bazie odpowiedni fakt
* Podjęcie próby przejścia należy sygnalizować komunikatem: [przechodze_z, a, do, b]
* Opuszczenie pomieszczenia (powrót do poprzedniego pomieszczenia) należy sygnalizować komunikatem: [wychodze_z, d]
* W niektórych pomieszczeniach można znaleźć klucz
* Kluczy może być więcej niż jeden. Nie każdy klucz musi otwierać jakieś drzwi
* Dla uproszczenia klucz otwierał będzie zawsze jedne drzwi
* To, gdzie znajdują się klucze określają fakty
* Jeżeli na taki klucz natrafimy, należy go zabrać i zasygnalizować to komunikatem: [znalazlem_klucz, kluczE]
* Niektóre klucze pozwalają na otwarcie powiązanych z nimi drzwi wyjściowych pokoju
* To z jakimi drzwiami powiązany jest dany klucz określa odpowiedni fakt

<br>

*Zakładam, że jedynymi drzwiami które trzeba otworzyć, są drzwi do wyjścia z całego labiryntu. Między pokojami można poruszać się bez przeszkód.
Dodatkowe klucze służą sprawdzeniu, czy odpowiedni z nich zostanie podniesiony we właściwym pokoju. Na stronie przedmiotu nie ma przykładowych danych,
dlatego stworzyłem własny labirynt do przetestowania programu (wizualizacja graficzna w oddzielnym pliku).*

<br>

**Rozwiązanie:**

```
% ==============================
% BAZA WIEDZY
% ==============================
drzwi(a, b).
drzwi(b, a).
drzwi(b, c).
drzwi(c, b).
drzwi(c, d).
drzwi(d, c).
drzwi(d, e).
drzwi(e, d).
drzwi(e, f).
drzwi(f, e).
drzwi(f, g).
drzwi(g, f).

klucz(d, kluczFalszywy).
klucz(f, kluczWyjsciowy).

otwiera(a, kluczWyjsciowy).



% ==============================
% REGULY
% ==============================
szukaj_wyjscia(POKOJ_POCZATKOWY, POKOJ_Z_KLUCZEM, KLUCZ, POKOJ_Z_WYJSCIEM) :-
    droga(POKOJ_POCZATKOWY, POKOJ_Z_KLUCZEM, [], KLUCZ),
    otwiera(POKOJ_Z_WYJSCIEM, KLUCZ).

droga(PunktStartowy, PunktStartowy, _, KLUCZ) :-
    klucz(PunktStartowy, KLUCZ),
    format('[znalazlem_klucz, ~w]~n', [KLUCZ]).

droga(PunktStartowy, PunktDocelowy, OdwiedzonePokoje, KLUCZ) :-
    drzwi(PunktStartowy, Pokoj),
    \+ member(Pokoj, OdwiedzonePokoje),
    format('[przechodze_z, ~w, do, ~w]~n', [PunktStartowy, Pokoj]),
    (klucz(Pokoj, KLUCZ),
    format('[znalazlem_klucz, ~w]~n', [KLUCZ]);
    droga(Pokoj, PunktDocelowy, [Pokoj|OdwiedzonePokoje], KLUCZ)),
    format('[wychodze_z, ~w]~n', [Pokoj]).
```

<br>

**Wyjaśnienie:**

> szukaj_wyjscia(POKOJ_POCZATKOWY, POKOJ_Z_KLUCZEM, KLUCZ, POKOJ_Z_WYJSCIEM)

* Ta reguła najpierw wywołuje regułę **'droga'**, aby znaleźć drogę z początkowego pokoju do pokoju z kluczem (wyjściowym)
* Przekazywana jest do niej pusta lista, ponieważ żadne pokoje nie zostały jeszcze odwiedzone
* Po znalezieniu drogi, wywoływana jest reguła **'otwiera'**, która sprawdza, czy znaleziony klucz otwiera drzwi wyjściowe. Jeśli tak, to znaleziona droga spełnia warunek zadania

<br>

> droga(PunktStartowy, PunktStartowy, _, KLUCZ)

* Jest to przypadek, gdzie punkt startowy jest tym samym punktem, w którym znajduje się klucz
* Reguła ta wyświetla informację o znalezieniu klucza

<br>

> droga(PunktStartowy, PunktDocelowy, OdwiedzonePokoje, KLUCZ)

* Jest to przypadek, gdzie rekurencyjnie przechodzimy przez pokoje, unikając tych odwiedzonych

<br>
<br>

**Szczegółowy opis drugiego przypadku:**

> drzwi(PunktStartowy, Pokoj)

* Ta część sprawdza, czy istnieje połączenie drzwi między punktem startowym a pokojem
* To znaczy, że **drzwi(a, b)** oraz **drzwi(b, a)** pozwolą nam swobodnie przechodzić między tymi dwoma pokojami

<br>

> \\+ member(Pokoj, OdwiedzonePokoje)

* Ta negacja oznacza, że Pokoj nie jest członkiem listy OdwiedzonePokoje
* Dzięki temu, jeśli odwiedzimy już dany pokój, to nie będziemy do niego ponownie wracać

<br>

> fragmenty z 'format(....)'

* Służą do formatowania i wyświetlania napisów potrzebnych do zadania
* Symbol **'~w'** oznacza, że w danym miejscu zostanie podstawiony odpowiadający w kolejności argument
* Symbol **'~n'** oznacza nową linię
* W nawiasach kwadratowych przekazujemy listę argumentów

<br>

> (klucz(Pokoj, KLUCZ), format('[znalazlem_klucz, ~w]~n', [KLUCZ]); droga(Pokoj, PunktDocelowy, [Pokoj|OdwiedzonePokoje], KLUCZ))

* Dzieli się to na dwa przypadki
* Jeśli w Pokoj znajduje się klucz **(klucz(Pokoj, KLUCZ))**, to wtedy wyświetlamy informację o znalezieniu klucza
* W przeciwnym razie kontynuujemy rekurencyjnie, przechodząc do kolejnego pokoju **(droga(Pokoj, PunktDocelowy, [Pokoj|OdwiedzonePokoje], KLUCZ))**
* W powyższej regule dodajemy również Pokoj do listy OdwiedzonePokoje, aby uniknąć odwiedzania tego samego pokoju
* Operator **'|'** jest nazywany operatorem listowym. Używa się go do tworzenia listy łączącej pierwszy element z resztą listy
* Tutaj, **Pokoj** jest głową nowej listy, a **OdwiedzonePokoje** jest ogonem. Za każdym razem dodawany jest nowy element na początek, a poprzednie elementy są przepisywane


<br>
<br>

## ⭐ Zadanie dla chętnych - reprezentacja wiedzy


### 🌟 Zadanie 1
<br>

**Podpunkt a)**

Przedstaw w języku logiki predykatów poniższe stwierdzenia – zacznij od zidentyfikowania w nich tzw. stałych indywiduowych oraz określenia występujących predykatów:
1. Markus był człowiekiem
2. Markus był pompejańczykiem (obywatelem Pompejów)
3. Wszyscy pompejańczycy byli Rzymianami
4. Cezar był władcą
5. Wszyscy Rzymianie albo byli lojalni wobec Cezara, albo go nienawidzili
6. Każdy jest lojalny wobec kogoś
7. Ludzie próbują dokonać zamachu tylko na tych władców, wobec których nie są lojalni
8. Markus próbował dokonać zamachu na Cezara

<br>

**Rozwiązanie:**

```
% 1) Markus byl czlowiekiem
czlowiek(markus).

% 2) Markus byl pompejanczykiem
pompejanczyk(markus).

% 3) Wszyscy pompejanczycy byli Rzymianami
rzymianin(X) :-
    pompejanczyk(X).

% 4) Cezar byl wladca
wladca(cezar).

% 5) Wszyscy Rzymianie albo byli lojalni wobec Cezara, albo go nienawidzili
lojalny_wobec_wladcy(X, Y) :-
    rzymianin(X),
    wladca(Y).

nie_lojalny_wobec_wladcy(X, Y) :-
    rzymianin(X),
    wladca(Y),
    \+ lojalny_wobec_wladcy(X, Y).

% 6) Kazdy jest lojalny wobec kogos
lojalny(_, _).

% 7) Ludzie probuja dokonac zamachu tylko na tych wladcow, wobec ktorych nie sa lojalni
probuje_dokonac_zamachu(X, Y) :-
    czlowiek(X),
    wladca(Y),
    nie_lojalny_wobec_wladcy(X, Y).

% 8) Markus probowal dokonac zamachu na Cezara
probuje_dokonac_zamachu(markus, cezar).
```

<br>
<br>

**Podpunkt b)**

Posługując się powyższymi predykatami spróbuj odpowiedzieć na pytanie, czy Markus był lojalny wobec Cezara. Czy są one wystarczające/kompletne, by można było przeprowadzić formalny dowód takiego stwierdzenia? Jeżeli nie, uzupełnij powyższą listę i podaj dowód

<br>

**Rozwiązanie:**

Predykaty określone powyżej nie są kompletne aby odpowiedzieć na to pytanie. Logicznie, jeżeli Markus próbował dokonać zamachu na Cezara, nie jest on lojalny. Reguła 'lojalny_wobec_wladcy' nie sprawdza jednak faktu dokonania zamachu. Dlatego w języku Prolog, wszystko, co nie jest zdefiniowane, będzie określone jako true. Poniżej przedstawiam zmienione i poprawione reguły. Zachowałem poprawną kolejność, która pozwoli na wywnioskowanie, że **'lojalny_wobec_wladcy(markus, cezar)'** zwróci false. Modyfikacja warunków jest konieczna, aby uniknąć nieskończonej pętli

```
czlowiek(markus).
pompejanczyk(markus).
rzymianin(X) :- pompejanczyk(X).
wladca(cezar).

probuje_dokonac_zamachu(X, Y) :-
    czlowiek(X),
    wladca(Y).

probuje_dokonac_zamachu(markus, cezar).

lojalny_wobec_wladcy(X, Y) :-
    rzymianin(X),
    wladca(Y),
    \+ probuje_dokonac_zamachu(X, Y).

nie_lojalny_wobec_wladcy(X, Y) :-
    rzymianin(X),
    wladca(Y),
    \+ lojalny_wobec_wladcy(X, Y).
```


<br>
<br>

**Podpunkt c) oraz d)**

Przekształć skonstruowane przez siebie formuły do postaci koniunkcyjnej normalnej (CNF). Przeprowadź dowód z punktu b) metodą rezolucji.

<br>

Powyższe ćwiczenia znajdowały się pod tematem 'Prolog', dlatego rozumiem, że należy wykorzystać ten język przy rozwiązywaniu zadań. Dlatego postać CNF zapisana "matematycznie" będzie analogiczna do zapisanych przeze mnie reguł. Ponadto Prolog automatycznie stosuje metodę rezolucji w swoim mechanizmie wnioskowania, dlatego dowód został niejawnie przeprowadzony wykorzystując to narzędzie. Kolejne, podobne zadanie spróbuję wykonać matematycznie, aby mieć przećwiczone oba sposoby.

<br>
<br>

**Podpunkt e)**

Wskaż dwa różne sposoby interpretacji zdania: "Ludzie próbują dokonać zamachu tylko na tych władców, wobec których nie są lojalni". W oryginale zdanie to brzmi: "People only try to assassinate rulers they are not loyal to". Która z nich została użyta w tłumaczeniu?

<br>

**Rozwiązanie:**

Zdanie "People only try to assassinate rulers they are not loyal to" można zinterpretować na dwa sposoby:
1. Jeśli ktoś nie jest lojalny wobec swojego władcy, będzie próbował go zabić
2. Brak lojalności może prowadzić do prób zamachu, ale nie jest jedynym czynnikiem (nie oznacza to, że wszyscy, którzy nie są lojalni, próbują dokonać zamachu)


Tłumaczenie jest bliższe pierwszej interpretacji. Słowo "tylko" sugeruje, że jedynymi osobami, które próbują dokonać zamachu, są osoby nie lojalne wobec władców

<br>
<br>

### 🌟 Zadanie 2
<br>

**Podpunkt a)**

Wyraź następujące zdania w języku logiki predykatów:

<br>

Jan lubi każdy rodzaj pożywienia
```
∀x (Pożywienie(x) → Lubi(Jan, x))
```

<br>

Jabłka są pożywieniem
```
Pożywienie(Jabłka)
```

<br>

Kurczak jest pożywieniem
```
Pożywienie(Kurczak)
```

<br>

Cokolwiek, co ktokolwiek je i go nie zabija, jest pożywieniem
```
∀x ∀y (Je(y, x) ∧ NieZabija(y, x) → Pożywienie(x))
```

<br>

Adam je orzeszki i wciąż żyje
```
Je(Adam, Orzeszki) ∧ Żyje(Adam)
```

<br>

Basia je wszystko to, co Adam
```
∀x (Je(Adam, x) → Je(Basia, x))
```

<br>
<br>

**Podpunkt b)**

Przekształć skonstruowane przez siebie formuły do postaci koniunkcyjnej normalnej (CNF)

<br>

Jan lubi każdy rodzaj pożywienia
```
¬Pożywienie(x) ∨ Lubi(Jan, x)
```

<br>

Jabłka są pożywieniem
```
Pożywienie(Jabłka)
```

<br>

Kurczak jest pożywieniem
```
Pożywienie(Kurczak)
```

<br>

Cokolwiek, co ktokolwiek je i go nie zabija, jest pożywieniem
```
¬Je(y, x) ∨ ¬NieZabija(y, x) ∨ Pożywienie(x)
```

<br>

Adam je orzeszki i wciąż żyje
```
Je(Adam, Orzeszki) ∧ Żyje(Adam)
```

<br>

Basia je wszystko to, co Adam
```
¬Je(Adam, x) ∨ Je(Basia, x)
```

<br>
<br>

**Podpunkt c)**

Udowodnij metodą rezolucji (ewentualnie uzupełniając stworzone poprzednio formuły), że Jan lubi orzeszki

<br>

**Rozwiązanie:**

* Możemy dodać do powyższych formuł negację tezy, którą chcemy udowodnić (czyli ¬Lubi(Jan, Orzeszki))
* Następnie szukamy par formuł, które mogą być zredukowane przez rezolucję
* W tym przypadku możemy użyć formuł 1 oraz 5 (zakładając, że orzeszki są pożywieniem)
* Daje nam to Lubi(Jan, Orzeszki), co jest sprzeczne z dodaną negacją
* Zatem teza jest prawdziwa

<br>
<br>

**Podpunkt d)**

Spróbuj (metodą rezolucji) odpowiedzieć na pytanie, jakie pożywienie je Basia

<br>

**Rozwiązanie:**

* Wykorzystujemy formułę 6, więc wiemy, że "Basia je wszystko to, co Adam"
* Na podstawie formuł 4 oraz 5 udowadniamy, że orzeszki są pożywieniem
* Zatem daje nam to, że Basia je orzeszki
* Nie mamy informacji o tym, że Jan je pożywienie, które lubi, dlatego lista jedzenia ogranicza się tylko do orzeszków

<br>
<br>

### 🌟 Zadanie 3
Markus urodził się 40 lat po Chrystusie, a Pompeje zostały zniszczone w wyniku wybuchu Wezuwiusza w roku 79 naszej ery – erupcja zabiła wszystkich mieszkańców. Do tej pory żaden z ludzi nie żył dłużej niż 150 lat. Czy Markus żyje, jeżeli mamy rok 2021? Na ile sposobów można to wykazać? Użyj logiki predykatów do rozwiązania tego zadania.

<br>

**Rozwiązanie:**

* Niech P(x) oznacza "osoba x żyje w roku 2021"
* Niech Q(x) oznacza "osoba x urodziła się w roku 40 n.e."
* Niech R(x) oznacza "osoba x żyje dłużej niż 150 lat"
* Wtedy twierdzenie brzmi: dla każdego x, jeśli Q(x) jest prawdziwe, to P(x) jest fałszywe, chyba że R(x) jest prawdziwe
* Wiemy, że R(x) jest fałszywe dla wszystkich x, więc P(Markus) musi być fałszywe
* Markus nie żyje

<br>

**Alternatywne, historyczne rozwiązanie:**

* Markus urodził się w roku 40 n.e.
* Więc miałby 1981 lat w roku 2021
* Ponieważ żaden człowiek nie żył dłużej niż 122 lata i 164 dni, Markus nie mógłby żyć do roku 2021
