# Zasady

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

### Akcja

``namespace act``

Obiekt uruchamiany na potrzeby konkretnego zadnia. Ma dostęp do algorytmów, stanów, danych i repo. Raczej zawsze będzie tworzony przez port lub podczas obsługi zdarzenia. Akcje opisują funkcjonalność aplikacji z perspektywy klienta, operatora. Wszystko co można wykonać na aplikacji jako jej klient musi być opisane klasą umieszczoną na liście akcji. Lista akcji dla czytelnika jest podstawowym dokumentem o możliwościach aplikacji (jest definicją aplikacji).

Dostęp do stanów, danych i algorytmów.

### Zdarzenie

``namespace ev``

Obsługuje sygnały od interfejsów graficznych (menu, przyciski, pól edycyjnych, itp) oraz wejść, klawiatura.

Dostęp do akcji i danych.

### Port

``namespace prt``

Jego zdaniem jest wykonać akcję zgodną z odebranym sygnałem. Port ma dostęp do klas wykonujących akcje, do klas obsługujących dane wejściowe i klas generujących dane wyjściowe.  Obsługuje sieć, porty USB konsole. Ponieważ port umieszczony jest na "granicach" aplikacji będzie do niego należało przygotowanie danych wejściowych dla akcji i utworzenie strumienia wyjściowego z danych zwróconych przez akcję. 

Dostęp do wejść wyjść i akcji

### Stan
``namespace stt``

Singleton lub inaczej utworzona klasa trwająca przez cały czas życia aplikacji. Na podstawie własnych mechanizmów i wywołujących je akcji modyfikują przechowywane przez siebie zmienne. Prawdopodobnie będzie posiadał uruchomiony wątek.

Dostęp do danych.

### Algorytm
``namespace alg``

Prosta funkcja (może klasa) przetwarzająca, modyfikująca, licząca.

Dostęp do danych.

### Dane
``namespace data``

Złożone typy danych wykorzystywane przez porty, zdarzenia, akcje i algorytmy.
Przechowuje w bazie danych pozwala wyszukiwać i modyfikować.
Proste typu danych np: ```typedef int Kwota``` tworzone w pliku przeznaczonym na definicje, w ciele innych klas a w razie konieczności umieszczane w domenowej przestrzeni nazw.

``namespace data::entity`` encja 

``namespace data::filtr`` Klasy definiujące listę pól dla wyszukiwania.

``namespace data::record`` encja przechowywana na dysku, wyposażona w identyfikator

``namespace data::map`` Zestawienia

``namespace data::repo`` przechowywanie na dysku


### Wejście
``namespace in``

Analiza danych wejściowych. Sprawdzenie ich poprawności pod względem bezpieczeństwa aplikacji oraz zgodnością z wykonywanym zadaniem.

### Wyjście
``namespace out``

Przygotowuje dane do wysłania na zewnątrz aplikacji.



## Przypadki użycia. 

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
    data::filtr::Faktura getFaktura();
	// ...
}

// Metoda wywołana gdy rozpoznano żądanie przesłania faktury. 
void Port::getFaktura(net::Buffer data) {
	in::GetFaktura input(data);
	if (input.parse()) {
		data::map::GetFaktura params = input.getParams();
		act::GetFaktura getFaktura(params);
		if(getFaktura.action()) {
        	data::map::Faktura szukanaFaktura = getFaktura.faktura();
        	out::GetFaktura output(szukanaFaktura);
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

}
```



## Procedury 

### Obsługa żądań zewnętrznych