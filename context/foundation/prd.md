---
project: "DnD Merchants Generator"
version: 1
status: draft
created: 2026-06-11
context_type: greenfield
product_type: web-app
target_scale:
  users: small
  qps: low
  data_volume: small
timeline_budget:
  mvp_weeks: 3
  hard_deadline: 2026-07-04
  after_hours_only: false
---

# DnD Merchants Generator — PRD

## Vision & Problem Statement

Mistrz Gry (DM) prowadzący kampanię w D&D 5e musi ręcznie zarządzać sklepami w świecie gry: dobierać asortyment pasujący do typu kupca i zamożności miejscowości, przeliczać ceny po rzutach graczy na Perswazję i pamiętać, co znajdowało się u danego kupca między sesjami. Dziś robi to ręcznie — dobiera przedmioty z głowy, sięga po fizyczny kalkulator przy negocjacjach cen i prowadzi notatki „na boku", co przerywa tempo sesji.

Insight: istniejące generatory lootu/arkusze rozwiązują jednorazowe losowanie przedmiotów, ale nie dają dwóch rzeczy, które są sednem tego narzędzia — (1) trwałości sklepu między sesjami (ten sam kupiec ma dokładnie ten sam towar kilka sesji później) oraz (2) mechaniki Perswazji, która natychmiast przelicza ceny całej listy towarów na podstawie wyniku rzutu gracza.

## User & Persona

Główna persona: **Mistrz Gry (DM) w D&D 5e**, prowadzący własne kampanie — w tym sam autor narzędzia. Sięga po aplikację w trakci
e trwającej sesji, gdy drużyna wchodzi do sklepu lub spotyka kupca. Narzędzie jest przeznaczone wyłącznie dla oczu DM-a (brak osobnego widoku gracza). Model jednoosobowy: każdy DM widzi wyłącznie własne, zapisane sklepy i kampanie.

## Success Criteria

### Primary
- Utworzenie nowego sklepu z pełnym, wygenerowanym asortymentem zajmuje ≤ 3 kliknięcia i < 15 s, nie przerywając tempa trwającej sesji.
- ≥ 90% automatycznie wygenerowanych przedmiotów pasuje do wybranego profilu kupca i poziomu bogactwa miejscowości — DM akceptuje asortyment bez ręcznego usuwania niepasujących pozycji.

### Secondary
- (Brak osobnego kryterium Secondary — jedyny kandydat, regeneracja asortymentu, awansował podczas rundy sokratejskiej do must-have FR-009. Patrz Open Questions, jeśli pojawi się nowy kandydat.)

### Guardrails
- Wpisanie wyniku rzutu na Perswazję przelicza i wyświetla zmodyfikowane ceny całej listy towarów w < 1 s — bez sięgania po zewnętrzny kalkulator.
- Ponowne otwarcie zapisanego sklepu na kolejnej sesji w 100% przypadków ładuje dokładnie ten sam zestaw przedmiotów i ich ceny bazowe (zero notatek po stronie DM-a).

## User Stories

### US-01: DM tworzy sklep z automatycznie wygenerowanym asortymentem

- **Given** zalogowany DM w trakcie sesji
- **When** podaje nazwę sklepu, typ asortymentu i poziom bogactwa miejscowości, po czym zatwierdza
- **Then** widzi widok sklepu z kilkoma przedmiotami z wbudowanej bazy, dobranymi tak, by typ i moc/wartość pasowały do profilu i poziomu bogactwa miejscowości

#### Acceptance Criteria
- Utworzenie sklepu z asortymentem zajmuje ≤ 3 kliknięcia i < 15 s.
- ≥ 90% wygenerowanych przedmiotów pasuje do profilu kupca i poziomu bogactwa miejscowości.
- Każdy przedmiot ma cenę bazową w sztukach złota.

### US-02: DM przelicza ceny po rzucie gracza na Perswazję

- **Given** otwarty sklep z wygenerowanym asortymentem i cenami bazowymi
- **When** DM wpisuje wynik rzutu gracza na Perswazję
- **Then** wszystkie ceny na liście przeliczają się i wyświetlają zmodyfikowane wartości w < 1 s

#### Acceptance Criteria
- Przeliczenie obejmuje całą listę towarów, nie pojedynczy przedmiot.
- Ceny bazowe pozostają zachowane — przeliczenie nie nadpisuje ich trwale.

### US-03: DM wraca do sklepu na kolejnej sesji

- **Given** sklep zapisany podczas wcześniejszej sesji
- **When** DM otwiera go ponownie
- **Then** widzi dokładnie ten sam zestaw przedmiotów i te same ceny bazowe co poprzednio

#### Acceptance Criteria
- W 100% przypadków asortyment i ceny bazowe są identyczne jak przy zapisie.
- DM nie musi prowadzić żadnych notatek poza aplikacją.

## Functional Requirements

### Authentication
- FR-001: DM może zarejestrować się i zalogować przy użyciu e-maila i hasła. Priority: must-have
  > Socrates: Rozważony kontrargument: "auth to narzut opóźniający pierwszą wartość / własne hasła to obowiązki". Rozstrzygnięcie: zostaje — izolacja danych między DM-ami jest wymagana, a logowanie e-mail+hasło to wybór produktowy potwierdzony w Fazie 2.

### Shop management
- FR-002: DM może utworzyć profil sklepu, podając nazwę, typ asortymentu (np. kowal) i poziom bogactwa miejscowości (np. wioska / miasteczko / metropolia). Priority: must-have
  > Socrates: Rozważony kontrargument: "poziom drużyny to zły parametr doboru". Rozstrzygnięcie: ZMIENIONO — parametrem sterującym jest poziom bogactwa miejscowości, nie poziom drużyny. Bogactwo osady wprost mapuje na moc/wartość dostępnych towarów.
- FR-003: DM może przeglądać listę swoich zapisanych sklepów. Priority: must-have
  > Socrates: Rozważony kontrargument: "płaska lista bez kampanii się zagraca". Rozstrzygnięcie: zostaje — prosta lista wystarcza dla MVP; grupowanie po kampanii to późniejsza optymalizacja (patrz Open Questions).
- FR-004: DM może usunąć profil sklepu, z potwierdzeniem przed nieodwracalnym usunięciem. Priority: must-have
  > Socrates: Rozważony kontrargument: "twarde usuwanie = utrata danych, do których gracze mieli wrócić". Rozstrzygnięcie: zaakceptowano ryzyko i dodano wymóg potwierdzenia przed usunięciem.
- FR-010: DM może edytować profil istniejącego sklepu (nazwa, typ asortymentu, poziom bogactwa miejscowości). Priority: must-have
  > Socrates: Rozważony kontrargument: "edycja typu/poziomu wymaga regeneracji asortymentu". Rozstrzygnięcie: zachowano FR; zależność edycja→regeneracja zanotowana do rozstrzygnięcia w projektowaniu (patrz Open Questions).

### Assortment generation
- FR-005: System generuje asortyment sklepu, losując przedmioty z wbudowanej bazy i filtrując je tak, by typ i moc/wartość pasowały do profilu sklepu i poziomu bogactwa miejscowości. Priority: must-have
  > Socrates: Rozważony kontrargument: "losowanie da niespójny asortyment, który DM i tak odrzuci". Rozstrzygnięcie: zostaje — cel ≥ 90% trafności (Success Criteria) jest miarą jakości filtra; losowanie odbywa się w obrębie dopasowanej puli.
- FR-006: DM może otworzyć widok sklepu z wygenerowanym asortymentem i cenami bazowymi. Priority: must-have
  > Socrates: Rozważony kontrargument: brak. Rozstrzygnięcie: zostaje — to rdzeń przepływu MVP.
- FR-009: DM może jednym kliknięciem przelosować asortyment istniejącego sklepu. Priority: must-have
  > Socrates: Rozważony kontrargument: "regeneracja niszczy zapisany stan / albo powinna być must-have". Rozstrzygnięcie: AWANSOWANO do must-have — możliwość przelosowania jest na tyle ważna, że wchodzi do rdzenia MVP. Pozostaje świadomą akcją DM-a (nie automatem), więc nie zagraża trwałości (FR-008).

### Pricing
- FR-007: DM może wpisać wynik rzutu gracza na Perswazję i natychmiast zobaczyć przeliczone ceny całej listy towarów. Priority: must-have
  > Socrates: Rozważony kontrargument: "wzór przeliczania (rzut na Perswazję → rabat) jest niejednoznaczny i bez definicji FR jest pusty". Rozstrzygnięcie: FR zostaje, ale konkretny wzór mapowania rzutu na modyfikator ceny jest OTWARTY — do zdefiniowania w sekcji Business Logic / Open Questions.

### Persistence
- FR-008: System trwale zapisuje wygenerowany asortyment, tak że ponowne otwarcie sklepu pokazuje dokładnie te same przedmioty i ceny bazowe. Priority: must-have
  > Socrates: Rozważony kontrargument: brak. Rozstrzygnięcie: zostaje — trwałość między sesjami to główny insight produktu.

## Non-Functional Requirements

- Żaden Mistrz Gry nie ma dostępu do sklepów ani danych innego Mistrza Gry — dane są w pełni izolowane między kontami.
- Utworzenie sklepu z pełnym, wygenerowanym asortymentem daje widoczny wynik w < 15 s od zatwierdzenia.
- Przeliczenie cen całej listy po wpisaniu wyniku rzutu na Perswazję jest widoczne dla użytkownika w < 1 s.

## Business Logic

Aplikacja dobiera kupcowi dopasowany, losowy asortyment i dynamicznie modyfikuje jego ceny na podstawie wyniku rzutu gracza na Perswazję. Reguła rozbija się na dwie powiązane części:

**Reguła 1 — generowanie asortymentu.** Aplikacja wybiera losowy zestaw przedmiotów ograniczony do tych, których typ oraz moc/wartość pasują do typu asortymentu sklepu i poziomu bogactwa miejscowości. Inputy użytkownika: typ asortymentu (np. kowal) i poziom bogactwa miejscowości (np. wioska / miasteczko / metropolia). Output: kilka przedmiotów z wbudowanej bazy, każdy z ceną bazową w sztukach złota. Użytkownik napotyka regułę w momencie tworzenia sklepu — wynik pojawia się od razu w widoku kupca.

**Reguła 2 — przeliczanie cen wg Perswazji.** Aplikacja modyfikuje wyświetlane ceny całej listy towarów na podstawie wyniku rzutu gracza na Perswazję, mapując wartość rzutu na rabat według progów. Input użytkownika: pojedynczy wynik rzutu na Perswazję. Output: zmodyfikowane ceny całej listy, przy zachowaniu cen bazowych. Użytkownik napotyka regułę w widoku sklepu, wpisując wynik rzutu podczas negocjacji.

> Domyślna propozycja progów (do dostrojenia — patrz Open Questions): wynik < 10 → brak rabatu; 10–14 → −5%; 15–19 → −10%; 20+ → −20%. Konkretne przedziały i wartości procentowe są decyzją balansującą, nie ustaleniem domenowym.

## Access Control

Model wieloużytkownikowy z uwierzytelnianiem **e-mail + hasło**. Płaski model ról — każdy zalogowany użytkownik jest Mistrzem Gry i ma dostęp wyłącznie do własnych sklepów i kampanii; dane są izolowane między kontami. Brak osobnej roli administracyjnej w MVP. Niezalogowany użytkownik nie ma dostępu do żadnych danych sklepów — wejście na trasę chronioną kieruje do logowania/rejestracji.

## Non-Goals

- **Bez udostępniania i publicznego katalogu** — żadnych linków ani wspólnego katalogu kupców dla innych DM-ów; narzędzie pozostaje prywatne dla właściciela konta. Zapobiega rozrostowi w stronę platformy społecznościowej.
- **Bez śledzenia stanu magazynowego** — „kupienie" przedmiotu przez gracza nie odejmuje go z asortymentu; brak wirtualnego koszyka i liczników sztuk. Upraszcza zakres i przepływ MVP.
- **Bez widoku gracza** — narzędzie jest wyłącznie dla oczu DM-a; gracze nie przeglądają asortymentu na własnych urządzeniach. Eliminuje drugą personę i osobny interfejs.
- **Tylko jedna waluta (sztuki złota)** — brak przeliczania miedzi/srebra/elektronu/platyny; operujemy wyłącznie na złocie. Usuwa mozolną arytmetykę walutową.
- **Bez kreatora własnych artefaktów** — brak panelu tworzenia autorskich magicznych przedmiotów od zera; opieramy się w całości na wbudowanej (importowanej z SRD) bazie. Trzyma zakres MVP przy gotowych danych.

## Open Questions

1. **Jaki dokładnie wzór mapuje rzut na Perswazję na modyfikator ceny?** — propozycja progowa zapisana w Business Logic jest domyślna; konkretne przedziały i procenty wymagają potwierdzenia. Owner: użytkownik. Block: nie (MVP może wystartować z domyślnymi progami).
2. **Czy edycja typu/poziomu bogactwa sklepu (FR-010) wymusza regenerację asortymentu?** — zależność do rozstrzygnięcia w projektowaniu. Owner: użytkownik/implementacja.
3. **Czy lista sklepów (FR-003) będzie wymagać grupowania po kampanii w miarę wzrostu liczby sklepów?** — poza MVP; do obserwacji. Owner: użytkownik.
