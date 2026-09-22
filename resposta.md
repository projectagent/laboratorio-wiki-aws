# 📝 Resposta do Laboratório: A Wiki Perdida dos Arquivos Corporativos

> Preencha este arquivo com a sua proposta de solução.
>
> Sua resposta deve explicar como transformar os documentos brutos da pasta `raw/` em uma Wiki Corporativa Inteligente, pesquisável e segura usando apenas serviços da AWS.

---

## 👤 Identificação

**Nome:**  
Arthur Carlos Oliveira Rossini

**Data:**  
22/09/2026

**Link do repositório:**  
[Repositório](https://github.com/projectagent/laboratorio-wiki-aws)

---

# ✅ Quest 1: O Mapa dos Arquivos Perdidos

## 1.1 Formatos encontrados na pasta `raw/`

Descreva quais tipos de arquivos existem dentro da pasta `raw/`.
> Abra a pasta e liste o que voce encontrou de fato. Esta quest avalia a sua
> leitura do acervo, entao a resposta certa e a que corresponde aos arquivos.

**Sua resposta:**
Foram identificados três tipos distintos de arquivos, exigindo abordagens de extração diferentes:

ata_reuniao_vendas_sa.pdf: PDF digital nativo (5 páginas) com camada de texto (não requer OCR).

ata_resultados_vendas_novos_dados.png/.jpg: Imagem digitalizada (1 página). Composto por pixels, requer OCR.

vendas_sa_dados_ficticios_laboratorio.csv: Arquivo estruturado tabular (240 linhas). Não possui texto corrido.

---

## 1.2 Principais desafios encontrados

Explique quais dificuldades esses documentos podem apresentar.

**Sua resposta:**
Na imagem: Requer reconhecimento ótico de caracteres (OCR) avançado capaz de lidar com caligrafia (anotações soltas, datas circuladas) e contornar artefatos visuais como carimbos sobrepostos.

No PDF digital: O desafio é preservar a coerência espacial (layout) para não misturar colunas de tabelas (ex: responsabilidades e prazos) ao realizar a leitura do texto linha por linha.

No CSV: O processamento como texto corrido destruiria o contexto dos dados. É necessário transformar as linhas em dicionários/documentos estruturados antes da indexação para evitar excesso de tokens e ruído na busca.

---

## 1.3 Informações importantes a serem extraídas

Liste quais informações precisam ser identificadas para transformar os documentos em conhecimento pesquisável.

**Sua resposta:**
Para garantir uma base pesquisável útil, é fundamental extrair:

Datas e horários (reuniões, prazos).

Participantes (nomes, departamentos, cargos).

Temas discutidos e pauta.

Indicadores financeiros e de negócio mencionados.

Decisões tomadas (status e descrição).

Responsáveis diretos e prazos atrelados às ações.

Riscos mapeados e pendências identificadas.

---

## 1.4 Estratégia de classificação inicial

Como você classificaria os documentos sem depender de subpastas dentro de `raw/`?

**Sua resposta:**
Sem depender de subpastas, a classificação será flat, baseada na inferência de metadados. Um LLM inicial fará a triagem do cabeçalho/conteúdo do arquivo para classificar o tipo de documento (ex: Ata de Reunião, Planilha Financeira). Em seguida, essas categorias serão aplicadas diretamente aos arquivos no Amazon S3 utilizando S3 Object Tags (ex: Type=Meeting_Minutes), permitindo filtragem nativa no próprio bucket.
---

# ✅ Quest 2: O Portal de Entrada na AWS

## 2.1 Armazenamento dos arquivos brutos

Explique como os arquivos da pasta `raw/` seriam enviados e armazenados na AWS.

Serviços que você pode considerar:

- Amazon S3
- AWS IAM
- AWS KMS
- Amazon S3 Versioning
- Amazon S3 Lifecycle

**Sua resposta:**
Os arquivos são depositados em um bucket inicial do Amazon S3 (wiki-corp-raw-data). O upload dispara um evento de processamento. O acesso ao bucket é restrito por políticas estritas do AWS IAM (mínimo privilégio), e todos os dados em repouso são encriptados utilizando o AWS KMS para garantir a segurança de informações corporativas e financeiras sensíveis.

---

## 2.2 Preservação dos arquivos originais

Explique como garantir que os arquivos originais sejam mantidos intactos e rastreáveis.

**Sua resposta:**
Para garantir que a pasta raw/ seja a fonte da verdade imutável, o bucket no Amazon S3 terá o S3 Versioning habilitado e regras de S3 Object Lock (modo WORM - Write Once Read Many). Nenhuma ferramenta ou script terá permissão para alterar ou deletar os arquivos nesta etapa. Todos os outputs de processamento serão salvos em um bucket separado.
---

## 2.3 Extração de texto dos documentos

Explique como cada tipo de arquivo seria processado.

Considere:

- PDFs escaneados;
- Imagens;
- PDFs digitais;
- Arquivos `.txt`;
- Arquivos `.docx`;
- Arquivos `.md`.

Serviços que você pode considerar:

- Amazon Textract
- AWS Lambda
- AWS Step Functions
- Amazon S3
- Amazon CloudWatch

**Sua resposta:**
A extração é orquestrada pelo AWS Step Functions, que utiliza uma função AWS Lambda para identificar a extensão e a camada de texto, roteando o arquivo:

Imagens (e PDFs escaneados): Direcionados à API do Amazon Textract (AnalyzeDocument), que extrai texto, caligrafia solta e estrutura de formulários/tabelas.

PDFs digitais / .docx / .md / .txt: Processados por uma AWS Lambda com bibliotecas nativas de parsing (ex: pdfplumber), extraindo texto e estruturas com alta fidelidade sem o custo de chamadas OCR.

CSVs: Uma AWS Lambda (usando Pandas) transforma cada linha em um micro-documento JSON em formato chave-valor.
Ao fim, todo o texto é salvo no bucket wiki-corp-processed-data.

---

## 2.4 Tratamento de falhas

Explique como sua solução identificaria e registraria erros de processamento.

**Sua resposta:**
O AWS Step Functions implementa blocos nativos de Catch e Retry. Se o Textract falhar ou um Lambda sofrer timeout, o estado falho é interceptado. O erro é detalhado e logado no Amazon CloudWatch Logs. Filtros de métrica no CloudWatch monitoram termos como Exception e disparam alarmes para um tópico do Amazon SNS, notificando imediatamente a equipe de engenharia via e-mail sobre o arquivo não processado.

---

# ✅ Quest 3: A Relíquia dos Metadados

## 3.1 Padronização dos textos processados

Explique como os textos extraídos seriam limpos, normalizados e preparados para consulta.

**Sua resposta:**
Os textos que chegam no bucket processed-data passam por um script Python em uma AWS Lambda que utiliza Expressões Regulares (Regex). Essa etapa remove cabeçalhos/rodapés repetitivos, limpa múltiplos espaços em branco, consolida quebras de linha defeituosas e transforma tudo em um formato JSON limpo, otimizando o consumo de tokens na IA e aumentando a precisão da busca.

---

## 3.2 Metadados propostos

Defina quais metadados você extrairia de cada documento.

| Metadado | Por que ele é importante? |
|---|---|
| Nome do documento | Identificação básica e auditoria. |
| Tipo do documento | Permite filtrar buscas (ex: pesquisar apenas em Atas). |
| Data identificada | Crucial para ordenação cronológica e compreensão de contexto temporal. |
| Tema principal | Facilita a sumarização e a busca por agrupamento de assuntos. |
| Participantes | Permite mapear a presença e autoria de decisões a colaboradores específicos. |
| Decisões tomadas | Transforma texto passivo em conhecimento acionável corporativo. |
| Responsáveis | Permite buscas do tipo "Quais são as tarefas da Maria?". |
| Próximos passos | Rastreabilidade do andamento de projetos entre diferentes reuniões. |
| Nível de confidencialidade | Base para a futura implementação de regras de acesso (RBAC) |
| Caminho do arquivo original | Essencial para citar a fonte da verdade na resposta gerada pela IA. |

Adicione outros metadados, se necessário.

---

## 3.3 Uso de IA para enriquecimento dos documentos

Explique como o Amazon Bedrock poderia ajudar a identificar temas, decisões, responsáveis, pendências e resumos dos documentos.

**Sua resposta:**
Uma vez que o texto está limpo, ele é enviado ao Amazon Bedrock (utilizando modelos rápidos como o Claude 3 Haiku). A IA não recebe um prompt aberto, mas sim um System Prompt de extração restrita em formato JSON. Ela analisa o documento e infere quem são os responsáveis, resume os tópicos discutidos e isola blocos de ação, devolvendo um pacote de metadados estruturado.

---

## 3.4 Armazenamento dos metadados

Explique onde os metadados seriam armazenados e como seriam conectados aos documentos originais.

Serviços que você pode considerar:

- Amazon S3
- Amazon DynamoDB
- AWS Glue Data Catalog
- Amazon Bedrock Knowledge Bases

**Sua resposta:**
Cada documento processado ganha um UUID exclusivo. Os metadados extraídos pela IA são gravados em uma tabela NoSQL no Amazon DynamoDB, usando o UUID como Chave Primária (Partition Key). Nesta mesma tabela, gravamos o campo Original_S3_URI apontando para o PDF/Imagem original no bucket raw. Isso conecta permanentemente o dado processado e seus metadados ao arquivo raiz, podendo ser catalogado via AWS Glue Data Catalog.

---

# ✅ Quest 4: O Oráculo da Wiki Inteligente

## 4.1 Estratégia de indexação

Explique como os documentos seriam divididos em trechos menores e preparados para busca semântica.

**Sua resposta:**
Os textos limpos não podem ser injetados integralmente na IA durante a busca. O Amazon Bedrock Knowledge Bases assume a função de aplicar Chunking: divide automaticamente o texto longo em pedaços (ex: 300 tokens) com um pequeno sombreamento (overlap de ~20%) entre os fragmentos. Isso garante que frases no final de um parágrafo não percam o contexto que inicia no parágrafo seguinte.

---

## 4.2 Busca semântica e base vetorial

Explique como embeddings seriam gerados e onde seriam armazenados.

Serviços que você pode considerar:

- Amazon Bedrock Knowledge Bases
- Amazon OpenSearch Serverless
- Amazon Aurora PostgreSQL com pgvector
- Amazon S3 Vectors
- Modelos de embeddings no Amazon Bedrock

**Sua resposta:**
Os chunks de texto são submetidos a um modelo de IA (ex: Amazon Titan Text Embeddings hospedado no Amazon Bedrock), que transforma o significado semântico do texto em vetores numéricos de alta dimensão. Esses vetores são armazenados no Amazon OpenSearch Serverless (utilizando uma Vector Search Collection), que atua como o mecanismo de pesquisa ultrarrápido para cálculo de similaridade matemática entre a pergunta e os documentos.

---

## 4.3 Geração de respostas com IA

Explique como a Wiki responderia perguntas em linguagem natural com base nos documentos originais.

Considere explicar:

- Como a pergunta do usuário seria recebida;
- Como os trechos relevantes seriam recuperados;
- Como o Amazon Bedrock geraria a resposta;
- Como a resposta indicaria as fontes utilizadas.

**Sua resposta:**
O fluxo opera sob a arquitetura RAG (Retrieval-Augmented Generation):

  1. O usuário faz a pergunta e o Bedrock a transforma em um vetor.

  2. O OpenSearch Serverless compara este vetor com a base e retorna os chunks mais relevantes.

  3. O Amazon Bedrock injeta a pergunta original e os chunks recuperados em um LLM avançado (ex: Claude 3.5 Sonnet) com instruções rígidas para gerar a resposta apenas usando o contexto fornecido.

  4. A API retorna a resposta e os blocos de Citação. O sistema cruza o ID do chunk utilizado com a tabela do DynamoDB e anexa o link (Original_S3_URI) do documento fonte, provando a origem do dado.

---

## 4.4 Interface de consulta

Proponha como os usuários acessariam essa Wiki Inteligente.

Serviços que você pode considerar:

- Amazon Q Business
- AWS Amplify
- Amazon API Gateway
- AWS Lambda
- Amazon Cognito

**Sua resposta:**
Os usuários acessarão o oráculo através de uma aplicação Web segura hospedada no Amazon S3 via CloudFront. O backend das pesquisas será exposto por uma API REST via Amazon API Gateway, que aciona as AWS Lambdas de comunicação com o Bedrock. O acesso de funcionários será autenticado pelo Amazon Cognito, garantindo que apenas usuários logados (via SSO corporativo) possam fazer consultas na Wiki.

---

## 4.5 Segurança, auditoria e monitoramento

Explique como controlar acesso, proteger dados, auditar consultas e monitorar custos, erros e qualidade das respostas.

Serviços que você pode considerar:

- AWS IAM
- AWS KMS
- Amazon Cognito
- AWS CloudTrail
- Amazon CloudWatch
- Amazon Macie
- AWS Cost Explorer

**Sua resposta:**
Controle de Acesso e Proteção: O AWS IAM impõe o princípio do menor privilégio aos serviços (ex: Lambda só pode ler do OpenSearch). O AWS KMS criptografa toda a arquitetura de banco de dados e S3.

Auditoria: O AWS CloudTrail registra o log de todas as chamadas de API, garantindo auditoria de quem acessou e pesquisou informações.

Monitoramento e Custo: O Amazon CloudWatch monitora latência de APIs e métricas de erro. O AWS Cost Explorer e alertas do AWS Budgets previnem saltos de custos relacionados ao faturamento de tokens no Amazon Bedrock.

---

# 🧩 Arquitetura Final da Solução

Agora reúna tudo em uma visão única.

## 1. Visão geral

Explique em poucas linhas a ideia central da sua arquitetura.

**Sua resposta:**
A arquitetura propõe um fluxo 100% serverless de ponta a ponta na AWS para transformar ativos corporativos desestruturados em conhecimento consultável. Utiliza orquestração de microsserviços para lidar com a ingestão heterogênea (OCR, Parse Nativo, CSV), estrutura metadados via IA gerativa, e constrói uma base vetorial robusta que entrega respostas auditáveis, com alta segurança e rastreabilidade da fonte original.

---

## 2. Serviços AWS utilizados

| Serviço AWS | Papel na solução |
|---|---|
| Amazon S3 | Data Lake primário e armazenamento imutável de artefatos brutos e processados. |
| Amazon Textract | Motor de OCR para extrair texto, estrutura tabular e manuscritos de imagens escaneadas. |
| Amazon Bedrock | Extração semântica de metadados e geração RAG(LLMs base e Titan Embeddings). |
| Amazon Bedrock Knowledge Bases | Orquestração nativa do chunking e indexação para a criação da base de conhecimento. |
| AWS Lambda | Execução sob demanda para roteamento, limpeza de texto via regex e APIs de backend. |
| AWS Step Functions | Orquestração visual, controle de estado e tratamento de falhas do pipeline de ingestão. |
| Amazon CloudWatch | Observabilidade global: logs de aplicação, métricas operacionais e disparo de alarmes. |
| AWS IAM | Controle rigoroso de acesso e garantia de privilégio mínimo entre componentes e serviços. |
| AWS KMS | Criptografia em repouso ponta-a-ponta garantindo a segurança de dados corporativos. |
| Amazon OpenSearch Serverless | Banco de dados vetorial de alta performance para a busca semântica de chunks. |
| Amazon DynamoDB | Catálogo secundário de alta velocidade para indexação de metadados e URIs originais. |

Adicione, remova ou ajuste os serviços conforme sua proposta.

---

## 3. Fluxo de dados de ponta a ponta

Descreva o caminho dos dados desde a pasta `raw/` até a Wiki Inteligente.

**Sua resposta:**

  1. Arquivos são enviados para o bucket imutável raw/ no Amazon S3.

  2. Evento de upload aciona o AWS Step Functions para orquestrar o fluxo.

  3. Arquivos são analisados: Imagens vão para Textract; PDFs/Textos para extração em Lambda; CSVs são convertidos linha-a-linha em JSON.

  4. Textos são limpos (remoção de ruídos visuais e de caracteres especiais).

  5. Amazon Bedrock processa o texto e extrai metadados estruturados (responsáveis, datas, decisões).

  6. Metadados e links de origem (Original_S3_URI) são gravados no Amazon DynamoDB.

  7. Textos processados são divididos em chunks e vetorizados via Bedrock Knowledge Bases.

  8. Vetores são indexados no Amazon OpenSearch Serverless.

  9. Usuário autenticado faz uma pergunta através de portal Web.

  10. RAG orquestra busca vetorial e IA responde em linguagem natural anexando o link seguro da fonte original.


---

## 4. Diagrama textual da arquitetura

Crie um diagrama simples usando texto.

**Sua resposta:**
```
Upload(S3 Raw WORM) 
  ↳ EventBridge Trigger
    ↳ Step Functions (Orquestrador)
      ├── Choice: Imagem ➔ Textract ➔ Texto Limpo
      ├── Choice: PDF ➔ Lambda Parser ➔ Texto Limpo
      └── Choice: CSV ➔ Lambda Data Prep ➔ Texto Limpo
            ↳ S3 Processed Data + Metadados extraídos para DynamoDB
               ↳ Bedrock Knowledge Bases (Chunking & Embeddings)
                  ↳ OpenSearch Serverless (Base Vetorial)
                      ⇡ 
                      (RAG / Semantic Search)
                      ⇡
Usuário ➔ UI ➔ API Gateway ➔ Lambda Backend ➔ Bedrock (Claude 3.5)
(Autenticação via Amazon Cognito / Auditoria via AWS CloudTrail)
```

---

## 5. Riscos e limitações

Liste possíveis desafios da sua solução.

**Sua resposta:**

- Textract tem limitações perante imagens fisicamente danificadas ou com péssima resolução.
- Escalonamento imprevisível de custos de inferência LLM em caso de picos maciços de buscas.
- Os modelos de linguagem base podem ainda ser suscetíveis a leves alucinações caso os documentos de fonte contenham jargões corporativos ambíguos.
- Sem um mecanismo robusto de RBAC (Controle de Acesso Baseado em Função), as pesquisas em documentos abertos podem violar sigilo departamental na resposta generativa.

---

## 6. Melhorias futuras

Descreva como a solução poderia evoluir.

**Sua resposta:**

- Adicionar segurança por hierarquia (RBAC) interligada aos metadados, impedindo que departamentos não autorizados pesquisem atas de outros setores.
- Implementar ChatOps via Amazon Lex para interagir com a Wiki diretamente do Microsoft Teams ou Slack.
- Criação de rotinas em lote no Step Functions para enviar relatórios semanais de "Ações Pendentes" para os e-mails dos responsáveis baseados nos metadados.
- Integração com Amazon QuickSight lendo o DynamoDB para gerar painéis analíticos gerenciais sobre volume de decisões e resoluções.

---

# 🧠 Checklist Final

Antes de entregar, confirme se sua solução responde:

- [x] Como transformar documentos escaneados em texto?
- [x] Como lidar com diferentes formatos dentro da mesma pasta `raw/`?
- [x] Como armazenar os documentos originais?
- [x] Como preservar a rastreabilidade entre resposta e documento fonte?
- [x] Como organizar metadados?
- [x] Como criar busca semântica?
- [x] Como usar Amazon Bedrock na solução?
- [x] Como proteger documentos sensíveis?
- [x] Como monitorar falhas?
- [x] Como a empresa usaria essa Wiki no dia a dia?

---

# 🏁 Conclusão

Escreva uma breve conclusão defendendo sua solução como se estivesse apresentando para uma liderança técnica ou de negócio.

**Sua resposta:**

A arquitetura desenhada resolve o caos do conhecimento corporativo oculto sem a necessidade de manter e provisionar servidores (100% serverless). Ao desacoplar a ingestão brutal de dados do processamento de metadados, entregamos previsibilidade de falhas e modularidade ao sistema. Com o Amazon OpenSearch integrado nativamente à IA gerativa do Bedrock, substituímos a ineficiência das buscas manuais por pastas baseadas em palavras-chave por um modelo que "entende" os processos de negócios de maneira semântica. Trata-se de uma solução resiliente, segura por design, altamente escalável e cuja rastreabilidade garantida na resposta atende aos padrões de governança exigidos por cenários executivos e operacionais.
