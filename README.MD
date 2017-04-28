# Projeto 2 - Grupo 06
Allana Idalgo 145166
Lucas Calzolari 146987
Matheus Koezuka 137017

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
* Pipeline : 5, 7 e 13 estágios
* Escalar vs Superescalar
* Branch predictor : Sempre especular que a alteração no fluxo sempre ocorre, ou preditor de 2 bits.
* Cache : 4 combinações de tamanho de cache e de bloco da cache. (Cache de instruções e a l2 não serão alterados)

Os parametros relacionados a superescalar e cache serão avaliados isoladamente, enquanto todas as combinações de pipelines e branch predictors serão avaliadas para se evitar muitas combinações possíveis. Totalizando assim 8 configurações distintas.