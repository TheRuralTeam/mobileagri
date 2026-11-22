# AgriLink Mobile — documentação do UI

> Estado: documentação técnica inicial baseada no código da branch `main`.
> Repositório: `TheRuralTeam/mobileagri`
> Aplicação: Expo / React Native / Expo Router (Android e iOS).
>
> Esta documentação descreve o que está declarado no código. A presença de um botão ou de uma integração no código não prova, por si só, que o fluxo funciona em produção. Os pontos que dependem de credenciais, políticas RLS, migrações ou serviços externos devem ser validados no dispositivo e no Supabase.

## 1. Objetivo e âmbito

O mobile é a aplicação móvel da AgriLink. O UI organiza-se em torno de quatro perfis principais — fornecedor/agricultor, agente, comprador e motorista — além do modo visitante e da área administrativa.

Este documento cobre:
- mapa de rotas e navegação;
- ecrãs e respetiva finalidade;
- componentes partilhados e padrões visuais;
- estados de interface que devem ser considerados;
- integrações visíveis no cliente;
- dependências e validações necessárias para considerar cada fluxo funcional.

Não cobre, nem substitui, a documentação do backend, das políticas RLS ou das migrações SQL.

## 2. Base técnica

| Área | Implementação observada |
|---|---|
| Framework | React Native 0.86 / Expo SDK 57 |
| Navegação | Expo Router, entrada `expo-router/entry` |
| Linguagem | TypeScript |
| UI e estilos | React Native StyleSheet, NativeWind |
| Animações | React Native Reanimated |
| Ícones | componente interno `Icon`, lucide-react-native e Expo Vector Icons |
| Estado local persistente | AsyncStorage |
| Backend | Supabase JS |
| Internacionalização | i18next / react-i18next |
| Mapas | react-native-maps, WebView + Leaflet/OpenStreetMap |
| Localização | expo-location / expo-task-manager |
| Imagens | expo-image-picker |
| Ficheiros | expo-document-picker |
| Gestos e áreas seguras | gesture-handler / safe-area-context |

Configuração de aplicação em `app.json`: orientação vertical, interface clara, scheme `agrilink`, ícone próprio e permissões de localização em uso e em segundo plano. O pacote Android declarado é `com.lucamba.agrilink`.

## 3. Identidade visual e linguagem de interface

### 3.1 Tokens observados

A paleta mais recorrente nas telas principais é:

| Token | Valor | Uso observado |
|---|---|---|
| Verde primário | `#218B4A` | ações principais, ícones ativos, identidade |
| Verde suave | `#E9F5EC` / `#EAF5ED` | fundos de destaque e estados ativos |
| Texto | `#16231C` | títulos e conteúdo |
| Texto secundário | `#78877D` | legendas e metadados |
| Fundo | `#F3F6F2` | canvas geral |
| Superfície | `#FFFEFC` | cartões e campos |
| Borda | `#E8ECE6` | separadores e contornos |
| Alerta | `#DD5138` / `#B54747` | erros e ações destrutivas |
| Destaque | `#B9741A` / `#E2932F` | estados e informação complementar |

Os valores são tokens encontrados nos ficheiros; algumas telas mantêm pequenas variações locais. Não se deve assumir que todos os ecrãs já usam um único ficheiro central de design tokens.

### 3.2 Padrões recorrentes

- cartões com fundos claros, bordas finas e sombras discretas;
- botões verdes para ações primárias;
- ícones acompanhados de rótulos ou informação contextual;
- formulários com ícone, foco visual e mensagens de validação;
- animações de entrada e feedback de toque em ecrãs como login e cadastro;
- uso de safe areas e scroll vertical para acomodar diferentes dimensões de ecrã.

### 3.3 Recomendações de consistência

Ao evoluir o UI, preservar a identidade verde da AgriLink, manter contraste legível, não depender apenas da cor para comunicar estados e garantir que textos longos, erros e traduções não cortam botões ou cartões. Novos componentes devem reutilizar os tokens e componentes partilhados sempre que possível.

## 4. Arranque, sessão e controlo de acesso

### 4.1 Arranque

1. A rota `/` redireciona para `/onboarding`.
2. O onboarding apresenta a marca com animação e progresso.
3. Depois de a sessão/modo visitante estar resolvido, encaminha utilizadores autenticados ou visitantes para `/home`; utilizadores sem sessão seguem para `/login`.
4. O layout raiz aguarda a preferência de idioma guardada antes de mostrar o navegador principal.

### 4.2 Layout raiz

O ficheiro `src/app/_layout.tsx` declara um Stack sem cabeçalho nativo, transição horizontal e fundo `#F3F6F2`. Também envolve a aplicação com `RoleProvider` e `ProcessingScreenProvider`.

No modo visitante, o layout permite navegar em `/home`, `/search`, `/green-points` e detalhes de produto, além das rotas de autenticação. As restantes rotas redirecionam para `/home`. O componente `GuestModeCard` é apresentado nas rotas de exploração permitidas.

### 4.3 Papéis e ação central

O papel é normalizado em `src/constants/roleActions.ts` e resolvido por `RoleContext`, procurando dados nas tabelas `users` e `profiles`, com fallback para metadados da sessão.

| Papel normalizado | Ação central da barra inferior | Rota |
|---|---|---|
| agricultor / fornecedor | Publicar produto | `/publicar-produto` |
| agente | Registar produto | `/publicar-produto` |
| comprador | Fichas técnicas | `/fichas-recebimento` |
| motorista | Cargas | `/entregas` |
| sem papel resolvido | Início (fallback) | `/home` |

A normalização reconhece sinónimos em português e inglês, como `farmer`, `supplier`, `buyer` e `driver`.

## 5. Navegação principal

### 5.1 Barra inferior

O componente `BottomToolbar` apresenta cinco áreas:

1. Início — `/home`; para motorista, o primeiro destino passa a ser `/notifications`.
2. Mensagens — `/messages`.
3. Ação contextual — rota determinada pelo papel.
4. Mapa — `/mapa`.
5. Perfil — `/profile`.

A barra é ocultada no modo visitante. O item central usa o verde da marca. A navegação é baseada no papel, não numa lista independente de permissões; os próprios ecrãs também devem aplicar guardas de acesso.

### 5.2 Rotas registadas no Stack

| Rota | Ecrã / finalidade |
|---|---|
| `/` | redirecionamento inicial |
| `/onboarding` | apresentação animada da marca e encaminhamento |
| `/login` | autenticação |
| `/register` | criação de conta em etapas |
| `/verify-otp` | confirmação por código |
| `/forgot-password` | pedido de recuperação |
| `/reset-password` | definição de nova palavra-passe |
| `/home` | feed principal |
| `/search` | pesquisa de produtos e pessoas |
| `/product/[id]` | detalhe de produto |
| `/profile` | perfil do utilizador e opções associadas |
| `/profile/[id]` | perfil público de outro utilizador |
| `/notifications` | notificações |
| `/dashboard` | painel administrativo |
| `/green-points` | pontos verdes, produtos, stock e reservas |
| `/agrilink-ads` | gestão de anúncios |
| `/publicar-produto` | publicação de produto |
| `/fichas-recebimento` | criação/gestão de fichas de recebimento |
| `/entregas` | cargas disponíveis e cargas do motorista |
| `/rastreamentos` | lista de entregas e rastreios |
| `/rastreamento/[id]` | detalhe e acompanhamento de uma carga |
| `/mapa` | mapa de produtos e participantes |
| `/conversations` | lista e pesquisa de conversas |
| `/messages` | conversa e envio de mensagens |
| `/seguranca` | segurança e privacidade |
| `/historicopagamentos` | histórico de pagamentos |

Nota: a lista acima reflete os nomes declarados no Stack raiz. Rotas adicionais podem existir no diretório e devem ser acrescentadas após inventário integral do projeto.

## 6. Catálogo dos ecrãs

### 6.1 Acesso e conta

#### Onboarding
- Apresenta marca AgriLink, símbolo de folha, wordmark e linha de progresso.
- Usa animação de opacidade, escala e deslocamento.
- Encaminha para início ou login conforme o estado de autenticação/visitante.

#### Login
- Campos de e-mail/telefone e palavra-passe.
- Alternância para mostrar/ocultar palavra-passe.
- Login com credenciais Supabase e opção Google.
- Entrada em modo visitante.
- Navegação para cadastro e recuperação de palavra-passe.
- Feedback de validação e erros via Alert.

#### Cadastro
- Formulário dividido em três etapas: perfil, contacto e segurança.
- Papéis: fornecedor/agricultor, agente, comprador e motorista.
- Dados observados: nome, documento, capacidade de carga, e-mail, telefone, província, município, palavra-passe e confirmação.
- Inclui opção de referência por agente e validação de código.
- Inclui opção Google e modo visitante.
- O fluxo de confirmação é configurável por e-mail, telefone ou ambos.

#### Verificação OTP
- Recebe parâmetros do fluxo de cadastro.
- Apresenta campo para código de seis dígitos, verificação e reenvio com temporizador.
- Guarda o perfil verificado através de helper próprio.
- Deve ser validado com os modelos de e-mail/SMS e configuração Auth efetiva no projeto Supabase.

#### Recuperação de palavra-passe
- `/forgot-password`: recolhe e-mail, solicita link de recuperação e apresenta confirmação.
- `/reset-password`: processa link/deep link, valida sessão de recuperação e permite definir e confirmar nova palavra-passe.

### 6.2 Descoberta e mercado

#### Home / Feed
- Feed de produtos com categorias, cartões de produto e atualização.
- Declara categorias como frutas, cítricos, legumes, verduras, cereais, temperos, pescado, carnes, ovos, pão, lacticínios, bebidas e processados.
- Usa `ProductCard`, anúncios, ligações entre utilizadores e barra inferior.
- O cartão de produto inclui galeria de imagens, preço, quantidade, produtor, likes, comentários, mapa e ação de pré-encomenda.
- Atenção: o código de `home.tsx` declara produtos mock de demonstração quando não há produtos reais no Supabase. Estes dados não devem ser confundidos com inventário real nem apresentados como dados de produção.

#### Pesquisa
- Pesquisa produtos e utilizadores.
- Abas: Tudo, Produtos e Pessoas.
- Filtros de categoria e ordenação por recentes, populares e preço ascendente/descendente.
- Reutiliza cartões de produto e fluxo de pré-encomenda.
- A pesquisa de pessoas consulta perfis no Supabase.

#### Detalhe do produto
- Carrega um produto por identificador.
- Mostra fotografia, nome, preço/unidade, tipo, descrição, quantidade e localização.
- Ação de contacto do produtor abre o telefone do dispositivo quando existe contacto.
- Estados de carregamento, identificador inválido, produto indisponível e erro de consulta.

#### Perfil próprio
- Apresenta informação pessoal, papel, avatar/capa, estatísticas e ligações do utilizador.
- O código inclui edição de perfil, seleção de imagens, partilha/cópia de informação e opções de idioma/conta.
- O perfil também agrega áreas dependentes do papel, como produtos, pedidos, cargas e fichas; a visibilidade exata depende dos blocos condicionais do ecrã e deve ser validada por papel.

#### Perfil público
- Apresenta dados públicos do utilizador, papel, localização, produtos e estatísticas disponíveis.
- Pode iniciar contacto e consultar ligações.
- Deve respeitar RLS e não expor campos privados como documentos, credenciais ou dados de contacto não autorizados.

### 6.3 Operação e logística

#### Publicar produto
- Formulário com tipo/categoria, quantidade, unidade, data de colheita, preço, província, município, nome do fornecedor, contacto, descrição e acesso logístico.
- Distingue produtos frescos e processados.
- Inclui sugestões de produtos, seleção de imagens e seleção de localização no mapa.
- Usa guarda de papel para limitar acesso.
- A publicação depende de permissões de armazenamento, esquema da tabela `products` e políticas RLS.

#### Fichas de recebimento
- Fluxo em cinco etapas: negócio, produtos, local de entrega, contacto e confirmação.
- Permite identificar tipo de negócio, produtos, qualidade, embalagem, transporte, locais de entrega com coordenadas e contacto.
- Usa `MapPicker` para selecionar local.
- A interface contempla várias fichas/produtos e revisão antes de submeter.
- Confirmar no backend como as fichas e respetivos itens são persistidos.

#### Entregas / cargas
- Ecrã orientado a motoristas, com separação entre cargas disponíveis e cargas próprias.
- Apresenta estado, origem, destino, peso, preço e data.
- Usa localização do dispositivo, permissões de localização e tarefa de localização em segundo plano.
- Pode apresentar mapa/rota através de `MapViewer`.
- Estados de carga observados no código: disponível, aceite, em trânsito, entregue e cancelada.
- A aceitação e atualização de carga devem ser verificadas contra as regras do banco de dados e permissões de motorista.

#### Lista de rastreamentos
- Lista cargas em que o utilizador participa; administradores podem consultar o conjunto permitido.
- Atualiza por gesto de refresh e subscreve alterações em tempo real na tabela `freight_loads`.
- Encaminha para detalhe de rastreio.

#### Detalhe de rastreamento
- Consulta carga, posições GPS, eventos e liquidação.
- Apresenta mapa/rota, linha temporal de eventos e dados financeiros disponíveis.
- Eventos reconhecidos incluem aceitação, entrega do fornecedor ao motorista, recolha, entrega pelo motorista e confirmação do comprador.
- O acompanhamento em tempo real depende da publicação de posições, permissões de localização e políticas RLS.

#### Mapa
- Mapa com camadas para produtos, motoristas, agentes e agricultores.
- Declara pesquisa/listagem de entidades georreferenciadas, localização atual e apresentação de rotas.
- Usa WebView e serviços de mapa/geocodificação; a disponibilidade depende da rede e dos serviços externos configurados.
- Não assumir que uma linha entre dois pontos representa uma rota rodoviária válida sem confirmação do motor de rotas.

### 6.4 Comunicação e atividade

#### Notificações
- Lista notificações categorizadas como interesse, mensagem, produto e sistema.
- Usa ícones específicos, marcação de leitura, atualização e feedback visual/toast.
- Deve ser verificada a origem dos eventos, permissões de leitura e atualização e comportamento sem notificações.

#### Conversas
- Lista conversas, pesquisa por título e pesquisa de utilizadores para iniciar uma nova conversa.
- Consulta `conversations` e atualiza a lista através de Supabase Realtime.
- Utilizadores não autenticados são encaminhados para login.

#### Mensagens
- Apresenta histórico de mensagens e envio de texto.
- Permite selecionar e anexar documentos/imagens através de Document Picker.
- Declara bucket `chatfiles`, limite de ficheiro de 35 MB e limite de texto de 5.000 caracteres.
- Depende da existência do bucket, políticas de Storage e autorização de participantes.

### 6.5 Administração e serviços

#### Dashboard administrativo
- Protegido por `AdminOnly`.
- Abas declaradas: visão geral, utilizadores, produtos, pedidos, notificações, logística, pagamentos, pontos verdes e anúncios.
- Métricas declaradas incluem utilizadores, produtos, pedidos, notificações, cargas, pagamentos, métodos de pagamento, pontos verdes e pedidos de pontos verdes.
- Estados sem valor usam `—`; os dados devem vir de consultas reais, não de valores inventados.
- O carregamento e os erros de consulta devem ser expostos claramente.

#### Pontos Verdes
- Apresenta pontos físicos, produtos, stock e pedidos de reserva.
- Inclui mapa, seleção de produto, quantidade e data de levantamento (hoje/amanhã).
- O código declara operações administrativas de gestão de pontos/produtos e operações de reserva para compradores.
- A documentação funcional do repositório informa que a reserva é paga no ponto de levantamento; não é um fluxo de pagamento online.
- Depende das migrações SQL indicadas no README e das políticas RLS.

#### Gestão de anúncios
- Ecrã administrativo protegido por `AdminOnly`.
- Formulário para anunciante, título, descrição, URL e imagens.
- Permite selecionar até oito imagens; a publicação valida campos, URL e tamanho dos ficheiros.
- Inclui carregamento da lista e ativação/desativação de anúncios.

#### Segurança e histórico de pagamentos
- Rotas registadas no Stack.
- Os detalhes de campos, ações e estados destas duas telas ainda precisam de inspeção direta dos respetivos ficheiros para completar esta ficha. Não inferir funções a partir apenas do nome da rota.

## 7. Componentes de UI partilhados

| Componente | Responsabilidade observada |
|---|---|
| `BottomToolbar` | navegação inferior contextual ao papel |
| `ProductCard` | apresentação e ações de produto: galeria, like, comentários, mapa e pré-encomenda |
| `Icon` | catálogo interno de ícones |
| `ProcessingScreen` | estado global/local de carregamento |
| `GuestModeCard` | aviso e chamadas à ação do modo visitante |
| `RoleGuard` | restringe ecrãs/ações por papel |
| `FreightAccessGuard` | restringe áreas de logística |
| `AdminOnly` | restringe conteúdo administrativo |
| `MapPicker` | seleção de coordenadas/endereço sobre OpenStreetMap |
| `MapViewer` | apresentação de mapa e rota |
| `PaymentSheet` | interface de pagamento reutilizada no feed/pesquisa |
| `UserConnections` / `UserConnectionsSummary` | relações e resumo de ligações |
| `AgrilinkAdCard` | apresentação de anúncio |
| `GoogleLogo` | marca do fornecedor Google no login |

A lista é baseada nos imports e ficheiros examinados; não é ainda um inventário integral de todos os componentes em `src/components`.

## 8. Idiomas e conteúdo

O sistema i18n declara os idiomas `pt-AO`, `fr-CD` e `en-ZA`. A preferência é guardada no dispositivo através de AsyncStorage. O README do projeto descreve Português (Angola) e Français (Congo), pelo que a documentação e a interface devem ser alinhadas para esclarecer se English (Southern Africa) está ativo em todas as telas.

Nem todos os textos observados estão traduzidos: existem textos literais em português em ecrãs que também usam `t(...)`. A revisão de internacionalização deve procurar strings hardcoded, traduções ausentes e diferenças de pluralização/formatação.

## 9. Estados de interface a documentar e testar em cada ecrã

Cada tela deve ter comportamento explícito para:

- carregamento inicial e carregamento de ação;
- conteúdo carregado e lista vazia;
- erro de rede/servidor e ação de tentar novamente;
- sessão expirada e utilizador sem permissão;
- campos inválidos, validação em linha e submissão bloqueada;
- sucesso confirmado pelo backend;
- imagem/ficheiro sem permissão, inválido ou demasiado grande;
- localização negada, serviço desligado ou localização indisponível;
- estado offline e recuperação de ligação;
- texto longo, ecrã pequeno, teclado aberto e safe areas;
- prevenção de duplo toque e submissões concorrentes.

Não apresentar uma ação como concluída antes da confirmação do serviço responsável.

## 10. Integrações e pré-requisitos externos

| Integração | Uso no UI | Verificação necessária |
|---|---|---|
| Supabase Auth | login, cadastro, OTP, recuperação e sessão | providers, redirects/deep links, templates e SMTP/SMS |
| Supabase Database | perfis, produtos, conversas, notificações, cargas e dashboard | tabelas, colunas, índices, funções e RLS |
| Supabase Storage | imagens, capas, anúncios e anexos | buckets, limites, MIME types e políticas |
| Google OAuth | login Google | credenciais nativas/web e redirects |
| Expo Location | localização e rastreio | permissões Android/iOS e execução em background |
| OpenStreetMap / Nominatim | mapas, pesquisa e geocodificação | conectividade, atribuição, limites e política de uso |
| Leaflet via WebView | seleção/visualização de mapas | carregamento dos tiles e mensagens WebView |
| EAS Build | APK/AAB | conta Expo e variáveis de ambiente |

O cliente precisa de `EXPO_PUBLIC_SUPABASE_URL` e `EXPO_PUBLIC_SUPABASE_ANON_KEY` no ambiente de build. Nunca incluir chaves secretas de servidor no bundle mobile.

## 11. Observações e riscos identificados na leitura estática

1. **Dados de demonstração no feed:** `home.tsx` contém produtos mock e comentário explícito de fallback quando não existem produtos reais. Deve ser visível para QA e nunca confundido com dados reais.
2. **Papéis e navegação:** a ação central depende da resolução de papel. Testar contas com cada papel, papel ausente e sessão em troca de utilizador.
3. **Modo visitante:** várias rotas são explicitamente permitidas; ações que exigem conta devem encaminhar para autenticação sem perder o contexto.
4. **Dependências de schema:** telas de dashboard, mensagens, cargas, pagamentos e pontos verdes pressupõem tabelas/colunas específicas. Confirmar migrations e RLS antes de validar o UI.
5. **Serviços de mapa:** o MapPicker usa Nominatim e tiles OpenStreetMap; disponibilidade, limites e atribuição devem ser respeitados.
6. **Localização em segundo plano:** requer permissões e configuração nativa próprias; testar em dispositivos reais, não apenas simulador.
7. **Idioma:** os idiomas declarados em código e a descrição do README não estão totalmente alinhados.
8. **Rotas com ficha incompleta:** `/seguranca` e `/historicopagamentos` precisam de inspeção de implementação para esta documentação detalhar todos os controlos e estados.
9. **Acessibilidade:** a leitura estática não confirma labels de leitor de ecrã, foco, contraste efetivo, áreas de toque ou navegação por teclado. Validar em Android e iOS.
10. **Documentação não é certificação funcional:** nenhum fluxo é declarado “100% funcional” sem execução ponta a ponta com contas, dados e serviços reais.

## 12. Plano de validação visual e funcional

Executar a matriz abaixo em Android e iOS, com dados de teste controlados:

| Cenário | Perfis | Resultado esperado |
|---|---|---|
| Primeiro arranque, sem sessão | visitante novo | onboarding → login |
| Sessão existente | cada papel | onboarding → home ou destino previsto |
| Modo visitante | visitante | navegação permitida; ações restritas pedem login |
| Cadastro e OTP | cada tipo de conta | validação, confirmação e perfil persistido |
| Feed e pesquisa | visitante + autenticado | dados reais, vazio e erro tratados corretamente |
| Publicar produto | fornecedor/agente | validação, upload, persistência e produto visível |
| Ficha de recebimento | comprador | etapas, mapa, revisão e gravação |
| Cargas e rastreio | motorista + participantes | aceitar, atualizar estado, GPS e eventos autorizados |
| Mensagens | dois utilizadores | conversa, envio, anexos e realtime |
| Perfil | todos os papéis | edição, avatar/capa, idioma e logout |
| Administração | administrador e não administrador | acesso permitido/negado corretamente |
| Pontos Verdes | comprador + administrador | stock, reserva e atualização atómica |
| Rede e permissões | todos | estados de erro, retry e recuperação sem perda de dados |

### Critérios de conclusão

- todas as rotas do Stack têm descrição, capturas de ecrã e estados;
- todas as ações visíveis têm destino/efeito identificado e validação real;
- nenhum dado de demonstração aparece como dado real;
- permissões e RLS são testadas com utilizadores distintos;
- não há erros de TypeScript, lint ou dependências Expo;
- layouts foram verificados em ecrãs pequenos e grandes, com teclado e textos longos;
- acessibilidade básica validada com TalkBack e VoiceOver.

## 13. Comandos de verificação do projeto

Na raiz de `mobileagri`:

```bash
npm ci
npm run doctor
npm run check:expo-deps
npm run typecheck
npm run lint
npm start
```

O script `build:apk` executa verificações Expo e TypeScript antes do EAS Build.

## 14. Fontes de código consultadas

- `README.md`
- `package.json`
- `app.json`
- `src/app/_layout.tsx`
- `src/app/index.tsx`, `onboarding.tsx`
- ecrãs de autenticação e cadastro
- `src/app/home.tsx`, `search.tsx`, `product/[id].tsx`
- `src/app/profile.tsx`, `profile/[id].tsx`
- ecrãs de entregas, rastreamento, mapa e fichas
- ecrãs de mensagens, notificações e administração
- `src/components/BottomToolbar.tsx`, `ProductCard.tsx`, `MapPicker.tsx`
- `src/constants/roleActions.ts`, `src/constants/i18n.ts`
- `src/context/RoleContext.tsx`

---

**Próxima revisão:** completar a inspeção linha a linha de todas as rotas e componentes, acrescentar capturas reais de cada ecrã e registar para cada botão o handler, a tabela/RPC/serviço chamado, o estado de sucesso/erro e o teste correspondente.
