# Node por node do fluxo N8N
Descrição do funcionamento e lógica do fluxo [ping_loss](https://github.com/Lesszera/Monitoramento-de-Redes-com-Zabbix-e-N8N/blob/main/docs/WORKFLOW)


## 1. Webhook (Webhook)
Tipo: Webhook (HTTP POST)

O que faz: recebe a chamada HTTP do Zabbix com os parâmetros do evento, iniciando o fluxo (método POST).
<img width="537" height="748" alt="webwook" src="https://github.com/user-attachments/assets/b06eb0ce-006c-4f19-b394-887b5e51504a" />

Dados importantes disponíveis: body.host_name, body.host_ip, body.event_opdata ou outros campos enviados pelo Zabbix — usados nas expressões dos nodes seguintes.

## 2. Zabbix HTTP Request 1
Tipo: API Call (HTTP Request)

O que faz: consulta a API do Zabbix para buscar itens (icmppingloss) filtrados pelo host recebido no webhook.
<img width="425" height="667" alt="Node HTTP REQUEST" src="https://github.com/user-attachments/assets/1f734021-6760-4488-81c8-e9454e271b1d" />

Configuração chave observada:
```
{
  "jsonrpc": "2.0",
  "method": "item.get",
  "params": {
    "output": ["itemid","name","key_","lastvalue","units"],
    "search": {"key_":"icmppingloss"},
    "filter": {"host": ["{{ $json.body.host_name }}"]},
    "sortfield":"name","sortorder":"ASC"
  },
  "id":1
}
```
Monta e envia a requisição POST para http://"zabbix-server-ip"/zabbix/api_jsonrpc.php usando as credenciais definidas (credential Zabbix account).

## 3. Switch 1
Tipo: Switch

O que faz: avalia o valor retornado pelo Zabbix (result[0].lastvalue) e cria caminhos de execução diferentes conforme regras (=100%, >40%).
<img width="1918" height="887" alt="image" src="https://github.com/user-attachments/assets/8e77f813-72b0-4372-b037-fe91b269af45" />

Regras observadas:

Saída 100% → neste fluxo a saída conecta para No Operation, do nothing (ou outra ação conforme sua configuração).

Saída 40% (>= 40) → encaminha para o node Execute a command (SSH) para tentar uma ação corretiva.

Dica: reveja a ordem das regras: switches avaliam sequencialmente — certifique-se que a regra mais específica venha antes da mais genérica.

## 4. Execute a command — SSH
Tipo: SSH

O que faz: executa um comando remoto via SSH no host alvo.
```
command: restart & exit
```
<img width="1257" height="583" alt="image" src="https://github.com/user-attachments/assets/1550c486-afbf-469d-bbf0-a7f735906901" />

Recebe o item do Switch (quando lastvalue >= 40 no exemplo), abre conexão SSH usando as credenciais configuradas, captura a saída e status do comando e encaminha o resultado para o node Wait.

Observações de segurança: confirme que a conta SSH tem permissões adequadas, que os host da rede permitem SSH, que as credenciais dos hosts sejam iguais para todos os host e avalie riscos de executar restart automaticamente.

## 5. Wait
Tipo: Wait (espera)

O que faz: pausa o fluxo por um período configurado (2 minutos) antes de prosseguir — usado para dar tempo ao dispositivo/ação corretiva surtir efeito.

<img width="1248" height="565" alt="image" src="https://github.com/user-attachments/assets/6506b089-d2ac-47c6-b415-74f513e5cda8" />

Após o tempo expirar, passa para o próximo node (Zabbix HTTP Request 2) para re-checar o estado.

## 6. Zabbix HTTP Request 2
Tipo: HTTP Request (nova chamada à API do Zabbix)

O que faz: reconsulta o Zabbix após o término do tempo de espera para verificar se o lastvalue (ex.: perda ICMP) mudou. Similar ao Zabbix HTTP Request 1.

Configuração chave observada: corpo JSON semelhante ao item.get buscando icmppingloss e filtrando pelo host.

## 7. Switch 2
No seu fluxo existem duas variantes em exports distintos: um Switch1 (com regras semelhantes ao primeiro Switch) e/ou um node If (que testa result[0].lastvalue >= 40). Ambas fazem o papel de decidir se tomar nova ação ou encerrar.

Tipo: Switch.

Recebe o resultado atualizado do Zabbix HTTP Request 2.

Avalia a condição:

Se o valor for igual a 100 (ex.: =100) → Segue para o node No Operation, isso indica que o host não reiniciou corretamente, evidenciando a necessidade de intervenção humana.

Se o valor ainda estiver alto (ex.: >= 40) → segue para Execute a command novamente (pode tentar nova intervenção).

Se o valor estiver dentro do esperado (ex.: <= 40), encerra o fluxo.

Observação: esse padrão cria um laço de tentativa → espera → validação. Se for preciso limitar tentativas, adicione um contador (ex.: incrementar um campo no item e checar limite) para evitar loops infinitos.

## 8. No Operation, do nothing
Tipo: NoOp

O que faz(não faz:/): ponto final que não realiza ação — serve para encerrar elegantemente um caminho sem erros.

## 9. Conexões e fluxo lógico (resumo sequencial)
Webhook recebe evento externo.
Zabbix HTTP Request 1 consulta o item (icmppingloss) do host.
Switch 1, decide com base em lastvalue: Se condição crítica → Execute a command (SSH). Senão → No Operation e fim de ramo.
Execute a command faz ação corretiva.
Wait (espera 2 minutos)
Zabbix HTTP Request 2 reconsulta o Zabbix.
Switch 2, valida resultado pós-correção: Se ainda fora do aceitável → repetir Execute a command (ou outro tratamento). Se não envia sinal para NoOp.
No Operation (fim).

## 10. Recomendações práticas / pontos a conferir
Confirme qual expressão de host usar na filter ({{ $json.body.host_name }}.

A função .toNumber adicionada ao valor do evento é necessária para converter o valor da perda de pacote (que é recebido como string) para um Int (um número inteiro é necessário para a comparaçãodo switch). Ex.:
```
={{ $json.result[0].lastvalue.toNumber() }}
```
Evite loops infinitos: implemente contador de tentativas ou limite de re-execuções.

Valide permissões da conta SSH e a segurança do comando remoto (especialmente comandos que fazem restart).

Teste com pinData (resultado simulado) para validar roteamento sem mexer no Zabbix de produção.

## 11. Workflow completo: 
<img width="1822" height="756" alt="Ping Loss Workflow" src="https://github.com/user-attachments/assets/c775e458-e103-4369-8684-929a493d0b54" />

