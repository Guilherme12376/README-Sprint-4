README — Monitoramento de Clima com FIWARE

👥 Integrantes
 Guilherme Acacio rm 562475
 Gustavo Mendez rm 563753
 Enrico Almeida rm 563265

🎯 Visão Geral

Este projeto conecta um dispositivo IoT (como um sensor de temperatura, umidade ou potenciômetro) ao FIWARE, permitindo:

📡 Coletar dados em tempo real via IoT-Agent.

💾 Armazenar e consultar informações com o Orion Context Broker.

📈 Visualizar medições e histórico com o STH-Comet e uma interface web interativa.

Tudo isso forma uma solução prática de monitoramento climático com arquitetura FIWARE.

🧭 Arquitetura do Sistema
[Sensor IoT] → [IoT-Agent] → [Orion Context Broker] → [STH-Comet]
       ↓                                      ↑
        └──────────── Interface Web (HTML + Chart.js) ─────────────┘


IoT-Agent: Traduz os dados enviados pelo sensor para o FIWARE.

Orion: Armazena e disponibiliza os valores atuais.

STH-Comet: Mantém o histórico das leituras.

Interface Web: Mostra os valores e gráficos em tempo real.

⚙️ Pré-requisitos

Acesso ao FIWARE Orion Context Broker, IoT-Agent e STH-Comet.

Ferramenta para chamadas HTTP (ex.: Postman, Insomnia ou curl).

Servidor configurado com IP acessível (http://44.223.0.185 no exemplo).

Um dispositivo com ID único e atributos definidos (ex.: p para potenciômetro).

🚀 Etapas de Configuração
1️⃣ Provisionar o Dispositivo (IoT-Agent)

Cria-se o registro do sensor no FIWARE, informando atributos e comandos disponíveis.

Requisição:

{
  "devices": [
    {
      "device_id": "device001",
      "entity_name": "urn:ngsi-ld:device:001",
      "entity_type": "device",
      "protocol": "PDI-IoTA-UltraLight",
      "transport": "MQTT",
      "commands": [
        { "name": "on", "type": "command" },
        { "name": "off", "type": "command" }
      ],
      "attributes": [
        { "object_id": "s", "name": "state", "type": "Text" },
        { "object_id": "p", "name": "p", "type": "Integer" }
      ]
    }
  ]
}


Enviar para:
POST http://{{url}}:4041/iot/devices

Retorno esperado: Confirmação de criação (201 Created).

2️⃣ Registrar no Orion Context Broker

Informa ao Orion que este dispositivo existe e que ele deve aceitar comandos vindos da aplicação.

Requisição:

{
  "description": "device Commands",
  "dataProvided": {
    "entities": [
      { "id": "urn:ngsi-ld:device:001", "type": "device" }
    ],
    "attrs": ["on", "off"]
  },
  "provider": {
    "http": { "url": "http://{{url}}:4041" },
    "legacyForwarding": true
  }
}

3️⃣ Verificar Dispositivos Provisionados

Confirme se o IoT-Agent registrou o dispositivo corretamente:
GET http://{{url}}:4041/iot/devices

Você deve ver o device_id e os atributos listados.

4️⃣ Consultar Valor do Sensor

Para visualizar o valor atual de um atributo (ex.: p — potenciômetro):

GET http://{{url}}:1026/v2/entities/urn:ngsi-ld:device:001/attrs/p


Retorno exemplo:

{ "value": 47 }

5️⃣ (Opcional) Remover o Dispositivo

Para excluir o dispositivo do IoT-Agent e do Orion:

DELETE http://{{url}}:4041/iot/devices/device001
DELETE http://{{url}}:1026/v2/entities/urn:ngsi-ld:device:001

6️⃣ Criar Assinatura para Histórico (STH-Comet)

Permite que o Orion envie notificações ao STH-Comet sempre que o valor p mudar.

{
  "description": "Notify STH-Comet of Potentiometer changes",
  "subject": {
    "entities": [
      { "id": "urn:ngsi-ld:device:001", "type": "device" }
    ],
    "condition": { "attrs": ["p"] }
  },
  "notification": {
    "http": { "url": "http://{{url}}:8666/notify" },
    "attrs": ["p"],
    "attrsFormat": "legacy"
  }
}

7️⃣ Consultar Histórico no STH-Comet

Para ver os últimos valores registrados:

GET http://{{url}}:8666/STH/v1/contextEntities/type/device/id/urn:ngsi-ld:device:001/attributes/p?lastN=30


Retorna as últimas 30 medições.

💻 Interface Web (Monitoramento de Clima)

A página HTML usa Chart.js para exibir dois gráficos:

📈 Gráfico 1: Variação do potenciômetro ao longo do tempo.

📊 Gráfico 2: Resumo estatístico (mínimo, médio, máximo).

Exemplo visual (simulação):

╔════════════════════════════════════════════════╗
║ 🌦️ Monitoramento de Clima                     ║
╠════════════════════════════════════════════════╣
║ Potenciômetro: [ 45 ]                         ║
║                                                ║
║ 📈 Variação ao longo do tempo                 ║
║ 📊 Resumo das leituras                        ║
╚════════════════════════════════════════════════╝


🔁 O sistema atualiza automaticamente a cada 5 segundos.
💡 O botão do potenciômetro é interativo e responsivo.

🧩 Glossário Simplificado
Termo	Descrição
IoT-Agent	Responsável por traduzir dados dos dispositivos para o FIWARE.
Orion Context Broker	Armazena e distribui os dados em tempo real.
STH-Comet	Mantém o histórico (séries temporais) das medições.
Entidade (Entity)	Representação lógica do dispositivo dentro do FIWARE.
Atributo (Attribute)	Valor variável do dispositivo (ex.: temperatura, p).
🔍 Dicas Práticas

Substitua {{url}} pelo IP ou domínio do seu servidor FIWARE.

Cada dispositivo precisa de um ID único.

Se algo falhar, verifique logs do IoT-Agent e Orion.

Teste primeiro com lastN=5 para validar o histórico.

Use Postman para facilitar os testes de requisições.

🏁 Resultado Final

Ao concluir todas as etapas, você terá:

O painel web exibindo as medições em tempo real.

O Orion registrando os valores atuais.

O STH-Comet armazenando o histórico.

Tudo funcionando de forma integrada, representando um sistema completo de monitoramento climático com FIWARE. 🌎
