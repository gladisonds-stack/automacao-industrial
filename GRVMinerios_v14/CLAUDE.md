# CLAUDE.md — GRVMinerios_v14

Referência completa gerada a partir da leitura do código SCL, UDTs e tags do
projeto. Todos os fatos aqui são derivados diretamente dos arquivos fontes.

---

## 1. Visão Geral do Projeto

Planta de britagem de minérios com circuito fechado de trituração. O CLP
controla três linhas de produção independentes com exclusão mútua entre os
britadores cônicos (02/03) e o VSI (04), imposta por limitação de subestação.

Equipamentos controlados:
- **3 britadores**: Cônicos 02 e 03 (FSM 6 estados) + VSI 04 (FSM 7 estados)
- **3 peneiras vibratórias**: PNR02, PNR03, PNR04
- **14 transportadores de correia**: TRP06 a TRP19
- **3 calhas vibratórias**: CVR01 (Linha A / BRT02), CVR02 (Linha B / BRT03),
  CVR03 (Linha C / BRT04-VSI)
- **3 detectores de metal**: DTC01 (TC06), DTC02 (TC12), DTC03 (TC16)
- **Sirene industrial** com FSM própria de 5 estados
- **Torre luminosa** tri-cor
- **Sistemas auxiliares** por britador: lubrificação, radiador, resistência de
  aquecimento, hidroset (analógico), ventilador

---

## 2. Hardware

### CLP

| Atributo  | Valor                     |
|-----------|---------------------------|
| Modelo    | Siemens S7-1214C DC/DC/DC |
| ID        | 20A1                      |
| Linguagem | SCL, TIA Portal           |

### Módulos de I/O

| Slot  | Entradas (%I)    | Saídas (%Q)    | Conteúdo principal                                    |
|-------|------------------|----------------|-------------------------------------------------------|
| 20A1  | %I0.0 – %I1.7   | %Q0.0 – %Q1.7 | Mesa controle, cmds britadores/PV/TRP06-09, VERDE TL  |
| 20A2  | %I2.0 – %I3.7   | %Q2.0 – %Q3.7 | TRP10-19 cmd, calhas cmd, emerg. painel sec., AMARELO/VERMELHO TL |
| 20A3  | %I4.0 – %I5.7   | %Q4.0 – %Q5.7 | Térmicos TRP06-12, falhas calhas, HABILITA/ACIONA calhas, BRT02-03 |
| 20A4  | %I6.0 – %I7.7   | %Q6.0 – %Q7.7 | Feedbacks PV/TRP/calhas ligados, LEDs detect., sirene, hidroset |
| 20A5  | %I8.0 – %I9.7   | %Q8.0 – %Q9.7 | Feedbacks calhas ligadas, BRT02-03 feedback, lube 01  |
| 20A6  | %I10.0 – %I11.7 | %Q10.0 – %Q11.7 | Lube, hidroset, ventiladores, resistências (reserva Q) |
| 20A7  | %I12.0 – %I13.7 | %Q12.0 – %Q13.7 | BRT04-VSI, detectores metal, emergências externas (reserva Q) |
| 20A8  | %I14.0 – %I15.7 | %Q14.0 – %Q15.7 | Monitoramento óleo (temp, nível, filtro), hidroset subindo/descendo (reserva Q) |
| 20A9  | %IW208, %IW210  | —              | Entradas analógicas: pressão hidroset 01 e 02 (Int, 4–20mA) |

**Escala analógica**: `NORM_X(MIN:=5530, MAX:=27648)` → `SCALE_X(MIN:=0.0, MAX:=25.0)`
(5530 = 4mA / 27648 = 20mA → saída em bar/MPa, escala 0–25)

### HMI

| Atributo             | Valor               |
|----------------------|---------------------|
| Modelo               | Siemens KTP900 Basic PN |
| Conexão              | HMI_Connection_1    |
| Ciclo de atualização | 100 ms              |

---

## 3. Convenções de Nomenclatura

### Prefixos de variáveis

| Prefixo | Significado                        | Exemplo                              |
|---------|------------------------------------|--------------------------------------|
| `Par`   | Parâmetro configurável (RETAIN)    | `ParPrepTime`, `ParSdaTime`          |
| `St`    | Status (somente leitura pelo FB)   | `StFaultGlobal`, `StHidrosetPress`   |
| `Cmd`   | Comando pulsado (IHM → CLP)        | `CmdOn`, `CmdOff`, `CmdFalhaReset`   |
| `Sda`   | Variável de saída/temporização interna da FSM | `SdaF`, `SdaF_2`, `SdaTime` |
| `En`    | Enable — permissivo de entrada da FSM | `EnHabilita`, `EnFLigado`         |
| `EL`    | Elemento — struct interna do equipamento | `BRT02.EL.StHidrosetPress`    |
| `tmp`   | Variável temporária (VAR_TEMP)     | `tmpFalha`, `tmpSireneOk`            |

### Códigos de equipamento

| Código | Tipo                     | Instâncias              |
|--------|--------------------------|-------------------------|
| `BR`   | Britador (parâmetros)    | BR02, BR03, BR04        |
| `BRT`  | Britador (instâncias FB) | BRT02, BRT03, BRT04     |
| `TRP`  | Transportador            | TRP06 … TRP19           |
| `CVR`  | Calha Vibratória         | CVR01, CVR02, CVR03     |
| `PNR`  | Peneira Vibratória       | PNR02, PNR03, PNR04     |
| `DTC`  | Detector de Metal        | DTC01, DTC02, DTC03     |

### Sufixos de blocos

| Sufixo | Tipo de bloco                                     |
|--------|---------------------------------------------------|
| `_FB`  | Function Block (stateful, tem DB de instância)    |
| `_FC`  | Function (stateless, sem DB)                      |
| `_DB`  | Data Block (global ou instância)                  |
| `_OB`  | Organization Block (chamado pelo SO do S7)        |

### Saídas físicas da FSM

| Variável  | Função                                          |
|-----------|-------------------------------------------------|
| `SdaF`    | Motor 1 (bomba de óleo / lubrificação / acionamento principal simples) |
| `SdaF_2`  | Motor 2 (rotor principal do britador)           |
| `SdaFalhaReset` | Pisca 2Hz — indica falha aguardando reset |
| `SdaAquecedor`  | Resistência de aquecimento do óleo       |
| `SdaVentoinha`  | Ventoinha do radiador de óleo            |
| `SdaTurbinaPust`| Turbina de pressurização (dust seal)     |

---

## 4. Estrutura de Pastas e Responsabilidade de Cada Bloco

```
GRVMinerios_v14/
├── PLC_20A1/Program_Blocks/
│   ├── Main_OB1.scl              ← Ciclo principal (ordem de execução documentada abaixo)
│   ├── Erros_FC2.scl             ← Mapeia falhas → StErros[0..4] no IHM_DB
│   ├── CONTROLE_STATUS_DB7.scl   ← DB global: StOnGlobal, StStartingGlobal, StFaultGlobal,
│   │                                StFalhaDetectores, StEmergenciaOk, StSireneOk
│   ├── IHM_DB6.scl               ← DB RETAIN: parâmetros, status e comandos da HMI
│   ├── BRITADORES/
│   │   ├── BRITADORES_FB2.scl    ← FB2: gerencia BRT02 (EL_6St), BRT03 (EL_6St), BRT04 (EL_7St)
│   │   └── BRITADORES_DB2.scl    ← DB2: instância (BRT02, BRT03 como EL_6St_Fct; BRT04 como EL_7St_Fct)
│   ├── CALHA_VIBRATORIA/
│   │   ├── CALHA_VIBRATORIA_FB7.scl ← FB7: CVR01, CVR02, CVR03 (cada um EL_4St_Fct)
│   │   └── CALHA_VIBRATORIA_DB5.scl
│   ├── DETECTORES/
│   │   └── Detectores_FC4.scl    ← FC4: DTC01/02/03 (lógica NF + reset anti-memória)
│   ├── GERAL/
│   │   ├── MAQ_ESTADOS/
│   │   │   ├── EL_4St_Fct_FB3.scl    ← FSM 4 estados: TRP, PNR, CVR (motores simples)
│   │   │   ├── EL_6St_BRT_Fct_FB9.scl ← FSM 6 estados: britadores cônicos (02 e 03)
│   │   │   └── EL_7St_Fct_FB1.scl    ← FSM 7 estados: britador VSI 04
│   │   ├── SIRENE/
│   │   │   ├── SIRENE_FB10.scl   ← FB10: FSM 5 estados da sirene industrial
│   │   │   └── SIRENE_DB9.scl    ← DB9: instância da sirene (+ exposição de ParTempoToque)
│   │   └── SYS/
│   │       ├── Sys_Safety_FC3.scl ← FC3: emergências → GERAL_EMERGENCIA_OK + torre luminosa
│   │       ├── Sys_FB4.scl       ← FB4: acumula tempo de ciclo → OutPulse1s, OutCycleTime
│   │       └── Sys_DB3.scl       ← DB3: instância do Sys_FB4 (OutPulse1s, OutCycleTime)
│   ├── PARAMETROS/
│   │   └── SincParam_FC5.scl     ← FC5: IHM_DB.Parametros → Par* dos DBs de controle (1x/s)
│   ├── PENEIRAS/
│   │   ├── PENEIRAS_FB6.scl      ← FB6: PNR02, PNR03, PNR04 (cada um EL_4St_Fct)
│   │   └── PENEIRAS_DB4.scl
│   └── TRANSPORTADORES/
│       ├── TRANSPORTADORES_FB5.scl ← FB5: TRP06..TRP19 (cada um EL_4St_Fct)
│       └── TRANSPORTADORES_DB1.scl
├── PLC_Tags/
│   ├── ENTRADAS_132.tag          ← Mapeamento completo %I (132 entradas)
│   └── SAIDAS_128.tag            ← Mapeamento completo %Q (128 saídas)
├── PLC_DataTypes/
│   ├── MAQ_ESTADOS/
│   │   ├── EL_4St_UDT.udt        ← Struct interna da EL_4St_Fct_FB3
│   │   ├── EL_6St_BRT_UDT.udt    ← Struct interna da EL_6St_BRT_Fct_FB9
│   │   └── EL_7St_UDT.udt        ← Struct interna da EL_7St_Fct_FB1
│   ├── STATUS/
│   │   └── UDT_STATUS_*.udt      ← Structs de status por categoria (PLANTA, BRITADORES, etc.)
│   └── TIME/
│       └── UDT_TIME_*.udt        ← Structs de tempo por categoria
└── HMI_KTP900/
    ├── HMI_Tags/Default_tag_table_218.tag
    └── Screens/                  ← Documentação das telas (a preencher)
```

---

## 5. Padrões Obrigatórios de Código

### Ordem de execução no Main_OB1 (crítica)

```
Network 1:  RUNTIME + Sys_DB.Sys    → gera OutPulse1s e OutCycleTime
Network 2:  Sys_Safety              → calcula GERAL_EMERGENCIA_OK e torre luminosa
Network 3:  SincParam (a cada 1s)   → propaga IHM_DB.Parametros → Par* dos FBs
Network 4:  Erros                   → consolida falhas em StErros[0..4] e StFaultGlobal
Network 5:  SIRENE_DB.SIRENE        → avança FSM da sirene, gera StSireneOk
Network 6:  Detectores              → atualiza StFalhaDetectores.DTC01/02/03
Network 7:  TRANSPORTADORES_DB.TRANSPORTADORES
Network 8:  CALHA VIBRATÓRIA_DB.CALHA VIBRATÓRIA
Network 9:  PENEIRAS_DB.PENEIRAS
Network 10: BRITADORES_DB.BRITADORES
```

**Atenção**: `Sys_Safety` executa na posição 2, **não ao final**. Ela escreve em
`GERAL_EMERGENCIA_OK` (%M1.1) que os FBs de processo lêem nos seus permissivos.

### Padrão de implementação de todos os FBs de processo

```
REGION "1. MAP_IN"
    // Snapshot das entradas — lidas UMA VEZ no início do ciclo
    #EL.EnFDispFalha := "TÉRMICO TAG";
    #EL.EnFLigado    := "FEEDBACK TAG";
    ...
END_REGION

REGION "2. PROCESS"
    // Cálculo de permissivos + tratamento de botão toggle pulsado
    #permissivos := GERAL_EMERGENCIA_OK AND NOT StFaultGlobal.EQUIPAMENTO;
    #EL.EnHabilita := (MANUAL AND #permissivos) OR (AUTO AND #permissivos AND downstream.StOn);
    // Botão toggle:
    IF #EL.StBotao AND NOT #EL.StAnteriorBotao THEN
        IF #EL.StOff THEN #EL.CmdOn := TRUE; ELSE #EL.CmdOff := TRUE; END_IF;
    END_IF;
    #EL.StAnteriorBotao := #EL.StBotao;
END_REGION

REGION "3. CORE"
    #INSTANCIA();   // Chama a FSM (EL_4St, EL_6St ou EL_7St)
END_REGION

REGION "4. MAP_OUT"
    "SAÍDA FÍSICA" := #INSTANCIA.EL.SdaF;
END_REGION
```

### Loop REPEAT na FSM

Todas as FSMs usam `REPEAT...UNTIL` para processar transições em cascata no
mesmo ciclo de scan (evita atraso de 1 ciclo por transição). A condição de saída
é: nenhuma transição pendente. Não quebrar este padrão.

### Limpeza obrigatória de comandos (fim de cada FB)

```scl
#EL.CmdOn := 0;
#EL.CmdOff := 0;
#EL.CmdFalhaReset := 0;
```
Comandos são **sempre** zerados ao fim do ciclo. Eles funcionam como pulsos
de 1 scan. Nunca manter `CmdOn/Off` = TRUE entre ciclos.

### Snapshot de entradas (Region Init)

Entradas físicas são lidas para variáveis `tmp*` no início do FB, antes do REPEAT.
Nunca ler `%I` diretamente dentro dos estados da FSM.

### SincParam — regra do valor > T#0ms

`SincParam_FC5` só propaga um parâmetro se o novo valor for **diferente do atual**
E **maior que T#0ms**. Isso impede que um campo zerado na IHM sobrescreva um valor
válido no DB de controle. Nunca zerá um `Par*` via HMI esperando que seja propagado.

---

## 6. Relação entre FBs, DBs e UDTs

```
EL_4St_UDT       ──► campo EL dentro de EL_4St_Fct_FB3
EL_6St_BRT_UDT   ──► campo EL dentro de EL_6St_BRT_Fct_FB9
EL_7St_UDT       ──► campo EL dentro de EL_7St_Fct_FB1

BRITADORES_FB2 (instância BRITADORES_DB2):
  VAR:
    BRT02 : "EL_6St_BRT_Fct"  → usa EL_6St_BRT_UDT internamente
    BRT03 : "EL_6St_BRT_Fct"  → idem
    BRT04 : "EL_7St_Fct"      → usa EL_7St_UDT internamente

PENEIRAS_FB6 (instância PENEIRAS_DB4):
  VAR:
    PNR02, PNR03, PNR04 : "EL_4St_Fct"  → usa EL_4St_UDT internamente

TRANSPORTADORES_FB5 (instância TRANSPORTADORES_DB1):
  VAR:
    TRP06..TRP19 : "EL_4St_Fct"  → usa EL_4St_UDT internamente

CALHA_VIBRATORIA_FB7 (instância CALHA_VIBRATORIA_DB5):
  VAR:
    CVR01, CVR02, CVR03 : "EL_4St_Fct"  → usa EL_4St_UDT internamente

SIRENE_FB10 (instância SIRENE_DB9):
  lê    → IHM_DB.Parametros.Sirene.{ParTempoToque, ParTempoMaxPartida, ParTempoValidade}
  lê    → CONTROLE_STATUS_DB.StStartingGlobal.* (todos equipamentos)
  lê    → CONTROLE_STATUS_DB.StOnGlobal.* (todos equipamentos)
  escreve → IHM_DB.Status.Sirene.{StStep, StTempoRestante, StValidadeRestante}
  escreve → IHM_DB.Status.Sirene.StSda (bool)
  saída → CmdSireneFisica → "SIRENE 220VCA" (%Q8.1) via Main_OB1

CONTROLE_STATUS_DB7 (NON_RETAIN):
  StSomaFalhas    : Struct{StFalhaBritadores, StFalhaPeneiras, ...} ← calculado pelo FB de cada grupo
  StOnGlobal      : UDT_STATUS_PLANTA  ← escrito pelos FBs de processo
  StStartingGlobal: UDT_STATUS_PLANTA  ← escrito pelos FBs de processo
  StFaultGlobal   : UDT_STATUS_PLANTA  ← escrito pelos FBs + Erros_FC2
  StFalhaDetectores: UDT_STATUS_DETECTORES ← escrito por Detectores_FC4
  StEmergenciaOk  : Bool               ← escrito por Sys_Safety_FC3
  StSireneOk      : Bool               ← escrito por SIRENE_DB.SIRENE via Main_OB1

IHM_DB6 (NON_RETAIN + RETAIN Parametros):
  Comandos.CmdTesteSirene       : Bool  ← pulso HMI → SIRENE
  Comandos.CmdDesabilitaSincParam: Bool ← trava manutenção da sincronização
  Status.StErros[0..4]          : Array of Word  ← escrito por Erros_FC2
  Status.Sirene.*               : Struct ← escrito por SIRENE_FB10
  Parametros.Tempos_Processo.*  : RETAIN ← lido por SincParam e propagado

SincParam_FC5:
  lê   → IHM_DB.Parametros.Tempos_Processo.*
  escreve → Par* diretamente nos campos EL.Par* de cada instância nos DBs
  frequência: 1x por segundo (gateado por Sys_DB.OutPulse1s no Main_OB1)
```

---

## 7. Estados das FSMs e Códigos de Diagnóstico

### EL_4St (TRP, PNR, CVR) — 4 estados internos

| StStep | Categoria | Descrição                                      |
|--------|-----------|------------------------------------------------|
| 0      | Operação  | Desligado em repouso                           |
| 1      | Operação  | Iniciando sequência (aguardando sirene)        |
| 2      | Operação  | Aguardando status OK da sirene                 |
| 3      | Operação  | Sirene OK, aguardando feedback do motor        |
| 4      | Operação  | Em operação normal                             |
| 5      | Aviso     | Desligado por comando                          |
| 6      | Aviso     | Desligado por perder permissivo                |
| 7      | Aviso     | Partida bloqueada: falta permissivo            |
| 8      | Falha     | Falha de segurança/hardware (térmico/disjuntor)|
| 9      | Falha     | Falha de partida: watchdog da sirene           |
| 10     | Falha     | Falha de partida: timeout do motor             |
| 11     | Falha     | Falha em operação: perda de feedback mecânico  |

**Saída física** (`SdaF`): `(StOn OR (StStarting AND StSireneOk)) AND NOT falha AND NOT StFault AND habilita`

### EL_6St_BRT (BRT02, BRT03 — Cônicos) — 6 estados internos

| StStep | Categoria | Descrição                                             |
|--------|-----------|-------------------------------------------------------|
| 0      | Operação  | Desligado em repouso                                  |
| 1      | Operação  | Sirene e preparação da unidade hidráulica             |
| 2      | Operação  | Aguardando partida do rotor principal                 |
| 3      | Operação  | Em operação normal                                    |
| 4      | Operação  | Desligando: resfriamento e inércia ativos (Idling)    |
| 8      | Aviso     | Desligado por comando                                 |
| 9      | Aviso     | Desligado por perder permissivo                       |
| 10     | Aviso     | Partida bloqueada: falta permissivo                   |
| 11     | Falha     | Falha de segurança/hardware (térmicos)                |
| 12     | Falha     | Timeout de aquecimento/pressão de óleo                |
| 13     | Falha     | Timeout do motor principal                            |
| 14     | Falha     | Nível baixo ou fervura (trip térmico)                 |
| 15     | Falha     | Falha em operação: perda de feedback mecânico         |

**Proteção de inércia (Idling)**: ao receber `CmdOff` ou perder permissivo durante
`StStarting` ou `StOn`, o britador vai para `StIdling` (não desliga imediatamente).
**Flying Start**: durante `StIdling`, se `CmdOn` chegar com permissivo → volta a `StPreparing`.

**Lube Overrun**: se o britador entrar em falha enquanto o rotor gira, a bomba de
óleo (`SdaF`) continua por `ParSdaDelayIdling` via contador de overrun. Não desliga.

### EL_7St (BRT04 — VSI) — 7 estados internos

| StStep | Categoria | Descrição                                           |
|--------|-----------|-----------------------------------------------------|
| 0      | Operação  | Desligado em repouso                                |
| 1      | Operação  | Iniciando sequência (sirene)                        |
| 2      | Operação  | Aguardando status OK da sirene                      |
| 3      | Operação  | Sirene OK, partindo lubrificação                    |
| 4      | Operação  | Lubrificação OK, executando pré-lubrificação        |
| 5      | Operação  | Partindo motor principal (Rotor VSI)                |
| 6      | Operação  | Em operação normal                                  |
| 7      | Operação  | Pós-lubrificação ativa / inércia (Idling)           |
| 8      | Aviso     | Desligado por comando                               |
| 9      | Aviso     | Desligado por perder permissivo                     |
| 10     | Aviso     | Partida bloqueada: falta permissivo                 |
| 11     | Falha     | Falha de segurança/hardware                         |
| 12     | Falha     | Watchdog da sirene                                  |
| 13     | Falha     | Timeout da lubrificação                             |
| 14     | Falha     | Timeout do rotor VSI                                |
| 15     | Falha     | Perda de feedback mecânico                          |

### Sirene (SIRENE_FB10) — 5 estados + 7 códigos diagnóstico

| StStep | Categoria | Descrição                                         |
|--------|-----------|---------------------------------------------------|
| 0      | Operação  | Desligado em repouso                              |
| 1      | Operação  | Teste manual via HMI ativo                        |
| 2      | Operação  | Toque físico ativo (CmdSireneFisica = 1)          |
| 3      | Operação  | Sirene OK, aguardando feedback (StSireneOk = 1)   |
| 4      | Operação  | Em operação normal (janela de 30min válida)       |
| 5      | Aviso     | Desligado por comando / fim de teste              |
| 6      | Aviso     | Desligado por perder permissivo                   |
| 7      | Aviso     | Partida bloqueada: falta permissivo               |
| 8      | Falha     | Falha de segurança/hardware                       |
| 9      | Falha     | Aborto crítico durante o aviso                    |
| 10     | Falha     | Timeout do motor (watchdog)                       |
| 11     | Falha     | Perda de feedback mecânico                        |

**`StSireneOk`** fica TRUE apenas durante `StFeedback (3)` e `StOn (4)`.
Os FBs de equipamento só avançam para "ligar motor" quando `StSireneOk = 1`.

---

## 8. Regras de Sequenciamento e Interlocks

### Exclusão mútua entre britadores (limitação de subestação)

```
BRT02 habilitado: GERAL_EMERGENCIA_OK AND NOT StOnGlobal.BRT04
BRT03 habilitado: GERAL_EMERGENCIA_OK AND NOT StOnGlobal.BRT04
BRT04 habilitado: GERAL_EMERGENCIA_OK AND NOT StOnGlobal.BRT02 AND NOT StOnGlobal.BRT03
```
BRT02/BRT03 (cônicos) e BRT04 (VSI) são **mutuamente exclusivos**.

### Intertravamentos por linha (modo automático)

**Linha A — BRT02 (Cônico)**

| Equipamento | Habilita AUTO quando downstream estiver `StOn` | Permissivo adicional             |
|-------------|------------------------------------------------|----------------------------------|
| TRP06       | BRT02 StOn                                     | NOT BRT02 fault, NOT DTC01       |
| BRT02       | TRP07 StOn                                     | NOT BRT04 StOn                   |
| TRP07       | PNR02 StOn                                     | NOT BRT02 fault, NOT PNR02 fault |
| CVR01       | TRP06 StOn                                     | NOT DTC01, NOT TRP06 fault, NOT BRT02 fault |

**Linha B — BRT03 (Cônico)**

| Equipamento | Habilita AUTO quando downstream estiver `StOn` | Permissivo adicional             |
|-------------|------------------------------------------------|----------------------------------|
| BRT03       | TRP10 StOn                                     | NOT BRT04 StOn                   |
| PNR02       | BRT03 StOn AND TRP09 StOn                      | NOT TRP09 fault, NOT BRT03 fault |
| CVR02       | TRP12 StOn                                     | NOT DTC02, NOT TRP12 fault, NOT BRT03 fault |

**Linha C — BRT04 (VSI)**

| Equipamento | Habilita AUTO quando downstream estiver `StOn` | Permissivo adicional             |
|-------------|------------------------------------------------|----------------------------------|
| BRT04       | TRP14 StOn                                     | NOT BRT02 StOn, NOT BRT03 StOn   |
| PNR03       | BRT04 StOn AND TRP13 StOn                      | NOT TRP13 fault, NOT BRT04 fault |
| CVR03       | TRP16 StOn                                     | NOT DTC03, NOT TRP16 fault, NOT BRT04 fault |

**Peneira de Expedição**

| Equipamento | Habilita AUTO quando downstream estiver `StOn` | Permissivo adicional                   |
|-------------|------------------------------------------------|----------------------------------------|
| PNR04       | TRP17 StOn AND TRP18 StOn AND TRP19 StOn       | NOT TRP17/18/19 fault (intertrav. triplo) |

### Lógica de Detectores de Metal

- Sensor NF: 0 = metal detectado (ou sensor com problema)
- Detecção trava o bit na memória (`StFalhaDetectores.DTC0X := TRUE`)
- Reset exige: `(RESET MESA DE CONTROLE OR RESET LOCAL DTC0X)` **E** sensor físico = 1
- Efeito: bloqueia o permissivo da calha e transportador da linha correspondente
- **Não para a linha automaticamente** — bloqueia nova partida e impede habilita

### Comportamento da Emergência (GERAL_EMERGENCIA_OK)

```
Qualquer emergência ativa → GERAL_EMERGENCIA_OK := FALSE (imediato)
Todos os emergências OK → GERAL_EMERGENCIA_OK permanece FALSE (memória)
Para resetar: todas as emergências OK E RESET MESA DE CONTROLE pressionado
```
`GERAL_EMERGENCIA_OK` é um bit de memória (%M1.1) — **não reseta sozinho**.

### Torre Luminosa (lógica da Sys_Safety)

| Condição                                    | VERDE | AMARELO    | VERMELHO       |
|---------------------------------------------|-------|------------|----------------|
| Sistema OK, sem falhas                      | ON    | OFF        | OFF            |
| Falha de processo OU aguardando rearme      | OFF   | OFF        | Pisca 1Hz      |
| Emergência física ativa                     | OFF   | OFF        | ON (estático)  |
| Sirene tocando (CmdSireneFisica)            | —     | Pisca 1Hz  | —              |

### Valores padrão dos parâmetros (definidos em IHM_DB6.scl)

| Parâmetro                         | BR02    | BR03    | BR04    | TRP/PNR/CVR |
|-----------------------------------|---------|---------|---------|-------------|
| ParPrepTime                       | 5 min   | 5 min   | 5 min   | —           |
| ParPreLubeTime                    | —       | —       | 1 min   | —           |
| ParSdaTime                        | 15 s    | 15 s    | 25 s    | 5 s         |
| ParSdaDelayFalha                  | 2 s     | 2 s     | 2 s     | 2 s         |
| ParSdaDelayIdling                 | 3 min   | 3 min   | 5 min   | —           |
| ParTempoToque (Sirene)            | 60 s    |         |         |             |
| ParTempoMaxPartida (Sirene)       | 5 min   |         |         |             |
| ParTempoValidade (Sirene)         | 30 min  |         |         |             |

---

## 9. Mapeamento de Erros (StErros[0..4])

`IHM_DB.Status.StErros` é `Array[0..4] of Word` (5 words = 80 bits). Apenas [0..2] usados.

**StErros[0]** — Peneiras e Transportadores (parte 1):
```
X0=PNR02  X1=PNR03  X2=PNR04  X3=TRP06  X4=TRP07  X5=TRP08  X6=TRP09  X7=TRP10
X8=TRP11  X9=TRP12  X10=TRP13 X11=TRP14 X12=TRP15 X13=TRP16 X14=TRP17 X15=TRP18
```

**StErros[1]** — Transportadores e auxiliares:
```
X0=TRP19  X1=CVR01  X2=CVR02  X3=CVR03  X4=BRT02  X5=BRT03
X6=FALHA LUB01  X7=FALHA RAD01  X8=FALHA RES01  X9=FALHA HST01
X10=FALHA VNT01  X11=FALHA LUB02  X12=FALHA RAD02  X13=FALHA RES02
X14=FALHA HST02  X15=FALHA VNT02
```

**StErros[2]** — Britador 04, Emergências, Detectores:
```
X0=BRT04  X1=BRT04-M2  X2=EMERG MESA  X3=EMERG PN SEC  X4=EMERG PN VSI
X5=EMERG PN CÔNICO  X6=EMERG EXT BRT02  X7=EMERG EXT BRT03  X8=EMERG EXT VSI04
X9=DTC01(TC06)  X10=DTC02(TC12)  X11=DTC03(TC16)
```

**StErros[3] e [4]**: reservados, não mapeados.

---

## 10. Tags Importantes de Referência

### Emergências (entradas NF — 0 = emergência ativa)

| Tag                                   | Endereço |
|---------------------------------------|----------|
| EMERGÊNCIA MESA DE CONTROLE           | %I0.0    |
| EMERGÊNCIA PAINEL SECUNDÁRIO          | %I3.5    |
| EMERGÊNCIA PAINEL BRITADOR VSI        | %I12.0   |
| EMERGÊNCIA PAINEL BRITADOR CÔNICO     | %I12.6   |
| EMERGÊNCIA EXTERNA BRITADOR CÔNICO 02 | %I13.2   |
| EMERGÊNCIA EXTERNA BRITADOR CÔNICO 03 | %I13.3   |
| EMERGÊNCIA EXTERNA BRITADOR VSI 04    | %I13.4   |

### Modo de operação e reset

| Tag                     | Endereço |
|-------------------------|----------|
| SELEÇÃO MANUAL          | %I0.2    |
| SELEÇÃO AUTOMÁTICO      | %I0.3    |
| RESET MESA DE CONTROLE  | %I0.1    |
| RESET SEGURANÇA (saída) | %Q5.5    |

### Status global (via DB)

| Campo                                    | Tipo | Significado                      |
|------------------------------------------|------|----------------------------------|
| `CONTROLE_STATUS_DB.StFaultGlobal.GLOBAL`| Bool | Qualquer falha ativa na planta   |
| `CONTROLE_STATUS_DB.StSireneOk`          | Bool | Sirene operacional                |
| `CONTROLE_STATUS_DB.StEmergenciaOk`      | Bool | Igual a GERAL_EMERGENCIA_OK (%M1.1) |
| `IHM_DB.Status.StErros[0..4]`           | Word | Bits de falha mapeados            |
| `IHM_DB.Status.Sirene.StStep`           | Int  | Estado atual da sirene            |

### Torre luminosa

| Cor      | Endereço |
|----------|----------|
| VERDE    | %Q1.1    |
| AMARELO  | %Q2.0    |
| VERMELHO | %Q2.1    |

---

## 11. Anomalias de Hardware Documentadas no Código

Estas correções já estão implementadas nos FBs e **não devem ser revertidas**:

**BRT02 — Turbina com relé NF no painel:**
```scl
"01 TURBINA" := NOT #BRT02.EL.SdaTurbinaPust;  // Inversão necessária
```

**BRT03 — Tags de temperatura com polaridade invertida no painel:**
```scl
#BRT03.EL.EnTempBaixaOK := NOT "02 BAIXO TEMPERATURA DO ÓLEO";  // Era NF
#BRT03.EL.EnTempAltaOK  := NOT "02 ALTA TEMPERATURA DO ÓLEO";   // Era NA
```

**BRT03 — Turbina sem inversão (montagem correta):**
```scl
"02 TURBINA" := #BRT03.EL.SdaTurbinaPust;  // Sem NOT — relé montado corretamente
```

**BRT02/BRT03 — Falha da bomba não separada da falha do motor no hardware:**
```scl
#BRT02.EL.EnFDispFalha_BombaOleo := "FALHA BRITADOR 02";  // Mesmo sinal do motor
```

**BRT04 — Etiqueta errada na IHM (é VSI, não cônico):**
```scl
#BRT04.EL.StBotao := "LIGA BRITADOR CÔNICO 04";  // Adesivo IHM incorreto (É VSI)
```

**Inconsistência nas tags de reset de detectores (HMI_Tags):**
- Tag HMI `RESET DETECTOR DE METAL 02` aponta para PLC `"RESET DETECTOR DE METAL 01"`
- Tag HMI `RESET DETECTOR DE METAL 03` aponta para PLC `"RESET DETECTOR DE METAL 02"`
Possível erro de exportação do TIA Portal. A lógica do `Detectores_FC4.scl` usa
os tags físicos corretos (`%I13.6`, `%I13.7`, `%I14.0`).

---

## 12. O que NÃO Inferir

### Topologia completa dos transportadores (TRP08–TRP19)

Apenas TRP06 e TRP07 foram lidos integralmente no código. Os demais seguem o
mesmo padrão Map_In/Process/Core/Map_Out, mas os interlocks exatos e a qual
equipamento cada TRP se conecta a jusante precisam ser verificados no código.

### EMERGÊNCIA EXTERNA PV04 (%I13.5)

Esta tag existe nas ENTRADAS_132.tag mas **não aparece** em `Sys_Safety_FC3.scl`
no cálculo de `GERAL_EMERGENCIA_OK`. Verificar se foi omitida propositalmente
ou se é um bug de implementação antes de presumir que ela não tem efeito.

### SIRENE 24VCA (%Q7.6)

O circuito de 220VCA é acionado por `Main_OB1` via saída `CmdSirene_Fisica`.
O circuito de 24VCA (%Q7.6) não foi encontrado em nenhum código lido.
Verificar se é redundância, circuito auxiliar ou saída não implementada.

### Saídas "CHECAR" do Hidroset (%Q8.2, %Q8.3, %Q8.4)

`01 DESCE HIDROSET CHECAR`, `02 SOBE HIDROSET CHECAR`, `02 DESCE HIDROSET CHECAR`
existem nas tags mas seu acionamento não aparece no código lido.

### Conteúdo dos DBs de instância (arquivos .scl vazios)

Os arquivos `BRITADORES_DB2.scl`, `TRANSPORTADORES_DB1.scl`, `PENEIRAS_DB4.scl`,
`CALHA_VIBRATORIA_DB5.scl`, `SIRENE_DB9.scl` e `Sys_DB3.scl` estão vazios —
a estrutura desses DBs é inferida pela declaração VAR dos FBs correspondentes,
não por declaração explícita. No TIA Portal, DBs de instância são gerados
automaticamente a partir do FB.

### Estrutura dos UDTs de STATUS e TIME

**PLC_DataTypes/STATUS/**

```
UDT_STATUS_BRITADORES   { BRT02: Bool; BRT03: Bool; BRT04: Bool }
UDT_STATUS_CALHAS       { CVR01: Bool; CVR02: Bool; CVR03: Bool }
UDT_STATUS_PENEIRAS     { PNR02: Bool; PNR03: Bool; PNR04: Bool }
UDT_STATUS_DETECTORES   { DTC01: Bool; DTC02: Bool; DTC03: Bool }
UDT_STATUS_TRANSPORTADORES {
    TRP06..TRP19: Bool  (14 campos)
}
UDT_STATUS_PLANTA {        ← usado em CONTROLE_STATUS_DB7 (StOnGlobal, StStartingGlobal, StFaultGlobal)
    BRITADORES      : "UDT_STATUS_BRITADORES"
    TRANSPORTADORES : "UDT_STATUS_TRANSPORTADORES"
    PENEIRAS        : "UDT_STATUS_PENEIRAS"
    CALHAS          : "UDT_STATUS_CALHAS"
    GLOBAL          : Bool   ← bit de falha geral da planta
}
```

**PLC_DataTypes/TIME/**

```
UDT_TIME_BRITADORES     { BR02: Time; BR03: Time; BR04: Time }
UDT_TIME_CALHAS         { CVR01: Time; CVR02: Time; CVR03: Time }
UDT_TIME_PENEIRAS       { PNR02: Time; PNR03: Time; PNR04: Time }
UDT_TIME_TRANSPORTADORES {
    TRP06..TRP19: Time  (14 campos)
}
UDT_TIME_PLANTA {          ← usado em IHM_DB6 (ParSdaTime, ParSdaDelayFalha)
    BRITADORES      : "UDT_TIME_BRITADORES"
    TRANSPORTADORES : "UDT_TIME_TRANSPORTADORES"
    PENEIRAS        : "UDT_TIME_PENEIRAS"
    CALHAS          : "UDT_TIME_CALHAS"
}
```
