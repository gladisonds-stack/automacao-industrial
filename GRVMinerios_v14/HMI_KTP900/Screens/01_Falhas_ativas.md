# 01_Falhas_ativas — Falhas Ativas/Pendentes

## Template
Baseada em Template_1 (cabeçalho + barra de navegação + teclas F1-F8)

## Área de conteúdo

### Cabeçalho da tela
- Botão "◄" (navega para tela anterior)
- Título "Falhas Ativas/Pendentes" centralizado
- Botão "►" (navega para tela seguinte)

### Lista de alarmes
Componente nativo HMI — Alarm View
- Colunas: Hora | Data | Descrição
- Scroll vertical (scrollbar lateral direita)
- Exibe apenas alarmes ativos/pendentes de reconhecimento

### Comportamento
- Atualização em tempo real via Alarm Server do TIA
- Alarmes somem da lista quando reconhecidos e resolvidos