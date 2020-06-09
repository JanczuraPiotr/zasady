# Zasady odbioru komend i udzielania odpowiedzi.

## Definicje:

* Domena aplikacji - główne zagadnienie do obsługi którego utworzono aplikację    
* Encja - zestaw atrybutów wspólnie opisujących złożony obiekt. Atrybuty są typów prostych.
* Rekord - Encja zapisana na dysku rozszerzona o unikalny identyfikator czasami o daty utworzenia i ostatniej modyfikacji rekordu.  

## Architektura

### Główne elementy

Architektura aplikacji rzutowana jest na strukturę katalogów przechowujących jej źródła i przestrzenie nazw.

Kod dzielony jest na  **akcje**, **kontrolery**, **stany**, **algorytmy**, **dane**, **repozytoria**.

**Kontrolery** obsługują sygnały za pomocą **akcji**.

**Akcje** wykonują główne zadania aplikacji. **Akcja** pozostawia po sobie ślad w postaci zmiany **stanu** aplikacji i/lub zapisu **danych** w **repozytorium**.   

**Wejścia** analizują dane wejściowe zwracając obiekt z przestrzeni data:: . 

**Wyjścia** wykonują konwersję obiektu przestrzeni data:: do formatu wymaganego przez przez kanał wyjściowy przez który zostaną wysłane dane.

**Wejścia**, **wyjścia**, **kontrolery**, **stany** wymieniają się **danymi**. Wszystkie **dane** nawet o najprostszej strukturze zdefiniowane są jako osobna klasa (encja). 

**Stan**, obiekt przechowujący informacje, dostępny dla wszystkich innych obiektów aplikacji. 

Wejścia i wyjścia wyznaczają brzegi aplikacji. Praca na brzegu aplikacji oznacza przetwarzanie danych otrzymanych z wejście. 

### Akcja

``namespace act``

Obiekt uruchamiany na potrzeby konkretnego zadnia. Raczej zawsze będzie tworzony przez metodę kontrolera. Akcje opisują funkcjonalność aplikacji z perspektywy klienta. Wszystko co można wykonać na aplikacji jako jej klient musi być opisane klasą umieszczoną na liście akcji. Lista akcji dla czytelnika jest podstawowym dokumentem o możliwościach aplikacji (jest definicją aplikacji).

Dostęp do stanów, danych i algorytmów.

### Kontroler

``namespace ctrl``

Obsługuje sygnały od interfejsów graficznych z klawiatury i od obiektów przechowujących stan, oraz wejść. Sygnały - zdarzenia wewnątrz interfejsów graficznych nie są wyszczególniane. Obsługa polega na wywołaniu właściwej akcji i w zależności od jej wyniku przekazaniu danych do wyjścia lub wywołaniu kolejnej akcji. Akcje tworzone są na podstawie danych przekazanych w parametrach metody oraz stanów.

Dostęp do akcji, stanów, wejść, wyjść i danych. 

### Stan
``namespace stt`` 

Obiekt trwający przez cały czas życia aplikacji. Na podstawie własnych mechanizmów i wywołujących je akcji modyfikują przechowywane przez siebie zmienne. Prawdopodobnie będzie posiadał uruchomiony wątek. Prawdopodobnie zmiany w obrębie atrybutów będą opisane algorytmem i będzie modyfikował repozytoria. Obiekty klas tej przestrzeni generują sygnały o zmianie stanu.
Definicją przez przykład można napisać że obiekty utworzony jest w funkcji main() i tam uruchomiony a referencja do niego przekazywana jest wszystkim zainteresowanym obiektom.

Dostęp do danych i algorytmów. 

### Algorytm
``namespace alg``

Prosta funkcja (może klasa) przetwarzająca, modyfikująca, licząca. 

Dostęp do danych bez możliwości zapisu.

### Dane
``namespace data``

Złożone typy danych wykorzystywane przez porty, zdarzenia, akcje i algorytmy.
Przechowuje w bazie danych pozwala wyszukiwać i modyfikować.

``namespace data::entity`` Encja. Może istnieć w niekompletnej formie i stanowić podstawę do filtrowania.

``namespace data::record`` Encja przechowywana na dysku.

``namespace data::map`` Zestawienia. Typ zwracany z repo jako wynik wyszukiwania.

``namespace data::repo`` Przechowywanie rekordy zapisane na podstawie encji po uprzednim walidowaniu. 

Poprawność encji kontrolowana jest w repo podczas tworzenia rekordu. Samodzielnie może być stosowana jako parametr filtrów albo lista parametrów. Encja nie musi być zapisana w bazie. Szczegóły na ten temat w zasadach posługiwania się zbiorami danych. 

Wejścia mogą otrzymywać dane w formie, która dopiero po przeanalizowaniu pozwoli określić typ danej. Prawdopodobnie będzie nim bufor lub strumień danych. Wyjścia będą otrzymywać jako parametr typ zdefiniowany w ``data``. 

Zakładam ogólny typ dla danych wejściowych i wyjściowych w warstwie transportowej (sieciowej):

  ```c++
  namespace net {
      typedef unsigned char Variable; 
      typedef std::vector<Variable> Buffers; 
  }
  ```

### Wejście

``namespace in``

Wejście w sensie miejsca, w którym dane dostają się do aplikacji. Analiza danych wejściowych. Sprawdzenie ich poprawności pod względem bezpieczeństwa aplikacji oraz zgodnością z wykonywanym zadaniem. 

### Wyjście
``namespace out``

Przygotowuje dane do wysłania na zewnątrz aplikacji.

### Zależności między elementami.

Elementy współpracujące w celu wykonania nazwanego zadanie posługują się tą samą nazwą. Odróżnia je namespace z którego pochodzą. Założenie, w przykładzie obsługujemy akcję obsługującą żądanie podania faktury.

Główną składową cyklu obsługi jest ``act::GetFaktura``. Ona jest wyposażona we wszystkie dane i mechanizmu do wykonania zadania. Zwrócenia wyniku lub informacji o błędzie. ``act::GetFaktura`` inicjowana jest obiektem ``data::entity::Faktura``. Wynikiem jest pracy jest encja lub mapa encji czyli ``data::entity::Faktura`` lub ``data::map::Faktura``. Będzie to zależało od założeń projektowych.

Jeżeli ``act::GetFaktura`` nie będzie wywoływana na brzegu aplikacji to dane ją inicjujące mogą pochodzić od innej akcji a on sama może być producentem danych dla innej akcji. Wywołanie akcji nie na brzegach aplikacji powinno być rzadkością. Akcje tego typu można tworzyć w celu widocznego wyodrębnienia istotnej operacji. Dobrym rozwiązaniem może być umieszczanie kodu wykonywanego przez okna dialogowe w akcjach.

Wszystkie elementy wymieniają się danymi zdefiniowanymi w przestrzeni ``data``.

Akcja jest wykonywana na podstawie ``data::entity::Faktura``. Gdy jest na skraju aplikacji obiekt tej zmiennej musi być utworzony z danych wejściowych. Najprawdopodobniej będzie to ``in::GetFaktura input(net::Buffer data);`` zwracająca ``data::entity::Faktura faktura = input.data();`` Zakładając, że ``act::GetFaktura`` znalazła większość ilość faktur: ``data::map::Faktura faktura = act::GetFaktura.faktury()`` . 

Jeżeli wynik pracy akcji przewidziany jest do przekazania kolejnej akcji w celu wydrukowania, umieszczamy go jako parametr konstruktora np : ``act::PrintFaktura printFaktura(faktura)``. Jeżeli wynik pracy ma być przekazany poza aplikację należy przetworzyć ją do postaci obsługiwanej przez kanał transmisyjny. Zakładam że będzie to ``net::Buffer`` : ``out::GetFaktura getFaktura(faktura)``  ``getFaktura.parse()`` a następnie: ``jakaśMetodaWykonującaWysyłkę(getFaktura.buffer())``


## Więcej szczegółów  

Czyli podejście utworzenia nowej funkcjonalności, którą jest zwrócenie faktury. Uwaga na pseudokod.

### Komunikacja

Zakładam, że ``stt::Siec``  potrafi skompletować bufor wejściowy i wyciągnąć z niego kod komendy.

```C++
// Przedstawione są główne najistotniejsze składowe klasy.
// Klasa po odebraniu kompletnego bufora sprawdzi jego poprawność i komendę z jaką
// jest związany a następnie wyemituje sygnał o właściwej nazwie.
class stt::Siec {
public:
    // ...
    // Sygnał jaki wyemituje ta klasa po rozpoznaniu komendy o kodzie GET_FAKTURA
    void getFakturaRequest(model::entity::Faktura faktura); // <- sygnał
    // Metoda do której przekazujemy fakturę którą chcemy wysłać klientowi.
    void getFakturaResponse(model::entity::Faktura faktura) {// <- slot
        out::GetFakturaResponse output(faktura);
        metodaWysyłającaDane(output.getBuffer());
    }
    // ...
private:
    // Założenia
    // Klasa ma zdefiniowane metody odczytu i zapisu do sieci.
    // Metoda która odczytała bufor wywołuje metodę definiowaną w tym miejscu.
    void processCommand(net::Buffer &buffer) {
        Command command = getCommand(buffer);
        switch(command) {
            //.. 
            case GET_FAKTURA: {
                in::GetFaktura input(buffer);
                if (input.parse()) {
                    // Wysyłamy sygnał o typie odebranego komunikatu.
                    // W zależności od zastosowanej biblioteki utworzyć właściwą 
                    // konstrukcję. 
                    data::entity::Faktura faktura = input.data();
                    sygnał: getFakturaRequest(faktura);
                } else {
                    // Błędne dane wejściowe
                }
            }
            // ...    
            default: {}    
        }
    }
}
```

Istotne jest, że od momentu rozpoznania komunikatu w warstwie sieciowej posługujemy się danymi zdefiniowanymi, co najmniej, jako encje. 

### Klasy obsługującej wejścia.

W klasie bazowej należy umieścić wszystkie metody pozwalające analizować bufor.

```c++
// Przedstawione są główne najistotniejsze składowe klasy.
// Klasa posiadająca wszystkie mechanizmy pozwalającej jej analizować bufor
// wejściowy sformatowany zgodnie z protokołem.
class in::Input {
public:
    Input(net::Buffer buffer) 
        : cursor(0), buffer(buffer) 
        {};
    
    virtual bool parse() = 0;
    
protected:
    // Poglądowo, metody których istnienia w takiej klasie można się spodziewać. 
    int getInt(); 
    double getDouble();
    std::string getString();
    
protected:
    std::size_t cursor; // Aktualne położenie znaku czytania kolejnej zmiennej. 
                        // Każda metoda getXxx() rozpoczyna odczyt z pozycji na którą
                        // wskazuje cursor. Po zakończeniu odczytu metoda ustawia
                        // kursor na następną pozycję po miejscu na którym zakończyła 
                        // odczyt.
    const net::Buffer buffer; // dane wejściowe,
}
```

Mając klasę bazową parsującą wejście dziedziczymy po niej specjalizowaną klasę obsługującą konkretny komunikat:

```c++
// Przedstawione są główne najistotniejsze składowe klasy.
class in::GetFaktura : public in::Input {
public:	
    GetFaktura(net::Buffer buffer)
        : Input(buffer) 
        {};
    
    bool parse() override {
        bool result = true;
        // W rzeczywistym kodzie, gdzieś po drodze jest prowadzona kontrola			
        // poprawności by ustawić result na false w razie niepowodzenia. 
        // Może to być prze wyjątek rzucony w dowolnym momencie.
        try {
            faktura.setId(getInt());
            faktura.setOdbiorca(getString());
            faktura.setValue(getDouble());
        } catch (...) {
            result = false;
        }
        return result;
    }   
    
    data::entity::Faktura getFaktura() {
        return faktura;
    }
private:
    data::entity::Faktura faktura;    
}
```

### Klasy obsługujących wyjścia

W klasie bazowej należy umieścić wszystkie metody pozwalające złożyć bufor.

```c++
// Przedstawione są główne najistotniejsze składowe klasy.
// Klasa posiada wszystkie metody pozwalające umieścić wartość wszystkich typów
class out::Output {
public:
    Output() 
        : cursor(0) , buffer() 
        {};
    
    virtual bool serialize() = 0;
    
    net::Buffer getBuffer() {
        return buffer;
    }
    
protected:
    // Poglądowo, metody których istnienia w takiej klasie można się spodziewać 
    void insertInt(int value); 
    void insertDouble(double value);
    void insertString(const std::string &value);
    //...
private:
    std::size_t cursor; // Położenie w buffer na którym metody insertXxx mają zacząć 
                        // umieszczać zmienne. Po zakończeniu swojej pracy ustawiają
                        // cursor na miejsce od którego następna metoda ma rozpocząć
                        // umieszczanie swojej zmiennej.
    net::Buffer buffer;
}
```

Tworzymy klasę specjalizowaną pakującą pola klasy data::entity::Faktura.

```c++
class out::GetFaktura: public out::Output {
public:
    GetFaktura(data::entity::Faktura faktura) 
        : Output(), faktura(faktura) {};
    
    void serialize() {
        insertInt(faktura.getId());
        insertString(faktura.getOdbiorca());
        insertDouble(faktura.getValue());
    }
    
private:
    const data::entity::Faktura faktura;
}
```

### Akcje 

Akcja zależy od skomplikowania zadania które ma wykonać więc trudno narzucać (proponować) formę. Najważniejsze : w konstruktorze umieszczamy wszystkie zależności dla wykonania akcji.  Metoda Action::action() wykonuje wszystkie operacje i zwraca bool informujący o powodzeniu. Musi istnieć metoda zwracająca odpowiedź w postaci obiektu klasy z namespace data::.

```C++
namespace act {
    
// Minimalny interfejs dla wszystkich klas akcji.
class Action {
public:
    virtual bool action() = 0;     
}
}
```
```C++
namespace act {

// Przedstawione są główne najistotniejsze składowe klasy.
class GetFaktura : public Action {
public:
    GetFaktura(data::entity::Faktura filtr, data::repo::Faktura repoFaktury)
        : filtr(filtr), faktura(nullptr), repoFaktury(repoFaktury){}

    bool action() override {
        faktura = repoFaktury.find(filtr);
        if (faktura != nullptr) {
            return true;
        } else {
            return false;
        }
    }

    data::record::Faktura getFakturaRecord() {
        return faktura; 
    }

    data::entity::Faktura getFakturaEntity() {
        return faktura.entity();
    }

private: // atrybuty
    const data::entity::Faktura filtr;
    data::record::Faktura faktura;
    data::repo::Faktura repoFaktury;
}

}
```

### Kontroler.

Wyżej zdefiniowane elementy wykonywane są na poziomie sieci. Klasa odbierająca i wysyłająca dane komunikuje się z klasami kontrolerów sygnalizując im odebrane komendy.  Sposoby takiej komunikacji będą zależały od wykorzystanego rozwiązania.  W przykładzie wyżej zasugerowane jest, że w jakiś sposób zostanie wysłany sygnał. Więc zostając w tej konwencji bez wskazywania gdzie wystąpi miejsce zestawienia takiego połączenia należy przyjąć że metody kontrolera zostały połączone do sygnału z stt::Siec.

```c++
// Przedstawione są główne najistotniejsze składowe klasy.
class ctrl::Ksiegowosc {
public:
    // ...
    // Żądanie zwrócenia informacji o fakturze o której część informacji
    // zawarta jest w encji faktura. 
    // Dla uproszczenia przyjęto, że w repozytorium nie ma możliwości znalezienia 
    // większej ilości faktur pasujących do wzorca podanego w parametrze filtr
    void getFaktura(data::entity::Faktura filtr) {
        data::repo::Faktura repoFaktura;
        act::GetFaktura getFaktura(filtr, repoFaktura);
        if(getFaktura.action()) {
            // Odpowiedzią na wyszukiwanie zawsze jest mapa.
            data::entity::Faktura faktura = getFaktura.getFakturaEntity();
            sygnał : getFakturaResponse(faktura)
        } else {
            // Nie udało się wykonać zadania.
            // Od kontekstu będzie zależało czy nie powodzeniem jest nie znalezienie 
            // faktury czy problem z wykonaniem czynności 
            sygnał : brakFaktury(filtr);
        }		
    }   
    // ...
}
```



