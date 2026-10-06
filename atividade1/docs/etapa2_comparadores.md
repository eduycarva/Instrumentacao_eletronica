# Etapa 2 — Montagem dos 7 Comparadores

## 2.1 Princípio de Funcionamento do Comparador

Um comparador é um circuito que compara duas tensões e gera uma saída digital:

```
Se  V(+) > V(−)  →  Saída = ALTO  (≈ Vcc ou 5V)
Se  V(+) < V(−)  →  Saída = BAIXO (≈ 0V ou GND)
```

No nosso ADC Flash:
- **Entrada não-inversora (+):** recebe o sinal analógico **Vin** (mesma tensão em todos os 7)
- **Entrada inversora (−):** recebe a tensão de referência do nó da escada (diferente para cada comparador)

### Comportamento Esperado

Quando Vin sobe gradualmente de 0V a 5V:

```
Vin = 0,3V  →  Nenhum comparador "liga"     →  Termômetro: 0000000
Vin = 0,8V  →  C1 liga (0,8 > 0,625)        →  Termômetro: 0000001
Vin = 1,5V  →  C1, C2 ligam                 →  Termômetro: 0000011
Vin = 2,0V  →  C1, C2, C3 ligam             →  Termômetro: 0000111
   ...e assim por diante...
Vin = 4,5V  →  Todos ligam                  →  Termômetro: 1111111
```

> [!IMPORTANT]
> O nome **"código termômetro"** vem justamente dessa característica: as saídas dos comparadores "preenchem" de baixo para cima, como mercúrio subindo em um termômetro.

---

## 2.2 Escolha do Componente

### No LTspice — Comparador Ideal (Behavioral Source)

No LTspice, a forma mais simples e confiável de criar um comparador ideal é usando uma **fonte comportamental (behavioral voltage source)**:

```spice
B_C1 c1_out 0 V = if(V(vin) > V(node1), 5, 0)
```

Isso significa: "Se Vin > Vref_C1, a saída é 5V; senão é 0V."

> [!TIP]
> **Por que comparador ideal e não LM339?**
> - O comparador ideal é **mais simples** de configurar
> - Evita problemas com saída open-collector, pull-ups, alimentação
> - Para o relatório acadêmico, o comportamento é idêntico
> - Se o professor exigir componente real, basta trocar depois — a lógica não muda

### Na prática (componente real)

Se precisar usar componentes reais:
- **LM339** → 4 comparadores por chip (precisa de 2 chips para 7 comparadores)
- **LM393** → 2 comparadores por chip (precisa de 4 chips)
- Ambos têm saída **open-collector** → necessitam resistor de pull-up (~10kΩ para Vcc)

---

## 2.3 Diagrama de Ligação

```
                          ┌────────────────────────────────────┐
                          │         7 COMPARADORES             │
                          │                                    │
    Vin ──────────┬───────┤──→ (+) C7  ──  (−) ← 4,375V ────→ │── c7_out
    (rampa        │       │                                    │
     0V a 5V)     ├───────┤──→ (+) C6  ──  (−) ← 3,750V ────→ │── c6_out
                  │       │                                    │
                  ├───────┤──→ (+) C5  ──  (−) ← 3,125V ────→ │── c5_out
                  │       │                                    │
                  ├───────┤──→ (+) C4  ──  (−) ← 2,500V ────→ │── c4_out
                  │       │                                    │
                  ├───────┤──→ (+) C3  ──  (−) ← 1,875V ────→ │── c3_out
                  │       │                                    │
                  ├───────┤──→ (+) C2  ──  (−) ← 1,250V ────→ │── c2_out
                  │       │                                    │
                  └───────┤──→ (+) C1  ──  (−) ← 0,625V ────→ │── c1_out
                          │                                    │
                          └────────────────────────────────────┘
                                          ▲
                                          │
                                   Tensões vêm da
                                   escada resistiva
                                   (Etapa 1)
```

### Detalhamento das conexões:

| Comparador | Entrada (+) | Entrada (−) | Saída | Liga quando... |
|---|---|---|---|---|
| C1 | Vin | node1 = 0,625V | c1_out | Vin > 0,625V |
| C2 | Vin | node2 = 1,250V | c2_out | Vin > 1,250V |
| C3 | Vin | node3 = 1,875V | c3_out | Vin > 1,875V |
| C4 | Vin | node4 = 2,500V | c4_out | Vin > 2,500V |
| C5 | Vin | node5 = 3,125V | c5_out | Vin > 3,125V |
| C6 | Vin | node6 = 3,750V | c6_out | Vin > 3,750V |
| C7 | Vin | node7 = 4,375V | c7_out | Vin > 4,375V |

---

## 2.4 Tabela Verdade — Código Termômetro

| Faixa de Vin | C7 | C6 | C5 | C4 | C3 | C2 | C1 | Significado |
|---|---|---|---|---|---|---|---|---|
| 0 – 0,625V | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Nenhum comparador ativo |
| 0,625 – 1,250V | 0 | 0 | 0 | 0 | 0 | 0 | **1** | 1 comparador ativo |
| 1,250 – 1,875V | 0 | 0 | 0 | 0 | 0 | **1** | **1** | 2 comparadores ativos |
| 1,875 – 2,500V | 0 | 0 | 0 | 0 | **1** | **1** | **1** | 3 comparadores ativos |
| 2,500 – 3,125V | 0 | 0 | 0 | **1** | **1** | **1** | **1** | 4 comparadores ativos |
| 3,125 – 3,750V | 0 | 0 | **1** | **1** | **1** | **1** | **1** | 5 comparadores ativos |
| 3,750 – 4,375V | 0 | **1** | **1** | **1** | **1** | **1** | **1** | 6 comparadores ativos |
| 4,375 – 5,000V | **1** | **1** | **1** | **1** | **1** | **1** | **1** | 7 comparadores ativos |

> [!NOTE]
> Observe o padrão: os "1s" sempre preenchem de C1 (baixo) para C7 (cima). Nunca haverá um padrão como "0010100" — isso seria um erro no circuito.

---

## 2.5 Netlist LTspice — Escada + Comparadores

Salve o conteúdo abaixo como **`adc_flash_comparadores.cir`** e abra no LTspice.

```spice
* ============================================================
* Conversor A/D Flash 3 bits - Etapa 2
* Escada Resistiva + 7 Comparadores Ideais
* Vref = 5V, Vin = rampa de 0V a 5V
* ============================================================

* ---- FONTE DE REFERÊNCIA ----
Vref vref 0 5V

* ---- SINAL DE ENTRADA (rampa de 0V a 5V em 10ms) ----
Vin vin 0 PWL(0 0 10m 5)

* ---- ESCADA RESISTIVA (8 x 1kΩ) ----
R8 vref  node7  1k
R7 node7  node6  1k
R6 node6  node5  1k
R5 node5  node4  1k
R4 node4  node3  1k
R3 node3  node2  1k
R2 node2  node1  1k
R1 node1  0      1k

* ---- 7 COMPARADORES IDEAIS ----
* Cada comparador: se Vin > Vref_nó, saída = 5V; senão = 0V

B_C1 c1 0 V = if(V(vin) > V(node1), 5, 0)
B_C2 c2 0 V = if(V(vin) > V(node2), 5, 0)
B_C3 c3 0 V = if(V(vin) > V(node3), 5, 0)
B_C4 c4 0 V = if(V(vin) > V(node4), 5, 0)
B_C5 c5 0 V = if(V(vin) > V(node5), 5, 0)
B_C6 c6 0 V = if(V(vin) > V(node6), 5, 0)
B_C7 c7 0 V = if(V(vin) > V(node7), 5, 0)

* ---- SIMULAÇÃO ----
.tran 0 10m 0 1u

* ---- PLOT ----
.plot tran V(vin) V(c1) V(c2) V(c3) V(c4) V(c5) V(c6) V(c7)

.end
```

---

## 2.6 Como Simular no LTspice

### Passo a passo:

1. **Salvar** o código acima como `adc_flash_comparadores.cir`
2. **Abrir** no LTspice (File → Open → selecionar o arquivo)
3. **Rodar** a simulação (Run ou botão ▶)
4. **Plotar as formas de onda:**
   - Clique com botão direito no gráfico → **Add Trace**
   - Adicione: `V(vin)`, `V(c1)`, `V(c2)`, `V(c3)`, `V(c4)`, `V(c5)`, `V(c6)`, `V(c7)`

### O que você deve observar:

```
Tempo (ms)    Vin (V)    Saídas dos Comparadores
──────────────────────────────────────────────────
0,0 – 1,25    0 – 0,625   Todos em 0V (0000000)
1,25           0,625       C1 sobe para 5V (0000001)
2,50           1,250       C2 sobe para 5V (0000011)
3,75           1,875       C3 sobe para 5V (0000111)
5,00           2,500       C4 sobe para 5V (0001111)
6,25           3,125       C5 sobe para 5V (0011111)
7,50           3,750       C6 sobe para 5V (0111111)
8,75           4,375       C7 sobe para 5V (1111111)
```

> [!TIP]
> **Para o relatório:** Capture um screenshot mostrando:
> 1. A rampa de Vin (linha diagonal subindo)
> 2. As 7 saídas dos comparadores "ligando" uma por uma em sequência (como degraus)
> 
> Esse gráfico é a **prova visual** do código termômetro funcionando!

### Dica de visualização no LTspice:

Para ver melhor cada sinal separado, você pode:
- **Separar em painéis:** Clique direito no gráfico → **Add Plot Pane** → arraste os sinais para painéis diferentes
- **Ou usar offset:** Plot `V(c1)`, `V(c2)+6`, `V(c3)+12`, etc. para empilhar os sinais

---

## 2.7 Verificação Detalhada

### Teste pontual — Vin = 2,0V

Quando Vin = 2,0V, quais comparadores devem estar ativos?

```
C1: 2,0V > 0,625V?  → SIM → Saída = 5V  ✅
C2: 2,0V > 1,250V?  → SIM → Saída = 5V  ✅
C3: 2,0V > 1,875V?  → SIM → Saída = 5V  ✅
C4: 2,0V > 2,500V?  → NÃO → Saída = 0V  ✅
C5: 2,0V > 3,125V?  → NÃO → Saída = 0V  ✅
C6: 2,0V > 3,750V?  → NÃO → Saída = 0V  ✅
C7: 2,0V > 4,375V?  → NÃO → Saída = 0V  ✅

Código termômetro: 0000111  →  3 comparadores ativos  →  Nível 3 (011 em binário)
```

### Teste pontual — Vin = 3,5V

```
C1: 3,5V > 0,625V?  → SIM → 5V  ✅
C2: 3,5V > 1,250V?  → SIM → 5V  ✅
C3: 3,5V > 1,875V?  → SIM → 5V  ✅
C4: 3,5V > 2,500V?  → SIM → 5V  ✅
C5: 3,5V > 3,125V?  → SIM → 5V  ✅
C6: 3,5V > 3,750V?  → NÃO → 0V  ✅
C7: 3,5V > 4,375V?  → NÃO → 0V  ✅

Código termômetro: 0011111  →  5 comparadores ativos  →  Nível 5 (101 em binário)
```

---

## 2.8 Checklist de Validação — Etapa 2

- [ ] 7 comparadores montados (C1 a C7)
- [ ] Vin conectado na entrada (+) de todos
- [ ] Cada tensão de referência na entrada (−) correta
- [ ] Simulação com rampa mostra os comparadores ligando em sequência
- [ ] Código termômetro confere com a tabela do item 2.4
- [ ] Screenshots capturados para o relatório

---

## Próxima Etapa

> **Etapa 3 — Codificador Termômetro → Binário**
> - Derivar as equações lógicas (B2, B1, B0) a partir do código termômetro
> - Implementar usando portas lógicas (ou CI 74LS148)
> - Adicionar ao circuito LTspice
> - Simular o ADC completo (entrada analógica → saída digital de 3 bits)

**Aguardando sua aprovação para prosseguir com a Etapa 3!** 🚀
