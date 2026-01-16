# LetsDefend SOC166: Detecção de Código JavaScript em URL (XSS)

Este write-up descreve o processo de triagem e investigação de uma tentativa de ataque de Cross-Site Scripting (XSS) Refletido contra um servidor web corporativo.

## 1. Detalhes do Alerta

* **ID do Evento:** 116
* **Regra:** SOC166 - Javascript Code Detected in Requested URL
* **Host Alvo:** WebServer1002 (172.16.17.17)
* **Origem:** 112.85.42.13 (IP Externo)
* **Data/Hora:** Feb, 26, 2022, 06:56 PM

## 2. Investigação e Análise de Logs

### 2.1. Análise do Payload
O atacante tentou injetar o seguinte código no parâmetro de busca da URL:
`https://172.16.17.17/search/?q=<$script>javascript:$alert(1)<$/script>`

O uso de caracteres como `$` e `<$` sugere uma técnica de **evasão de WAF** para tentar contornar filtros de segurança baseados em assinaturas simples.

### 2.2. Verificação de Sucesso (Baselining de Logs)
Para determinar se o ataque foi bem-sucedido, comparamos a requisição maliciosa com uma busca legítima realizada no sistema:

* **Busca Legítima (`?q=test`):** Status **HTTP 200** e tamanho de resposta de **885 bytes**.
* **Tentativa de Ataque (XSS):** Status **HTTP 302** (Redirecionamento) e tamanho de resposta de **0 bytes**.

**Conclusão:** O servidor redirecionou a requisição e não retornou conteúdo, indicando que o script **não foi refletido nem executado** no navegador.

## 3. Análise do Endpoint e Contexto

A análise do host `WebServer1002` (Windows Server 2019) revelou o seguinte:
* **Usuário:** webadmin15.
* **Histórico do Navegador:** O histórico mostra apenas atividades legítimas de administração (instalação de Docker e certificados SSL) realizadas em 02/02/2022.
* **Veredito:** O ataque não teve origem interna e não houve interação do usuário administrativo com o IP malicioso.

## 4. Playbook e Resolução

1. **Malicioso?** Sim (True Positive). Houve tentativa clara de injeção de código.
2. **Tipo de Ataque:** Reflected XSS.
3. **Teste Planejado?** Não. Nenhuma evidência de Pentest ou simulação foi encontrada no Mailbox.
4. **Sucesso?** Não. O servidor barrou a execução conforme evidenciado pelo status HTTP 302.
5. **Escalação:** Não é necessária escalação para Tier 2, pois o ataque falhou e não houve comprometimento de rede interna.

---

**Recomendações:**
* Bloquear o IP `112.85.42.13` no Firewall de borda.
* Revisar as regras de sanitização de entrada na aplicação web.
