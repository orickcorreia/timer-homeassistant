# Timer para Home Assistant com Node-RED

## Recursos usados:
* Button-card para exibir os botões com nomes personalizados https://github.com/custom-cards/button-card
<br>Obs. O recurso button-card pode ser instalado através do HACS - Home Assistant Community Store

* Node-RED para criar a programação do timer https://github.com/hassio-addons/addon-node-red
<br>Obs. O Node-RED pode ser instalado através da Add-on Store dentro do Supervisor


## Entidades que usaremos:
* input_boolean
* input_number
* input_select
* sensor

## Configuração


## Input Boolean:

Começamos com o input que será usado para ativar a sequencia da programação do timer; 
```
input_boolean:

  #Input para ativar o timer 
  timer_ativar:
    name: Timer
    initial: off
    icon: mdi:progress-check
```

Depois acrescentamos os inputs para cada dispositivo. <br>
Esses inputs serão usados como botões que sinalizam quando um timer estiver ativado e para cancelar um timer ao clicar.<br>
Faça um input_boolean para cada dispositivo que desejar adicionar na lista do timer.
```
  #Mantenha essa formatação para facilitar a configuração "timer_nome_do_dispositivo"
  #Mantenha um padrão na formatação do nome, pois é esse nome que será exibido quando o timer estiver ativo.
  
  timer_tv_quarto:
    name: Timer - TV do Quarto 
    initial: off
    icon: mdi:television

  timer_lampadas:
    name: Timer - Lampadas
    initial: off
    icon: mdi:lightbulb-group-outline
```
Resultado:
* input_boolean.timer_ativar
* input_boolean.timer_tv_quarto
* input_boolean.timer_lampadas



## Input Number:
Vamos criar um input_number que será usado para definir os minutos do timer.<br>
Você pode escolher um input com slider (como no gif).
```
input_number:

  timer_minutos:
    icon: mdi:timer
    name: Temporizador
    initial: 5
    min: 5
    max: 180
    step: 5
```
Ou pode escolher um input com box para digitar o valor:
```
input_number:

  timer_minutos:
    icon: mdi:timer
    name: Temporizador
    initial: 5
    min: 5
    max: 180
    step: 5
    mode: box
```
Resultado:
* input_number.timer_minutos


## Input Select:

O input_select será usado para criar a lista de seleção com os dispositivos na interface do timer.<br>
Acrescente as oções de acordo com sua necessidade.<br>
ATENÇÃO! Só acrescente opcões que tiverem input_booleans criados.
```
input_select:

  timer_lista:
    name: 'Timer - Dispositivos'
    icon: mdi:devices
    options: #Só acrescente as opções que tiverem input_boolean
      - TV do Quarto
      - Lampadas
```
Resultado:
* input_select.timer_lista

## Sensor

Vamos criar sensores com templates para calcular e exibir o tempo que ainda falta para o dispositivo ser desligado.<br>
ATENÇÃO! Se ao criar mais sensores para os seus dispositivos, altere também o template do calculo. Por isso que é importante manter um padrão "timer_nome_do_dispositivo". Observe que o "value_template" faz referência ao próprio sensor e ao input_boolean do dispositivo em questão. Compare os dois exemplos abaixo.
```
sensor:
  - platform: template
    sensors:

      timer_tv_quarto:
        friendly_name: 'Timer - TV do Quarto'
        unit_of_measurement: min
        value_template: "{% if (states.sensor.time.state) | float == 0 %}
        {{ ((((state_attr('sensor.timer_tv_quarto', 'timer') | float()) * 60) - (as_timestamp(now()) -      as_timestamp(states.input_boolean.timer_tv_quarto.last_changed))) / 60) | round(0) }}
        {% endif %}"
        attribute_templates:
          timer: >-
            {{ state_attr('sensor.timer_tv_quarto', 'timer')}}

      timer_lampadas:
        friendly_name: 'Timer - Lampadas'
        unit_of_measurement: min
        value_template: "{% if (states.sensor.time.state) | float == 0 %}
        {{ ((((state_attr('sensor.timer_lampadas', 'timer') | float()) * 60) - (as_timestamp(now()) - as_timestamp(states.input_boolean.timer_lampadas.last_changed))) / 60) | round(0) }}
        {% endif %}"
        attribute_templates:
          timer: >-
            {{ state_attr('sensor.timer_lampadas', 'timer')}}
```
Resultado:
* sensor.timer_tv_quarto
* sensor.timer_lampadas


## Node-RED
O fluxo do Node-RED deve ser importado e depois editado de acordo com sua necessidade.<br>
Para fazer a importação você pode baixar o arquivo .json ou copiar o código e colar na janela de importação do Node-RED.<br>
[Clique aqui para copiar ou fazer download do código dos fluxos do Node-RED](https://github.com/orickcorreia/timer-homeassistant/blob/master/nodered_timer.json)

## Aplicando na interface (Lovelace)

```
- type: vertical-stack
  cards:

  # Título Cabeçalho
  - type: 'custom:button-card'
    layout: icon_name
    name: Timer
    icon: mdi:timer
    styles:
      grid:
        - grid-template-areas: '"n i"'
        - grid-template-columns: 1fr 20% 
      icon:
        - align-self: end
        - color: var(--text-primary-color)
        - height: 35px
      card:
        - padding: 5px
        - height: 45px
        - background: var(--primary-color)
      name:
        - color: var(--text-primary-color)
        - justify-self: start
        - padding-left: 10%
        - font-weight: 400
        - font-size: 20px

  # Painel Timer
  - type: entities
    show_header_toggle: false
    entities:
      - entity: input_select.timer_lista
        name: Dispositivo
      - entity: input_number.timer_minutos
        name: Tempo
      - entity: input_boolean.timer
        name: Ativar
        
  #Condicional para exibir apenas quando o timer estiver ativo as Lampadas
  - type: conditional
    conditions:
      - entity: input_boolean.timer_lampadas
        state: "on"
    card:
      entity: input_boolean.timer_lampadas
      type: "custom:button-card"
      layout: icon_name
      tap_action:
        action: toggle
      label: >
        [[[
          return 'Desligando em ' + (states['sensor.timer_lampadas'].state) + ' min' ; ;
        ]]]
      show_label: true
      styles:
        grid:
          - grid-template-areas: '"i n l"'
          - grid-template-columns: 15% 35% 50%  
        icon:
          - align-self: start
          - color: var(--primary-color)
          - height: 40px
        card:
          - padding: 5px
          - height: 45px
        name:
          - justify-self: start
          - font-weight: 400
          - font-size: 14px
        label:
          - justify-self: end
          - font-weight: 400
          - font-size: 14px
          - padding-right: 10%

  #Condicional para exibir apenas quando o timer estiver ativo as TV do Quarto
  - type: conditional
    conditions:
      - entity: input_boolean.timer_tv_quarto
        state: "on"
    card:
      entity: input_boolean.timer_tv_quarto
      type: "custom:button-card"
      layout: icon_name
      tap_action:
        action: toggle
      label: >
        [[[
          return 'Desligando em ' + (states['sensor.timer_tv_quarto'].state) + ' min' ; ;
        ]]]
      show_label: true
      styles:
        grid:
          - grid-template-areas: '"i n l"'
          - grid-template-columns: 15% 35% 50%  
        icon:
          - align-self: start
          - color: var(--primary-color)
          - height: 40px
        card:
          - padding: 5px
          - height: 45px
        name:
          - justify-self: start
          - font-weight: 400
          - font-size: 14px
        label:
          - justify-self: end
          - font-weight: 400
          - font-size: 14px
          - padding-right: 10%
```






