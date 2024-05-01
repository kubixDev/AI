 # 📗 Projekt 3 - drzewa decyzyjne


## ⭐ Zadanie 1
Zaimplementuj klasę DecisionTree, która może być użyta w poniższym skrypcie do treningu

<br>

**Dostarczony skrypt do testu działania:**

```
from sklearn import datasets
from sklearn.model_selection import train_test_split
from decision_tree import DecisionTree
import numpy as np

data = datasets.load_iris()
X, y = data.data, data.target

X_train, X_test, y_train, y_test = train_test_split(
	X, y, test_size=0.2, random_state=1234
)

clf = DecisionTree(max_depth=10)
clf.fit(X_train, y_train)
predictions = clf.predict(X_test)

def accuracy(y_test, y_pred):
	return np.sum(y_test == y_pred) / len(y_test)

acc = accuracy(y_test, predictions)
print(acc)
```

<br>

**Implementacja DecisionTree:**

```
import numpy as np
from collections import Counter

##############################
# Wezel w drzewie decyzyjnym
##############################
class Node:
    def __init__(self, feature = None, threshold = None, left = None, right = None, *, value = None):
        self.feature = feature       # zmienna na podstawie ktorej dokonujemy podzialu w drzewie
        self.threshold = threshold   # prog podzialu
        self.left = left             # lewe poddrzewo
        self.right = right           # prawe poddrzewo
        self.value = value           # wartosc w lisciu (jesli to lisc)

    def is_leaf_node(self):
        return self.value is not None


####################################
# Implementacja drzewa decyzyjnego
####################################
class DecisionTree:
    def __init__(self, min_samples_split = 2, max_depth = 100, n_features = None):
        self.min_samples_split = min_samples_split   # minimalna liczba probek do podzialu wezla
        self.max_depth = max_depth                   # maksymalna glebokosc drzewa
        self.n_features = n_features                 # liczba cech branych pod uwage przy kazdym podziale
        self.root = None                             # korzen drzewa


    #######################################
    # Metoda inicjujaca trenowanie modelu 
    #######################################
    def fit(self, X, y):
        if self.n_features is None:
            self.n_features = X.shape[1]                          # domyslnie liczba cech rowna liczbie cech w danych
        else:
            self.n_features = min(self.n_features, X.shape[1])    # jesli podano inna liczbe cech, to bierzemy minimum

        self.root = self._grow_tree(X, y)


    ##########################################
    # Metoda do budowania drzewa decyzyjnego
    ##########################################
    def _grow_tree(self, X, y, depth = 0):
        n_samples, n_features = X.shape
        n_labels = len(np.unique(y))

                                                                   # Sprawdzamy kryteria zatrzymania:
        if (depth >= self.max_depth                                #   - czy osiagnelismy maksymalna glebokosc
                or n_samples < self.min_samples_split              #   - czy liczba probek jest mniejsza niz minimalna wymagana do podzialu
                or n_labels == 1):                                 #   - czy wszystkie probki naleza do tej samej etykiety
            leaf_value = Counter(y).most_common(1)[0][0]
            return Node(value=leaf_value)                          # zwracamy wartosc najczesciej wystepujacej etykiety


        # znajdowanie najlepszego podzialu
        feat_idxs = np.random.choice(n_features, self.n_features, replace=False)
        best_feature, best_threshold = self._best_split(X, y, feat_idxs)

        # podzial zbioru danych
        left_idxs, right_idxs = self._split(X[:, best_feature], best_threshold)
        left = self._grow_tree(X[left_idxs, :], y[left_idxs], depth + 1)
        right = self._grow_tree(X[right_idxs, :], y[right_idxs], depth + 1)
        
        return Node(best_feature, best_threshold, left, right)


    ####################################################
    # Metoda szukajaca najlepszego podzialu dla danych
    ####################################################
    def _best_split(self, X, y, feat_idxs):
        best_feature, best_threshold = None, None
        best_gain = -1

        for feat_idx in feat_idxs:
            X_column = X[:, feat_idx]
            thresholds = np.unique(X_column)

            for threshold in thresholds:
                gain = self._information_gain(y, X_column, threshold)

                if gain > best_gain:
                    best_feature = feat_idx
                    best_threshold = threshold
                    best_gain = gain


        return best_feature, best_threshold


    ############################################################
    # Metoda obliczajaca zysk informacyjny dla podziału danych
    ############################################################
    def _information_gain(self, y, X_column, threshold):
        left_idxs, right_idxs = self._split(X_column, threshold)

        if len(left_idxs) == 0 or len(right_idxs) == 0:
            return 0

        y_left = y[left_idxs]
        y_right = y[right_idxs]
        entropy_left = self._entropy(y_left)
        entropy_right = self._entropy(y_right)
        entropy_split = (len(left_idxs) * entropy_left + len(right_idxs) * entropy_right) / len(y)
        entropy_total = self._entropy(y)

        return entropy_total - entropy_split


    ###########################################
    # Metoda dzielaca dane na podstawie progu
    ###########################################
    def _split(self, X_column, split_thresh):
        left_idxs = np.where(X_column <= split_thresh)[0]
        right_idxs = np.where(X_column > split_thresh)[0]

        return left_idxs, right_idxs


    ##########################################################
    # Metoda obliczajaca entropie dla danego zestawu etykiet
    ##########################################################
    def _entropy(self, y):
        hist = np.bincount(y)
        ps = hist / len(y)

        return -np.sum([p * np.log(p) for p in ps if p > 0])


    ####################################
    # Metoda przechodzaca przez drzewo
    ####################################
    def _traverse_tree(self, x, node):
        if node.is_leaf_node():
            return node.value

        if x[node.feature] <= node.threshold:
            return self._traverse_tree(x, node.left)
        
        else: return self._traverse_tree(x, node.right)


    ############################################################
    # Metoda przewidujaca etykiety dla danego zestawu probek X
    ############################################################
    def predict(self, X):
        return np.array([self._traverse_tree(x, self.root) for x in X])
 ```

<br>

****Wyjaśnienie:****

> def fit(self, X, y):

* Jest to metoda, która inicjuje trenowanie modelu drzewa decyzyjnego
* **X** - dane treningowe, macierz (zbiór cech)
* **y** - odpowiadające im etykiety (oczekiwane wartości)
* sprawdzamy czy użytkownik podał wartość **n_features** (liczbę cech)
* w pierwszym przypadku, kiedy nie została podana żadna wartość, to liczba cech jest ustawiana na liczbę wszystkich cech w danych **X**
* w drugim przypadku, kiedy wpiszemy własną ilość, zostanie przypisana mniejsza wartość (własna ilość/ilość cech w danych X), aby nie przekroczyć z góry ustalonej liczby cech z danych **X**
* fragment **X.shape[1]** zwraca liczbę kolumn w macierzy danych **X**. Wiersz zawiera jedną próbkę danych, a kolumna konkretną cechę
* na końcu przypisujemy do korzenia wynik metody grow_tree, która zwraca zbudowane drzewo decyzyjne

<br>

> def _grow_tree(self, X, y, depth=0):

* Jest to metoda, która odpowiada za budowanie drzewa
* na początku przypisujemy liczbę próbek oraz liczbę cech do odpowiadających im zmiennych przy użyciu **X.shape** (zwraca krotkę typu: liczba wierszy, liczba kolumn)
* zmienna **n_labels** zawiera ilość unikalnych etykiet, im większa liczba, tym lepiej - oznacza to większą różnorodność dostarczonych danych
* instrukcja if sprawdza warunki zatrzymania rekurencyjnej budowy drzewa
	* jeżeli osiągnęliśmy z góry ustaloną głębokość **max_depth**
	* albo jeżeli liczba próbek **n_samples** jest zbyt mała, aby kontynuować podział
	* albo jeżeli liczba unikalnych etykiet **n_labels** wynosi 1, co oznaczałoby że nie są unikalne i podział nie ma sensu
* następnie w środku ifa przypisujemy do **leaf_value** najczęściej występującą etykietę (przy pomocy licznika Counter) i zwracamy to jako wartość liścia drzewa
* w przypadku kiedy warunki końca nie zostały spełnione, przypisujemy do tablicy indeksów **feat_idxs** losowo pewną liczbę cech, które będą brane pod uwagę podczas budowy drzewa
* następnie metoda **_best_split** znajduje najlepszą cechę (best_feature) oraz najlepszy próg podziału (best_threshold) spośród wylosowanych cech
* dalej dzielimy dane na dwie grupy na podstawie wybranej cechy i progu
	* jedna grupa dla próbek, gdzie wartość tej cechy jest mniejsza lub równa progowi
	* druga grupa dla pozostałych próbek
* potem dla obu grup budujemy rekurencyjnie osobne poddrzewo decyzyjne
* na końcu tworzymy węzeł drzewa zawierający najlepszą cechę, najlepszy próg podziału oraz odnośniki do lewego i prawego poddrzewa

<br>

> def _best_split(self, X, y, feat_idxs):

* Jest to metoda, która znajduje najlepszy podział danych wejściowych
* inicjalizujemy best_feature oraz best_threshold
* ustawiając **best_gain** na wartość ujemną mamy pewność, że każdy dodatni (czyli poprawny) zysk informacyjny będzie od tego większy
* następnie iterujemy po każdym indeksie cechy
* dla każdej cechy pobieramy kolumnę danych z macierzy **X**
* potem znajdujemy unikalne wartości, które posłużą jako progi podziału
* w końcu, dla każdego progu podziału obliczamy zysk informacyjny
* na otrzymanej wartości realizujemy algorytm max (który zamieni wartości, gdy nowo obliczony gain będzie większy od obecnie największego)
* ostatecznie zwracamy najlepszy podział

<br>

> def _information_gain(self, y, X_column, threshold):

* Jest to metoda, która oblicza zysk informacyjny dla podziału danych
* na początku znowu dzielimy dane na dwie grupy na podstawie progu podziału
	* jedna grupa dla próbek, gdzie wartość cechy jest mniejsza lub równa progowi
	* druga grupa dla pozostałych próbek
* warunek if sprawdza sytuację, w której po podziale któraś z grup jest pusta (czyli nie zawiera żadnych próbek) - w takiej sytuacji zwracamy 0, czyli **brak zysku** informacyjnego
* jeżeli jednak tak się nie stało, obliczamy entropię dla każdej z tych dwóch grup
	* **y_left** oraz **y_right** przypisują etykiety dla próbek po lewej i prawej stronie podziału
	* **entropy_left** oraz **entropy_right** obliczają entropię dla etykiet w odpowiednich grupach
	* **entropy_split** oblicza średnią ważoną z entropii obu grup po podziale
	* **entropy_total** oblicza początkową entropię przed podziałem danych
* wynik to zysk informacyjny (różnica między entropią przed i po podziale danych)
* jeżeli różnica jest duża, to znaczy że podział był skuteczny

<br>

> def _split(self, X_column, split_thresh):

* Jest to metoda, która odpowiada za podział danych na podstawie progu dla konkretnej cechy
* funkcja **np.where** przeszukuje tablicę i zwraca indeksy elementów, dla których warunek jest spełniony
* dokonujemy podziału indeksów próbek na dwie grupy na podstawie progu
	* **left_idxs** znajduje indeksy elementów w **X_column**, które są mniejsze lub równe niż wartość progu podziału
	* **right_idxs** znajduje indeksy elementów w **X_column**, które są większe niż wartość progu podziału
* konieczne jest dodanie **[0]** na końcu, ponieważ domyślnie np.where zwraca krotkę z jednym elementem, a po dodaniu zamienimy to na zwykłą tablicę z indeksami
* na koniec zwracamy te tablice

<br>

> def _entropy(self, y):

* Jest to dostarczona metoda, która oblicza entropię dla podanego zestawu etykiet
* entropia to miara nieuporządkowania w zbiorze danych, **mierzy różnorodność etykiet**
* im większa entropia, tym bardziej zróżnicowane mamy etykiety
* natomiast entropia równa zero oznacza, że wszystkie etykiety w zbiorze są identyczne
* jest obliczana z gotowego schematu
	* najpierw obliczamy ilość wystąpień każdej etykiety w zbiorze danych
	* następnie obliczamy prawdopodobieństwo wystąpienia każdej etykiety
	* używamy wzoru i sumujemy wyniki dla wszystkich etykiet
* wynik to obliczona entropia 

<br>

> def _traverse_tree(self, x, node):

* Jest to metoda, która przechodzi przez drzewo w celu przewidzenia etykiety dla określonej próbki
* **x** - próbka, dla której chcemy przewidzieć etykietę
* **node** - bieżący węzeł, od którego rozpoczynamy przeszukiwanie
* początkowa instrukcja if sprawdza, czy bieżący węzeł jest liściem (czyli elementem skrajnym drzewa, z którym nic już się nie łączy)
* jeżeli tak, to od razu możemy zwrócić wartość tego węzła
* w przeciwnym wypadku sprawdzamy czy próbka **x** spełnia warunek podziału dla danej cechy i progu
* w zależności od wyniku przechodzimy do lewego lub prawego poddrzewa
* powtarzamy to rekurencyjnie aż dotrzemy do liścia i zwrócimy jego wartość

<br>

> def predict(self, X):

* Jest to dostarczona metoda, która służy do przewidywania etykiet dla zestawu próbek, używająca wytrenowanego drzewa decyzyjnego
* iterujemy przez każdą próbkę w zestawie **X**
* dla każdej próbki przewidujemy etykietę (metoda **_traverse_tree**) 
* na końcu zbieramy i zwracamy przewidziane etykiety w tablicy
