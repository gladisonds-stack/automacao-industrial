# 08_Parametros_Sdadelayfalha — Parâmetros de Tempo (Anti-Flicker)

## Template
Baseada em Template_1 (cabeçalho + barra de navegação + teclas F1-F8)

## Área de conteúdo

### Cabeçalho da tela
- Botão "◄" (navega para tela anterior → 07_Parametros_Sdatime)
- Título "Parâmetros de Tempo" centralizado
- Botão "►" (navega para tela seguinte)

### Diferença em relação a 07_Parametros_Sdatime
- 07 configura o **ParSdaTime** — timeout de confirmação de partida
- 08 configura o **ParSdaDelayFalha** — tolerância Anti-Flicker do feedback em operação

### Layout
3 colunas de campos editáveis — todos com label "Anti-Flicker" e formato 0000 s

| Equipamento | Tag IHM |
|---|---|
| Britador 02 | IHM_DB.Parametros.Tempo.Britadores.ParSdaDelayFalha_BR02 |
| Britador 03 | IHM_DB.Parametros.Tempo.Britadores.ParSdaDelayFalha_BR03 |
| Britador 04 | IHM_DB.Parametros.Tempo.Britadores.ParSdaDelayFalha_BR04 |
| Calha 01 | IHM_DB.Parametros.Tempo.Calhas.ParSdaDelayFalha_CA01 |
| Calha 02 | IHM_DB.Parametros.Tempo.Calhas.ParSdaDelayFalha_CA02 |
| Calha 03 | IHM_DB.Parametros.Tempo.Calhas.ParSdaDelayFalha_CA03 |
| Peneira 02 | IHM_DB.Parametros.Tempo.Peneiras.ParSdaDelayFalha_PE02 |
| Peneira 03 | IHM_DB.Parametros.Tempo.Peneiras.ParSdaDelayFalha_PE03 |
| Peneira 04 | IHM_DB.Parametros.Tempo.Peneiras.ParSdaDelayFalha_PE04 |
| Correia 06 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC06 |
| Correia 07 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC07 |
| Correia 08 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC08 |
| Correia 09 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC09 |
| Correia 10 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC10 |
| Correia 11 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC11 |
| Correia 12 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC12 |
| Correia 13 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC13 |
| Correia 14 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC14 |
| Correia 15 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC15 |
| Correia 16 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC16 |
| Correia 17 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC17 |
| Correia 18 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC18 |
| Correia 19 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaDelayFalha_TC19 |

### Elementos visuais
- Campos numéricos editáveis — formato 0000 s
- Entrada via teclado numérico da HMI

### Observação
ParSdaDelayFalha = tempo de tolerância antes de declarar falha por perda de
feedback mecânico durante operação normal (Status 11 no bloco de equipamento).
Tags sugeridas — confirmar nomes reais no IHM_DB quando disponível.
Parâmetros sincronizados ao CLP via SincParam (Network 3 do Main).