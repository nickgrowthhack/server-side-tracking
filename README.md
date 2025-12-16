# Server-Side Tracking Scripts

Este repositório contém scripts personalizados desenvolvidos para serem implementados via Google Tag Manager (GTM). O objetivo principal é garantir a persistência de dados de rastreamento, atribuição de origem (UTMs) e geolocalização, além de manter a consistência desses dados durante a navegação do usuário.

## Estrutura do Projeto

O projeto é composto por dois arquivos HTML principais, cada um com um propósito e momento de disparo específicos dentro do ciclo de vida do GTM.

### 1. setCookieIndex.html

Este script é o núcleo do rastreamento de sessão e atribuição. Ele gerencia identificadores únicos e garante que os parâmetros de origem acompanhem o usuário.

- **Gatilho no GTM:** `DOM Ready` (Disparado quando o modelo de objeto do documento está pronto).
- **Domínio de Cookies:** `.alemdafotografia.net`

#### Funcionalidades Principais:
1.  **Identificação do Usuário (`index`):**
    - Verifica a existência do cookie `index`. Se não existir, tenta recuperar do `localStorage` ou cria um novo usando a variável do GTM `{{event_id}}`.
    - Persiste o ID em Cookie (1 ano), `localStorage` e `sessionStorage`.
2.  **Registro de Data de Entrada (`entry_date`):**
    - Salva o timestamp do primeiro acesso do usuário para análises de coorte ou tempo de vida.
3.  **Persistência de UTMs:**
    - Captura parâmetros UTM (`utm_source`, `utm_medium`, etc.) e identificadores de clique (`gclid`, `fbclid`) da URL.
    - Salva esses valores em cookies de longa duração (2 anos) para garantir atribuição mesmo em visitas futuras diretas (exceto se a fonte for 'organico').
4.  **Enriquecimento de Links e Iframes:**
    - Varre automaticamente todos os links (`<a>`) e iframes da página.
    - Adiciona os parâmetros de rastreamento (`sck` para o ID e `src` para as UTMs) às URLs de destino. Isso garante que o rastreamento não seja perdido ao navegar para outros domínios ou subdomínios controlados.

#### Variáveis GTM Necessárias:
- `{{event_id}}`: Usado como valor base para o cookie `index` se nenhum existir.

---

### 2. setCookiesGeoLoc.html

Este script foca no enriquecimento de dados do usuário com informações de geolocalização obtidas via API.

- **Gatilho no GTM:** `visitor-api-success` (Deve ser configurado para disparar após o sucesso da chamada a uma API de identificação de visitante, como a VisitorAPI).
- **Domínio de Cookies:** `.alemdafotografia.net`

#### Funcionalidades Principais:
- Define cookies seguros (`Secure`, `SameSite=None`) com duração de 1 ano para:
    - **Cidade:** Cookie `LeadCity`.
    - **Estado/Região:** Cookie `LeadState`.
    - **País:** Cookie `LeadCountry`.

#### Variáveis GTM Necessárias:
- `{{jsc - city (fb)}}`: Para popular o cookie `LeadCity`.
- `{{dlv - visitorApiRegion}}`: Para popular o cookie `LeadState`.
- `{{dlv - visitorApiCountryCode}}`: Para popular o cookie `LeadCountry`.

## Como Implementar

1.  Crie duas tags do tipo **HTML Personalizado** no Google Tag Manager.
2.  Copie o conteúdo de `setCookieIndex.html` para a primeira tag e configure o acionador para **DOM Ready**.
3.  Copie o conteúdo de `setCookiesGeoLoc.html` para a segunda tag e configure o acionador para o evento personalizado **visitor-api-success**.
4.  Certifique-se de que as variáveis GTM citadas acima estejam criadas e populadas corretamente no seu contêiner.
