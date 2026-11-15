# agente-claude-codex

[![Agente Claude â†’ Codex: migre ou fique agnÃ³stico](guia/assets/banner.jpg)](https://inematds.github.io/agente-claude-codex/guia/)

Kit pra migrar um setup Claude Code pro Codex **ou** deixar o trabalho independente de modelo: separar o "cÃ©rebro" (contexto, decisÃµes, tarefas, handoffs, skills) do executor (Claude, Codex, Gemini, local).

Baseado na newsletter "Como migrar do Claude para o Codex â€” ou ficar independente de modelo" e na prompt library "Model-agnostic workspaces" (07 SEP 2026). Leia `PLANO.md` pra entender a anÃ¡lise e o piloto recomendado.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/agente-claude-codex/guia/**

## NÃ£o tem interface grÃ¡fica

Ã‰ um kit de linha de comando (bash) mais arquivos Markdown. TrÃªs formas de usar:

1. **Scripts** (`scripts/`): auditar, adaptar, instalar nÃºcleo, portar skill, provar. Ã‰ o caminho principal, descrito abaixo.
2. **Prompts** (`prompts/`): os mega-prompts A e B em texto copiÃ¡vel, pra colar num agente (Claude Code, Codex, outro) e fazer a migraÃ§Ã£o conversando. Comece sempre com `MODE: audit`.
3. **Template** (`template/`): o nÃºcleo portÃ¡til pra copiar em qualquer projeto, mesmo sem rodar os scripts.

## PrÃ©-requisitos

| Ferramenta | Pra quÃª | Como conferir |
|---|---|---|
| Claude Code | fonte da migraÃ§Ã£o (`~/.claude/skills`, `CLAUDE.md`) | `claude --version` |
| Codex CLI | destino (`~/.agents/skills`, `AGENTS.md`) | `codex --version` e `codex doctor` |
| polyskill | portar skill com fonte Ãºnica pros dois runtimes | `npm i -g polyskill` |
| bash, git, python3 | os scripts usam sÃ³ isso | jÃ¡ vem no Linux/macOS |

Sem Claude ou Codex instalado, o passo correspondente Ã© marcado como **nÃ£o rodado**, nÃ£o falha.

## Passo a passo

### 0. Clonar e diagnosticar o ambiente

```bash
git clone https://github.com/inematds/agente-claude-codex
cd agente-claude-codex
scripts/doctor.sh
```

Responde "meu ambiente estÃ¡ pronto?" antes de qualquer coisa. Confere git, python3, node, Claude Code (skills, CLAUDE.md, hooks), Codex CLI (skills, config, sandbox, MCP), polyskill e os arquivos do prÃ³prio kit. Cada item sai como `[ok]`, `[aviso]` (funciona, com limitaÃ§Ã£o) ou `[FALTA]` (essencial, com o comando pra resolver). Somente leitura; sai com cÃ³digo 1 se faltar algo essencial.

### 1. Auditar o que existe (somente leitura)

```bash
scripts/audit.sh
```

Produz `relatorios/auditoria-<data>.md` com: versÃµes, quantidade de skills / comandos / subagentes / hooks / MCP em cada runtime, e a matriz de cada skill que existe sÃ³ no Claude classificada como **reutilizÃ¡vel** (Markdown puro), **adaptador** (depende de MCP ou plugin do Claude), **nativo** (depende de hook) ou **nÃ£o resolvido**. Nada em `~/.claude` ou `~/.codex` Ã© alterado.

### 2. Separar as instruÃ§Ãµes portÃ¡teis do resÃ­duo Claude

```bash
scripts/adapt-instructions.sh ~/projetos/meu-projeto
```

LÃª o `CLAUDE.md` do projeto e grava dois arquivos ao lado, sem sobrescrever nada:

- `AGENTS.proposto.md`: regras portÃ¡teis, com ordem de leitura no topo (Codex, Gemini e OpenCode leem `AGENTS.md`).
- `CLAUDE.proposto.md`: `@AGENTS.md` mais sÃ³ o que Ã© especÃ­fico do Claude Code (plugins, hooks, AskUserQuestion).

Revise, renomeie pra `AGENTS.md` e `CLAUDE.md`. AtenÃ§Ã£o: menÃ§Ãµes a `CLAUDE.md` de outros projetos tambÃ©m sÃ£o renomeadas; confira.

### 3. Instalar o nÃºcleo portÃ¡til no projeto

```bash
scripts/init-core.sh ~/projetos/meu-projeto
```

Copia `template/` sem sobrescrever o que jÃ¡ existe e lista o que criou e o que manteve. Depois preencha:

| Arquivo | O que vai | Quem atualiza |
|---|---|---|
| `AGENTS.md` | regras estÃ¡veis + ordem de leitura | humano |
| `context/overview.md` | o que Ã©, fatos verificados (com fonte e data), preferÃªncias, hipÃ³teses | agente, ao promover fato |
| `context/current-state.md` | o que funciona e o que estÃ¡ pendente | agente, fim de sessÃ£o |
| `context/sources.md` | de onde vem cada informaÃ§Ã£o e regra de refresh | humano |
| `context/decisions/` | uma decisÃ£o aceita por arquivo | humano |
| `tasks/current.md` | objetivo, dono, critÃ©rio de pronto, prÃ³xima aÃ§Ã£o | humano define, agente marca |
| `handoffs/latest.md` | continuaÃ§Ã£o pra prÃ³xima sessÃ£o (cÃ³pia do Ãºltimo snapshot) | agente, fim de sessÃ£o |
| `handoffs/history/` | um snapshot por handoff, `AAAA-MM-DDTHHMMSSZ.md`, nunca sobrescrito | agente, fim de sessÃ£o |

Esses nomes sÃ£o convenÃ§Ã£o: nenhum runtime os carrega sozinho. O `AGENTS.md` do template jÃ¡ diz ao agente pra ler nessa ordem.

### 4. Portar uma skill com fonte Ãºnica (quatro destinos)

```bash
scripts/sync-skills.sh targets                        # claude, codex, dsh (dsh-sandbox), v3 (openpcbotv3): existem?
scripts/sync-skills.sh import session-handoff         # ~/.claude/skills/<skill> â†’ skills/<skill>/ (formato polyskill)
scripts/sync-skills.sh build                          # gera skills/*/dist/claude e dist/codex
scripts/sync-skills.sh install session-handoff --all  # PRÃ‰VIA: CREATE / UNCHANGED / CONFLICT / SYMLINK / SKIP por destino
scripts/sync-skills.sh install session-handoff --all --apply            # grava (sÃ³ se nenhum destino tiver conflito)
scripts/sync-skills.sh install session-handoff --all --apply --replace  # troca versÃ£o antiga, com backup
scripts/sync-skills.sh drift                          # [ok] / [DRIFT] / [nÃ£o instalada] por destino
scripts/sync-skills.sh mirror formato-curso-v5 --dsh  # skill cuja fonte mora em outro repo: espelha e vigia
scripts/sync-skills.sh mirror-drift                   # drift das espelhadas
```

O kit jÃ¡ traz duas skills canÃ´nicas em `skills/`: `session-handoff` (escreve o handoff no fim da sessÃ£o) e `prime` (lÃª AGENTS.md, context/, tasks/ e handoffs/ antes de agir e devolve um briefing com fontes). Juntas, sÃ£o o ciclo diÃ¡rio. O `install` Ã© **prÃ©via por padrÃ£o**: mostra o destino e o estado de cada runtime e sÃ³ grava com `--apply`. Ele checa todos os destinos antes de escrever qualquer um (nada parcial), Ã© idempotente (skill igual = `UNCHANGED`, nada Ã© regravado), recusa sobrescrever skill existente diferente (`CONFLICT`) a menos que vocÃª passe `--replace`, e recusa sempre destino que seja ou contenha symlink. Backups do `--replace` vÃ£o para `~/.claude/skills-backup/` e `~/.agents/skills-backup/`, nunca dentro da pasta de skills. O destino do Codex Ã© `~/.agents/skills` (onde o Codex CLI lÃª skills de usuÃ¡rio).

Teste automatizado do instalador (HOME temporÃ¡rio, nÃ£o toca nos seus runtimes):

```bash
tests/test-sync-skills.sh   # 20 casos: prÃ©via, apply, idempotÃªncia, conflito sem instalaÃ§Ã£o parcial, --replace, symlinks
```

### 5. Provar com sessÃ£o nova em cada runtime

```bash
scripts/readback-test.sh ~/projetos/meu-projeto both
```

Abre uma sessÃ£o nova no Claude (`claude -p`) e no Codex (`codex exec`) dentro do projeto e faz as 5 perguntas de continuidade: objetivo e critÃ©rio de pronto, uma regra com o arquivo de origem, Ãºltima decisÃ£o aceita, prÃ³xima aÃ§Ã£o, conflitos ou acesso faltando. Salva a resposta bruta em `relatorios/readback-<runtime>-<data>.md`.

AprovaÃ§Ã£o Ã© sua, lendo o texto: as respostas citam `AGENTS.md`, `tasks/current.md` e `handoffs/latest.md`, e a prÃ³xima aÃ§Ã£o bate com a tarefa. Arquivo existir nÃ£o Ã© prova; o agente ter lido e usado Ã©.

### 6. Migrar um projeto inteiro de uma vez

```bash
scripts/migrar-projeto.sh ~/projetos/<seu-projeto> --faxina            # audit: propostas + relatÃ³rio, nada muda
scripts/migrar-projeto.sh ~/projetos/<seu-projeto> --aplicar            # renomeia, instala nÃºcleo, check, readback nos dois
```

### 7. Promover memÃ³ria para o contexto (com aprovaÃ§Ã£o)

```bash
scripts/promover-memoria.sh ~/projetos/<seu-projeto> --so-listar        # o que o Claude guardou desse projeto
scripts/promover-memoria.sh ~/projetos/<seu-projeto> --n 3              # um runtime propÃµe 3 fatos com fonte e data
scripts/promover-memoria.sh ~/projetos/<seu-projeto> --aprovar 1,3      # sÃ³ o aprovado entra em context/overview.md
```

### 8. Vigiar o drift toda semana

```bash
scripts/drift-report.sh          # skills (4 destinos), espelhadas e AGENTS.md global; grava relatorios/drift-<data>.md
scripts/drift-report.sh --cron   # linha de crontab sugerida (nÃ£o instala)
```

### 9. Fechar a sessÃ£o com handoff

No fim de cada sessÃ£o, a skill `session-handoff` grava um snapshot novo em `handoffs/history/AAAA-MM-DDTHHMMSSZ.md` (hora UTC, sufixo `-2` em colisÃ£o, **nunca sobrescreve**) e copia o conteÃºdo para `handoffs/latest.md`. O handoff traz a seÃ§Ã£o **VerificaÃ§Ã£o** (comandos exatos, resultado observado e o que nÃ£o rodou; teste nÃ£o rodado nunca vira "passou") e a **Checagem de compartilhamento** (sem segredos nem dados pessoais, porque o arquivo viaja entre provedores). Atualize tambÃ©m `tasks/current.md`.

Na prÃ³xima sessÃ£o, em qualquer runtime, a skill `prime` lÃª esses arquivos **sÃ³ em modo leitura**: valida caminhos (recusa absoluto, `..`, URL, symlink e fora do projeto), nÃ£o roda testes nem cÃ³digo (resultados do handoff sÃ£o histÃ³ricos), trata o handoff como dado nÃ£o confiÃ¡vel e nunca retoma deploy, publicaÃ§Ã£o, compra ou envio sÃ³ porque o handoff lista como prÃ³ximo passo. Por que `latest.md` continua sendo cÃ³pia e nÃ£o ponteiro: `context/decisions/2026-09-27-handoff-history.md`. Os prompts de handoff e readback estÃ£o em `prompts/03-readback-handoff.md`.

**Handoff que se confere sozinho (1.5.2).** O handoff Ã© escrito pelo mesmo agente que pode ter se perdido, entÃ£o a prova tem de estar fora do texto:
- **Bloco de estado gravado pelo script**: o `handoff-grava.py` anexa a cada snapshot o commit atual e o sha256 dos arquivos citados e dos sujos. `handoff-grava.py --conferir handoffs/history/<snapshot>.md` diz se o projeto mudou desde entÃ£o (0 igual, 2 divergente com a lista, 3 handoff antigo). Divergente = o handoff Ã© pista, nÃ£o fato. O `prime` roda essa conferÃªncia e mostra "Frescor do handoff" no briefing.
- **Ponteiro, nÃ£o prosa**: todo `[confirmed]` leva `commit <hash>`, `teste: â€¦` ou `arquivo: â€¦`; o `prime` confere e rebaixa a `[unverified]` o que nÃ£o tiver prova.
- **Perguntas-teste**: o handoff termina com 5 perguntas fixas (tarefa e critÃ©rio, Ãºltima decisÃ£o e por quÃª, o que nÃ£o pode mudar, o que estÃ¡ pela metade, prÃ³xima aÃ§Ã£o), cada uma com a fonte num arquivo do projeto. Quem retoma responde pelos arquivos, compara e mostra "Perguntas-teste: N/5"; o que diverge Ã© lacuna do handoff e nÃ£o vira aÃ§Ã£o sem vocÃª confirmar.

**Retomar o handoff certo (1.6.3).** Com vÃ¡rias sessÃµes no mesmo projeto, o `/prime` sem argumento abre o handoff mais novo, que pode ser de outra sessÃ£o. Agora o `handoff-grava.py` termina imprimindo o comando com o arquivo exato, e a skill session-handoff fecha a resposta com ele:
```
/clear                                              # no Codex: /new
/prime handoffs/history/2026-10-17T183328Z-robot.md
```
O `prime` aceita esse caminho (usa exatamente esse snapshot; se houver um mais novo do mesmo assunto, avisa como conflito) ou o assunto (`/prime robot`). Sem argumento, lista os trabalhos abertos como antes.

**Trabalho em outro repo (1.7.3).** Se a sessÃ£o estÃ¡ aberta num projeto mas edita outro (ex.: aberta no `wifi`, trabalhando no `robot`), grave com `--repo ~/projetos/robot`: o bloco de estado guarda tambÃ©m o commit e os arquivos alterados desse repo, o `--conferir` confere os dois, e o `prime` busca o estado real lÃ¡ quando precisar. O script avisa (`aviso: â€¦`) quando o corpo cita outro repo sem `--repo` e quando falta a seÃ§Ã£o Perguntas-teste.

**Nota de retomada pelo script (1.12.3).** Cada pergunta-teste leva uma chave curta; o `nota-retomada.py` (execucao-longa) dÃ¡ a nota, guarda as respostas e separa handoff desatualizado de informaÃ§Ã£o perdida. O agente nÃ£o se corrige mais.

**`latest.md` leve (1.11.3).** O Ã­ndice traz sÃ³ um ponteiro para o snapshot mais novo, sem copiar o corpo (medido: wifi 19,9 â†’ 4,8 KB; reel7etapas 22,2 â†’ 0,7 KB; r7ex 14,3 â†’ 0,6 KB).

**Lista de fechados (1.10.3).** Trabalho fechado vai para `handoffs/fechados.md` (slug, snapshot, data e tÃ­tulo), para pesquisar depois; `--recuperar-fechados` traz os antigos pelo log do git e `--arrumar` aplica parados + fechados num projeto existente de uma vez.

**Ãndice com trava e parados (1.9.3).** VÃ¡rios terminais gravando handoff ao mesmo tempo esperam a vez (`flock`) em vez de um apagar a linha do outro, e assunto sem handoff hÃ¡ mais de 3 dias sai dos abertos para **Parados** (volta ao gravar de novo com o mesmo slug; nada Ã© apagado).

**NÃ£o fazer completo (1.8.3).** O handoff separa **DecisÃµes** de **NÃ£o fazer (restriÃ§Ãµes e travas)**, esta com todas as restriÃ§Ãµes, uma por item e com fonte, e ganha **Pela metade** e o porquÃª da prÃ³xima aÃ§Ã£o. O `prime` mostra a lista NÃ£o fazer inteira no briefing (nÃ£o uma regra sÃ³); o script avisa quando a seÃ§Ã£o falta.

## Usar por prompt, sem scripts

| Arquivo | Quando usar |
|---|---|
| `prompts/01-migrate-claude.md` | Prompt A: vocÃª jÃ¡ tem um setup Claude e quer levar partes selecionadas pro Codex |
| `prompts/02-build-agnostic-workspace.md` | Prompt B: criar ou adaptar um projeto pra ser portÃ¡til entre ferramentas (com add-ons pessoal / cliente) |
| `prompts/03-readback-handoff.md` | prompts curtos de verificaÃ§Ã£o e de handoff |
| `prompts/04-quickstart-exemplos.md` | versÃµes curtas em modo audit e exemplos preenchidos |
| `prompts/05-usar-os-dois.md` | usar Claude e Codex juntos: quem faz o quÃª, planejar e criticar (mÃ¡x. 2 rodadas), construir em branch e revisar o diff, meta com condiÃ§Ã£o de parada, handoff/prime entre runtimes. Adaptado do kit MIT "Use Both" (Prompt Advisers / Mark Kashef): [guia](https://inematds.github.io/use-both-claude-codex/guia/) Â· [repo](https://github.com/inematds/use-both-claude-codex) |

Preencha os campos entre colchetes, mantenha `MODE: audit` na primeira rodada, leia o plano que o agente devolve, e sÃ³ entÃ£o rode de novo com `MODE: implement`.

## Um contexto, vÃ¡rias plataformas (vÃ­deo + modelo pronto)

[![Claude e Codex com o mesmo contexto: vÃ­deo de 7min36 com desenhos](videos/poster.jpg)](https://inematds.github.io/agente-claude-codex/videos/)

**VÃ­deo:** [Claude e Codex com o mesmo contexto](https://inematds.github.io/agente-claude-codex/videos/) (7min36, PT, legendas, 13 capÃ­tulos). Em desenhos: do que o contexto Ã© feito, nÃ­vel 1 no computador, nÃ­vel 2 na nuvem (GitHub privado), as instruÃ§Ãµes do projeto e como trocar de IA sem refazer nada.

| | |
|---|---|
| ![A camada de contexto portÃ¡til](guia/assets/il/02-camada.jpg) | ![Ida e volta: o Codex cria, o Claude edita](guia/assets/il/11b-ida-volta.jpg) |
| ![As instruÃ§Ãµes do projeto](guia/assets/il/10a-instrucoes.jpg) | ![O que viaja e o que nÃ£o viaja](guia/assets/il/13a-mala.jpg) |

**Modelo pronto** (`template/`, instalado pelo `scripts/init-core.sh`): o contexto mora num GitHub **privado** e cada IA sÃ³ ganha acesso, as mesmas instruÃ§Ãµes e um teste.

- `template/plataformas/instrucoes-projeto.md` â€” o bloco Ãºnico para colar no Projeto do Claude.ai, no Projeto do ChatGPT e no ambiente do Codex na nuvem (ler a main antes, commit + push depois, sÃ³ editar quando pedido, sem segredos, memÃ³ria vira skill/regra/fato).
- `template/plataformas/README.md` â€” como ligar cada plataforma (Claude Code, Codex, Gemini CLI, Claude.ai, ChatGPT, Codex na nuvem) e o teste de ida e volta.
- `template/CLAUDE.md` e `template/GEMINI.md` sÃ³ apontam para `AGENTS.md`; `scripts/check.sh` confere 8 arquivos.

```bash
scripts/init-core.sh ~/projetos/minhaempresa-contexto
bash ~/projetos/minhaempresa-contexto/scripts/check.sh
```

Testado em 13/10/2026: com o mesmo bloco de instruÃ§Ãµes, o Codex criou `context/voz.md` (commit + push) e o Claude leu, editou e fez commit + push no mesmo remoto. Os conectores do Claude.ai e do ChatGPT seguem o mesmo bloco, mas dependem da conta de cada um: faÃ§a o teste de ida e volta ao ligar. IlustraÃ§Ãµes em EN/ES no guia (`guia/en/`, `guia/es/`).

## O que jÃ¡ foi testado nesta mÃ¡quina (2026-09-13 e 2026-09-16)

| Passo | Resultado |
|---|---|
| doctor.sh | passou (2 avisos: sem MCP no Codex, Codex CLI sem import) |
| audit.sh | passou: 89 skills sÃ³ no Claude (71 reutilizÃ¡veis, 15 adaptador, 2 nativo, 1 sem SKILL.md) |
| adapt-instructions.sh | passou em dry-run no CLAUDE.md global (71 linhas portÃ¡teis, 7 resÃ­duo) |
| init-core.sh + check.sh em clone isolado | passou |
| readback-test.sh neste repo | passou no Codex e no Claude, com as respostas em `relatorios/` |
| sync-skills.sh (2026-09-16) | passou: session-handoff e prime instaladas em claude, codex, dsh e v3, drift zero; 3 skills espelhadas no dsh sem drift |
| Fase 0 (2026-09-16) | passou: `~/.codex/AGENTS.md` gerado; sessÃ£o nova do Codex citou autor, keys e regra de ritmo com arquivo e linha; magnific e metricool registrados no Codex (login OAuth pendente: `codex mcp login <nome>`) |
| prime em sessÃ£o nova do Codex (2026-09-16) | passou: briefing com fontes e apontou 3 contradiÃ§Ãµes reais entre handoff e tarefa |
| migrar-projeto.sh em audit no wifi (2026-09-16) | passou (relatÃ³rio em relatorios/) |
| promover-memoria.sh com 50 memÃ³rias (2026-09-16) | passou: 3 fatos propostos, 1 aprovado entrou no overview de teste |
| drift-report.sh (2026-09-16) | passou: sem drift |
| tests/test-sync-skills.sh (2026-09-27) | passou: 20/20 (instalador em modo prÃ©via) |
| readback-test.sh codex (2026-09-27, v1.1.0) | passou: citou tasks/current.md, AGENTS.md, a decisÃ£o de 2026-09-27 e handoffs/latest.md; conferiu latest = snapshot em handoffs/history/ |

## Cache, handoff automÃ¡tico e compactaÃ§Ã£o: como funciona

**Quanto custa voltar a uma sessÃ£o parada.** A cada mensagem, o Claude Code reenvia a conversa inteira. Com o cache quente (atÃ© ~1 h desde a Ãºltima resposta), essa releitura sai por ~10% do preÃ§o normal de entrada. Depois de 1 h o cache vence: a prÃ³xima mensagem reprocessa tudo e grava o cache de novo (~2Ã— o preÃ§o normal, cerca de 20Ã— uma mensagem com cache quente). Numa sessÃ£o em 48% do contexto, sÃ£o ~480 mil tokens relidos de uma vez. Na assinatura isso nÃ£o vira dinheiro, mas sai da cota de uso (limite de 5 h e semanal). Paga-se uma vez sÃ³: as mensagens seguintes voltam a ser baratas.

**Por isso o handoff tem hora certa.** Gravado com o cache quente, o handoff custa uma mensagem barata. Pedido depois que o cache venceu, ele prÃ³prio paga a releitura cheia. O `/compact` custa o mesmo, porque tambÃ©m lÃª tudo. Se o trabalho jÃ¡ estÃ¡ em arquivos (commits, pendÃªncias, CLAUDE.md), dÃ¡ para fazer `/clear` + `/prime` sem handoff, e sai barato, porque o prime lÃª sÃ³ o necessÃ¡rio. SÃ³ vale pagar a releitura quando hÃ¡ decisÃµes que ficaram apenas na conversa.

**Handoff automÃ¡tico por ociosidade.** O agente sÃ³ roda quando chega uma mensagem, e os hooks disparam por evento, nÃ£o por relÃ³gio: sem ajuda, nada grava o handoff enquanto vocÃª estÃ¡ em outra coisa. O hook `tools/hook-longrun.py` do projeto [execucao-longa](https://github.com/inematds/execucao-longa) resolve isso no evento `Stop`, gravando com o `handoff-grava.py` da skill `session-handoff` deste kit:

1. Ao fim de cada resposta, com contexto â‰¥ 200 mil tokens (~20% da janela de 1M) e a pasta `handoffs/` no projeto, agenda um timer (`systemd-run --user`, um por sessÃ£o, refeito a cada resposta) para daqui a 45 min.
2. Quando o timer dispara, o hook confere trÃªs coisas: a sessÃ£o continua parada, nÃ£o hÃ¡ handoff desta sessÃ£o gravado atÃ© 5 min antes da Ãºltima resposta, e o cache ainda vale (< 55 min).
3. Se as trÃªs baterem, grava o handoff numa **cÃ³pia** da sessÃ£o (`claude -p --resume --fork-session`, mesmo modelo, pela assinatura). A cÃ³pia lÃª com o cache quente e sÃ³ pode rodar o `handoff-grava.py` e ler arquivos. A sessÃ£o aberta nÃ£o Ã© tocada.
4. Cada conferÃªncia fica registrada em `~/.local/state/execucao-longa/ocioso.log` (gravou, ou por que nÃ£o gravou).

**Linha do tempo de uma sessÃ£o parada:**

| Tempo parado | O que acontece |
|---|---|
| 0 | a resposta termina e o timer de 45 min Ã© agendado |
| 3â€“12 min | se vocÃª trocou de foco, o Claude Code grava um "resumo de ausÃªncia" (`away_summary`) na conversa; isso **nÃ£o** conta como atividade |
| 45 min | handoff gravado na cÃ³pia, com o cache quente |
| ~54 min | o Claude Code compacta sozinho por ociosidade (sÃ³ acima de ~200 mil tokens), ainda com o cache quente |
| 60 min | o cache vence |
| ao voltar | se compactou: a conversa segue do resumo, barata, e o hook pede para reler o handoff junto. Se nÃ£o compactou e passou de 60 min: o hook segura a primeira mensagem uma vez (funÃ§Ã£o `pausa` do hook) e mostra `/clear` + `/prime <handoff desta sessÃ£o>` |

**Por que 45 min e nÃ£o 50.** A compactaÃ§Ã£o por ociosidade vem entre 53 e 58 min, e a cÃ³pia leva 30â€“60 s mais a partida do Claude. Aos 50 min ela pode chegar depois do resumo ou passar do limite de 55 min do cache. O custo Ã© o mesmo a qualquer hora antes de 1 h (uma leitura com o cache quente); 45 deixa folga.

**A compactaÃ§Ã£o automÃ¡tica nÃ£o substitui o handoff.** O resumo perde detalhe (decisÃµes, arquivos, o que nÃ£o fazer), e ela nem sempre acontece. Em 18/10/2026, uma sessÃ£o parada com 480 mil tokens nÃ£o foi compactada, e a volta pagou a releitura cheia. O handoff aos 45 min Ã© a proteÃ§Ã£o; a compactaÃ§Ã£o Ã© bÃ´nus quando vem.

**O que jÃ¡ fez o handoff automÃ¡tico falhar** (corrigido na 1.8.5 e na 1.8.6):

- **Data do arquivo.** O Claude Code acrescenta linhas sem carimbo depois da resposta, e o hook lia isso como "a sessÃ£o andou". Agora ele decide pela Ãºltima linha com carimbo.
- **Troca de foco e `/exit`.** O `away_summary` e os comandos locais tÃªm carimbo e tambÃ©m pareciam atividade: 7 sessÃµes de 16/10 ficaram sem handoff. Agora sÃ³ contam mensagens do usuÃ¡rio e respostas do agente.
- **Handoff velho contando como recente.** Um handoff das 07:38 cobria uma resposta das 08:06, e a meia hora final se perdia. Agora sÃ³ vale o handoff gravado atÃ© 5 min antes da Ãºltima resposta. Se o cache vencer com um handoff automÃ¡tico anterior ao Ãºltimo trabalho, o aviso diz isso, em vez de "nenhum handoff".

| VariÃ¡vel | PadrÃ£o | Efeito |
|---|---|---|
| `LONGRUN_OCIOSO_MIN` | `45` | minutos parado antes de gravar (nÃ£o passe de ~45) |
| `LONGRUN_OCIOSO_TOKENS` | `200000` | contexto mÃ­nimo para gravar; `0` desliga |
| `LONGRUN_CACHE_MIN` | `55` | depois disso a cÃ³pia nÃ£o roda (o cache jÃ¡ estaria frio) |

## O que tem aqui

| Pasta | ConteÃºdo |
|---|---|
| `PLANO.md` | anÃ¡lise dos docs + auditoria desta mÃ¡quina + passos e critÃ©rios de aceite |
| `prompts/` | Prompt A, Prompt B, readback/handoff, quick-starts, usar os dois juntos, em texto copiÃ¡vel |
| `scripts/doctor.sh` | diagnÃ³stico do ambiente: ok / aviso / falta, com o que instalar |
| `scripts/audit.sh` | inventÃ¡rio somente leitura Claude x Codex â†’ matriz reutilizÃ¡vel / adaptador / nativo |
| `scripts/adapt-instructions.sh` | CLAUDE.md â†’ AGENTS.md (portÃ¡til) + CLAUDE.md (`@AGENTS.md` + resÃ­duo) |
| `scripts/init-core.sh` | copia `template/` pra um projeto sem sobrescrever nada existente |
| `scripts/sync-skills.sh` | import / build / install (prÃ©via por padrÃ£o, `--apply` para gravar) / drift de skills via polyskill (uma fonte canÃ´nica) |
| `tests/test-sync-skills.sh` | teste automatizado do instalador em HOME temporÃ¡rio |
| `scripts/readback-test.sh` | teste de continuidade: sessÃ£o nova em cada runtime responde as 5 perguntas |
| `scripts/migrar-projeto.sh` | encadeia adapt â†’ init-core â†’ check â†’ readback num projeto e grava `relatorios/migracao-<projeto>.md` (audit por padrÃ£o, `--aplicar` para valer) |
| `scripts/faxina.sh` | classifica cada seÃ§Ã£o de um CLAUDE.md longo: fica no AGENTS.md, vira `context/`, ou Ã© resÃ­duo |
| `scripts/drift-report.sh` | relatÃ³rio de drift: skills canÃ´nicas nos 4 destinos, skills espelhadas e AGENTS.md global; `--cron` mostra a linha semanal |
| `scripts/drift-instructions.sh` | avisa quando `~/.claude/CLAUDE.md` mudou depois do `~/.codex/AGENTS.md` ou quando falta regra portÃ¡til lÃ¡ |
| `scripts/promover-memoria.sh` | lista a memÃ³ria nativa do Claude de um projeto, um runtime propÃµe atÃ© N fatos com fonte e data, e sÃ³ `--aprovar` grava no `context/overview.md` |
| `skills/` | skills canÃ´nicas em formato polyskill: `session-handoff` (escreve o handoff) e `prime` (lÃª o contexto antes de agir) |
| `template/` | nÃºcleo portÃ¡til (AGENTS.md, CLAUDE.md, context/, tasks/, handoffs/, .agents/skills/, scripts/check.sh) |
| `relatorios/` | saÃ­da das auditorias e readbacks |
| `context/`, `tasks/`, `handoffs/` | o prÃ³prio repo usa o nÃºcleo que propÃµe |
| `FALHAS.md` | uma linha por falha corrigida |
| `VERSION`, `CHANGELOG.md` | versÃ£o semver do kit e o que mudou em cada uma |
| `docs/` | material de origem (local, fora do git) |

## Regras

- Modo audit antes de implement. Nada em `~/.claude` ou `~/.codex` Ã© copiado em massa ou apagado.
- Instalar skill Ã© prÃ©via por padrÃ£o; sÃ³ grava com `--apply`, e sÃ³ sobrescreve com `--replace` (com backup).
- Segredos nunca entram no repo; keys sÃ£o referenciadas, nÃ£o copiadas.
- Todo check Ã© marcado passou, falhou ou nÃ£o rodado. Sem evidÃªncia, conta como nÃ£o rodado.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe