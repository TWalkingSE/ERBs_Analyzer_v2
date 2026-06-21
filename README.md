# Guia do Usuário do Executável

## 1. O que é este pacote

O `ERBs Analyzer` é uma aplicação local para análise técnico-investigativa de extratos telefônicos de telefonia móvel. O executável abre um servidor local na própria máquina e entrega a interface no navegador padrão.

O processamento principal é local: casos, extratos importados, análises, correlações, grafos, exportações e configurações ficam na própria estação de trabalho.

O pacote distribuído normalmente contém:

- `erbs_analyzer_server.exe`: executável principal.
- `_internal`: bibliotecas e arquivos internos do aplicativo.
- `data`: pasta de dados locais da instalação.
- `erbs_analyzer.ico`: ícone distribuído com o pacote.
- `GUIA_USUARIO_EXECUTAVEL.md`: este guia completo.
- `LEIA-ME_TESTADORES.txt`: instruções rápidas para validação e suporte.

Não mova o `.exe` para fora da pasta do pacote e não execute o aplicativo de dentro de um `.zip`.

<img width="859" height="545" alt="Screenshot 2026-06-21 130246" src="https://github.com/user-attachments/assets/6d662bee-0596-4f0f-b047-69e44a9bceda" />

<img width="1600" height="808" alt="WhatsApp Image 2026-06-21 at 13 03 50" src="https://github.com/user-attachments/assets/e3965dc5-d541-485c-855c-98ca78650bb1" />



## 2. Como o executável funciona

Ao abrir `erbs_analyzer_server.exe`, o sistema:

- sobe um servidor local em `127.0.0.1`;
- carrega os dados da pasta `data`;
- serve a interface web já embutida no pacote;
- abre o navegador automaticamente quando a aplicação estiver pronta.

Se o navegador não abrir sozinho, use manualmente a URL exibida na janela do executável, normalmente `http://127.0.0.1:8765`.

## 3. Onde os dados ficam

Por padrão, a instalação usa a pasta `data` do próprio pacote.

Locais mais importantes:

- `data\cases`: casos, extratos importados, auditoria do caso e exportações.
- `data\erb_db`: base mestre de ERBs usada pelos mapas e cruzamentos.
- `data\config\app_settings.json`: configurações locais da instalação, inclusive Google Maps em runtime.
- `data\audit\system.jsonl`: trilha de auditoria sistêmica de eventos administrativos e operacionais.

Se você receber uma nova versão do executável, preserve a pasta `data` caso queira manter casos, base ERB e configurações já existentes.

## 4. O que funciona online e offline

### 4.1. O que funciona localmente, mesmo sem internet

As operações abaixo continuam locais e independem de um servidor remoto da aplicação:

- criação e gestão de casos;
- importação de extratos `.xlsx`;
- normalização e persistência de dados canônicos;
- navegação entre módulos analíticos;
- análises de Voz, Conexão, Avançado, Grafo, Correlação e Gantt;
- exportações CSV, XLSX, KML, PDF e HTML;
- leitura de POIs cadastrados manualmente por coordenada.

### 4.2. O que depende de internet

Alguns recursos cartográficos e de geocodificação usam serviços externos:

- `Leaflet` usa basemaps online, como OpenStreetMap, Esri e CARTO.
- `Google Maps` carrega a SDK e o mapa-base do Google.
- geocodificação por endereço usa a API Google quando a chave está configurada.

Sem internet, o sistema pode continuar abrindo e processando os dados locais, mas o mapa-base pode ficar incompleto ou em branco e a geocodificação por endereço pode falhar.

### 4.3. Leitura prática do modo offline

- o caso, os extratos e os painéis analíticos continuam locais;
- o mapa pode continuar útil para overlays, filtros e contexto interno, mas pode perder o pano de fundo cartográfico se os tiles não carregarem;
- o Google Maps exige internet, billing e chave válida;
- POIs por coordenada manual continuam sendo uma alternativa segura quando a geocodificação não estiver disponível.

## 5. Primeiro uso

### 5.1. Como abrir

1. Extraia o pacote inteiro para uma pasta local do Windows.
2. Mantenha juntos: `erbs_analyzer_server.exe`, `_internal`, `data`, `GUIA_USUARIO_EXECUTAVEL.md` e `LEIA-ME_TESTADORES.txt`.
3. Execute `erbs_analyzer_server.exe`.
4. Aguarde a interface abrir no navegador.

### 5.2. Fluxo operacional recomendado

O uso padrão da ferramenta normalmente segue esta ordem:

1. criar ou abrir um caso;
2. importar um ou mais extratos;
3. validar o caso no `Mapa ERBs`;
4. aprofundar em `Telefones`, `Voz`, `Conexão`, `Avançado`, `Gantt`, `Grafo` e `Correlação`;
5. gerar exportações quando a análise estiver madura.

## 6. Módulos e funções do sistema

## 6.1. Visão geral dos módulos

| Módulo | O que entrega | Quando usar |
| --- | --- | --- |
| Dashboard de casos | Lista, cria e organiza investigações locais. | Sempre que iniciar ou retomar um caso. |
| Painel do caso | Resume o caso, os extratos importados e os atalhos operacionais. | Como centro de navegação da investigação. |
| Importação | Recebe extratos `.xlsx`, detecta parser, operadora e tipo do registro. | Para ingestão de novos arquivos. |
| Mapa ERBs | Exibe eventos, heatmap, setores, cobertura, playback, breadcrumb, trajeto, geofence, corredor, permanência e POIs. | Leitura espacial principal do caso. |
| Split-view | Compara dois extratos com enquadramento sincronizado. | Para confrontar fontes, períodos ou operadoras. |
| Gantt | Organiza a linha do tempo por dia e horário. | Para leitura cronológica precisa. |
| Telefones | Consolida qualidade de MSISDNs, DDDs, alertas e vínculos com IMEI. | Para saneamento e leitura de consistência dos números. |
| Voz | Analisa chamadas, contrapartes, ERBs, reciprocidade e sinais comportamentais. | Para relação social e dinâmica de chamadas. |
| Conexão | Analisa sessões de dados, bytes, duração, IMEIs, IPs, CGIs e ERBs. | Para rastro técnico do uso de dados. |
| Avançado | Reúne indicadores forenses como bursts, silêncios, burner, co-localização e roaming. | Para leitura de padrão, anomalia e risco. |
| Grafo | Visualiza a rede de relacionamentos do caso. | Para cadeia de contato e clusters. |
| Correlação | Cruza extratos por MSISDN, IMEI, CGI e overlap temporal. | Para confirmar convergência entre fontes. |
| Exportações | Gera saídas CSV, XLSX, KML, PDF e HTML. | Para consolidação, auditoria e entrega. |
| Base ERBs | Administra a base mestre de estações. | Para manutenção da infraestrutura cartográfica do sistema. |

## 6.2. Dashboard de casos

No dashboard você consegue:

- criar novos casos;
- abrir casos existentes;
- revisar rapidamente o inventário local de investigações.

## 6.3. Painel do caso

O painel do caso funciona como hub operacional. Ele mostra:

- nome e descrição do caso;
- datas de criação e atualização;
- lista de extratos já importados;
- operadora, tipo do registro, quantidade de linhas e alvo inferido por extrato;
- warnings de importação;
- atalhos para todos os módulos do caso;
- remoção de extratos quando necessário.

## 6.4. Importação de extratos

### O que a tela faz

A tela `Importação` envia o arquivo ao backend e tenta identificar automaticamente:

- a operadora;
- o tipo do extrato, como `VOZ` ou `CONEXÃO`;
- o parser mais adequado;
- o MSISDN alvo, quando a estrutura permitir.

### Formato aceito

O fluxo principal da ferramenta usa arquivos `.xlsx`. Sempre prefira as planilhas originais exportadas pela fonte oficial.

### Resultado esperado

Ao concluir a importação, o sistema grava na pasta do caso:

- o arquivo bruto `raw.xlsx`;
- os parquet canônicos de chamadas, conexões e ERBs;
- os metadados do extrato no manifesto do caso.

## 6.5. Mapa ERBs

O `Mapa ERBs` é a principal tela de leitura espacial e temporal. Ele permite:

- alternar entre `Leaflet` e `Google Maps`;
- alternar entre modo `Agrupado` e `Sem agregação`;
- visualizar marcadores táticos, heatmap, setores de azimute, cobertura estimada, playback, breadcrumb e trajeto;
- filtrar por extrato, MSISDN, operadora, cidade, período e janela temporal;
- ajustar o raio e a abertura do setor;
- desenhar geofence e corredor;
- consultar permanência por raio e duração;
- destacar POIs e focar no contexto de um marcador analítico.

### Recursos táticos principais

- `Heatmap`: mostra concentração espacial.
- `Breadcrumb`: mostra a sequência numerada do alvo.
- `Trajeto`: liga os pontos e resume o deslocamento.
- `Playback`: reproduz a dinâmica temporal do caso.
- `Setores`: desenha o azimute estimado das ERBs.
- `Cobertura`: mostra círculos de alcance estimado.
- `Geofence e corredor`: delimitam área e eixo de passagem.
- `Permanência`: resume pontos com maior tempo de retenção.

## 6.6. POIs

Os POIs podem ser criados no caso e usados no mapa para leitura contextual. O sistema permite:

- cadastrar POIs por coordenada manual;
- cadastrar POIs por endereço manual;
- geocodificar endereço quando o Google estiver configurado;
- definir raio, tipo e observações;
- destacar hits de voz e conexão no raio do POI;
- resumir ERBs dominantes, dias fortes e horários fortes;
- persistir classificação e sugestão assistida de residência ou trabalho.

## 6.7. Split-view sincronizado

O `Split-view` coloca dois extratos lado a lado com pan e zoom sincronizados. Ele é útil para:

- comparar duas operadoras;
- comparar períodos diferentes;
- validar convergência ou divergência espacial;
- confrontar extratos mais limpos com extratos mais ruidosos.

## 6.8. Gantt

O `Gantt do alvo` organiza os eventos em uma linha do tempo por dia e horário. Use quando precisar:

- localizar janelas de atividade intensa;
- visualizar a ordem cronológica dos eventos;
- comparar tempo e espaço em conjunto com o mapa;
- voltar ao mapa mantendo o mesmo contexto temporal.

## 6.9. Telefones

O painel `Telefones` consolida a qualidade dos identificadores do caso. Ele ajuda a revisar:

- MSISDNs válidos e problemáticos;
- DDDs faltantes ou inconsistentes;
- alvos inferidos;
- alertas de importação;
- IMEIs compartilhados;
- relações entre números e aparelhos.

## 6.10. Voz

O módulo `Voz` foca no comportamento de chamadas. Ele destaca, entre outros:

- contrapartes recorrentes;
- reciprocidade;
- hit de ERBs;
- swaps de IMEI e de chip;
- silêncios;
- bursts;
- co-localização;
- comportamento temporal e espacial do alvo.

## 6.11. Conexão

O módulo `Conexão` foca nas sessões de dados. Ele ajuda a responder perguntas como:

- quantas sessões ocorreram;
- quais IMEIs apareceram;
- quais IPs foram usados;
- quais CGIs estiveram presentes;
- qual foi o volume de bytes e a duração das sessões;
- em que janelas ocorreu o pico de atividade de dados.

## 6.12. Avançado

O módulo `Avançado` concentra indicadores forenses comportamentais, como:

- bursts;
- silêncios;
- burner indicators;
- co-localização;
- roaming;
- swaps de IMEI;
- swaps de chip;
- velocidade impossível;
- padrões auxiliares de risco e anomalia.

## 6.13. Grafo

O `Grafo` exibe a rede de relacionamentos do caso. Você pode trabalhar com:

- grafo completo;
- ego-grafo por MSISDN focal;
- profundidade de hops.

É especialmente útil para identificar conectores, intermediários e clusters de relacionamento.

## 6.14. Correlação

O módulo `Correlação` cruza extratos em busca de elementos compartilhados, como:

- MSISDNs em comum;
- IMEIs em comum;
- CGIs em comum;
- sobreposição temporal compatível.

Use esta tela quando houver mais de um extrato relevante e você precisar provar convergência entre fontes diferentes.

## 6.15. Exportações

O módulo `Exportações` gera saídas para auditoria, compartilhamento ou entrega final. O pacote atual pode gerar:

- `calls.csv`;
- `conns.csv`;
- `pois.csv`;
- `all.xlsx`;
- `markers.kml`;
- `report.pdf`;
- `report.html`.

O `report.html` pode ser protegido por senha. Alguns formatos usam `MSISDN alvo` para enriquecer a análise.

## 6.16. Base ERBs

O módulo `Base ERBs` é usado para administrar a base mestre de estações. Ele permite:

- buscar por operadora, UF, cidade, bairro e estação;
- editar registros;
- criar registros manuais;
- importar atualização por planilha;
- revisar staging e diff;
- executar commit e rollback de versões;
- consultar histórico da base.

É uma área sensível. Só altere essa base com critério documental.

## 7. Como habilitar o Google Maps na instalação

O Google Maps é opcional. O aplicativo funciona integralmente com `Leaflet`, mas você pode habilitar o Google para usar a engine alternativa e, se desejar, geocodificação por endereço.

## 7.1. O que você precisa no Google Cloud

Antes de salvar a chave no aplicativo, prepare o projeto no Google Cloud:

1. crie ou escolha um projeto;
2. ative o billing do projeto;
3. habilite a `Maps JavaScript API`;
4. se quiser geocodificação por endereço dentro da ferramenta, habilite também a `Geocoding API`;
5. crie uma `API Key`;
6. opcionalmente, crie um `Map ID` se quiser estilo personalizado;
7. restrinja a chave por API e, depois de validar o funcionamento, restrinja também o uso conforme a política da sua organização.

### Recomendação prática de segurança

- habilite somente as APIs realmente necessárias;
- teste primeiro com restrições mínimas e endureça depois;
- não distribua a pasta do executável com uma chave real sem antes restringi-la;
- trate `data\config\app_settings.json` como arquivo sensível quando houver uma chave Google válida gravada.

## 7.2. Como salvar a chave no aplicativo

1. Abra um caso.
2. Entre em `Mapa ERBs`.
3. Abra a seção `Engine e basemap`.
4. Clique em `Configurar Google`.
5. Preencha:
   - `API Key`: obrigatória.
   - `Map ID`: opcional.
6. Salve a configuração.

Resultado esperado:

- a instalação passa a reconhecer o Google Maps;
- a engine Google fica disponível para uso no mapa;
- a configuração é reaproveitada nessa mesma instalação;
- não é necessário rebuild do executável.

## 7.3. Onde a configuração é gravada

A configuração runtime do Google Maps é salva em:

`data\config\app_settings.json`

Isso significa que:

- a chave é local daquela instalação;
- navegadores diferentes na mesma máquina continuam usando a configuração salva;
- se você copiar a pasta inteira com `data`, leva junto essa configuração.

## 7.4. Como remover a configuração do Google

Na mesma área `Engine e basemap`, use a opção de limpar ou remover a configuração do Google Maps.

Quando isso acontece:

- a chave local é apagada;
- o sistema volta para `Leaflet`;
- o restante da ferramenta continua funcionando normalmente.

## 7.5. Erros comuns do Google Maps

Se o Google não carregar, verifique:

- billing ativo no Google Cloud;
- `Maps JavaScript API` habilitada;
- `Geocoding API` habilitada, se você for usar busca por endereço;
- chave correta;
- restrições compatíveis com o seu ambiente local;
- conectividade da máquina;
- `Map ID` correto, se você informou um.

Mensagens de falha normalmente estão ligadas a:

- chave ausente ou inválida;
- billing desativado;
- restrição de chave incompatível;
- falta de conectividade;
- falha ao carregar a SDK do Google Maps.

## 8. Segurança e boas práticas

### 8.1. Segurança local

O executável foi desenhado para uso local. Por padrão:

- o servidor sobe em `127.0.0.1`;
- os dados ficam na pasta `data` da própria instalação;
- a aplicação não depende de um backend remoto do produto;
- a trilha de auditoria fica em arquivo local.

### 8.2. O que proteger

Proteja especialmente:

- `data\cases`, porque contém extratos e resultados da análise;
- `data\config\app_settings.json`, se houver chave Google Maps válida;
- `data\audit\system.jsonl`, porque registra eventos operacionais;
- exportações geradas para compartilhamento externo.

### 8.3. Modo protegido opcional

Se a instalação for configurada com chave administrativa, algumas operações sensíveis podem exigir credencial adicional. Nessa situação, o frontend oferece o painel `Acesso admin` para guardar a credencial apenas na sessão atual do navegador.

### 8.4. Boas práticas de uso

- mantenha o pacote em pasta local confiável;
- não execute o aplicativo de dentro do `.zip`;
- não compartilhe a pasta `data` sem revisar o conteúdo;
- revise filtros e alvo antes de exportar evidências;
- remova ou restrinja a chave Google antes de distribuir a instalação a terceiros;
- preserve a pasta `data` quando atualizar o pacote para nova versão.

## 9. Problemas comuns

### 9.1. O navegador abriu, mas a tela ficou em branco

Faça esta sequência:

1. feche o navegador;
2. feche o executável;
3. abra o executável novamente;
4. confirme que `erbs_analyzer_server.exe`, `_internal` e `data` continuam juntos;
5. tente abrir manualmente a URL mostrada na janela do executável.

### 9.2. O mapa não mostra pontos

Verifique:

- se existem extratos ativos no caso;
- se há filtro excessivo ligado;
- se os eventos possuem ERB válida;
- se muitos registros foram ocultados por CGI insuficiente ou coordenada inválida.

### 9.3. O Google Maps não habilita

Verifique:

- se a chave foi salva corretamente;
- se o billing está ativo;
- se a API foi habilitada no Google Cloud;
- se a internet da máquina está funcionando;
- se a chave está restrita de forma compatível com o uso local.

### 9.4. O executável abre, mas a interface não carrega

- aguarde alguns segundos;
- veja a URL mostrada na janela do executável;
- tente abrir essa URL manualmente;
- confira se algum antivírus ou política local está bloqueando a execução;
- verifique se a porta local está ocupada por outro processo.

### 9.5. A geocodificação por endereço falha

Na maior parte dos casos, o problema é um destes:

- Google Maps não configurado na instalação;
- `Geocoding API` não habilitada no projeto do Google;
- falta de internet;
- erro de quota, billing ou restrição da chave.

Se necessário, use coordenadas manuais para não interromper o fluxo investigativo.

## 10. O que informar ao suporte ou ao time de testes

Se precisar relatar erro, envie:

- o que estava fazendo;
- nome do módulo aberto;
- horário aproximado;
- print da tela;
- mensagem exibida no navegador ou na janela do executável;
- se o Google Maps estava ativo ou não;
- se havia filtro temporal, de cidade ou de operadora ligado;
- se o problema aconteceu com um extrato específico.

## 11. Resumo final

O `ERBs Analyzer` foi desenhado para que o usuário do executável consiga:

- criar casos;
- importar extratos;
- validar o contexto espacial no mapa;
- comparar extratos no split-view;
- ler a linha do tempo no Gantt;
- investigar qualidade de números em `Telefones`;
- aprofundar em `Voz`, `Conexão` e `Avançado`;
- mapear relacionamentos no `Grafo`;
- cruzar fontes em `Correlação`;
- gerar evidências em `Exportações`;
- opcionalmente habilitar o Google Maps sem rebuild do executável.

Se estiver em dúvida, siga esta sequência curta:

1. abrir ou criar um caso;
2. importar um extrato `.xlsx`;
3. validar o caso no mapa;
4. aprofundar em `Telefones`, `Gantt`, `Voz` e `Avançado`;
5. cruzar em `Grafo` e `Correlação`;
6. exportar quando os filtros e o alvo estiverem estáveis.
