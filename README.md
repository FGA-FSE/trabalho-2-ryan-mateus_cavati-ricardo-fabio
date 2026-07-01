# Trabalho 2 - 2026/1

Trabalho 2 da disciplina de Fundamentos de Sistemas Embarcados (2026/1)

#### Integrantes:

- Fabio G. S. Barbosa - 222006712
- Mateus Cavalcante de Sousa - 222006991
- Ricardo L. W. V. Branco - 221008409
- Ryan A. B. Salles - 22100436

# Reprodução do Controle DualShock 4 com Reinserção de Botões de Pressão 

**Produto em estudo:** Controle **DualShock 4 (DS4)**, do PlayStation 4

---

## 1. Descrição do produto selecionado

### Funções principais, público-alvo e contexto de uso

O produto em estudo é o **DualShock 4 (DS4)**, controle oficial do PlayStation 4. Suas principais funções incluem:

- Entradas digitais (D-pad, botões de face X/○/□/△, botões de ombro L1/R1/L2/R2 digitais, L3/R3 — cliques dos analógicos, Options/Share, PS button);
- Dois **analógicos** com eixos X/Y (potenciômetros/sensores Hall);
- **Gatilhos L2/R2 analógicos** (pressão contínua, usados por exemplo para acelerar/frear em jogos de corrida);
- **Touchpad** capacitivo com clique;
- **Giroscópio e acelerômetro** (sensor de movimento em 6 eixos);
- **Motor de vibração** (rumble) duplo;
- **Barra de luz (light bar)** RGB, usada para identificação de jogador e, em alguns jogos, feedback visual;
- **Alto-falante interno** e **entrada para headset** (P2);
- Bateria interna recarregável via USB.

- **Público-alvo:** jogadores de console (PS4), com uso também em PC via cabo USB ou Bluetooth (suporte parcial/via drivers, dependendo do jogo).
- **Contexto de uso:** entretenimento doméstico, conectado ao console via **Bluetooth** ou cabo USB (também usado para carregar a bateria).

> **Observação importante:** diferente dos controles DualShock de PS2/PS3, o DS4 **não possui** botões de face sensíveis à pressão (essa feature existia no PS2/PS3 e foi removida no PS4, substituída pelos gatilhos analógicos L2/R2 e pelo touchpad). A proposta deste projeto é justamente **reintroduzir** esse recurso como uma extensão/modificação conceitual sobre a base do DS4, e será detalhada na Seção 3.

### Componentes e sensores utilizados (produto original)

- Botões de face e D-pad: contatos digitais tipo membrana/dome switch (sem sensibilidade à pressão).
- Analógicos: potenciômetros ou sensores de efeito Hall (dependendo da revisão do hardware).
- Gatilhos L2/R2: sensores analógicos (potenciômetro), permitindo leitura de pressão contínua nesses dois botões específicos.
- IMU (giroscópio + acelerômetro) para detecção de movimento/orientação.
- Touchpad capacitivo com botão de clique embutido.
- Motores de vibração (rumble), light bar RGB, alto-falante e microfone (em alguns modelos/jogos via headset).
- Microcontrolador dedicado responsável pela leitura de todos os sensores/botões e empacotamento dos dados.
- Bateria de íon de lítio recarregável.

### Tecnologias de comunicação e controle embarcadas

- **Bluetooth** (perfil HID customizado da Sony, não um HID genérico padrão — por isso alguns jogos em PC exigem drivers/emuladores específicos, como o DS4Windows) e **USB com fio** como alternativa.
- Leitura analógica multiplexada dos eixos dos analógicos, gatilhos L2/R2 e sensores de movimento.
- Firmware embarcado responsável por debounce dos botões digitais, calibração dos analógicos/gatilhos, fusão de sensores do IMU e empacotamento de todos os dados no protocolo de comunicação com o console.
- Gerenciamento de energia da bateria interna (indicação de carga via light bar, modos de economia quando ocioso).

---

## 2. Análise técnica do funcionamento

### Principais módulos do sistema

| Módulo | Função no DualShock 4 |
|---|---|
| **Sensores** | Botões digitais (face, D-pad, ombro), gatilhos L2/R2 analógicos, analógicos direcionais (potenciômetros/Hall), touchpad capacitivo, IMU (giroscópio + acelerômetro) |
| **Atuadores** | Motores de vibração (rumble duplo), light bar RGB, alto-falante interno |
| **Controle** | Microcontrolador interno responsável por ler todos os sensores, aplicar debounce/filtragem, fundir dados do IMU e montar o pacote de dados |
| **Interface** | Light bar (feedback visual), vibração (feedback háptico), alto-falante/microfone (áudio, quando via headset) |
| **Conectividade** | Bluetooth (perfil HID customizado) ou USB com fio; USB também usado para carregamento da bateria |

### Tecnologias críticas identificadas

- **Bluetooth com perfil HID customizado:** ao contrário de um HID genérico de teclado/mouse, o DS4 usa um relatório de dados específico da Sony, o que exige tradução/emulação (ex: DS4Windows) para uso pleno em PC.
- **Conversão analógico-digital (ADC):** usada nos gatilhos L2/R2 e nos analógicos — é o mecanismo que este projeto pretende **estender também aos botões de face**, reintroduzindo a sensibilidade à pressão que existia no PS2/PS3.
- **Fusão de sensores (IMU):** giroscópio + acelerômetro combinados para detecção de orientação/movimento — fora do escopo do MVP proposto, mas pode ser citado como possível extensão futura.
- **Debounce e filtragem de sinal:** necessário tanto para os botões digitais quanto para suavizar ruído nos sinais analógicos dos gatilhos/analógicos.
- **Gestão de energia:** o DS4 opera com bateria interna recarregável, com indicação de carga via light bar — o firmware provavelmente aplica técnicas de baixo consumo (sleep entre ciclos de leitura, desligamento de LEDs quando ocioso). O protótipo em protoboard, alimentado via USB do PC/ESP32, não replicará essa característica — ponto a ser citado como limitação consciente de escopo.

---

## 3. Proposta de reprodução com ESP32



O objetivo é reproduzir, em protótipo, um subconjunto das funções do DS4 e **adicionar** a funcionalidade de botões de face sensíveis à pressão (recurso presente no PS2/PS3, ausente no DS4 original), como proposta de melhoria/estudo comparativo. O MVP contempla:

- **4 botões de face (X/○/□/△)** com leitura de nível de pressão, calibração e saída em tempo real via **serial + display OLED**;
- Conexão ao computador para uso real em um jogo (via **Bluetooth HID Gamepad** ou, como alternativa mais simples, USB serial + software intermediário tipo *vJoy*/*joystick emulator*), replicando a via de conectividade real do DS4;
- um **analógico** (joystick 2 eixos), aproximando-se dos analógicos direcionais reais do DS4.

Como o principal ponto de incerteza da disponibilidade do sensor FSR, a proposta contempla três variantes de implementação para os botões, que podem inclusive ser comparadas entre si no relatório final como parte da análise técnica.

#### Opção A — FSR (Force Sensitive Resistor) 
- **Como funciona:** o FSR é montado em divisor de tensão (FSR + resistor fixo) e lido em um pino ADC do ESP32. A resistência cai conforme a pressão aumenta, então a tensão lida varia continuamente.
- **Fidelidade ao original:**  é o método que mede pressão  de forma analógica direta.
- **Prós:** sinal contínuo, calibração com mapeamento não-linear.
- **Contras:** FSR tem histerese (não retorna exatamente ao mesmo valor de repouso) e resposta não-linear, exigindo filtragem/calibração no firmware.

#### Opção B — Botão de membrana + FSR sobreposto (híbrido)
- **Como funciona:** usa-se a cúpula de borracha da membrana apenas pelo *feel* tátil (o "clique"), com um FSR fino posicionado entre a membrana e a base, capturando a força aplicada.
- **Fidelidade ao original:** e é a opção que **mais se aproxima fisicamente** da construção real do DualShock (cúpula de contato + camada sensível à pressão).
- **Prós:** sensação de pressionar mais próxima da experiência real; ainda fornece leitura analógica pelo FSR.
- **Contras:** exige montagem mecânica mais cuidadosa (empilhamento de camadas) e mais um componente físico a se conseguir.

#### Opção C — Botão digital comum (tátil/push-button), sem FSR disponível
é possível **simular** um nível de pressão a partir de sinais puramente digitais, com duas heurísticas:

1. **Baseada em tempo de contato:** quanto mais tempo o botão permanece pressionado, maior o "nível" reportado. É uma proxy temporal, não uma medida física de força.
2. **Baseada na velocidade/brusquidão do toque:** mede-se o intervalo entre transições de borda (bounce) do sinal; toques mais firmes tendem a gerar transições mais rápidas/bruscas. Um pouco mais sofisticada, mas ainda é uma heurística, não uma medição direta.
<!-- 3. *(Variante mecânica, opcional)*: empilhar 2–3 microswitches com espaçadores de espuma entre eles, de forma que níveis crescentes de pressão acionem switches adicionais — gera 2–3 níveis discretos e pressão usando só componentes digitais, sem inferência por tempo. -->



### Diagrama conceitual do sistema (blocos)

```
 ┌─────────────────────┐        ┌────────────────────────┐
 │  Botões de face      │        │   Analógico  │
 │  X / O / □ / △       │        │   Joystick 2 eixos       │
 │  (FSR / membrana+FSR │        │   (potenciômetros)       │
 │   / push-button)     │        └───────────┬──────────────┘
 └──────────┬───────────┘                    │
            │  sinal analógico (ADC1)         │  sinal analógico (ADC1)
            ▼                                 ▼
      ┌────────────────────────────────────────────┐
      │                   ESP32                      │
      │  - Leitura ADC (multiplexada)                 │
      │  - Debounce / filtragem (média móvel)          │
      │  - Calibração (mapeamento não-linear)           │
      │  - Empacotamento dos dados                       │
      └───────────┬───────────────────┬─────────────┘
                  │                   │
         Serial / I2C (debug)     Bluetooth HID
                  │                   │
                  ▼                   ▼
        ┌──────────────────┐  ┌──────────────────────┐
        │  Display OLED     │  │  Computador (host)     │
        │  (nível em tempo   │  │  - Reconhecido como     │
        │   real, por botão) │  │    gamepad HID           │
        └──────────────────┘  │  - Testável em jogo real  │
                               └──────────────────────────┘
```

### Tecnologias/bibliotecas do ecossistema ESP-IDF / Arduino

- **ADC1** do ESP32 para leitura dos sensores de pressão (evitar ADC2, que conflita com o uso simultâneo do Wi-Fi).
- Biblioteca de **BLE HID Gamepad** (ex: `ESP32-BLE-Gamepad`, disponível para Arduino/ESP-IDF) para emular o controle como periférico Bluetooth reconhecido nativamente pelo PC.
- Display **OLED via I2C** (SSD1306, por exemplo) para exibição dos níveis de pressão em tempo real durante testes/calibração.
- Rotinas de filtragem (média móvel simples) e calibração (mapeamento min/max por botão) implementadas em firmware.

### Limitações e desafios esperados

- **Não-linearidade e histerese dos FSRs**, exigindo etapa de calibração e possivelmente compensação por software.
- **Latência do Bluetooth HID** pode ser perceptível em jogos que exigem resposta muito rápida, ao contrário do controle original (otimizado para baixa latência).

- **Gestão de energia:** diferente do controle original (alimentado a bateria com técnicas de economia de energia), o protótipo em protoboard normalmente será alimentado via USB, então essa característica do produto original não será plenamente reproduzida — pode ser citada como limitação consciente do escopo.
- **Fidelidade tátil:** apenas a Opção B (membrana + FSR) reproduz de fato a sensação física de pressionar um botão do DualShock; as demais priorizam a funcionalidade elétrica/lógica sobre a experiência tátil.

### Comparativo com produtos similares

O DualShock 4 pertence à categoria de controles (gamepads) para consoles e computadores. Ao longo da evolução dos videogames, diversos modelos da própria Sony e de fabricantes concorrentes apresentaram soluções semelhantes ou evoluções das funcionalidades presentes no DualShock 4. A tabela abaixo compara o produto estudado com controles de diferentes gerações.

| Produto | Ano | Plataforma | Conectividade | Principais Recursos | Alimentação | Diferenças em relação ao DualShock 4 |
|---------|:---:|------------|----------------|---------------------|-------------|--------------------------------------|
| **DualShock 3** | 2007 | PlayStation 3 | Bluetooth e USB | Analógicos, acelerômetro, giroscópio, vibração, botões de face sensíveis à pressão | Bateria interna | Possui botões sensíveis à pressão, mas não possui touchpad, barra de luz nem alto-falante. |
| **DualShock 4 (Produto estudado)** | 2013 | PlayStation 4 | Bluetooth e USB | Touchpad capacitivo, giroscópio, acelerômetro, barra de luz RGB, alto-falante, gatilhos analógicos, vibração | Bateria interna | Produto de referência deste trabalho. |
| **DualSense** | 2020 | PlayStation 5 | Bluetooth e USB-C | Feedback háptico, gatilhos adaptáveis, touchpad, microfone, giroscópio, acelerômetro | Bateria interna | Evolução direta do DS4, com feedback tátil e gatilhos inteligentes. |
| **Xbox Wireless Controller (Series X/S)** | 2020 | Xbox Series X/S e PC | Bluetooth, USB-C e Wireless Xbox | Vibração, gatilhos analógicos, botão Share | Pilhas AA ou bateria recarregável | Não possui touchpad nem sensores de movimento. Excelente compatibilidade com PCs. |
| **Nintendo Switch Pro Controller** | 2017 | Nintendo Switch | Bluetooth e USB-C | Giroscópio, acelerômetro, HD Rumble, NFC (Amiibo) | Bateria interna | Não possui touchpad nem alto-falante, mas possui NFC e vibração HD. |
| **8BitDo Ultimate Bluetooth Controller** | 2022 | PC, Nintendo Switch | Bluetooth, USB e 2.4 GHz | Analógicos Hall Effect, giroscópio, vibração, botões programáveis | Bateria interna com dock | Utiliza sensores Hall Effect, reduzindo o problema de drift e aumentando a durabilidade. |

### Análise Comparativa

O **DualShock 4** representou uma evolução significativa em relação ao **DualShock 3**, trazendo novos recursos como o **touchpad capacitivo**, **barra de luz RGB**, **alto-falante integrado** e melhorias na ergonomia. Em contrapartida, removeu os **botões de face sensíveis à pressão**, recurso existente no DualShock 3 e que é justamente o foco da proposta deste trabalho.

Seu sucessor, o **DualSense**, introduziu tecnologias ainda mais avançadas, como **feedback háptico** e **gatilhos adaptáveis**, proporcionando maior imersão ao jogador. Apesar disso, o DualShock 4 continua sendo amplamente utilizado devido à sua compatibilidade com PlayStation 4 e computadores.

Entre os concorrentes diretos, o **Xbox Wireless Controller** destaca-se pela simplicidade, ergonomia e ampla compatibilidade com Windows, porém não oferece recursos como touchpad ou sensores de movimento. Já o **Nintendo Switch Pro Controller** incorpora giroscópio, acelerômetro e vibração HD, mas também não possui touchpad.

Controles mais recentes de fabricantes terceiras, como o **8BitDo Ultimate Bluetooth**, demonstram a evolução da categoria ao utilizar **sensores Hall Effect**, que praticamente eliminam o problema de *drift* dos analógicos e aumentam sua vida útil.

A proposta deste projeto consiste em combinar características de diferentes gerações da linha PlayStation, reintroduzindo ao **DualShock 4** uma funcionalidade presente no **DualShock 3** (botões de face sensíveis à pressão), mantendo os recursos modernos do DS4. Dessa forma, o protótipo busca unir funcionalidades clássicas e atuais em uma solução embarcada baseada em ESP32.
