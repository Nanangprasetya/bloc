# Conceptos básicos

?> Asegúrese de leer detenidamente y comprender las siguientes secciones antes de trabajar con [bloc](https://pub.dev/packages/bloc).

Hay varios conceptos básicos que son críticos para entender cómo usar Bloc.

En las próximas secciones, vamos a discutir cada uno de ellos en detalle, así como a analizar cómo se aplicarían a una aplicación del mundo real: una aplicación de contador.

## Events

> Los eventos o events son la entrada a un bloc. Se agregan comúnmente en respuesta a las interacciones del usuario, como presionar botones o eventos del ciclo de vida, como cargas de página.

Al diseñar una aplicación, debemos dar un paso atrás y definir cómo los usuarios interactuarán con ella. En el contexto de nuestra aplicación de contador, tendremos dos botones para aumentar y disminuir nuestro contador.

Cuando un usuario toca uno de estos botones, algo debe suceder para notificar a los "cerebros" de nuestra aplicación para que pueda responder a la entrada del usuario; aquí es donde los eventos entran en juego.

Necesitamos poder notificar a los "cerebros" de nuestra aplicación tanto de un incremento como de decremento, por lo que debemos definir estos eventos.

```dart
enum CounterEvent { increment, decrement }
```

En este caso, podemos representar los eventos usando un `enum`, pero para casos más complejos puede ser necesario usar una `class`, especialmente si es necesario pasar información al bloc.

En este punto, hemos definido nuestro primer evento! Tenga en cuenta que hasta ahora no hemos usado Bloc de ninguna manera y que no ocurre ninguna magia; es simplemente código Dart.

## States

> Los estados o states son la salida de un Bloc y representan una parte del estado de su aplicación. Los componentes de la interfaz de usuario pueden ser notificados de los estados y volver a dibujar partes de ellos mismos en función del estado actual.

Hasta ahora, hemos definido los dos eventos a los que nuestra aplicación responderá: `CounterEvent.increment` y `CounterEvent.decrement`.

Ahora necesitamos definir cómo representar el estado de nuestra aplicación.

Como estamos construyendo un contador, nuestro estado es muy simple: es solo un número entero que representa el valor actual del contador.

Veremos ejemplos más complejos de estado más adelante, pero en este caso un tipo primitivo es perfectamente adecuado como representación de estado.

## Transitions

> El cambio de un estado a otro se llama Transición o Transitions. Una transición consta del estado actual, el evento y el siguiente estado.

A medida que un usuario interactúa con nuestra aplicación de contador, activará eventos de `Increment` y `Decrement` que actualizarán el estado del contador. Todos estos cambios de estado se pueden describir como una serie de Transiciones.

Por ejemplo, si un usuario abre nuestra aplicación y toca el botón de incremento una vez, veremos la siguiente `Transition`.

```json
{
  "currentState": 0,
  "event": "CounterEvent.increment",
  "nextState": 1
}
```

Debido a que cada cambio de estado se registra, podemos instrumentar muy fácilmente nuestras aplicaciones y rastrear todas las interacciones del usuario y los cambios de estado en un solo lugar. Además, esto hace posible la depuración de viajes en el tiempo.

## Streams

?> Consulte la documentación oficial de [Dart](https://dart.dev/tutorials/language/streams) para obtener más información sobre Streams.

> Un stream es una secuencia de datos asincrónicos.

Bloc está construido sobre [RxDart](https://pub.dev/packages/rxdart); sin embargo, abstrae todos los detalles de implementación específicos de `RxDart`.

Para usar Bloc, es crítico tener una comprensión sólida de `Streams` y cómo funcionan.

> Si no está familiarizado con `Streams`, piense en una tubería con agua que fluye a través de ella. La tubería es el `Stream` y el agua son los datos asincrónicos.

Podemos crear un `Stream` en Dart escribiendo una función `async*`.

```dart
Stream<int> countStream(int max) async* {
    for (int i = 0; i < max; i++) {
        yield i;
    }
}
```

Al marcar una función como `async*`, podemos usar la palabra reservada `yield` y devolver un `Stream` de datos. En el ejemplo anterior, estamos devolviendo un `Stream` de enteros hasta el parámetro entero `max`.

Cada vez que llamamos a `yield` en una función `async*` estamos empujando ese dato a través del `Stream`.

Podemos consumir el `Stream` anterior de varias maneras. Si quisiéramos escribir una función para devolver la suma de un `Stream` de enteros, podría ser algo así:

```dart
Future<int> sumStream(Stream<int> stream) async {
    int sum = 0;
    await for (int value in stream) {
        sum += value;
    }
    return sum;
}
```

Al marcar la función anterior como `async` podemos usar la palabra reservada `await` y devolver un `Future` de enteros. En este ejemplo, estamos esperando cada valor en el stream y devolviendo la suma de todos los enteros en el stream.

Podemos poner todo junto de esta manera:

```dart
void main() async {
    /// Inicializa una secuencia de enteros 0-9
    Stream<int> stream = countStream(10);
    /// Calcula la suma de la secuencia de enteros
    int sum = await sumStream(stream);
    /// Imprime la suma
    print(sum); // 45
}
```

## Blocs

> Un Bloc(Business Logic Component) es un componente que convierte un `Stream` de `Events` entrantes en un `Stream` de `States` salientes. Piense en un bloc como el "cerebro" descrito anteriormente.

> Cada Bloc debe extender la clase de `Bloc` base que es parte del paquete del bloc central.

```dart
import 'package:bloc/bloc.dart';

class CounterBloc extends Bloc<CounterEvent, int> {

}
```

En el fragmento de código anterior, estamos declarando nuestro `CounterBloc` como un Bloc que convierte `CounterEvents` en `ints`.

> Cada bloc debe definir un estado inicial que es el estado antes de que se haya recibido cualquier evento.

En este caso, queremos que nuestro contador comience en `0`.

```dart
@override
int get initialState => 0;
```

> Cada bloc debe implementar una función llamada `mapEventToState`. La función toma el `event` entrante como argumento y debe devolver un `Stream` de nuevos `states` que es consumida por la capa de presentación. Podemos acceder al estado del bloc actual en cualquier momento utilizando la propiedad `state`.

```dart
@override
Stream<int> mapEventToState(CounterEvent event) async* {
    switch (event) {
      case CounterEvent.decrement:
        yield state - 1;
        break;
      case CounterEvent.increment:
        yield state + 1;
        break;
    }
}
```

En este punto, tenemos un `CounterBloc` completamente funcional.

```dart
import 'package:bloc/bloc.dart';

enum CounterEvent { increment, decrement }

class CounterBloc extends Bloc<CounterEvent, int> {
  @override
  int get initialState => 0;

  @override
  Stream<int> mapEventToState(CounterEvent event) async* {
    switch (event) {
      case CounterEvent.decrement:
        yield state - 1;
        break;
      case CounterEvent.increment:
        yield state + 1;
        break;
    }
  }
}
```

!> Los blocs ignorarán los estados duplicados. Si un bloc arroja `State nextState` donde `state == nextState`, entonces no ocurrirá ninguna transición y no se realizará ningún cambio en el `Stream <State>`.

En este punto, probablemente se está preguntando _"¿Cómo notifico a un bloc de un evento?"_.

> Cada bloc tiene un método `add`. `Add` toma un `event` y activa `mapEventToState`. Se puede llamar a `Add` desde la capa de presentación o desde dentro del Bloc y notifica al Bloc de un nuevo `evento`.

Podemos crear una aplicación simple que cuente de 0 a 3.

```dart
void main() {
    CounterBloc bloc = CounterBloc();

    for (int i = 0; i < 3; i++) {
        bloc.add(CounterEvent.increment);
    }
}
```

!> De manera predeterminada, los eventos siempre se procesarán en el orden en que se agregaron y los eventos recién agregados se colocan en cola. Un evento se considera completamente procesado una vez que `mapEventToState` ha terminado de ejecutarse.

Las `Transitions` en el fragmento de código anterior serían

```json
{
    "currentState": 0,
    "event": "CounterEvent.increment",
    "nextState": 1
}
{
    "currentState": 1,
    "event": "CounterEvent.increment",
    "nextState": 2
}
{
    "currentState": 2,
    "event": "CounterEvent.increment",
    "nextState": 3
}
```

Desafortunadamente, en el estado actual no podremos ver ninguna de estas transiciones a menos que anulemos `onTransition`.

> `onTransition` es un método que se puede anular para manejar cada `Transition` del Bloc local. `onTransition` se llama justo antes de que se actualice el `estado` de un bloc.

?> **Consejo**: `onTransition` es un gran lugar para agregar registros / análisis específicos de bloc.

```dart
@override
void onTransition(Transition<CounterEvent, int> transition) {
    print(transition);
}
```

Ahora que hemos anulado `onTransition` podemos hacer lo que queramos cuando ocurra una `Transition`.

Al igual que podemos manejar `Transition` a nivel de bloc, también podemos manejar `Exceptions`.

> `onError` es un método que se puede anular para manejar cada `Excepción` del Bloc local. Por defecto, todas las excepciones serán ignoradas y la funcionalidad `Bloc` no se verá afectada.

?> **Nota**: El argumento stacktrace puede ser `null` si la secuencia de estado recibió un error sin un `StackTrace`.

?> **Consejo**: `onError` es un gran lugar para agregar el manejo de errores específicos del bloc.

```dart
@override
void onError(Object error, StackTrace stackTrace) {
  print('$error, $stackTrace');
}
```

Ahora que hemos anulado `onError` podemos hacer lo que queramos siempre que se produzca un `Exception`.

## BlocDelegate

Una ventaja adicional de usar Bloc es que podemos tener acceso a todas las `Transitions` en un solo lugar. Aunque en esta aplicación solo tenemos un Bloc, es bastante común en aplicaciones más grandes tener muchos Blocs que administran diferentes partes del estado de la aplicación.

Si queremos poder hacer algo en respuesta a todas las `Transitions`, simplemente podemos crear nuestro propio `BlocDelegate`.

```dart
class SimpleBlocDelegate extends BlocDelegate {
  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }
}
```

?> **Nota**: Todo lo que necesitamos hacer es extender `BlocDelegate` y anular el método `onTransition`.

Para decirle a Bloc que use nuestro `SimpleBlocDelegate`, solo necesitamos ajustar nuestra función `main`.

```dart
void main() {
  BlocSupervisor.delegate = SimpleBlocDelegate();
  CounterBloc bloc = CounterBloc();

  for (int i = 0; i < 3; i++) {
    bloc.add(CounterEvent.increment);
  }
}
```

Si queremos poder hacer algo en respuesta a todos los `Events` agregados, también podemos anular el método `onEvent` en nuestro `SimpleBlocDelegate`.

```dart
class SimpleBlocDelegate extends BlocDelegate {
  @override
  void onEvent(Bloc bloc, Object event) {
    super.onEvent(bloc, event);
    print(event);
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }
}
```

Si queremos poder hacer algo en respuesta a todas las `Exceptions` lanzadas en un bloc, también podemos anular el método `onError` en nuestro `SimpleBlocDelegate`.

```dart
class SimpleBlocDelegate extends BlocDelegate {
  @override
  void onEvent(Bloc bloc, Object event) {
    super.onEvent(bloc, event);
    print(event);
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }

  @override
  void onError(Bloc bloc, Object error, StackTrace stacktrace) {
    super.onError(bloc, error, stacktrace);
    print('$error, $stacktrace');
  }
}
```

?> **Nota**: `BlocSupervisor` es un singleton que supervisa todos los Blocs y delega responsabilidades al `BlocDelegate`.
