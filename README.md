README — Monitoramento de Clima com FIWARE

👥 Integrantes
 Guilherme Acacio rm 562475
 Gustavo Mendez rm 563753
 Enrico Almeida rm 563265


Requisitos rápidos

 1 Acesso ao servidor FIWARE (Orion Context Broker, IoT-Agent, STH-Comet, etc.).

 2 Ferramenta para fazer requisições HTTP (ex.: curl, Postman).

 3 Ter o device_id e os atributos do seu dispositivo (por ex.: p para potenciômetro).




Resumo dos passos (o que você vai fazer)

 1 Provisionar (configurar) o dispositivo no IoT-Agent — dizer ao FIWARE quais atributos o dispositivo tem.

 2 Registrar comandos no Orion — configurar notificações/ações para o dispositivo.

 3 Listar dispositivos provisionados — checar se ficou tudo certo.

 4 Ler um atributo no Orion — pedir o valor atual (ex.: valor do potenciômetro).

 5 (Se precisar) Deletar o dispositivo — remover do IoT-Agent e do Orion.

 6 Assinar notificações para STH-Comet — mandar mudanças para o histórico temporal.

 7 Pedir série temporal no STH-Comet — consultar histórico (últimos N pontos).




Passo 1 — Provisionar o dispositivo (IoT-Agent)

O que é: dizer ao IoT-Agent como é seu dispositivo (ID, atributos, comandos, protocolo).

JSON de exemplo — adapte as partes destacadas (id, atributos, protocolo, transport):

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

O que mudar: coloque o device_id do seu dispositivo e os attributes corretos (Text, Integer, Float, etc.).

Resultado esperado: o IoT-Agent responde confirmando que o dispositivo foi criado.




Passo 2 — Registrar comandos no Orion

O que é: criar um subscription/registro para os comandos (por exemplo, on e off) que chegam da aplicação.

JSON de exemplo:

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

O que mudar: coloque a id/type do seu dispositivo e a url do serviço que receberá notificações.

Resultado esperado: Orion aceita o registro e encaminhará eventos ao provider.




Passo 3 — Listar devices provisionados (verificar)

O que é: pedir ao IoT-Agent a lista de devices cadastrados para confirmar se o seu dispositivo apareceu com os atributos corretos.

Como fazer: usar a API do IoT-Agent (ex.: via navegador ou curl) para listar devices.
O que procurar: device_id, entity_name, attributes — tudo deve corresponder ao que você enviou.



Passo 4 — Ler um atributo no Orion

O que é: pedir ao Orion o valor atual de um atributo (ex.: p — potenciômetro).

Exemplo de URL (GET): http://{{url}}:1026/v2/entities/urn:ngsi-ld:device:001/attrs/p

O que você receberá: um JSON com o valor atual do atributo p.
Ex.: { "value": 42 } (dependendo do formato do seu IoT-Agent, pode ter pequenas diferenças).




Passo 5 — Deletar dispositivo (se necessário)

Se quiser remover o dispositivo do sistema:

 1 Deletar no IoT-Agent (remove o provisioning):   http://{{url}}:4041/iot/devices/device001
 
 2 Deletar no Orion (remove a entidade/declaracão):  http://{{url}}:1026/v2/entities/urn:ngsi-ld:device:001

Observação: sempre confirme com GETs antes/depois para garantir que foi removido.



Passo 6 — Assinar notificações para STH-Comet (Salvar histórico)

O que é: criar uma subscription no Orion que notifica o STH-Comet quando o atributo p mudar — assim o STH grava a série temporal.

JSON de exemplo:
{
  "description": "Notify STH-Comet of all Motion Sensor count changes",
  "subject": {
    "entities": [
      {
        "id": "urn:ngsi-ld:device:001",
        "type": "device"
      }
    ],
    "condition": { "attrs": ["p"] }
  },
  "notification": {
    "http": {
      "url": "http://{{url}}:8666/notify"
    },
    "attrs": [ "p" ],
    "attrsFormat": "legacy"
  }
}
O que mudar: id, type, attrs e url do STH-Comet.

Resultado esperado: STH-Comet será notificado e começará a guardar as leituras de p.




Passo 7 — Solicitar série temporal no STH-Comet

O que é: pedir os últimos N pontos do atributo p que o STH armazenou.

Exemplo de URL:  http://{{url}}:8666/STH/v1/contextEntities/type/device/id/urn:ngsi-ld:device:001/attributes/p?lastN=30

O que isso pede: os últimos 30 valores registrados para p. A resposta vem em JSON com a série temporal.



Glossário simples

 1 IoT-Agent: “intérprete” que conhece seu tipo de dispositivo e converte mensagens para o FIWARE.

 2 Orion Context Broker: guarda o estado atual das entidades (por exemplo, o valor atual do potenciômetro).

 3 STH-Comet: guarda o histórico (séries temporais) das medições.

 4 Provisionar: cadastrar / registrar um dispositivo no IoT-Agent.

 5 Entity (entidade): representação do dispositivo no Orion (ex.: urn:ngsi-ld:device:001).

 6 Attribute (atributo): uma característica que pode mudar (ex.: p, state).



Dicas fáceis (para evitar problemas)

 1 Sempre troque {{url}} pelo endereço correto.

 2 Use device_id único para cada dispositivo.

 3 Se a API pedir autenticação, coloque as credenciais corretamente.

 4 Se um passo der erro, copie a mensagem de erro — ela ajuda a descobrir o problema.

 5 Teste primeiro com poucos pontos (lastN=5) para ver se os dados aparecem.


 

Problemas comuns & soluções rápidas

 1 Resposta vazia ao pedir atributo: verifique se o dispositivo está devidamente provisionado e se o IoT-Agent está recebendo mensagens.

 2 STH não mostra histórico: confira se a subscription aponta corretamente para http://{{url}}:8666/notify.

 3 Erro 404 ao deletar: confirme o device_id e a URL (porta e caminho).

 4 Dados com formato inesperado: verifique o type dos atributos no provisioning (Integer, Float, Text).




Final (resumindo)

 1 Provisionar no IoT-Agent (dizer como é o device).

 2 Registrar no Orion (commands/subscriptions).

 3 Verificar (listar devices).

 4 Ler atributos no Orion.

 5 Deletar quando necessário (IoT-Agent e Orion).

 6 Assinar mudanças para o STH-Comet (guardar histórico).

 7 Pedir séries temporais ao STH-Comet.

<img width="1505" height="919" alt="image" src="https://github.com/user-attachments/assets/4eb3b1ae-b9c4-417d-a8a7-b278bea18755" />






 
