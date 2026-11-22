# AgriLink Mobile — alinhamento com o Design System Web

> Estado: plano de alinhamento; não representa implementação concluída.
> Repositório Mobile: `TheRuralTeam/mobileagri`.
> Referência Web: `TheRuralTeam/agrilinks`, especialmente `src/lib/brand.ts`, `src/index.css` e `tailwind.config.ts`.
> Especificação transversal: ver `docs/UNIFIED-DESIGN-SYSTEM.md` no repositório Web.

## Objetivo

O Mobile deve ser a expressão nativa do mesmo sistema visual AgriLink usado na Web. Não deve manter uma identidade independente. O layout pode adaptar-se a Android/iOS, mas a marca, tokens, tipografia, ícones, componentes conceptuais, linguagem e estados devem coincidir.

## Diferenças confirmadas na leitura estática

| Tema | Web (referência atual) | Mobile (estado observado) |
|---|---|---|
| Verde principal | `#2c863b` em `src/lib/brand.ts` | `#218B4A` em `roleActions.ts` e outros componentes |
| Tipografia | Plus Jakarta Sans | confirmar fontes carregadas e escala em todas as rotas |
| Texto/fundo | branco e preto na fundação CSS atual | `#16231C`, `#F3F6F2`, `#FFFEFC` e variantes locais |
| Estilos | tokens CSS/Tailwind e componentes React | NativeWind, StyleSheet, inline styles e `COLORS` |
| Estados semânticos | aliases success/warning/destructive atualmente convergem para verde | validar cores e rótulos efetivos no UI nativo |

A Web é a referência inicial da marca porque já contém `src/lib/brand.ts`. A paleta final precisa de auditoria de contraste e semântica; não copiar cegamente aliases atuais onde sucesso, aviso e erro são indistinguíveis.

## Requisitos de migração Mobile

1. Consumir os mesmos tokens semânticos e a mesma versão do manifesto transversal.
2. Substituir cores locais de marca por tokens, incluindo `ACTION_COLOR`, `COLORS` e estilos locais, preservando os estados semânticos.
3. Carregar e aplicar Plus Jakarta Sans de forma consistente, incluindo pesos realmente disponíveis no bundle.
4. Consolidar raios, espaçamentos, tipografia, dimensões de ícones e alvos de toque.
5. Definir componentes nativos equivalentes aos contratos Web: botões, campos, cartões de produto, etiquetas de estado, listas, diálogos/folhas, feedback, loading, empty, error e offline.
6. Preservar navegação e interação nativas por papel; não copiar literalmente a sidebar ou hover da Web.
7. Garantir que cada componente continua a chamar a ação real existente e aguarda confirmação do backend.

## Responsividade mobile

A validação deve incluir Android e iOS, telemóveis compactos e grandes e tablets. Verificar safe areas, teclado, orientação, texto ampliado, descrições longas, imagens, listas, modais, mapas e alvos de toque. Não fixar dimensões baseadas apenas no tamanho de um único telefone.

## Plano de trabalho

1. Inventariar todos os usos de cores, fontes, raios, espaçamentos e componentes em `src/app` e `src/components`.
2. Comparar cada rota mobile com a área equivalente da Web.
3. Introduzir o adaptador de tokens mobile a partir do manifesto partilhado.
4. Migrar primeiro Feed → detalhe de produto → contacto/pré-encomenda.
5. Validar screenshots lado a lado e testes de regressão.
6. Migrar os restantes domínios sem remover funcionalidades.
7. Validar TalkBack, VoiceOver, contraste, texto ampliado, typecheck, lint e build.

## Guardrails

- Não alterar Supabase, schema, RLS, RPC, Auth ou regras de negócio numa migração visual.
- Não substituir dados reais por mocks nem esconder estados de erro.
- Não remover fluxos existentes.
- Não introduzir serviços pagos.
- Não fazer merge/deploy automaticamente.
- A documentação anterior `docs/UI-MOBILE.md` continua a ser o inventário funcional inicial; esta especificação acrescenta a obrigação de alinhamento visual com a Web.
