# Zasady

## Definicje:

* Domena aplikacji - główne zagadnienie do obsługi którego utworzono aplikację    
* Encja - zestaw atrybutów wspólnie opisujących złożony obiekt. Atrybuty są typów prostych.
* Rekord - Encja zapisana na dysku rozszerzona o unikalny identyfikator.  


## Architektura

Architektura aplikacji rzutowana jest na strukturę katalogów przechowujących jej źródła.

Kod dzielony jest na **kontrolery**, **akcje**, **stany**, **algorytmy**, **dane**, **repozytoria**. Podział ma swoje odzwierciedlenie w strukturze katalogów i przestrzeni nazw. Kontroler może rozpoczynać swoje działanie na danych wejściowych a także wysyłać te dane na zewnątrz aplikacji. Analizę danych wejściowych wykonują **wejścia**. Przygotowanie danych wyjściowych wykonują **wyjścia**. Główne zadanie wykonuje za pomocą **akcji**.
**Wejścia**, **wyjścia**, **akcje**, **stany** wymieniają się **danymi**. **Akcja** pozostawia po sobie ślad w postaci zmiany stanu aplikacji lib/i zapisu **danych** w **repozytorium**. **Stan** klasy singletony, na podstawie własnych mechanizmów i wywołujących je akcji modyfikują przechowywane przez siebie zmienne aplikacji.

### Kontroler
**namespace ctr**

Jego zdaniem jest wykonać akcję zgodną z odebranym sygnałem. Kontroler ma dostęp do klas wykonujących akcje do klas obsługujących dane wejściowe i klas generujących dane wyjściowe. Obsługuje sygnały od interfejsów graficznych (menu, przyciski, pól edycyjnych, itp) oraz wejść, klawiatura, sieć, porty usb... . Ponieważ kontroler umieszczony jest na "granicach" aplikacji będzie do niego należało przygotowanie danych wejściowych dla akcji i utworzenie strumienia wyjściowego z danych zwróconych przez akcję. 

### Akcja
**namespace act**

Obiekt uruchamiany na potrzeby konkretnego zadnia. Ma dostęp do algorytmów, stanów, danych i repo. Raczej zawsze będzie tworzony przez kontroler w celu wykonania czynności rozpoznanej przez kontroler. Akcje opisują funkcjonalność aplikacji z perspektywy klienta, operatora. Wszystko co można wykonać na aplikacji jako jej klient musi być opisane klasą umieszczoną na liście akcji. Lista akcji dla czytelnika jest podstawowym dokumentem o aplikacji jest definicją aplikacji.

### Stan
**namespace stt**

Obiekt istniejący jako singleton wywoływany na potrzeby konkretnego zadania, różni się od akcji tym, że pamięta stan pomiędzy wywołaniami. Ma dostęp do algorytmów, danych i repo. Prawdopodobnie będzie posiadał uruchomiony wątek.

### Algorytm
**namespace alg** 

Nie ma dostępu do innych przestrzeni. Prosta funkcja (może klasa) przetwarzająca, modyfikująca, licząca.

### Dane
**namespace data::**

Złożone typy danych wykorzystywane przez operatory i algorytmy.
Przechowuje w bazie danych pozwala wyszukiwać i modyfikować. Model zakłada się z rekordów i ich kolekcji. Kolekcje realizują wszystkie operacje na zbiorach danych.
Proste typu danych np: ```typedef int Kwota``` tworzone w pliku przeznaczonym na definicje, w ciele innych klas a w razie konieczności umieszczane w domenowej przestrzeni nazw.

**namespace data::entity** encja 

**namespace data::record** encja przechowywana na dysku, wyposażona w identyfikator

**namespace data::map** Zestawienia

**namespace data::repo** przechowyanie na dysku


### Wejście
**namespace in**

Analiza danych wejściowych. Sprawdzenie ich poprawności pod względem bezpieczeństwa aplikacji oraz zgodnością z wykonywanym zadaniem.

### Wyjście
**namespace out**

Przygotowuje dane do wysłania na zewnątrz aplikacji.



## Zależności

```c++
// Metoda wywołana gdy rozpoznano żądanie przesłania faktury. 
void Controller::GetFaktura(input data) {
	in::GetFaktura input(data);
	if (input.parse()) {
		data::map::GetFaktura params = input.getParams();
		act::GetFaktura getFaktura(params);
		if(getFaktura.action()) {
			
            data::map::Faktura szukanaFaktura = getFaktura.faktura();

            out::GetFaktura output(szukanaFaktura);

            std::out << output;
		} else {
			// Nie udało się wykonać zadania.
			// Od kontekstu będzie zależało czy nie powodzeniem jest nie znalezienie 
            // faktury czy problem z wykonaniem czynności 
		}
	} else {
		// obsługa błędnych danych wejściowych
	}
}

```



## Procedury 

### Obsługa żądań zewnętrznych