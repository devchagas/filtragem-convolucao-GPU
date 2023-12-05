# Filtragem de Convolução utilizando GPU e Vetorização

### Projeto desenvolvido para a matéria de Introdução a Programação Paralela

### ALGORITMOS DE PROCESSAMENTO DE IMAGENS


##### Filtragem de Convolução

A filtragem de convolução é um processo comum em processamento de imagem, onde 
uma máscara (também chamada de kernel) é aplicada a uma imagem para realçar ou 
atenuar características específicas da imagem, por exemplo melhorar a qualidade da 
imagem, detecção de borda, etc.

A máscara também chamada de kernel é uma matriz de números que é deslizado 
(convoluída) sobre a matriz de pixels da imagem. O tamanho dessa máscara e os 
seus valores depende do efeito desejado. Por exemplo uma máscara conhecida como 
filtro de média simples pode ser usada para suavizar a imagem, enquanto um filtro de 
detecção de bordas como o filtro Sobel é utilizado para realçar as bordas da imagem.
As máscaras de convolução geralmente são filtros pequenos, ou seja, matriz de 
dimensões pequenas, escolhidas justamente para que a computação envolvida seja 
eficiente e garanta que o filtro se concentre em uma pequena região da imagem por 
vez. Alguns exemplos de filtros são o já citado acima, filtro da média simples usado 
para suavizar a imagem reduzindo o ruído, a matriz é preenchida com valores iguais 
que representam pesos uniformes para a média, por exemplo um filtro da média 3x3:

filtroMedia = 
| 1/9 | 1/9 | 1/9 |
|---|---|---|
| 1/9 | 1/9 | 1/9 |
| 1/9 | 1/9 | 1/9 |

Outro filtro muito utilizado e também já citado é o Filtro Sobel usado comumente para 
realçar e detectar bordas, elas consistem em dois tipos de matrizes. Uma para detecção 
de bordas horizontais e outra para bordas verticais, um exemplo do filtro Sobel 3x3:

sobelHorizontal = 
| -1 | -2 | -1 |
|---|---|---|
| 0 | 0 | 0 |
| 1 | 2 | 1 |

sobelVertical =
| -1 | 0 | 1 |
|---|---|---|
| 2 | 0 | 2 |
| -1 | 0 | 1 |

Com o filtro escolhido devemos então aplica-lo na imagem, para isso fazemos então o 
processo de convolução, isto é, deslizar a pequena matriz do filtro sobre a matriz de 
pixels da imagem colocando a matriz do filtro sobre uma parte da matriz de pixels da 
imagem e então multiplica cada valor da matriz pelo pixel correspondente. Por fim, 
soma os resultados multiplicados e retorna o valor da soma como o novo pixel central 
daquela região da matriz da imagem, então esse processo é realizado em toda a 
imagem.

Algoritmos de processamento de imagens como esses podem se tornar 
computacionalmente caros e pesados tendo em vista que o custo de computação 
envolvida em uma convolução de uma matriz N x N, por exemplo, é de N2 x N2
. 
Portanto uma solução proposta é a de aplicar vetorização para que então a filtragem 
seja paralelizada de forma que o filtro seja aplicado a imagem inteira de uma só vez.

Para paralelizar o algoritmo da filtragem de convolução nós utilizamos da mesma 
estratégia utilizada para paralelizar a multiplicação de matrizes, pois a filtragem de 
convolução não deixa de ser uma multiplicação de matrizes.

```
pragma omp target teams distribute parallel for simd 
private (j, x, y, soma) schedule(guided) 
map(to:imagem[0:N*N], mascara[0:M*M]) map(tofrom:resultado[0:N*N])
```

Usamos primeiramente a diretiva target para indicar que o bloco estruturado a seguir 
deveria ser executado por uma nova thread no dispositivo alvo (target device) que no 
caso fizemos uso da GPU para vetorização ainda melhor da filtragem da convolução.
Depois usamos a diretiva teams para criar uma liga de times onde cada time da liga tem 
uma thread inicial associada a ela, com o uso dessa diretiva podemos permitir o uso de 
todo o paralelismo disponível na GPU. Após criar essa liga de times ainda sim 
precisamos dividir as tarefas entre elas para que não executem o mesmo código, para 
isso usamos a diretiva distribute.

Agora cada time tem uma thread e cada thread realiza uma tarefa diferente, entretanto 
ainda não há paralelização pois justamente apenas uma thread está associada a cada 
time, precisamos criar um novo time de threads onde cada thread inicial da diretiva 
teams será responsável por um novo time de threads, tudo isso fazemos ao utilizar a 
diretiva parallel.

Agora basta que usemos a diretiva for para que o laço da convolução seja quebrado 
entre as threads do time, e por fim usamos a diretiva simd para que cada thread possa 
usar vetorização.

A diretiva private usamos para que as variáveis que deveriam ser individuais de cada 
time de threads executando tarefas diferentes se tornem individuais para cada tarefa 
paralelizada. A diretiva schedule foi usado para otimizar ainda mais a paralelização, foi 
observado que com a clausula guided conseguimos obter alguns poucos segundos a 
mais de speedup. E a diretiva map to foi usada para fazer o onloading das matrizes a 
serem multiplicadas na GPU e o map tofrom para realizar o offloading da matriz 
resultado da GPU.

Com o uso de todas essas diretivas podemos observar um speedup associado no 
processo de filtragem de convolução de imagens fazendo com que a imagem seja 
filtrada de uma vez só já que a tarefa de aplicar o filtro é agora paralelizada e realizada 
em simultâneo. Para confirmar a melhora no processamento de imagens fizemos 
algumas medições de tempo e tiramos a média para então tabelar e plotar um gráfico 
de speedup.

Vale ressaltar, entretanto, que no nosso código da atividade não abrimos diretamente 
uma imagem e de fato aplicamos o filtro para obter no final a imagem processada. Como 
a ideia da atividade é trabalhar mais justamente a vetorização e paralelização utilizamos 
uma matriz de valores randômicos e de dimensões iguais (N x N) que não representa
resoluções reais de imagens como Full HD (1920 x 1080), 4K (3840 x 2160), 8K (7680 
x 4320), 10K (10240 x 4320), entretanto usamos o maior valor da resolução, no caso 
sempre a largura para os testes. Para o filtro, usamos por padrão sempre um filtro Sobel 
Horizontal 5x5 para poder utilizar de um filtro maior e com mais computações afim de 
observar ainda melhor a melhoria da vetorização.

![tabela-medicao-tempo.png](/assets/tabela-medicao-tempo.png "Tabela de Medições de tempo")

![grafico-speedup.png](/assets/grafico-speedup.png "Gráfico de Speedup em Diferentes Cenários")

Podemos observar que matrizes da imagem de tamanho menores (100, 1000, 1500) 
não apresentam speedup, possivelmente devido ao custo associado da criação e 
distribuição dos times e threads, assim o custo da computação serial é mais rápido do 
que parar para criar as threads e times para então paralelizar. Entretanto a partir de um 
tamanho da imagem, na tabela 1920, podemos observar que já obtemos um speedup
que somente aumenta conforme aumentamos ainda mais o tamanho da imagem.

Podemos concluir então que a vetorização para o processamento de imagens utilizando 
filtragem de convolução se torna interessante para imagens de alta resolução.
