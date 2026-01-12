# Monitoramento de Redes com Zabbix e n8n

## 📌 Objetivo do Projeto
Este projeto tem como objetivo demonstrar a integração entre o **Zabbix** e o **n8n** para criação de um fluxo automatizado de **monitoramento de redes**, permitindo o envio de alertas e execução de ações automáticas a partir de eventos detectados pelo Zabbix.

A solução utiliza **Media Types personalizados**, **Webhooks** e **workflows no n8n**, tornando o monitoramento mais flexível e extensível.

---

## 🧰 Tecnologias Utilizadas
- **Zabbix Server**
- **n8n**
- **Docker**
- **Ubuntu Server**
- **Webhook HTTP**
- **JSON**

---

## 📈 Visão Geral do Fluxo
1. O **Zabbix** monitora dispositivos e serviços da rede.
2. Ao detectar um problema (ex: *ping_loss*), o Zabbix dispara uma **Action**:
<img width="1407" height="727" alt="Action-Operation" src="https://github.com/user-attachments/assets/99554ee1-410f-46f9-9b24-61eb0a9788ff" />

3. A Action utiliza um **Media Type HTTP**:
<img width="1849" height="489" alt="Users-Notification-Media" src="https://github.com/user-attachments/assets/7b805fe1-72d9-4ca6-8624-820d9ee66b88" />

4. Os dados do evento são enviados via **Webhook** para o **n8n**:
<img width="537" height="748" alt="webwook" src="https://github.com/user-attachments/assets/6196d27b-6991-4ccb-a413-80b75dbf14b7" />

5. O **n8n** processa as informações e executa ações automatizadas (ex: notificação, registro ou integração com terceiros).


---

## 📂 Estrutura do Repositório
```text
Monitoramento-de-Redes-com-Zabbix-e-N8N/
├── docs/
│   ├── zbx_export_mediatypes.yaml
│   └── workflow_n8n.json
└── Fluxo_no_N8N.md
└── README.md
```
## Documentação

📄 [Descrição do fluxo no n8n](/Fluxo_no_N8N.md)

⚙️ [Media Type do Zabbix](https://github.com/Lesszera/Monitoramento-de-Redes-com-Zabbix-e-N8N/blob/main/docs/MEDIATYP_%20ZABBIX.yaml)

## 📥 Importação e Configuração
🔹 Importar Media Type no Zabbix

Acesse o painel Web do Zabbix

Vá em:
```
Administration → Media types
```

Clique em Import

Selecione o arquivo:

 [Media Type do Zabbix](https://github.com/Lesszera/Monitoramento-de-Redes-com-Zabbix-e-N8N/blob/main/docs/MEDIATYP_%20ZABBIX.yaml)


Marque as opções de importação e confirme

🔹 Importar Workflow no n8n

Acesse a interface do n8n

Vá em Workflows

Clique em Import from File

Selecione o arquivo:

[WORKFLOW](https://github.com/Lesszera/Monitoramento-de-Redes-com-Zabbix-e-N8N/blob/main/docs/WORKFLOW)

## ⚙️ Configurações Importantes

Ajustar a URL do Webhook do n8n no Media Type do Zabbix

Garantir que o Zabbix tenha acesso ao endpoint HTTP do n8n

Verificar permissões e autenticação, se aplicável

## 🧠 Observações Técnicas

O Media Type utiliza macros do Zabbix como:

{EVENT.ID}

{EVENT.NAME}

{EVENT.SEVERITY}

{HOST.NAME}

O fluxo no n8n foi estruturado para facilitar expansão e integração com outros serviços

## 📌 Possíveis Expansões

Integração com Discord ou Telegram

Ações automáticas de correção

Registro de eventos em banco de dados

Dashboards personalizados.
