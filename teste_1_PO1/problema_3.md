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
