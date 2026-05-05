# Template_1 — Layout Base HMI KTP900

## Descrição
Template base aplicado a todas as telas do projeto.
Resolução: 900x600 | HMI: Siemens KTP900 Basic PN

## Elementos Fixos (presentes em todas as telas)

### Cabeçalho (topo)
- Logo ELETROCER (canto superior esquerdo)
- Ícone de alerta + botão "Falhas" (canto superior direito) → navega para 01_Falhas_ativas
- Data e hora (canto superior direito)

### Barra de navegação inferior (toolbar)
| Posição | Ícone | Função |
|---|---|---|
| 1 | Casa | Navega para 00_Home |
| 2 | Alerta/lista | Navega para 01_Falhas_ativas |
| 3 | Lupa/diagnóstico | Navega para 02_Histórico_de_Falhas |
| 4 | Cronômetro | Navega para 09_Parametros_tempo_britadores |
| 5 | (vazio) | Reserva |
| 6 | (vazio) | Reserva |
| 7 | (vazio) | Reserva |
| 8 | Power | Desliga/encerra sessão |

### Teclas de função (rodapé)
F1 a F8 — teclas físicas do KTP900
F1, F2, F4, F8 com indicador verde ativo
F5, F6, F7 sem função definida (reserva)

## Área de conteúdo
Região central livre — cada tela define seu próprio conteúdo nessa área.