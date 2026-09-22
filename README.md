# 🧭 Laboratório Prático: A Wiki Perdida dos Arquivos Corporativos

[![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)](#)
[![Serverless](https://img.shields.io/badge/Serverless-%23FD5750.svg?style=for-the-badge&logo=serverless&logoColor=white)](#)

Este repositório contém a resolução do Desafio de Projeto da plataforma Digital Innovation One (DIO). O objetivo é desenhar uma arquitetura 100% AWS para organizar e pesquisar documentos não estruturados corporativos.

## 🎯 O Problema que o Projeto Resolve
A empresa fictícia "Vendas S.A." acumulou anos de registros comerciais em formatos totalmente diferentes (atas em PDF digital, relatórios escaneados com anotações manuais e planilhas de CRM em CSV). Todos esses arquivos foram jogados em uma única pasta (`raw/`), sem organização, subpastas ou padronização. 

O desafio consiste em projetar uma solução que transforme esse caos de dados brutos em uma **Wiki Inteligente**. A solução deve permitir que um funcionário faça perguntas em linguagem natural (ex: *"Quais as decisões sobre o projeto X?"*) e receba respostas exatas, citando as fontes de onde a informação foi retirada.

## 🏗️ Como a Arquitetura Funciona (Do arquivo bruto à resposta)
A arquitetura proposta foi desenhada em um pipeline Serverless que separa ingestão, extração de metadados e busca semântica (RAG):

1. **Ingestão:** Os arquivos são depositados no Amazon S3, acionando um fluxo de orquestração automatizado no AWS Step Functions.
2. **Processamento Especializado:** Cada formato de arquivo toma uma rota diferente para extração de texto, garantindo que o contexto visual e espacial não se perca.
3. **Enriquecimento:** O texto extraído é limpo via AWS Lambda. Em seguida, o Amazon Bedrock (IA) analisa o documento para extrair metadados como responsáveis, prazos e decisões, guardando essas chaves no Amazon DynamoDB junto com o link do arquivo original.
4. **Vetorização:** O texto processado é fatiado (chunking) e transformado em embeddings vetoriais via Bedrock Knowledge Bases.
5. **Busca RAG:** O usuário pergunta na interface; o Amazon OpenSearch Serverless cruza a similaridade matemática, e o Claude (LLM) formula a resposta referenciando a fonte arquivada no S3.

## 🛠️ Serviços Escolhidos e Por Quê
*   **Amazon S3 (com Object Lock):** Escolhido para ser o Data Lake primário pela capacidade de impor imutabilidade aos arquivos brutos (WORM), garantindo que a fonte da verdade nunca seja alterada.
*   **AWS Step Functions:** Vital para orquestrar o roteamento dos arquivos sem necessidade de gerenciar servidores, utilizando lógicas nativas de tentativa (*retry*) em caso de falha de processamento.
*   **Amazon Textract:** Escolhido especificamente para a rota de arquivos de imagem, pois consegue extrair tabelas mantendo a estrutura espacial e ler caligrafia (anotações à mão).
*   **Amazon DynamoDB:** Banco NoSQL perfeito para atuar como catálogo de metadados devido à sua latência de milissegundos para buscas por atributos (ex: encontrar todos os documentos onde "Responsável = Rafael Nunes").
*   **Amazon Bedrock & OpenSearch Serverless:** A combinação ideal para executar a técnica RAG. O OpenSearch atua como banco vetorial para a busca semântica, e o Bedrock isola a IA da infraestrutura, garantindo segurança corporativa aos dados inferidos.

## 🗂️ Como os Três Formatos são Tratados
O maior desafio deste laboratório é a heterogeneidade da pasta `raw/`. A solução trata cada arquivo de forma isolada:
*   **PDF com camada de texto:** Evita-se o uso oneroso de OCR. O arquivo é encaminhado para uma função AWS Lambda com bibliotecas nativas de *parsing* (como pdfplumber), que extraem as tabelas respeitando as colunas originais.
*   **Imagem escaneada:** Direcionada para a API do Amazon Textract (`AnalyzeDocument`), que consegue "ler" os pixels, traduzindo carimbos e notas redigidas sobre o documento.
*   **CSV (Dados estruturados):** Não pode ser lido como texto corrido para não causar ruído semântico. Uma função Lambda transforma cada linha da tabela de vendas em um micro-documento JSON (ex: *"Venda de R$ 50.000 para o cliente Y, responsável Z"*), dando contexto à IA.

## 🎓 O Que Aprendi Durante o Desafio
Este desafio consolidou minha visão sobre arquitetura de dados e inteligência artificial na nuvem. Os principais aprendizados foram:
1.  **Governança de Dados:** Entendi a importância de manter a rastreabilidade ligando os metadados extraídos (DynamoDB) à fonte original (URI do S3), para que a IA possa citar o documento exato.
2.  **Abordagens Específicas:** Aprendi que nem todo dado precisa de OCR e que dados tabulares (CSV) exigem pré-estruturação antes da vetorização. 
3.  **Orquestração Serverless:** Compreendi na prática o valor de desacoplar lógicas de negócios usando o Step Functions em vez de criar dependências monolíticas, aumentando a resiliência a falhas do sistema.

---
*Este é o documento principal do projeto. O detalhamento completo da arquitetura e as respostas técnicas das 4 Quests encontram-se no arquivo `resposta.md`.*
