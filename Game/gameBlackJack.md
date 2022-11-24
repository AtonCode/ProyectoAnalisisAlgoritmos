# Juego de Blackjack hecho con Python

El blackjack es un juego de cartas popular que se juega en la mayoría de los casinos. Esta es una intuición para replicar el mismo juego de cartas usando el programa Python.
Este código usa la línea de comando para que las entradas de los usuarios sean interactivas.

## Reglas del Blackjack

Breve conjunto de reglas para lectores que nunca han jugado Blackjack. El número mágico para el Blackjack es 21. Los valores de todas las cartas repartidas a un jugador se suman y si la suma supera los 21, el jugador se pasa y pierde instantáneamente. Si un jugador obtiene un 21 exacto, el jugador gana contra el crupier. De lo contrario, para ganar, la suma de las cartas del jugador debe ser mayor que la suma de las cartas del crupier. Cada figura tiene un valor definido de 10, mientras que el as se puede contar como 1 u 11 según las posibilidades de ganar del jugador. El valor del resto de cartas se define por su número.

## Módulo utilizado

Los módulos en Python pueden tener algunas clases, funciones y variables.

- En este **Proyecto Python** he usado el módulo **Random**, para que las cartas repartidas no estén sesgadas para el jugador y el crupier.
- En el módulo aleatorio, **random.choice()** se usa para seleccionar una carta del mazo de cartas.

## Conceptos utilizados

- Usó los conceptos de **Declaraciones condicionales** (`if` y `else`), **Declaraciones en bucle** (`for` y `while`)
- **Funciones** usadas para reducir la codificación repetitiva de las mismas condiciones.

## Diseño de juego

### Algunos valores fundamentales

Cada juego de cartas requiere valores fundamentales como las cartas y los valores de cada carta. En este juego le damos al As un valor de 1/11, Rey-Reina-Jota y Diez un valor de 10 y el resto de las cartas toman su respectivo valor

``` Python
cards = [11, 2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10]
```

### Dealing card

Con la ayuda del módulo 'random' he usado 'random.choice()' para seleccionar una carta para el mazo que creé anteriormente
y envolví la función 'deal_card ()' para usarla sin repetir el código todo el tiempo

``` Python
def deal_card():
    cards = [11, 2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10]
    rand_card = random.choice(cards)
    return rand_card
```

Con el uso de `for` y `range`, se reparten 2 cartas al usuario y al crupier y se almacenan en la lista `users_card` y `dealer_card`

``` Python
user_card = []
dealer_card = []
is_game_over = False

for _ in range(2):
    user_card.append(deal_card())
    dealer_card.append(deal_card())
```

### Calculate and Compare of scores

Para calcular las puntuaciones de la carta repartida se crea una función para conocer los resultados

``` Python
def calculate_score(cards):
    if sum(cards) == 21 and len(cards) == 2:
        return 0
    if 11 in cards and sum(cards) > 21:
        cards.remove(11)
        cards.append(1)
    return sum(cards)
```

Para comparar las puntuaciones obtenidas por el crupier y el usuario, los resultados se devuelven como las reglas del Blackjack

``` Python
user_score = calculate_score(user_card)
dealer_score = calculate_score(dealer_card)

def compare_scores(dealercards, usercards):
    if user_score == 0:
        return "You win with a blackjack"
    elif dealer_score == 0:
        return "Dealer has a Blackjack "
    elif user_score == dealer_score :
        return "PUSH"
    elif user_score > 21:
        return "YOU burst,Dealer win "
    elif dealer_score > 21:
        return "Dealer burst,you win"
    elif user_score > dealer_score :
        return "You win"
    else:
        return "Dealer win"
```

### The Game

Hemos usado un bucle while para ejecutar el juego ya que no sabemos cuándo se parará el usuario. He definido `is_game_over` como una bandera para mantener el control de la terminación del bucle.

``` Python
while not is_game_over:
    user_score = calculate_score(user_card)
    computer_score = calculate_score(dealer_card)
    print(f"Your cards are {user_card} and score is {user_score}")
    print(f"Dealer's first card: {dealer_card[0]} ")

    if user_score == 0 or dealer_score == 0 or user_score > 21:
        is_game_over = True
    else:
        if user_score == 21 :
            is_game_over = True
        hit = input("Type 'Y' to hit and 'N' to stand\n").upper()
        if hit == "Y":
            user_card.append(deal_card())
        elif hit == "N":
            is_game_over = True
        else:
            is_game_over = True

while dealer_score != 0 and dealer_score < 17:
    dealer_card.append(deal_card())
    dealer_score = calculate_score(dealer_card)
```

### The result

El ganador del juego se muestra después de comparar los puntajes de todas las cartas repartidas al repartidor y al usuario.

``` Python
print(f"computer cards: {dealer_card}",)
print(f"compute score: {dealer_score}")
print(compare_scores(dealer_score, user_score))
```
