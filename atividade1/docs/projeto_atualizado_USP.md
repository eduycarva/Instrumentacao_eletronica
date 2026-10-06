# Projeto Atualizado — Conversor A/D Flash 3 bits

## Parâmetros Derivados do Número USP

**Número USP: 15636150** → a=1, b=5, c=6, d=3, e=6, f=1, g=5, h=0

| Parâmetro | Fórmula | Valor |
|---|---|---|
| Alimentação | ±ab V | **±15V** |
| Tensão de referência | +g,h V | **+5,0V** |
| Resolução | — | **3 bits** |
| LSB | 5,0V / 2³ | **0,625V** |
| Comparadores | 2³ − 1 | **7** |
| Resistores | 2³ | **8 × 1kΩ** |

> [!IMPORTANT]
> **O que mudou em relação ao projeto anterior:**
> - ~~Comparadores ideais~~ → **LM741** (op-amp em malha aberta) e **LM311** (comparador dedicado)
> - ~~Alimentação 5V~~ → **±15V** (alimentação simétrica)
> - Duas simulações obrigatórias (uma com cada componente)
> - Relatório: **máximo 5 páginas** (sem capa)
> - A **Vref = 5,0V** e a **escada resistiva** permanecem iguais ✅

---

## O que é cada componente?

### LM741 — Op-Amp usado como Comparador

O LM741 é um **amplificador operacional** clássico. Quando usado em **malha aberta** (sem realimentação), ele funciona como comparador:
- Se V(+) > V(−) → saída satura em **≈ +13,5V** (perto de +Vcc)
- Se V(+) < V(−) → saída satura em **≈ −13,5V** (perto de −Vcc)

**Características:**
- Slew rate: **0,5 V/µs** (lento!)
- Tempo de resposta: **~20µs** para transição completa
- Saída: swing rail-to-rail (quase ±15V)
- ⚠️ Não é projetado para ser comparador — funciona, mas é lento

### LM311 — Comparador Dedicado

O LM311 é um **comparador analógico** projetado especificamente para esta função:
- Saída **open-collector** (precisa de resistor de pull-up)
- Muito mais rápido que o LM741
- Tempo de resposta: **~200ns** (100× mais rápido!)

**Características:**
- Saída open-collector → pull-up para +5V gera saída digital limpa (0V / 5V)
- Pode acionar lógica TTL/CMOS diretamente
- Projetado para alta velocidade de comutação

### Tabela Comparativa Rápida

| Característica | LM741 | LM311 |
|---|---|---|
| Tipo | Op-amp | Comparador |
| Slew rate | 0,5 V/µs | N/A (muito rápido) |
| Tempo de resposta | ~20 µs | ~200 ns |
| Saída (com ±15V) | ≈ −13,5V a +13,5V | Open-collector (0V a Vpullup) |
| Nível lógico alto | ~+13,5V | Definido pelo pull-up (5V) |
| Nível lógico baixo | ~−13,5V | ~0,2V (Vce_sat) |
| Adequado como comparador? | Funciona, mas lento | **Sim — projetado para isso** |

---

## Circuito Completo — Diagrama de Blocos

```
                 ±15V (alimentação dos comparadores)
                   │
    Vin ──────┬────┤──→ [7× LM741 ou LM311] ──→ [Codificador] ──→ B2 B1 B0
    (0 a 5V)  │    │         ↑
              │    │    [Escada Resistiva]
              │    │    (8 × 1kΩ, Vref=5V)
              │    │
              │   GND
              │
         Rampa PWL
         (0V a 5V)
```

---

## Netlist 1 — Simulação com LM741

Salve como **`adc_flash_LM741.cir`**

```spice
* ================================================================
*  CONVERSOR A/D FLASH 3 BITS — SIMULAÇÃO COM LM741
*  Alimentação: ±15V | Vref: 5,0V | N.USP: 15636150
* ================================================================

* ---- MODELO SPICE DO LM741 ----
.SUBCKT LM741 1 2 3 4 5
* Pinagem: 1=IN+ 2=IN- 3=V+ 4=V- 5=OUT
C1 11 12 8.661E-12
C2 6 7 30.00E-12
DC 5 53 DX
DE 54 5 DX
DLP 90 91 DX
DLN 92 90 DX
DP 4 3 DX
EGND 99 0 POLY(2) (3,0) (4,0) 0 .5 .5
FB 7 99 POLY(5) VB VC VE VLP VLN 0 10.61E6 -10E6 10E6 10E6 -10E6
GA 6 0 11 12 188.5E-6
GCM 0 6 10 99 5.961E-9
IEE 10 4 DC 15.16E-6
HLIM 90 0 VLIM 1K
Q1 11 2 13 QX
Q2 12 1 14 QX
R2 6 9 100.0E3
RC1 3 11 5.305E3
RC2 3 12 5.305E3
RE1 13 10 1.836E3
RE2 14 10 1.836E3
REE 10 99 13.19E6
RO1 8 5 50
RO2 7 99 100
RP 3 4 18.16E3
VB 9 0 DC 0
VC 3 53 DC 1
VE 54 4 DC 1
VLIM 7 8 DC 0
VLP 91 0 DC 40
VLN 0 92 DC 40
.MODEL DX D(IS=800.0E-18 BV=50)
.MODEL QX NPN(IS=800.0E-18 BF=93.75)
.ENDS LM741

* ---- FONTES DE ALIMENTAÇÃO ----
Vcc  vcc 0 DC 15      ; +15V
Vee  vee 0 DC -15     ; -15V

* ---- FONTE DE REFERÊNCIA ----
Vref vref 0 DC 5      ; +5,0V (Vref = +g,h = +5,0V)

* ---- SINAL DE ENTRADA ----
* Rampa lenta de 0V a 5V em 5ms
Vin vin 0 PWL(0 0 5m 5)

* ---- ESCADA RESISTIVA (8 × 1kΩ) ----
R8 vref   node7  1k
R7 node7  node6  1k
R6 node6  node5  1k
R5 node5  node4  1k
R4 node4  node3  1k
R3 node3  node2  1k
R2 node2  node1  1k
R1 node1  0      1k

* ---- 7 COMPARADORES LM741 (op-amp em malha aberta) ----
* IN+ = vin | IN- = nó da escada | V+ = vcc | V- = vee | OUT = ck
XC1 vin node1 vcc vee c1 LM741
XC2 vin node2 vcc vee c2 LM741
XC3 vin node3 vcc vee c3 LM741
XC4 vin node4 vcc vee c4 LM741
XC5 vin node5 vcc vee c5 LM741
XC6 vin node6 vcc vee c6 LM741
XC7 vin node7 vcc vee c7 LM741

* ---- CODIFICADOR (TERMÔMETRO → BINÁRIO) ----
* Threshold = 0V (saída LM741 oscila entre ~-13,5V e ~+13,5V)
* B2 = C4
B_B2 b2 0 V = if(V(c4) > 0, 5, 0)
* B1 = C2·(NOT C4) + C6
B_B1 b1 0 V = if( (V(c2)>0 & V(c4)<0) | V(c6)>0, 5, 0)
* B0 = C1·(NOT C2) + C3·(NOT C4) + C5·(NOT C6) + C7
B_B0 b0 0 V = if( (V(c1)>0 & V(c2)<0) | (V(c3)>0 & V(c4)<0) | (V(c5)>0 & V(c6)<0) | V(c7)>0, 5, 0)

* ---- SIMULAÇÃO ----
.tran 0 5m 0 0.5u

.end
```

> [!NOTE]
> **Sobre a saída do LM741:**
> - Com ±15V de alimentação, a saída satura em aproximadamente **+13,5V** (nível alto) e **−13,5V** (nível baixo)
> - O threshold do codificador usa **0V** como limiar (positivo = "1", negativo = "0")
> - Observe que a transição NÃO é instantânea — o slew rate de 0,5V/µs causa uma rampa visível

---

## Netlist 2 — Simulação com LM311

Salve como **`adc_flash_LM311.cir`**

```spice
* ================================================================
*  CONVERSOR A/D FLASH 3 BITS — SIMULAÇÃO COM LM311
*  Alimentação: ±15V | Vref: 5,0V | N.USP: 15636150
* ================================================================

* ---- MODELO SPICE DO LM311 ----
.SUBCKT LM311 1 2 5 3 4
* Pinagem: 1=IN+ 2=IN- 5=OUT(collector) 3=V+ 4=V-
* Modelo simplificado com saída open-collector
RI 1 2 200MEG
GA 0 6 1 2 2.5E-3
R1 6 0 400
CC 6 0 20P
D1 6 7 DCLAMP
D2 8 6 DCLAMP
V1 3 7 DC 1.4
V2 8 4 DC 1.4
EOUT 9 0 6 0 1
Q1 5 10 4 QOUT
R2 9 10 50
.MODEL DCLAMP D(IS=1E-15 BV=50)
.MODEL QOUT NPN(IS=1E-15 BF=500 TF=0.5N)
.ENDS LM311

* ---- FONTES DE ALIMENTAÇÃO ----
Vcc  vcc 0 DC 15      ; +15V
Vee  vee 0 DC -15     ; -15V

* ---- TENSÃO DO PULL-UP ----
Vpu  vpu 0 DC 5       ; Pull-up para 5V (nível lógico TTL)

* ---- FONTE DE REFERÊNCIA ----
Vref vref 0 DC 5      ; +5,0V

* ---- SINAL DE ENTRADA ----
* Rampa lenta de 0V a 5V em 5ms
Vin vin 0 PWL(0 0 5m 5)

* ---- ESCADA RESISTIVA (8 × 1kΩ) ----
R8 vref   node7  1k
R7 node7  node6  1k
R6 node6  node5  1k
R5 node5  node4  1k
R4 node4  node3  1k
R3 node3  node2  1k
R2 node2  node1  1k
R1 node1  0      1k

* ---- 7 COMPARADORES LM311 (saída open-collector) ----
* IN+ = vin | IN- = nó da escada | OUT = collector | V+ = vcc | V- = vee
XC1 vin node1 c1 vcc vee LM311
XC2 vin node2 c2 vcc vee LM311
XC3 vin node3 c3 vcc vee LM311
XC4 vin node4 c4 vcc vee LM311
XC5 vin node5 c5 vcc vee LM311
XC6 vin node6 c6 vcc vee LM311
XC7 vin node7 c7 vcc vee LM311

* ---- RESISTORES DE PULL-UP (10kΩ para +5V) ----
* Necessários porque o LM311 tem saída open-collector
Rpu1 vpu c1 10k
Rpu2 vpu c2 10k
Rpu3 vpu c3 10k
Rpu4 vpu c4 10k
Rpu5 vpu c5 10k
Rpu6 vpu c6 10k
Rpu7 vpu c7 10k

* ---- CODIFICADOR (TERMÔMETRO → BINÁRIO) ----
* Threshold = 2.5V (saída LM311 oscila entre ~0V e ~5V com pull-up)
* B2 = C4
B_B2 b2 0 V = if(V(c4) > 2.5, 5, 0)
* B1 = C2·(NOT C4) + C6
B_B1 b1 0 V = if( (V(c2)>2.5 & V(c4)<2.5) | V(c6)>2.5, 5, 0)
* B0 = C1·(NOT C2) + C3·(NOT C4) + C5·(NOT C6) + C7
B_B0 b0 0 V = if( (V(c1)>2.5 & V(c2)<2.5) | (V(c3)>2.5 & V(c4)<2.5) | (V(c5)>2.5 & V(c6)<2.5) | V(c7)>2.5, 5, 0)

* ---- SIMULAÇÃO ----
.tran 0 5m 0 0.5u

.end
```

> [!NOTE]
> **Sobre a saída do LM311:**
> - Saída **open-collector**: quando Vin > Vref → transistor de saída desliga → pull-up puxa para **+5V**
> - Quando Vin < Vref → transistor satura → saída vai para **≈ 0,2V** (Vce_sat)
> - Resultado: saída digital limpa entre **0V e 5V** — ideal para lógica TTL/CMOS
> - Transições muito mais rápidas que o LM741

---

## O que Observar na Comparação

### Gráficos para Capturar

Para **cada simulação** (LM741 e LM311), capture:

1. **Saída dos comparadores (C1–C7):** observe a forma de onda
2. **Saída binária (B2, B1, B0):** confirme a contagem 000→111
3. **Zoom na transição:** amplie uma transição (ex: quando Vin cruza 2,5V)

### O que será diferente entre os dois

| Aspecto | LM741 | LM311 |
|---|---|---|
| **Nível de saída ALTO** | ~+13,5V | ~+5V (via pull-up) |
| **Nível de saída BAIXO** | ~−13,5V | ~0,2V |
| **Velocidade da transição** | Lenta (rampa visível ~20µs) | Rápida (praticamente vertical ~200ns) |
| **Overshoot/ringing** | Possível | Mínimo |
| **Compatibilidade TTL** | ❌ Precisa de interface | ✅ Direto |
| **Precisão do limiar** | Pode ter offset (~2mV) | Melhor (~2mV, mas projetado para isso) |

> [!IMPORTANT]
> **Para o relatório, destaque estas diferenças:**
> 1. O LM741 é **mais lento** — no zoom da transição, você verá uma rampa em vez de um degrau
> 2. O LM741 tem **excursão de saída inadequada** para lógica digital (±13,5V vs. 0/5V)
> 3. O LM311 produz saída **compatível com TTL** diretamente (com pull-up)
> 4. O LM311 é a **escolha correta** para ADC Flash real

---

## Estrutura do Relatório (Máximo 5 Páginas)

> [!CAUTION]
> O professor exige **no máximo 5 páginas** (sem contar a capa). Seja conciso!

### Página 1 — Introdução + Parâmetros

- Objetivo do projeto (2–3 linhas)
- Parâmetros derivados do N.USP 15636150 (tabela)
- Breve explicação do ADC Flash (1 parágrafo)

### Página 2 — Circuito e Funcionamento

- Diagrama de blocos do circuito completo
- Escada resistiva: cálculo das tensões de referência (tabela)
- Equações do codificador: B2 = C4, B1 = C2·C̄4 + C6, B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7
- Tabela verdade (termômetro → binário)

### Página 3 — Simulação com LM741

- Screenshot do esquemático (ou print da netlist)
- Gráfico: entrada Vin + saída B2 B1 B0
- Gráfico: zoom em uma transição (mostrando slew rate lento)
- Breve comentário sobre o comportamento

### Página 4 — Simulação com LM311

- Screenshot do esquemático
- Gráfico: entrada Vin + saída B2 B1 B0
- Gráfico: zoom na mesma transição (mostrando resposta rápida)
- Breve comentário sobre o comportamento

### Página 5 — Comparação e Conclusão

- Tabela comparativa LM741 vs LM311 (tempo de resposta, níveis de saída, adequação)
- Vantagens e limitações do ADC Flash
- Conclusão (1 parágrafo)
- Referências (datasheets LM741, LM311)

---

## Arquivos para Entrega

De acordo com as instruções do professor:

| Arquivo | Descrição |
|---|---|
| `relatorio_ADC_flash.pdf` | Relatório (máx. 5 páginas + capa) |
| `simulacao.zip` contendo: | Arquivo compactado |
| → `adc_flash_LM741.cir` | Netlist LTspice com LM741 |
| → `adc_flash_LM311.cir` | Netlist LTspice com LM311 |

Se usar simulação **online** (ex: Falstad, Multisim Live), inclua também:
- Arquivo `.txt` com o link da simulação

---

## Checklist de Entrega (Prazo: 28/08/2026 às 23:59)

- [ ] Netlist `adc_flash_LM741.cir` funcionando no LTspice
- [ ] Netlist `adc_flash_LM311.cir` funcionando no LTspice
- [ ] Screenshots da simulação com LM741 (esquemático + gráficos)
- [ ] Screenshots da simulação com LM311 (esquemático + gráficos)
- [ ] Zoom nas transições para comparação
- [ ] Relatório PDF (máx. 5 páginas sem capa)
- [ ] Arquivo .zip com as netlists
- [ ] (Se online) arquivo .txt com link da simulação
- [ ] Entrega antes de **28/08/2026 às 23:59**
