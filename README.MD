# Projeto 2 Roteiro & Relatório - Grupo 06
Allana Idalgo 145166  
Lucas Calzolari 146987  
Matheus Koezuka 137017  

# Roteiro

## Metodologia
A implementação do simulador mips será modificada para permitir um registro de metainformações baseada em hazards e cache misses. A biblioteca do dinero será utilizada para contabilizar os acessos á memória e configuração de diferentes tamanhos de memória e cache. E na implementação das instruções um controle sobre hazards e bolhas será implementado para se obter o número real de ciclos baseado no tamanho do pipeline e no fato de ser ou não super escalar.

## Hazards
Hazards ocorrem quando a execução de uma instrução depende de outra á sua frente no pipeline que ainda não foi concluída, sendo classificada em 2 tipos:

* Data hazards quando uma operação utiliza o conteúdo de um registrador que será escrito por outra instrução ainda no pipeline, que pode ser evitado em alguns casos utilizando forwarding.
* Control hazards quando há uma operação que irá sobrescrever o registrador PC, assim alterando o fluxo de execução. Esse tipo de hazard pode ser evitado em alguns casos utilizando-se branch prediction, que baseado no histórico de execução tenta advinhar se um pulo condicional ocorrerá ou não.

### Hazard para instruções
A tabela a baixo inclui algumas características relevantes para a identificação do tipo de hazard que cada instrução poderá gerar.

| Instructions   |      read register      |  write register | read memory | write memory | write pc |
|----------|:------:|:----:|:-----: |:------:|:----:|
|ADD|\*|\*||||
|ADDI|\*|\*||||
|ADDIU|\*|\*||||
|ADDU|\*|\*||||
|AND|\*|\*||||
|ANDI|\*|\*||||
|BEQ|\*||||\*|
|BGEZ|\*||||\*|
|BGEZAL|\*||||\*|
|BGTZ|\*||||\*|
|BLEZ|\*||||\*|
|BLTZ|\*||||\*|
|BLTZAL|\*||||\*|
|BNE|\*||||\*|
|DIV|\*|\*||||
|DIVU|\*|\*||||
|J|||||\*|
|JAL|||||\*|
|JR|\*||||\*|
|LB|\*|\*|\*|||
|LUI||\*||||
|LW|\*|\*|\*|||
|MFHI|\*|\*||||
|MFLO|\*|\*||||
|MULT|\*|\*||||
|MULTU|\*|\*||||
|NOOP||||||
|OR|\*|\*||||
|ORI|\*|\*||||
|SB|\*|||\*||
|SLL|\*|\*||||
|SLLV|\*|\*||||
|SLT|\*|\*||||
|SLTI|\*|\*||||
|SLTIU|\*|\*||||
|SLTU|\*|\*||||
|SRA|\*|\*||||
|SRL|\*|\*||||
|SRLV|\*|\*||||
|SUB|\*|\*||||
|SUBU|\*|\*||||
|SW|\*|||\*||
|XOR|\*|\*||||
|XORI|\*|\*||||

## Configurações
As configurações considerarão os seguintes parâmetros:
* Pipeline : 5, 7 e 9 estágios
* Escalar vs Superescalar
* Branch predictor : Sempre especular que a alteração no fluxo sempre ocorre, ou preditor de 2 bits.
* Cache : 4 combinações de tamanho de cache e de bloco da cache. (Cache de instruções e a l2 não serão alterados)

Os parametros relacionados a superescalar e cache serão avaliados isoladamente, enquanto todas as combinações de pipelines e branch predictors serão avaliadas para se evitar muitas combinações possíveis. Totalizando assim 8 configurações distintas.

# Relatório 

## Pipeline
Para os testes foram avaliadas três arquiteturas de pipelines diferentes, contendo 5, 7 e 9 estágios.  Considerando que não havia forwarding permitindo uma avaliação do impacto dos data hazards. 

## Branch Predictor
O arquivo mips_isa.cpp foi alterado para gerar um arquivo de saída "output.txt" contendo os endereços de todos os branches e se ele realizou o salto ou não. Em seguida foi criado um programa em python denominado analyze_branch.py que lê esse arquivo de saída linha por linha e determina se ocorreu um salto que não deveria ou o inverso; não ocorreu um salto e deveria. Em ambos os casos é somado um contador b_miss. O código de analyze_branch.py para 1 bit pode ser visto abaixo:

```
with open("output.txt", "r") as ins:
    d = {}
    b_miss = 0
    lines = ins.readlines()
    for line in lines:
        a = line.strip().split(" ")
        d[a[0].strip()] = 0
    print(d)
    for line in lines:
        a = line.strip().split(" ")
        a[0] = a[0].strip()
        if(len(a) > 1):  # pulou
            if(d[a[0]] == 0):  # não deveria
                b_miss += 1
                d[a[0]] = 1
        else:  # não pulou
            if(d[a[0]] == 1):  # deveria
                b_miss += 1
                d[a[0]] = 0
    print(b_miss)
```
Assim, foram obtidos os seguintes números (porcentagem calculada através do número total de linhas em output.txt):

|        Programa       | sha (small) | qsort (small) | dijkstra (large) |
|:---------------------:|:-----------:|---------------|------------------|
|     Branch Misses     |    70606    |     758296    |     16319658     |
| Porcentagem de Misses |    12.45%   |     28.02%    |      38.94%      |

Para 2bits, e código acima foi modificado para:
```
with open("output.txt", "r") as ins:
    d = {}
    b_miss = 0
    lines = ins.readlines()
    for line in lines:
        a = line.strip().split(" ")
        d[a[0].strip()] = 0
    print(d)
    for line in lines:
        a = line.strip().split(" ")
        a[0] = a[0].strip()
        if(len(a) > 1):  # pulou
            if(d[a[0]] != 2):  # not strongly taken
                if(d[a[0]] != 1): # not taken
                    b_miss += 1
                d[a[0]] += 1
        else:  # não pulou
            if(d[a[0]] != -1):  # not strongly not taken
                if(d[a[0]] != 0):  # taken
                    b_miss += 1
                d[a[0]] -= 1
    print(b_miss)
```
E os resultados foram:

|        Programa       | sha (small) | qsort (small) | dijkstra (large) |
|:---------------------:|:-----------:|---------------|------------------|
|     Branch Misses     |    46444    |     837126    |     9002438      |
| Porcentagem de Misses |    8.18%    |     30.94%    |      21.48%      |

Por fim, para um predictor "always taken" foi feito um grep da forma "grep " $" output.txt |wc" para determinar o número de linhas com espaço no final (ou seja, o número de branches que não pularam). Utilizando novamente o número total de linhas do arquivo output.txt para obter total de branches, chegamos nos seguintes dados:

|        Programa       | sha (small) | qsort (small) | dijkstra (large) |
|:---------------------:|:-----------:|---------------|------------------|
|     Branch Misses     |    41550    |     684031    |     23953352     |
| Porcentagem de Misses |    7.32%    |     25.28%    |      57.15%      |

## Cache
Testamos quatro variações de tamanho da cache L1: 8k, 16k, 32k, 64k com o intuito de encontrar a melhor configuração. Então, utilizamos o Dinero para obter os miss rates, e os resultados podem ser vistos abaixo:

 - QSort

| dcache size | Total Misses  | Miss Rate |
|-------------|---------------|-----------|
| 8k          | 289473        | 0.0183    |
| 16k         | 234836        | 0.0149    |
| 32k         | 197261        | 0.0125    |
| 64k         | 166616        | 0.0105    |


 - Sha
 
| dcache size | Total Misses  | Miss Rate |
|-------------|---------------|-----------|
| 8k          | 36635         | 0.0268    |
| 16k         | 31686         | 0.0232    |
| 32k         | 30428         | 0.0223    |
| 64k         | 29040         | 0.0213    |

 

 - Djisktra

 | dcache size | Total Misses  | Miss Rate |
|-------------|---------------|-----------|
| 8k          | 229169        | 0.0263    |
| 16k         | 171930        | 0.0198    |
| 32k         | 93925         | 0.0108    |
| 64k         | 25203         | 0.0029    |

Considerando os valores acima, concluimos que o maior custo benefício entre queda de miss rates e aumento de cache size foi 16k, sendo esta a configuração escolhida para avaliar os demais parâmetros.

## Configurações  

| tamanho pipeline | cache bloco  | cache l1     | Branch algorithm |
|------------------|--------------|--------------|------------------|
| 5                | 8            | cache 1      | always           |
| 5                | 16           | cache 2      | always           |
| 5                | 32           | cache 1      | always           |
| 5                | 64           | cache 2      | always           |
| 5                | bloco melhor | cache melhor | 2 bit            |
| 7                | bloco melhor | cache melhor | always           |
| 7                | bloco melhor | cache melhor | 2 bit            |
| 9                | bloco melhor | cache melhor | always           |
| 9                | bloco melhor | cache melhor | 2 bit            |

  

# Resultados
## Tabelas
- Sha  

| configs | Ciclos   | Data hazards | Instruções | branch miss | branch cost | number branches fail | cache miss rate | memory access | miss cost | Total Ciclos | CPI         |
|---------|----------|--------------|------------|-------------|-------------|----------------------|-----------------|---------------|-----------|--------------|-------------|
| 1       | 9496914  | 1040275      | 6968956    | 0,0732      | 2           | 41550                | 0,0268          | 1366102       | 200       | 16902321     | 2,42537343  |
| 2       | 9496914  | 1040275      | 6968956    | 0,0732      | 2           | 41550                | 0,0232          | 1366102       | 200       | 15918727     | 2,284234149 |
| 3       | 9496914  | 1040275      | 6968956    | 0,0732      | 2           | 41550                | 0,0223          | 1366102       | 200       | 15672829     | 2,248949329 |
| 4       | 9496914  | 1040275      | 6968956    | 0,0732      | 2           | 41550                | 0,0213          | 1366102       | 200       | 15399609     | 2,209743973 |
| 5       | 9496914  | 1040275      | 6968956    | 0,0818      | 2           | 46444                | 0,0232          | 1366102       | 200       | 15928515     | 2,285638664 |
| 6       | 10711412 | 1420866      | 6968956    | 0,0732      | 3           | 41550                | 0,0232          | 1366102       | 200       | 17174775     | 2,464468893 |
| 7       | 10711412 | 1420866      | 6968956    | 0,0818      | 3           | 46444                | 0,0232          | 1366102       | 200       | 17189457     | 2,466575665 |
| 8       | 12171865 | 1669836      | 6968956    | 0,0732      | 4           | 41550                | 0,0232          | 1366102       | 200       | 18676778     | 2,679996585 |
| 9       | 12171865 | 1669836      | 6968956    | 0,0818      | 4           | 46444                | 0,0232          | 1366102       | 200       | 18696354     | 2,682805614 |


 - Qsort  
 
 | configs | Ciclos   | Data hazards | Instruções | branch miss | branch cost | number branches | cache miss rate | memory access | miss cost |           |             |
|---------|----------|--------------|------------|-------------|-------------|-----------------|-----------------|---------------|-----------|-----------|-------------|
| 1       | 53484099 | 5994545      | 39454159   | 0,2528      | 2           | 684031          | 0,0183          | 15797640      | 200       | 112671523 | 2,855757828 |
| 2       | 53484099 | 5994545      | 39454159   | 0,2528      | 2           | 684031          | 0,0149          | 15797640      | 200       | 101929128 | 2,583482471 |
| 3       | 53484099 | 5994545      | 39454159   | 0,2528      | 2           | 684031          | 0,0125          | 15797640      | 200       | 94346261  | 2,391288102 |
| 4       | 53484099 | 5994545      | 39454159   | 0,2528      | 2           | 684031          | 0,0105          | 15797640      | 200       | 88027205  | 2,231126128 |
| 5       | 53484099 | 5994545      | 39454159   | 0,3094      | 2           | 837126          | 0,0149          | 15797640      | 200       | 102235318 | 2,591243123 |
| 6       | 59795234 | 6311135      | 39454159   | 0,2528      | 3           | 684031          | 0,0149          | 15797640      | 200       | 108924294 | 2,760781042 |
| 7       | 59795234 | 6311135      | 39454159   | 0,3094      | 3           | 837126          | 0,0149          | 15797640      | 200       | 109383579 | 2,77242202  |
| 8       | 66164149 | 6369159      | 39454159   | 0,2528      | 4           | 684031          | 0,0149          | 15797640      | 200       | 115977240 | 2,939544097 |
| 9       | 66164149 | 6369159      | 39454159   | 0,3094      | 4           | 837126          | 0,0149          | 15797640      | 200       | 116589620 | 2,955065401 |


 - Dijkstra  
 
 | configs | Ciclos   | Data hazards | Instruções | branch miss | branch cost | number branches | cache miss rate | memory access | miss cost |           |             |
|---------|----------|--------------|------------|-------------|-------------|-----------------|-----------------|---------------|-----------|-----------|-------------|
| 1       | 40358286 | 4431977      | 28682038   | 0,5715      | 2           | 23953352        | 0,0263          | 8702765       | 200       | 134041534 | 4,673361562 |
| 2       | 40358286 | 4431977      | 28682038   | 0,5715      | 2           | 23953352        | 0,0198          | 8702765       | 200       | 122727939 | 4,278912796 |
| 3       | 40358286 | 4431977      | 28682038   | 0,5715      | 2           | 23953352        | 0,0108          | 8702765       | 200       | 107062962 | 3,732752965 |
| 4       | 40358286 | 4431977      | 28682038   | 0,5715      | 2           | 23953352        | 0,0029          | 8702765       | 200       | 93312594  | 3,253346004 |
| 5       | 40358286 | 4431977      | 28682038   | 0,2148      | 2           | 9002438         | 0,0198          | 8702765       | 200       | 92826111  | 3,236384785 |
| 6       | 44917932 | 4559646      | 28682038   | 0,5715      | 3           | 23953352        | 0,0198          | 8702765       | 200       | 151240937 | 5,273019212 |
| 7       | 44917932 | 4559646      | 28682038   | 0,2148      | 3           | 9002438         | 0,0198          | 8702765       | 200       | 106388195 | 3,709227196 |
| 8       | 50499259 | 5581332      | 28682038   | 0,5715      | 4           | 23953352        | 0,0198          | 8702765       | 200       | 180775616 | 6,302746562 |
| 9       | 50499259 | 5581332      | 28682038   | 0,2148      | 4           | 9002438         | 0,0198          | 8702765       | 200       | 120971960 | 4,217690542 |
 


