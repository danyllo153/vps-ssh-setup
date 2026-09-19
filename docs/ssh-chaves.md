# Autenticação por chave SSH (Windows para Linux)

Guia do que fiz para entrar num servidor Linux sem digitar senha, usando o PowerShell do Windows.

## Conceito

| Item | Onde fica | Pode compartilhar? |
|---|---|---|
| Chave **privada** (`id_ed25519`) | Só no meu computador | **Nunca** |
| Chave **pública** (`id_ed25519.pub`) | Vai para o servidor | Sim |
| **Fingerprint** | Aparece ao gerar a chave ou conectar | Só serve para conferência |

O servidor guarda a chave pública. Quando me conecto, o meu computador prova que tem a privada correspondente, sem nunca enviá-la.

## 1. Gerar o par de chaves

No PowerShell do meu computador (o prompt começa com `PS C:\Users\SEU_USER>`):

```powershell
ssh-keygen -t ed25519 -C "notebook-SEU_USER"
```

Ele faz três perguntas:

1. **Onde salvar:** Enter para aceitar `C:\Users\SEU_USER\.ssh\id_ed25519`.
2. **Passphrase:** Enter para deixar vazio, ou definir uma senha para a chave (mais seguro).
3. **Repetir a passphrase.**

O parâmetro `-C` é só um comentário. Uso o nome do dispositivo (`notebook-...`, `desktop-...`) para saber depois qual linha do servidor pertence a qual máquina.

Se aparecer `already exists. Overwrite (y/n)?`, responder **n**: já existe uma chave e sobrescrever a apagaria.

## 2. Ver a chave pública

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

A saída é uma linha só:

```
ssh-ed25519 AAAA... notebook-SEU_USER
```

`USERPROFILE` é o **nome** de uma variável do Windows e não deve ser trocado pelo meu usuário. O PowerShell converte sozinho para `C:\Users\SEU_USER`.

## 3. Autorizar a chave no servidor

**Opção A: enviar a linha ao administrador**, que a adiciona ao `authorized_keys` do meu usuário.

**Opção B: fazer eu mesmo**, quando já tenho acesso por senha. O Windows não tem `ssh-copy-id`, então uso um pipe:

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh SEU_USUARIO@SEU_IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && tr -d '\r' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

O que cada parte faz:

| Parte | Função |
|---|---|
| `mkdir -p ~/.ssh` | Cria a pasta se ela não existir |
| `chmod 700 ~/.ssh` | Só o dono acessa a pasta (ver [permissões](permissoes-chmod.md)) |
| `tr -d '\r'` | Remove o retorno de carro do Windows, que pode invalidar a chave |
| `>>` | **Acrescenta** ao arquivo. Com um só `>` eu apagaria as chaves que já estavam lá |
| `chmod 600 ...authorized_keys` | Só o dono lê e escreve o arquivo |

O comando pede a senha uma vez e não imprime nada quando dá certo.

## 4. Testar

```powershell
ssh SEU_USUARIO@SEU_IP
```

- **Entrou sem pedir a senha do servidor:** a chave funciona.
- **Pediu senha:** a chave não foi reconhecida. Rodar `ssh -v SEU_USUARIO@SEU_IP` e ler o final da saída para ver quais chaves o cliente ofereceu.

Para conferir no servidor, depois de entrar:

```bash
cat ~/.ssh/authorized_keys
```

Deve haver uma linha por dispositivo, cada uma terminando no comentário que defini no `-C`.

## 5. Uma chave por dispositivo

Cada computador gera a sua própria chave, e todas as públicas ficam no `authorized_keys`. Vantagem: se um computador for perdido ou roubado, removo só a linha dele.

```bash
sed -i '/notebook-SEU_USER/d' ~/.ssh/authorized_keys
```

Esse comando apaga a linha que contém o comentário indicado. Vale conferir com `cat` antes e depois.

## 6. Atalho de conexão (opcional)

Arquivo `C:\Users\SEU_USER\.ssh\config`, sem extensão:

```
Host meu-vps
    HostName SEU_IP
    User SEU_USUARIO
    IdentityFile ~/.ssh/id_ed25519
```

Depois, `ssh meu-vps` basta. O arquivo `config` existe por computador.

## 7. Proteger a chave com passphrase depois

Se gerei a chave sem passphrase, dá para adicionar uma sem criar outra:

```powershell
ssh-keygen -p -f $env:USERPROFILE\.ssh\id_ed25519
```

Sem passphrase, quem copiar o arquivo `id_ed25519` do meu computador consegue entrar no servidor.

## Erros comuns

- **Rodar `ssh-keygen` dentro do servidor.** A chave precisa ser gerada na máquina de onde vou me conectar. Conferir o prompt antes: `PS C:\...` é o meu PC, `usuario@servidor:~$` é o servidor.
- **Enviar a chave privada.** Só o arquivo `.pub` é enviado.
- **Chave quebrada em duas linhas** ao copiar e colar. A linha precisa estar inteira.
- **Permissões abertas** em `~/.ssh` ou `authorized_keys`. Ver [permissões](permissoes-chmod.md).
