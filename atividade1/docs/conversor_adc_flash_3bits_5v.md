# Conversor A/D Flash de 3 Bits — Vref = 5V

## Parâmetros do Projeto

| Parâmetro | Valor |
|---|---|
| Resolução (N) | **3 bits** |
| Níveis de quantização | 2³ = **8 níveis** (000 a 111) |
| Tensão de referência (Vref) | **5V** |
| LSB (menor bit) | Vref / 2ᴺ = 5V / 8 = **0,625V** |
| Comparadores necessários | 2ᴺ − 1 = **7** |
| Resistores na escada | 2ᴺ = **8** (mesmo valor) |

---

## Passo a Passo Completo

### Passo 1 — Fixar os Parâmetros

- Resolução: N = 3 bits → saída de 000 a 111
- Vref = **5V** (tensão de bancada padrão)
- LSB = 5V / 8 = **0,625V**
- Faixa total de entrada: **0V a 5V**

---

### Passo 2 — Montar a Escada Resistiva (R-Ladder)

- Use **8 resistores de mesmo valor** (ex: R = 1kΩ cada) em **série** entre Vref (5V) e GND.
- Isso cria **7 nós intermediários**, gerando as 7 tensões de referência para os comparadores:

| Comparador | Tensão de Referência | Cálculo |
|---|---|---|
| C1 | **0,625V** | 1 × (5V / 8) |
| C2 | **1,250V** | 2 × (5V / 8) |
| C3 | **1,875V** | 3 × (5V / 8) |
| C4 | **2,500V** | 4 × (5V / 8) |
| C5 | **3,125V** | 5 × (5V / 8) |
| C6 | **3,750V** | 6 × (5V / 8) |
| C7 | **4,375V** | 7 × (5V / 8) |

> [!TIP]
> Na prática, use resistores de **1kΩ (1%)** para minimizar erros. Resistores de 5% funcionam, mas geram imprecisão nas tensões de referência.

**Esquema da escada:**
```
 5V (Vref)
  │
 [R8 = 1kΩ]
  ├──── 4,375V → C7
 [R7 = 1kΩ]
  ├──── 3,750V → C6
 [R6 = 1kΩ]
  ├──── 3,125V → C5
 [R5 = 1kΩ]
  ├──── 2,500V → C4
 [R4 = 1kΩ]
  ├──── 1,875V → C3
 [R3 = 1kΩ]
  ├──── 1,250V → C2
 [R2 = 1kΩ]
  ├──── 0,625V → C1
 [R1 = 1kΩ]
  │
 GND
```

---

### Passo 3 — Montar os 7 Comparadores

- Cada comparador Ck compara a **tensão de entrada (Vin)** com a **tensão de referência** do nó correspondente.
- **Saída = 1** se Vin > Vref_k, **senão 0**.
- Vin é ligado à entrada **não-inversora (+)** de todos os 7 comparadores.
- Cada tensão de referência vai na entrada **inversora (−)** do respectivo comparador.

**Componentes sugeridos para simulação:**
- **Comparador ideal** (mais simples, disponível no LTspice)
- **LM339** ou **LM393** (componentes reais — o LM339 tem 4 comparadores por chip, então precisa de **2 chips**)

> [!NOTE]
> O LM339/LM393 tem saída **open-collector**, então precisa de **resistor de pull-up** (~10kΩ para Vcc) em cada saída.

---

### Passo 4 — Tabela Verdade Completa (Código Termômetro → Binário)

> [!IMPORTANT]
> Esta tabela é o **coração do relatório**. Construa-a e valide-a na simulação.

| Faixa de Vin | C7 C6 C5 C4 C3 C2 C1 (termômetro) | B2 B1 B0 (binário) | Valor Decimal |
|---|---|---|---|
| 0 – 0,625V | 0000000 | **000** | 0 |
| 0,625 – 1,250V | 0000001 | **001** | 1 |
| 1,250 – 1,875V | 0000011 | **010** | 2 |
| 1,875 – 2,500V | 0000111 | **011** | 3 |
| 2,500 – 3,125V | 0001111 | **100** | 4 |
| 3,125 – 3,750V | 0011111 | **101** | 5 |
| 3,750 – 4,375V | 0111111 | **110** | 6 |
| 4,375 – 5,000V | 1111111 | **111** | 7 |

---

### Passo 5 — Projetar o Codificador (Termômetro → Binário)

O codificador converte os 7 bits do código termômetro em 3 bits binários. Há duas abordagens:

#### Opção A — "Na mão" com Mapas de Karnaugh (mais didático)

Monte mapas de Karnaugh para cada saída (B2, B1, B0) em função de C1…C7, simplifique e implemente com portas lógicas.

**Equações simplificadas:**

```
B0 = C1 ⊕ C2 ⊕ C3 ⊕ C4 ⊕ C5 ⊕ C6 ⊕ C7

B1 = C2·(C̄3) + C4·(C̄5) + C6·(C̄7)
   = (C2 XOR C3) + (C4 XOR C5) + (C6 XOR C7)
   [simplificação prática considerando o padrão termômetro]

B2 = C4
```

> [!TIP]
> Uma forma mais direta (válida para código termômetro puro):
> ```
> B2 = C4
> B1 = C2·(C̄4) + C6·(C̄4)  →  equivalente a: (C2 + C6)·(C̄4) + C6
>    = C2 ⊕ C3  +  C6 ⊕ C7  (no padrão termômetro)
> B0 = C1 ⊕ C2 + C3 ⊕ C4 + C5 ⊕ C6 + C7
> ```
> A melhor prática é derivar pela tabela verdade do seu caso específico.

**Portas necessárias (exemplo):** XOR (74HC86), OR (74HC32), AND (74HC08), NOT (74HC04).

#### Opção B — Usar o CI 74LS148 (encoder de prioridade, mais rápido)

- O **74LS148** é um encoder de prioridade 8-para-3.
- ⚠️ É **ativo em nível baixo** (entradas e saídas invertidas), então pode precisar de **portas NOT** (74HC04) nas entradas e/ou saídas.
- Ligações: as saídas dos comparadores (C1–C7) vão nas entradas do 74LS148, com a entrada 0 fixada em nível ativo.

---

### Passo 6 — Escolher a Ferramenta de Simulação

| Programa | Tipo | Melhor para | Gratuito? |
|---|---|---|---|
| **LTspice** | SPICE | Parte analógica (escada + comparadores reais) | ✅ Sim |
| **Proteus (ISIS)** | Misto | Analógico + Digital no mesmo esquema, CIs reais | ❌ Licença (tem versão demo) |
| **Multisim** | Misto | Analógico + Digital, muito usado em universidades | ❌ Licença educacional |
| **Logisim** | Digital | Apenas a parte do codificador (portas lógicas) | ✅ Sim |
| **Falstad Circuit Simulator** | Online | Protótipos rápidos, bom para entender o conceito | ✅ Sim |

> [!IMPORTANT]
> **Recomendação para o projeto completo:**
> - **Proteus** ou **Multisim** → permite simular tudo (analógico + digital) em um único esquemático.
> - **LTspice** → excelente para a parte analógica, mas a parte digital (codificador) precisa ser improvisada com modelos SPICE ou feita separadamente.
> - **Logisim** → use como complemento para projetar e validar apenas o codificador.

---

### Passo 7 — Montar o Esquemático Completo

O fluxo completo do circuito:

```
                    ┌──────────────┐     ┌──────────────┐     ┌─────────────┐
 Vin ──────────────►│ 7 Comparadores│────►│ Codificador  │────►│ 3 LEDs ou   │
                    │ (C1 a C7)    │     │ (termômetro  │     │ Display     │
                    └──────┬───────┘     │  → binário)  │     │ (B2 B1 B0)  │
                           │             └──────────────┘     └─────────────┘
                    ┌──────┴───────┐
                    │ Escada       │
                    │ Resistiva    │
                    │ (8 × 1kΩ)   │
                    │ Vref = 5V    │
                    └──────────────┘
```

**Lista de componentes:**

| Componente | Quantidade | Observação |
|---|---|---|
| Resistores 1kΩ | 8 | Escada resistiva (tolerância 1% ideal) |
| Resistores 10kΩ | 7 | Pull-up (se usar LM339/LM393) |
| LM339 | 2 chips | 4 comparadores cada (sobra 1) |
| 74LS148 | 1 | Encoder de prioridade (ou portas lógicas) |
| 74HC04 | 1 | Inversores (se usar 74LS148) |
| LEDs + resistores 330Ω | 3 | Visualizar saída B2, B1, B0 |
| Fonte DC 5V | 1 | Vref e alimentação |
| Fonte variável / rampa | 1 | Sinal de entrada Vin (0 a 5V) |

---

### Passo 8 — Simular e Validar

1. **Aplique uma rampa de tensão** de 0V a 5V na entrada (fonte PWL ou VPULSE em rampa lenta).
2. **Observe a saída digital** "subindo" degrau a degrau, de 000 a 111.
3. **Compare** o resultado obtido com a tabela verdade teórica do Passo 4.
4. **Capture screenshots** de:
   - O esquemático completo
   - As formas de onda (entrada × saída)
   - A tabela de resultados

> [!IMPORTANT]
> Para o relatório, o gráfico mostrando a **rampa de entrada vs. saída digital em escada** é a prova visual de que o conversor funciona corretamente.

---

## Estrutura Sugerida para o Relatório

```
1. CAPA
   - Título: "Conversor Analógico-Digital Flash de 3 Bits"
   - Disciplina, professor, aluno(s), data

2. INTRODUÇÃO
   - O que é um conversor A/D
   - Princípio de funcionamento do tipo Flash
   - Objetivo do trabalho

3. FUNDAMENTAÇÃO TEÓRICA
   - Resolução, LSB, quantização
   - Código termômetro
   - Escada resistiva
   - Comparadores (ideal vs. real)
   - Codificador de prioridade

4. PARÂMETROS DO PROJETO
   - N = 3 bits, Vref = 5V, LSB = 0,625V
   - Tabela de tensões de referência

5. DESENVOLVIMENTO
   5.1 Escada resistiva — cálculos e esquema
   5.2 Comparadores — escolha do componente e ligações
   5.3 Tabela verdade (termômetro → binário)
   5.4 Codificador — equações lógicas ou CI 74LS148
   5.5 Esquemático completo

6. SIMULAÇÃO
   6.1 Ferramenta utilizada
   6.2 Screenshots do esquemático
   6.3 Formas de onda (rampa × saída)
   6.4 Comparação: resultado simulado vs. teórico

7. RESULTADOS E DISCUSSÃO
   - O conversor funciona conforme esperado?
   - Fontes de erro (tolerância dos resistores, offset dos comparadores)
   - Limitações do ADC Flash

8. CONCLUSÃO
   - Resumo dos resultados
   - Aprendizados

9. REFERÊNCIAS
   - Datasheet LM339, 74LS148
   - Livros de eletrônica digital
```

---

## Dicas Finais

> [!TIP]
> - No Proteus/Multisim, você pode usar um **voltímetro virtual** nos nós da escada para confirmar os valores 0,625V, 1,25V, etc.
> - Use **cores diferentes** nos fios do esquemático para facilitar a leitura.
> - No relatório, sempre mostre o **cálculo do LSB** e a **derivação das tensões de referência** — professores valorizam isso.
> - Se usar o LTspice, a diretiva `.tran 0 10ms 0 1us` simula 10ms com passo de 1µs — suficiente para uma rampa lenta.
