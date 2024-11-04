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
