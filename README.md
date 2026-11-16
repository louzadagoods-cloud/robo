# Servidor de WhatsApp — NexPlay

Este pacote é o servidor que conecta o seu número de WhatsApp ao painel NexPlay.
Ele gera o QR Code, recebe as mensagens dos clientes, conversa com o robô de IA do painel
e envia a resposta de volta.

> **Importante:** sem este servidor no ar, o painel não consegue gerar QR Code.
> O QR Code nasce aqui, não no site.

## O que tem no pacote

| Arquivo | Para que serve |
|---|---|
| `server.js` | o servidor (QR Code, status, envio e recebimento) |
| `package.json` | o que o servidor precisa para funcionar |
| `Dockerfile` | receita de instalação usada pelo Render |
| `render.yaml` | configura o Render sozinho (plano, disco e endereço do robô) |
| `.dockerignore` / `.gitignore` | arquivos que não devem subir |

Todos os arquivos ficam na **raiz** do repositório — não crie subpasta.

---

## Passo 1 — Subir no GitHub

1. Acesse `github.com` e clique em **New repository**.
2. Dê um nome, por exemplo `nexplay-whatsapp`, marque **Private** e clique em **Create repository**.
3. Clique em **Add file → Upload files**.
4. Arraste **todos** os arquivos deste pacote.
5. Confirme que `package.json`, `Dockerfile` e `render.yaml` aparecem na tela principal do repositório.
6. Clique em **Commit changes**.

> Os arquivos que começam com ponto (`.dockerignore` e `.gitignore`) podem ser criados pelo botão
> **Add file → Create new file**, digitando o nome exato.

---

## Passo 2 — Subir no Render

1. Acesse `render.com` e entre com a conta do GitHub.
2. Clique em **New → Blueprint**.
3. Escolha o repositório criado.
4. Confira dois pontos:
   - **Plano: Starter** (o plano Free dorme e não tem disco, então a sessão do WhatsApp se perde).
   - **Disco montado em `/var/data`**.
5. Clique em **Apply** e espere o deploy ficar **Live** (3 a 8 minutos).
6. Copie o endereço mostrado no topo, algo como `https://nexplay-whatsapp.onrender.com`.

Se o seu cartão ainda não estiver habilitado no Render, o blueprint pode recusar o plano Starter.
Nesse caso, troque para **Free** apenas para testar — mas saiba que o serviço dorme e a conexão cai.

---

## Passo 3 — Conferir se está tudo certo

Abra no navegador:

```
https://SEU-ENDERECO.onrender.com/api/diagnostico
```

O ideal é ver:

- `"ok": true`
- `"versao": "1.8.4"`
- `"pasta_persistente": true`
- `"permissoes_banco": "completa"`

Se aparecer algo na lista `problemas`, leia a mensagem — quase sempre é o disco fora de `/var/data`.

---

## Passo 4 — Conectar no painel

1. Abra a aba **WhatsApp → Conexão** no painel.
2. Cole o endereço do servidor e clique em **Salvar**.
3. Clique em **Testar servidor** (deve responder o tempo em milissegundos).
4. Clique em **Gerar QR Code**.
5. No celular: *WhatsApp → Configurações → Dispositivos conectados → Conectar dispositivo*.
6. Quando conectar, a tela mostra o número e o nome do perfil.

---

## Problemas mais comuns

| O que aparece | O que significa | O que fazer |
|---|---|---|
| **HTTP 503** no teste | O serviço existe, mas não está rodando: suspenso, dormindo ou deploy falhado | Abra o Render, veja se está **Live** e leia os **Logs** |
| **HTTP 502 / 504** | O serviço está reiniciando ou travou | Aguarde um minuto e teste de novo; se repetir, veja os Logs |
| **HTTP 404** | O endereço está certo, mas a rota não existe | Salve só o endereço do serviço, sem `/api` no final |
| Página do site no diagnóstico | O endereço salvo é do site, não do servidor | Copie o endereço que aparece no topo do serviço no Render |
| `"pasta_persistente": false` | O disco não está montado | Monte o disco em `/var/data` |
| QR aparece, mas o WhatsApp diz que não é do WhatsApp | O código veio de uma resposta inválida | Clique em **Reiniciar sessão** e confira o diagnóstico |

---

## Atualizar um servidor que já está no ar

1. Baixe o pacote de novo pelo botão **Baixar servidor (.zip)** na aba WhatsApp.
2. No GitHub, abra o repositório do servidor → **Add file → Upload files**.
3. Envie `server.js` (os outros arquivos raramente mudam) e confirme em **Commit changes**.
4. O Render faz o deploy sozinho, aguarde ficar **Live**.
5. Confirme em `/api/diagnostico` que a versão é `1.8.4`.

---

## Rotas disponíveis

| Rota | Método | Função |
|---|---|---|
| `/api/diagnostico` | GET | verifica se está tudo certo |
| `/api/health` | GET | resposta rápida para o Render |
| `/api/status` | GET | conexão, número, nome e QR |
| `/api/conectar` | POST | inicia a conexão e devolve o QR Code |
| `/api/qr` | GET | devolve o QR Code atual |
| `/api/reiniciar` | POST | apaga a sessão e gera um QR Code novo |
| `/api/desconectar` | POST | desconecta e apaga a sessão |
| `/api/enviar` | POST | envia mensagem (`numero` e `mensagem`) |
| `/api/agente` | POST | repassa uma consulta ao robô do painel |
| `/api/guardiao` | GET | situação do relógio dos testes |
| `/api/guardiao` | POST | força uma verificação dos testes agora |

---

## Relógio dos testes gratuitos (a cada 5 minutos)

O painel do provedor trabalha com **data inteira**, então um teste de 6 horas ficaria liberado lá
até o fim do dia. Este servidor chama a rotina de corte a cada 5 minutos, 24 horas por dia, mesmo
sem ninguém usando o site.

Conferir: abra `https://SEU-ENDERECO.onrender.com/api/guardiao`.
Você deve ver `"ligado": true`, `"intervalo_minutos": 5` e uma `"ultima_execucao"` recente.

---

## Avisos importantes

- Depois do primeiro QR Code lido, a sessão fica salva no disco e o servidor reconecta sozinho.
- Não use o plano Free em produção: ele dorme e a conexão do WhatsApp cai junto.
- Mantenha o endereço do serviço em sigilo — é o controle do seu WhatsApp.
- A IA só responde se a chave do Google Gemini estiver cadastrada no Backend. Sem ela, as mensagens
  chegam no painel e ficam aguardando você responder manualmente.