# Extração de Notícias e Anúncios do Blog do Instagram (about.instagram.com) - Explorando Múltiplas Estratégias

Este notebook Jupyter/Colab contém código Python para extrair informações sobre notícias e anúncios do blog oficial do Instagram, especificamente da seção `https://about.instagram.com/pt-br/blog/announcements`.

## Objetivo

O presente notebook reúne diversas tentativas e scripts para extração de avisos e notícias do blog oficial do Instagram (Announcements e potencialmente conteúdo marcado com #AVISOS). Cada abordagem explora uma estratégia distinta de coleta de dados, em função das limitações da plataforma e dos métodos de publicação.

## Contexto e Múltiplas Tentativas

É importante notar que este notebook apresenta **várias abordagens e estratégias diferentes** para alcançar o mesmo objetivo: extrair dados do blog. O motivo para ter múltiplos blocos de código que parecem fazer coisas semelhantes é que o processo de web scraping e a disponibilidade/estrutura de feeds RSS podem variar. Cada célula de código Python (após as instalações iniciais) representa uma tentativa ou uma evolução na técnica de extração, testando diferentes métodos como:

1.  **Parsing de RSS:** Tentativa de usar um feed RSS, caso esteja disponível e bem estruturado.
2.  **Web Scraping Direto (HTTP):** Extração de dados navegando pelas páginas do blog (listagem e páginas de artigos).
3.  **Uso de Sitemap:** Identificação de URLs relevantes através do arquivo sitemap.xml do site.
4.  **Scraping Dinâmico (Selenium):** Considerado como uma abordagem para lidar com conteúdo carregado via JavaScript, embora o código final presente neste notebook não implemente explicitamente essa estratégia.
5.  **Refinamentos de Scraping:** Melhorias na extração de conteúdo completo e inclusão de lógica para lidar com problemas como limites de requisição (Erro 429), delays, e tratamento de diferentes estruturas HTML.

Portanto, ao executar o notebook, você verá diferentes blocos de código, alguns mais bem-sucedidos ou robustos que outros, dependendo da estrutura atual do site e da disponibilidade de recursos como RSS ou sitemaps com o formato esperado.

## Requisitos

Para executar este notebook, você precisará das seguintes bibliotecas Python. Elas são instaladas automaticamente pelas primeiras células:

*   `requests`: Para fazer requisições HTTP (baixar páginas).
*   `beautifulsoup4` (`bs4`): Para parsear HTML e XML e navegar na estrutura.
*   `python-dateutil`: Para parsear datas em vários formatos.
*   `feedparser`: Para tentar parsear feeds RSS/Atom.
*   `pandas`: Para organizar os dados extraídos em DataFrames (utilizado em algumas tentativas).
*   `tqdm`: Para exibir barras de progresso no notebook (utilizado em algumas tentativas).
*   *(Instalados, mas não usados no código final):* `selenium` e `webdriver-manager`.

As células de instalação (`!pip install ...`) devem ser executadas primeiro.

## Conteúdo e Descrição das Células de Código

Este notebook contém células que implementam as seguintes estratégias (que, em um projeto maior, poderiam ser scripts separados como `extrair_avisos_rss.py`, etc.):

### Células de Instalação (`!pip install ...`)

Instalam as bibliotecas necessárias no ambiente Colab/Jupyter. Devem ser executadas no início.

### Célula Python 1 (ID `O-KD2leVhsPR`): Feed RSS (Feedparser)

*   **Estratégia:** Tenta extrair avisos utilizando **parsing de um feed RSS** (`feedparser`).
*   **Fonte:** Configura `FEED_URL` diretamente para o URL da página de anúncios (`https://about.instagram.com/pt-br/blog/announcements`). *Nota: Esta URL pode não ser um feed RSS válido, levando o `feedparser` a tentar parsear o HTML da página, o que pode causar erros (`bozo`) ou resultados inesperados.* Uma URL mais apropriada para um feed de tag seria `https://about.instagram.com/pt-br/blog/tag/avisos/feed`, se disponível.
*   **Funcionamento:** Parseia o que `feedparser` encontra no URL. Itera sobre as "entradas" encontradas, tenta extrair título, link, data (`published` ou `updated`) e um resumo (`summary`). Inclui uma **tentativa opcional de web scraping** para buscar o conteúdo completo na URL da entrada, usando `requests` e `BeautifulSoup` para encontrar parágrafos (`<p>`) dentro de seletores CSS específicos (`.wysiwyg`, `.ContentBody`).
*   **Filtragem:** Filtra as entradas por uma `START_DATE` configurada (inicialmente 01/01/2025).
*   **Saída:** Salva os resultados em formato JSON (`avisos_blog.json`) com os campos `title`, `url`, `date`, `summary` e `content`.

### Célula Python 2 (ID `cNUP_0c9i_9X`): Scraping por Paginação HTTP

*   **Estratégia:** Utiliza **Web Scraping direto**, iterando por possíveis páginas numeradas da seção de anúncios (`/blog/announcements/page/{n}`).
*   **Fonte:** Começa no `BASE_URL` e constrói URLs para páginas subsequentes.
*   **Funcionamento:** Define funções para buscar uma página de listagem (`fetch_announcements_page`), extrair URLs de artigos que contêm uma `#HASH_TAG` específica (`#AVISOS`) no texto do cartão (`<article>` ou `.ContentItem`), e buscar o conteúdo completo de uma URL de artigo individual (`fetch_full_post`). A função `fetch_full_post` extrai título (`<h1>`, `<h2>`), data (procurando `<time>`) e conteúdo completo (procurando parágrafos em `.wysiwyg` ou `.ContentBody`).
*   **Filtragem:** Filtra os cartões na listagem pelo texto `#AVISOS` e opcionalmente filtra artigos individuais por `START_DATE` (inicialmente 01/01/2025).
*   **Saída:** Coleta os dados em uma lista e salva em formato JSON (`avisos_announcements.json`) com os campos `url`, `title`, `date` e `content`.

### Célula Python 3 (ID `RynxieVZXViZ`): Scraping via Sitemap (Abordagem 1)

*   **Estratégia:** Utiliza o **Sitemap** do site para encontrar URLs de anúncios e, em seguida, faz **Web Scraping** de cada URL encontrada.
*   **Fonte:** Usa o URL do sitemap específico para anúncios (`sitemap_url = "https://about.instagram.com/pt-br/announcements/sitemap.xml"`).
*   **Funcionamento:** A função `fetch_sitemap_urls` baixa e parseia o XML do sitemap, buscando URLs (`<loc>`) que contenham `/pt-br/blog/announcements/` e extraindo a data de última modificação (`<lastmod>`). A função `fetch_full_content` baixa a página de cada URL encontrada e tenta extrair o conteúdo principal (em `<article>` ou `div.blog-post-content`) e o título (`<h1>`).
*   **Filtragem:** A filtragem inicial é feita pela estrutura do sitemap e pelo padrão de URL. *Nesta versão específica, não há um filtro explícito por data aplicado após a extração do sitemap.*
*   **Saída:** Cria um DataFrame pandas (`df_announcements`) com as colunas `titulo`, `link`, `data_publicacao` e `conteudo_completo`.

### Célula Python 4 (ID `cQ-byPK3YduS`): Scraping via Sitemap (Abordagem 2 - Conteúdo Abrangente)

*   **Estratégia:** Refinamento da abordagem do **Sitemap** (Célula 3), com uma extração de conteúdo mais abrangente.
*   **Fonte:** Usa o mesmo URL do sitemap.
*   **Funcionamento:** Similar à Célula 3 para obter URLs do sitemap. A diferença principal está na função `fetch_all_text_elements`, que extrai texto de uma variedade maior de tags HTML (`h1`, `h2`, `h3`, `h4`, `p`, `ul`, `ol`) para construir um `conteudo_completo` mais rico. `fetch_full_page_content` orquestra a busca da página e a extração do título (`<h1>`) e do conteúdo usando `fetch_all_text_elements`.
*   **Filtragem:** Similar à Célula 3, a filtragem primária é pelo sitemap. *Também não há filtro de data explícito aplicado no loop principal nesta versão.*
*   **Saída:** Cria um DataFrame pandas (`df_anuncios_completos`).

### Célula Python 5 (ID `f7vvYbWvaVz4`): Scraping da Página de Listagem (Abordagem com Data e Polidez)

*   **Estratégia:** Retorna ao **Web Scraping da página de listagem** principal, utilizando seletores CSS específicos e incluindo filtro de data e polidez básica.
*   **Fonte:** Usa o URL da página principal de anúncios (`FULL_LISTING_URL`).
*   **Funcionamento:** `get_page_content` baixa a página com headers e timeout. `extract_article_links_and_dates` usa seletores baseados na estrutura HTML (`_aajr`, `_aajs`, `_aajt` - *esses seletores podem mudar com o tempo!*) para encontrar links, títulos e datas na página de listagem. `parse_date` lida com o formato da data ("Month Day, Year"). `extract_article_content` tenta extrair o conteúdo completo da página individual do artigo, focando em um container específico (`_aajk _aajp` - *também sujeito a mudanças!*) e parágrafos dentro dele. Inclui `time.sleep` entre as requisições de artigos.
*   **Filtragem:** Filtra os artigos encontrados na página de listagem por `START_DATE` (definido como o início do ano atual).
*   **Robustez:** Inclui tratamento básico de erros HTTP e delays (`time.sleep`).
*   **Saída:** Cria um DataFrame pandas (`df`) com Título, URL, Data e Conteúdo Completo. Imprime head e total.

### Célula Python 6 (ID `aKcS7pxNcW2P`): Scraping da Página de Listagem (Abordagem Robusta com Retry)

*   **Estratégia:** **Web Scraping da página de listagem** (evolução da Célula 5), adicionando **lógica de retentativa (retry)** para erros como 429 (Too Many Requests) e refinando mensagens.
*   **Fonte:** Mesma página de listagem (`FULL_LISTING_URL`).
*   **Funcionamento:** Similar à Célula 5 em termos de extração (`extract_article_links_and_dates` e `extract_article_content` usando seletores). A principal melhoria está em `get_page_content`, que agora implementa retentativas com espera exponencial (`time.sleep`) se encontrar um erro 429, até um `MAX_RETRIES`. Também adiciona mais `time.sleep`s para maior polidez geral antes da primeira requisição e entre o processamento de cada artigo.
*   **Filtragem:** Mantém o filtro por `START_DATE` (início do ano atual).
*   **Robustez:** É a versão mais robusta das abordagens de web scraping direto da página de listagem presentes no notebook, com retry para 429 e múltiplos delays.
*   **Saída:** Cria um DataFrame pandas (`df`). Imprime head e total.

### Célula Python 7 (ID `hRNUXI4yfWW_`): Scraping Básico Simplificado (Incompleto)

*   **Estratégia:** Uma **tentativa básica e simplificada** de Web Scraping da página de listagem.
*   **Fonte:** Página principal de anúncios.
*   **Funcionamento:** Busca todos os links (`<a>`) na página. Filtra links que começam com `/pt-br/blog/` mas *não* contêm `announcements`. Isso provavelmente **ignora os artigos de anúncio desejados** e captura outros posts do blog. **Não** baixa nem extrai o conteúdo completo das páginas dos artigos, apenas o link e o texto do link.
*   **Filtragem:** O filtro (`'announcements' not in href`) aplicado aos URLs dos links encontrados na página de listagem não corresponde ao objetivo de extrair *anúncios*.
*   **Saída:** Salva Título (texto do link) e Link em um arquivo de texto simples (`conteudo_instagram.txt`).
*   **Nota:** Esta célula parece ser uma tentativa inicial com um filtro de URL incorreto e funcionalidade limitada.

### Scraping Dinâmico (Selenium) - *Estratégia Mencionada, Código Não Presente*

*   **Estratégia:** Uso de navegador headless (como Chrome via `webdriver-manager`) para carregar a página e interagir com elementos dinâmicos (ex: clicar em "Ver mais") para revelar conteúdo carregado via JavaScript.
*   **Funcionamento:** O fluxo envolveria inicializar o navegador, navegar até a página, identificar o botão "Ver mais" ou equivalente, clicar repetidamente até todo o conteúdo ser carregado, e então usar BeautifulSoup (ou seletores do Selenium) na página carregada para extrair os dados dos avisos.
*   **Filtragem:** Seria aplicada após a extração dos dados visíveis.
*   **Nota:** Embora mencionada como uma estratégia possível para lidar com carregamento dinâmico, o código para esta abordagem específica usando Selenium **não está presente nas células deste notebook**. As dependências `selenium` e `webdriver-manager` foram instaladas, mas o código de implementação não consta.

## Estrutura

Este notebook contém células que implementam as estratégias mencionadas. Tipicamente, estas abordagens poderiam ser organizadas em arquivos `.py` separados (ex: `extrair_avisos_rss.py`, `extrair_avisos_http.py`, `extrair_avisos_selenium.py`), acompanhados de um `requirements.txt` listando as dependências específicas de cada script.

Os arquivos de saída gerados pelas células bem-sucedidas são `avisos_blog.json`, `avisos_announcements.json`, e DataFrames pandas exibidos diretamente no notebook.

## Dependências

As bibliotecas necessárias para as estratégias implementadas neste notebook são: `requests`, `beautifulsoup4`, `python-dateutil`, `feedparser`, `pandas`, `tqdm`. Elas são instaladas pelas células iniciais:

```bash
!pip install requests beautifulsoup4 python-dateutil feedparser pandas tqdm
!pip install selenium webdriver-manager # Instalado, mas código Selenium não presente
