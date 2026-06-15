# DnD Merchants Generator - MVP ideas

### Gólwne problemy
Stworzenie webowego narzędzia dla Mistrzów Gry (DM) w D&D 5e, które ułatwia zarządzanie sklepami w świecie gry. Aplikacja pozwala zapisywać sklepy, generować ich asortyment w oparciu o poziom drużyny oraz dynamicznie przeliczać ceny przedmiotów na podstawie rzutów graczy na Perswazję.

### Najmniejszy zestaw funkcjonalności
    - Tworzenie, przeglądanie i usuwanie profili sklepów, gdzie przy tworzeniu definiujesz tylko nazwę, typ asortymentu (np. kowal) oraz docelowy poziom drużyny.
    - Proste logowanie (np. przez Google), aby każdy DM miał bezpieczny dostęp wyłącznie do swoich zapisanych sklepów i kampanii.
    - System, który po utworzeniu sklepu losuje do niego kilka przedmiotów z wbudowanej bazy, filtrując je tak, by typ i moc przedmiotów pasowały do profilu sklepu i poziomu bohaterów.
    - Interfejs w widoku sklepu, w którym wpisanie wyniku rzutu gracza na Perswazję błyskawicznie przelicza i wyświetla zmodyfikowane ceny dla całej listy dostępnych u kupca towarów.
    - Trzymanie wygenerowanego asortymentu w bazie danych, aby gracze mogli wrócić do tego samego kupca kilka sesji później i zastać dokładnie ten sam towar.

### Co NIE wchodzi w zakres MVP
    - Brak publicznego katalogu czy opcji udostępniania linków do stworzonych kupców innym Mistrzom Gry.
    - Aplikacja nie śledzi ilości towaru, więc "kupienie" przez gracza miecza nie odejmuje go z asortymentu kupca ani nie wymaga wirtualnego koszyka.
    - Narzędzie jest przeznaczone wyłącznie dla oczu Mistrza Gry, więc nie budujesz osobnego "widoku gracza", z którego mogliby oni samodzielnie przeglądać asortyment na swoich telefonach.
    - Rezygnacja z mozolnego przeliczania sztuk miedzi, srebra, elektronu i platyny na rzecz operowania wyłącznie na jednej bazowej walucie (sztukach złota).
    - Brak rozbudowanego panelu do tworzenia autorskich, magicznych artefaktów od zera, co pozwala oprzeć się w całości na gotowej, wbudowanej bazie przedmiotów.

### Kryteria sukcesu
    - Przynajmniej 90% wygenerowanych automatycznie przedmiotów idealnie pasuje do wybranego profilu kupca i poziomu drużyny, dzięki czemu Mistrz Gry akceptuje asortyment bez ręcznego usuwania niepasujących rzeczy.
    - Stworzenie nowego sklepu z pełnym asortymentem w trakcie trwającej sesji wymaga maksymalnie 3 kliknięć i zajmuje poniżej 15 sekund, aby nie przerywać tempa gry.
    - Wpisanie wyniku rzutu na Perswazję powoduje natychmiastową zmianę wszystkich cen na ekranie (poniżej 1 sekundy), całkowicie eliminując potrzebę sięgania przez prowadzącego po fizyczny kalkulator.
    - W 100% przypadków ponowne otwarcie zapisanego sklepu na kolejnej sesji ładuje dokładnie ten sam zestaw przedmiotów i cen bazowych, zdejmując z Mistrza Gry obowiązek robienia jakichkolwiek notatek z zakupów.