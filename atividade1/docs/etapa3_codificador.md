# Etapa 3 — Codificador Termômetro → Binário

## 3.1 O Problema

Temos 7 saídas de comparadores (código termômetro) e precisamos convertê-las em **3 bits binários** (B2, B1, B0). É uma função lógica combinacional.

### Tabela de Conversão (relembrando)

| Nível | C7 | C6 | C5 | C4 | C3 | C2 | C1 | → | B2 | B1 | B0 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | → | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | → | 0 | 0 | 1 |
| 2 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | → | 0 | 1 | 0 |
| 3 | 0 | 0 | 0 | 0 | 1 | 1 | 1 | → | 0 | 1 | 1 |
| 4 | 0 | 0 | 0 | 1 | 1 | 1 | 1 | → | 1 | 0 | 0 |
| 5 | 0 | 0 | 1 | 1 | 1 | 1 | 1 | → | 1 | 0 | 1 |
| 6 | 0 | 1 | 1 | 1 | 1 | 1 | 1 | → | 1 | 1 | 0 |
| 7 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | → | 1 | 1 | 1 |

> [!NOTE]
> Das 2⁷ = 128 combinações possíveis de C1–C7, apenas **8 são válidas** (código termômetro puro). Todas as outras são "don't care" (×), o que facilita muito a simplificação.

---

## 3.2 Derivação das Equações Booleanas

### B2 (bit mais significativo)

**Pergunta:** Quando B2 = 1?

| Nível | B2 | C4 |
|---|---|---|
| 0 | 0 | 0 |
| 1 | 0 | 0 |
| 2 | 0 | 0 |
| 3 | 0 | 0 |
| 4 | **1** | **1** |
| 5 | **1** | **1** |
| 6 | **1** | **1** |
| 7 | **1** | **1** |

**Observação:** B2 = 1 exatamente quando **C4 = 1** (4 ou mais comparadores ativos).

```
┌─────────────────────────┐
│  B2 = C4                │
└─────────────────────────┘
```

> Implementação: **apenas um fio!** A saída de C4 é diretamente B2.

---

### B1 (bit intermediário)

**Pergunta:** Quando B1 = 1?

| Nível | B1 | C2 | C4 | C6 | Padrão |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | — |
| 1 | 0 | 0 | 0 | 0 | — |
| 2 | **1** | **1** | 0 | 0 | C2=1 e C4=0 |
| 3 | **1** | **1** | 0 | 0 | C2=1 e C4=0 |
| 4 | 0 | 1 | 1 | 0 | — |
| 5 | 0 | 1 | 1 | 0 | — |
| 6 | **1** | 1 | 1 | **1** | C6=1 |
| 7 | **1** | 1 | 1 | **1** | C6=1 |

**Observação:** B1 = 1 em duas situações:
1. **C2 está ativo E C4 ainda não** (níveis 2 e 3)
2. **C6 está ativo** (níveis 6 e 7)

```
┌─────────────────────────┐
│  B1 = C2·C̄4 + C6        │
└─────────────────────────┘
```

**Leitura:** "B1 é alto quando C2 está ligado mas C4 não, OU quando C6 está ligado."

**Portas necessárias:** 1× NOT, 1× AND, 1× OR

---

### B0 (bit menos significativo)

**Pergunta:** Quando B0 = 1?

| Nível | B0 | Padrão (quais Ck "acendem" neste nível) |
|---|---|---|
| 0 | 0 | Nenhum novo |
| 1 | **1** | C1 acende (C1=1, C2 ainda=0) |
| 2 | 0 | C2 acende |
| 3 | **1** | C3 acende (C3=1, C4 ainda=0) |
| 4 | 0 | C4 acende |
| 5 | **1** | C5 acende (C5=1, C6 ainda=0) |
| 6 | 0 | C6 acende |
| 7 | **1** | C7 acende (é o último) |

**Observação:** B0 = 1 nos níveis **ímpares**. O padrão é: um comparador ímpar está ativo mas o próximo par ainda não ligou, **ou** todos estão ligados (C7).

```
┌──────────────────────────────────────────┐
│  B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7      │
└──────────────────────────────────────────┘
```

**Leitura:** "B0 é alto quando C1 está ligado mas C2 não, OU C3 ligado mas C4 não, OU C5 ligado mas C6 não, OU C7 está ligado."

**Portas necessárias:** 3× NOT, 3× AND, 1× OR (3 entradas) + 1 entrada direta (C7)

---

## 3.3 Resumo das Equações

```
╔══════════════════════════════════════════════╗
║                                              ║
║   B2 = C4                                   ║
║                                              ║
║   B1 = C2·C̄4 + C6                           ║
║                                              ║
║   B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7         ║
║                                              ║
╚══════════════════════════════════════════════╝
```

---

## 3.4 Verificação Exaustiva

Vamos conferir **todos os 8 níveis** com as equações:

### Nível 0 — Vin < 0,625V → C1..C7 = 0000000
```
B2 = C4 = 0                                    ✅
B1 = C2·C̄4 + C6 = 0·1 + 0 = 0                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 0·1 + 0·1 + 0·1 + 0 = 0                   ✅
Resultado: 000 ✅
```

### Nível 1 — C1..C7 = 0000001
```
B2 = C4 = 0                                    ✅
B1 = C2·C̄4 + C6 = 0·1 + 0 = 0                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 1·1 + 0·1 + 0·1 + 0 = 1                   ✅
Resultado: 001 ✅
```

### Nível 2 — C1..C7 = 0000011
```
B2 = C4 = 0                                    ✅
B1 = C2·C̄4 + C6 = 1·1 + 0 = 1                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 1·0 + 0·1 + 0·1 + 0 = 0                   ✅
Resultado: 010 ✅
```

### Nível 3 — C1..C7 = 0000111
```
B2 = C4 = 0                                    ✅
B1 = C2·C̄4 + C6 = 1·1 + 0 = 1                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 1·0 + 1·1 + 0·1 + 0 = 1                   ✅
Resultado: 011 ✅
```

### Nível 4 — C1..C7 = 0001111
```
B2 = C4 = 1                                    ✅
B1 = C2·C̄4 + C6 = 1·0 + 0 = 0                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 1·0 + 1·0 + 0·1 + 0 = 0                   ✅
Resultado: 100 ✅
```

### Nível 5 — C1..C7 = 0011111
```
B2 = C4 = 1                                    ✅
B1 = C2·C̄4 + C6 = 1·0 + 0 = 0                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 1·0 + 1·0 + 1·1 + 0 = 1                   ✅
Resultado: 101 ✅
```

### Nível 6 — C1..C7 = 0111111
```
B2 = C4 = 1                                    ✅
B1 = C2·C̄4 + C6 = 1·0 + 1 = 1                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 1·0 + 1·0 + 1·0 + 0 = 0                   ✅
Resultado: 110 ✅
```

### Nível 7 — C1..C7 = 1111111
```
B2 = C4 = 1                                    ✅
B1 = C2·C̄4 + C6 = 1·0 + 1 = 1                 ✅
B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
   = 1·0 + 1·0 + 1·0 + 1 = 1                   ✅
Resultado: 111 ✅
```

> [!IMPORTANT]
> **8/8 níveis conferidos! As equações estão 100% corretas.** ✅

---

## 3.5 Diagrama de Portas Lógicas

```
               CODIFICADOR TERMÔMETRO → BINÁRIO
    ═══════════════════════════════════════════════

    ┌─── B2 (MSB) ─────────────────────────────┐
    │                                           │
    │   C4 ────────────────────────────→ B2     │
    │                                           │
    └───────────────────────────────────────────┘

    ┌─── B1 ────────────────────────────────────┐
    │                                           │
    │   C4 ──→[NOT]──→┐                         │
    │                  ├──→[AND]──→┐             │
    │   C2 ───────────→┘           ├──→[OR]──→ B1
    │                              │             │
    │   C6 ────────────────────────┘             │
    │                                           │
    └───────────────────────────────────────────┘

    ┌─── B0 (LSB) ──────────────────────────────┐
    │                                           │
    │   C2 ──→[NOT]──→┐                         │
    │                  ├──→[AND]──→┐             │
    │   C1 ───────────→┘           │             │
    │                              │             │
    │   C4 ──→[NOT]──→┐           │             │
    │                  ├──→[AND]──→├──→[OR]──→ B0
    │   C3 ───────────→┘           │    (4 ent) │
    │                              │             │
    │   C6 ──→[NOT]──→┐           │             │
    │                  ├──→[AND]──→┘             │
    │   C5 ───────────→┘                         │
    │                              │             │
    │   C7 ────────────────────────┘             │
    │                                           │
    └───────────────────────────────────────────┘
```

### Lista de Portas Necessárias

| Porta | Quantidade | Uso |
|---|---|---|
| NOT | 4 | Inverter C̄2, C̄4 (×2), C̄6 |
| AND (2 entradas) | 4 | C2·C̄4, C1·C̄2, C3·C̄4, C5·C̄6 |
| OR (2 entradas) | 1 | Para B1 |
| OR (4 entradas) | 1 | Para B0 (ou 2× OR de 2 entradas em cascata) |

### CIs TTL/CMOS Equivalentes (se montar em hardware)

| CI | Função | Qtd necessária |
|---|---|---|
| **74HC04** | 6× NOT (Hex Inverter) | 1 chip (usa 4 de 6) |
| **74HC08** | 4× AND de 2 entradas | 1 chip (usa 4 de 4) |
| **74HC32** | 4× OR de 2 entradas | 1 chip (usa 3 de 4)* |

*Para B0: usa 2× OR em cascata → (C1·C̄2 OR C3·C̄4) OR (C5·C̄6 OR C7) = 3 portas OR

---

## 3.6 Netlist LTspice — ADC Flash Completo (3 bits)

Salve como **`adc_flash_3bits_completo.cir`** — este é o **circuito final completo**.

```spice
* ================================================================
* CONVERSOR A/D FLASH DE 3 BITS - CIRCUITO COMPLETO
* Vref = 5V | N = 3 bits | LSB = 0,625V
*
* Blocos:
*   1. Escada resistiva (8 x 1kΩ)
*   2. 7 Comparadores ideais
*   3. Codificador termômetro → binário
* ================================================================

* ==== FONTES ====

* Fonte de referência
Vref vref 0 5V

* Sinal de entrada: rampa de 0V a 5V em 10ms
Vin vin 0 PWL(0 0 10m 5)

* ==== BLOCO 1: ESCADA RESISTIVA (8 x 1kΩ) ====

R8 vref   node7  1k
R7 node7  node6  1k
R6 node6  node5  1k
R5 node5  node4  1k
R4 node4  node3  1k
R3 node3  node2  1k
R2 node2  node1  1k
R1 node1  0      1k

* ==== BLOCO 2: 7 COMPARADORES IDEAIS ====
* Saída = 5V se Vin > Vref_nó, senão 0V

B_C1 c1 0 V = if(V(vin) > V(node1), 5, 0)
B_C2 c2 0 V = if(V(vin) > V(node2), 5, 0)
B_C3 c3 0 V = if(V(vin) > V(node3), 5, 0)
B_C4 c4 0 V = if(V(vin) > V(node4), 5, 0)
B_C5 c5 0 V = if(V(vin) > V(node5), 5, 0)
B_C6 c6 0 V = if(V(vin) > V(node6), 5, 0)
B_C7 c7 0 V = if(V(vin) > V(node7), 5, 0)

* ==== BLOCO 3: CODIFICADOR (TERMÔMETRO → BINÁRIO) ====

* B2 = C4
B_B2 b2 0 V = if(V(c4) > 2.5, 5, 0)

* B1 = C2·(NOT C4) + C6
B_B1 b1 0 V = if( (V(c2)>2.5 & V(c4)<2.5) | V(c6)>2.5, 5, 0)

* B0 = C1·(NOT C2) + C3·(NOT C4) + C5·(NOT C6) + C7
B_B0 b0 0 V = if( (V(c1)>2.5 & V(c2)<2.5) | (V(c3)>2.5 & V(c4)<2.5) | (V(c5)>2.5 & V(c6)<2.5) | V(c7)>2.5, 5, 0)

* ==== SIMULAÇÃO ====
.tran 0 10m 0 1u

* ==== PLOTS ====
* Plot 1: Entrada analógica
.plot tran V(vin)

* Plot 2: Código termômetro (7 bits)
.plot tran V(c1) V(c2) V(c3) V(c4) V(c5) V(c6) V(c7)

* Plot 3: Saída binária (3 bits)
.plot tran V(b2) V(b1) V(b0)

.end
```

---

## 3.7 Como Simular e o que Observar

### Passo a passo:

1. Salve o código acima como `adc_flash_3bits_completo.cir`
2. Abra no LTspice → **Run** (▶)
3. Adicione **3 painéis de gráfico** separados:
   - **Painel 1:** `V(vin)` — rampa de entrada
   - **Painel 2:** `V(c1)` a `V(c7)` — código termômetro
   - **Painel 3:** `V(b2)`, `V(b1)`, `V(b0)` — saída binária

### Resultados Esperados

```
Tempo    Vin       Termômetro    B2  B1  B0   Decimal
─────────────────────────────────────────────────────
0,0ms    0,000V    0000000       0   0   0      0
1,25ms   0,625V    0000001       0   0   1      1
2,50ms   1,250V    0000011       0   1   0      2
3,75ms   1,875V    0000111       0   1   1      3
5,00ms   2,500V    0001111       1   0   0      4
6,25ms   3,125V    0011111       1   0   1      5
7,50ms   3,750V    0111111       1   1   0      6
8,75ms   4,375V    1111111       1   1   1      7
```

> [!TIP]
> **Para o relatório**, o gráfico mais importante é o **Painel 3** (saída binária). Ele deve mostrar a saída B2 B1 B0 formando uma **"escada"** em contagem binária conforme Vin sobe. Esse gráfico prova que o conversor A/D funciona corretamente!

### Como configurar os painéis no LTspice:

1. Após a simulação rodar, clique com **botão direito** na área do gráfico
2. Selecione **Add Plot Pane** para criar painéis separados
3. Clique no ícone do gráfico (ou **Plot Settings → Add Trace**) para adicionar os sinais
4. Para facilitar a visualização digital, clique direito no eixo Y → ajuste o range para **-0.5 a 5.5**

---

## 3.8 Interpretação do Gráfico de Saída

O que você deve ver no gráfico B2 B1 B0:

```
5V ┤                                    ┌─────── B2
   │                                    │
   │                          ┌─────────┘
   │                          │  (100→101)
0V ┤──────────────────────────┘
   │
5V ┤            ┌───┐         │         ┌─────── B1
   │            │   │         │         │
   │     ┌──────┘   └─────────┘  ┌──────┘
   │     │  (010→011)     (110→111)
0V ┤─────┘                       │
   │
5V ┤      ┌──┐  ┌──┐  ┌──┐  ┌──┐─────── B0
   │      │  │  │  │  │  │  │  │
   │   ┌──┘  └──┘  └──┘  └──┘  │
   │   │ (001)(011)(101)(111)
0V ┤───┘
   └──────────────────────────────→ Tempo
    0    1.25  2.5  3.75  5   6.25 7.5  8.75  10ms
         ↑     ↑    ↑    ↑    ↑    ↑    ↑
        0,625 1,25 1,875 2,5 3,125 3,75 4,375  (Vin em V)
```

> [!IMPORTANT]
> Note que:
> - **B2** muda apenas 1 vez (na metade — quando Vin = 2,5V)
> - **B1** muda 2 vezes (nos quartos)
> - **B0** muda 4 vezes (nos oitavos) — é o bit que "pisca" mais rápido
> 
> Esse é exatamente o comportamento de uma **contagem binária crescente** (000, 001, 010, ..., 111).

---

## 3.9 Checklist de Validação — Etapa 3

- [ ] Equações booleanas derivadas e simplificadas
- [ ] Verificação exaustiva dos 8 níveis (todas corretas)
- [ ] Diagrama de portas lógicas desenhado
- [ ] Netlist LTspice do ADC completo rodando
- [ ] Saída binária B2 B1 B0 formando escada de 000 a 111
- [ ] Screenshots dos 3 painéis capturados para o relatório

---

## Próxima Etapa

> **Etapa 4 — Simulação Final, Validação e Material para o Relatório**
> - Simulação completa com análise detalhada
> - Tabela comparativa: teórico vs. simulado
> - Organização de todos os screenshots e materiais
> - Montagem final de todo o conteúdo para o relatório

**Aguardando sua aprovação para prosseguir com a Etapa 4 (final)!** 🚀
