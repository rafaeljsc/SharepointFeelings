# SharepointFeelings

**Conceito:** harvester de engajamento da intranet que varre as páginas SharePoint, captura reações/curtidas e comentários (com respostas e curtidas nos comentários) e **persiste tudo num modelo relacional no Dataverse**.

**Objetivo:** transformar as interações dos colaboradores com o conteúdo da intranet num conjunto de dados estruturado e consultável (base para relatórios/BI), no período de **D-1**.

**Funcionamento:**

- **Gatilho:** recorrência diária.
- **Autenticação:** Azure AD OAuth **app-only com certificado (PFX)** via ações HTTP diretas às APIs REST/Search do SharePoint (necessário para acessar comentários e reações).
- **Para cada site → cada página** (paginação de 500 via Search API num laço *Until* até esvaziar):
  - **Reações da página:** busca quem curtiu e **grava cada curtida** na tabela de *likes* da página.
  - **Comentários:** busca comentários (com respostas e curtidas), filtra pelo período e, havendo comentários, **grava**: cada comentário; cada **resposta** a comentário; e cada **curtida em comentário** (associando à página e ao comentário pai).
  - **Resumo da página:** se houve qualquer comentário ou reação, **grava um registro-resumo da página** com contagens de visualizações, curtidas e comentários.
- **Modelo gravado no Dataverse**: páginas, curtidas de página, comentários, respostas de comentários e curtidas de comentários — um esquema normalizado encadeado por `page_id` e `comment_id`.
