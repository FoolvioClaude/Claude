# Recon map: ManyChat — automação de Instagram (web)

Escopo: só a automação de respostas no Instagram. (1) Alguém comenta num post
ou Reel → resposta pública no comentário + DM automática. (2) Alguém manda
uma palavra-chave no Direct → resposta automática. Mais o mínimo em volta:
conectar a conta, lista de automações, contatos e números.
Para: uso próprio, uma única conta — @ofulvioporto. Não é um SaaS para
terceiros (isso muda muito o tamanho do projeto e o que a Meta exige; veja
"Size").
Data: 2026-10-30

> Nota sobre as fontes: neste ambiente, `manychat.com` e
> `developers.facebook.com` estão bloqueados para leitura direta. Tudo abaixo
> vem de resumos de busca dessas páginas e de guias de terceiros. Nada foi
> tirado do app logado (não usamos a conta de ninguém). Os pontos que precisam
> ser conferidos estão marcados com **[conferir]**. `replica/screens/` está
> vazio: não há capturas ainda.

## Sources

| # | source | URL | notes |
| --- | --- | --- | --- |
| 1 | blog ManyChat (guia oficial de comment automation) | https://manychat.com/blog/the-ultimate-guide-for-instagram-comment-automation/ | 3º passo = resposta pública sob o comentário |
| 2 | blog ManyChat (automatizar comentários) | https://manychat.com/blog/how-to-automate-your-comments-with-manychat/ | fluxo do gatilho de comentário |
| 3 | blog ManyChat (always-on automations) | https://manychat.com/blog/always-on-automations-manychat-2/ | automações permanentes |
| 4 | guia de terceiros (Kanal) | https://getkanal.com/blog/manychat-instagram | lista de gatilhos, variações aleatórias de resposta, atraso de até 10 min, regra de 1 resposta privada |
| 5 | guia de terceiros (Coffee Contracts) | https://support.coffeecontracts.com/en/articles/9822109-how-to-set-up-a-comment-automation-on-instagram-with-manychat | passo a passo: escolher Reel → "Specific Keywords" → palavra; sem diferenciar maiúsculas |
| 6 | Meta — Private Replies | https://developers.facebook.com/docs/messenger-platform/instagram/features/private-replies | 1 mensagem por comentário, até 7 dias, Live só durante a live |
| 7 | Meta — Send Messages (Instagram Login) | https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/messaging-api | `graph.instagram.com/me/messages`, janela de 24 h, permissões |
| 8 | Meta — Webhooks | https://developers.secure.facebook.com/docs/instagram/platform/instagram-api/webhooks | campos `comments`, `live_comments`, `messages`, `messaging_postbacks` |
| 9 | Meta — Comment Moderation | https://developers.facebook.com/docs/instagram-platform/comment-moderation | responder publicamente a um comentário |
| 10 | Meta — User Profile API | https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/messaging-api/user-profile | `is_user_follow_business` (checar se segue); só após consentimento |
| 11 | limites de quick replies / ice breakers / templates | https://docs.gupshup.io/docs/instagram-features-and-limitations | 13 quick replies, 4 ice breakers, 10 cards x 3 botões |
| 12 | comunidade ManyChat (problemas reais) | https://community.manychat.com/general-q-a-43/keyword-automation-not-triggering-at-all-instagram-comments-dm-0-runs-even-on-a-fresh-flow-10781 | "ativa mas 0 execuções" — o maior incômodo dos usuários |
| 13 | comunidade ManyChat | https://community.manychat.com/general-q-a-43/instagram-story-reply-automation-shows-0-sends-despite-correct-setup-10592 | correção comum: reconectar permissões |
| 14 | preços (terceiros, 2026) | https://www.featurebase.app/blog/manychat-pricing | grátis caiu para 25 contatos em mar/2026; "todos os posts" é recurso pago |
| 15 | termos de uso ManyChat | https://manychat.com/legal/tos | cláusula anti-concorrência não confirmada **[conferir]**; irrelevante aqui porque não usamos conta ManyChat |

## Core loop

Seguidor comenta uma palavra (ex.: "LINK") num post → recebe resposta pública
e uma DM com o material → clica e vira contato. É por isso que as pessoas
pagam o ManyChat.

## Screens

| ID | screen | route / how to reach | purpose | key components | states seen |
| --- | --- | --- | --- | --- | --- |
| S01 | Login | `/login` | entrar no painel (só você) | campo e-mail, senha / magic link | vazio, erro |
| S02 | Conectar Instagram | `/settings/instagram` → "Conectar" | OAuth do Instagram Login, salvar token | botão conectar, cartão da conta (@, foto), status do token | não conectado, conectado, token expirado, conta não profissional |
| S03 | Automações (home) | `/automations` | listar e ligar/desligar | tabela/cartões, toggle ativo, contadores (execuções, DMs, cliques), botão "Nova" | vazio, cheio, pausada, com erro |
| S04 | Nova automação: escolher gatilho | modal em S03 | escolher tipo | cartões: "Comentário em post/Reel", "Palavra no Direct", (could) "Resposta a story" | — |
| S05 | Editor — gatilho de comentário | `/automations/:id` aba Gatilho | onde vale: post específico / qualquer post / próximo post | radio de escopo, botão escolher post | sem post escolhido, ok |
| S06 | Seletor de post/Reel | modal em S05 | escolher a publicação | grade de miniaturas, legenda, data, tipo (post/reel/carrossel) | carregando, vazio, erro da API |
| S07 | Regras de palavra-chave | S05 seção "Quando" | quais comentários disparam | radio "qualquer comentário" / "contém palavras", chips de palavras, chips de exclusão | sem palavras (erro), ok |
| S08 | Resposta pública | S05 seção "Responder no comentário" | responder sob o comentário | toggle, lista de variações (sorteadas), contador de caracteres | desligado, 1+ variações |
| S09 | Editor da DM (fluxo) | `/automations/:id` aba Mensagens | o que a DM envia | balão de mensagem de abertura, botões (postback / link), bloco "pedir para seguir", atraso, pré-visualização em celular | vazio, com botões, erro de validação |
| S10 | Editor — palavra no Direct | `/automations/:id` | gatilho de DM | modo de match (é exatamente / contém), chips de palavras, mesma área de mensagens de S09 | — |
| S11 | Resposta padrão | `/automations/default-reply` | DM que não casou com nenhuma palavra | toggle, mensagem, "só 1 vez por contato a cada X h" | desligado, ligado |
| S12 | Contatos | `/contacts` | quem já interagiu | tabela: @, nome, origem (automação), segue?, tags, última interação; busca; exportar CSV | vazio, cheio |
| S13 | Contato / conversa | `/contacts/:id` | histórico de um contato | linha do tempo (comentário, DM enviada, clique), tags, botão "abrir no Instagram" | — |
| S14 | Resultados da automação | `/automations/:id/stats` | números | cartões: comentários recebidos, respostas públicas, DMs enviadas, cliques no botão, CTR; tabela de execuções com status/erro | sem dados, com dados |
| S15 | Saúde / registro | `/settings/health` | diagnosticar "0 execuções" | último webhook recebido, eventos recentes, erros da API, botão "enviar teste" | ok, sem eventos, erros |

Fora da fatia (existem no ManyChat, não entram): Live Chat completo,
Messenger/WhatsApp/TikTok, gatilhos de anúncios, IA, sequências de broadcast,
biblioteca de templates, integrações.

## Flows

```
F01 Conectar a conta @ofulvioporto
    S01 -> S02 (OAuth Instagram) -> S03
    happy path clicks: 4
    edge: conta pessoal (precisa ser Profissional: Criador ou Empresa),
          permissão negada no OAuth, token expira em ~60 dias e precisa ser renovado

F02 Criar automação "comentou LINK no Reel → manda o link"
    S03 -> S04 -> S05 -> S06 (escolhe Reel) -> S07 (palavra LINK)
        -> S08 (2-3 variações de resposta pública) -> S09 (abertura + botão + link) -> ativar
    happy path clicks: ~10 (número a bater)
    edge: nenhuma palavra preenchida, duas automações ativas no mesmo post
          com a mesma palavra (qual vence?), post apagado depois

F03 Execução: seguidor comenta (sem UI, é o motor)
    webhook `comments` -> casa regra -> responde publicamente (variação sorteada)
        -> private reply com mensagem de abertura + botão
        -> seguidor toca no botão (webhook `messaging_postbacks`, abre janela de 24 h)
        -> envia a mensagem com o link -> registra execução
    edge: comentário do próprio @ofulvioporto (ignorar), mesma pessoa comenta 5x
          (mandar DM 1 vez só), comentário com mais de 7 dias, resposta a outro
          comentário (thread), Live (só durante a live), limite de taxa da API,
          webhook duplicado (idempotência)

F04 Pedir para seguir antes de mandar o link (follow gate)
    toque no botão -> consulta `is_user_follow_business`
        -> segue: manda o link
        -> não segue: "Me segue e toca em 'Já segui'" -> toque -> checa de novo
    happy path clicks (seguidor): 1-3
    edge: só dá para checar depois que a pessoa interage na DM (consentimento);
          pessoa diz que seguiu e não seguiu (limitar tentativas)

F05 Palavra no Direct
    S03 -> S04 -> S10 -> S09 -> ativar ; em execução: webhook `messages` -> casa -> responde
    edge: maiúsculas/acentos ("Link", "lînk"), palavra no meio da frase,
          você mesmo respondendo pelo celular (eco de mensagem não pode disparar)

F06 Ver se funcionou
    S03 -> S14 (ou S15 se mostrar 0 execuções)
    happy path clicks: 1

F07 Pausar uma automação
    S03 toggle
    happy path clicks: 1
```

## Components

| component | variants | states | used on |
| --- | --- | --- | --- |
| Button | primary, secondary, ghost, danger | default, hover, focus, disabled, loading | todas |
| Toggle (ativo/pausado) | sm, md | on, off, disabled, loading | S03, S08, S11 |
| Card de automação | comentário, DM | ativa, pausada, com erro | S03 |
| Chip input de palavras | incluir, excluir | vazio, com chips, erro | S07, S10 |
| Radio group | — | — | S05, S07, S10 |
| Grade de mídia | post, reel, carrossel | carregando, vazio, selecionado | S06 |
| Balão de mensagem (editor) | texto, texto+botões, imagem | editando, erro de limite | S09, S10, S11 |
| Botão de mensagem (editor) | postback, link | — | S09 |
| Lista de variações | — | 1..n itens | S08 |
| Pré-visualização de celular | Instagram DM, comentário | — | S08, S09 |
| Modal | — | aberto, fechado | S04, S06 |
| Tabela | contatos, execuções, eventos | vazio, carregando, cheio, paginada | S12, S14, S15 |
| Cartão de métrica | número, porcentagem | sem dados | S14 |
| Toast | sucesso, erro | — | todas |
| Banner de status da conta | ok, token expirando, desconectada | — | todas (topo) |
| Navegação lateral | — | item ativo | todas |

## Inferred data model

```
InstagramAccount  id, ig_user_id, username, profile_pic_url, account_type,
                  access_token (cifrado), token_expires_at, connected_at, status
                  evidence: S02, fonte 7 (token Instagram Login)
                  confidence: high

Automation        id, account_id, name, type (comment | dm_keyword | default_reply),
                  status (active | paused | error), created_at, updated_at
                  evidence: S03, S04, fontes 1-5
                  confidence: high

CommentTrigger    automation_id, scope (specific | any | next_post),
                  media_ids [] (quando specific), match (any | keywords),
                  keywords [], exclude_keywords [], include_live (bool)
                  evidence: S05-S07, fontes 4-5
                  confidence: high (next_post: medium)

DmKeywordTrigger  automation_id, match (exact | contains), keywords []
                  evidence: S10, fonte 4
                  confidence: medium

PublicReply       automation_id, enabled, variants [] (texto)
                  evidence: S08, fontes 1, 4
                  confidence: high

FlowStep          id, automation_id, order, kind (message | follow_gate | delay),
                  text, buttons [ {label, kind: postback|url, url, next_step_id} ],
                  delay_seconds
                  evidence: S09, fontes 4, 10, 11
                  confidence: medium (o ManyChat tem um editor de fluxo bem mais
                  geral; aqui é guess deliberadamente reduzido)

Contact           id, account_id, igsid, username, name, follows_you (bool, last_checked_at),
                  first_source_automation_id, tags [], first_seen_at, last_interaction_at,
                  window_open_until
                  evidence: S12-S13, fontes 7, 10
                  confidence: high

Run               id, automation_id, contact_id, comment_id, media_id,
                  status (matched | public_replied | dm_sent | clicked | completed | failed),
                  error, created_at
                  evidence: S14, fonte 12 (contagem de "runs")
                  confidence: high

WebhookEvent      id, field (comments | messages | messaging_postbacks | ...),
                  external_id (para deduplicar), payload (json), received_at, processed_at, error
                  evidence: S15, fonte 8 (Meta não guarda payloads)
                  confidence: high (não é visível no ManyChat; é necessário para o clone)
```

Relationships: InstagramAccount 1-n Automation; Automation 1-1 CommentTrigger |
DmKeywordTrigger; Automation 1-1 PublicReply; Automation 1-n FlowStep;
Automation 1-n Run; Contact 1-n Run; WebhookEvent 1-0..1 Run.

## Regras da Meta que definem o produto

São essas regras, não o ManyChat, que limitam o que o clone pode fazer:

- A conta precisa ser **Profissional** (Criador ou Empresa).
- Permissões: `instagram_business_basic`, `instagram_business_manage_messages`,
  `instagram_business_manage_comments`.
- **Private reply**: 1 mensagem por comentário, até 7 dias depois do comentário.
  Na Live, só durante a transmissão.
- A conversa livre só começa quando a pessoa responde ou toca num botão. A partir daí
  vale uma **janela de 24 h**. Por isso a DM de abertura tem um botão.
- Só dá para ver se a pessoa te segue (`is_user_follow_business`) depois que ela
  interage na DM.
- Limites: 13 quick replies, 4 ice breakers, 10 cards x 3 botões por template
  **[conferir na doc da Meta]**. Limites de taxa por hora **[conferir]**.
- Para uma só conta, sua, o app da Meta pode ficar em modo de desenvolvimento
  com você como testador, **sem App Review** **[conferir]**. Atender contas de
  terceiros exigiria Advanced Access e revisão.

## Feature matrix

See `features.csv`. Must: 17, should: 10, could: 7, skip: 9.

## Out of scope (cannot or should not be cloned)

- Messenger, WhatsApp, TikTok, Telegram, SMS e e-mail: fora da fatia.
- Gatilhos de anúncios (Click-to-DM ads): exigem Ads API e outra revisão.
- IA do ManyChat (respostas com IA): fora da fatia. Pode entrar depois como recurso próprio.
- Biblioteca de templates e marketplace de integrações: são o ecossistema deles.
- Multi-conta, agências, cobrança e planos: é uso próprio.
- A marca, os textos e os ícones do ManyChat: o clone tem identidade própria.
- Recursos "beta" não confirmados (ex.: "Follow to DM" automático ao seguir):
  não há fonte pública confiável **[conferir]**.

## Size

Telas: 15. Fluxos: 7. Entidades: 9.

Partes difíceis:
1. **Configurar o app da Meta e o token**: OAuth do Instagram Login, token de
   ~60 dias com renovação, webhooks verificados e em HTTPS público.
2. **Confiabilidade do webhook**: deduplicar eventos, não responder 2x à mesma
   pessoa, ignorar seus próprios comentários e ecos, respeitar limites de taxa
   com fila e novas tentativas. O "0 execuções" é a reclamação nº 1 no ManyChat.
3. **Regras de mensagens da Meta**: 1 private reply, 7 dias, janela de 24 h,
   consentimento para o follow gate.

Tamanho: **S a M**. Um MVP só para @ofulvioporto (F01–F04 + S14 simples) cabe
em 1–2 fins de semana de trabalho. Com contatos, resposta padrão, painel de
saúde e o editor completo, são algumas semanas. Não prometemos um clone
perfeito do editor de fluxo do ManyChat, e não precisa ser.
