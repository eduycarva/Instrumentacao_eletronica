# Etapa 4 — Simulação Final, Validação e Material para Relatório

## 4.1 Netlist Final Definitiva

> [!IMPORTANT]
> Esta é a **versão final** da netlist. Salve como `adc_flash_3bits_5V.cir` e use para todas as capturas do relatório.

```spice
* ================================================================
*  CONVERSOR ANALÓGICO-DIGITAL FLASH DE 3 BITS
* ================================================================
*  Parâmetros:
*    Resolução:          N = 3 bits
*    Tensão de ref.:     Vref = 5V
*    LSB:                0,625V
*    Comparadores:       7 (ideais)
*    Resistores:         8 x 1kΩ
* ================================================================

* ================ FONTES ================

* Tensão de referência (alimenta a escada)
Vref vref 0 DC 5

* Sinal de entrada: rampa lenta de 0V a 5V em 10ms
Vin vin 0 PWL(0 0 10m 5)

* ================ ESCADA RESISTIVA ================
* 8 resistores de 1kΩ em série entre Vref e GND
* Cria 7 nós com tensões de 0,625V a 4,375V

R8 vref   node7  1k  ; entre 5V e 4,375V
R7 node7  node6  1k  ; entre 4,375V e 3,750V
R6 node6  node5  1k  ; entre 3,750V e 3,125V
R5 node5  node4  1k  ; entre 3,125V e 2,500V
R4 node4  node3  1k  ; entre 2,500V e 1,875V
R3 node3  node2  1k  ; entre 1,875V e 1,250V
R2 node2  node1  1k  ; entre 1,250V e 0,625V
R1 node1  0      1k  ; entre 0,625V e GND

* ================ 7 COMPARADORES IDEAIS ================
* Entrada (+) = Vin (sinal analógico)
* Entrada (-) = tensão de referência do nó
* Saída: 5V se Vin > Vref_nó, senão 0V

B_C1 c1 0 V = if(V(vin) > V(node1), 5, 0)  ; ref = 0,625V
B_C2 c2 0 V = if(V(vin) > V(node2), 5, 0)  ; ref = 1,250V
B_C3 c3 0 V = if(V(vin) > V(node3), 5, 0)  ; ref = 1,875V
B_C4 c4 0 V = if(V(vin) > V(node4), 5, 0)  ; ref = 2,500V
B_C5 c5 0 V = if(V(vin) > V(node5), 5, 0)  ; ref = 3,125V
B_C6 c6 0 V = if(V(vin) > V(node6), 5, 0)  ; ref = 3,750V
B_C7 c7 0 V = if(V(vin) > V(node7), 5, 0)  ; ref = 4,375V

* ================ CODIFICADOR (TERMÔMETRO → BINÁRIO) ================
* Equações derivadas da tabela verdade:
*   B2 = C4
*   B1 = C2·(NOT C4) + C6
*   B0 = C1·(NOT C2) + C3·(NOT C4) + C5·(NOT C6) + C7

B_B2 b2 0 V = if(V(c4)>2.5, 5, 0)

B_B1 b1 0 V = if( (V(c2)>2.5 & V(c4)<2.5) | V(c6)>2.5, 5, 0)

B_B0 b0 0 V = if( (V(c1)>2.5 & V(c2)<2.5) | (V(c3)>2.5 & V(c4)<2.5) | (V(c5)>2.5 & V(c6)<2.5) | V(c7)>2.5, 5, 0)

* ================ SIMULAÇÃO ================
.tran 0 10m 0 1u

.end
```

---

## 4.2 Roteiro de Simulação no LTspice

### Passo 1 — Abrir e Rodar

1. Abra o LTspice
2. **File → Open** → selecione `adc_flash_3bits_5V.cir`
3. Clique em **Run** (▶) ou pressione **Ctrl+R** (ou F5, dependendo da versão)
4. Aguarde a simulação completar (leva poucos segundos)

### Passo 2 — Configurar os Gráficos

Você precisa de **3 painéis** separados para as capturas do relatório:

#### Painel 1 — Entrada Analógica (Vin)
1. Clique com botão direito na área do gráfico → **Add Trace**
2. Digite: `V(vin)`
3. Você verá uma **rampa diagonal** subindo de 0V a 5V

#### Painel 2 — Código Termômetro (C1 a C7)
1. Clique direito → **Add Plot Pane** (cria um novo painel)
2. **Add Trace** → adicione um por um: `V(c1)`, `V(c2)`, `V(c3)`, `V(c4)`, `V(c5)`, `V(c6)`, `V(c7)`
3. Você verá 7 sinais digitais "ligando" em sequência

#### Painel 3 — Saída Binária (B2, B1, B0)
1. Clique direito → **Add Plot Pane** (mais um painel)
2. **Add Trace** → adicione: `V(b2)`, `V(b1)`, `V(b0)`
3. Você verá a contagem binária em escada: 000 → 001 → 010 → ... → 111

### Passo 3 — Ajustar Visualização

Para melhor apresentação no relatório:

- **Eixo Y:** Clique direito no eixo Y → definir range de **-0.5V a 5.5V** (evita corte nos sinais digitais)
- **Cores:** Clique direito em cada trace → escolha cores distintas para B2 (vermelho), B1 (verde), B0 (azul)
- **Grid:** Clique direito no gráfico → marque "Grid" para facilitar leitura
- **Cursores:** Use cursores (clique no nome do trace no topo) para medir os pontos exatos de transição

### Passo 4 — Capturar Screenshots

> [!IMPORTANT]
> **Capture estas 4 imagens para o relatório:**

| Screenshot | O que mostra | Onde usar no relatório |
|---|---|---|
| **Screenshot 1** | Todos os 3 painéis juntos (visão geral) | Seção "Simulação" |
| **Screenshot 2** | Zoom no Painel 2 (termômetro) | Seção "Desenvolvimento — Comparadores" |
| **Screenshot 3** | Zoom no Painel 3 (saída binária) | Seção "Resultados" |
| **Screenshot 4** | Cursores medindo ponto de transição (ex: Vin = 2,5V → 100) | Seção "Resultados" |

**Como capturar no LTspice:**
- **Tools → Copy bitmap to clipboard** (cola direto no Word/documento)
- Ou **Print Screen** e recorte a região desejada

---

## 4.3 Tabela Comparativa: Teórico vs. Simulado

> [!TIP]
> Preencha a coluna "Simulado" com os valores lidos nos cursores do LTspice. O ideal é que sejam idênticos aos teóricos (no caso de comparadores ideais, serão exatamente iguais).

| Nível | Saída Esperada | Limiar Teórico (V) | Limiar Simulado (V) | Erro (V) | Erro (%) |
|---|---|---|---|---|---|
| 0 → 1 | 000 → 001 | 0,625 | ______ | ______ | ______ |
| 1 → 2 | 001 → 010 | 1,250 | ______ | ______ | ______ |
| 2 → 3 | 010 → 011 | 1,875 | ______ | ______ | ______ |
| 3 → 4 | 011 → 100 | 2,500 | ______ | ______ | ______ |
| 4 → 5 | 100 → 101 | 3,125 | ______ | ______ | ______ |
| 5 → 6 | 101 → 110 | 3,750 | ______ | ______ | ______ |
| 6 → 7 | 110 → 111 | 4,375 | ______ | ______ | ______ |

**Fórmula do erro:**
```
Erro (V) = |Limiar_simulado - Limiar_teórico|
Erro (%) = (Erro_V / LSB) × 100 = (Erro_V / 0,625) × 100
```

> [!NOTE]
> Com comparadores **ideais**, o erro será **0V (0%)** — o resultado simulado será idêntico ao teórico. No relatório, mencione que "com componentes ideais, o erro de quantização é nulo, mas em implementação real, erros surgem devido à tolerância dos resistores e offset dos comparadores."

---

## 4.4 Análise de Erros e Discussão

### Fontes de Erro em Implementação Real

| Fonte de Erro | Causa | Impacto | Mitigação |
|---|---|---|---|
| **Tolerância dos resistores** | Resistores de 5% geram tensões de referência imprecisas | Limiares de transição deslocados | Usar resistores de 1% ou 0,1% |
| **Offset dos comparadores** | Diferença de tensão intrínseca entre entradas (+) e (−) | Erro de até ±5mV (LM339) | Usar comparadores de precisão |
| **Corrente de bias** | Corrente que o comparador drena da escada | Altera as tensões dos nós | Usar resistores de baixo valor (mais corrente na escada) |
| **Erro de quantização** | Inerente a qualquer ADC de N bits | ±½ LSB = ±0,3125V | Aumentar o número de bits |
| **Ruído** | Interferência eletromagnética | Transições instáveis | Capacitores de desacoplamento, layout cuidadoso |

### Erro de Quantização Teórico

```
Erro máximo de quantização = ± ½ LSB = ± 0,3125V

Isso significa que qualquer tensão dentro de uma faixa de 0,625V
será representada pelo mesmo código digital.

Exemplo: Vin = 1,0V e Vin = 1,2V → ambos geram saída 001
         Erro máximo: |1,249 - 0,625| / 2 = 0,3125V
```

### Vantagens e Desvantagens do ADC Flash

**Vantagens:**
- ⚡ **Velocidade máxima** — conversão em um único ciclo de clock (todos os comparadores operam em paralelo)
- 🔧 **Circuito simples** — sem lógica de controle sequencial
- 📊 **Sem necessidade de sample-and-hold** para sinais lentos

**Desvantagens:**
- 📈 **Escala mal** — para N bits, precisa de 2ᴺ−1 comparadores (ex: 8 bits = 255 comparadores!)
- 💰 **Custo** — muitos componentes para alta resolução
- ⚡ **Consumo de energia** — alto para muitos bits
- 🎯 **Precisão limitada** — depende de resistores de precisão

---

## 4.5 Material Organizado para o Relatório

Abaixo está **todo o conteúdo técnico** organizado na ordem das seções do relatório. Use como base para escrever cada seção.

---

### 📄 SEÇÃO 1 — INTRODUÇÃO

**Texto sugerido:**

> O presente trabalho tem como objetivo o projeto, simulação e análise de um Conversor Analógico-Digital (ADC) do tipo Flash com resolução de 3 bits e tensão de referência de 5V.
>
> Conversores A/D são circuitos fundamentais na eletrônica moderna, responsáveis por transformar sinais analógicos (contínuos) em sinais digitais (discretos), permitindo o processamento por sistemas digitais como microcontroladores e computadores.
>
> O conversor Flash (também chamado de conversor paralelo) é a arquitetura mais rápida de ADC existente, pois realiza a conversão em um único ciclo. Seu princípio de funcionamento baseia-se na comparação simultânea da tensão de entrada com múltiplas tensões de referência, geradas por uma escada resistiva (resistor ladder).
>
> Para uma resolução de N = 3 bits, o conversor possui 2³ = 8 níveis de quantização, utilizando 7 comparadores e 8 resistores. A saída dos comparadores gera um código termômetro de 7 bits, que é então convertido em código binário de 3 bits por um circuito codificador.

---

### 📄 SEÇÃO 2 — FUNDAMENTAÇÃO TEÓRICA

**Conceitos a incluir (com as definições):**

**Resolução:**
> A resolução de um ADC é definida pelo número de bits N da saída digital. Com N bits, o conversor pode representar 2ᴺ níveis distintos. No caso de 3 bits: 2³ = 8 níveis (000 a 111).

**LSB (Least Significant Bit):**
> O LSB é o menor incremento de tensão que o conversor pode distinguir, calculado por:
> LSB = Vref / 2ᴺ = 5V / 8 = 0,625V

**Código Termômetro:**
> O código termômetro é uma representação unária onde o número de bits "1" consecutivos, contados a partir do bit menos significativo, indica o nível de quantização. O nome vem da analogia com um termômetro de mercúrio, onde a "coluna" de 1s sobe conforme a tensão aumenta.

**Escada Resistiva:**
> Uma cadeia de resistores iguais em série entre Vref e GND, criando divisores de tensão que geram as tensões de referência para os comparadores.

**Comparador Analógico:**
> Circuito que compara duas tensões de entrada e produz uma saída digital: nível alto se V(+) > V(−), nível baixo caso contrário.

---

### 📄 SEÇÃO 3 — PARÂMETROS DO PROJETO

**Incluir a tabela:**

| Parâmetro | Símbolo | Valor |
|---|---|---|
| Resolução | N | 3 bits |
| Níveis de quantização | 2ᴺ | 8 |
| Tensão de referência | Vref | 5V |
| LSB | Vref/2ᴺ | 0,625V |
| Comparadores | 2ᴺ−1 | 7 |
| Resistores na escada | 2ᴺ | 8 × 1kΩ |
| Faixa de entrada | — | 0V a 5V |
| Erro de quantização | ±½ LSB | ±0,3125V |

---

### 📄 SEÇÃO 4 — DESENVOLVIMENTO

**4.1 — Escada Resistiva** (usar conteúdo da Etapa 1)
- Diagrama do circuito
- Cálculo das 7 tensões de referência
- Verificação por divisão de tensão

**4.2 — Comparadores** (usar conteúdo da Etapa 2)
- Princípio de funcionamento
- Diagrama de ligação (Vin em todos os +, referências nos −)
- Tabela do código termômetro

**4.3 — Codificador** (usar conteúdo da Etapa 3)
- Tabela verdade completa
- Derivação das equações booleanas
- Verificação exaustiva dos 8 níveis
- Diagrama de portas lógicas

**4.4 — Esquemático Completo**
```
 Vin ──→ [7 Comparadores] ──→ [Codificador] ──→ B2 B1 B0
              ↑
         [Escada Resistiva]
              ↑
           Vref = 5V
```

---

### 📄 SEÇÃO 5 — SIMULAÇÃO

**Incluir:**
- Ferramenta: LTspice (versão XVII ou posterior)
- Tipo de simulação: Análise transitória (.tran)
- Sinal de entrada: rampa PWL de 0V a 5V em 10ms
- Passo de simulação: 1µs

**Screenshots a incluir:**
1. Gráfico completo (Vin + termômetro + saída binária)
2. Zoom na saída binária mostrando a escada
3. Medição com cursores nos pontos de transição

---

### 📄 SEÇÃO 6 — RESULTADOS E DISCUSSÃO

**Incluir:**
- Tabela comparativa teórico vs. simulado (seção 4.3 deste documento)
- Análise do erro (idealmente 0% com comparadores ideais)
- Discussão sobre fontes de erro em implementação real
- Vantagens e desvantagens do ADC Flash (seção 4.4 deste documento)

**Texto sugerido para resultados:**

> Os resultados da simulação demonstram que o conversor A/D Flash projetado opera conforme esperado. A saída binária de 3 bits (B2, B1, B0) apresenta uma resposta em escada perfeitamente alinhada com os valores teóricos calculados.
>
> Os limiares de transição observados na simulação coincidem exatamente com os valores teóricos (0,625V, 1,250V, ..., 4,375V), resultando em erro de 0% em todas as transições. Este resultado é esperado para uma simulação com componentes ideais.
>
> Em uma implementação real, espera-se que erros da ordem de ±1% a ±5% surjam devido à tolerância dos resistores, offset dos comparadores e ruído do circuito.

---

### 📄 SEÇÃO 7 — CONCLUSÃO

**Texto sugerido:**

> O conversor A/D Flash de 3 bits com Vref = 5V foi projetado, simulado e validado com sucesso utilizando o software LTspice.
>
> O projeto envolveu três blocos funcionais: uma escada resistiva de 8 resistores de 1kΩ (gerando 7 tensões de referência espaçadas em 0,625V), 7 comparadores analógicos operando em paralelo, e um codificador combinacional que converte o código termômetro de 7 bits em código binário de 3 bits.
>
> As equações booleanas do codificador foram derivadas analiticamente (B2 = C4, B1 = C2·C̄4 + C6, B0 = C1·C̄2 + C3·C̄4 + C5·C̄6 + C7) e verificadas exaustivamente para todos os 8 níveis de quantização.
>
> Os resultados da simulação confirmaram o funcionamento correto do conversor, com a saída digital formando uma escada binária crescente (000 a 111) em resposta a uma rampa de tensão de 0V a 5V na entrada.
>
> O conversor Flash é a arquitetura mais rápida de ADC, mas seu principal limitante é a escalabilidade: para N bits, requer 2ᴺ−1 comparadores, tornando-se impraticável para resoluções acima de 8 bits.

---

### 📄 SEÇÃO 8 — REFERÊNCIAS

```
[1] SEDRA, A. S.; SMITH, K. C. Microeletrônica. 5ª ed. São Paulo: Pearson, 2007.

[2] TOCCI, R. J.; WIDMER, N. S.; MOSS, G. L. Sistemas Digitais: Princípios
    e Aplicações. 12ª ed. São Paulo: Pearson, 2018.

[3] RAZAVI, B. Principles of Data Conversion System Design.
    Wiley-IEEE Press, 1995.

[4] Texas Instruments. LM339 Quad Differential Comparator — Datasheet.
    Disponível em: https://www.ti.com/product/LM339

[5] Texas Instruments. SN74LS148 8-Line to 3-Line Priority Encoder — Datasheet.
    Disponível em: https://www.ti.com/product/SN74LS148

[6] Analog Devices. LTspice Simulator.
    Disponível em: https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html
```

---

## 4.6 Checklist Final de Entrega

### Materiais Produzidos

- [x] Etapa 1 — Cálculos fundamentais e escada resistiva
- [x] Etapa 2 — 7 comparadores e código termômetro
- [x] Etapa 3 — Codificador (equações booleanas + verificação)
- [x] Etapa 4 — Simulação final e material para relatório
- [x] Netlist LTspice completa (`adc_flash_3bits_5V.cir`)

### Para Você Completar

- [ ] Rodar a simulação no LTspice
- [ ] Capturar os 4 screenshots descritos na seção 4.2
- [ ] Preencher a tabela teórico vs. simulado (seção 4.3)
- [ ] Montar o relatório usando os textos e materiais fornecidos
- [ ] Revisar e formatar o documento final

---

## 🎯 Resumo de Todos os Arquivos do Projeto

| Arquivo | Descrição | Etapa |
|---|---|---|
| `escada_resistiva.cir` | Somente a escada (verificação inicial) | Etapa 1 |
| `adc_flash_comparadores.cir` | Escada + comparadores (código termômetro) | Etapa 2 |
| `adc_flash_3bits_5V.cir` | **Circuito completo** (escada + comparadores + codificador) | Etapa 3/4 |

> [!TIP]
> Para o relatório, use apenas o arquivo `adc_flash_3bits_5V.cir` — ele contém tudo. Os outros dois servem como etapas intermediárias de desenvolvimento.

---

**Projeto concluído! 🎉** Todas as 4 etapas foram finalizadas. Bom relatório! 📝
