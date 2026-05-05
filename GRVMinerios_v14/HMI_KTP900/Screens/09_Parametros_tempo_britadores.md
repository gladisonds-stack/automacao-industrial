# 09_Parametros_tempo_britadores — Parâmetros de Tempo Britadores

## Template
Baseada em Template_1 (cabeçalho + barra de navegação + teclas F1-F8)

## Área de conteúdo

### Cabeçalho da tela
- Botão "◄" (navega para tela anterior → 08_Parametros_Sdadelayfalha)
- Título "Parâmetros de Tempo Britadores" (duas linhas) centralizado
- Botão "►" (navega para tela seguinte)

### Colunas
3 colunas de equipamentos: BRITADOR 02 | BRITADOR 03 | BRITADOR 04

### Parâmetro 1: Tempo de Preparo da Lubrificação
> Tempo máximo para a resistência aquecer o óleo e a bomba pressurizar o sistema
> Tag: ParPrepTime

| Equipamento | Display |
|---|---|
| Britador 02 | 00 m |
| Britador 03 | 00 m |
| Britador 04 | 00 m |

### Parâmetro 2: Tempo de Intervalo Liga Rotor VSI
> Tempo mandatório de circulação para o banho de óleo do mancal superior antes do rotor girar
> Tag: ParPreLubeTime

| Equipamento | Display |
|---|---|
| Britador 04 | 00 m |

*Aplicável apenas ao Britador 04 (VSI) — BR02 e BR03 não têm este parâmetro*

### Parâmetro 3: Tempo de Desligamento
> Tempo mandatório de inércia/cooldown, até o desligamento completo da lubrificação
> Tag: ParSdaDelayIdling

| Equipamento | Display |
|---|---|
| Britador 02 | 00 m |
| Britador 03 | 00 m |
| Britador 04 | 00 m |

### Elementos visuais
- Campos numéricos editáveis — formato 00 m
- Descrição detalhada de cada parâmetro exibida na própria tela
- Tags documentadas diretamente na IHM — referência importante para o código SCL

### Observação
Tela revela a lógica de sequenciamento dos britadores:
1. Preparo da lubrificação (aquecimento + pressurização)
2. Intervalo pré-liga do rotor VSI (somente BR04)
3. Cooldown pós-desligamento antes de parar a lubrificação

Essa sequência deve ser implementada no FB BRITADORES como SFC de processo.