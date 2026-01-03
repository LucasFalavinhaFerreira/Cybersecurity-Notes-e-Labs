# CyberDefenders - PoisonedCredentials: Análise de Envenenamento e Credenciais

Este write-up descreve a investigação de um incidente de rede envolvendo envenenamento de protocolos de resolução de nomes e a subsequente captura de credenciais NTLMv2.

## 1. Identificação do Typo e Envenenamento (LLMNR/NBNS)

O ataque iniciou-se quando uma máquina vítima (`192.168.232.162`) tentou acessar um recurso de rede inexistente, gerando consultas de broadcast que foram exploradas pelo atacante.

* **Metodologia:** Utilização de filtros no Wireshark para isolar protocolos de resolução de nomes.
* **Filtro Aplicado:** `ip.addr == 192.168.232.162 && (llmnr || nbns)`
* **Vulnerabilidade:** A máquina vítima enviou consultas para um nome digitado incorretamente (*typo*), permitindo que o atacante respondesse como se fosse o recurso legítimo.

## 2. Identificação da Máquina Atacante (Rogue Machine)

O atacante foi identificado ao responder às consultas de broadcast da vítima, alegando possuir o endereço do recurso solicitado.

* **IP do Atacante:** `192.168.232.215`
* **Vetor:** O atacante utilizou ferramentas (como o Responder) para envenenar o cache da vítima e forçar a autenticação em seu próprio sistema.

## 3. Análise de Credenciais Comprometidas

Após o envenenamento, a vítima tentou se autenticar no sistema do atacante via SMB/HTTP, expondo o hash de suas credenciais.

* **Filtro de Autenticação:** `ntlmssp`
* **Usuário Identificado:** Através da análise dos pacotes de negociação NTLM (Type 3 Message), o nome de usuário foi extraído em texto claro no campo **User**.
* **Conta Comprometida:** `janesmith`

## 4. Identificação do Hostname Alvo via SMB

O desafio final consistiu em identificar o hostname real da máquina que o atacante acessou via SMB. Embora os cabeçalhos genéricos mostrassem "WORKSTATION", a análise profunda do handshake de autenticação revelou a identidade real.

* **Metodologia de Análise Profunda:** Inspeção do pacote de resposta do servidor (`Session Setup Response`) contendo o `NTLMSSP_CHALLENGE`.
* **Caminho da Evidência:**
    * `SMB2` -> `Session Setup Response` -> `Security Blob` -> `NTLMSSP_CHALLENGE` -> `Target Info`.
* **Hostname Identificado:** **ACCOUNTINGPC**
    * O nome foi extraído do atributo **DNS Computer Name** dentro da estrutura de informações do alvo (Target Info).

---

**Lição:** Cabeçalhos de protocolo podem conter placeholders ou informações falsificáveis (como "WORKSTATION"). A verdade forense reside nos campos de autenticação e informações de sistema (Target Info) trocados durante o handshake de segurança.
