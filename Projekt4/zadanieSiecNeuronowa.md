  # 📗 Projekt 4 - sztuczne sieci neuronowe


## ⭐ Zadanie 1
Wykorzystując kod klasy Neuron zaimplementuj sieć neuronową z poniższą strukturą oraz dostarcz kod do wizualizacji struktury sieci.

<br>

**Dostarczony kod:**

```
import numpy as np 

class Neuron:  
    def __init__(self, n_inputs, bias = 0., weights = None):  
        self.b = bias
        if weights: self.ws = np.array(weights)
        else: self.ws = np.random.rand(n_inputs)

    def _f(self, x):                                            # Activation function (here: leaky_relu)
        return max(x*.1, x)   

    def __call__(self, xs):                                     # Calculate the neuron's output:
                return self._f(xs @ self.ws + self.b)           #   - multiply the inputs with the weights and sum the values together
                                                                #   - add the bias value
                                                                #   - then transform the value via an activation function
```

<br>

**Implementacja Neuron wraz z siecią neuronową ANN:**

```
import numpy as np
import matplotlib.pyplot as plt
import networkx as nx

##############################
# Dostarczona klasa Neuron
##############################
class Neuron:
    def __init__(self, n_inputs, bias = 0.0, weights = None):       # Konstruktor klasy Neuron
        self.b = bias

        if weights is not None:
            self.ws = np.array(weights)
        else:
            self.ws = np.random.rand(n_inputs)


    def _f(self, x):                                                # Metoda definiujaca funkcje aktywacji
        return max(0.01 * x, x)                                     # z tabelki jest to Leaky ReLU


    def __call__(self, xs):                                         # Pozwala na wywolanie obiektu klasy
        return self._f(np.dot(xs, self.ws) + self.b)                # neuron jak funkcji



########################################################
# Implementacja klasy ANN (Artificial Neural Network)
########################################################
class ANN:
    def __init__(self, layers):                                     # Konstruktor klasy ANN
        self.layers = []
        
        for i in range(1, len(layers)):
            layer = []

            for _ in range(layers[i]):
                neuron = Neuron(layers[i - 1])
                layer.append(neuron)

            self.layers.append(layer)


    def feedforward(self, inputs):                                  # Metoda, ktora przekazuje dane                          
        self.inputs = [inputs]                                      # wejsciowe przez siec neuronowa

        for layer in self.layers:
            new_inputs = []

            for neuron in layer:
                output = neuron(self.inputs[-1].reshape(-1, len(neuron.ws)))
                new_inputs.append(output)

            self.inputs.append(np.array(new_inputs))

        return self.inputs[-1]
    

    def backpropagation(self, expected_output, learning_rate):
        deltas = [expected_output - self.inputs[-1]]

        for i in reversed(range(len(self.layers))):
            layer = self.layers[i]
            next_deltas = np.zeros(len(self.inputs[i]))

            for j, neuron in enumerate(layer):
                output = self.inputs[i + 1][j]
                delta = deltas[-1][j] * (neuron._f(output) - output)
                next_deltas = next_deltas + neuron.ws * delta
                grad_w = output * delta
                grad_b = delta
                neuron.ws = neuron.ws + learning_rate * grad_w
                neuron.b = neuron.b + learning_rate * grad_b

            deltas.append(next_deltas)


    def train(self, X, y, epochs, learning_rate):
        for epoch in range(epochs):
            for xi, yi in zip(X, y):
                self.feedforward(xi)
                self.backpropagation(yi, learning_rate)
                
            print("Epoch:", epoch)



###############################
# Wizualizacja ANN z obrazka
###############################
def visualize_ann(layers):
    G = nx.DiGraph()
    pos = {}
    labels = {}
    node_id = 0


    # Tworzenie wezlow dla warstwy wejsciowej (input layer)
    for i in range(layers[0]):
        G.add_node(node_id)
        pos[node_id] = (0, -i)
        labels[node_id] = f"Input {i+1}"
        node_id = node_id + 1


    # Tworzenie wezlow dla ukrytych warstw (hidden layers)
    layer_count = len(layers) - 1

    for layer_idx, num_nodes in enumerate(layers[1:-1], start = 1):
        for i in range(num_nodes):
            G.add_node(node_id)
            pos[node_id] = (layer_idx, -i)
            labels[node_id] = f"Hidden{layer_idx} {i+1}"
            node_id = node_id + 1


    # Tworzenie wezlow dla warstwy wyjsciowej (output layer)
    for i in range(layers[-1]):
        G.add_node(node_id)
        pos[node_id] = (layer_count, -i)
        labels[node_id] = f"Output {i+1}"
        node_id = node_id + 1


    # Dodawanie krawedzi miedzy warstwami
    prev_layer_nodes = range(layers[0])
    current_layer_start = layers[0]

    for layer_size in layers[1:]:
        current_layer_nodes = range(current_layer_start, current_layer_start + layer_size)

        for prev_node in prev_layer_nodes:
            for curr_node in current_layer_nodes:
                G.add_edge(prev_node, curr_node)

        prev_layer_nodes = current_layer_nodes
        current_layer_start = current_layer_start + layer_size


    # Rysowanie sieci przy uzyciu biblioteki NetworkX
    nx.draw(G, pos, labels = labels, with_labels = True, node_size = 3000, node_color = 'lightgreen', font_size = 8)
    plt.show()



######################################
# Wybor parametrow i uzycie funkcji
######################################
input_size = 3
hidden_layer_1_size = 4
hidden_layer_2_size = 4
output_size = 1
layers = [input_size, hidden_layer_1_size, hidden_layer_2_size, output_size]

# Inicjalizacja sztucznej sieci neuronowej
ann = ANN(layers)

# Przykladowe inputy i outputy (X to liczby, a y to wyniki sum tych liczb)
X = np.array([[1, 1, 1], [1, 1, 2], [1, 1, 3], [1, 1, 4], [1, 1, 5], [1, 1, 6], [1, 1, 7]])
y = np.array([[3], [4], [5], [6], [7], [8], [9]])

# Testowy input dla ktorego bedzie zwracal wynik (reshape dla poprawnego ksztaltu macierzy)
test_input = np.array([1, 1, 8]).reshape(-1, 3)

# Rozpoczecie trenowania i zwrocenie wyniku dla testowego inputu
ann.train(X, y, epochs=5000, learning_rate=0.01)
predicted_sum = ann.feedforward(test_input).flatten()[0]
print(predicted_sum)

# Wizualizacja struktury
visualize_ann(layers)
 ```

<br>

****Importy:****

* **numpy** - do wykonywania operacji matematycznych
*  **matplotlib.pyplot** - do rysowania wykresów
*  **networkx** - do tworzenia i wizualizacji grafów

<br>

****Wyjaśnienie:****

> [Neuron] def  \_\_init__(self, n_inputs, bias = 0.0, weights = None):

* Jest to dostarczony konstruktor klasy Neuron
* **n_inputs** - liczba wejść dla neuronu
* **bias** - przesunięcie neuronu, domyślnie "0.0"
* **weights** - wagi dla wejść neuronu, domyślnie "None"
* na początek ustawia wartość biasu z konstruktora w nowej zmiennej
* następnie warunek sprawdza czy podano parametr wagi
* jeśli są podane, przekształcamy je w tablicę numpy i zapisujemy do zmiennej
* w przeciwnym przypadku tworzymy losowe wagi o długości **n_inputs** i również zapisujemy do zmiennej

<br>

> [Neuron] def _f(self, x):

* Jest to dostarczona metoda, która definiuje funkcję aktywacji
* **x** - wartość wejściowa do funkcji aktywacji
* w tym przypadku została nam podana funkcja Leaky ReLU, która zwraca wartość **x**, jeśli x jest dodatnie, a **0.1 * x**, jeśli x jest ujemne
* funkcja aktywacji wprowadza nieliniowość, co pozwala sieci neuronowej na modelowanie skomplikowanych zależności
* działa jak bramka lub przełącznik, który decyduje, czy neuron wyśle sygnał dalej

<br>

> [Neuron] def  \_\_call__(self, xs):

* Jest to dostarczona metoda, która pozwala traktować obiekt klasy Neuron jak funkcję
* **xs** - wejściowe wartości (tablica liczb)
* fragment **np.dot()** oblicza iloczyn skalarny tablicy wejściowej **xs** i wag **ws**
* następnie do wyniku iloczynu dodawany jest bias
* ostatecznie zwracany jest wynik tego obliczenia z funkcji aktywacji

<br>

> def \_\_init__(self, layers):

* Jest to konstruktor klasy ANN (Artificial Neural Network)
* **layers** - lista, która określa liczbę neuronów w każdej warstwie
* na początku inicjalizujemy pustą listę warstw (każda warstwa to lista neuronów)
* następnie iterujemy przez indeksy od 1 do długości listy layers (czyli liczby warstw)
	* w pętli tworzona jest pusta lista **layer**, która będzie przechowywać neurony w bieżącej warstwie
	* następnie iterujemy tyle razy, ile jest neuronów w bieżącej warstwie
		* w pętli tworzymy nowy neuron, przekazując liczbę neuronów z poprzedniej warstwy jako liczbę wejść i dodajemy go do listy **layer**
	* na koniec pierwszej pętli dodajemy całą warstwę neuronów do listy warstw **self.layers**

<br>

> def feedforward(self, inputs):

* Jest to metoda, która przekazuje dane wejściowe przez sieć neuronową
* **inputs** - wejściowe wartości
* na początku inicjujemy listę wejściowych wartości do sieci
* następnie iterujemy przez każdą warstwę w sieci neuronowej
	* w pętli tworzona jest pusta lista **new_inputs**, która będzie przechowywać wyjścia neuronów z bieżącej warstwy
	* następnie iterujemy przez każdy neuron w bieżącej warstwie
		* w zmiennej **output** zapisujemy obliczony wynik dla bieżącego neuronu (przekazujemy odpowiednio sformatowane ostatnie wejście)
			* wejście musi pasować do liczby wag w bieżącym neuronie, **-1** oznacza, że liczba wierszy będzie automatycznie obliczona aby zgadzała się z liczbą wag
		* dodajemy obliczony wynik do listy **new_inputs**
	* pod koniec pierwszej pętli dodajemy całą listę **new_inputs** do listy wszystkich wejść, aby można było ją użyć jako dane wejściowe dla następnej warstwy
* zwracamy wynik - ostatnie wejście z listy (jest już po przejściu przez wszystkie warstwy sieci)

<br>

> def  backpropagation(self, expected_output, learning_rate):

* Jest to metoda, która aktualizuje wagi i biasy, aby zmniejszyć błąd wyjściowy w przyszłych iteracjach
* **expected_output** - oczekiwane wyjście sieci neuronowej
* **learning_rate** - współczynnik uczenia się (im większy, tym szybciej i mniej dokładnie)
* na początku inicjujemy listę **deltas** zawierającą różnicę między oczekiwanym a rzeczywistym wyjściem
* następnie pętla iteruje przez wszystkie warstwy sieci od końca do początku
	* w **layer** ustawiamy obecnie przetwarzaną warstwę neuronów
	* dalej tworzymy tablicę **next_deltas**, która jest wypełniona zerami i ma za zadanie przechowywać błędy dla neuronów z poprzedniej warstwy
	* wewnętrzna pętla for iteruje przez każdy neuron w aktualnej warstwie
		* w zmiennej **output** zapisujemy wyjście neuronu "j" w warstwie "i + 1"
		* w zmiennej **delta** zapisujemy błąd dla neuronu
		* następnie aktualizujemy **next_deltas** o wpływ błędu obecnego neuronu na poprzednią warstwę
		* dalej obliczamy gradient wag w **grad_w** (jak bardzo każda waga powinna się zmienić)
		* kolejno obliczamy gradient biasu w w **grad_b** (jak bardzo bias powinien się zmienić)
		* pod koniec wewnętrznej pętli aktualizujemy wagi i bias dla neuronu na podstawie gradientów i współczynnika uczenia
	* natomiast pod koniec zewnętrznej pętli dodajemy obliczone błędy dla warstwy "i" do listy **deltas**, aby można było je wykorzystać przy następnym obrocie pętli

<br>

> def  train(self, X, y, epochs, learning_rate):

* Jest to metoda, która odpowiada za proces uczenia sieci neuronowej
* **X** - wejściowe dane treningowe
* **y** - oczekiwane wyjścia dla danych wejściowych
* **epochs** - ile razy sieć przejdzie przez cały zbiór danych treningowych
* **learning_rate** - współczynnik uczenia się
* na początku pętla iteruje przez określoną liczbę epochs
	* wewnętrzna pętla iteruje po danych treningowych (funkcja **zip()** łączy elementy z dwóch list "X" i "y" w pary)
		* wywołujemy metodę **feedforward** żeby obliczyć wyjścia sieci dla "xi"
		* wywołujemy metodę  **backpropagation** żeby zaktualizować wagi na podstawie błędów między rzeczywistym a oczekiwanym wyjściem dla "yi"
	* wyświetlamy numer aktualnej wartości epoch aby śledzić postęp

<br>

> def  visualize_ann(layers):

* Jest to funkcja do wizualizacji sieci neuronowej
* **layers** - lista, która określa liczbę neuronów w każdej warstwie
* **G** - graf skierowany utworzony za pomocą biblioteki NetworkX
* **pos** - słownik pozycji węzłów (pary klucz-wartość)
* **labels** - słownik etykiet węzłów (pary klucz-wartość)
* **node_id** - identyfikator węzła, licznik
***
* na samym początku musimy stworzyć węzły dla warstwy wejściowej
* iterujemy przez liczbę neuronów w warstwie wejściowej
	* dodajemy węzeł do grafu z identyfikatorem **node_id**
	* ustawiamy pozycję węzła (w formacie współrzędnych **x, y**)
		* wszystkie będą ustawiane pod sobą w pionowej linii, bo "x" to stała wartość, a "y" zmienia się w zależności od id
	* ustawiamy etykietę węzła w formacie "Input <numer węzła>"
	* na koniec zwiększamy identyfikator węzła o 1 dla kolejnych obrotów pętli
***
* następnie tworzymy węzły dla warstw ukrytych
* w zmiennej **layer_count** obliczamy liczbę warstw ukrytych i wyjściowej
* **layer_idx** - indeks bieżącej warstwy (1 dla pierwszej warstwy ukrytej itd.)
* **num_nodes** - liczba neuronów w bieżącej warstwie
* używamy **enumerate()** aby móc iterować przez elementy listy i mieć dostęp do ich indeksów
* w tym przypadku iterujemy przez listę **layers** od elementu z indeksem 1, do elementu przedostatniego (co określa [1:-1] oraz start = 1)
* działamy tak jak poprzednio, dodając węzły do grafu, ustalając ich kolejne pozycje, nazwy oraz zwiększając identyfikator
	* w pos[node_id] znajduje się teraz **layer_idx**, co pozwoli na ułożenie każdej z warstw kolumnami przyrastającymi w dół
***
* dalej tworzymy węzeł dla warstwy wyjściowej
* iterujemy po liczbie neuronów w warstwie wyjściowej (ostatni element listy **layers**)
* działamy tak jak poprzednio, dodając węzły do grafu, ustalając ich kolejne pozycje, nazwy oraz zwiększając identyfikator
	* w pos[node_id] znajduje się teraz **layer_count**, który ustawia węzły wyjściowe na samym końcu wykresu (w tym zadaniu jest to i tak tylko jeden węzeł)
***
* ostatecznie należy dodać krawędzie między węzłami w warstwach
* **prev_layer_nodes** - przechowuje numeracje węzłów z poprzedniej warstwy
	* początkowo przypisujemy jej sekwencję liczb od 0 do "layers[0] - 1" używając metody range(), co odpowiada liczbie neuronów w warstwie wejściowej
* **current_layer_start** - określa indeks początkowy węzłów w aktualnej warstwie
	* początkowo przypisujemy jej wartość layers[0], dzięki czemu zaczynamy od pierwszego węzła w warstwie wejściowej

<br>

* zaczynamy od iterowania przez warstwy sieci neuronowej (z użyciem [1:] ponieważ już obsłużyliśmy warstwę wejściową)
	* w zmiennej **current_layer_nodes** przechowujemy numeracje węzłów aktualnej warstwy
	* dalej rozpoczynamy pętlę przez węzły z poprzedniej warstwy, aby utworzyć krawędzie
		* wewnętrzna pętla iteruje przez węzły z aktualnej warstwy
		* dodajemy krawędź między węzłem z **prev_node** a **curr_node**
	* pod koniec pętli po layers aktualizujemy **prev_layer_nodes** na wartość obecnie rozpatrywanej warstwy na potrzeby kolejnego obrotu
	* aktualizujemy również **current_layer_start** o wartość **layer_size** aby wskazać na pierwszy węzeł w następnej warstwie
***
* w końcu w funkcji rysujemy graf G na podstawie określonych parametrów za pomocą nx.draw()
* używając plt.show() wyświetlamy wykres

<br>
<br>

Pod koniec programu ustawiamy parametry warstw, aby były zgodne z dostarczoną strukturą ze zdjęcia. Inicjalizujemy sieć neuronową, tworzymy przykładowe inputy, outputy oraz wartość do przetestowania. Wyświetlamy również zwizualizowaną strukturę.
