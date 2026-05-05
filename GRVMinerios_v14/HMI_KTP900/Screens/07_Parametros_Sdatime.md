Cole no `07_Parametros_Sdatime.md`:

```markdown
# 07_Parametros_Sdatime — Parâmetros de Tempo

## Template
Baseada em Template_1 (cabeçalho + barra de navegação + teclas F1-F8)

## Área de conteúdo

### Cabeçalho da tela
- Botão "◄" (navega para tela anterior)
- Título "Parâmetros de Tempo" centralizado
- Botão "►" (navega para tela seguinte)

### Layout
3 colunas de campos editáveis — todos com label "Timeout Feedback" e formato 0000 s

| Equipamento | Tag IHM |
|---|---|
| Britador 02 | IHM_DB.Parametros.Tempo.Britadores.ParSdaTime_BR02 |
| Britador 03 | IHM_DB.Parametros.Tempo.Britadores.ParSdaTime_BR03 |
| Britador 04 | IHM_DB.Parametros.Tempo.Britadores.ParSdaTime_BR04 |
| Calha 01 | IHM_DB.Parametros.Tempo.Calhas.ParSdaTime_CA01 |
| Calha 02 | IHM_DB.Parametros.Tempo.Calhas.ParSdaTime_CA02 |
| Calha 03 | IHM_DB.Parametros.Tempo.Calhas.ParSdaTime_CA03 |
| Peneira 02 | IHM_DB.Parametros.Tempo.Peneiras.ParSdaTime_PE02 |
| Peneira 03 | IHM_DB.Parametros.Tempo.Peneiras.ParSdaTime_PE03 |
| Peneira 04 | IHM_DB.Parametros.Tempo.Peneiras.ParSdaTime_PE04 |
| Correia 06 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC06 |
| Correia 07 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC07 |
| Correia 08 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC08 |
| Correia 09 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC09 |
| Correia 10 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC10 |
| Correia 11 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC11 |
| Correia 12 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC12 |
| Correia 13 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC13 |
| Correia 14 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC14 |
| Correia 15 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC15 |
| Correia 16 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC16 |
| Correia 17 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC17 |
| Correia 18 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC18 |
| Correia 19 | IHM_DB.Parametros.Tempo.Transportadores.ParSdaTime_TC19 |

### Elementos visuais
- Campos numéricos editáveis — formato 0000 s
- Entrada via teclado numérico da HMI
- Parâmetro ParSdaTime = timeout de confirmação de feedback após comando de partida

### Observação
Tags sugeridas — confirmar nomes reais no IHM_DB quando disponível.
Parâmetros sincronizados ao CLP via SincParam (Network 3 do Main).
```

Manda a próxima.