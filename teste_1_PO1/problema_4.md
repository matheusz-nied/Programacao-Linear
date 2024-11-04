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
