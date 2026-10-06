# 📝 Resposta do Laboratório: A Wiki Perdida dos Arquivos Corporativos

> Preencha este arquivo com a sua proposta de solução.
>
> Sua resposta deve explicar como transformar os documentos brutos da pasta `raw/` em uma Wiki Corporativa Inteligente, pesquisável e segura usando apenas serviços da AWS.

---

## 👤 Identificação

**Nome:**  
Wendel Martins de Souza

**Data:**  
06/10/2026

**Link do repositório:**  
https://github.com/wmswendelmartins/laboratorio-wiki-aws/

---

# ✅ Quest 1: O Mapa dos Arquivos Perdidos

## 1.1 Formatos encontrados na pasta `raw/`

Descreva quais tipos de arquivos existem dentro da pasta `raw/`.

```md
Exemplo de como responder, com o formato e o que ele implica:
- <extensao>: <nasce digital ou precisa de OCR?>, <o que da para extrair>
```

> Abra a pasta e liste o que voce encontrou de fato. Esta quest avalia a sua
> leitura do acervo, entao a resposta certa e a que corresponde aos arquivos.

**Sua resposta:**

```md
OS arquivos presentes são PDF, .csv e PNG
```

---

## 1.2 Principais desafios encontrados

Explique quais dificuldades esses documentos podem apresentar.

```md
Exemplo:
- Arquivos sem padrão de nomenclatura
- Documentos escaneados com baixa qualidade
- Textos manuscritos ou parcialmente ilegíveis
- Atas com estruturas diferentes
- Informações importantes espalhadas em vários formatos
```

**Sua resposta:**

```md
PDF: pode contem tabelas e assinaturas dificultando a leitura e extração de informações
.csv: as colunas podem fazer um uso maior de token para organização
PNG: pode conter texto de caligrafia manual ou pixels que atrapalhe a leitura 
```

---

## 1.3 Informações importantes a serem extraídas

Liste quais informações precisam ser identificadas para transformar os documentos em conhecimento pesquisável.

**Sua resposta:**

```md
PDF: é a extração da hierarquia da reunião, separação de quem mandou e quem recebeu
.csv: por conta das colunas é necessario identificar e isolar os eixos importantes
PNG: é preciso fazer a separação do texto escrito do digitado
```

---

## 1.4 Estratégia de classificação inicial

Como você classificaria os documentos sem depender de subpastas dentro de `raw/`?

**Sua resposta:**

```md
Utilizaria o AWS Lambda no momento do upload para o AWS S3, assim ele faria uma verificação do tipo de arquivo e classificação em tags
```

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

```md
Seriam colocados no Amazon S3, juntamente com o Amazon Lambda para a verificação do tipo de arquivo para marcação de tags
```

---

## 2.2 Preservação dos arquivos originais

Explique como garantir que os arquivos originais sejam mantidos intactos e rastreáveis.

**Sua resposta:**

```md
Utilizaria a ferramenta S3 Object Lock em modo de governança, assim nenhum usuario mesmo com permissão de administrador não podera fazer alterações nos arquivos enquanto tiver configurando 
```

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

```md
PDF: ja possui caracteres digitais e é necessario manter a integridade estrutural, paginação e tabelas/listas presentes
.csv: o csv possui dados relacionais estruturados e é necessario o Lambda com um script Python para separar colunas e separar em um paragrafo de texto estruturado
PNG: sendo uma matriz de pixels sem texto digital é necessario focar na visão computacional utilizando analyzedcoument 
```

---

## 2.4 Tratamento de falhas

Explique como sua solução identificaria e registraria erros de processamento.

**Sua resposta:**

```md
PNG: Se a pontuação média da página for inferior menos de 80%, o arquivo não segue para indexação, ele é marcado como INSUFFICIENT_QUALITY_OCR.
PDF: Arquivos PDF corrompidos, arquivos protegidos por senha de abertura ou documentos com camadas de texto danificadas disparam PdfReadError / InvalidParameterException.
.csv: Se uma linha tiver N colunas em vez das N esperadas, ou campos nulos como valor de venda ou identificador, a linha é registrada como MALFORMED_ROW.
Com o Lambda e o Dead Letter Queue (DLQ) limitaria o criterio de erro e a 3 tentativas para não ficar em looping infinito, e notificaria o responsavel pela tarefa com o AWS EventBridge via email/slack 
```

---

# ✅ Quest 3: A Relíquia dos Metadados

## 3.1 Padronização dos textos processados

Explique como os textos extraídos seriam limpos, normalizados e preparados para consulta.

**Sua resposta:**

```md
PNG: Filtro de baixa confiança, descontinuidade de linhas e normalização de caracteres especiais
PDF: Filtro de cabeçalhos e rodapés repetitivos e reconstrução de palavras ao final do arquivo
.csv: tratamento de Nulos e remoção de colunas com ID sem significado para consulta
```

---

## 3.2 Metadados propostos

Defina quais metadados você extrairia de cada documento.

| Metadado | Por que ele é importante? |
|---|---|
| Nome do documento | Permite ao oráculo citar textualmente a fonte de onde tirou o fatoi |
| Tipo do documento | Habilita filtros rígidos na busca híbrida como restringir a resposta apenas a ata, anotacao_manuscrita ou crm |
| Data identificada | Essencial para desambiguação temporal e ordenação cronológica. |
| Tema principal | Cria um índice semântico macro que acelera a recuperação vetorial e agrupa documentos correlatos de diferentes formatos |
| Participantes | Mapeia quem estava presente ou envolvido na discussão |
| Decisões tomadas | É o núcleo de valor do acervo corporativo |
| Responsáveis | Conecta tarefas e projetos a nomes específicos de colaboradores ou equipes |
| Próximos passos | Captura pendências, prazos (deadlines) e planos de ação acordados |
| Nível de confidencialidade | Base para governança e controle de acesso, impede que a IA exponha dados comerciais sensíveis |
| Caminho do arquivo original | Fornece a linhagem do dado (data lineage) com a URI completa no Amazon S3 |

Adicione outros metadados, se necessário.

---

## 3.3 Uso de IA para enriquecimento dos documentos

Explique como o Amazon Bedrock poderia ajudar a identificar temas, decisões, responsáveis, pendências e resumos dos documentos.

**Sua resposta:**

```md
Extração estruturada com saída em JSON Schema Mode, AWS Lambda envia o texto extraido de cada documento para a API do Bedrock
PNG: o LLM utiliza de probabilidade para completar dados extraidos
PDF: o LLM analisa o arquivo inteiro para ajudar a decidir o que se o que está na primeira pagina condiz com a ultima
.csv: o LLM pode analisar o bloco inteiro em vez de uma unica linha e fazer um resumo agregado 
```

---

## 3.4 Armazenamento dos metadados

Explique onde os metadados seriam armazenados e como seriam conectados aos documentos originais.

Serviços que você pode considerar:

- Amazon S3
- Amazon DynamoDB
- AWS Glue Data Catalog
- Amazon Bedrock Knowledge Bases

**Sua resposta:**

```md
Os metadados ficam em três camadas complementares, cada com um propósito especifico: Amazon S3, DynamoDB e OpenSearch Serverless
A conexão é feita por meio de identificadores determinados e imutáveis inseridos no esquema de metadado em JSON, Quando o Amazon Bedrock recupera os fragmentos relevantes no OpenSearch para responder a uma pergunta, cada fragmento traz consigo os campos key e page_number, o modelo de linguagem utiliza esses campos para redigir a resposta com citação exata da origem
```

---

# ✅ Quest 4: O Oráculo da Wiki Inteligente

## 4.1 Estratégia de indexação

Explique como os documentos seriam divididos em trechos menores e preparados para busca semântica.

**Sua resposta:**

```md
Estrategia de divição por tipo de documento:
PDF: Respeitando paragrafos e titulos de topicos, evitando cortar deliberações ao meio e mantendo a continuidade entre páginas.
PNG: Agrupadando por blocos de anotações visuais ou topicos inteiros extraidos, garantindo que frases correlatas fiquem no mesmo trecho.
.csv: Cada linha é transformada em uma sentença descritiva unica e autocontida, formando um chunk isolado.
```

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

```md
Os blocos de texto chunks passam pelo modelo Amazon Titan Text Embeddings V2 através do serviço gerenciado Amazon Bedrock, o modelo traduz o significado semântico de cada trecho em um vetor numérico e quando o usuário faz uma pergunta, ela passa pelo mesmo modelo para gerar um "vetor de busca" compatível.
No Amazon OpenSearch Serverless, configurado de forma nativa e automática pelo Amazon Bedrock Knowledge Bases, estrutura do Registro: Cada entrada no banco armazena o vetor numérico, o texto original do trecho e os metadados de rastreio e esse armazenamento permite cruzar a busca por similaridade semântica com filtros exatos de metadados e palavras-chave.
```

---

## 4.3 Geração de respostas com IA

Explique como a Wiki responderia perguntas em linguagem natural com base nos documentos originais.

Considere explicar:

- Como a pergunta do usuário seria recebida;
- Como os trechos relevantes seriam recuperados;
- Como o Amazon Bedrock geraria a resposta;
- Como a resposta indicaria as fontes utilizadas.

**Sua resposta:**

```md
O fluxo de atendimento da Wiki Inteligente opera pelo padrão RAG (Retrieval-Augmented Generation):

Recebimento da Pergunta: O usuário digita a consulta em linguagem natural na interface web, a requisição chega via Amazon API Gateway, que autentica a chamada e aciona o Amazon Bedrock Knowledge Bases, a frase da pergunta é convertida em um vetor numérico pelo Amazon Titan Text Embeddings V2.
Recuperação dos Trechos Relevantes: O Amazon OpenSearch Serverless realiza uma busca vetorial combinada com busca por palavras-chave, ele resgata os 3 a 5 trechos com maior proximidade semântica em relação à dúvida, trazendo juntos os metadados associados.
Geração da Resposta pelo Amazon Bedrock: Um Modelo Fundacional recebe um prompt com diretrizes rígidas contendo a pergunta do usuário e os trechos recuperados como único contexto permitido, o modelo interpreta o material, sintetiza a explicação em linguagem natural e, caso a resposta não esteja nos trechos enviados, declara expressamente que a informação não foi encontrada para evitar alucinações.
Indicação e Citação das Fontes: Com base nos metadados injetados no contexto, o modelo inclui referências explícitas no corpo do texto ou em notas de rodapé e converte essas menções em links seguros para que o usuário possa abrir e conferir o trecho no arquivo original.
```

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

```md
É possivel pelo Amazon Q Business, em vez de construir uma aplicação do zero, utiliza-se a interface conversacional nativa do Amazon Q Business. Os colaboradores fazem login direto no portal web gerenciado pela própria AWS, que já traz chat, histórico de conversas, filtros de segurança integrados ao Amazon Cognito e citação clicável de documentos sem necessidade de programar nenhuma linha de frontend.
```

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

```md
Controle de Acesso e Autenticação: Amazon Cognito, controla o acesso dos usuários finais à interface da Wiki via login com autenticação multifator (MFA) e grupos de perfil.
Proteção de Dados e Conformidade: AWS KMS (Key Management Service), gerencia chaves criptográficas para proteger todos os dados no Amazon S3, nos índices vetoriais e nas tabelas de log.
Auditoria de Consultas e Operações: AWS CloudTrail, rastreia e armazena registros de todas as chamadas de API executadas na infraestrutura, identificando quem fez a requisição, horário, IP de origem e quais documentos foram lidos no S3 para fins de compliance e LGPD.
Monitoramento de Custos: AWS Cost Explorer, permite acompanhar e projetar os gastos por serviço, tokens do Bedrock, execuções do Lambda e páginas do Textract, em conjunto com o AWS Budgets, dispara alertas automáticos via e-mail caso o consumo mensal se aproxime do teto estabelecido.
Monitoramento de Erros e Qualidade das Respostas: Amazon CloudWatch, coleta métricas de latência, taxa de erros nas APIs e falhas de execução no Lambda em tempo real, gerando alarmes automáticos em caso de indisponibilidade.
```

---

# 🧩 Arquitetura Final da Solução

Agora reúna tudo em uma visão única.

## 1. Visão geral

Explique em poucas linhas a ideia central da sua arquitetura.

**Sua resposta:**

```md
A ideia central é uma arquitetura Serverless baseada no padrão RAG (Retrieval-Augmented Generation), os arquivos chegam brutos e imutáveis no Amazon S3, um AWS Lambda identifica o formato de cada um e aciona a esteira correta (Amazon Textract para OCR de manuscrito e PDFs; scripts para transformar CSV em texto semântico), particiona o conteúdo com metadados de rastreio (arquivo, página e linha) e armazena os vetores no Amazon OpenSearch Serverless via Amazon Bedrock Knowledge Bases. Quando o usuário faz uma pergunta em linguagem natural, o modelo fundamento no Bedrock busca os trechos correspondentes e gera uma resposta, citando explicitamente a fonte original de onde extraiu o fato.
```

---

## 2. Serviços AWS utilizados

| Serviço AWS | Papel na solução |
|---|---|
| Amazon S3 | OK |
| Amazon Textract | OK |
| AWS Lambda | OK |
| AWS S3 Object Lock | OK |
| AWS IAM | OK |
| Dead Letter Queue (DLQ) | OK |
| AWS EventBridge | OK |
| Amazon Bedrock | OK |


Adicione, remova ou ajuste os serviços conforme sua proposta.

---

## 3. Fluxo de dados de ponta a ponta

Descreva o caminho dos dados desde a pasta `raw/` até a Wiki Inteligente.

```md
Exemplo de estrutura:

1. Arquivos estão inicialmente na pasta raw/
2. Arquivos são enviados para o Amazon S3
3. Documentos escaneados passam pelo Amazon Textract
4. Arquivos digitais têm seus textos extraídos
5. Textos são limpos e padronizados
6. Metadados são extraídos
7. Conteúdos são indexados em uma base pesquisável
8. Usuário pesquisa na Wiki
9. IA responde com base nos documentos originais
```

**Sua resposta:**

```md
Armazenamento: Arquivos da pasta raw/ são salvos de forma imutável no Amazon S3.
Roteamento: Um AWS Lambda identifica o tipo de cada arquivo e aciona a esteira correta.
Extração: Amazon Textract extrai textos e manuscritos (PDF e imagem); script no Lambda converte as linhas do CSV em frases descritivas.
Limpeza e Metadados: Textos são limpos, divididos em blocos menores (chunking) e associados a etiquetas de origem (arquivo, página e linha).
Vetorização: Amazon Titan Embeddings converte os blocos em vetores, indexados no Amazon OpenSearch Serverless via Amazon Bedrock Knowledge Bases.
Consulta: Usuário faz uma pergunta em linguagem natural na interface web (Amplify + API Gateway + Cognito).
Resposta: O modelo Claude 3.5 no Bedrock analisa a pergunta junto aos trechos resgatados e gera a resposta citando a fonte e a página exatas.
```

---

## 4. Diagrama textual da arquitetura

Crie um diagrama simples usando texto.

```md
Exemplo:

raw/ → Amazon S3 → Lambda/Step Functions → Textract → S3 Processado → Bedrock Knowledge Bases → Interface de Consulta → Usuário Final
```

**Sua resposta:**

```md
[raw/] → [Amazon S3 (Bruto)] → [AWS Lambda] → [Textract / Script Python] → [S3 Processado] → [Bedrock Knowledge Bases] ⇄ [Claude 3.5] ⇄ [API Gateway + Amplify] ⇄ [Usuário Final]
```

---

## 5. Riscos e limitações

Liste possíveis desafios da sua solução.

```md
Exemplo:
- Documentos ilegíveis podem prejudicar a extração de texto.
- OCR pode gerar erros em documentos com baixa qualidade.
- Custos podem aumentar conforme o volume de documentos.
- Metadados inferidos por IA podem precisar de validação humana.
- Respostas geradas por IA devem sempre referenciar documentos de origem.
```

**Sua resposta:**

```md
Variações caligráficas, manchas, dobras ou ruídos de escaneamento na folha manuscrita podem ser interpretados como caracteres espúrios pelo OCR.

Quebras de parágrafo no particionamento (chunking) da ata em PDF podem fragmentar deliberações longas e diluir o contexto entre páginas.

Ambiguidade em linhas do CSV com dados ausentes ou preenchidos fora de padrão pode gerar frases semânticas imprecisas para o modelo.

Gargalos de concorrência ou limites de taxa (throttling) nas APIs do Amazon Textract e Amazon Bedrock durante uploads em massa.

Sincronização desatualizada da base vetorial se houver falha no gatilho de reindexação automática após o envio de novos documentos.

Perda de rastreabilidade da citação se os metadados de página ou linha forem omitidos ou sobrescritos durante a etapa de limpeza.
```

---

## 6. Melhorias futuras

Descreva como a solução poderia evoluir.

```md
Exemplo:
- Criar uma interface web para consulta.
- Criar um chat interno para perguntas sobre atas.
- Adicionar controle de acesso por departamento.
- Criar dashboard de decisões e pendências.
- Gerar alertas automáticos sobre ações em aberto.
- Integrar com ferramentas corporativas.
```

**Sua resposta:**

```md
Preencha aqui.
```

---

# 🧠 Checklist Final

Antes de entregar, confirme se sua solução responde:

- [X] Como transformar documentos escaneados em texto?
- [X] Como lidar com diferentes formatos dentro da mesma pasta `raw/`?
- [X] Como armazenar os documentos originais?
- [X] Como preservar a rastreabilidade entre resposta e documento fonte?
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

```md
Preencha aqui.
```
