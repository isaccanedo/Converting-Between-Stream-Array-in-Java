## Convertendo entre Stream e Array em Java

# 1. Introdução
É comum precisar converter várias estruturas de dados dinâmicas em matrizes.

Neste tutorial, vamos demonstrar como converter um Stream em um array e vice-versa em Java.

# 2. Convertendo um Stream em um Array

### 2.1. Referência do Método
A melhor maneira de con`verter um Stream em uma matriz é usar o método toArray () do Stream:

``` 
public String[] usingMethodReference(Stream<String> stringStream) {
    return stringStream.toArray(String[]::new);
}
```

Agora, podemos testar facilmente se a conversão foi bem-sucedida:

```
Stream<String> stringStream = Stream.of("isaccanedo", "convert", "to", "string", "array");
assertArrayEquals(new String[] { "isaccanedo", "convert", "to", "string", "array" },
    usingMethodReference(stringStream));
```

### 2.2. Expressão Lambda
Outro equivalente é passar uma expressão lambda para o método toArray():

```
public static String[] usingLambda(Stream<String> stringStream) {
    return stringStream.toArray(size -> new String[size]);
}
```

Isso nos daria o mesmo resultado do uso da referência de método.

### 2.3. Classe Personalizada
Ou podemos fazer tudo e criar uma classe completa.

Como podemos ver na documentação do Stream, ele recebe um IntFunction como argumento. Ele pega o tamanho do array como entrada e retorna um array desse tamanho.

Claro, IntFunction é uma interface para que possamos implementá-lo:

```
class MyArrayFunction implements IntFunction<String[]> {
    @Override
    public String[] apply(int size) {
        return new String[size];
    }
};
```

Podemos então construir e usar normalmente:

```
public String[] usingCustomClass(Stream<String> stringStream) {
    return stringStream.toArray(new MyArrayFunction());
}
```

Conseqüentemente, podemos fazer a mesma afirmação anterior.

### 2.4. Matrizes primitivas
Nas seções anteriores, exploramos como converter um String Stream em um array String. Na verdade, podemos realizar a conversão dessa forma para qualquer objeto e seria muito semelhante aos exemplos de String acima.

No entanto, é um pouco diferente para os primitivos. Se tivermos um fluxo de inteiros que queremos converter em int[], por exemplo, primeiro precisamos chamar o método mapToInt():

```
public int[] intStreamToPrimitiveIntArray(Stream<Integer> integerStream) {
    return integerStream.mapToInt(i -> i).toArray();
}
```

Também há métodos mapToLong() e mapToDouble() à nossa disposição. Além disso, observe que não passamos nenhum argumento para toArray() desta vez.

Finalmente, vamos fazer a asserção de igualdade e confirmar que obtivemos nosso array int corretamente:

```
Stream<Integer> integerStream = IntStream.rangeClosed(1, 7).boxed();
assertArrayEquals(new int[]{1, 2, 3, 4, 5, 6, 7}, intStreamToPrimitiveIntArray(integerStream));
```

E se precisarmos fazer o oposto, no entanto? Vamos dar uma olhada.

# 3. Convertendo uma matriz em um fluxo
Podemos, é claro, ir por outro caminho também. E Java tem alguns métodos dedicados para isso.

### 3.1. Matriz de objetos
Podemos converter a matriz em um Stream usando os métodos Arrays.stream() ou Stream.of():

```
public Stream<String> stringArrayToStreamUsingArraysStream(String[] stringArray) {
    return Arrays.stream(stringArray);
}

public Stream<String> stringArrayToStreamUsingStreamOf(String[] stringArray) {
    return Stream.of(stringArray);
}
```

Devemos notar que, em ambos os casos, nosso Stream é ao mesmo tempo que nosso array.

### 3.2. Matriz de primitivos
Da mesma forma, podemos converter uma matriz de primitivas:

```
public IntStream primitiveIntArrayToStreamUsingArraysStream(int[] intArray) {
    return Arrays.stream(intArray);
}

public Stream<int[]> primitiveIntArrayToStreamUsingStreamOf(int[] intArray) {
    return Stream.of(intArray);
}
```

Mas, em contraste com a conversão de matrizes de objetos, há uma diferença importante. Ao converter o array de primitivos, Arrays.stream () retorna IntStream, enquanto Stream.of () retorna Stream <int []>.

### 3.3. Arrays.stream vs. Stream.of
Para entender as diferenças mencionadas nas seções anteriores, daremos uma olhada na implementação dos métodos correspondentes.

Vamos primeiro dar uma olhada na implementação Java desses dois métodos:

```
public <T> Stream<T> stream(T[] array) {
    return stream(array, 0, array.length);
}

public <T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}
```

Podemos ver que Stream.of() está, na verdade, chamando Arrays.stream() internamente e essa é obviamente a razão pela qual obtemos os mesmos resultados.

Agora, vamos verificar os métodos no caso em que queremos converter uma matriz de primitivas:

```
public IntStream stream(int[] array) {
    return stream(array, 0, array.length);
}

public <T> Stream<T> of(T t) {
    return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
```

Desta vez, Stream.of() não está chamando Arrays.stream().

# 4. Conclusão
Neste artigo, vimos como podemos converter Streams em arrays em Java e vice-versa. Também explicamos por que obtemos resultados diferentes ao converter uma matriz de objetos e quando usamos uma matriz de primitivas.