# Problema 1 A QUESTING Pesquisas Eleitorais

A QUESTING Pesquisas Eleitorais precisa de operadores de campo para a coleta de dados e realiza um programa de treinamento para novos contratados. Historicamente, apenas 70% dos trainees concluem o treinamento. A demanda mensal por operadores treinados e as restrições impostas pelo acordo sindical são fatores críticos na formulação deste problema de otimização. O objetivo é minimizar os custos de contratação e treinamento, enquanto se atende à demanda mensal.


## Configurações do projeto


```python
!pip install -q amplpy pandas

```


```python
from amplpy import AMPL, tools
ampl = tools.ampl_notebook(
modules=["highs"], 
license_uuid="default",
g=globals()) 
```

    AMPL Development Version 20240606 (Linux-5.15.0-1064-azure, 64-bit)
    Demo license with maintenance expiring 20260131.
    Using license file "/home/sehmer/Documents/sync/EMC/PO1/teste_1/.venv/lib/python3.12/site-packages/ampl_module_base/bin/ampl.lic".
    


# Formulação do problema

## 1. Variáveis de Decisão
Definimos as seguintes variáveis de decisão:
- ***\( x_n \)***: Número de trainees contratados no mês `( n )` (onde `( n )` varia de 1 a 4, representando os meses de janeiro a abril).
- (***\( y_n \)***): Número de operadores coletando dados no mês `\( n \)`.
- ***\( z_n \)***: Número de operadores ensinando no mês `( n )`.
- ***\( o_n \)***: Número de operadores ociosos no mês `( n )`.



```python
%%ampl_eval

var x{1..4} >= 0;  
var y{1..4} >= 0;  
var z{1..4} >= 0;  
var o{1..4} >= 0;  

param Trainee_Cost := 400;
param Operator_Cost := 700;
param Idle_Cost := 500;


param Initial_Operators := 130;
```

## 2. Função Objetivo
A função objetivo é minimizar o custo total associado à contratação, treinamento e manutenção de operadores. O custo mensal inclui:
- Custo por trainee: 400 dólares.
- Custo por operador treinado (coletando ou ensinando): 700 dólares.
- Custo por operador ocioso: 500 dólares.

A função objetivo é expressa como:
`sum{t in 1..4} (Trainee_Cost * x[t] + Operator_Cost * (y[t] + z[t]) + Idle_Cost * o[t]);`


```python
%%ampl_eval



minimize Total_Cost:
    sum{t in 1..4} (Trainee_Cost * x[t] + Operator_Cost * (y[t] + z[t]) + Idle_Cost * o[t]);

```

## 3. Restrições
As seguintes restrições foram formuladas para garantir que a demanda seja atendida e que o balanço de operadores seja mantido:

### Restrições de Demanda Mensal
- **Janeiro**: `( y_1 = 100 )`
- **Fevereiro**: `( y_2 = 150 )`
- **Março**: `( y_3 = 200 )`
- **Abril**: `( y_4 = 250 )`

### Restrições de Treinamento
Cada instrutor treina até 10 novos contratados:
- **Janeiro**: `( z_1 * 10 = x_1 )`
- **Fevereiro**: `( z_2 * 10 = x_2 )`
- **Março**: `( z_3 * 10 = x_3 )`

### Restrições de Operadores Disponíveis
Para cada mês, o número total de operadores deve ser mantido:
- **Janeiro**:
  \[
  `y_1 + z_1 + o_1 = 130`
  \]
- **Fevereiro**:
  \[
  `y_2 + z_2 + o_2 = y_1 + z_1 + o_1 + 0.7 * x_1`
  \]
- **Março**:
  \[
  `y_3 + z_3 + o_3 = y_2 + z_2 + o_2 + 0.7 * x_2`
  \]
- **Abril**:
  \[
  `y_4 + z_4 + o_4 = y_3 + z_3 + o_3 + 0.7 * x_3`
  \]


```python
%%ampl_eval


subject to Demanda_Janeiro:
    y[1]  =  100;

subject to Demanda_Fevereiro:
    y[2] = 150;

subject to Demanda_Marco:
    y[3] = 200;

subject to Demanda_Abril:
    y[4] = 250;

subject to Trainers_Janeiro:
    z[1] *  10 =  x[1];
    
subject to Trainers_Fevereiro:
    z[2] *  10 =  x[2];

subject to Trainers_Marco:
    z[3] *  10 =  x[3];
    

subject to Operadores_Janeiro:
    y[1] + z[1] + o[1] = Initial_Operators ;

subject to Operadores_Fevereiro:
    y[2] + z[2] + o[2] = y[1] + z[1] + o[1] + 0.7 * x[1];

subject to Operadores_Marco:
    y[3] + z[3] + o[3] = y[2] + z[2] + o[2] + 0.7 * x[2];


subject to Operadores_Abril:
    y[4] + z[4] + o[4] = y[3] + z[3] + o[3] + 0.7 * x[3];




```

## Resultados da Análise de Custos - Modelo de Programação Linear



```python
%%ampl_eval
# Solucionar o problema com o solver Highs
option solver 'highs';
solve;

# Exibir os resultados
display x, y, z, o, Total_Cost;

```

    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 583640.625
    4 simplex iterations
    0 barrier iterations
    :      x       y       z         o       :=
    1   38.6161   100   3.86161   26.1384
    2   70.3125   150   7.03125    0
    3   62.5      200   6.25       0
    4    0        250   0          0
    ;
    
    Total_Cost = 583641
    



## Resumo da Solução

A solução do problema foi obtida utilizando o solver HiGHS na versão 1.7.1. A solução ótima encontrada foi:

- **Valor Objetivo**: $583640.625
- **Total de Iterações**:
  - 4 iterações do método Simplex
  - 0 iterações do método de Barreira

## Resultados das Variáveis de Decisão

As variáveis de decisão resultantes são as seguintes:

| Mês       | Trainees Contratados (x) | Operadores Coletando (y) | Operadores Ensinando (z) | Operadores Ociosos (o) |
|-----------|---------------------------|---------------------------|---------------------------|-------------------------|
| Janeiro   | 38.6161                   | 100                       | 3.86161                   | 26.1384                 |
| Fevereiro | 70.3125                   | 150                       | 7.03125                   | 0                       |
| Março     | 62.5                      | 200                       | 6.25                      | 0                       |
| Abril     | 0                         | 250                       | 0                         | 0                       |

## Custo Total

O custo total associado à solução proposta é de **$583641**.




```python

```
