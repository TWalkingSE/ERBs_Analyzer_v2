# Guia de Funcionalidades do ERBs Analyzer v2

## 1. O que é este pacote

O **ERBs Analyzer v2** é uma aplicação de análise técnico-investigativa de extratos telefônicos de telefonia móvel. Ele recebe planilhas de operadoras, normaliza os eventos de voz e conexão, enriquece os dados com a localização de torres (ERBs) e entrega visualizações interativas, análises comportamentais, alertas e relatórios exportáveis.

<img width="886" height="563" alt="image" src="https://github.com/user-attachments/assets/2475f104-45df-4cb6-9541-8c38f6e2e0a0" />


<img width="874" height="822" alt="image" src="https://github.com/user-attachments/assets/d8573a3a-2d7b-456c-b2c5-e774d66e1478" />


<img width="1903" height="950" alt="image" src="https://github.com/user-attachments/assets/de33a525-08e9-4122-b0e4-edf5b7497dd4" />


Este documento explica de forma detalhada o que cada funcionalidade faz, como é interpretada e como o usuário pode aproveitá-la na rotina investigativa.

---

## 1. Conceitos fundamentais

### 1.1 ERB — Estação Rádio Base

ERB é a torre de celular usada para conectar o aparelho à rede móvel. Cada ERB possui um identificador único chamado **CGI** (Cell Global Identity) e, quando disponível, coordenadas geográficas (`latitude` e `longitude`) e o **azimute** — a direção do setor da antena (de 0° a 360°).

No ERBs Analyzer, a ERB é usada para:
- posicionar eventos no mapa;
- estimar área de cobertura por CGI;
- identificar padrões de permanência e deslocamento;
- cruzar eventos com pontos de interesse (POIs).

### 1.2 CGI — Cell Global Identity

O CGI é uma chave composta que identifica uma célula de rede:

`MCC-MNC-LAC-CI`

- **MCC**: Mobile Country Code (país). No Brasil, geralmente `724`.
- **MNC**: Mobile Network Code (operadora). Exemplos: `02`/`04` (TIM), `05`/`12` (Claro), `06`/`10`/`11` (Vivo).
- **LAC**: Location Area Code (área de localização).
- **CI**: Cell Identity (célula específica).

O parser extrai o CGI do extrato e o utiliza para localizar a ERB na base mestre ou na base de CGIs dos próprios extratos.

Nem toda operadora entrega um CGI completo. Quando o extrato traz apenas coordenada ou identificação própria de site, o parser gera uma **chave sintética** (`VIVO:noCGI:...`, `CLARO:{site}:{setor}`) que cumpre o mesmo papel de chave de célula. O filtro por CGI ([3.4.2](#342-filtros)) trabalha com as duas formas.

O CGI também é a unidade de recorte mais fina do mapa: ele identifica um **setor** específico, e não a estação inteira. Uma mesma torre com três setores aparece como três CGIs distintos.

### 1.3 MSISDN, IMEI e IMSI

- **MSISDN**: número do telefone do assinante, com DDD e `55` (ex: `5511912345678`).
- **IMEI**: identificador do aparelho. Serve para saber se o mesmo celular foi usado por chips diferentes.
- **IMSI**: identificador do chip/SIM. Ajuda a detectar troca de chips no mesmo aparelho.

### 1.4 Target / alvo

O alvo é o MSISDN principal da investigação. O sistema tenta detectá-lo automaticamente a partir do nome do arquivo, cabeçalho da planilha, abas, e da frequência do número nos eventos. Quando o dado não permite inferência, o usuário pode selecionar manualmente.

### 1.5 POI — Point of Interest

POI é um ponto de interesse cadastrado manualmente pelo usuário: endereço de residência, trabalho, local de encontro, endereço de testemunha etc. Cada POI tem coordenadas e um raio configurável. O sistema cruza eventos no raio e gera resumos analíticos.

Cada POI recebe um **tipo**, e o tipo define o símbolo e a cor com que ele aparece no mapa (ver [3.5](#35-marcadores-analíticos-pois)). São 13 tipos, agrupados por natureza:

| Grupo | Tipos |
|---|---|
| Locais do fato | Local do crime · Ponto de tráfico · Encontro |
| Moradia | Casa/Residência · Apartamento/Condomínio · Hotel/Motel |
| Atividade | Empresa/Trabalho · Loja/Comércio · Escola/Faculdade · Hospital/Saúde |
| Operacional | Base operacional · Presídio |
| — | Outros |

O tipo é editável a qualquer momento e não altera os dados do POI, apenas como ele é lido no mapa.

### 1.6 Geofence e Corredor

- **Geofence**: área poligonal desenhada no mapa. O sistema responde quais eventos ocorreram dentro da área, em quais dias, por quanto tempo e quais ERBs/CGIs foram dominantes.
- **Corredor**: faixa desenhada ao longo de uma rota (como uma via). Serve para verificar se o alvo passou por determinado caminho.

### 1.7 Stay points (pontos de permanência)

São agrupamentos de eventos em que o aparelho permanece em uma mesma região por um tempo mínimo e dentro de um raio mínimo. Servem para identificar locais de moradia, trabalho, pontos de encontro e padrões de repouso.

### 1.8 Colocalização

**Colocalização** identifica outros MSISDNs que apareceram na mesma célula (CGI) — ou próximos — no mesmo intervalo de tempo do alvo. A ideia é encontrar possíveis companheiros de deslocamento, encontros ou pessoas que estavam no mesmo ambiente no mesmo momento.

O sistema permite configurar:
- **minutos próximos**: margem de tempo para considerar que dois eventos são coincidentes;
- **máximo de pares**: quantos resultados mostrar;
- **limite de mínimos eventos**: quantas coincidências são necessárias para considerar uma colocação relevante.

### 1.9 Velocidade implausível

A ferramenta detecta deslocamentos entre dois eventos consecutivos que exigiriam velocidade superior ao limite configurado (padrão `200 km/h`). Isso pode indicar:
- erro nos dados ou CGIs distantes em curto tempo;
- deslocamento real por meio de transporte muito rápido (avião, etc.);
- necessidade de verificação com rotas reais (via Google Distance Matrix, quando configurado).

### 1.10 Burner phone

**Burner phone** é um telefone descartável, normalmente identificado por uso intenso em pouco tempo e poucos contatos. A ferramenta aplica regras configuráveis para sinalizar aparelhos com:
- vida útil curta (poucos dias);
- baixo número de contatos únicos;
- alta concentração de eventos em janela curta.

### 1.11 Bursts

**Burst** é um pico repentino de atividade (chamadas, conexões ou ambos) em janela curta. Serve para detectar momentos de tensão, preparação de ação, comunicação coordenada ou comportamento atípico.

### 1.12 Período de silêncio

Identifica intervalos longos sem eventos em um aparelho ativo. Pode indicar desligamento, abandono do chip, mudança de aparelho ou viagem.

### 1.13 Troca de IMEI e troca de chip

- **IMEI swap**: o mesmo MSISDN aparece com IMEIs diferentes em curto espaço de tempo.
- **Chip swap**: o mesmo IMEI é usado por IMSIs diferentes, indicando troca de chip no mesmo aparelho.

### 1.14 Chamadas redirecionadas

A função detecta cadeias de redirecionamento de chamadas (`A -> B -> C`). Ajuda a entender encaminhamentos, secretárias eletrônicas e possíveis ocultações de destino final.

### 1.15 Correlacionamento

Permite cruzar dois ou mais extratos para encontrar:
- MSISDNs comuns;
- IMEIs compartilhados;
- CGIs comuns;
- sobreposição temporal de eventos.

---

## 2. Fluxo de trabalho típico

1. **Criar um caso** — organiza os extratos e configurações.
2. **Importar extratos** — upload de planilhas `.xlsx` de operadoras.
3. **Revisar o alvo detectado** — confirmar ou corrigir o MSISDN principal.
4. **Cadastrar POIs** — endereços de interesse com coordenadas e raios.
5. **Explorar o mapa tático** — visualizar eventos, trajetos, heatmap e playback.
6. **Aplicar análises** — Telefones e o dossiê do número, Voz, Conexão, Grafo, Correlação.
7. **Exportar evidências** — CSV, XLSX, KML, PDF, HTML, mapa interativo.

---

## 3. Módulos e funcionalidades

### 3.1 Gestão de casos

Cada investigação é um **caso**. O caso agrupa extratos, POIs, configurações e resultados. É possível:
- criar casos e editar nome e descrição (o identificador do caso não muda);
- abrir o painel do caso com resumo e atalhos;
- ver o histórico de extratos importados;
- acompanhar a auditoria de eventos administrativos.

**Lixeira.** Excluir um caso não apaga nada: a pasta inteira (extratos, POIs, apelidos e a trilha de auditoria do caso) vai para a lixeira, no painel de casos. De lá o caso volta igual com *Restaurar*, ou sai do disco com *Apagar de vez*, que pede confirmação. Não há limpeza automática. Se já existir um caso ativo com o mesmo identificador, a restauração é recusada para não sobrescrever nada. Os casos GPS têm a sua própria lixeira, no painel GPS.

**Levar um caso para outra máquina.** O `case.zip` da tela de exportações leva o caso inteiro com um `SHA256SUMS.txt`. Em *Importar caso (.zip)*, no painel de casos, ele é instalado de volta. Cada arquivo é conferido contra o `SHA256SUMS.txt` antes de qualquer coisa entrar na lista de casos. Um arquivo a mais, a menos, com hash diferente ou com caminho suspeito (`..`, caminho absoluto, link) recusa o pacote inteiro, e a mensagem diz qual arquivo falhou. Se o identificador já existir, o caso entra como `<id>-importado`. A trilha registra o SHA-256 do pacote recebido.

### 3.2 Importação de extratos

A importação recebe arquivos `.xlsx` das operadoras TIM, VIVO e Claro. O sistema:
- detecta automaticamente o parser correto;
- identifica a operadora e o tipo (voz ou conexão);
- preserva o nome original, hash e metadados do arquivo;
- normaliza MSISDNs, IMEIs, IMSIs, CGIs, datas e coordenadas;
- detecta e propaga o MSISDN alvo quando possível;
- grava parquets canônicos (`canonical_calls.parquet`, `canonical_conns.parquet`, `canonical_erbs.parquet`);
- gera warnings de qualidade (números parciais, alvos ambíguos, CGIs inválidos, etc.).

### 3.3 Base mestre de ERBs

Módulo dedicado a manter a base de torres. Permite:
- importar planilhas de ERBs em `.xlsx` ou `.csv`;
- revisar o staging e comparar com a base atual (`diff`);
- fazer `upsert`, `delete`, `commit` e `rollback`;
- manter histórico versionado;
- pesquisar CGIs e estender a base com coordenadas de outros extratos;
- geolocalizar CGIs ausentes via Google Geolocation (quando configurado).

**Registro CGI → sítio.** Além do master, o sistema mantém um registro de cada CGI já visto (`data/erb_db/cgi_registry.parquet`), compartilhado entre os casos: um CGI aprendido num caso passa a resolver nos outros.
- **Precedência:** as origens seguem `manual > extract > geolocation > master_snap`. Uma origem de menor precedência não troca a coordenada, só preenche o que falta (estação, cidade, faixa).
- **Casamento com o master:** cada CGI é ligado à estação do master mais próxima, até 150 m, que traz número da estação, endereço e as faixas usadas no raio de cobertura.
- **Reaprender dos casos:** este botão relê os extratos de todos os casos. Use-o depois de atualizar o master; atualizar o master sozinho não reposiciona pontos no mapa.
- **Cobertura da base:** o painel de cobertura de cada caso mostra quanta evidência ficou fora do mapa e quais células a levaram.

### 3.4 Mapa tático

O mapa tático é o centro de visualização. Suporta dois motores de mapa: **Leaflet** (padrão, offline) e **Google Maps** (opcional, com API key).

#### 3.4.1 Camadas de visualização

- **Marcadores de eventos**: cada ponto representa uma chamada, conexão ou evento, com cor distinta para voz e conexão. No modo **agrupado** (padrão), um pin por ponto consolida os eventos e as ERBs dali; no modo **sem agregação**, cada evento é um marcador, até 18.000 na tela (o aviso diz quando o corte acontece; setores, calor e painéis seguem considerando todos).
- **Heatmap**: aglomerado de densidade de eventos.
- **Breadcrumb**: marcadores numerados do trajeto do alvo, em ordem cronológica.
- **Trajetória**: linha que liga os eventos do alvo no tempo.
- **Setores de azimute**: setor de abertura da antena. A abertura é estimada por estação (360° dividido pelo número de setores conhecidos, limitada a 30–120°) e vem do backend por CGI.
- **Cobertura estimada**: raio contínuo derivado da densidade real de antenas vizinhas na base mestre e da faixa licenciada — 700 MHz alcança bem mais que 3,5 GHz a partir da mesma torre. O slider da tela é um **multiplicador** (×0.5 a ×2) sobre esse raio, não um valor absoluto.
- **Melhor servidor (Voronoi)**: divide o mapa pela antena mais próxima de cada ponto. É a candidata mais plausível, não a cobertura medida — terreno, potência e azimute mandam mais. Disponível apenas no engine Leaflet.
- **POIs**: farol tático sobre o local de interesse — distintivo hexagonal com ícone do tipo (casa, maleta, alerta, escudo, cadeado, pessoas, pin), haste apontando a coordenada exata, halo pulsante e etiqueta com o nome, mais o círculo do raio configurado. O POI selecionado ganha anel âmbar e pulso mais rápido. O desenho é idêntico no Leaflet e no Google Maps.
- **Geofence / Corredor**: polígonos e rotas desenhadas manualmente.

#### 3.4.0 Painel de cobertura da base

Diz quanta evidência do caso está **fora** do mapa e por quê: quantos CGIs resolvem para uma antena com coordenada, quantos eventos ficaram sem plotar, e quais células pesam mais nessa perda. É o que transforma "o mapa parece vazio" em uma lista de CGIs para resolver em `/erbs`.

#### 3.4.5 Área provável de presença

Um evento sozinho diz "em algum lugar deste setor". Três eventos consecutivos em setores diferentes dizem "na interseção", que costuma ser muito menor — a diferença entre apontar um bairro e apontar uma quadra.

A ferramenta cruza as cunhas de setor de eventos próximos no tempo e devolve as janelas ordenadas da área menor para a maior. **Área pequena significa precisão alta.** Num caso real de referência, a janela mais precisa fechou em 0,036 km² — cerca de 3,6 hectares.

A interseção é calculada por amostragem numérica, não por geometria exata. Quando a faixa comum é mais fina do que a resolução da amostragem, a resposta marca `grid_limited` e a área devolvida é o **teto** que o método consegue afirmar, não uma medição.

#### 3.4.6 Direção de deslocamento

Rumo e velocidade entre estações consecutivas, com rosa dos ventos de 16 pontos e vetor dominante. O índice de consistência distingue "foi de A para B" de "ficou indo e voltando": perto de 1, os trechos apontam todos para o mesmo lado; perto de 0, se cancelam.

Trocas de setor dentro da mesma torre não contam como deslocamento — sem esse filtro a rosa dos ventos vira ruído em torno de zero.

#### 3.4.7 Mudança de base

Compara as estações dominantes antes e depois de um corte no tempo, separando em **surgidas**, **abandonadas** e **mantidas**. Serve para responder se o alvo mudou de rotina, de endereço ou de cidade a partir de uma data.

#### 3.4.2 Filtros

O painel tem duas camadas que **se somam**:

**Filtro principal** — apenas um modo ativo por vez:

- cidade;
- operadora;
- período: janela temporal (`from`, `to`) e/ou faixa do dia (manhã, tarde, noite, madrugada).

**Filtros independentes** — combinam com o modo ativo e entre si:

- extrato;
- alvo / MSISDN;
- **célula (CGI)**.

##### Filtro por CGI

Isola os eventos de uma ou mais células específicas. Serve para responder perguntas que o filtro por cidade não alcança: *"o que aconteceu nesta antena, e só nela?"* — útil quando várias ERBs se sobrepõem na mesma região e a cidade inteira é ruído.

Como funciona:

- a lista de opções vem do próprio caso, com **contagem de eventos** e endereço de cada célula, ordenada da mais usada para a menos usada;
- a busca aceita CGI, endereço ou cidade;
- a seleção é múltipla: dois ou mais CGIs devolvem a **união** dos eventos;
- células sem coordenada resolvida aparecem marcadas como tal — selecioná-las não plota nada, porque a base não sabe onde elas ficam;
- o filtro vale para markers, heatmap, timeline, trajetória, breadcrumb, viagem impossível e para os painéis espaciais (permanência, resumo de POI), de modo que nenhum painel discorde do que está desenhado;
- a lista de opções **não** encolhe conforme você seleciona — ela continua mostrando o caso inteiro, senão seria impossível ajustar a seleção.

A seleção fica na URL (`cgi=`), então sobrevive a recarregar a página e pode ser compartilhada com outro analista junto com o resto do recorte. O split-view herda esse mesmo filtro.

> **Nota de contabilidade**: eventos descartados pelo filtro não entram no aviso de "registros fora da plotagem". Aquele aviso é reservado a evidência que se perdeu por falta de ERB resolvida — misturar as duas coisas produziria alarme falso.

#### 3.4.3 Playback temporal

Controle de reprodução cumulativa: o usuário seleciona um intervalo e vê os eventos aparecendo no mapa progressivamente, como uma linha do tempo animada. O breadcrumb e a trajetória se atualizam juntos.

#### 3.4.4 Detecção de anomalias no mapa

O painel de anomalias do mapa identifica saltos de velocidade implausível entre eventos consecutivos. Se o Google Distance Matrix estiver configurado, o usuário pode verificar o tempo real de percurso por rota viável.

### 3.5 Marcadores analíticos (POIs)

POIs enriquecem a investigação com locais conhecidos. No mapa cada um aparece como um **farol tático**: o tipo escolhido no cadastro define a cor e o ícone do distintivo, e o nome fica escrito ao lado do marcador — dá para ler o mapa sem abrir popup nenhum.

##### Símbolo por tipo

Cada um dos 13 tipos ([1.5](#15-poi--point-of-interest)) tem símbolo e cor próprios, para o analista distinguir os pontos à distância sem clicar em nada:

| Tipo | Símbolo | Tipo | Símbolo |
|---|---|---|---|
| Local do crime | triângulo de alerta | Empresa / Trabalho | maleta |
| Ponto de tráfico | caixa/pacote | Loja / Comércio | vitrine com toldo |
| Encontro | duas pessoas | Escola / Faculdade | capelo |
| Casa / Residência | casa | Hospital / Saúde | cruz |
| Apartamento / Condomínio | prédio | Base operacional | escudo |
| Hotel / Motel | cama | Presídio | cadeado |
| Outros | pin genérico | | |

O **símbolo é o discriminador principal** e a cor é secundária: dois tipos podem ter matizes próximos, mas nunca o mesmo desenho. A legenda do mapa lista apenas os tipos que o caso realmente usa.

O mesmo desenho aparece nos dois engines (Leaflet e Google) e também no mapa HTML offline exportado, porque todos leem o mesmo catálogo.

Tipos cadastrados antes desta versão continuam válidos — nenhum POI precisa ser recadastrado.

O usuário pode:
- cadastrar por coordenadas ou endereço;
- usar autocomplete do Google Places para preenchimento automático;
- definir raio de abrangência;
- visualizar hits no raio (voz, conexão, total);
- ver dias fortes, horários fortes, ERBs/CGIs dominantes;
- obter sugestão assistida de **residência** e **trabalho** com base no padrão de permanência;
- persistir classificação e observações.

### 3.6 Painel Telefones

A tela `Telefones` é o ponto de entrada para tudo que envolve números. Ela reúne:

**Inventário pesquisável** — todos os números que aparecem no caso, não só os mais relevantes:
- busca por número, apelido, vínculo ou UF;
- ordenação por relevância, eventos, contatos, aparelhos, extratos, período ou apelido;
- paginação;
- recortes de triagem combináveis: alvos declarados, com mais de um aparelho, presentes em mais de um extrato, já identificados, ainda sem identificação, ou com DDD fora do Plano Nacional de Numeração.

**Caderno de identificação** — qualquer número pode receber apelido, vínculo (irmão, advogado, fornecedor), observação e cor. O apelido passa a aparecer nas tabelas, no grafo, no Gantt, nos popups do mapa e nos arquivos exportados.

**Qualidade dos dados**:
- score de confiança explicável para cada MSISDN/IMEI/IMSI;
- IMEIs compartilhados (mesmo aparelho, chips diferentes);
- DDDs e estados;
- contrapartes mais recorrentes;
- duplicatas entre extratos;
- alertas trazidos da importação (números parciais, alvos ambíguos, dados incompletos).

**Sinais do caso** — achados que valem para a investigação inteira, não para um número: trocas de aparelho, chips que passaram pelo mesmo aparelho, possíveis descartáveis e intermediários (quem está no caminho entre grupos que não se falam diretamente).

### 3.6.1 Dossiê do número

Em `/telefones/:msisdn`, a ficha completa de um número num único recorte temporal — o que importa quando o resultado vai para um relatório, porque todos os indicadores enxergam exatamente a mesma janela.

Reúne: contrapartes (com duração média e mediana, split SMS contra voz, primeiro e último contato), reciprocidade, aparelhos, células e as estações físicas por trás delas, calendário 7×24, picos de atividade, silêncios, redirecionamentos, trocas de aparelho e de chip, sinal de descartável, coincidências de local, pernoite, afastamento da base, deslocamentos improváveis, POIs e linha do tempo.

O campo `Torre` na tabela de células é o que impede o erro mais comum de leitura: três CGIs distintos podem ser três setores da mesma antena, ou seja, um lugar só e não três.

### 3.6.2 Comparar dois números

Em `/telefones/comparar`, o cruzamento entre dois alvos — diferente da correlação, que cruza *extratos*:
- contatos em comum, com o peso de cada lado;
- estações físicas que os dois usaram;
- coincidências de local dentro de uma janela de tempo (lugar em comum é fraco; lugar em comum na mesma janela é o indício);
- contato direto entre eles, com volume, duração e direção;
- similaridade de rotina, pelo cosseno entre os calendários 7×24. Mede o *formato* do horário e não o volume: dois números com a mesma rotina e volumes muito diferentes saem parecidos.

### 3.7 Análise de Voz

A tela `Voz` analisa chamadas do alvo. Funcionalidades:
- resumo de chamadas (total, dia, hora, duração, operadora);
- top contrapartes e chamadas recorrentes;
- análise de reciprocidade (A liga para B e B liga para A);
- análise de silêncios e bursts;
- detecção de troca de IMEI;
- cadeias de redirecionamento;
- Cruzamento com POIs e ERBs.

### 3.8 Análise de Conexão

A tela `Conexão` analisa sessões de dados:
- total de sessões, bytes e duração;
- top CGIs, IMEIs, IPs e operadoras;
- padrões de permanência e deslocamento;
- heatmap de uso e trajetória;
- cruzamento com POIs.

### 3.9 Análise Avançada (removida)

A tela `Avançado` **não existe mais**. Ela misturava dois tipos de coisa: indicadores de um número e indicadores do caso. Cada metade foi para onde faz sentido:

| O que era | Onde está agora |
|---|---|
| Colocalização, velocidade implausível, bursts, silêncios, roaming, dormitório, redirecionamentos, calendário | Dossiê do número (`/telefones/:msisdn`) |
| Troca de IMEI/chip, burner phones, intermediários | Sinais do caso (`/telefones`) |
| Stay points | Painel de permanência no mapa |

Links antigos para `/cases/:caseId/advanced` continuam funcionando: redirecionam para `/telefones`, ou direto para o dossiê quando a URL trazia `msisdn=`.

### 3.10 Gantt

A visualização `Gantt` mostra uma linha do tempo dos eventos, organizada por dia e tipo. Permite navegar cronologicamente e retornar ao mapa mantendo a mesma janela temporal selecionada.

### 3.11 Split-view

Permite comparar dois extratos lado a lado no mesmo enquadramento cartográfico. Útil para verificar o mesmo período de dois chips ou aparelhos, com pan e zoom sincronizados.

### 3.12 Grafo

A tela `Grafo` desenha a rede de relacionamentos do alvo:
- nós = MSISDNs;
- arestas = chamadas, mensagens ou eventos compartilhados;
- ego-grafo: foca em um número e mostra seus vizinhos até uma profundidade configurável.

### 3.13 Correlação

Cruza extratos de casos diferentes ou do mesmo caso para encontrar:
- MSISDNs em comum;
- IMEIs compartilhados;
- CGIs comuns;
- sobreposição temporal de eventos.

### 3.14 Exportações

O usuário pode exportar os dados e relatórios em vários formatos:

- `calls.csv` / `conns.csv`: registros canônicos de voz e conexão.
- `pois.csv`: resumo analítico dos POIs. Traz o tipo em duas colunas — `kind` (identificador técnico, estável) e `kind_label` (rótulo legível, ex.: "Hotel / Motel") —, o mesmo valendo para a sugestão automática (`suggested_kind` / `suggested_kind_label`).
- `all.xlsx`: planilha com abas canônicas e resumo de POIs.
- `markers.kml`: marcadores cartográficos para ferramentas externas (Google Earth, etc.).
- `report.pdf` / `report.html`: relatório completo com resumo, analytics, POIs e mapa estático opcional.
- `map.html`: mapa tático standalone interativo, com Leaflet embutido, marcadores, heatmap, trajetória, POIs (mesmo farol tático da aplicação) e playback — funciona offline sem backend.

- `case.zip`: o caso inteiro (manifesto, trilha, `raw` e parquet de cada extrato) com um `SHA256SUMS.txt`. Quem recebe confere com `sha256sum -c SHA256SUMS.txt`, sem precisar desta ferramenta, ou instala com *Importar caso (.zip)* (ver 3.1).

O `report.html` pode ser gerado com proteção por senha (WebCrypto), e o `map.html` é uma página única portátil para anexar em relatórios ou compartilhar. O `map.html` sai com o recorte que está na tela: filtros, alvo e extratos ligados.

**Integridade.** Toda exportação é registrada na trilha de auditoria (`data/audit/system.jsonl`) com o SHA-256 do arquivo gerado. O mesmo hash vem no cabeçalho `X-Content-SHA256` da resposta. A trilha é encadeada por HMAC e pode ser conferida com `python scripts\verify_audit.py` ou em `GET /api/v1/audit/verify`. O resultado é um de três:
- `ok`: a trilha está íntegra;
- `partial`: há um trecho inicial gravado antes de a trilha ser assinada, ou assinado com uma chave de admin anterior (use `--old-key` para conferi-lo);
- `broken`: há linha editada, removida, inserida ou fora de ordem.

### 3.15 Análise GPS (módulo independente)

Para relatórios de rastreador veicular, com casos próprios em `data/gps_cases`, separados dos extratos de operadora:
- **Importação:** aceita `.xls`, `.xlsx` e `.csv` com as colunas essenciais data, latitude e longitude. Arquivos grandes são processados em segundo plano, e a placa e o ID do dispositivo são tirados do título do relatório quando existem.
- **Mapa:** em Leaflet ou Google, mostra pontos, trajeto, playback, calor e permanências, com filtros por período, velocidade mínima, ignição e track.
- **POIs:** têm raio e o mesmo farol do mapa de ERBs, e cada um traz o resumo de presença: hits, dias e janela de horário.
- **Detecções:** permanências e saltos de velocidade implausível.
- **Exportações:** `positions.csv`, `poi_summary.csv`, `track.kml`, `all.xlsx` e `report.html`.

---

## 4. Tipos de mapas e engines

### 4.1 Leaflet

- Engine padrão e offline.
- Camadas: OpenStreetMap padrão, OpenStreetMap Humanitarian, Esri World Imagery (satélite) e CartoDB Dark Matter.
- Tema escuro e claro.
- Funciona sem API key e sem conexão após carregamento inicial.

### 4.2 Google Maps

- Engine opcional, ativada via `API Key` e `Map ID` na configuração.
- Modos: `roadmap`, `satellite`, `hybrid`, `terrain`.
- Habilita recursos extras: Places autocomplete, Geocoding, Geolocation, Maps Static e Distance Matrix.
- Se a SDK não carregar, o sistema faz fallback automático para Leaflet.

---

## 5. Configuração e operação

### 5.1 Configuração do Google Maps

Através da interface, o usuário informa `API Key` e `Map ID` do Google Cloud. A configuração é salva em `data/config/app_settings.json` e não exige rebuild do executável.

### 5.2 Modo protegido

A ferramenta suporta modo protegido via `ERBS_ADMIN_API_KEY`. Quando ativado, operações administrativas (criar, excluir, restaurar e importar casos; alterar base ERB; importar dados) exigem a chave no header. Eventos de acesso e negação são registrados em `data/audit/system.jsonl`.

No painel `Acesso admin`, a chave é instalada uma única vez por navegador e permanece válida nas sessões seguintes — não é preciso digitá-la a cada abertura do aplicativo. Ela só deixa de valer quando removida no botão `Remover chave` do próprio painel (ou quando os dados do site são limpos), e nesse caso precisa ser instalada novamente.

### 5.3 Cache de APIs

As respostas pagas do Google são cacheadas em disco para reduzir custos e repetir consultas. O tempo de vida do cache é configurável (padrão 30 dias).

---

## 6. Para que serve cada análise na prática

| Análise | Quando usar | O que responde |
|---|---|---|
| **Mapa tático** | Visualização geral e apresentação | Onde o alvo esteve, quando e com quem. |
| **Heatmap** | Identificar áreas de concentração | Onde o alvo passou mais tempo. |
| **Breadcrumb** | Reconstruir trajeto | Qual foi a sequência de locais no tempo. |
| **Geofence** | Verificar presença em endereço | O alvo entrou nessa área? Quando e por quanto tempo? |
| **Corredor** | Confirmar rota | O alvo passou por essa via? |
| **Stay points** | Descobrir endereço de moradia/trabalho | Onde o alvo parou por muito tempo? |
| **Colocalização** | Encontrar companheiros/cúmplices | Quem estava na mesma torre no mesmo horário? |
| **Velocidade implausível** | Validar dados | Existe deslocamento fisicamente impossível? |
| **Burner phones** | Detectar números descartáveis | Existe aparelho usado por poucos dias e poucos contatos? |
| **Bursts** | Detectar momentos de crise | Houve pico de comunicação em curto prazo? |
| **Silêncios** | Identificar desligamento/mudança | Quando o aparelho parou de eventar? |
| **Troca de IMEI/Chip** | Rastrear dispositivo e SIM | O mesmo chip mudou de aparelho? O mesmo aparelho mudou de chip? |
| **Chamadas redirecionadas** | Entender ocultação | A ligação foi encaminhada para onde? |
| **Correlação** | Cruzar extratos | Dois alvos se encontram? Compartilham aparelho ou torre? |
| **Grafo** | Visualizar rede | Quem é o elo mais forte do alvo? |
| **Resumo de POI** | Medir proximidade com locais de interesse | Quantos eventos ocorreram perto da casa do suspeito? |
| **Filtro por CGI** | Isolar uma antena específica | O que aconteceu nesta célula, e só nela? Em que horários ela foi usada? |

---

## 7. Considerações de uso e limitações

1. **A ferramenta é local**: os dados são processados e armazenados na máquina do usuário. Não há envio obrigatório para a nuvem, exceto quando APIs do Google estão ativadas.
2. **Google Maps é opcional**: sem API key, o Leaflet atende todo o mapa e a maioria das análises espaciais.
3. **Precisão de localização**: a posição estimada depende da qualidade da base ERB e das coordenadas do extrato. O azimute e a distância real podem variar.
4. **Velocidade implausível**: o limite é configurável. Resultados devem ser verificados com contexto (transporte, erro de CGI, etc.).
5. **Colocalização**: presença na mesma célula não prova proximidade física — células rurais podem ter quilômetros de alcance. Use como indício.
6. **Burner phone**: as regras são heurísticas. Devem ser combinadas com outros indícios.
7. **Filtro por CGI**: filtrar por uma célula não é o mesmo que filtrar por um endereço. O CGI diz qual antena atendeu o aparelho, e o alcance dessa antena pode ser de centenas de metros em área urbana ou de quilômetros em área rural. Além disso, boa parte das células de um caso costuma não ter coordenada resolvida na base — elas aparecem na lista, mas não plotam.
8. **Tipo de POI**: a classificação é do analista, não do sistema. As sugestões automáticas cobrem apenas residência e trabalho; os demais tipos são atribuídos manualmente e valem como anotação, não como conclusão.

---

## 8. Resumo

O ERBs Analyzer v2 oferece um ambiente completo para transformar extratos telefônicos em informações investigativas. Do upload à exportação, passando por mapa tático, análises comportamentais, alertas e relatórios, cada funcionalidade foi pensada para ajudar a responder perguntas como **onde**, **quando**, **com quem** e **como** o alvo se comunicou.
