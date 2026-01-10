## Monitoramento de Redes com Zabbix e N8N

# 📒 Descrição
Esse projeto foi desenvolvido com o objetivo de criar um sistema capaz de monitorar, identificar e resolver problemas em redes LAN de forma imediata e automática. Tal projeto foi elaborado ao CETEP Bacia do Jacuípe como parte das atividades acadêmicas do Curso de Redes de Computadores.

# 🛠 Softwares Utilizados
a) Ubuntu Server, ambiente escolhido para suportar o projeto;

b) Zabbix Server, responsável pelo monitoramento da rede e identificação de problemas;

d) Docker, utilizado como gerenciador de containers;

e) N8N, executado dentro de um container Docker, responsável pela automação de ações corretivas.

# 🤖 Lógica da Automação
A automação de ações corretivas foi realizada utilizando a plataforma n8n, que, mediante alertas enviados pelo Zabbix, executava fluxos de análise e tomada de decisão. A sequência de execuções pode ser entendida da seguinte forma:

Um trigger(gatilho) é acionado quando o zabbix detecta algum problema.

O problema é notificado para um perfil no zabbix através de uma action(ação).

Um mediatype vinculado ao perfil cria um HTTP call para enviar os parâmetros do evento para o N8N

O N8N recebe os parâmetros pelo trigger node webhook

Com os parâmetros do evento, o fluxo no N8N executa uma ação para resolver o problema.

O fluxo checa se o problema foi resolvido e, caso não, repete a ação, até que o problema seja resolvido.

# 🌐 Integração
## 1. HTTP call - Webhook N8N
Para a correta execução do workflow é necessário que a integração entre o zabbix e o N8N seja feita de maneira correta.

1.1 Trigger

A maioria dos gatilhos já vem configurados por padrão no zabbix, caso seu problema não esteja entre eles, é necessário criá-lo. O gatilho usado para esse projeto foi o de perda de pacotes: 
<img width="1400" height="727" alt="trigger" src="https://github.com/user-attachments/assets/e02e7580-d7ba-43f9-bf47-cd0fb820e0a1" />

1.2 Action

A ação é necessária para o envio dos parâmetros. Essa deve ser configurada de acordo ao problema. Nesse exemplo, a condição para o acionamento da ação(imagem 1) é o status do trigger ser igual a TRUE. Quando a condição for cumprida, a ação tomada(imagem 2) sera a operação de notificar o perfil admin e o mediatype n8n_ping_loss.

Condição da ação: <img width="1400" height="434" alt="ação2" src="https://github.com/user-attachments/assets/69d5e16c-aa8d-4d39-be93-80266a918467" />


Operação da ação: <img width="1407" height="727" alt="ação" src="https://github.com/user-attachments/assets/c0f6d9c4-1685-4879-99ff-57aa9a9deae5" />


1.3 Mediatype

O mediatype zbx_export_mediatypes.yaml é o que enviará as métricas para o N8N. Nele deve ser modificado o valor do campo URL para o endereço URL do webhook do workflow no N8N. O mediatype deve ser adicionado na aba user > notification(imagem 1).

Notificação: <img width="1849" height="489" alt="notificações" src="https://github.com/user-attachments/assets/9f4d7a6a-4454-486a-bb00-e92e31207f96" />


1.4 Webhook

O webhook irá receber os parametros do mediatype via HTTP call na URL correpondente ao webhook: #####imagem aqui#####

## 2 HTTP request - API zabbix
A API do Zabbix possibilita coletar metricas de hosts através de um HTTP request. Esse método é usado para verificar a gravidade do problema.

2.1 Node HTTP request

O N8N possui um node nativa para o zabbix. Através dele é possivel obter as metricas do evento desejado. Para isso foi configurado um HTTP request pelo metodo post, assim os parametros serão carregados pelo corpo da mensagem.

######imagem aqui#####
Para requisitar apenas as metricas necessarias e preciso especificar o corpo da requisição. O corpo usado tem o seguinte formato:

{
  "jsonrpc": "2.0",
  "method": "item.get",
  "params": {
    "output": ["itemid", "name", "key_", "lastvalue", "units"],
    "search": {
    "key_": "icmppingloss"
    },
    "filter": {"host": ["{{ $json.body.host_name }}"] },
    "sortfield": "name",
    "sortorder": "ASC"
  },
  "id": 1
}
2.2 Credencial Zabbix

Para que a requisição seja aceita pela API do zabbix é preciso de uma credencial de acesso. São necessarios a URL do zabbix e um token da API, o segundo pode ser gerado dentro do zabbix na aba Users>API tokens.
<img width="1849" height="889" alt="api" src="https://github.com/user-attachments/assets/b6e48bd8-fddd-42b8-bbb6-2534e03eb32d" />
Workflow completo: <img width="1811" height="756" alt="workflow" src="https://github.com/user-attachments/assets/4a6628f4-0246-40ff-bd55-2fffc371b080" />

# ⚠️ Importante
Este projeto é apenas um protótipo e que não deve ser usado em ambientes reais devido as suas brechas de segurança.
