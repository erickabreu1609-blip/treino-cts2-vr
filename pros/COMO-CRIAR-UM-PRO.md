# Como transformar um PRO em treinamento VR

Cada procedimento (PRO, PGS, IT) vira **um arquivo JSON** nesta pasta. O app lê os arquivos listados em `indice.json` e monta sozinho, a partir de cada arquivo:

- o cartão na biblioteca de treinamentos;
- o menu do treinamento, com um botão por roteiro;
- a escolha de papel (coordenador ou executante; nas trocas com o KGT, também operador de EVP) antes de começar;
- as etapas (decisões, toques nas peças da cena, falas de rádio, efeitos na cena);
- os desafios desse tema, que entram no menu Desafios e no sorteio geral;
- o resultado no relatório e na planilha CSV.

Nenhuma linha de código precisa mudar para um PRO novo.

## Caminho para um PRO novo

1. Envie o PDF ou DOCX do PRO no projeto e peça: "transforme este PRO em treinamento VR".
2. O Claude lê o PRO e escreve o JSON seguindo este guia, citando o item do PRO em cada explicação.
3. Para testar antes de publicar: na biblioteca do app, toque em **Carregar PRO (.json)** e escolha o arquivo. Ele fica só naquele celular e pode ser removido na mesma tela.
4. Para todos os aparelhos: o arquivo vai para a pasta `pros/` do site e o nome dele entra em `indice.json`.

O app confere o arquivo ao carregar. Erros (campo faltando, tipo de etapa desconhecido, efeito que não existe) impedem o carregamento e aparecem numa lista. Avisos (por exemplo, uma peça que ainda não existe na cena 3D) não impedem: a etapa vira uma confirmação com o botão Continuar.

## Regras de conteúdo

- Tudo vem do PRO. Cada `explica` cita o item ("PRO-022721 10.11: ..."). Não inventar valor, faixa ou regra.
- Rádio: falas entre a equipe de campo e o CCP, nos dois sentidos, são de rádio. Começam com o bip do botão e terminam com "câmbio" (o app põe os dois sozinho). O padrão de fraseologia vem do ROF (PGS-002722 itens 20 a 24); não inventar além dele.
- Instruções ao participante (do coordenador para a equipe no local) e leitura do texto do quadro ou de uma placa **não** usam câmbio nem bip. Marque essas falas com `"instrucao": true` ou ponha em `falas`.
- Papéis: cada etapa diz quem a faz. Onde o PRO não define, use o bom senso: **coordenador** fala com o CCP, pede e devolve a licença ou a LDL, decide restrições, vigia a segurança e confere o serviço; **executante** faz as ações na máquina e na via e as medições.
- Bitola 1600 mm (EFC), navegação por dois toques e a regra do comutador continuam valendo; nada disso vai no JSON.

## Estrutura do arquivo

```json
{
  "formato": "treino-pro-vr/1",
  "id": "lub",                               // letras minúsculas, números, - ou _ (único)
  "pro": "PRO-016249 rev. 08 · EFC",         // código e revisão, aparece no cartão
  "titulo": "Lubrificação manual do AMV",
  "etiqueta": "Lubrificação do AMV",         // nome curto para o topo do quadro (opcional)
  "resumo": "Texto do cartão na biblioteca.",
  "grupo": "AMV",                            // prefixo do módulo no relatório
  "prefixo": "VP",                           // prefixo de rádio sorteado: VP-10 a VP-89
  "exigeLicenca": true,                      // desafios: tocar nas peças pede licença do CCP (padrão true)
  "papeis": {                                // sem "papeis" o treino não pergunta o papel
    "coordenador": "O que o coordenador faz neste PRO.",
    "executante": "O que o executante faz neste PRO."
  },
  "falaInicio": "Instrução do coordenador ao começar (sem câmbio).",
  "falaFim": "Instrução ao terminar sem erros.",
  "sorteio": { ... },                        // variáveis sorteadas a cada vez (abaixo)
  "listas": { ... },                         // listas de casos para etapas do tipo "lista"
  "roteiros": [ ... ],                       // um botão no menu para cada roteiro
  "desafios": [ ... ]                        // pelo menos dois, ou nenhum
}
```

### Variáveis e textos com {campos}

Sempre existem: `{prefixo}` (ex.: VP-37), `{num}` (número da LDL), `{numErrado}` (número parecido, para a resposta errada do cotejamento), `{ini}` e `{fim}` (agora e daqui a 2 horas), `{hora}` (hora do momento da fala).

Em `sorteio`:

| tipo | exemplo | uso no texto |
|---|---|---|
| `chance` | `"chuva": { "tipo": "chance", "p": 0.4 }` | `"se": "chuva"` na etapa; `{chuva?texto se sim|texto se não}` |
| `faixa` | `"folga": { "tipo": "faixa", "ok": [12, 13], "fora": [[8, 10], [15, 17]], "casas": 0 }` | `{folga.txt}` mostra o valor; `{folga.ok?Dentro|Fora}`. 60% das vezes cai dentro da faixa |
| `numero` | `"t": { "tipo": "numero", "min": 28, "max": 50 }` | `{t}` |
| `um` | `"amv": { "tipo": "um", "de": ["AMV 03", "AMV 05"] }` | `{amv}` |
| `texto` | `"licenca": { "tipo": "texto", "valor": "LDL número {num}, das {ini} às {fim}" }` | `{licenca}`, montado com as outras variáveis |

`"se": "!chuva"` mostra a etapa só quando a variável é falsa.

### Roteiro

```json
{ "id": "lubrificacao", "botao": "Começar", "modulo": "Lubrificação manual (PRO-016249)",
  "descricao": "Uma linha que aparece no menu do treino.", "etapas": [ ... ] }
```

Quiz (perguntas de sim ou não, sem papel):

```json
{ "id": "quiz", "tipo": "quiz", "botao": "Precisa\nde LDL?", "pergunta": "Precisa de LDL?",
  "respostas": { "s": "Precisa de LDL", "n": "Não precisa de LDL" }, "quantas": 6,
  "itens": [ ["Situação.", "s", "Explicação com o item do PRO."] ] }
```

### Etapas

Campos comuns: `tipo`, `papel` (`coordenador`, `executante`, `operador` ou `ambos`), `se`, `titulo`.

**decisao**: múltipla escolha. A certa e as erradas são embaralhadas na tela.

```json
{ "tipo": "decisao", "papel": "coordenador", "titulo": "Chamada ao CCP",
  "visto": "O que o participante vê ou ouve.",
  "certa": "A ação certa pelo PRO.",
  "erradas": ["Erro comum 1.", "Erro comum 2."],
  "explica": "Item do PRO que justifica.",
  "critico": { "2": "Texto do erro crítico se escolher a 2ª errada." },
  "repetir": true,
  "radioAntes": [ { "quem": "CCP", "texto": "{prefixo}, ..." } ],
  "radioSeErrar": [ { "quem": "CCP", "texto": "{prefixo}, negativo. Repita." } ],
  "radio": [ { "quem": "Coordenador", "texto": "CCP, {prefixo}. ..." } ],
  "falas": [ { "quem": "Coordenador", "texto": "Equipe, ..." } ],
  "efeitos": ["ldl:abrir:preventiva do AMV"] }
```

- `critico`: o número é a posição da resposta errada (1 = primeira). Erro crítico reprova o treino e toca a sirene.
- `repetir`: errou, tem de escolher de novo (cotejamento).
- `radio` e `radioAntes`: rádio com o CCP (bip e câmbio automáticos). `falas`: instrução, sem câmbio.
- `efeitos`: rodam depois da resposta.
- Decisão do **executante**, quando o participante é o coordenador, vira **conferência**: o executante propõe uma ação (às vezes errada) e o coordenador diz se confere. Para não ter conferência, use `"conferencia": false`.

**tocar**: o participante toca na peça que pisca.

```json
{ "tipo": "tocar", "papel": "executante", "alvo": "agulhas", "titulo": "Bloqueie as partes móveis",
  "corpo": "Instrução. Toque nas agulhas.",
  "prep": ["objetos:ghost:1:off"],
  "errado": { "placaDentro": "Erro crítico se tocar nesta peça." },
  "efeitos": ["cepo:on", "som:encaixe"],
  "radio": [ ... ] }
```

Quando a etapa é do outro papel, o avatar faz sozinho e o quadro mostra "Acompanhe".

**lista**: sorteia `n` casos de uma lista e faz uma decisão para cada um.

```json
"listas": { "ANOMALIAS": [ ["O que foi visto.", "Classificação certa", ["Errada 1", "Errada 2"]] ] },
{ "tipo": "lista", "lista": "ANOMALIAS", "n": 3, "papel": "executante", "titulo": "Classifique",
  "pergunta": "Qual a prioridade?", "nome": "Prioridade: {visto}", "explica": "PGS-006334: {visto} {certa}." }
```

**sorteio**: escolhe `n` etapas de `de` (imprevistos).

```json
{ "tipo": "sorteio", "n": 2, "de": [ { "tipo": "decisao", ... }, { "tipo": "decisao", ... } ] }
```

**info**: só mostra um texto e o botão Continuar (pode ter `falas`, `radio` e `efeitos`).

**operar**: o participante vai para a cabine do KGT da cena e cumpre uma meta com os controles (o quadro de controles fica dentro da cabine). O KGT dirige como um carro (volante, Frente/Ré ou o controle na tela). Se ele entrar na via sem LDL ou licença, toca o alarme de ocupação indevida e o próximo trem pode bater nele (falha crítica, o treino recomeça). Quando a etapa é do outro papel, o KGT se move sozinho. Pode ter `falas`, `radio` e `efeitos`, que rodam quando a meta é cumprida.

```json
{ "tipo": "operar", "papel": "executante", "meta": "freio", "titulo": "Teste de freio",
  "corpo": "Solte o freio de estacionamento, ande a no máximo 5 km/h e freie.",
  "explica": "PRO-031308, teste dos freios: aplicar o freio com velocidade máxima de 5 km/h." }
```

Metas: `partida` (ligar o motor com o freio de mão puxado), `transporte` (braço e cabine para a frente, lança elevada), `freio` (andar a no máximo 5 km/h e frear), `circular` (buzinar uma vez, andar 3 m a no máximo 20 km/h e parar), `patolas` (baixar os estabilizadores), `giro` (girar 90 graus com patolas baixadas e voltar), `recolherPatolas`, `estacionar` (parar, freio de estacionamento e motor desligado). Passar do limite de velocidade ou dar a partida sem freio conta como erro. Com alguém no raio de ação a máquina não gira nem anda.

**troca**: troca de componente de verdade na cena 3D. O participante solta as fixações na ordem (cada uma pisca), o KGT da cena retira a peça velha e assenta a nova, e depois o participante aplica as fixações (aparecem transparentes). Durante a movimentação do KGT a equipe vai para a zona segura (faixa verde com cones); quem entra no raio de ação pausa o KGT e conta erro. Quando a etapa é do outro papel, o mantenedor faz sozinho.

```json
{ "tipo": "troca", "componente": "travamento", "acao": "soltar", "grupo": "juncao",
  "titulo": "Desacople as unidades", "corpo": "Solte os parafusos da peça de junção.",
  "explica": "PRO-001122 Anexo 5.4, passo 5." }
```

- `componente` e seus grupos de fixações (`grupo`, um só, ou `grupos`, lista; sem nenhum, vale o componente inteiro):
  - `agulha` (agulha do lado da máquina): `barras` (contrapino, pino e cliper de cada lado), `coice` (4 parafusos e as 2 talas de junção), `bondeamento` (cabo de cobre do coice).
  - `motor` (unidade de motor da CTS-2): `conector` (conector militar), `juncao` (parafusos da peça de junção), `dormente` (4 parafusos no dormente oco).
  - `travamento` (unidade de travamento da CTS-2): `clipers`, `conector`, `juncao`, `dormente`.
- `acao`: `soltar` e `fixar` (fixações do grupo, na ordem em que piscam; `fixar` é a ordem inversa), `descarregar` (o KGT traz a peça nova pendurada e a deposita ao lado da via), `retirar` (o KGT iça a peça velha; se ainda houver fixação presa, não iça e conta erro), `posicionar` (o KGT pega a peça nova e assenta no lugar). Sem `descarregar`, a peça nova já está no palete ao lado da via.
- `ferramenta` (opcional): ferramenta mostrada em cada fixação, no lugar da padrão de cada uma.
- Nas ações com o KGT (`descarregar`, `retirar`, `posicionar`) o papel é `operador` (operador de EVP), e o roteiro passa a oferecer esse terceiro papel. **Operador de EVP**: o participante vai para a cabine, o KGT se posiciona e aciona os estabilizadores, e ele leva a garra até a peça com os controles (lança, braço, giro e Girar garra, com Movimento fino para o ajuste), fecha a garra, leva a carga até a caixa verde e toca em Assentar e abrir a garra. O sinaleiro mostra no quadro quanto falta (gire, afaste ou aproxime, suba ou desça, graus da garra). As outras etapas são feitas pela equipe (avatares). **Coordenador e executante**: o operador do KGT (avatar) faz a troca sozinho com a garra.
- Enquanto o KGT trabalha, toda a equipe (mantenedor, coordenador e os outros trabalhadores) fica na zona segura, fora do raio de ação; quando a peça termina de assentar, a equipe volta e aplica as fixações.
- Os componentes novos (agulha, unidade de motor e unidade de travamento) já ficam lado a lado ao lado da via desde a primeira etapa de troca, e o KGT já vem com a garra montada.
- **Implemento do KGT**: o operador de EVP pode trocar a garra pela concha (gancho e cinta; aí o executante passa a cinta) no suporte "IMPLEMENTO RESERVA" ao lado da vaga, na fase de preparação ou pelo botão "Trocar para a concha" da cabine, com o KGT parado e sem carga. Não há campo novo no JSON.
- `papeis.operador` (opcional) descreve o operador de EVP na tela de escolha de papel.
- `papel` padrão: `executante` (nas ações com o KGT, `operador`).

### Peças da cena (`alvo`, `errado`, `pistas`)

`agulhas`, `encosto` (trilho de encosto), `clips`, `barra`, `dormente` (dormente oco), `coifa`, `trava`, `motor`, `tampa` (tampa do motor), `comutador`, `manivela`, `caixa` (caixa de junção), `chave` (chave de comando local), `lampada`, `conA`, `conB` (conectores militares), `dam`, `cordoalha`, `placaPT` (placa Pessoas Trabalhando), `placaDentro` (placa no lugar errado, dentro do gabarito), `jumper`, `cepo`.

Uma peça que não está na lista vira etapa de confirmação, até alguém modelar a peça em 3D.

### Efeitos (`efeitos` e `prep`)

| efeito | o que faz |
|---|---|
| `cepo:on` / `cepo:off` | põe ou tira o cepo entre a agulha aberta e o trilho de encosto |
| `objetos:<placa>:<dentro>:<jumper>` | placa Pessoas Trabalhando e jumper: `on`, `off` ou `ghost` (marcado para tocar); `<dentro>` 1 mostra a placa errada dentro do gabarito. Ex.: `objetos:on:0:off` |
| `caixa:abrir` / `caixa:fechar` | tampa da caixa de junção |
| `amv:normal:auto`, `amv:reverso:manual` | movimenta as agulhas por comando ou pela manivela (espera terminar) |
| `ferramenta:<id>:<peça>:<leitura>` | mostra a ferramenta na peça com a leitura no visor (`regua`, `trena`, `multimetro`, `celula_carga`, `chave_boca`, `radio`) |
| `ferramentas:guardar` | guarda as ferramentas |
| `lampada:verde` / `lampada:off` | lâmpada piloto |
| `som:<nome>` | `encaixe`, `rosca`, `clique`, `chave`, `tampa`, `rele`, `trava`, `bip` |
| `espera:<ms>` | pausa antes do próximo efeito |
| `ldl:abrir:<serviço>` | CCP abre a LDL pelo rádio, com placa e jumper na via |
| `ldl:aberta` | marca a LDL como aberta, sem fala (quando o JSON já tem as falas) |
| `ldl:devolver:<resumo>` | coordenador devolve a LDL pelo rádio e o CCP encerra |
| `ldl:encerrar:<motivo>` | encerra a LDL sem fala |
| `evento:<texto>` | registra um evento do CCP no relatório |
| `trem:<segundos>` | um trem de minério passa daqui a tantos segundos |
| `licenca:conceder:<quem>:<registro>` | licença do CCP fora de uma LDL (ex.: acesso do KGT à via): o trem fica retido e o CCP pede permissão para circular. `licenca:devolver:<registro>` devolve a linha |
| `comutador:manual` / `comutador:auto` | chave de comutação do motor em MANUAL ou de volta para AUTOMÁTICO |
| `dam:instalar` / `dam:retirar` | conecta ou desconecta o DAM da máquina |
| `kgt:voltar` | o KGT recolhe a lança e volta para a vaga (fim de um roteiro com `troca`) |

### Desafios

Cada desafio é um problema que o participante investiga. O diagnóstico oferece como opções todos os desafios do mesmo PRO, então são necessários pelo menos dois.

```json
{ "id": "cepo_esquecido", "nome": "Cepo esquecido nas agulhas", "curto": "Cepo\nesquecido", "critico": true,
  "relato": [ { "quem": "CCP", "texto": "Comandamos o AMV 03 para reversa e ele não completou." } ],
  "prep": ["cepo:on"],
  "pistas": { "cepo": "O que se vê ao tocar nesta peça.", "agulhas": "..." },
  "verificacoes": [ { "id": "registro", "botao": "Ver o\nregistro", "texto": "O que a verificação mostra." } ],
  "correcao": "O que fazer, com o item do PRO." }
```

- `relato`: quem avisa o problema. CCP é rádio; o coordenador passando a tarefa ao participante usa `"instrucao": true`.
- `pistas`: peça tocada → texto. Peça sem pista mostra "Nada anormal".
- `verificacoes`: botões extras (consultar o PRO, ler um registro, olhar algo que não tem peça 3D).
- `critico`: diagnóstico errado toca a sirene e conta como erro crítico.

## Conferência rápida antes de publicar

- O JSON abre sem erro em qualquer validador de JSON.
- Carregou pelo botão Carregar PRO sem erros.
- Cada `explica` e cada `correcao` citam o item do PRO.
- Fala com o CCP está em `radio`; fala do coordenador para a equipe está em `falas` ou com `"instrucao": true`.
- Cada etapa tem `papel`, e os dois papéis têm tarefas.
