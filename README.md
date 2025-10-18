# Análise de Vulnerabilidades: Ataque de Força Bruta com Medusa

**Autor:** Madison S. Oliveira - Especialista em Cibersegurança

## 📜 Visão Geral

Olá! Meu nome é Madison S. Oliveira. Este repositório documenta a solução prática para o desafio final do **Bootcamp de Cibersegurança da DIO**. O objetivo é implementar e documentar um cenário de teste de invasão (pentest), utilizando Kali Linux e a ferramenta Medusa em um ambiente vulnerável para simular ataques de força bruta e, mais importante, exercitar as contramedidas de prevenção.

---

## 🎯 Cenário: Teste de Invasão no "Tabajara Club"

Nesta simulação, assumimos o papel de um pentester contratado pela empresa "Tabajara Club" para identificar vulnerabilidades em seu servidor. O escopo do contrato autoriza a execução de testes para identificar falhas de segurança e tentar obter acesso ao sistema.

Ao final do processo, este documento servirá como relatório técnico, contendo:
1.  **Evidências** do ataque (prints de tela).
2.  **Comandos** executados.
3.  **Documentação** de todo o caminho percorrido.
4.  **Recomendações de mitigação** para proteger a instituição contra ataques similares.

---

## 🛠️ Ambiente e Ferramentas

| Categoria | Tecnologia Utilizada | Propósito |
| :--- | :--- | :--- |
| **Virtualizador** | Oracle VirtualBox | Criação e gerenciamento das máquinas virtuais. |
| **Sistema Atacante** | Kali Linux | SO com ferramentas de pentest pré-instaladas. |
| **Sistema Alvo** | Metasploitable 2 | SO intencionalmente vulnerável para fins de estudo. |
| **Análise e Ataque** | `ping`, `nmap`, `medusa` | Ferramentas para reconhecimento e execução do ataque. |

> **Nota:** Este laboratório foi construído com VirtualBox. No entanto, você pode adaptar o projeto para outras plataformas como **Docker** ou **Proxmox**, de acordo com sua necessidade e domínio técnico.

---

## ⚙️ Montagem do Laboratório

#### 1. Downloads Necessários
* **VirtualBox:** [Página de Downloads](https://www.virtualbox.org/wiki/Downloads) (Baixe os pacotes da plataforma e o *Extension Pack*).
* **Kali Linux (VM):** [Imagens para Máquinas Virtuais](https://www.kali.org/get-kali/#kali-virtual-machines).
* **Metasploitable 2:** [Página do Projeto no SourceForge](https://sourceforge.net/projects/metasploitable/).

#### 2. Configuração de Rede
Após importar as VMs para o VirtualBox, configure a interface de rede de **ambas** para o modo **"Rede Exclusiva de Hospedeiro (Host-Only)"**. Isso cria uma rede interna segura, permitindo que as máquinas se comuniquem apenas entre si.

---

## 🚶‍♂️ Passo a Passo do Ataque

### Etapa 1: Reconhecimento (Máquina Alvo)

Primeiro, precisamos descobrir o endereço IP da nossa máquina alvo (Metasploitable 2).

```bash
# Execute este comando no terminal do Metasploitable 2
ifconfig
```
*No meu ambiente, o IP do alvo foi `192.168.56.103`. Anote o IP que aparecer no seu terminal.*

  <br> </br>
> <img width="727" height="255" alt="Image" src="https://github.com/user-attachments/assets/38ac6045-e756-49f0-af48-854fac441806" />

### Etapa 2: Enumeração de Serviços (Máquina Atacante)

Agora, no Kali Linux, vamos confirmar a conectividade e descobrir os serviços que estão rodando no alvo.

1.  **Verificar se o alvo está ativo na rede:**

    ```bash
    ping -c 3 192.168.56.103
    ```
  <br> </br>
  > <img width="569" height="254" alt="Image" src="https://github.com/user-attachments/assets/eaf9c087-fc8d-4673-a974-4436d170523a" />

2.  **Escanear portas e serviços com Nmap:**
    O IP respondeu. Vamos usar o `Nmap` para identificar portas abertas e as versões dos serviços rodando nelas. Focaremos nos serviços de transferência de arquivos (FTP), acesso web (HTTP) e compartilhamento (SMB).

    ```bash
    nmap -sV -p 21,22,80,139 192.168.56.103
    ```
<br> </br>
<img width="1218" height="315" alt="Image" src="https://github.com/user-attachments/assets/4c9addda-7025-4d04-8eed-84a490f4cd30" />

### Etapa 3: Preparação para o Ataque de Força Bruta

O resultado do `Nmap` identificou o serviço FTP (porta 21) ativo. Uma tentativa de conexão manual confirma que ele exige autenticação:

```bash
ftp 192.168.56.103
```
<br></br>
<img width="305" height="109" alt="Image" src="https://github.com/user-attachments/assets/16599568-423e-42ab-b99c-474ecf3a1f04" />

Sem as credenciais, não podemos acessar. Vamos preparar um ataque de força bruta com a ferramenta **Medusa**. Para isso, criaremos duas listas simples: uma de usuários e outra de senhas.

1.  **Criar a lista de usuários (`usuarios.txt`):**
      
    ```bash echo -e "user\nmsfadmin\nadmin\nroot"> users.txt ```
    
    *Adicione algumas usuario comuns, uma por linha (ex: `msfadmin`, `root`, `helo`, `admin`)*
    <img width="534" height="43" alt="Image" src="https://github.com/user-attachments/assets/9b07e81c-8c50-4fb9-a5fe-4f8d2c15786a" />

3.  **Criar a lista de senhas (`senhas.txt`):**
   bash
     ```echo -e "123456\npassword\nqwerty\nmsfadmin"> pass.txt  ``` 
    
    <img width="489" height="94" alt="Image" src="https://github.com/user-attachments/assets/27aa97ef-59af-45f3-a237-9a4eac557ff1" />
    
    *Adicione algumas senhas comuns, uma por linha (ex: `msfadmin`, `password`, `123456`, `admin`)*

### Etapa 4: Execução do Ataque com Medusa

Com as listas prontas, vamos iniciar o ataque ao serviço FTP.

```bash
 medusa -h 192.168.56.103 -U users.txt -P pass.txt -M ftp -t 6
```
* `-h`: Host alvo.
* `-U`: Arquivo de usuários.
* `-P`: Arquivo de senhas.
* `-M`: Módulo do serviço a ser atacado (FTP).

<img width="1275" height="327" alt="Image" src="https://github.com/user-attachments/assets/a15d4f61-9de3-4a47-b9c8-911fbbe333f6" />

### Etapa 5: Validação do Acesso

O Medusa encontrou a credencial correta! Agora, vamos usá-la para acessar o servidor FTP e confirmar o ganho de acesso.

```bash
ftp 192.168.56.103
```
* **Usuário:** `msfadmin`
* **Senha:** `msfadmin`

> <img width="307" height="114" alt="Image" src="https://github.com/user-attachments/assets/a1eec6df-bf7a-47c7-abe6-7dcbe1b560ad" />

**Sucesso! O acesso ao servidor FTP foi obtido.** O mesmo processo pode ser adaptado para atacar outros serviços identificados, como o HTTP.

---


## 🛡️ Recomendações de Mitigação

Identificar uma vulnerabilidade é apenas o primeiro passo. O mais importante é saber como corrigi-la. Para proteger a "Tabajara Club" contra ataques de força bruta, as seguintes medidas devem ser implementadas:

1.  **Política de Senhas Fortes:** Exigir senhas com alta complexidade (letras maiúsculas e minúsculas, números, símbolos) e um comprimento mínimo de 12 caracteres.
2.  **Bloqueio de Contas:** Implementar uma política que bloqueie temporariamente uma conta após um número limitado de tentativas de login falhas (ex: 5 tentativas em 10 minutos).
3.  **Autenticação Multifator (MFA):** Ativar uma segunda camada de verificação para todos os acessos a serviços críticos, especialmente os expostos à internet.
4.  **Monitoramento e Alertas:** Utilizar ferramentas como **Fail2Ban** para monitorar logs de autenticação em tempo real e banir automaticamente os IPs que demonstrarem comportamento de ataque.
5.  **Limitar Acessos Administrativos:** Garantir que contas com altos privilégios (como `root`) não possam ser acessadas diretamente por serviços como FTP ou SSH.
