# vps-ssh-setup

Registro do meu estudo de **acesso remoto seguro por SSH com autenticação por chave** a um servidor Linux (VPS), incluindo o passo a passo, os conceitos que aprendi e o diário dos erros que resolvi no caminho.

Faz parte do meu portfólio de transição para TI, na área de infraestrutura. O outro projeto do portfólio é o [LabFlow](https://github.com/danyllo153/labflow-whatsapp-automation), de automação com n8n.

## Objetivo

Sair do login por senha e chegar ao login **somente por chave**, em dois computadores (desktop e notebook), entendendo cada peça do processo: chaves, permissões, `authorized_keys` e o que muda quando o login por senha é desativado.

## Ambiente

| Item | Detalhe |
|---|---|
| Cliente | Windows, PowerShell, cliente OpenSSH nativo |
| Servidor | VPS Ubuntu (Linux), de estudo, cedido por um amigo que é o administrador |
| Acesso | Usuário comum, sem `root` |
| Tipo de chave | `ed25519` |

## O que foi feito

- [x] Conectar ao servidor por senha e aceitar a fingerprint do host
- [x] Gerar o par de chaves `ed25519` no desktop
- [x] Gerar uma segunda chave, separada, no notebook (uma chave por dispositivo)
- [x] Autorizar a chave do notebook no servidor sem `ssh-copy-id`, que não existe no Windows
- [ ] Validar o login sem senha nos dois computadores
- [ ] Administrador desativar o login por senha
- [ ] Validar os dois computadores depois da desativação

> Atualize os itens acima conforme concluir cada etapa.

## Conceitos que aprendi

| Conceito | Resumo |
|---|---|
| Chave pública x privada | A pública pode ser compartilhada e vai para o servidor. A privada nunca sai do meu computador. |
| `authorized_keys` | Arquivo do servidor com uma chave pública por linha. Quem tem a privada correspondente entra. |
| Fingerprint | Impressão digital para conferir uma chave ou um servidor. Não é a chave em si. |
| Uma chave por dispositivo | Se um computador for perdido, revogo só a linha dele no servidor. |
| Permissões `700` e `600` | O SSH ignora as chaves se `~/.ssh` ou `authorized_keys` estiverem abertos demais. |
| Ordem de mudança | Ativar o método novo, validar e só então desativar o antigo, para não ficar sem acesso. |

## Documentação

- [Autenticação por chave SSH](docs/ssh-chaves.md): gerar, enviar, autorizar e testar.
- [Permissões no Linux (chmod)](docs/permissoes-chmod.md): o que significam 700 e 600 e por que o SSH exige.
- [Checklist de hardening](docs/checklist-hardening.md): como desativar o login por senha sem se trancar para fora.
- [Diário de problemas](docs/diario-de-problemas.md): sintoma, causa e solução de cada erro que encontrei.

## O que não está neste repositório

Por segurança, este repositório **não contém**: IP ou nome do servidor, senhas, chaves (nem as públicas), fingerprints ou prints de terminal com esses dados. Nos exemplos, uso `SEU_IP`, `SEU_USUARIO` e `SEU_USER`.

## Próximos passos

- Configurar firewall com `ufw`
- Instalar `fail2ban` contra tentativas de senha em massa
- Subir um site simples com Nginx
- Fazer um backup automático com `cron`
- Documentar o deploy do LabFlow em um servidor como este
