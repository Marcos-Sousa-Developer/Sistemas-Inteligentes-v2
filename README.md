---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.14.4
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---


# Sistemas Inteligentes 2021/2022

## Mini-projeto 2: Quadrados Latinos



## Grupo: 02

### Elementos do Grupo

Número: 55852         Nome: Marcos Leitão   
Número: 56909         Nome: Miguel Fernandes       


## Representação de variáveis, domínios, vizinhos e restrições

* **`variables`** : Lista de variáveis (tuplos).

* **`domains`** : Dicionário com elementos do tipo `{var:[(x1,y1), ...]}`.

* **`neighbors`**: Dicionário com elementos do tipo `{var:[(x1,y1),...]}` em que cada variável var (chave) tem como valor uma lista das variáveis com as quais tem restrições (vizinhos no grafo de restrições), que define o grafo de restrições.

* **`constraints`** : Função do tipo `f(A, a, B, b)` que devolve `True` se os vizinhos `A` e `B` satisfazem a restrição quando têm valores `A=a` e `B=b`. 


## Formulação do problema quadrado latino simples 
Consideremos que temos n*n variáveis, e que cada variável tem valores possíveis de {1, .., n}, em que as restrições obrigam a que todas as variavés na mesma linha e coluna tenham valores diferentes.

```python
from csp import *
from collections import defaultdict


def parse_neighbors(neighbors, variables=[]): #Novo parse_neighbors
    """Convert a dict of the form {'X': [Y,Z], 'Y': ['Z']} into a dict mapping
    regions to neighbors.  The syntax is a region name followed by the key
    followed by zero or more region names, followed by the values, repeated for
    each region name.  If you say 'X': ['Y'] you don't need 'Y': ['X'].
    >>> parse_neighbors({'X': [Y,Z], 'Y': ['Z']}) == {'Y': ['X', 'Z'], 'X': ['Y', 'Z'], 'Z': ['X', 'Y']}
    True
    """

    dic = defaultdict(list)
    specs = [[key,value] for key,value in neighbors.items()]

    for (A, Aneighbors) in specs:
        for B in Aneighbors:
            dic[A].append(B)
            dic[B].append(A)
            
    return dic

def quadrado_latino(n=4,instrucoes={}):
    """
    Retorna um objecto da classe CSP inicializado com as variáveis, os domínios,
    os vizinhos e restrições do problema quadrado_latino, different_values_constraint.
    """
    
    #definir as variáveis e dominios 
    variaveis, dominios = [], {}
    
    for l in range(1,n+1):
        for c in range(1,n+1):
            variaveis.append((l,c))
            dominios[(l,c)] = [i for i in range(1,n+1)] #Devolve um dicionario com os domínios das variáveis do problema. 
            
    #inserir as instruções
    for key,value in instrucoes.items(): 
        dominios[key] = [value]

    #definir vizinhos
    dic = {}
    for v in variaveis:
        if v != (n,n):
            dic[v] = [ x for x in variaveis if (x[0] > v[0] and x[1] == v[1]) or (x[0] == v[0] and x[1] > v[1]) ]
    
    # parse_neighbors trata da simetria
    vizinhos = parse_neighbors(dic)
    
    return CSP(variaveis, dominios, vizinhos, different_values_constraint)
```

## Criação do problema do quadrado latino simples

Vamos agora criar um objecto CSP usando a função quadrado_latino que implementámos em cima:

**Nota:** A função quadrado_latino, pode ou não receber argumentos, por omissão, assume-se o **<span style="color:green"> n=4 </span>** e sem instruções **<span style="color:green"> instrucoes = {} </span>**, caso queiramos inserir numeros em certas posições de inicio.

```python
p = quadrado_latino()
```

E verificar os valores das variáveis, dos domínios e dos vizinhos (grafo de restrições):

```python
print("Variáveis = ", p.variables)
```

```python
print("Domínios = ", p.domains)
```

```python
print("Restricoes = ", p.neighbors)
```

A função restricoes foi bem implementada, testando algumas afectações a pares de variáveis, confirmando que a função retorna True, se os valores não têm conflito com as restrições entre o par de variáveis passados como argumento para o par de valores concreto, e False, caso contrário.

```python
# Restrição (x,y) != (xn,yn)
print(p.constraints((1, 1), 2, (1, 2), 3)) #True
print(p.constraints((1, 3), 3, (1, 2), 2)) #True
print(p.constraints((4, 4), 2, (2, 4), 2)) #True
print(p.constraints((3, 2), 3, (3, 4), 2)) #True
```

## Solução do quadrado_latino

Vamos agora usar o algoritmo **BACKTRACKING-SEARCH** para resolver o problema. 


### Backtracking sem inferencia


Vamos invocar a profundidade com retrocesso com os argumentos por omissão.

A procura com retrocesso falha quando uma variável com um determinado valor viola as restrições envolvendo essa variável com as que já têm valores afetados

```python
r = backtracking_search(p)
print('Assignment p = ',r) 
```

Vamos colocar o verbose=True para seguirmos com detalhe todo o processo de resolução do CSP.

```python
p = quadrado_latino()
r = backtracking_search(p,verbose = True)
print('Assignment p = ',r) 
```

### Inferência Forward Checking

Com a inferência do tipo *forward checking*, os domínios das restantes variáveis, fora da afetação parcial, por afetar ainda, daí o nome "para a frente" (*forward*), vão ser filtradas dos valores que entram em conflito, pois tem deser diferentes.



```python
p = quadrado_latino()
r = backtracking_search(p,inference = forward_checking)
print('Assignment = ',r)
```

Agora, a versão com verbose = True

```python
p = quadrado_latino()
r = backtracking_search(p,inference = forward_checking, verbose = True)
print('Assignment = ',r)
```

### Procura com inferência em que se mantém a consistência (mac), ou AC3

**MAC** (Mantain Arc Consistenty, usando o AC3): Utilizando o **AC3**

O algoritmo AC3 verifica a consistência dos arcos e vai para além disso forçando essa consistência alterando os domínios das variáveis. Se por acaso uma das variáveis ficar com o seu domínio vazio a função falha retornando `False`.

A consistência de um arco X->Y é forçada removendo do domínio de X todos os valores para os quais não há qualquer valor do domínio de Y que satisfaça a restrição associada ao arco.


```python
p = quadrado_latino()
r = backtracking_search(p,inference = mac, verbose=True)
print('Afetações r = ',r)
```

### Utilização de heurísticas no Backtracking-Search (MRV)

MINIMUM-REMAINING-VALUES (MRV) - heurística que escolhe a variável com o menor número de valores no domínio. Nesta implementação em caso de empate escolhe-se a próxima var de modo aleatório.

A ideia por detrás desta heurística é que, provavelmente, iremos antecipar as falhas escolhendo variáveis com menores domínios, diminuindo a procura. Isso irá ter implicações na ordem da escolha das variáveis para a afectação como podem comprovar.

```python
p = quadrado_latino()
r = backtracking_search(p,select_unassigned_variable = mrv, verbose=True)
print('Afetações r = ',r)
```

## Visualização do problema

A função display_quadrado_latino, recebe como parametro uma solução final = r, e instruções caso seja o caso.

```python
import math

def concatena(estado):
    
    posicoes = list(estado.keys())
    
    n = int(math.sqrt((len(posicoes))))
    
    col = "  "
    values = ""
    for l in range(1,n+1):
        col += " | " + str(l) + "c" 
        values += str(l) + "l" + " |  "
        for c in range(1,n+1):
            values += str(estado[(l,c)]) + " |  "
        values += '\n'
    
    col += " | "
    return col,values
    
    
def display_quadrado_latino(final, instrucoes = {}):
    
    print("Estado Inicial: ")
    
    dic = {}
    
    
    for var in list(final.keys()):
        dic[var] = "-"
    
    for key,value in instrucoes.items():
        dic[key] = str(value)
    
    estado_inicial = concatena(dic)
    
    print(estado_inicial[0])
    print(estado_inicial[1])
    
    
    print()
    print("Estado Final: ") 
    
    estado_final = concatena(final)

    print(estado_final[0])
    print(estado_final[1])

display_quadrado_latino(r)
```

#### Visualização do problema, com instruções a cerca das posições e n=5 

```python
instrucoes = {
    
    (1,1): 1,
    
    (5,5): 1
}

p = quadrado_latino(5,instrucoes)
r = backtracking_search(p)
display_quadrado_latino(r,instrucoes)
```

## Formulação do problema Futoshiki *5x5*

Consideremos que temos n*n variáveis, e que cada variável tem valores possíveis de {1, .., n}, em que as restrições obrigam a que todas as variavés na mesma linha e coluna tenham valores diferentes. <br>
No início do jogo alguns dígitos podem ser revelados. O quadro também pode conter algumas desigualdades entre as células do quadro; essas desigualdades devem ser respeitadas e podem ser usadas como pistas para descobrir os dígitos ocultos restantes.

```python
from csp import *
from collections import defaultdict

def parse_neighbors(neighbors, variables=[]):
    """Convert a dict of the form {'X': [Y,Z], 'Y': ['Z']} into a dict mapping
    regions to neighbors.  The syntax is a region name followed by the key
    followed by zero or more region names, followed by the values, repeated for
    each region name.  If you say 'X': ['Y'] you don't need 'Y': ['X'].
    >>> parse_neighbors({'X': [Y,Z], 'Y': ['Z']}) == {'Y': ['X', 'Z'], 'X': ['Y', 'Z'], 'Z': ['X', 'Y']}
    True
    """

    dic = defaultdict(list)
    specs = [[key,value] for key,value in neighbors.items()]

    for (A, Aneighbors) in specs:
        for B in Aneighbors:
            dic[A].append(B)
            dic[B].append(A)
            
    return dic

def diferentes(x,y):
    """x diferente de y"""
    return x != y

def menor(x,y):
    """ x menor que y"""
    
    return x < y

def maior(x,y):
    """x maior que y"""
    
    return x > y

def futoshiki(n=5, instrucoes={}, restricoes = {}):
    """
    Pode receber parametros ou não.
    Deve devolver um CSP, à semelhança dos guiões das aulas PL.
    Comente o código.
    """
    
    #definir as variáveis e dominios 
    variaveis, dominios = [], {}
    
    for l in range(1,n+1):
        for c in range(1,n+1):
            variaveis.append((l,c))
            dominios[(l,c)] = [i for i in range(1,n+1)]
    
    #definir vizinhos
    dic = {}
    for v in variaveis:
        if v != (n,n):
            dic[v] = [ x for x in variaveis if (x[0] > v[0] and x[1] == v[1]) or (x[0] == v[0] and x[1] > v[1]) ]
    
    #inserir as instruções
    for key,value in instrucoes.items():
        dominios[key] = [value]
        
    vizinhos = parse_neighbors(dic)
    
    def constraint(X, a, Y, b):

        if X in variaveis and Y in variaveis:
            if (X,Y) not in restricoes.keys():
                restricoes[(X,Y)] = diferentes 
        
        if (X,Y) in restricoes :
            return restricoes[(X,Y)](a,b)
    
    return CSP(variaveis, dominios, vizinhos, constraint)
```

## Criação do problema Futoshiki *5x5*

Vamos agora criar um objecto CSP usando a função futoshiki que implementámos em cima:

**Nota:** A função futoshiki, pode ou não receber argumentos, por omissão, assume-se o **<span style="color:green"> n=5</span>**, sem instruções **<span style="color:green"> (instrucoes = {})</span>**, caso queiramos inserir numeros em certas posições de inicio e sem restrições **<span style="color:green"> (restricoes = {})</span>**, ou seja, basta ser colunas e linhas diferentes. <br>

Desta vez, iremos inserir algumas instruções e algumas restrições.

![image.png](attachment:image.png)

```python
instrucoes = {
    (2,4) : 1,
    (2,5) : 5
}


restricoes = { ((1,1),(1,2)) : maior, ((1,2),(1,1)) : menor,
               ((1,3),(1,4)) : maior, ((1,4),(1,3)) : menor,
               ((2,2),(3,2)) : menor, ((3,2),(2,2)) : maior,
               ((3,1),(3,2)) : maior, ((3,2),(3,1)) : menor,
               ((4,2),(4,3)) : menor, ((4,3),(4,2)) : maior,
               ((4,4),(4,5)) : menor, ((4,5),(4,4)) : maior,
               ((5,4),(5,5)) : maior, ((5,5),(5,4)) : menor
              
             }

p = futoshiki(5,instrucoes,restricoes)
```

E verificar os valores das variáveis, dos domínios e dos vizinhos (grafo de restrições):

```python
print("Variáveis = ", p.variables)
```

```python
print("Domínios = ", p.domains)
```

```python
print("Restricoes = ", p.neighbors)
```

A função restricoes foi bem implementada, testando algumas afectações a pares de variáveis, confirmando que a função retorna True, se os valores não têm conflito com as restrições entre o par de variáveis passados como argumento para o par de valores concreto, e False, caso contrário.

```python
print(p.constraints((1, 1), 2, (1, 2), 3)) #False
print(p.constraints((1, 3), 3, (1, 2), 2)) #True
print(p.constraints((4, 4), 2, (2, 4), 2)) #False
print(p.constraints((3, 2), 3, (3, 4), 2)) #True
```

## Solução do futoshiki

Vamos agora usar o algoritmo **BACKTRACKING-SEARCH** para resolver o problema. 


### Backtracking sem inferencia

Vamos invocar a profundidade com retrocesso com os argumentos por omissão.

```python
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p)
print('Assignment p = ',r) 
```

Vamos colocar o verbose=True para seguirmos com detalhe todo o processo de resolução do CSP.

```python
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,verbose = True)
print('Assignment p = ',r) 
```

### Inferência Forward Checking

```python
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,inference = forward_checking)
print('Assignment = ',r)
```

Agora, a versão com verbose = True

```python
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,inference = forward_checking, verbose = True)
print('Assignment = ',r)
```

### Procura com inferência em que se mantém a consistência (mac), ou AC3

```python
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,inference = mac, verbose=True)
print('Afetações r = ',r)
```

### Utilização de heurísticas no Backtracking-Search (MRV)

***NOTA:*** Para resolver este problema com verbose=True é preciso mais do que um minuto

```python
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,select_unassigned_variable = mrv)
print('Afetações r = ',r)
```

## Visualização do problema

A função display_futoshiki, recebe como parametro uma solução final = r, instrucooes e restricoes se necessário.

![image.png](attachment:image.png)


```python
def concatena(estado):
    
    posicoes = list(estado.keys())
    
    n = int(math.sqrt((len(posicoes))))
    
    col = "  "
    values = ""
    for l in range(1,n+1):
        col += " | " + str(l) + "c" 
        values += str(l) + "l" + " |  "
        for c in range(1,n+1):
            values += str(estado[(l,c)]) + " |  "
        values += '\n'
    
    col += " | "
    return col,values
    
    
def display_futoshiki(final, instrucoes = {}, restricoes = {}):
    
    print("Estado Inicial: ")
    
    dic = {}
    
    for var in list(final.keys()):
        if var in instrucoes.keys():
            dic[var] = str(instrucoes[var])
        else:
            dic[var] = '.'
    
    estado_inicial = concatena(dic)
    
    print(estado_inicial[0])
    print(estado_inicial[1])
    
    print("Restrições: ")
    
    if restricoes == {}:
        print("Não há restrições")
    count = 1
    for key,value in restricoes.items():
        
        if str(value.__name__) in [str(maior.__name__), str(menor.__name__)]:
            print("R{}: ".format(count) + str(key) + " -> " + str(value.__name__))
        count +=1
    
    
    print()
    print("Estado Final: ") 
    
    estado_final = concatena(final)

    print(estado_final[0])
    print(estado_final[1])

    
display_futoshiki(r,instrucoes, restricoes)
```

### EM SUMA

1. MAC (Mantain Arc Consistenty, usando o AC3): Utilizando o AC3 após cada afectação da variável é mais caro computacionalmente.

2. Inferência Forward Checking, o menos caro computacionalmente.

```python
import time

#Backtracking sem inferencia
start = time.time()
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p)
fim = time.time() - start
print("Backtracking sem inferencia:",fim)

#Inferência Forward Checking
start = time.time()
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,inference = forward_checking)
fim = time.time() - start
print("Inferência Forward Checking:",fim)

#Procura com inferência em que se mantém a consistência (mac), ou AC3
start = time.time()
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,inference = mac)
fim = time.time() - start
print("MAC:",fim)

#Utilização de heurísticas no Backtracking-Search (MRV)
start = time.time()
p = futoshiki(5,instrucoes,restricoes)
r = backtracking_search(p,select_unassigned_variable = mrv)
fim = time.time() - start
print("MRV:",fim)

```

```python

```
