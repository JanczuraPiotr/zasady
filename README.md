# Zasady odbioru komend i udzielania odpowiedzi.

## Definicje:

* Domena aplikacji - główne zagadnienie do obsługi którego utworzono aplikację    
* Encja - zestaw atrybutów wspólnie opisujących złożony obiekt. Atrybuty są typów prostych.
* Rekord - Encja zapisana na dysku rozszerzona o unikalny identyfikator.  

## Architektura

### Główne elementy

Architektura aplikacji rzutowana jest na strukturę katalogów przechowujących jej źródła i przestrzenie nazw.

Kod dzielony jest na  **akcje**, **porty**, **zdarzenia**, **stany**, **algorytmy**, **dane**, **repozytoria**.

**Akcje** wykonują główne zadania aplikacji. **Akcja** pozostawia po sobie ślad w postaci zmiany **stanu** aplikacji i/lub zapisu **danych** w **repozytorium**. **Porty** obsługują operacje wejścia wyjścia za pomocą **akcji**.  Analizę danych wejściowych wykonują **wejścia**. Przygotowanie danych wyjściowych wykonują **wyjścia**. **Zdarzenia** głównie czynności wykonywane przez operatora na interfejsie ale też przez np upływ czasu.

**Wejścia**, **wyjścia**, **zdarzenia**, **akcje**, **stany** wymieniają się **danymi**. Wszystkie **dane** nawet o najprostszej strukturze zdefiniowane są jako osobna klasa. 

**Stan**, obiekt przechowujący informacje dostępny dla wszystkich innych obiektów aplikacji. 

**Porty** i **zdarzenia** Są metodami w klasach kontrolerach. Klasa kontroler obsługuje tylko sygnały o odebranych danych. 

### Akcja

``namespace act``

Obiekt uruchamiany na potrzeby konkretnego zadnia. Raczej zawsze będzie tworzony przez port lub podczas obsługi zdarzenia. Akcje opisują funkcjonalność aplikacji z perspektywy klienta, operatora. Wszystko co można wykonać na aplikacji jako jej klient musi być opisane klasą umieszczoną na liście akcji. Lista akcji dla czytelnika jest podstawowym dokumentem o możliwościach aplikacji (jest definicją aplikacji).

Dostęp do stanów, danych i algorytmów i repo.

### Zdarzenie

``namespace ev``

Obsługuje sygnały od interfejsów graficznych (menu, przyciski, pól edycyjnych, itp) oraz wejść, klawiatura. Sygnał - zdarzenia wewnątrz interfejsów graficznych nie są wyszczególniane.

Dostęp do akcji i danych.

### Port

``namespace prt``

Jego zdaniem jest wykonać akcję zgodną z odebranym sygnałem. Port ma dostęp do klas wykonujących akcje, do klas obsługujących dane wejściowe i klas generujących dane wyjściowe.  Obsługuje sieć, porty USB konsole. Ponieważ port umieszczony jest na "granicach" aplikacji będzie do niego należało przygotowanie danych wejściowych dla akcji i utworzenie strumienia wyjściowego z danych zwróconych przez akcję. 

Dostęp do wejść wyjść i akcji.

### Stan
``namespace stt`` 

Klasa trwająca przez cały czas życia aplikacji. Na podstawie własnych mechanizmów i wywołującyPrzypadki użycia.ch je akcji modyfikują przechowywane przez siebie zmienne. Prawdopodobnie będzie posiadał uruchomiony wątek. Prawdopodobnie zmiany w obrębie atrybutów będą opisane algorytmem i będzie modyfikował repozytoria.

Dostęp do danych i algorytmów.

### Algorytm
``namespace alg``

Prosta funkcja (może klasa) przetwarzająca, modyfikująca, licząca.

Dostęp do danych.

### Dane
``namespace data``

Złożone typy danych wykorzystywane przez porty, zdarzenia, akcje i algorytmy.
Przechowuje w bazie danych pozwala wyszukiwać i modyfikować.
Proste typu danych np: ```typedef int Kwota``` tworzone w pliku przeznaczonym na definicje, w ciele innych klas a w razie konieczności umieszczane w domenowej przestrzeni nazw.

``namespace data::entity`` Encja. Może istnieć w nie kompletnej formie i stanowić podstawę do filtrowania.

``namespace data::record`` Encja przechowywana na dysku, wyposażona w identyfikator

``namespace data::map`` Zestawienia. Produkt zwracany  z repo

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

Analiza danych wejściowych. Sprawdzenie ich poprawności pod względem bezpieczeństwa aplikacji oraz zgodnością z wykonywanym zadaniem.

### Wyjście
``namespace out``

Przygotowuje dane do wysłania na zewnątrz aplikacji.

### Zależności między elementami.

Elementy współpracujące w celu wykonania nazwanego zadanie posługują się tą samą nazwą. Odróżnia je namespace z którego pochodzą. Założę że w przykładzie obsługujemy akcję obsługującą żądanie podania faktury.

Główną składową cyklu obsługi jest ``act::GetFaktura``. Ona jest wyposażona we wszystkie dane i mechanizmu do wykonania zadania. Zwrócenia wyniku lub informacji o błędzie. ``act::GetFaktura`` inicjowana jest obiektem ``data::entity::Faktura``. Wynikiem jest pracy jest encja lub mapa encji czyli ``data::entity::Faktura`` lub ``data::map::Faktura``. Będzie to zależało od założeń projektowych.

Jeżeli ``act::GetFaktura`` nie będzie wywoływana na brzegu aplikacji to dane ją inicjującą mogą pochodzić od innej akcji a on sama może być producentem danych dla innej akcji. Wywołanie akcji nie na brzegach aplikacji powinno być rzadkością. Można je tworzyć w celu widocznego wyodrębnienia istotnej operacji. Dobrym rozwiązaniem może być umieszczanie kodu wykonywanego przez okna dialogowe w akcjach.

Wszystkie elementy wymieniają się danymi zdefiniowanymi w przestrzeni ``data``.

Akcja jest wykonywana na podstawie ``data::entity::Faktura``. Gdy jest na skraju aplikacji obiekt tej zmiennej musi być utworzony z danych wejściowych. Najprawdopodobniej będzie to ``in::GetFaktura input(net::Buffer data);`` zwracająca ``data::entity::Faktura faktura = input.data();`` Zakładając, że ``act::GetFaktura`` znalazła większość ilość faktur: ``data::map::Faktura faktura = act::GetFaktura.faktury()`` . 

Jeżeli wynik pracy akcji przewidziany jest do przekazania kolejnej akcji umieszczamy go jako parametr konstruktora : ``act::PrintFaktura printFaktura(faktura)``. Jeżeli wynik pracy ma być przekazany poza aplikację należy przetworzyć ją do postaci obsługiwanej przez kanał transmisyjny. Zakładam że będzie to ``net::Buffer`` : ``out::GetFaktura getFaktura(faktura)``  ``getFaktura.parse()`` a następnie: ``jakaśMetodaWykonującaWysyłkę(getFaktura.buffer())``


## Więcej szczegółów  

Czyli podejście utworzenia nowej funkcjonalności, którą jest zwrócenie faktury.

### Komunikacja

Zakładam, że ``stt::Siec``  potrafi skompletować bufor wejściowy i wyciągnąć z niego kod komendy.

```C++
// Główne składowe klasy.
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
    // Metoda tej klasy która ma skompletowany bufor wejściowy wywołuje tą metodę.
    void processCommand(net::Buffer &buffer) {
        Command command = getCommand(buffer);
        switch(command) {
            //.. 
            // Na poziomie sieci 
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

Istotne jest, że od momentu rozpoznania komunikatu w warstwie sieciowej posługujemy się danymi zdefiniowanymi jako encje, wraz z przypisanymi do nich rekordami mapami rekordów i repozytoriów. 

### Klasy obsługującej wejścia.

W klasie bazowej należy umieścić wszystkie metody pozwalające analizować bufor.

```c++
// Klasa posiadająca wszystkie mechanizmy pozwalającej jej analizować bufor
// wejściowy sformatowany zgodnie z protokołem.
class in::Input {
public:
    Input(net::Buffer buffer) 
        : cursor(0), buffer() {};
    
    virtual bool parse() = 0;
    
protected: // metody
    // Poglądowo, metody których istnienia w takiej klasie można się spodziewać 
    int getInt(); 
    double getDouble();
    std::string getString();
    
protected: // atrybuty
    std::size_t cursor; // Aktualne położenie znaku czytania kolejnej zmiennej. Każda 							
                        // metoda getXxx() ustawia rozpoczyna odczyt z pozycji na którą
                        // wskazuje cursor. Po zakończeniu odczytu metoda ustawia kursor
                        // na następną pozycję po miejscu na którym zakończyła odczyt.
    const net::Buffer buffer; // dane wejściowe,
}
```

Mając klasę bazową parsującą wejście dziedziczymy po niej specjalizowaną klasę obsługującą konkretny komunikat:

```c++
class in::GetFaktura : public in::Input {
public:	
    GetFaktura(net::Buffer buffer);
    bool parse() override {
        bool result = true;
        // W rzeczywistym kodzie, gdzieś po drodze jest prowadzona kontrola poprawności 
        // by ustawić result na false w razie niepowodzenia. 
        // Może to być prze wyjątek rzucony w dowolnym momencie.
        try {
            faktura.setId(getInt(3));
            faktura.setOdbiorca(getString(7));
            faktura.setValue(getDouble(35));
        } catch (...) {
            result = false;
        }
        return result;
    }   
    
    data::entity::Faktura getFaktura() {
        return faktura;
    }
   
private : //metody
    
private: // atrybuty
    data::entity::Faktura faktura;    
}
```

### Klasy obsługujących wyjścia

W klasie bazowej należy umieścić wszystkie metody pozwalające złożyć bufor.

```c++
// Klasa posiada wszystkie metody pozwalające umieścić wartość wszystkich typów
class out::Output {
public:
    Output() : cursor(0) , buffer() {};
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





----------------------------------------------



NOTATKI


## Kontroler , port



```c++

typedef unsigned char net::Variable;
typedef std::vector<net::Variable> net::Buffer;

// Klasa reprezentująca zapytanie o pobranie danych
class data::filtr::Faktura {
    
}

class in::GetFaktura {
public:
    GetFaktura(const net::Buffer &data);
    GetFaktura(const Dialog &dialog);
    bool parse();
    data::filtr::Faktura filtr();
    // ...
}

// Metoda wywołana gdy rozpoznano żądanie przesłania faktury. 
void Port::getFaktura(net::Buffer data) {
    in::GetFaktura input(data);
    if (input.parse()) {
        data::filtr::Faktura filtr = input.filtr();
        act::GetFaktura getFaktura(filtr);
        if(getFaktura.action()) {
            // Odpowiedzią na wyszukiwanie zawsze jest mapa.
            data::map::Faktura faktury = getFaktura.faktura();
            out::GetFaktura output(faktury);
            signalFaktura(output);
        } else {
            // Nie udało się wykonać zadania.
            // Od kontekstu będzie zależało czy nie powodzeniem jest nie znalezienie 
            // faktury czy problem z wykonaniem czynności 
        }
    } else {
        // obsługa błędnych danych wejściowych
    }
}

// Metoda wywoływana gdy zatwierdzono okno dialogowe z żądaniem pokazania faktury
void Dialog::getFaktura() {
    in::GetFaktura getFaktura(this);
    // jw.
    // ...
    data::map::Faktura faktury = getFaktura.faktura();
    FakturaListView fakturaListView();
    data::Faktura wybranaFaktura = fakturaListView.show();
    FakturaView fakturaView(wybranaFaktura);
    fakturaView.show();
}
```



### Obsługa żądań zewnętrznych

Akcje w oknach dialogowych