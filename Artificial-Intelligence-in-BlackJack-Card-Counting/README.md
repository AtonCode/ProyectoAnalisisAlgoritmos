# Inteligencia Artificial en el conteo de cartas de Blackjack
Un programa que entrena una red neural de TensorFlow para jugar Blackjack y contar cartas.

# Blackjack.py
Blackjack.py es usado para generar datasets de manos de blackjack mediante simulaciones Monte Carlo. Esto lo realiza generando manos aleatorias, permitiendo que el computador haga movimientos aleatorios y guarda las representaciones de las manos con un archivo .tags que es la decisión tomada de la mano 'h' hits para tomar una carta y 's' stay para no tomar carta y dejar la mano actual.

# Representando una mano de blackjack y generar datasets

Para generar el dataset de manos de Blackjack usando simulaciones Monte Carlo se tiene la siguiente función:

<pre>
import Blackjack as blackjack
blackjack.gen_data_set( 100, "test", 1 )
</pre>

El primer parámetro es el número de manos a jugar.

El segundo parámetro es el nombre del archivo que se va a guardar en la carpeta '/data_sets'. El código Example.py genera 2 archivos '/data_sets/test.data' y '/data_sets/test.tags' Si el dataset ya existe se agregan los nuevos registros al final.

El tercer parametro es el nivel a poner en el dataset:

Nivel 1 guarda la información solo de la mano del jugador.
Nivel 2 guarda los datos del nivel 1 más la carta boca arriba del repartidor de cartas.
Nivel 3 guarda la información del nivel 2 más un registro de todas las cartas vistas.

Con el fin de entrenar una red nuerel para jugar al blackjack, queremos representar una mano de forma que nos diga si debemos 'pedir' o 'quedarnos'. Por suerte, sólo necesitamos almacenar unos pocos enteros. Entonces etiquetamos los datos como "h" o "s" para "hit" o "stay".

Se determina 's' o 'h' de acuerdo a la siguiente forma:


<pre>
//Si el usuario pide carta y se pasa
if user hits and busts:
	tag = 's'
//Si el usuario pide carta y no se pasa
elif user hits and doesn't bust:
	tag = 'h'
//Si el usuario no pide carta y gana
elif user stays and wins hand:
	tag = 's'
//Si el usuario no pide carta y no gana
elif user stays and loses hand:
	tag = 'h'
</pre>

Ejemplos de escenarios y datos esperados:

<pre>
Ejemplo 1

mano del jugador: ( 3-s, 4-c )
jugador toma carta
mano del jugador: ( 3-s, 4-c, 12-d )
judador no toma carta
el dealer se pasa de 21

genera los siguientes datos y etiquetas:
(como el juegador gano la mano se asume quet todos los movimientos fueron buenos)
7,  h # toma en 7
19, s # se queda en 19

Ejemplo 2

mano del jugador: ( 2-d, 10-c )
jugador toma carta
mano del jugador: ( 2-d, 10-c, 2-s )
jugador toma carta
mano del jugador: ( 2-d, 10-c, 2-s, 9-c )
el jugador se pasa de 21

datos y etiquetas generadas:

12, h # toma en 7
14, s # se queda en 14
</pre>

# Primer modelo Blackjack - data set Nivel 1

El primer modelo entrena la red neural unicamente con la mano actual. Un dataset para esta tarea fue producido con  3,959 simulaciones monte carlo generado con Blackjack.py. Los datasets están localizados en  /data_sets/blackjack.data.1 y /data_sets/blackjack.tags.1.

código para crear el dataset

<pre>
import Blackjack as bj
bj.gen_data_set( 3000, "blackjack1", 1 ) 
</pre>


código para procesar el dataset nivel level 1 

<pre>

data = open( "data_sets/blackjack2-2-out.data").readlines()
tags = open( "data_sets/blackjack2-2-out.tags").readlines()
data_clean = []
tags_clean = []

#elimina espacios en blanco
first = True
i = 0
for datum in data:

	# skip empty line first
	if first:
		first = False
		continue
	clean_datum = datum[:datum.index('\n')].strip()
	data_clean = data_clean + [ int(clean_datum) ]

first = True
for tag in tags:
	if first:
		first = False
		continue
	tag = tag[:tag.index('\n')]
	if tag is "h":
		tags_clean = tags_clean + [ 1.0 ]
	else:
		tags_clean = tags_clean + [ 0.0 ]

size = int( len(data)*(0.75) )

train_data = np.array( data_clean[1:size] )
train_tags = np.array( tags_clean[1:size] )
test_data = np.array( data_clean[size:] )
test_tags = np.array( tags_clean[size:] )

</pre>

El modelo en este ejemplo es una red neuronal densa de 2 capas. La primera capa contiene 4096 neuronas, mientras que la segunda solo 2, para 'h' tomar o 'stay' quedarse. El optimizador de Adam es utilizado, con una pérdida de 'sparse_categorical_crossentropy.' Los datos de entrenamiento y testeo fueron divididos ambos el 50% aleatoriamente. Se configuraron 10 epocas.

código del modelo

<pre>
model = keras.Sequential()
model.add( keras.layers.Dense(4096, input_dim=2) )
model.add( keras.layers.Dense(2, activation=tf.nn.softmax) )
model.compile(optimizer='adam',
	loss='sparse_categorical_crossentropy',
	metrics=['accuracy'])
model.fit(train_data, train_tags, epochs=10)
test_loss, test_acc = model.evaluate(test_data, test_tags)
print('Test accuracy:', test_acc)
</pre>

El modelo imprime su presición, se usan aletoriamente el 25% de los datos como casos de prueba. El modelo se guarda de la siguiente manera:

<pre>
# save model
# taken from https://machinelearningmastery.com/save-load-keras-deep-learning-models/
model_json = model.to_json()
with open( "models/blackjackmodel.1.json", "w") as json_file:
	json_file.write(model_json)
# serialize weights to HDF5
model.save_weights("models/blackjackmodel.1.h5")
print( "Model saved" )
</pre>

Para encontrar la heuristica, manos con valores entre 2 y 21 fueron testeados en el clasificador.Para hacer esto se necesita deserializar el modelo en un archivo:

<pre>
# open serialized model
# taken from https://machinelearningmastery.com/save-load-keras-deep-learning-models/
json_file = open('models/blackjackmodel.1.json', 'r')
loaded_model_json = json_file.read()
json_file.close()
model = keras.models.model_from_json( loaded_model_json, custom_objects={"GlorotUniform": tf.keras.initializers.glorot_uniform} )
model.load_weights( "models/blackjackmodel.1.h5" )
print( "Model loaded from disk" )
</pre>

Con este código se imprimen los casos de prueba:

<pre>
print( "testing model" )

for i in range(21):
	prediction = model.predict( np.array([ [i,10] ]) )
	if prediction[0][0] > prediction[0][1]:
		print( str(i) + " stay" )
	else:
		print( str(i) + " hit" )
</pre>

Sálida:

<pre>
0  hit
1  hit
2  hit
3  hit
4  hit
5  hit
6  hit
7  hit
8  hit
9  hit
10 hit
11 hit
12 hit
13 hit
14 hit
15 hit
16 hit
17 stay
18 stay
19 stay
20 stay
</pre>

El modelo aprendió a tomar o quedarse con la mano actual con un valor menor a 17. Esta estrategia es usada por el dealer.

Para probar el modelo contra un gran un número de simulaciones monte carlo, para esto podemos usar la función test_model() de Blackjack.py. El comando es:

<pre>
wins, losses, ties = test_model( "blackjackmodel.1", 10000, True, 1, False )
total = wins + losses + ties
win_percentage = (wins/total)*100.0
loss_percentage = (losses/total)*100.0
tie_percentage = (ties/total)*100.0
print( "Percentage won:  " + str( win_percentage ) )

</pre>

El porcentaje de victorias para 10,000 juegos de este modelo es 52.42%.

# Segundo modelo de Blackjack - conjunto de datos nivel 2

Este modelo utilizará todas las técnicas anteriores, pero el conjunto de datos ahora incluirá la carta boca arriba del crupier.

Entonces, donde anteriormente usábamos el entero único para los datos, ahora usaremos una tupla de players_hand y dealers_hand respectivamente.

Ejemplo

[ 18, 13, s ]

El conjunto de datos para esto consta de 5323 entradas, ubicadas en data_sets/blackjack.data.2 y data_sets/blackjack.tags.2. La carga de los datos se hace igual que en el modelo 1.

código para generar este conjunto de datos

<pre>
import Blackjack as bj
bj.gen_data_set( 4000, "test", 2 ) # this was renamed later
</pre>

Código para obtener los datos del conjunto de datos

<pre>
# get the data set
data = open( "data_sets/blackjack2-2-out.data").readlines()
tags = open( "data_sets/blackjack2-2-out.tags").readlines()
data_clean = []
tags_clean = []
#strip whitespace
first = True
i = 0
for datum in data:

	# skip empty line first
	if first:
		first = False
		continue
	clean_datum = datum[:datum.index('\n')].strip()
	clean_datum = clean_datum[1:-1].split(',')
	clean_datum[0] = int( clean_datum[0] )
	clean_datum[1] = int( clean_datum[1][1:] )
	print( clean_datum )
	data_clean = data_clean + [ clean_datum ]

first = True
for tag in tags:
	if first:
		first = False
		continue
	tag = tag[:tag.index('\n')]
	if tag is "h":
		tags_clean = tags_clean + [ 1.0 ]
	else:
		tags_clean = tags_clean + [ 0.0 ]

size = int( len(data)*(0.75) )

train_data = np.array( data_clean[1:size] )
train_tags = np.array( tags_clean[1:size] )
test_data = np.array( data_clean[size:] )
test_tags = np.array( tags_clean[size:] )
</pre>

La red neuronal utilizó un esquema de capas similar al anterior, con una segunda capa de 16 neuronas. El optimizador era 'nadam' y había 100 épocas.

código para entrenar esta red

<pre>
model = keras.Sequential()
model.add( keras.layers.Dense(16, input_dim=2) )
model.add( keras.layers.Dense(2, activation=tf.nn.softmax) )
model.compile(optimizer='nadam',
	loss='sparse_categorical_crossentropy',
	metrics=['accuracy'])
model.fit(train_data, train_tags, epochs=100)
test_loss, test_acc = model.evaluate(test_data, test_tags)
print('Test accuracy:', test_acc)
</pre>


Notice the only difference between the training of model 1 and model 2 is parameters and file names.

For testing purposes I found this nifty chart for Blackjack strategy at wizardofodds.com/games/blackjack/strategy/calculator/

![Basic Blackjack Strategy](https://raw.githubusercontent.com/justinbodnar/artificial-intelligence-in-card-games/master/docs/blackjack_odds.png)

Generating s imiliar table through the neural entwork can be done via

<pre>
results = []

for i in range(0,17):
	results = results + [ "" ]
	for j in range(0,9):
		prediction = model.predict( np.array([ [i+5,j+2] ] ) )
		if prediction[0][0] > prediction[0][1]:
			results[i] = results[i] + "s"
		else:
			results[i] = results[i] + "h"
print( "  ", end="" )
for x in range( len(results[0]) ):
	print( " " + str( (x+4)%10 ), end="" )
print( )
for i in range( len(results) ):
	print( i+5, end="" )
	if i+5 < 10:
		print( "  ", end="" )
	else:
		print( " ", end="" )
	for j in range( len(results[i] ) ):
		print( results[i][j], end=" " )
	print( )
</pre>

This produces the following chart.

![Blackjack classifier results](https://raw.githubusercontent.com/justinbodnar/artificial-intelligence-in-card-games/master/docs/blackjack_odds_mine.png)

There is a clear pattern on both. This confirms the neural network has begun to learn the strategy of Blackjack.

To test the model against a large number of monte carlo simulations, we can use the test_model() function in Blackjack.py. The command is

<pre>
wins, losses, ties = test_model( "blackjackmodel.2", 10000, True, 2, False )
total = wins + losses + ties
win_percentage = (wins/total)*100.0
loss_percentage = (losses/total)*100.0
tie_percentage = (ties/total)*100.0
print( "Percentage won:  " + str( win_percentage ) )
print( "Percentage lost: " + str( loss_percentage ) )
print( "Percentage tied: " + str( tie_percentage ) )
</pre>

The win percentage for 10,000 games for this model is 41.49%. Interestingly, this is less accurate than the model that used less information about the game. This implies the data set is incorrect, corrupt, etc. This will be revisited in future revisions.
# Tercer modelo de Blackjack - nivel de conjunto de datos 3

Este modelo apretó los mismos datos que los modelos anteriores, pero ahora también contendrá un registro de todas las cartas vistas hasta el momento. La simulación implica que el crupier utiliza una sola baraja hasta que se le acaban las cartas, y entonces las vuelve a barajar.

El conjunto de datos utilizados contiene aproximadamente 16.979 puntos de datos y puede encontrarse en /data_sets/blackjack.data.3 y /data_sets/blackjack.tags.3.

código para generar el conjunto de datos:

<antes>
importar Blackjack como blackjack
blackjack.gen_data_set( 12000, "prueba", 3 )
</pre>

codigo para generar el nivel 3 del dataset

<antes>
# obtener el conjunto de datos
datos = abrir( "conjuntos_de_datos/blackjack.datos.1").readlines()
etiquetas = abrir( "conjuntos_de_datos/blackjack.tags.1").readlines()
datos_limpiar = []
etiquetas_limpias = []
#quitar espacios en blanco
primero = Verdadero
para dato en datos:
# omitir la línea vacía primero
si primero:
primero = falso
Seguir
clean_datum = dato[1:dato.index('\n')-1].strip().split(', ')
dato_limpio[0] = int( dato_limpio[0] )
dato_limpio[1] = int( dato_limpio[1] )
imprimir (dato_limpio)
data_clean = data_clean + [clean_datum]

primero = Verdadero
para etiqueta en etiquetas:
si primero:
primero = falso
Seguir
etiqueta = etiqueta[:etiqueta.index('\n')]
si la etiqueta es "h":
etiquetas_limpias = etiquetas_limpias + [ 1.0 ]
más:
etiquetas_limpias = etiquetas_limpias + [ 0.0 ]

tamaño = int( len(datos)*(0.75) )

train_data = np.array( data_clean[1:size] )
train_tags = np.array( tags_clean[1:tamaño] )
test_data = np.array(data_clean[tamaño:])
test_tags = np.array(tags_clean[tamaño:])
</pre>

El tercer modelo tiene dos capas ocultas de 64 y 128 neuronas respectivamente. El optimizador es Adam y tiene 50 épocas. El modelo serializado se puede encontrar en /models/blackjackmodel.3.json y /models/blackjackmodel.3.h5.

Codigo usado para entrenar la red neuronal:

<antes>
modelo = keras.secuencial()
modelo.add(keras.layers.Dense(54, input_dim=54))
modelo.add(keras.layers.Dense(64, input_dim=26))
modelo.add(keras.layers.Dense(128, input_dim=13))
modelo.add(keras.layers.Dense(2, activación=tf.nn.softmax) )

modelo.compilar(optimizador='adam',
        loss='parse_categorical_crossentropy',
        métricas=['precisión'])

model.fit(train_data, train_tags, epochs=50)

test_loss, test_acc = model.evaluate(test_data, test_tags)

print('Exactitud de la prueba:', test_acc)
</pre>

El modelo tiene una presición de 0.73, similar a los otros tres modelos.

Para probar el modelo contra un gran número de simulaciones monte carlo, podemos utilizar la función test_model() en Blackjack.py. El comando es

<antes>
victorias, derrotas, empates = test_model ("modelo de blackjack.3", 10000, Verdadero, 3, Falso)
total = victorias + derrotas + empates
win_percentage = (ganancias/total)*100.0
loss_percentage = (pérdidas/total)*100.0
empate_porcentaje = (empates/total)*100.0
print( "Porcentaje ganado: " + str(ganancia_porcentaje) )
print("Porcentaje perdido: " + str(porcentaje_perdida))
print( "Porcentaje empatado: " + str( porcentaje_empate ) )
</pre>

El porcentaje de victorias en 10.000 juegos para este modelo es del 41,33%. Esto es menos preciso que todos los demás modelos que utilizaron menos información sobre el juego. Esto implica que el conjunto de datos es incorrecto, corrupto, etc. Esto será revisado en revisiones futuras.
# Problemas relacionados con los conjuntos de datos

La tabla completa para las pruebas sale a:

nivel 1: 51,82%

nivel 2: 41,49%

nivel 3: 41,33%

solo acertando: 3,42%

solo estancia: 41,99%

movimientos aleatorios: 30,67%

El nivel 1 es el modelo más preciso y el modelo se deteriora a medida que obtenemos más información sobre el juego. Los conjuntos de datos claramente necesitan algo de trabajo. El mejor clasificador es solo un 9,17 % mejor que quedarse en todas las manos. Los niveles 2 y 3 son ligeramente más bajos que quedarse solo, aunque la diferencia es insignificante.

Problemas potenciales:

-repetir manos

-diferentes conclusiones en el mismo escenario dan como resultado etiquetas opuestas

En un intento por corregir este problema, se escribió un script en /data_sets/preproc.py para preprocesar conjuntos de datos antes de entrenar modelos. El script cumple 2 funciones:

- si se encuentran puntos de datos opuestos, se elimina el que tiene el menor número de instancias
- se eliminan los duplicados

El script no se puede ejecutar sin modificarlo. TODO incluye funcionalizar este script e incluirlo en Blackjack.py.