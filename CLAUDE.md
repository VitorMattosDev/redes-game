# CLAUDE.md

Guia para o Claude Code trabalhar neste repositório. Idioma do projeto: **português (pt-BR)**.

## O que é

**Operação Fibra — Simulador do Técnico de Provedor.** Um jogo educativo web que
**gamifica o "Manual de Redes de Computadores"** (`~/Downloads/Manual-de-Redes-de-Computadores.pdf`,
231 páginas, foco em provedores FTTx/PON e rádio rural). O objetivo é tornar o jogador
**capaz de administrar uma rede de provedor de verdade**: instalar ONT, provisionar OLT,
configurar VLANs/switches, projetar PON, ler OTDR, planejar enlaces de rádio, operar o NOC,
calcular sub-redes/CIDR e dominar OSI/TCP-IP, BGP, MPLS, segurança e troubleshooting.

O jogo segue o manual **parte por parte** (o manual tem 14 Partes; a prova tem 14 módulos
nomeados "Parte I" … "Parte XIV", na mesma ordem e com os mesmos temas). Há **22 missões
interativas** (m1–m22) além da prova.

## Arquitetura

- **Tudo vive em um único arquivo: `index.html`** (~1900 linhas). Não há build, bundler,
  framework, `package.json` nem dependências instaláveis. HTML + `<style>` + `<script>` inline.
- **Única dependência externa:** Google Fonts via CDN (Saira Condensed, IBM Plex Mono, Inter).
  Funciona offline exceto pelas fontes.
- **Rodar:** abrir `index.html` no navegador (duplo-clique ou `start index.html`).
  Para evitar qualquer questão de CDN/CORS, servir com `python -m http.server` e abrir
  `http://localhost:8000`. Não há passo de compilação — editar e recarregar.
- **Estrutura de telas:** `<section class="screen">` com `id="scr-*"`. Só uma fica `.active`
  por vez; a função `show('scr-id')` troca. Telas: `scr-map` (mapa/hub), `scr-exam`
  (grade de módulos da prova), `scr-quiz` (motor de questões), `scr-m1`…`scr-m22` (missões),
  `scr-cert` (certificado final).

### Estado e progresso

- O estado global é `const S = { xp, done{}, badges{}, ontRx, plan }`. As notas da prova ficam
  em `let E = { scores, ... }`. `S.done`/`S.badges` têm chaves `m1`…`m22` (+ `badges.exam`).
- **Persistência:** `saveState()`/`loadState()` gravam `{xp, done, badges, scores}` em
  `localStorage` sob a chave `SAVE_KEY = 'operacao-fibra-v1'`. `loadState()` roda no boot e faz
  merge defensivo (só sobrescreve chaves conhecidas — adicionar missões nova não quebra saves
  antigos). `saveState()` é chamado em `addXP`, `completeMission` e `quizEnd`. **Ao adicionar
  estado novo que precise sobreviver ao reload, inclua-o em `saveState`/`loadState`.**
- `resetGame()` pede confirmação, **limpa o `localStorage`** e recarrega.
- `addXP(n, ev)` soma XP, salva e dispara a animação flutuante. `completeMission(id)` marca
  `done`/`badge`, salva, atualiza o HUD e, quando **todas** as missões de `S.done` estão `true`,
  abre o certificado automaticamente.

### Estruturas de dados (todas no `<script>`, próximas ao topo)

- `BADGES[]` (~L492) — id, ícone, nome, descrição de cada conquista (m1–m10 + `exam`).
- `MISSIONS[]` (~L505) — cartões do mapa: `{id, ico, t (título), d (descrição), req}`.
  `req` é o id da missão pré-requisito (`null` = sempre liberada). O mapa bloqueia cartões
  cujo `req` ainda não está `done`.
- `EXAM[]` (~L1253) — os 14 módulos da prova. Cada módulo:
  `{id, parte:'Parte N', nome, topics, qs:[...]}`. Aprovação = **≥ 70%** por módulo
  (`passMod`). Concluir os 14 dá +200 XP e o badge `exam`.
- **Tipos de questão** suportados pelo motor de prova (`renderQ`/`bindQ`/`settleQ`, ~L1431):
  - `mc` — múltipla escolha: `{t:'mc', q, opts:[], ok:<índice>, exp}`
  - `tf` — verdadeiro/falso: `{t:'tf', q, ok:<bool>, exp}`
  - `num` — cálculo numérico: `{t:'num', q, ans, tol, unit, exp}` (aceita vírgula ou ponto)
  - `order` — ordenar itens: `{t:'order', q, items:[...na ordem correta], exp}`
  - `exp` é a explicação mostrada após responder (sempre presente — é o valor didático).

### Missões interativas (cada uma tem seu próprio bloco de JS, comentado com `/* MISSÃO N */`)

| Missão | Tela | Mecânica | Parte do manual |
|---|---|---|---|
| m1 Instalação da ONT | scr-m1 | passos: conector certo (APC/UPC), drop, LEDs, potência óptica | II / VI |
| m2 Provisionamento OLT | scr-m2 | **terminal simulado**: autofind → autorizar por SN → perfil → VLAN/PPPoE | VI / X |
| m3 Switch e VLANs | scr-m3 | segmentar VLANs, configurar trunk, evitar loop L2 | III |
| m4 Teste de banda | scr-m4 | cabo vs Wi-Fi, interpretar speedtest | V / XIII |
| m5 Camadas OSI/TCP-IP | scr-m5 | animação de encapsulamento + classificar protocolos por camada | I |
| m6 Plantão de chamados | scr-m6 | 5 tickets aleatórios: diagnosticar camada + ação (bottom-up) | XIV |
| m7 Lab Projeto PON | scr-m7 | orçamento de potência ao vivo (`SPLIT_LOSS`, classe, distância, fusões) | VI |
| m8 Lab OTDR | scr-m8 | classificar 5 eventos do traço alternando 1310/1550 nm | VIII |
| m9 NOC sob DDoS | scr-m9 | **terminal**: investigar flow e escolher mitigação (RTBH/Flowspec/scrubbing) | X / XI |
| m10 Treino de sub-redes | scr-m10 | exercícios gerados aleatoriamente: rede/broadcast/hosts pelo número mágico | IV |
| m11 Lab Política BGP | scr-m11 | Local Pref (saída), AS-Path prepending (entrada), RPKI vs hijack — 3 desafios | X |
| m12 Circuitos × Pacotes | scr-m12 | animação dos 3 fluxos (reserva×compartilhamento) + 2 perguntas | I |
| m13 MTU/MSS & PPPoE | scr-m13 | reproduzir o black-hole 1500>1492 e aplicar MSS clamping no BNG | V / XIV |
| m14 Q-in-Q | scr-m14 | empilhar S-TAG sobre C-TAG, ver +4 B por tag + pergunta do porquê | III |
| m15 Enlace de rádio | scr-m15 | sliders → FSPL/EIRP/Fresnel/fade margin ao vivo — 3 desafios (molde m7) | IX |
| m16 Topologias | scr-m16 | classificar física × lógica de 4 cenários (selects), 3/4 para o badge | I / V |
| m17 MPLS | scr-m17 | conduzir o pacote pelo LSP: push→swap→swap→pop salto a salto | XII |
| m18 Roteamento (LPM) | scr-m18 | 6 pacotes: clicar a rota vencedora por longest-prefix match | IV |
| m19 VLSM | scr-m19 | recortar um /24 em 5 setores, maior→menor, prefixo mínimo que cabe | IV |
| m20 DHCP DORA | scr-m20 | stepper Discover→Offer→Request→Ack + Opção 82 no relay | V |
| m21 IPTV & multicast | scr-m21 | toggle IGMP snooping: entrega seletiva vs flood, com join por porta | XIII |
| m22 Wi-Fi 1/6/11 | scr-m22 | atribuir canais a 4 APs vizinhos e zerar a interferência | XIII |

O motor de terminal (m2, m9) renderiza uma barra de comandos por toque (`renderCmdbar`,
`#cmdbar`/`#nocBar`) em vez de input livre — o jogador escolhe o comando certo. Os labs
m7/m11/m15 seguem o mesmo molde "controles → cálculo ao vivo → checklist de 3 desafios"
(funções `mNCalc` + checklist de `goals`).

## Convenções ao editar

- **Não introduza dependências nem build.** Manter single-file, vanilla JS, zero `npm`.
  Se algo exigir uma lib, prefira reescrever em JS puro.
- **Fidelidade ao manual é regra.** Todo conteúdo novo (questão, missão, feedback) deve sair
  do manual. Cada questão tem `exp` explicando o porquê — mantenha esse padrão didático.
  Quando viável, cite a Parte/conceito. Confira números (dB, MTU, prefixos) contra o PDF.
- **Estilo:** classes CSS reaproveitáveis já existem (`.panel`, `.btn`/`.btn.gold`/`.btn.ghost`,
  `.choice`, `.feedback.ok/.bad/.info`, `.kfact`, `.terminal`, `.cmdbar`). Reaproveite-as
  em vez de criar CSS novo. Variáveis de cor no `:root` (ex.: `--fiber`, `--apc`=verde APC,
  `--upc`=azul UPC, `--alarm`=LOS). Fontes display em maiúsculas (Saira), mono = IBM Plex.
- **Helpers:** `$('#sel')` = querySelector. `fb(el, tipo, html)` = caixa de feedback.
  `shuffleArr(a)`. Use-os.
- **Para adicionar uma questão de prova:** ache o módulo certo em `EXAM[]` e acrescente um
  objeto ao `qs[]` no formato do tipo desejado. Nada mais precisa mudar.
- **Para adicionar uma missão:** (1) novo objeto em `MISSIONS[]`; (2) novo
  `<section class="screen" id="scr-mN">`; (3) bloco de JS que renderiza dentro dela e chama
  `completeMission('mN')`/`addXP`; (4) entrada em `BADGES[]`. Siga um bloco existente como molde
  (m7/m10 são os mais autocontidos).

## Mapa de cobertura (manual → jogo)

As 14 Partes do manual **já têm módulo de prova** correspondente (cobertura teórica completa).
A coluna "lab" indica se há **missão interativa hands-on** além do quiz:

| Parte | Tema | Prova | Lab interativo |
|---|---|---|---|
| I | Fundamentos e Modelos (comutação circuitos×pacotes, banda×latência, BDP) | ✅ | ✅ m5 (OSI), m12 (circuitos×pacotes), m16 (topologias) |
| II | Camada Física e Fibra (dB, λ, APC/UPC, fusão, G.657) | ✅ | parcial (m1) |
| III | Camada de Enlace (MAC, flooding, Q-in-Q, STP/ERPS, 802.1Q) | ✅ | ✅ m3 (VLANs), m14 (Q-in-Q) |
| IV | Rede e Endereçamento IP (CIDR, VLSM, sumarização, CGNAT, IPv6) | ✅ | ✅ m10 (sub-redes), m18 (LPM), m19 (VLSM) |
| V | Transporte e Aplicação (TCP/UDP, DNS, DHCP DORA, QoS) | ✅ | ✅ m13 (MTU/MSS), m20 (DHCP), parcial m4 |
| VI | Redes PON (GPON, XGS-PON, splitters, orçamento de potência) | ✅ | ✅ m2, m7 |
| VII | Planta Externa OSP (cabo AS, reserva técnica, georref) | ✅ | — |
| VIII | Medição e Diagnóstico Óptico (OTDR, power-meter, VFL) | ✅ | ✅ m8 |
| IX | Redes via Rádio (FSPL, EIRP, Fresnel, TDMA, SNR, fade margin) | ✅ | ✅ m15 (enlace de rádio) |
| X | Operação de Provedor (BNG, AAA, BGP, peering/trânsito) | ✅ | ✅ m9 (NOC), m11 (BGP) |
| XI | Segurança e Confiabilidade (BCP38, DDoS, três noves) | ✅ | ✅ m9 (NOC + RTBH) |
| XII | Tópicos Avançados (MPLS, SDN/IaC, IPv6) | ✅ | ✅ m17 (MPLS) |
| XIII | Serviços ao Assinante (Wi-Fi 1/6/11, VoIP/RTP, IPTV/multicast) | ✅ | ✅ m21 (IPTV/IGMP), m22 (Wi-Fi) |
| XIV | Troubleshooting (escada bottom-up, MTU/MSS, mtr) | ✅ | ✅ m6, m13 (MTU/MSS) |

## Roadmap de gamificação

**Já implementados** (2 lotes): persistência + m11–m22 cobrindo BGP, topologias, rádio,
circuitos×pacotes, MTU/MSS+PPPoE, Q-in-Q, MPLS, roteamento (LPM), VLSM, DHCP DORA, IPTV/IGMP
e Wi-Fi 1/6/11. As 14 Partes do manual têm prova; **só a Parte VII (OSP) ainda não tem lab
hands-on**. Lacunas que ainda renderiam bons labs (ordem de valor):

1. **Lab de planta externa / OTDR↔GIS (Parte VII)** — converter "evento a 480 m" em endereço real
   e listar os clientes afetados. É a única Parte sem lab interativo.
2. **NAT/CGNAT — mapa de portas (Parte IV/X)** — visualizar tradução privado→público por bloco de
   portas, port exhaustion e o log por bloco (rastreabilidade do Marco Civil). Conceito central de ISP.
3. **Cadeia de resolução DNS (Parte V)** — recursivo → raiz → TLD → autoritativo, com cache/TTL.
4. **TCP em ação (Parte V)** — 3-way handshake, janela/BDP e controle de congestionamento animados
   (por que latência alta dói tanto). Hoje BDP/handshake são só quiz/ordenação.
5. **AAA/PPPoE + CoA (Parte X)** — terminal de sessão: autenticar no RADIUS, aplicar plano, e mudar
   velocidade ao vivo com CoA / mandar inadimplente ao walled garden.
6. **Topologias: virar "builder" de verdade** — hoje m16 é classificador (selects + SVG). Evoluir
   para arrastar OLT/splitter/switch/BNG e ver o broadcast lógico animar.

Ao implementar qualquer item: siga o passo-a-passo de "Para adicionar uma missão", inclua o
estado novo em `saveState`/`loadState` se precisar persistir, mantenha a fidelidade ao manual
e reaproveite o CSS/HUD/badges existentes. Rode o smoke test (abaixo) antes de concluir.

## Testar / validar

- **Sem build:** abra `index.html` no navegador e jogue. Ou sirva com `python -m http.server`.
- **Sanidade de sintaxe JS:** extraia o conteúdo de `<script>` e rode `node --check`.
- **Smoke test funcional:** o jogo foi validado com **jsdom** (`runScripts:'dangerously'`)
  dirigindo cada `initMNN` e conferindo `S.done.mNN===true`. Como `const S`/`let mNN` são
  bindings léxicos (não viram `window.S`), leia-os via `window.eval('S.done.m11')` no teste.
  Os labs de 3 desafios (m7/m11/m15) podem exigir **mais de uma configuração** de sliders na
  mesma sessão para disparar todos os `goals` (cada goal, uma vez cumprido, fica marcado).
