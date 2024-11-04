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
# Problema 2 Problema da RoboCarga S.A.

## Configurações do projeto


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
    


# Parte 1


## Formulação do Modelo
A RoboCarga S.A. precisa planejar a produção de robôs de carregamento para atender a uma demanda crescente ao longo de quatro meses. O problema pode ser abordado como um modelo de programação linear, onde o objetivo é minimizar os custos totais, que incluem:

- ***Custo de Fabricação***: $400 por robô produzido.
- ***Custo de Armazenamento***: $100 por robô mantido em estoque ao final do mês.
- ***Custo de Treinamento e Ajuste***: $700 por robô adicional quando a produção aumenta em relação ao mês anterior.
- ***Custo de Rescisão e Reorganização***: $600 por robô quando a produção diminui em relação ao mês anterior.

## ***Variáveis de Decisão***

- ***Producao_robo_a[n]***: Número de robôs produzidos no mês (onde n=1,2,3,4).
- ***Estoque_robo_a[n]***: Número de robôs em estoque ao final do mês n.


```python
%%ampl_eval

var Producao_robo_a{1..4} >= 0;  
var Estoque_robo_a{0..4} >= 0;  


param Custo_Fabricacao_a := 400;
param Custo_Estoque_a := 100;
param Custo_Aumento_a := 700;
param Custo_Diminuicao_a := 600;

```

## Função Objetivo
Minimizar o custo total:



```python
%%ampl_eval


minimize Custo_Total_a:
    sum{t in 1..4} (Custo_Fabricacao_a * Producao_robo_a[t] + Custo_Estoque_a * Estoque_robo_a[t]) +
    sum{t in 2..4} (Custo_Aumento_a * (max(Producao_robo_a[t] - Producao_robo_a[t-1], 0))) +
    sum{t in 2..4} (Custo_Diminuicao_a * (max(Producao_robo_a[t-1] - Producao_robo_a[t], 0)));

```

## Restrições
Atender a demanda mensal:

- Mês 1: `Producao_robo_a[1] + Estoque_robo_a[0] − Estoque_robo_a[1]=40`
- Mês 2:` Producao_robo_a[2] + Estoque_robo_a[1] − Estoque_robo_a[2]=70`
- Mês 3: `Producao_robo_a[3] + Estoque_robo_a[2] − Estoque_robo_a[3]=50`
- Mês 4: `Producao_robo_a[4] + Estoque_robo_a[3] − Estoque_robo_a[4]=20`


Condições iniciais:
- `Estoque_robo_a[0]=0`
- `Estoque_robo_a[4]=0`






```python
%%ampl_eval

subject to Demanda_mes_1_a:
    Producao_robo_a[1] + Estoque_robo_a[0] - Estoque_robo_a[1] = 40;
    
subject to Demanda_mes_2_a:
    Producao_robo_a[2] + Estoque_robo_a[1] - Estoque_robo_a[2] = 70;    

subject to Demanda_mes_3_a:
    Producao_robo_a[3] + Estoque_robo_a[2] - Estoque_robo_a[3] = 50;
    
subject to Demanda_mes_4_a:
    Producao_robo_a[4] + Estoque_robo_a[3] - Estoque_robo_a[4] = 20;



subject to Estoque_Inicial_a:
    Estoque_robo_a[0] = 0;
    
subject to Estoque_Final_a:
    Estoque_robo_a[4] = 0;
```

## Resultados


```python
%%ampl_eval


option solver 'highs';  
solve;


display Producao_robo_a, Estoque_robo_a, Custo_Total_a;

```

    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 94500
    7 simplex iterations
    0 barrier iterations
    : Producao_robo_a Estoque_robo_a    :=
    0         .               0
    1        55              15
    2        55               0
    3        50               0
    4        20               0
    ;
    
    Custo_Total_a = 94500
    


## Resumo da Solução

A solução do problema foi obtida utilizando o solver HiGHS na versão 1.7.1. A solução ótima encontrada foi:

- **Valor Objetivo**: $94,500
- **Total de Iterações**:
  - 7 iterações do método Simplex
  - 0 iterações do método de Barreira

## Resultados das Variáveis de Decisão

As variáveis de decisão resultantes são as seguintes:

| Mês       | Produção de Robôs (Producao_robo_a) | Estoque Final (Estoque_robo_a) |
|-----------|--------------------------------------|---------------------------------|
| 0         | 0                                    | 0                               |
| 1         | 55                                   | 15                              |
| 2         | 55                                   | 0                               |
| 3         | 50                                   | 0                               |
| 4         | 20                                   | 0                               |

## Custo Total

O custo total associado à solução proposta é de **$94,500**.

## Análise dos Resultados

A análise dos resultados indica o seguinte:

- **Produção de Robôs**: O maior nível de produção ocorreu nos meses 1 e 2, com 55 robôs produzidos em cada um desses meses, atendendo a uma demanda significativa. A produção diminui nos meses 3 e 4, sugerindo um ajuste à demanda reduzida.

- **Estoque Final**: O estoque ao final do mês 1 é de 15 robôs, o que pode indicar um planejamento eficiente que antecipa a demanda. Nos meses subsequentes, o estoque é zerado, indicando que a produção foi bem alinhada com as necessidades de demanda.


# Parte 2: Restrições de Capacidade de Produção

### Formulação do Problema
Agora, a RoboCarga S.A. enfrenta restrições de capacidade de produção, onde há um limite de robôs que podem ser produzidos em cada mês:

- Mês 1: 45 robôs
- Mês 2: 75 robôs
- Mês 3: 55 robôs
- Mês 4: 30 robôs

## Restrições

Restrições de demanda (iguais à Parte 1).

Restrições de capacidade:
- `Producao_robo_b[1] ≤ 45`
- `Producao_robo_b[2] ≤ 75`
- `Producao_robo_b[3] ≤ 55`
- `Producao_robo_b[4] ≤ 30`






```python
%%ampl_eval

var Producao_robo_b{1..4} >= 0;  
var Estoque_robo_b{0..4} >= 0;  

param Custo_Fabricacao_b := 400;
param Custo_Estoque_b := 100;
param Custo_Aumento_b := 700;
param Custo_Diminuicao_b := 600;

minimize Custo_Total_b:
    sum{t in 1..4} (Custo_Fabricacao_b * Producao_robo_b[t] + Custo_Estoque_b * Estoque_robo_b[t]) +
    sum{t in 2..4} (Custo_Aumento_b * (max(Producao_robo_b[t] - Producao_robo_b[t-1], 0))) +
    sum{t in 2..4} (Custo_Diminuicao_b * (max(Producao_robo_b[t-1] - Producao_robo_b[t], 0)));

subject to Demanda_mes_1_b:
    Producao_robo_b[1] + Estoque_robo_b[0] - Estoque_robo_b[1] = 40;
    
subject to Demanda_mes_2_b:
    Producao_robo_b[2] + Estoque_robo_b[1] - Estoque_robo_b[2] = 70;    

subject to Demanda_mes_3_b:
    Producao_robo_b[3] + Estoque_robo_b[2] - Estoque_robo_b[3] = 50;
    
subject to Demanda_mes_4_b:
    Producao_robo_b[4] + Estoque_robo_b[3] - Estoque_robo_b[4] = 20;


subject to restricao_mes_1_b:
    Producao_robo_b[1] = 45;
    
subject to restricao_mes_2_b:
    Producao_robo_b[2] = 75;
    

subject to restricao_mes_3_b:
    Producao_robo_b[3] = 55;
    
subject to restricao_mes_4_b:
    Producao_robo_b[4] = 30;    
    

subject to Estoque_Inicial_b:
    Estoque_robo_b[0] = 0;
    
subject to Estoque_Final_b:
    Estoque_robo_b[4] = 0;


option solver 'highs';  
solve;


display Producao_robo_b, Estoque_robo_b, Custo_Total_b;

```

    Warning:
    	presolve, variable Estoque_robo_b[3]:
    		impossible deduced bounds: lower = 0, upper = -10
    : Producao_robo_b Estoque_robo_b    :=
    0         .               0
    1        45               5
    2        75              10
    3        55              15
    4        30               0
    ;
    
    Custo_Total_b = 133000
    


## Resultados das Variáveis de Decisão parte 2

As variáveis de decisão resultantes são as seguintes:

| Mês       | Produção de Robôs (Producao_robo_a) | Estoque Final (Estoque_robo_a) |
|-----------|--------------------------------------|---------------------------------|
| 0         | 0                                    | 0                               |
| 1         | 45                                   | 5                               |
| 2         | 75                                   | 10                              |
| 3         | 55                                   | 15                              |
| 4         | 30                                   | 0                               |

## Custo Total

O custo total associado à solução proposta é de **$ 133,000**.

# Parte 3: Repetição do Problema

Considerando que o problema se repete por três quadrimestres ao longo do ano, o modelo pode ser adaptado para atender a 12 meses, considerando a mesma demanda mensal, porém ajustada para um período de 12 meses. As variáveis, a função objetivo e as restrições são estendidas para os novos meses.




```python
from amplpy import AMPL, Environment

def modelo_parte_c():
    ampl = AMPL(Environment())
    ampl.eval('''
    option solver 'highs';  # Ou outro solver que você tenha instalado

    var Producao_robo_c{1..12} >= 0;  # Produção de robôs no mês t
    var Estoque_robo_c{0..12} >= 0;  # Estoque de robôs ao final do mês t

    param Custo_Fabricacao_c := 400;
    param Custo_Estoque_c := 100;
    param Custo_Aumento_c := 700;
    param Custo_Diminuicao_c := 600;

    minimize Custo_Total_c:
        sum{t in 1..12} (Custo_Fabricacao_c * Producao_robo_c[t] + Custo_Estoque_c * Estoque_robo_c[t]) +
        sum{t in 2..12} (Custo_Aumento_c * (max(Producao_robo_c[t] - Producao_robo_c[t-1], 0))) +
        sum{t in 2..12} (Custo_Diminuicao_c * (max(Producao_robo_c[t-1] - Producao_robo_c[t], 0)));

    subject to Demanda_mes_1_c:
        Producao_robo_c[1] + Estoque_robo_c[0] - Estoque_robo_c[1] = 40;

    subject to Demanda_mes_2_c:
        Producao_robo_c[2] + Estoque_robo_c[1] - Estoque_robo_c[2] = 70;    

    subject to Demanda_mes_3_c:
        Producao_robo_c[3] + Estoque_robo_c[2] - Estoque_robo_c[3] = 50;
        
    subject to Demanda_mes_4_c:
        Producao_robo_c[4] + Estoque_robo_c[3] - Estoque_robo_c[4] = 20;

    subject to Demanda_mes_5_c:
        Producao_robo_c[5] + Estoque_robo_c[4] - Estoque_robo_c[5] = 40;

    subject to Demanda_mes_6_c:   
        Producao_robo_c[6] + Estoque_robo_c[5] - Estoque_robo_c[6] = 70;

    subject to Demanda_mes_7_c:
        Producao_robo_c[7] + Estoque_robo_c[6] - Estoque_robo_c[7] = 50;

    subject to Demanda_mes_8_c:
        Producao_robo_c[8] + Estoque_robo_c[7] - Estoque_robo_c[8] = 20;

    subject to Demanda_mes_9_c:
        Producao_robo_c[9] + Estoque_robo_c[8] - Estoque_robo_c[9] = 40;
        
    subject to Demanda_mes_10_c:
        Producao_robo_c[10] + Estoque_robo_c[9] - Estoque_robo_c[10] = 70;    

    subject to Demanda_mes_11_c:
        Producao_robo_c[11] + Estoque_robo_c[10] - Estoque_robo_c[11] = 50;
        
    subject to Demanda_mes_12_c:
        Producao_robo_c[12] + Estoque_robo_c[11] - Estoque_robo_c[12] = 20;

    subject to Estoque_Inicial_c:
        Estoque_robo_c[0] = 0;
        
    subject to Estoque_Final_c:
        Estoque_robo_c[12] = 0;

    solve;

    ''')
    ampl.eval('display Producao_robo_c, Estoque_robo_c, Custo_Total_c;')
    return ampl

# Execução dos modelos
ampl_c = modelo_parte_c()
 # Para ver se há algo no estoque

```

    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 250500
    27 simplex iterations
    0 barrier iterations
    :  Producao_robo_c Estoque_robo_c    :=
    0          .               0
    1         55              15
    2         55               0
    3         50               0
    4         45              25
    5         45              30
    6         45               5
    7         45               0
    8         45              25
    9         45              30
    10        45               5
    11        45               0
    12        20               0
    ;
    
    Custo_Total_c = 250500
    


## Resultados das Variáveis de Decisão

| Mês       | Produção de Robôs (Producao_robo_c) | Estoque Final (Estoque_robo_c) |
|-----------|--------------------------------------|---------------------------------|
| 0         | 0                                    | 0                               |
| 1         | 55                                   | 15                              |
| 2         | 55                                   | 0                               |
| 3         | 50                                   | 0                               |
| 4         | 45                                   | 25                              |
| 5         | 45                                   | 30                              |
| 6         | 45                                   | 5                               |
| 7         | 45                                   | 0                               |
| 8         | 45                                   | 25                              |
| 9         | 45                                   | 30                              |
| 10        | 45                                   | 5                               |
| 11        | 45                                   | 0                               |
| 12        | 20                                   | 0                               |

## Custo Total

O custo total associado à solução proposta é de **$250,500**.



# Discussão os resultados.

### Parte 1: Sem Restrições de Capacidade
- **Custo Total**: **$94,500**.
- A produção de **55 robôs** nos primeiros meses atende bem à demanda crescente, resultando em um estoque final de **15 robôs** no mês 1.

### Parte 2: Com Restrições de Capacidade
- **Custo Total**: Aumenta para **$133,000** devido às limitações de produção.
- As restrições impactam a capacidade de atender à demanda ideal, resultando em estoques maiores e custos adicionais.
- A produção é maximizada ao limite imposto, mas a necessidade de ajustes gera custos.

### Parte 3: Repetição do Problema
- **Custo Total**: **$250,500** ao longo de 12 meses.
- A produção é ajustada mensalmente para atender à demanda, mostrando uma gestão eficiente dos estoques.


### Conclusão
Os resultados enfatizam a importância do planejamento estratégico e da gestão eficiente da produção e do estoque para a RoboCarga S.A., visando minimizar custos e atender a uma demanda crescente no mercado de robôs de carregamento.
# Parte 3 Problema da VigilanceCo

A VigilanceCo deseja otimizar a contratação e o escalonamento de vigilantes em período integral para atender à demanda diária de vigilância, respeitando as exigências sindicais e minimizando custos. O problema é dividido em três partes, cada uma com suas características e restrições.

## Configurações do projeto


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
    


# Parte A: Minimização de Vigilantes em Período Integral


## Definição das Variáveis



```python
%%ampl_eval

var vigilante_segunda >= 0;  
var vigilante_terca >= 0;  
var vigilante_quarta >= 0;  
var vigilante_quinta >= 0;  
var vigilante_sexta >= 0;  
var vigilante_sabado >= 0;  
var vigilante_domingo >= 0;  
```

## 2. Função Objetivo
Minimizar o número total de vigilantes:




```python
%%ampl_eval

minimize Total_Vigilantes: vigilante_segunda + vigilante_terca + vigilante_quarta + vigilante_quinta + vigilante_sexta + vigilante_sabado + vigilante_domingo;


```

## Restrições
Cada dia da semana deve ter a quantidade mínima de vigilantes de acordo com a demanda:


```python
%%ampl_eval

subject to Demanda_Segunda:
    vigilante_segunda  + vigilante_quinta + vigilante_sexta + vigilante_sabado  + vigilante_domingo >= 39;  

subject to Demanda_Terça:
    vigilante_segunda + vigilante_terca  + vigilante_sexta + vigilante_sabado + vigilante_domingo  >= 21;  

subject to Demanda_Quarta:
    vigilante_segunda + vigilante_terca + vigilante_quarta + vigilante_sabado  + vigilante_domingo >= 35;  

subject to Demanda_Quinta:
    vigilante_segunda + vigilante_terca + vigilante_quarta + vigilante_quinta  + vigilante_domingo  >= 30;  

subject to Demanda_Sexta:
    vigilante_segunda + vigilante_terca + vigilante_quarta  + vigilante_quinta + vigilante_sexta >= 28;  

subject to Demanda_Sábado:
    vigilante_terca  + vigilante_quarta  + vigilante_quinta + vigilante_sexta + vigilante_sabado >= 43;  

subject to Demanda_Domingo:
    vigilante_quarta  + vigilante_quinta + vigilante_sexta + vigilante_sabado  + vigilante_domingo >= 19;  


```

    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 49
    6 simplex iterations
    0 barrier iterations
    vigilante_segunda = 6
    vigilante_terca = 0
    vigilante_quarta = 10
    vigilante_quinta = 14
    vigilante_sexta = 0
    vigilante_sabado = 19
    vigilante_domingo = 0
    Total_Vigilantes = 49
    


## Resultados


```python
%%ampl_eval
option solver 'highs';  
solve;


display vigilante_segunda, vigilante_terca, vigilante_quarta, vigilante_quinta, vigilante_sexta, vigilante_sabado, vigilante_domingo,Total_Vigilantes;
```

Aqui está a saída formatada em uma tabela para facilitar a leitura dos resultados:

| Dia da Semana   | Vigilantes Necessários |
|-----------------|------------------------|
| Segunda-feira   | 6                      |
| Terça-feira     | 0                      |
| Quarta-feira    | 10                     |
| Quinta-feira    | 14                     |
| Sexta-feira     | 0                      |
| Sábado          | 19                     |
| Domingo         | 0                      |
| **Total**       | **49**                 |

Esse resultado mostra a distribuição dos 49 vigilantes em período integral para atender à demanda mínima em cada dia da semana.

# Parte B: Minimização de Custo com Vigilantes em Período Integral e Meio Período

Além de vigilantes em período integral, a VigilanceCo pode contratar vigilantes de meio período, que trabalham 4 horas por dia. A quantidade de vigilantes de meio período é limitada a 1/3 da demanda semanal.

## Definição das Variáveis

- Vigias integrais que começam a trabalhar na segunda, terça, quarta, quinta, sexta, sábado, domingo.
- Vigias meio perído que começam a trabalhar na segunda, terça, quarta, quinta, sexta, sábado, domingo.


## Função Objetivo

Minimizar o custo total:


`{Minimizar } Z = 30 * 8 * (x_1 + x_2 + x_3 + x_4 + x_5 + x_6 + x_7) + 20 * 4 * (y_1 + y_2 + y_3 + y_4 + y_5 + y_6 + y_7)`




##  Restrições

#### Demanda de Horas por Dia

Cada dia deve ter a demanda total de horas atendida:

- **Segunda-feira**:


`8(x_1 + x_4 + x_5 + x_6 + x_7) + 4(y_1 + y_4 + y_5 + y_6 + y_7) >= 312`


- **Terça-feira**:

`8(x_1 + x_2 + x_5 + x_6 + x_7) + 4(y_1 + y_2 + y_5 + y_6 + y_7) >= 168`


E assim sucessivamente para os outros dias da semana.

---

#### Limite de Meio Período

A soma das horas dos vigilantes de meio período deve ser no máximo 1/3 do total de horas semanais:


`4 * 5 * (y_1 + y_2 + y_3 + y_4 + y_5 + y_6 + y_7) <= 1/3 * 1720`

---

Essa é a formatação correta, com as equações matemáticas apresentadas de forma clara em Markdown.


```python
from amplpy import AMPL, Environment

def modelo_parte_b():
    ampl = AMPL(Environment())
    ampl.eval('''

    

var vigia_seg_int >= 0;  
var vigia_ter_int >= 0;  
var vigia_qua_int >= 0;  
var vigia_qui_int >= 0;  
var vigia_sext_int >= 0;  
var vigia_sab_int >= 0;  
var vigia_dom_int >= 0;  


var vigia_seg_meio >= 0;  
var vigia_ter_meio >= 0;  
var vigia_qua_meio >= 0;  
var vigia_qui_meio >= 0;  
var vigia_sext_meio >= 0;  
var vigia_sab_meio >= 0;  
var vigia_dom_meio >= 0; 


minimize Custo_Total_b: 
    30 * 8 * (vigia_seg_int + vigia_ter_int + vigia_qua_int + vigia_qui_int + vigia_sext_int + vigia_sab_int + vigia_dom_int) 
  + 20 * 4 * (vigia_seg_meio + vigia_ter_meio + vigia_qua_meio + vigia_qui_meio + vigia_sext_meio + vigia_sab_meio + vigia_dom_meio);


subject to Demanda_Segunda_b:
    (vigia_seg_int  + vigia_qui_int + vigia_sext_int + vigia_sab_int  + vigia_dom_int) * 8  + 
    (vigia_seg_meio  + vigia_qui_meio  + vigia_sext_meio  + vigia_sab_meio   + vigia_dom_meio) * 4  >= 312;  

subject to Demanda_Terça_b:
   ( vigia_seg_int + vigia_ter_int  + vigia_sext_int + vigia_sab_int + vigia_dom_int) * 8  + 
     (vigia_seg_meio + vigia_ter_meio  + vigia_sext_meio + vigia_sab_meio + vigia_dom_meio) * 4 >= 168;  

subject to Demanda_Quarta_b:
    (vigia_seg_int + vigia_ter_int + vigia_qua_int + vigia_sab_int  + vigia_dom_int) * 8  +
    (vigia_seg_meio + vigia_ter_meio + vigia_qua_meio + vigia_sab_meio  + vigia_dom_meio)  * 4 >= 280;  

subject to Demanda_Quinta_b:
   ( vigia_seg_int + vigia_ter_int + vigia_qua_int + vigia_qui_int  + vigia_dom_int) * 8  +
     (vigia_seg_meio + vigia_ter_meio + vigia_qua_meio + vigia_qui_meio  + vigia_dom_meio ) * 4 >= 240;  

subject to Demanda_Sexta_b:
    (vigia_seg_int + vigia_ter_int + vigia_qua_int  + vigia_qui_int + vigia_sext_int) * 8  + 
    (vigia_seg_meio + vigia_ter_meio + vigia_qua_meio  + vigia_qui_meio + vigia_sext_meio) * 4 >= 224;  

subject to Demanda_Sabado_b:
    (vigia_ter_int  + vigia_qua_int  + vigia_qui_int + vigia_sext_int + vigia_sab_int  ) * 8 + 
    (vigia_ter_meio  + vigia_qua_meio  + vigia_qui_meio + vigia_sext_meio + vigia_sab_meio) * 4  >= 344;  

subject to Demanda_Domingo_b:
    (vigia_qua_int  + vigia_qui_int + vigia_sext_int + vigia_sab_int  + vigia_dom_int) * 8  + 
    (vigia_qua_meio  + vigia_qui_meio + vigia_sext_meio + vigia_sab_meio + vigia_dom_meio ) * 4 >= 152;  


subject to Limite_Meio_Periodo_b: 
    4 * 5 * (vigia_seg_meio + vigia_ter_meio + vigia_qua_meio + vigia_qui_meio + vigia_sext_meio + vigia_sab_meio + vigia_dom_meio) <= 573.33;  


var Total_Vigilantes_Int = vigia_seg_int + vigia_ter_int + vigia_qua_int + vigia_qui_int + vigia_sext_int + vigia_sab_int + vigia_dom_int;
var Total_Vigilantes_Meio = vigia_seg_meio + vigia_ter_meio + vigia_qua_meio + vigia_qui_meio + vigia_sext_meio + vigia_sab_meio + vigia_dom_meio;


option solver 'highs';  
solve;






    ''')
    ampl.eval('display vigia_seg_int, vigia_ter_int, vigia_qua_int, vigia_qui_int, vigia_sext_int, vigia_sab_int, vigia_dom_int,vigia_seg_meio, vigia_ter_meio, vigia_qua_meio, vigia_qui_meio, vigia_sext_meio, vigia_sab_meio, vigia_dom_meio, Total_Vigilantes_Int, Total_Vigilantes_Meio, Custo_Total_b;')
    return ampl


ampl_c = modelo_parte_b()

```

    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 10613.34
    8 simplex iterations
    0 barrier iterations
    vigia_seg_int = 6
    vigia_ter_int = 0
    vigia_qua_int = 0
    vigia_qui_int = 9.66675
    vigia_sext_int = 0
    vigia_sab_int = 19
    vigia_dom_int = 0
    vigia_seg_meio = 0
    vigia_ter_meio = 20
    vigia_qua_meio = 0
    vigia_qui_meio = 8.6665
    vigia_sext_meio = 0
    vigia_sab_meio = 0
    vigia_dom_meio = 0
    Total_Vigilantes_Int = 34.6667
    Total_Vigilantes_Meio = 28.6665
    Custo_Total_b = 10613.3
    


## Resultados

Aqui está a saída formatada em uma tabela com os valores de vigilantes de período integral e meio período:

| Dia da Semana      | Vigilantes Período Integral | Vigilantes Meio Período |
|--------------------|-----------------------------|-------------------------|
| Segunda-feira      | 6                           | 0                       |
| Terça-feira        | 0                           | 20                      |
| Quarta-feira       | 0                           | 0                       |
| Quinta-feira       | 9.66675                     | 8.6665                  |
| Sexta-feira        | 0                           | 0                       |
| Sábado             | 19                          | 0                       |
| Domingo            | 0                           | 0                       |

| **Total Vigilantes** | **Período Integral** | **Período Meio** |
|----------------------|----------------------|------------------|
|                      | 34.6667              | 28.6665          |

| **Custo Total**       | **R\$ 10613.34**    |

Esses valores representam o número de vigilantes por dia, divididos em períodos integral e meio período, além do custo total da solução.

# Parte C: Maximização de Dias de Folga nos Finais de Semana

Neste cenário, a VigilanceCo já tem 49 vigilantes em período integral e deseja maximizar o número de dias de folga nos finais de semana (sábados e domingos)

## Definição das Variáveis

- Nessa parte optei por definir as variáveis como `escalas` de 5 dias onde cada funcionario comeca em um dia e trabalha por 5 dias consecutivos

## Função Objetivo
Maximizar o número de folgas aos finais de semana:

`Maximizar Folgas_fim_de_semana = (escala_1 + escala_7) + (escala_1 + escala_2);`  

Onde a `escala_1` é somada 2 vezes pois possui 2 dias de fim de semana.
- `escala_1` : Folga sábado e domingo
- `escala_2` : Folga domingo
- `escala_7` : Folga sábado



## Restrições

- Demanda de Vigilantes Diária: Cada dia da semana deve ter a quantidade mínima de vigilantes assim como nos casos anteriores

- Total de Vigilantes: A soma de todos os vigilantes alocados deve ser igual a 49: 

    `escala_1 + escala_2 + escala_3 + escala_4 + escala_5 + escala_6 + escala_7 = 49;`





```python
from amplpy import AMPL, Environment

def modelo_parte_c():
    ampl = AMPL(Environment())
    ampl.eval('''    
    var escala_1 >= 0,;  
    var escala_2 >= 0;  
    var escala_3 >= 0;  
    var escala_4 >= 0;  
    var escala_5 >= 0;  
    var escala_6 >= 0;  
    var escala_7 >= 0;  

    maximize Folgas_Finais_Semana:
        (escala_1 + escala_7) + (escala_1 + escala_2);  

    subject to Total_Vigilantes:
        escala_1 + escala_2 + escala_3 + escala_4 + escala_5 + escala_6 + escala_7 = 49;

    subject to Demanda_Segunda:
        escala_1 + escala_4 + escala_5 + escala_6 + escala_7 >= 39;  

    subject to Demanda_Terça:
        escala_1 + escala_2 + escala_5 + escala_6 + escala_7 >= 21;  

    subject to Demanda_Quarta:
        escala_1 + escala_2 + escala_3 + escala_6 + escala_7 >= 35;  

    subject to Demanda_Quinta:
        escala_1 + escala_2 + escala_3 + escala_4 + escala_7 >= 30;  

    subject to Demanda_Sexta:
        escala_1 + escala_2 + escala_3 + escala_4 + escala_5 >= 28;  

    subject to Demanda_Sabado:
        escala_2 + escala_3 + escala_4 + escala_5 + escala_6 >= 43;  

    subject to Demanda_Domingo:
        escala_3 + escala_4 + escala_5 + escala_6 + escala_7 >= 19;  

    option solver 'highs';
    solve;
    ''')
    ampl.eval('display escala_1, escala_2, escala_3, escala_4, escala_5, escala_6, escala_7;')
    return ampl


ampl_c = modelo_parte_c()

```

    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 22
    7 simplex iterations
    0 barrier iterations
    escala_1 = 6
    escala_2 = 10
    escala_3 = 0
    escala_4 = 14
    escala_5 = 0
    escala_6 = 19
    escala_7 = 0
    


## Resultados
Aqui está a tabela formatada com os valores da solução fornecida:

| Dia da Semana      | Vigilantes |
|--------------------|------------|
| Escala 1 (Segunda) | 6          |
| Escala 2 (Terça)   | 10         |
| Escala 3 (Quarta)  | 0          |
| Escala 4 (Quinta)  | 14         |
| Escala 5 (Sexta)   | 0          |
| Escala 6 (Sábado)  | 19         |
| Escala 7 (Domingo) | 0          |

| **Total Objetivo** | **22** |  

Essa tabela mostra o número de vigilantes alocados por dia da semana em cada escala.

# Resumo da Solução
- **Parte A:** Minimiza o número total de vigilantes em período integral contratados, considerando a necessidade de cinco dias consecutivos de trabalho.

- **Parte B:** Minimiza o custo, utilizando uma combinação de vigilantes em período integral e meio período, com uma limitação no número de horas dos vigilantes de meio período.

- **Parte C:** Maximiza o número de dias de folga nos finais de semana, respeitando o total de 49 vigilantes disponíveis e a demanda diária de cada dia da semana.


```python

```
#  Problema da Avela Corporation de produção de laptops

A Avela Corporation deseja minimizar os custos de produção e armazenamento de laptops ao longo de quatro trimestres. Para isso, a empresa pode optar por produzir os laptops usando trabalho em tempo regular (com capacidade limitada) ou em horas extras (com custo adicional). A empresa precisa garantir que a demanda trimestral seja atendida, mantendo um nível eficiente de estoque.


## Configurações do projeto


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
    


## Definição das Variáveis de Decisão



```python
%%ampl_eval

var Producao_1_reg >= 0, <= 4000;   
var Producao_2_reg >= 0, <= 4000;   
var Producao_3_reg >= 0, <= 4000;   
var Producao_4_reg >= 0, <= 4000;   

var Producao_1_extra >= 0;          
var Producao_2_extra >= 0;          
var Producao_3_extra >= 0;          
var Producao_4_extra >= 0;          

var Estoque_1 >= 0;                
var Estoque_2 >= 0;                
var Estoque_3 >= 0;                
var Estoque_4 >= 0;                


param Demanda_1 := 4000;           
param Demanda_2 := 6000;           
param Demanda_3 := 7500;           
param Demanda_4 := 2500;           

param Estoque_inicial := 1000;           
```

## Função Objetivo

A função objetivo é minimizar o custo total de produção e armazenamento:



```python
%%ampl_eval

minimize Custo_Total:
    4000 * (Producao_1_reg + Producao_2_reg + Producao_3_reg + Producao_4_reg) +
    4500 * (Producao_1_extra + Producao_2_extra + Producao_3_extra + Producao_4_extra) +
    200 * (Estoque_1 + Estoque_2 + Estoque_3 + Estoque_4);

```

## Restrições

As restrições garantem que a demanda de cada trimestre seja atendida e que o balanço de estoque seja respeitado:


```python
%%ampl_eval

subject to Balanço_1:
    Estoque_inicial + Producao_1_reg + Producao_1_extra = Demanda_1 + Estoque_1;

subject to Balanço_2:
    Estoque_1 + Producao_2_reg + Producao_2_extra = Demanda_2 + Estoque_2;

subject to Balanço_3:
    Estoque_2 + Producao_3_reg + Producao_3_extra = Demanda_3 + Estoque_3;

subject to Balanço_4:
    Estoque_3 + Producao_4_reg + Producao_4_extra = Demanda_4 + Estoque_4;


subject to Estoque_NonNeg:
    Estoque_4 >= 0;
```

## Resultados


```python
%%ampl_eval


option solver 'highs';  
solve;


display Producao_1_reg, Producao_2_reg, Producao_3_reg, Producao_4_reg, Producao_1_extra, Producao_2_extra, Producao_3_extra, Producao_4_extra, Estoque_1, Estoque_2, Estoque_3, Estoque_4, Custo_Total;

```

    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 78450000
    5 simplex iterations
    0 barrier iterations
    Producao_1_reg = 4000
    Producao_2_reg = 4000
    Producao_3_reg = 4000
    Producao_4_reg = 2500
    Producao_1_extra = 0
    Producao_2_extra = 1000
    Producao_3_extra = 3500
    Producao_4_extra = 0
    Estoque_1 = 1000
    Estoque_2 = 0
    Estoque_3 = 0
    Estoque_4 = 0
    Custo_Total = 78450000
    




### Tabela de Resultados

Abaixo está a tabela com a solução ótima obtida pela execução do modelo de programação linear:

| Variável             | Valor    |
|----------------------|----------|
| **Produção Regular no Trimestre 1** | 4000     |
| **Produção Regular no Trimestre 2** | 4000     |
| **Produção Regular no Trimestre 3** | 4000     |
| **Produção Regular no Trimestre 4** | 2500     |
| **Produção Extra no Trimestre 1**   | 0        |
| **Produção Extra no Trimestre 2**   | 1000     |
| **Produção Extra no Trimestre 3**   | 3500     |
| **Produção Extra no Trimestre 4**   | 0        |
| **Estoque ao Final do Trimestre 1** | 1000     |
| **Estoque ao Final do Trimestre 2** | 0        |
| **Estoque ao Final do Trimestre 3** | 0        |
| **Estoque ao Final do Trimestre 4** | 0        |
| **Custo Total**                     | 78,450,000 dólares |

### Observações

- A Avela Corporation conseguiu atender à demanda de cada trimestre produzindo laptops em tempo regular, complementando com produção em horas extras nos trimestres de maior demanda (trimestres 2 e 3).
- Houve um pequeno acúmulo de estoque ao final do primeiro trimestre (1000 laptops), que foi utilizado para atender a demanda nos trimestres seguintes.
- Não houve estoque ao final dos trimestres 2, 3 e 4, o que indica que a produção foi ajustada precisamente para atender à demanda e minimizar custos de armazenamento.
- O custo total de produção e armazenamento ao longo dos quatro trimestres foi de **78,450,000 dólares**.

## Análise de Sensibilidade

A análise de sensibilidade do problema da **Avela Corporation** revela como alterações em certos parâmetros impactam o planejamento de produção e os custos totais.

1. **Custo de Produção em Horas Extras**:
   - **Redução para $4200**: A empresa utilizaria mais horas extras, reduzindo o estoque e o custo total.
   - **Aumento para $4800**: A dependência de horas extras diminuiria, e o estoque aumentaria, resultando em maior custo total.

2. **Demanda no Trimestre 3**:
   - **Aumento para 8500 laptops**: Mais horas extras seriam necessárias, elevando os custos de produção no terceiro trimestre.

3. **Capacidade de Produção Regular**:
   - **Aumento para 4500 laptops**: Menor dependência de horas extras, resultando em menor custo total.
   - **Redução para 3500 laptops**: Aumento da necessidade de horas extras, elevando os custos totais.

4. **Custo de Armazenagem**:
   - **Aumento para $500 por laptop**: Estoque seria reduzido, com maior uso de horas extras, aumentando o custo total.



### Modelo reutilizável


```python
from amplpy import AMPL, tools


def run_model(cost_extra, demand_trim3, regular_capacity, storage_cost):
    ampl = tools.ampl_notebook(modules=["highs"], license_uuid="default", g=globals())  
    
    
    ampl.eval('''
    var Producao_1_reg >= 0, <= {reg_cap};   
    var Producao_2_reg >= 0, <= {reg_cap};   
    var Producao_3_reg >= 0, <= {reg_cap};   
    var Producao_4_reg >= 0, <= {reg_cap};   

    var Producao_1_extra >= 0;          
    var Producao_2_extra >= 0;          
    var Producao_3_extra >= 0;          
    var Producao_4_extra >= 0;          

    var Estoque_1 >= 0;                
    var Estoque_2 >= 0;                
    var Estoque_3 >= 0;                
    var Estoque_4 >= 0;                

    param Demanda_1 := 4000;           
    param Demanda_2 := 6000;           
    param Demanda_3 := {dem_trim3};           
    param Demanda_4 := 2500;           

    param Estoque_inicial := 1000; 

    
    minimize Custo_Total:
        4000 * (Producao_1_reg + Producao_2_reg + Producao_3_reg + Producao_4_reg) +
        {cost_extra} * (Producao_1_extra + Producao_2_extra + Producao_3_extra + Producao_4_extra) +
        {storage_cost} * (Estoque_1 + Estoque_2 + Estoque_3 + Estoque_4);

    
    subject to Balanco_1:
        Estoque_inicial + Producao_1_reg + Producao_1_extra = Demanda_1 + Estoque_1;

    subject to Balanco_2:
        Estoque_1 + Producao_2_reg + Producao_2_extra = Demanda_2 + Estoque_2;

    subject to Balanco_3:
        Estoque_2 + Producao_3_reg + Producao_3_extra = Demanda_3 + Estoque_3;

    subject to Balanco_4:
        Estoque_3 + Producao_4_reg + Producao_4_extra = Demanda_4 + Estoque_4;

    
    subject to Estoque_NonNeg:
        Estoque_4 >= 0;
    '''.format(cost_extra=cost_extra, dem_trim3=demand_trim3, reg_cap=regular_capacity, storage_cost=storage_cost))

    
    ampl.eval('option solver "highs"; solve;')

    
    ampl.eval('''
    display Producao_1_reg, Producao_2_reg, Producao_3_reg, Producao_4_reg;
    display Producao_1_extra, Producao_2_extra, Producao_3_extra, Producao_4_extra;
    display Estoque_1, Estoque_2, Estoque_3, Estoque_4, Custo_Total;
    ''')




```

## Diferentes casos


```python
print("Cenário 1: Redução do custo de produção em horas extras para $4200")
run_model(cost_extra=4200, demand_trim3=7500, regular_capacity=4000, storage_cost=200)


```

    Cenário 1: Redução do custo de produção em horas extras para $4200
    AMPL Development Version 20240606 (Linux-5.15.0-1064-azure, 64-bit)
    Demo license with maintenance expiring 20260131.
    Using license file "/home/sehmer/Documents/sync/EMC/PO1/teste_1/.venv/lib/python3.12/site-packages/ampl_module_base/bin/ampl.lic".
    
    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 77100000
    6 simplex iterations
    0 barrier iterations
    Producao_1_reg = 3000
    Producao_2_reg = 4000
    Producao_3_reg = 4000
    Producao_4_reg = 2500
    
    Producao_1_extra = 0
    Producao_2_extra = 2000
    Producao_3_extra = 3500
    Producao_4_extra = 0
    
    Estoque_1 = 0
    Estoque_2 = 0
    Estoque_3 = 0
    Estoque_4 = 0
    Custo_Total = 77100000
    



```python
print("\nCenário 2: Aumento do custo de produção em horas extras para $4800")
run_model(cost_extra=4800, demand_trim3=7500, regular_capacity=4000, storage_cost=200)


```

    
    Cenário 2: Aumento do custo de produção em horas extras para $4800
    AMPL Development Version 20240606 (Linux-5.15.0-1064-azure, 64-bit)
    Demo license with maintenance expiring 20260131.
    Using license file "/home/sehmer/Documents/sync/EMC/PO1/teste_1/.venv/lib/python3.12/site-packages/ampl_module_base/bin/ampl.lic".
    
    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 79800000
    5 simplex iterations
    0 barrier iterations
    Producao_1_reg = 4000
    Producao_2_reg = 4000
    Producao_3_reg = 4000
    Producao_4_reg = 2500
    
    Producao_1_extra = 0
    Producao_2_extra = 1000
    Producao_3_extra = 3500
    Producao_4_extra = 0
    
    Estoque_1 = 1000
    Estoque_2 = 0
    Estoque_3 = 0
    Estoque_4 = 0
    Custo_Total = 79800000
    



```python
print("\nCenário 3: Aumento da demanda no trimestre 3 para 8500 laptops")
run_model(cost_extra=4500, demand_trim3=8500, regular_capacity=4000, storage_cost=200)

```

    
    Cenário 3: Aumento da demanda no trimestre 3 para 8500 laptops
    AMPL Development Version 20240606 (Linux-5.15.0-1064-azure, 64-bit)
    Demo license with maintenance expiring 20260131.
    Using license file "/home/sehmer/Documents/sync/EMC/PO1/teste_1/.venv/lib/python3.12/site-packages/ampl_module_base/bin/ampl.lic".
    
    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 82950000
    5 simplex iterations
    0 barrier iterations
    Producao_1_reg = 4000
    Producao_2_reg = 4000
    Producao_3_reg = 4000
    Producao_4_reg = 2500
    
    Producao_1_extra = 0
    Producao_2_extra = 1000
    Producao_3_extra = 4500
    Producao_4_extra = 0
    
    Estoque_1 = 1000
    Estoque_2 = 0
    Estoque_3 = 0
    Estoque_4 = 0
    Custo_Total = 82950000
    



```python

print("\nCenário 4: Aumento da capacidade de produção regular para 4500 laptops")
run_model(cost_extra=4500, demand_trim3=7500, regular_capacity=4500, storage_cost=200)

```

    
    Cenário 4: Aumento da capacidade de produção regular para 4500 laptops
    AMPL Development Version 20240606 (Linux-5.15.0-1064-azure, 64-bit)
    Demo license with maintenance expiring 20260131.
    Using license file "/home/sehmer/Documents/sync/EMC/PO1/teste_1/.venv/lib/python3.12/site-packages/ampl_module_base/bin/ampl.lic".
    
    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 77800000
    4 simplex iterations
    0 barrier iterations
    Producao_1_reg = 4500
    Producao_2_reg = 4500
    Producao_3_reg = 4500
    Producao_4_reg = 2500
    
    Producao_1_extra = 0
    Producao_2_extra = 0
    Producao_3_extra = 3000
    Producao_4_extra = 0
    
    Estoque_1 = 1500
    Estoque_2 = 0
    Estoque_3 = 0
    Estoque_4 = 0
    Custo_Total = 77800000
    



```python

print("\nCenário 5: Redução da capacidade de produção regular para 3500 laptops")
run_model(cost_extra=4500, demand_trim3=7500, regular_capacity=3500, storage_cost=200)

```

    
    Cenário 5: Redução da capacidade de produção regular para 3500 laptops
    AMPL Development Version 20240606 (Linux-5.15.0-1064-azure, 64-bit)
    Demo license with maintenance expiring 20260131.
    Using license file "/home/sehmer/Documents/sync/EMC/PO1/teste_1/.venv/lib/python3.12/site-packages/ampl_module_base/bin/ampl.lic".
    
    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 79100000
    5 simplex iterations
    0 barrier iterations
    Producao_1_reg = 3500
    Producao_2_reg = 3500
    Producao_3_reg = 3500
    Producao_4_reg = 2500
    
    Producao_1_extra = 0
    Producao_2_extra = 2000
    Producao_3_extra = 4000
    Producao_4_extra = 0
    
    Estoque_1 = 500
    Estoque_2 = 0
    Estoque_3 = 0
    Estoque_4 = 0
    Custo_Total = 79100000
    



```python

print("\nCenário 6: Aumento do custo de armazenagem para $300 por laptop")
run_model(cost_extra=4500, demand_trim3=7500, regular_capacity=4000, storage_cost=500)
```

    
    Cenário 6: Aumento do custo de armazenagem para $300 por laptop
    AMPL Development Version 20240606 (Linux-5.15.0-1064-azure, 64-bit)
    Demo license with maintenance expiring 20260131.
    Using license file "/home/sehmer/Documents/sync/EMC/PO1/teste_1/.venv/lib/python3.12/site-packages/ampl_module_base/bin/ampl.lic".
    
    HiGHS 1.7.1:HiGHS 1.7.1: optimal solution; objective 78750000
    6 simplex iterations
    0 barrier iterations
    Producao_1_reg = 3000
    Producao_2_reg = 4000
    Producao_3_reg = 4000
    Producao_4_reg = 2500
    
    Producao_1_extra = 0
    Producao_2_extra = 2000
    Producao_3_extra = 3500
    Producao_4_extra = 0
    
    Estoque_1 = 0
    Estoque_2 = 0
    Estoque_3 = 0
    Estoque_4 = 0
    Custo_Total = 78750000
    



##  **Resumo da Análise de Sensibilidade**

| Parâmetro Alterado                  | Mudança                      | Impacto na Solução Ótima                          |
|-------------------------------------|------------------------------|--------------------------------------------------|
| Custo de Produção em Horas Extras   | Redução para $4200            | Aumento da produção em horas extras, menos estoque |
| Custo de Produção em Horas Extras   | Aumento para $4800            | Redução da produção em horas extras, mais estoque |
| Demanda no Trimestre 3              | Aumento para 8500 laptops     | Maior uso de horas extras, aumento do custo total |
| Capacidade de Produção Regular      | Aumento para 4500 laptops     | Redução do uso de horas extras, menor custo total |
| Capacidade de Produção Regular      | Redução para 3500 laptops     | Maior dependência de horas extras, aumento do custo total |
| Custo de Armazenagem                | Aumento para $500 por laptop  | Redução de estoque, maior uso de horas extras     |


### Conclusão
Mudanças nos custos de horas extras, demanda e capacidade de produção têm um impacto significativo no planejamento e nos custos. A Avela deve ajustar suas decisões conforme as condições de produção ou demanda mudam para minimizar os custos.
