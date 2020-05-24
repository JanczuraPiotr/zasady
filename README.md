# zasady


## Architektura

Architektura aplikacji rzutowana jest na strukturę katalogów przechowujących jej źródła.

Kod dzielony jest na **kontrolery**, **akcje**, **serwisy**, **algorytmy**, **dane**, **repozytoria**. Akcje i serwisy to **operatorzy**, miejsca zmieniające stan aplikacji, zmienne aplikacji i magazyny danych. Podział ma swoje odzwierciedlenie w strukturze katalogów. Kontroler może rozpoczynać swoje działanie na danych wejściowych a także wysyłać te dane na zewnątrz aplikacji. Analizę danych wejściowych wykonują **wejścia**. Przygotowanie danych wyjściowych wykonują **wyjścia**.
Podział kodu odzwierciedlony jest w przestrzeni nazw.

### Kontroler

Obsługuje sygnały od interfejsów graficznych (menu, przyciski, ...) oraz wejść, klawiatura, sieć, porty usb... . Nie ma dostępu do modelu i algorytmów.
Odbierając sygnał kontroler analizuje dane wejściowe pod względem spójności dla swojego kontekstu i tworzy obiekt danych. Uruchamia obiekt przeznaczony do obsługi sygnału i przekazuje mu dane odebrane na wejściu. Odbiera od operatora obiekt danych będący wynikiem jego pracy i za pomocą obiektu przeznaczonego do przygotowania danych zwracanych na wyjście obsługuje w zależności od kontekstu (rest api, socket , konsola, plik).
Kontroler w jednym zadaniu może wykonać kilka operatorów.

### Akcja
Obiekt tworzony dynamicznie na potrzeby konkretnego zadnia. Ma dostęp do algorytmów, danych i repo. 

### Serwis
Obiekt istniejący jako singleton wywoływany na potrzeby konkretnego zadania, różni się od akcji tym, że pamięta stan pomiędzy wywołaniami. Ma dostęp do algorytmów, danych i repo.

### Algorytm
Klasa/funkcja przetwarzająca dane wejściowe, zwraca wynik - nie modyfikuje niczego w aplikacji. Nie ma dostępu do operatorów i modelu.

### Dane
Złożone typu danych wykorzystywane przez operatory i algorytmy.
Przechowuje w bazie danych pozwala wyszukiwać i modyfikować. Model zakłada się z rekordów i ich kolekcji. Kolekcje realizują wszystkie operacje na zbiorach danych.

### Wejście
Analiza danych wejściowych. Sprawdzenie ich poprawności pod względem bezpieczeństwa aplikacji oraz zgodnością z wykonywanym zadaniem.

### Wyjście
Przygotowuje dane do wysłania na zewnątrz aplikacji.
