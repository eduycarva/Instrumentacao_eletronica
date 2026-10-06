# Etapa 1 — Cálculos Fundamentais e Escada Resistiva

## 1.1 Cálculos Fundamentais

### Dados do Projeto

| Parâmetro | Símbolo | Valor |
|---|---|---|
| Resolução | N | **3 bits** |
| Tensão de referência | Vref | **5V** |

### Cálculos Derivados

**Número de níveis de quantização:**
```
Níveis = 2ᴺ = 2³ = 8 níveis (de 000 a 111)
```

**Valor do LSB (Least Significant Bit):**
```
LSB = Vref / 2ᴺ
LSB = 5V / 8
LSB = 0,625V
```

> [!IMPORTANT]
> O LSB = **0,625V** é o "degrau" do conversor. Cada vez que a tensão de entrada sobe 0,625V, a saída digital incrementa em 1 nível.

**Número de comparadores:**
```
Comparadores = 2ᴺ − 1 = 8 − 1 = 7 comparadores
```

**Número de resistores na escada:**
```
Resistores = 2ᴺ = 8 resistores
```

**Faixa de entrada:**
```
Vin mínimo = 0V        →  Saída: 000 (decimal 0)
Vin máximo = 5V         →  Saída: 111 (decimal 7)
```

**Tabela de transição:**

| Saída Digital | Decimal | Vin mínimo (limiar) | Vin máximo |
|---|---|---|---|
| 000 | 0 | 0,000V | 0,624V |
| 001 | 1 | 0,625V | 1,249V |
| 010 | 2 | 1,250V | 1,874V |
| 011 | 3 | 1,875V | 2,499V |
| 100 | 4 | 2,500V | 3,124V |
| 101 | 5 | 3,125V | 3,749V |
| 110 | 6 | 3,750V | 4,374V |
| 111 | 7 | 4,375V | 5,000V |

---

## 1.2 Projeto da Escada Resistiva

### Princípio de Funcionamento

A escada resistiva é um **divisor de tensão em cadeia**. Colocamos 8 resistores de **mesmo valor** em série entre Vref (5V) e GND. Nos 7 nós intermediários, obtemos as tensões de referência para os comparadores.

### Escolha do Valor dos Resistores

```
R = 1kΩ (para todos os 8 resistores)
```

> [!NOTE]
> Usamos 1kΩ por ser um valor comum e resultar em corrente total razoável:
> ```
> I_total = Vref / (8 × R) = 5V / 8kΩ = 0,625 mA
> ```
> Corrente baixa o suficiente para não desperdiçar energia, mas alta o suficiente para que os comparadores não perturbem as tensões de referência.

### Tensões de Referência nos Nós

A tensão em cada nó é calculada por divisão de tensão:

```
Vref_k = k × (Vref / 8)    onde k = 1, 2, 3, ..., 7
```

| Nó (k) | Comparador | Cálculo | Tensão de Referência |
|---|---|---|---|
| 1 | C1 | 1 × (5/8) | **0,625V** |
| 2 | C2 | 2 × (5/8) | **1,250V** |
| 3 | C3 | 3 × (5/8) | **1,875V** |
| 4 | C4 | 4 × (5/8) | **2,500V** |
| 5 | C5 | 5 × (5/8) | **3,125V** |
| 6 | C6 | 6 × (5/8) | **3,750V** |
| 7 | C7 | 7 × (5/8) | **4,375V** |

### Diagrama do Circuito

```
    Vref = 5V
      │
      │
    ┌─┴─┐
    │R8 │ 1kΩ
    └─┬─┘
      ├─────────── Nó 7: Vref_C7 = 4,375V ──→ Entrada (−) de C7
      │
    ┌─┴─┐
    │R7 │ 1kΩ
    └─┬─┘
      ├─────────── Nó 6: Vref_C6 = 3,750V ──→ Entrada (−) de C6
      │
    ┌─┴─┐
    │R6 │ 1kΩ
    └─┬─┘
      ├─────────── Nó 5: Vref_C5 = 3,125V ──→ Entrada (−) de C5
      │
    ┌─┴─┐
    │R5 │ 1kΩ
    └─┬─┘
      ├─────────── Nó 4: Vref_C4 = 2,500V ──→ Entrada (−) de C4
      │
    ┌─┴─┐
    │R4 │ 1kΩ
    └─┬─┘
      ├─────────── Nó 3: Vref_C3 = 1,875V ──→ Entrada (−) de C3
      │
    ┌─┴─┐
    │R3 │ 1kΩ
    └─┬─┘
      ├─────────── Nó 2: Vref_C2 = 1,250V ──→ Entrada (−) de C2
      │
    ┌─┴─┐
    │R2 │ 1kΩ
    └─┬─┘
      ├─────────── Nó 1: Vref_C1 = 0,625V ──→ Entrada (−) de C1
      │
    ┌─┴─┐
    │R1 │ 1kΩ
    └─┬─┘
      │
     GND
```

### Verificação dos Cálculos (Divisão de Tensão)

Para confirmar, usamos a fórmula de divisão de tensão para o nó k (contando de baixo para cima, do GND para Vref):

```
V_nó_k = Vref × (k / 8)
```

**Verificação ponto a ponto:**
```
V_nó_1 = 5V × (1/8) = 5V × 0,125  = 0,625V  ✅
V_nó_2 = 5V × (2/8) = 5V × 0,250  = 1,250V  ✅
V_nó_3 = 5V × (3/8) = 5V × 0,375  = 1,875V  ✅
V_nó_4 = 5V × (4/8) = 5V × 0,500  = 2,500V  ✅
V_nó_5 = 5V × (5/8) = 5V × 0,625  = 3,125V  ✅
V_nó_6 = 5V × (6/8) = 5V × 0,750  = 3,750V  ✅
V_nó_7 = 5V × (7/8) = 5V × 0,875  = 4,375V  ✅
```

**Potência total dissipada na escada:**
```
P_total = Vref² / R_total = (5V)² / 8kΩ = 25 / 8000 = 3,125 mW
```

> [!TIP]
> Potência muito baixa — sem problemas térmicos. Na prática, resistores de 1/4W são mais que suficientes.

---

## 1.3 Arquivo LTspice — Escada Resistiva

Abaixo está a **netlist SPICE** para você simular somente a escada resistiva no LTspice e verificar as tensões nos nós.

> [!NOTE]
> **Como usar:** Salve o conteúdo abaixo como `escada_resistiva.cir` e abra no LTspice. Rode a simulação `.op` (ponto de operação DC) para ver as tensões nos 7 nós.

```spice
* ============================================
* Escada Resistiva - Conversor A/D Flash 3 bits
* Vref = 5V, 8 resistores de 1kΩ
* ============================================

* Fonte de referência
Vref vref 0 5V

* Escada resistiva (de Vref até GND)
* R8: entre Vref e nó 7
R8 vref  node7  1k
* R7: entre nó 7 e nó 6
R7 node7  node6  1k
* R6: entre nó 6 e nó 5
R6 node6  node5  1k
* R5: entre nó 5 e nó 4
R5 node5  node4  1k
* R4: entre nó 4 e nó 3
R4 node4  node3  1k
* R3: entre nó 3 e nó 2
R3 node3  node2  1k
* R2: entre nó 2 e nó 1
R2 node2  node1  1k
* R1: entre nó 1 e GND
R1 node1  0      1k

* Análise DC (ponto de operação)
.op

* Exibir tensões nos nós
.print DC V(node1) V(node2) V(node3) V(node4) V(node5) V(node6) V(node7)

.end
```

**Resultados esperados na simulação:**
```
V(node1) = 0.625V   ← Ref para C1
V(node2) = 1.250V   ← Ref para C2
V(node3) = 1.875V   ← Ref para C3
V(node4) = 2.500V   ← Ref para C4
V(node5) = 3.125V   ← Ref para C5
V(node6) = 3.750V   ← Ref para C6
V(node7) = 4.375V   ← Ref para C7
```

---

## 1.4 Checklist de Validação — Etapa 1

- [ ] LSB calculado = 0,625V
- [ ] 8 resistores de 1kΩ na escada
- [ ] 7 tensões de referência conferidas
- [ ] Simulação no LTspice confirma os valores dos nós
- [ ] Screenshots da simulação capturados para o relatório

---

## Próxima Etapa

> **Etapa 2 — Montagem dos 7 Comparadores**
> - Adicionar os 7 comparadores ao circuito
> - Conectar Vin na entrada (+) de todos
> - Conectar cada tensão de referência na entrada (−)
> - Simular com rampa de tensão para ver o código termômetro

**Aguardando sua aprovação para prosseguir com a Etapa 2!** 🚀
