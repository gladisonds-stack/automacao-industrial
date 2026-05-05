# GRVMinerios_v14 — Automação de Planta de Britagem

Projeto de automação para uma planta de britagem de minérios com circuito
fechado de trituração. Desenvolvido em SCL (Structured Control Language) para
CLP Siemens S7-1200, programado via TIA Portal.

---

## O que o sistema controla

A planta possui três linhas de produção independentes, cada uma com um britador
principal, transportadores de correia, peneira vibratória e calha vibratória:

| Linha | Britador | Tipo     | Transportadores | Peneira | Calha |
|-------|----------|----------|-----------------|---------|-------|
| A     | BRT02    | Cônico   | TRP06–TRP09     | PNR02   | CVR01 |
| B     | BRT03    | Cônico   | TRP10–TRP13     | PNR03   | CVR02 |
| C     | BRT04    | VSI      | TRP14–TRP16     | —       | CVR03 |
| D     | —        | Expedição| TRP17–TRP19     | PNR04   | —     |

**Restrição operacional:** Os britadores cônicos (BRT02/BRT03) e o VSI (BRT04)
são mutuamente exclusivos por limitação de subestação elétrica. Apenas um grupo
pode estar em operação por vez.

---

## Hardware

| Componente | Modelo / Especificação                   |
|------------|------------------------------------------|
| CLP        | Siemens S7-1214C DC/DC/DC (ID: 20A1)    |
| IHM        | Siemens KTP900 Basic PN (9", 800×480)   |
| I/O digital| 9 módulos de expansão (20A1–20A9)        |
| I/O analógico | 2 entradas analógicas 4–20 mA (pressão hidroset) |
| Entradas digitais | 132 pontos (%I0.0 – %I15.7)    |
| Saídas digitais   | 128 pontos (%Q0.0 – %Q15.7)    |
| Rede       | PROFINET (HMI_Connection_1, 100 ms)      |

---

## Estrutura de pastas

```
GRVMinerios_v14/
├── PLC_20A1/
│   ├── Program_Blocks/
│   │   ├── Main_OB1.scl               # OB1 — ciclo principal (10 redes)
│   │   ├── BRITADORES/
│   │   │   └── BRITADORES_FB2.scl     # FB2 — BRT02, BRT03, BRT04
│   │   ├── TRANSPORTADORES/
│   │   │   └── TRANSPORTADORES_FB5.scl # FB5 — TRP06 a TRP19
│   │   ├── PENEIRAS/
│   │   │   └── PENEIRAS_FB6.scl       # FB6 — PNR02, PNR03, PNR04
│   │   ├── CALHA_VIBRATORIA/
│   │   │   └── CALHA_VIBRATORIA_FB7.scl # FB7 — CVR01, CVR02, CVR03
│   │   ├── PARAMETROS/
│   │   │   └── SincParam_FC5.scl      # FC5 — sincronismo IHM → CLP (1×/s)
│   │   └── GERAL/
│   │       ├── SYS/
│   │       │   └── Sys_Safety_FC3.scl # FC3 — emergências, torre luminosa
│   │       ├── SIRENE/
│   │       │   └── SIRENE_FB10.scl    # FB10 — FSM sirene industrial
│   │       ├── ERROS/
│   │       │   └── Erros_FC2.scl      # FC2 — mapeamento de falhas → StErros[]
│   │       └── MAQ_ESTADOS/
│   │           ├── EL_4St_Fct_FB3.scl # FSM 4 estados (TRP, PNR, CVR)
│   │           ├── EL_6St_BRT_Fct_FB9.scl # FSM 6 estados (britadores cônicos)
│   │           └── EL_7St_Fct_FB1.scl # FSM 7 estados (britador VSI)
│   ├── Data_Blocks/
│   │   ├── IHM_DB6.scl                # DB6 RETAIN — parâmetros e status para IHM
│   │   └── CONTROLE_STATUS_DB7.scl    # DB7 — status global da planta
│   └── Instance_DBs/                  # DBs de instância (gerados automaticamente pelo TIA)
├── PLC_DataTypes/
│   ├── MAQ_ESTADOS/                   # UDTs internos de cada FSM
│   │   ├── EL_4St_UDT.udt
│   │   ├── EL_6St_BRT_UDT.udt
│   │   └── EL_7St_UDT.udt
│   ├── STATUS/                        # UDTs de status por categoria de equipamento
│   │   ├── UDT_STATUS_PLANTA.udt
│   │   ├── UDT_STATUS_BRITADORES.udt
│   │   ├── UDT_STATUS_TRANSPORTADORES.udt
│   │   ├── UDT_STATUS_PENEIRAS.udt
│   │   ├── UDT_STATUS_CALHAS.udt
│   │   └── UDT_STATUS_DETECTORES.udt
│   └── TIME/                          # UDTs de parâmetros de tempo
│       ├── UDT_TIME_PLANTA.udt
│       ├── UDT_TIME_BRITADORES.udt
│       ├── UDT_TIME_TRANSPORTADORES.udt
│       ├── UDT_TIME_PENEIRAS.udt
│       └── UDT_TIME_CALHAS.udt
├── PLC_Tags/
│   ├── ENTRADAS_132.tag               # Mapeamento das 132 entradas digitais + 4 analógicas
│   └── SAIDAS_128.tag                 # Mapeamento das 128 saídas digitais
├── CLAUDE.md                          # Referência técnica completa (para Claude Code)
└── README.md                          # Este arquivo
```

---

## Arquitetura do software

Todos os blocos de função seguem o padrão **Map_In → Process → Core → Map_Out**,
implementado com Regions SCL:

1. **Map_In** — lê entradas físicas e aplica correções de hardware (inversão de
   lógica, filtros)
2. **Process** — calcula permissivos e intertravamentos; trata botão toggle da IHM
3. **Core** — chama a máquina de estados (FB genérica)
4. **Map_Out** — escreve saídas físicas e bits de status global

As máquinas de estados são FBs genéricas reutilizáveis:
- `EL_4St` — 4 estados: Off / Starting / On / Fault (transportadores, peneiras, calhas)
- `EL_6St_BRT` — 6 estados: adiciona Idling e ciclo de lubrificação (britadores cônicos)
- `EL_7St` — 7 estados: adiciona Waiting (pré-lubrificação obrigatória do VSI)

---

## Segurança e emergências

- `GERAL_EMERGENCIA_OK` (%M1.1) é um bit com memória — **não reseta sozinho**.
  Exige todas as emergências liberadas **e** o botão RESET MESA DE CONTROLE.
- Detectores de metal (DTC01/02/03): lógica NF — `0 = metal ou sensor com
  problema`. Detecção bloqueia nova partida da linha correspondente.
- Exclusão mútua BRT02/BRT03 ↔ BRT04 é aplicada nos permissivos de cada britador,
  impedindo habilita se o grupo oposto estiver com `StOn = TRUE`.

---

## Referência técnica detalhada

Para documentação completa (estados das FSMs, mapeamento de erros, anomalias de
hardware, valores padrão de parâmetros, mapeamento de I/O por slot), consulte
[CLAUDE.md](CLAUDE.md).
