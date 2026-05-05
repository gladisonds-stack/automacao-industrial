# 06_Parametros_Sirene — Parâmetros da Sirene

## Template
Baseada em Template_1 (cabeçalho + barra de navegação + teclas F1-F8)

## Área de conteúdo

### Cabeçalho da tela
- Botão "◄" (navega para tela anterior)
- Título "Parâmetros da Sirene" centralizado
- Botão "►" (navega para tela seguinte)

### Ícone
- Ícone de alto-falante/sirene centralizado no topo da área de conteúdo

### Painel: Temporização da Sirene
Grupo com borda rotulada "Temporização da Sirene"
Duas colunas: Atual (somente leitura) e Novo (editável pelo operador)

| Parâmetro | Tag Atual | Tag Novo |
|---|---|---|
| Tempo de toque | IHM_DB.Parametros.Sirene.ParTempoToque (leitura) | IHM_DB.Parametros.Sirene.ParTempoToque (escrita) |
| Tempo de validade | IHM_DB.Parametros.Sirene.ParTempoValidade (leitura) | IHM_DB.Parametros.Sirene.ParTempoValidade (escrita) |

- Formato display: 00 s / 00 m
- Campos "Novo" são editáveis via teclado numérico da HMI

### Painel: Teste da Sirene
Grupo com borda rotulada "Teste da Sirene"
- Botão **"Tocar"** → aciona `IHM_DB.Comandos.CmdTesteSirene`
- Dispara toque manual da sirene para verificação em campo

### Observação
Tela de configuração — acesso deve ser restrito ao supervisor/mantenedor.
Parâmetros gravados no IHM_DB e sincronizados ao CLP via SincParam (Network 3 do Main).