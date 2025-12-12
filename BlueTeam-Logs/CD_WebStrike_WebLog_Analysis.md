# 🔍 CyberDefenders WebStrike: Análise de Logs Web e Rastreamento de Ataque

Este documento detalha a metodologia de análise de logs de acesso e de erro de um servidor web para identificar e rastrear a execução de um ataque, simulando um incidente de segurança.

## 1. Metodologia de Análise de Logs

* **Ferramenta Principal:** **`grep`**, **`cat`** e **`awk`** (ou Splunk/ElasticSearch, se usado).
* **Foco da Análise:** Busca por códigos de resposta HTTP incomuns (como `400 Bad Request` ou `500 Internal Server Error`) e por caracteres de *Payload* nos campos `URI` e `User-Agent`.

## 2. Indicadores de Comprometimento (IoCs)

### 2.1. Identificação do Payload de Ataque

O ataque foi identificado através da busca por padrões de código malicioso no campo URI.
* **Payload Comum:** Strings como `union select`, `etc/passwd`, `wget`, `curl`, ou `base64_decode`.
* **Ação:** O rastreamento do endereço IP de origem do atacante (ex: `10.10.X.X`) foi crucial para isolar todas as suas atividades.

### 2.2. Rastreamento da Execução Bem-Sucedida

* **Resposta Crítica:** O sucesso da exploração foi confirmado por um código de resposta `200 OK` associado a uma requisição maliciosa, seguido por um acesso subsequente a um novo arquivo (indicando um *webshell* ou a exfiltração de dados).

## 3. Conclusão (Incident Response)

A conclusão do desafio exigiu a determinação da hora exata do comprometimento, o *payload* utilizado e a prova da persistência/execução.
