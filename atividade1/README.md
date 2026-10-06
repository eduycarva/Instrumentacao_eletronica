# Projeto de Conversor Analógico-Digital (ADC) Flash de 3 Bits

Projeto e validação em simulação SPICE de um Conversor Analógico-Digital (ADC) do tipo Flash com resolução de 3 bits, desenvolvido com amplificadores operacionais LM741 e lógica combinacional de codificação.

---

## 1. Visão Geral do Sistema

O conversor Flash realiza a quantização paralela instantânea de um sinal analógico contínuo $V_{in}$ na faixa de $0\text{ V}$ a $5\text{ V}$, produzindo uma palavra digital de 3 bits ($B_2 B_1 B_0$).

```
                 +-------------------+
Vin (0 a 5V) --->| Banco de          |---> Código Termômetro --->+----------------+---> B2 (MSB)
                 | 7 Comparadores    |     (X1 a X7)             | Codificador de |---> B1
Vref (5V)   --->| (LM741)           |                           | Prioridade     |---> B0 (LSB)
(Escada R)       +-------------------+                           +----------------+
```

### Especificações Técnicas

| Parâmetro | Descrição | Valor / Expressão |
| :--- | :--- | :--- |
| Resolução ($N$) | Número de bits de saída | $3\text{ bits}$ |
| Níveis de Quantização | Estados discretos ($2^N$) | $8\text{ níveis}$ ($000_2\text{ a }111_2$) |
| Tensão de Referência ($V_{ref}$) | Fundo de escala analógico | $5{,}000\text{ V}$ |
| Resolução de Tensão ($V_{LSB}$) | Degrau de quantização ($V_{ref} / 2^N$) | $0{,}625\text{ V}$ ($625\text{ mV}$) |
| Alimentação ($V_{cc} / V_{ee}$) | Barramentos de polarização dos amp-ops | $+15\text{ V} / -15\text{ V}$ |
| Quantidade de Comparadores | $2^N - 1$ | $7\text{ unidades}$ |
| Resistores da Escada | $2^N$ | $8\text{ resistores de }1\text{ k}\Omega$ |

---

## 2. Diagrama Esquemático

O circuito completo foi sintetizado no LTspice com o amplificador operacional comercial LM741:

![Diagrama Esquemático do Conversor ADC Flash](schematic_versaofinal.png)

### Blocos Funcionais do Circuito:
1. **Escada Resistiva (R-Ladder):** Divisor com 8 resistores de $1\text{ k}\Omega$ associados a um resistor de cabeceira $R_{top} = 16\text{ k}\Omega$ conectado a $+15\text{ V}$, derivando a referência exata de $5\text{ V}$ e os 7 limiares intermediários ($0{,}625\text{ V}$, $1{,}250\text{ V}$, $1{,}875\text{ V}$, $2{,}500\text{ V}$, $3{,}125\text{ V}$, $3{,}750\text{ V}$, $4{,}375\text{ V}$).
2. **Comparadores em Malha Aberta:** 7 unidades do LM741 recebem o sinal analógico comum $V_{in}$ nas entradas não-inversoras e comparam com seus respectivos limiares, gerando as linhas de código termômetro $X_1$ a $X_7$.
3. **Codificador Lógico:** Fontes de tensão comportamentais arbitrárias (`bv`) sintetizam a tabela verdade reduzida por mapas de Karnaugh para produzir as saídas binárias TTL ($0\text{ V}$ / $5\text{ V}$):
   - $B_2 = X_4$
   - $B_1 = X_2 \cdot \overline{X_4} + X_6$
   - $B_0 = X_1 \cdot \overline{X_2} + X_3 \cdot \overline{X_4} + X_5 \cdot \overline{X_6} + X_7$

---

## 3. Tabela Verdade de Quantização

| Faixa de Entrada $V_{in}$ | Código Termômetro ($X_7 \dots X_1$) | $B_2$ | $B_1$ | $B_0$ | Decimal |
| :---: | :---: | :---: | :---: | :---: | :---: |
| $0{,}000\text{ V} \le V_{in} < 0{,}625\text{ V}$ | `0000000` | 0 | 0 | 0 | 0 |
| $0{,}625\text{ V} \le V_{in} < 1{,}250\text{ V}$ | `0000001` | 0 | 0 | 1 | 1 |
| $1{,}250\text{ V} \le V_{in} < 1{,}875\text{ V}$ | `0000011` | 0 | 1 | 0 | 2 |
| $1{,}875\text{ V} \le V_{in} < 2{,}500\text{ V}$ | `0000111` | 0 | 1 | 1 | 3 |
| $2{,}500\text{ V} \le V_{in} < 3{,}125\text{ V}$ | `0001111` | 1 | 0 | 0 | 4 |
| $3{,}125\text{ V} \le V_{in} < 3{,}750\text{ V}$ | `0011111` | 1 | 0 | 1 | 5 |
| $3{,}750\text{ V} \le V_{in} < 4{,}375\text{ V}$ | `0111111` | 1 | 1 | 0 | 6 |
| $4{,}375\text{ V} \le V_{in} \le 5{,}000\text{ V}$ | `1111111` | 1 | 1 | 1 | 7 |

---

## 4. Resultados de Simulação

A simulação transiente no LTspice aplica uma rampa linear de tensão na entrada ($V_{in}$ variando de $0\text{ V}$ a $5\text{ V}$ em $1\text{ ms}$), validando a transição sequencial dos bits $B_2$, $B_1$ e $B_0$ em todos os 8 patamares de quantização:

![Formas de Onda da Simulação dos 8 Níveis](graficos_versaofinal.png)

A resposta comprova comutação límpida nos limiares teóricos, sem estados transitórios espúrios na palavra digital de saída.

---

## 5. Estrutura do Diretório

```text
atividade1/
├── .gitignore                  # Exclusão de arquivos binários e temporários do LTspice
├── 741.lib                     # Modelo SPICE do amplificador LM741
├── lm741_schematic.asc         # Esquemático principal no LTspice
├── schematic_versaofinal.png   # Imagem do circuito esquemático
├── graficos_versaofinal.png    # Imagem das formas de onda da simulação
├── README.md                   # Documentação técnica do conversor
├── lib/                        # Símbolos customizados necessários (LM741, voltage, bv)
└── docs/                       # Memórias de cálculo e relatórios analíticos
```

---

## 6. Instruções para Execução

1. Abra o arquivo `lm741_schematic.asc` no LTspice.
2. Certifique-se de manter o arquivo `741.lib` e a pasta `lib/` no mesmo diretório.
3. Clique em **Simulate -> Run**.
4. Visualize as formas de onda das saídas digitais `V(b2)`, `V(b1)` e `V(b0)`.
