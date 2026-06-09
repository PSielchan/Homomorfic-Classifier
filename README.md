# Homomorphic Classifier

Projekt demonstruje klasyfikację danych z użyciem szyfrowania homomorficznego.
Notebook trenuje klasyczne modele regresji logistycznej na
danych jawnych, a następnie wykonuje etap inferencji na zaszyfrowanych próbkach
przy pomocy biblioteki TenSEAL i schematu CKKS.

Główna idea projektu:

1. Model jest trenowany lokalnie na danych niezaszyfrowanych.
2. Próbka testowa jest szyfrowana jako wektor CKKS.
3. Klasyfikator liniowy liczy homomorficznie wynik `w * x + b`.
4. Wynik pozostaje zaszyfrowany do momentu odszyfrowania przez właściciela danych.
5. Predykcja klasy powstaje po odszyfrowaniu wyniku lub listy wyników dla klas.

## Zawartość repozytorium

- `krypto_projekt.ipynb` - notebook z eksperymentami i wynikami.
- `README.md` - opis projektu, wymagania i instrukcja uruchomienia.


## Parametry szyfrowania

W notebooku używany jest schemat CKKS z biblioteki TenSEAL:

- `poly_modulus_degree = 8192`
- `coeff_mod_bit_sizes = [60, 40, 40, 60]`
- `global_scale = 2**40`
- generowane są klucze Galois potrzebne do operacji wektorowych.

CKKS jest schematem przybliżonym, dlatego odszyfrowane wyniki mogą minimalnie
różnić się od obliczeń wykonanych na danych jawnych.

## Eksperymenty

### 1. Iris - klasyfikacja binarna

Pierwsza część notebooka korzysta z dwóch pierwszych klas zbioru Iris. Regresja
logistyczna jest trenowana na 4 cechach, a następnie pojedyncza próbka testowa
jest szyfrowana i klasyfikowana homomorficznie.


### 2. Iris - klasyfikacja wieloklasowa

Druga część używa pełnego zbioru Iris z trzema klasami. Model regresji logistycznej
zwraca osobny wektor wag dla każdej klasy. Dla zaszyfrowanej próbki liczone są
trzy zaszyfrowane score'y, a po odszyfrowaniu wybierana jest klasa z największą
wartością.

Wyniki dla testu homomorficznego:

| Metryka | Wartość |
| --- | ---: |
| Accuracy | 1.0000 |
| Precision weighted | 1.0000 |
| F1 weighted | 1.0000 |

### 3. Breast Cancer Wisconsin - klasyfikacja binarna

Trzecia część pokazuje analogiczny klasyfikator dla zbioru Breast Cancer
Wisconsin. Dane mają 569 próbek i 30 cech. Przed treningiem są standaryzowane
przez `StandardScaler`.

Model jawny osiągnął accuracy `0.9825`. Klasyfikacja homomorficzna na zbiorze
testowym uzyskała:

| Metryka | Wartość |
| --- | ---: |
| Accuracy | 0.9825 |
| Precision | 0.9861 |
| Recall | 0.9861 |
| F1-Score | 0.9861 |


### 4. Letter Recognition - klasyfikacja wieloklasowa

Ostatnia część korzysta ze zbioru `letter` z OpenML. Zbiór zawiera 20000 próbek,
16 cech i 26 klas odpowiadających literom `A-Z`.

Model regresji logistycznej jest trenowany na danych jawnych, a dla każdej
zaszyfrowanej próbki liczonych jest 26 score'ów:

```text
score_i = w_i * x + b_i
```

Ze względu na koszt obliczeń homomorficznych notebook testuje domyślnie po 5
próbek z każdej litery, czyli łącznie 130 próbek.

Wyniki modelu jawnego na pełnym zbiorze testowym:

| Metryka | Wartość |
| --- | ---: |
| Accuracy | 0.7695 |
| Precision weighted | 0.7699 |
| Recall weighted | 0.7695 |
| F1 weighted | 0.7687 |

Wyniki klasyfikacji homomorficznej na podzbiorze 130 próbek:

| Metryka | Wartość |
| --- | ---: |
| Accuracy | 0.8077 |
| Precision weighted | 0.8225 |
| Recall weighted | 0.8077 |
| F1 weighted | 0.8025 |
| Zgodność z modelem jawnym | 1.0000 |
| Maksymalny błąd score CKKS | 0.00000502 |
| Średni błąd score CKKS | 0.00000121 |

Czas zapisany w notebooku:

- całkowity czas klasyfikacji homomorficznej: `24.13 s`
- średni czas na próbkę: `0.1856 s`

## Ograniczenia

- Projekt używa regresji logistycznej, ponieważ liniowy score `w * x + b` dobrze
  pasuje do operacji wspieranych przez CKKS.
- Trening odbywa się na danych jawnych; szyfrowany jest etap inferencji.
- Wybór klasy przez `argmax` wykonywany jest po odszyfrowaniu score'ów.
- Pełny test homomorficzny dla zbioru Letter Recognition byłby kosztowny, dlatego
  notebook używa reprezentatywnego podzbioru próbek.

