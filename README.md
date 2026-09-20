# Check Point VPN no Ubuntu 26.04.1 LTS (Resolute Raccoon)

Guia para instalar o **Check Point Mobile Access Portal Agent (CShell)** e o **SSL Network Extender (SNX)** no Ubuntu **26.04.1 LTS (Resolute Raccoon)**.

## Alteração efetuada no `cshell_install.sh`

O instalador original determina o utilizador através de:

```sh
GetUserName()
{
    user_name=`who | head -n 1 | awk '{print $1}'`
    echo ${user_name}
}
```

No Ubuntu 26.04, nomeadamente em sessões gráficas modernas, `who` pode não devolver nenhum utilizador quando o instalador é executado com `sudo`.

Isso faz com que o instalador fique sem saber qual é o utilizador real e, consequentemente, pode falhar ao localizar o perfil do Firefox e a respetiva base de certificados.

A versão corrigida substitui essa linha por:

```sh
GetUserName()
{
    user_name="${SUDO_USER:-$(id -un)}"
    echo ${user_name}
}
```

Desta forma:

- quando o instalador é executado com `sudo`, é utilizado `SUDO_USER`;
- caso `SUDO_USER` não exista, é utilizado `id -un`.

A alteração foi feita sem adicionar ou remover linhas antes do payload interno do instalador, para não alterar o `ARCHIVE_OFFSET` do ficheiro autoextraível.

---

## Dependências

Ativar suporte para pacotes de 32 bits, caso ainda não esteja ativo:

```bash
dpkg --print-foreign-architectures
```

Se `i386` não aparecer:

```bash
sudo dpkg --add-architecture i386
sudo apt update
```

Instalar as dependências necessárias:

```bash
sudo apt update

sudo apt install   openjdk-11-jre   libnss3-tools   openssl   xterm   bzip2   tar   x11-xserver-utils   libc6:i386   libstdc++6:i386   libpam0g:i386
```

Ferramentas principais utilizadas pelos instaladores:

- `java` — execução do `CShell.jar`;
- `certutil` — gestão do certificado do CShell no perfil do browser;
- `openssl` — criação dos certificados locais usados pelo CShell;
- `xterm` — utilizado pelo instalador;
- `xhost` — acesso à sessão gráfica;
- `bunzip2` / `bzip2` — extração do payload interno dos instaladores;
- `tar` — extração dos ficheiros;
- bibliotecas `i386` — necessárias porque o binário SNX é de 32 bits.

---

## Instalar este CShell

Na pasta onde está o ficheiro, executar:

```bash
chmod +x cshell_install_fixed.sh
sudo bash ./cshell_install_fixed.sh
```

Durante a instalação poderá ser pedido que o Firefox seja fechado para permitir a instalação do certificado.

```bash
pkill firefox
```

No final, o CShell deverá ficar disponível em:

```text
/usr/bin/cshell/
```

NOTA: O serviço local utilizado pelo portal Check Point escuta normalmente em:

```text
127.0.0.1:14186
```

Para confirmar:

```bash
ss -ltnp | grep 14186
```

---

## Instalar o SNX

O instalador do SNX deve ser obtido através do portal Check Point utilizado pela UA.

Guardar o ficheiro:

```text
snx_install.sh
```

Depois:

```bash
chmod +x snx_install.sh
sudo bash ./snx_install.sh
```

O binário deverá ficar instalado em:

```text
/usr/bin/snx
```

Confirmar:

```bash
ls -l /usr/bin/snx
```

O SNX utilizado neste ambiente é um binário **i386/32-bit**, razão pela qual é necessário manter as bibliotecas `i386` instaladas.

---

## Ligação

Com o CShell em execução, abrir o portal Check Point no browser, autenticar e utilizar o botão **Connect**.

O portal lança o SNX automaticamente.

Quando a ligação estiver estabelecida deverá aparecer uma interface semelhante a:

```text
tunsnx
```

Confirmar com:

```bash
ip addr show tunsnx
```

Para desligar manualmente pelo terminal:

```bash
snx -d
```

... ou simplesmente desconectar no portal do site.
