## **Remover** um computador Ubuntu do domínio Active Directory

Link para ajudar o entendimento: https://www.server-world.info/en/note?os=Ubuntu_24.04&p=realmd

Se você quiser **remover** um computador Ubuntu do domínio Active Directory, o processo é bastante simples e envolve a remoção do sistema da configuração de domínio. Abaixo está o procedimento passo a passo para retirar o Ubuntu do AD.

### 1. Verificar o domínio em que o sistema está ingressado
Antes de realizar a remoção, é bom verificar em qual domínio o Ubuntu está atualmente ingressado. Para isso, use o comando:

```bash
realm list
```

Esse comando mostrará os domínios aos quais o sistema pertence, juntamente com as configurações relacionadas ao domínio.

### 2. Remover o Ubuntu do domínio Active Directory
Para remover o Ubuntu do domínio Active Directory, utilize o comando **`realm leave`**, que remove o computador do domínio e encerra a conexão com ele. Substitua `example.com` pelo nome do domínio do qual você deseja remover o sistema:

```bash
sudo realm leave example.com
```

Este comando remove todas as configurações de domínio associadas ao sistema.

### 3. Verificar se o sistema foi removido
Após a execução do comando, você pode verificar novamente se o sistema foi removido do domínio com o comando:

```bash
realm list
```

Se o processo foi bem-sucedido, o domínio não será mais listado.

### 4. Remover ou limpar as configurações associadas (opcional)
Embora o comando **`realm leave`** remova o computador do domínio, alguns arquivos de configuração relacionados ao **SSSD** e ao **Kerberos** podem permanecer no sistema. Se você quiser remover ou limpar essas configurações, siga as etapas abaixo:

#### 4.1. Limpar as configurações do SSSD
Se você não pretende usar o **SSSD** para outras autenticações, pode desativar e remover suas configurações.

- Para parar o serviço SSSD:

  ```bash
  sudo systemctl stop sssd
  ```

- Desabilitar o serviço para que ele não inicie mais automaticamente:

  ```bash
  sudo systemctl disable sssd
  ```

- Se quiser remover o SSSD completamente:

  ```bash
  sudo apt remove --purge sssd
  ```

#### 4.2. Limpar as configurações de Kerberos
Se o sistema foi configurado para autenticar usando **Kerberos**, você pode querer remover ou ajustar as configurações do Kerberos.

- Se você deseja manter o Kerberos mas ajustar as configurações, edite o arquivo `/etc/krb5.conf` e remova as entradas associadas ao domínio que foi retirado.

- Se quiser remover o Kerberos completamente, use o comando:

  ```bash
  sudo apt remove --purge krb5-user
  ```

### 5. Remover as entradas do DNS (opcional)
Se você tem controle sobre o servidor DNS ou deseja remover as entradas de DNS associadas ao Ubuntu que foi ingressado no AD, poderá precisar remover manualmente o registro do nome da máquina no servidor DNS.

### 6. Limpar o NetBIOS (opcional)
Se você configurou o **Winbind** ou ajustou o **NetBIOS Name**, como `FD3S01`, você pode também querer remover ou ajustar essas configurações editando o arquivo `/etc/samba/smb.conf` e limpando as entradas relacionadas ao AD.

Após esses procedimentos, o Ubuntu estará completamente removido do Active Directory e de quaisquer serviços associados.

### Resumo dos Passos
1. Use o comando `realm leave` para remover o Ubuntu do domínio.
2. Verifique se o sistema foi removido com `realm list`.
3. Opcionalmente, limpe as configurações relacionadas ao SSSD, Kerberos e DNS.
