# Extração de Notícias e Anúncios do Blog do Instagram (about.instagram.com)

Este notebook Jupyter/Colab contém código Python para extrair informações sobre notícias e anúncios do blog oficial do Instagram, especificamente da seção `https://about.instagram.com/pt-br/blog/announcements`.

## Contexto e Múltiplas Tentativas

É importante notar que este notebook apresenta **várias abordagens e estratégias diferentes** para alcançar o mesmo objetivo: extrair dados do blog. O motivo para ter múltiplos blocos de código que parecem fazer coisas semelhantes é que o processo de web scraping e a disponibilidade/estrutura de feeds RSS podem variar. Cada célula de código Python (após as instalações iniciais) representa uma tentativa ou uma evolução na técnica de extração, testando diferentes métodos como:

1.  **Parsing de RSS:** Tentativa de usar um feed RSS, caso esteja disponível e bem estruturado.
2.  **Web Scraping Direto:** Extração de dados navegando pelas páginas do blog.
3.  **Uso de Sitemap:** Identificação de URLs relevantes através do arquivo sitemap.xml do site.
4.  **Refinamentos de Scraping:** Melhorias na extração de conteúdo completo e inclusão de lógica para lidar com problemas como limites de requisição (Erro 429).

Portanto, ao executar o notebook, você verá diferentes blocos de código, alguns mais bem-sucedidos ou robustos que outros, dependendo da estrutura atual do site e da disponibilidade de recursos como RSS ou sitemaps com o formato esperado.

## Requisitos

Para executar este notebook, você precisará das seguintes bibliotecas Python. Elas são instaladas automaticamente pelas primeiras células:

*   `feedparser`: Para tentar parsear feeds RSS/Atom.
*   `beautifulsoup4` (`bs4`): Para parsear HTML e XML e navegar na estrutura.
*   `requests`: Para fazer requisições HTTP (baixar páginas).
*   `python-dateutil`: Para parsear datas em vários formatos.
*   `pandas`: Para organizar os dados extraídos em DataFrames (utilizado nas tentativas mais recentes).
*   `tqdm`: Para exibir barras de progresso no notebook (utilizado nas tentativas mais recentes).
*   *(Opcional/Não Utilizado no Código Final):* `selenium` e `webdriver-manager` foram instalados na célula inicial, mas não foram implementados nas estratégias de extração presentes, talvez fossem para uma abordagem que não foi concluída ou testada.

As células de instalação (`!pip install ...`) devem ser executadas primeiro.

## Descrição das Células de Código

### Células de Instalação (`!pip install ...`)

Instalam as bibliotecas necessárias no ambiente Colab/Jupyter. Devem ser executadas no início.

### Primeira Célula Python (Após instalações - `O-KD2leVhsPR`)

*   **Estratégia:** Tenta extrair avisos utilizando **parsing de um feed RSS** (`feedparser`).
*   **Fonte:** Configura `FEED_URL` diretamente para o URL da página de anúncios.
*   **Funcionamento:** Parseia o que `feedparser` encontra no URL (que, como observado, pode não ser um feed RSS válido e sim o HTML da página, levando a erros ou resultados inesperados). Itera sobre as "entradas" encontradas, tenta extrair título, link, data (`published` ou `updated`) e um resumo (`summary`). Inclui uma **tentativa opcional de web scraping** para buscar o conteúdo completo na URL da entrada, usando `requests` e `BeautifulSoup` para encontrar parágrafos (`<p>`) dentro de seletores CSS específicos (`.wysiwyg`, `.ContentBody`).
*   **Filtro:** Filtra as entradas por uma `START_DATE` configurada.
*   **Saída:** Salva os resultados em formato JSON (`avisos_blog.json`).
*   **Nota:** Esta abordagem assume a existência de um feed RSS válido no `FEED_URL` especificado, o que pode não ser o caso para o blog do Instagram. A parte de web scraping para conteúdo completo pode funcionar como um fallback.

### Segunda Célula Python (`cNUP_0c9i_9X`)

*   **Estratégia:** Utiliza **Web Scraping direto**, iterando por possíveis páginas numeradas da seção de anúncios e filtrando por um `#HASH_TAG` no texto do cartão.
*   **Fonte:** Começa no `BASE_URL` e constrói URLs para páginas subsequentes (`/page/{page}`).
*   **Funcionamento:** Define funções para buscar uma página de listagem (`fetch_announcements_page`), extrair URLs de artigos que contêm uma `#HASH_TAG` específica (`extract_avisos_urls`), e buscar o conteúdo completo de uma URL de artigo individual (`fetch_full_post`). A função `fetch_full_post` extrai título, data (procurando `<time>`) e conteúdo completo (procurando parágrafos em `.wysiwyg` ou `.ContentBody`).
*   **Filtro:** Filtra os cartões na listagem pelo texto `#AVISOS` e opcionalmente filtra artigos individuais por `START_DATE`.
*   **Saída:** Coleta os dados em uma lista e salva em formato JSON (`avisos_announcements.json`).

### Terceira Célula Python (`RynxieVZXViZ`)

*   **Estratégia:** Utiliza o **Sitemap** do site para encontrar URLs de anúncios e, em seguida, faz **Web Scraping** de cada URL encontrada.
*   **Fonte:** Usa o URL do sitemap específico para anúncios (`sitemap_url = "https://about.instagram.com/pt-br/announcements/sitemap.xml"`).
*   **Funcionamento:** A função `fetch_sitemap_urls` baixa e parseia o XML do sitemap, buscando URLs (`<loc>`) que contenham `/pt-br/blog/announcements/` e extraindo a data de última modificação (`<lastmod>`). A função `fetch_full_content` baixa a página de cada URL encontrada e tenta extrair o conteúdo principal (em `<article>` ou `div.blog-post-content`) e o título (`<h1>`).
*   **Filtro:** A filtragem inicial é feita pela estrutura do sitemap e pelo padrão de URL. *Nesta versão, não há um filtro explícito por `START_DATE` no loop principal, embora a data seja extraída do sitemap.*
*   **Saída:** Cria um DataFrame pandas (`df_announcements`) com título, link, data e conteúdo.

### Quarta Célula Python (`cQ-byPK3YduS`)

*   **Estratégia:** Refinamento da abordagem do **Sitemap** (Célula 3), com uma extração de conteúdo mais abrangente.
*   **Fonte:** Usa o mesmo URL do sitemap.
*   **Funcionamento:** Similar à Célula 3 para obter URLs do sitemap. A diferença principal está na função `fetch_all_text_elements`, que extrai texto de uma variedade maior de tags HTML (`h1`, `h2`, `h3`, `h4`, `p`, `ul`, `ol`) para construir um `conteudo_completo` mais rico. `fetch_full_page_content` orquestra a busca da página e a extração do título (`<h1>`) e do conteúdo usando `fetch_all_text_elements`.
*   **Filtro:** Similar à Célula 3, a filtragem primária é pelo sitemap. *Também não há filtro de `START_DATE` aplicado no loop principal nesta versão.*
*   **Saída:** Cria um DataFrame pandas (`df_anuncios_completos`).

### Quinta Célula Python (`f7vvYbWvaVz4`)

*   **Estratégia:** Retorna ao **Web Scraping da página de listagem** principal, utilizando seletores CSS específicos e incluindo filtro de data e polidez.
*   **Fonte:** Usa o URL da página principal de anúncios (`FULL_LISTING_URL`).
*   **Funcionamento:** `get_page_content` baixa a página com headers e timeout. `extract_article_links_and_dates` usa seletores baseados na estrutura HTML (`_aajr`, `_aajs`, `_aajt`) para encontrar links, títulos e datas na página de listagem. `parse_date` lida com o formato da data. `extract_article_content` tenta extrair o conteúdo completo da página individual do artigo, focando em um container específico (`_aajk _aajp`) e parágrafos dentro dele. Inclui `time.sleep` entre as requisições.
*   **Filtro:** Filtra os artigos encontrados na página de listagem por `START_DATE` (definido como o início do ano atual).
*   **Robustez:** Inclui tratamento básico de erros HTTP e delays (`time.sleep`).
*   **Saída:** Cria um DataFrame pandas (`df`) com Título, URL, Data e Conteúdo Completo. Imprime head e total.

### Sexta Célula Python (`aKcS7pxNcW2P`)

*   **Estratégia:** **Web Scraping da página de listagem** (evolução da Célula 5), adicionando **lógica de retentativa (retry)** para erros como 429 (Too Many Requests) e refinando mensagens.
*   **Fonte:** Mesma página de listagem (`FULL_LISTING_URL`).
*   **Funcionamento:** Similar à Célula 5 em termos de extração (`extract_article_links_and_dates` e `extract_article_content` usando seletores como `_aajr`, `_aajs`, `_aajt`, `_aajk _aajp`, etc.). A principal melhoria está em `get_page_content`, que agora implementa retentativas com espera exponencial (`time.sleep`) se encontrar um erro 429, até um `MAX_RETRIES`. Também adiciona mais `time.sleep`s para maior polidez geral.
*   **Filtro:** Mantém o filtro por `START_DATE` (início do ano atual).
*   **Robustez:** É a versão mais robusta das abordagens de web scraping direto da página de listagem, com retry para 429 e múltiplos delays.
*   **Saída:** Cria um DataFrame pandas (`df`). Imprime head e total. Esta célula é a que apresenta a abordagem mais completa e resiliente dentro do notebook para o scraping direto da listagem.

### Sétima Célula Python (`hRNUXI4yfWW_`)

*   **Estratégia:** Uma **tentativa básica e simplificada** de Web Scraping da página de listagem.
*   **Fonte:** Página principal de anúncios.
*   **Funcionamento:** Busca todos os links (`<a>`) na página. Filtra links que começam com `/pt-br/blog/` mas *não* contêm `announcements`. Isso provavelmente **ignora os artigos de anúncio desejados** e captura outros posts do blog. **Não** baixa nem extrai o conteúdo completo das páginas dos artigos.
*   **Filtro:** O filtro (`'announcements' not in href`) é aplicado aos URLs dos links encontrados na página de listagem.
*   **Saída:** Salva Título (texto do link) e Link em um arquivo de texto simples (`conteudo_instagram.txt`).
*   **Nota:** Esta célula parece ser uma tentativa inicial com um filtro de URL incorreto para o objetivo de extrair *anúncios* e não extrai o conteúdo completo dos artigos.

## Como Executar

1.  Abra o notebook em um ambiente compatível com Jupyter (como Google Colab ou Jupyter Notebook/Lab local).
2.  Execute as primeiras células (`!pip install ...`) para instalar as bibliotecas.
3.  Execute a célula de código que corresponde à estratégia que você deseja utilizar. As células são independentes umas das outras (após as instalações). As células 6 ou 8 provavelmente são as mais completas para web scraping direto da listagem, enquanto as 3 e 4 usam a abordagem de sitemap. A célula 3 tenta RSS (mas o URL pode ser inválido para isso).
4.  Observe a saída no notebook (prints de progresso/status, head do DataFrame) e verifique os arquivos gerados (`.json`, `.txt`) no ambiente de arquivos (à esquerda no Colab).

## Considerações Importantes

*   **Fragilidade:** Web scraping é inerentemente frágil. Alterações na estrutura HTML do site (`about.instagram.com`) podem quebrar os seletores CSS (`_aajr`, `.wysiwyg`, etc.) utilizados, exigindo que o código seja atualizado.
*   **Polidez:** O código inclui `time.sleep` em algumas tentativas (especialmente a Célula 8) para adicionar atrasos entre as requisições, sendo mais educado com o servidor do site. Ao raspar sites, é fundamental não sobrecarregá-los.
*   **Filtro de Data:** A `START_DATE` configurada em algumas tentativas (Células 1, 2, 5, 6) pode precisar ser ajustada para o período desejado. A Célula 8 usa o início do ano atual, o que é uma configuração comum.
*   **Erros:** Erros de rede, timeouts, ou mudanças súbitas no site podem fazer o script falhar. A Célula 8 inclui lógica básica de retry para mitigar erros temporários como 429.

Este notebook serve como um registro das diferentes abordagens experimentadas para extrair informações do blog de anúncios do Instagram, demonstrando a evolução e os desafios comuns no desenvolvimento de scripts de web scraping.
