# Fluxo n8n — Monitoramento de Ping Loss (Node por Node)

Descrição detalhada do funcionamento e da lógica do fluxo **[ping_loss](https://github.com/Lesszera/Monitoramento-de-Redes-com-Zabbix-e-N8N/blob/main/docs/WORKFLOW)**, responsável por receber eventos do Zabbix, avaliar perda de pacotes ICMP e executar ações corretivas automatizadas via n8n.

---

## 🔁 Resumo do Fluxo
Webhook → Zabbix API → Switch → SSH → Wait → Zabbix API → Switch → Encerramento

---

## 1. Webhook (HTTP POST)
**Tipo:** Webhook  

Recebe a chamada HTTP enviada pelo Zabbix contendo os dados do evento e inicia o fluxo.

<img src="https://github.com/user-attachments/assets/b06eb0ce-006c-4f19-b394-887b5e51504a" />

**Dados importantes recebidos:**
- `body.host_name`
- `body.host_ip`
- `body.event_opdata`

Esses dados são utilizados nos nodes seguintes para consultas e decisões.

---

## 2. Zabbix HTTP Request 1 — Consulta de Ping Loss
**Tipo:** HTTP Request (API Zabbix)

<img width="425" height="667" alt="Node HTTP REQUEST" src="https://github.com/user-attachments/assets/3ef29e8a-56e3-46fb-9ae6-b9c7fc811677" />

Consulta a API do Zabbix para buscar o item `icmppingloss` do host recebido no webhook.

```json
{
  "jsonrpc": "2.0",
  "method": "item.get",
  "params": {
    "output": ["itemid", "name", "key_", "lastvalue", "units"],
    "search": { "key_": "icmppingloss" },
    "filter": { "host": ["{{ $json.body.host_name }}"] },
    "sortfield": "name",
    "sortorder": "ASC"
  },
  "id": 1
}
```
## 3. Switch 1 — Avaliação Inicial

Tipo: Switch

<img width="1906" height="889" alt="Switch Node" src="https://github.com/user-attachments/assets/79476dc2-0b7f-4d2a-a4f0-2ad7d9a506db" />


Avalia o valor lastvalue retornado pelo Zabbix.

Regras:

= 100 → Nenhuma ação automática

>= 40 → Executa ação corretiva via SSH

⚠️ Importante: usar .toNumber() para conversão correta:

```json
={{ $json.result[0].lastvalue.toNumber() }}
```

## 4. Execute Command — SSH

Tipo: SSH

<img width="567" height="868" alt="SSH Node-Parameters" src="https://github.com/user-attachments/assets/747363ba-a33b-45d7-83fb-78c59c3248c8" />


Executa um comando remoto no host afetado para tentativa de correção.
```
restart && exit
```
<img src="https://github.com/user-attachments/assets/1550c486-afbf-469d-bbf0-a7f735906901" />

## 5. Wait — Tempo de Estabilização

Tipo: Wait

Pausa o fluxo por 2 minutos para permitir que a ação corretiva tenha efeito.

## 6. Zabbix HTTP Request 2 — Revalidação

Reconsulta o Zabbix após o tempo de espera para verificar se a perda de pacotes foi normalizada.

## 7. Switch 2 — Validação Final

Avalia novamente o lastvalue:

= 100 → Encaminha para NoOp (necessita intervenção humana)

>= 40 → Pode repetir a ação corretiva

<= 40 → Encerra o fluxo com sucesso

Recomenda-se implementar limite de tentativas para evitar loops infinitos.

## 8. No Operation (NoOp)

Node final que encerra o fluxo sem executar ações adicionais.

## 9. Considerações de Segurança

Garantir permissões adequadas da conta SSH

Avaliar riscos de execução automática de comandos

Preferir testes com pinData antes de produção

## 10. Workflow Completo
<img src="https://github.com/user-attachments/assets/c775e458-e103-4369-8684-929a493d0b54" /> 

