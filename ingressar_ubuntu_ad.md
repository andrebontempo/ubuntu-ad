## Ingressar um computador com **Ubuntu Linux** no **Active Directory**

Link para ajudar o entendimento: https://www.server-world.info/en/note?os=Ubuntu_24.04&p=realmd

Ingressar um computador com **Ubuntu Linux** no **Active Directory** (AD) é um processo que permite que o sistema Linux participe da mesma rede gerenciada centralmente pelo AD, permitindo que os usuários façam login com suas credenciais do AD. O Active Directory é amplamente utilizado em ambientes empresariais, e a integração de sistemas Linux pode ser útil para garantir uma administração centralizada.

A seguir está um guia passo a passo para ingressar um Ubuntu Linux no Active Directory, utilizando ferramentas como o **Realmd**, **SSSD** (System Security Services Daemon) e o **Winbind**.

### Requisitos
- Um servidor Active Directory funcional, geralmente rodando o Windows Server, e configurado corretamente.
- Credenciais de administrador de domínio ou de uma conta que tenha permissão para adicionar computadores ao domínio.
- Um servidor DNS configurado corretamente, permitindo que o Ubuntu resolva o domínio AD.

### Passos

#### 1. Atualizar o sistema
Primeiro, você precisa garantir que o seu sistema esteja atualizado. Execute os seguintes comandos para atualizar os pacotes do Ubuntu:

```bash
sudo apt update
sudo apt upgrade
```

#### 2. Instalar os pacotes necessários
Para ingressar no domínio Active Directory, instale o **Realmd**, o **SSSD**, o **Kerberos**, o **Winbind**, o **Samba** e algumas dependências adicionais que facilitarão o processo.

```bash
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin krb5-user chrony
```

Durante a instalação do pacote `krb5-user`, será solicitado o **nome do realm Kerberos**, que é geralmente o nome do domínio AD em **letras maiúsculas**. Por exemplo, se seu domínio for `example.com`, o realm será `EXAMPLE.COM`.

#### 3. Configurar o DNS para o Active Directory
O Ubuntu precisa ser capaz de resolver o nome do domínio Active Directory corretamente. Você pode configurar o DNS editando o arquivo `/etc/resolv.conf` ou configurando o DNS de forma permanente no gerenciador de rede. Edite o arquivo `/etc/netplan/00-installer-config.yaml` (ou equivalente) e adicione o servidor DNS do domínio.

```yaml
nameservers:
  addresses:
    - 192.168.1.1   # Substitua com o endereço do servidor DNS AD
  search:
    - example.com   # O nome do domínio AD
```

Depois, aplique as configurações:

```bash
sudo netplan apply
```

#### 4. Verificar a descoberta do domínio
Agora que o DNS está configurado corretamente, você pode verificar se o domínio é detectável usando o **Realmd**.

```bash
realm discover example.com
```

Esse comando deve retornar informações sobre o domínio AD, como o nome do domínio, o realm Kerberos e outros detalhes. Se a descoberta for bem-sucedida, você estará pronto para ingressar no domínio.

#### 5. Ingressar no domínio Active Directory
Para ingressar o Ubuntu no domínio, utilize o comando abaixo. Substitua "example.com" pelo nome do seu domínio e "Administrator" pela conta com permissões de administrador no AD:

```bash
sudo realm join --user=Administrator example.com
```

Será solicitado que você insira a senha do usuário administrador. Se o processo for bem-sucedido, o computador será adicionado ao domínio.

#### 6. Verificar se o sistema está no domínio
Para verificar se o sistema foi adicionado corretamente ao domínio, utilize o seguinte comando:

```bash
realm list
```

Este comando exibirá o domínio no qual o sistema foi ingressado e as permissões associadas.

#### 7. Configurar o SSSD para autenticação
Agora que o Ubuntu está no domínio, é necessário configurar o **SSSD** para gerenciar a autenticação de usuários. O arquivo de configuração do SSSD pode ser encontrado em `/etc/sssd/sssd.conf`.

Crie ou edite o arquivo `/etc/sssd/sssd.conf` com o seguinte conteúdo, substituindo `EXAMPLE.COM` pelo seu **realm Kerberos** e `example.com` pelo nome do seu domínio:

```ini
[sssd]
domains = example.com
config_file_version = 2
services = nss, pam

[domain/example.com]
ad_domain = example.com
krb5_realm = EXAMPLE.COM
realmd_tags = manages-system joined-with-samba
cache_credentials = True
id_provider = ad
fallback_homedir = /home/%u
default_shell = /bin/bash
ldap_id_mapping = True
use_fully_qualified_names = False
access_provider = ad
```

Depois de salvar as alterações, defina as permissões corretas para o arquivo:

```bash
sudo chmod 600 /etc/sssd/sssd.conf
```

Reinicie o serviço SSSD:

```bash
sudo systemctl restart sssd
```

#### 8. Configurar PAM para autenticação
Você precisa garantir que o **PAM (Pluggable Authentication Modules)** esteja configurado para autenticar os usuários do AD. Edite o arquivo `/etc/pam.d/common-session` e adicione a seguinte linha:

```bash
session required pam_mkhomedir.so skel=/etc/skel/ umask=0077
```

Isso criará automaticamente o diretório home do usuário quando ele fizer login pela primeira vez.

#### 9. Testar o login de um usuário do Active Directory
Agora, você pode testar o login de um usuário do Active Directory. Faça o logout do sistema ou abra um terminal e tente fazer login com o nome de usuário do AD.

No terminal, você pode testar o login de um usuário do AD com o seguinte comando:

```bash
id usuario_do_dominio@example.com
```

Se tudo estiver funcionando corretamente, você verá as informações do usuário do AD.

#### 10. Configurar sincronização de tempo
Para garantir que o sistema Linux e o Active Directory tenham o tempo sincronizado (o que é crítico para o funcionamento correto do Kerberos), configure o **Chrony** ou **NTP**. Como você já instalou o `chrony` no início, edite o arquivo de configuração `/etc/chrony/chrony.conf` e adicione o servidor NTP do Active Directory:

```bash
server dc.example.com iburst
```

Substitua `dc.example.com` pelo endereço do seu controlador de domínio.

Reinicie o serviço Chrony:

```bash
sudo systemctl restart chrony
```

#### 11. Ajustes no ambiente Desktop
Para desabilitar a lista de usuários na tela de login:
Ajustar a linha: disable-user-list=true

```bash
nano /etc/gdm3/greeter.dconf-defaults
```

Este autostart ainda não sei como configurar e o quê!!!
```bash
nano /etc/xdg/autostart
```
### 12. Impressoras

Instalei o cliente do samba
```bash
apt install smbclient
```
Caminho: 
smb://printserver01.sede.embrapa.br/GGTI_Color



#### Conclusão
Se todos os passos foram seguidos corretamente, seu Ubuntu agora está ingressado no Active Directory, e os usuários do AD podem se autenticar no sistema. Isso garante que você possa gerenciar permissões e logins de maneira centralizada, unificando a administração entre sistemas Windows e Linux.
