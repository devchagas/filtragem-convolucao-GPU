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

    

