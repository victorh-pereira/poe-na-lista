# Põe na Lista — Especificação de produto e arquitetura

**Status:** aguardando aprovação
**Data:** 23/10/2026
**Autor:** Victor Hugo Pereira (produto) + assistência técnica
**Versão:** 1.0

---

## 1. Visão do produto

### 1.1 Problema

Em uma casa, saber **o que está faltando** e **quanto a compra vai custar** depende de memória e de um bloco de notas compartilhado. O resultado previsível: item esquecido, compra repetida, ida extra ao mercado e nenhuma noção do total antes do caixa.

### 1.2 Objetivo

Um app Android de uso familiar que responda a três perguntas, nesta ordem:

1. **O que eu preciso comprar?**
2. **O que eu tenho em casa?**
3. **Quanto eu gastei e como o preço evoluiu?**

### 1.3 Público

Duas casas, três aparelhos, uso estritamente pessoal:

| Casa | Pessoas | Aparelhos | Conta |
|---|---|---|---|
| Casa do Victor | Victor + esposa | 2 Galaxy A54 | uma conta compartilhada |
| Casa da mãe | mãe | 1 Galaxy A54 | conta própria |

Não há publicação na Play Store. Distribuição por APK, instalação manual.

### 1.4 Proposta de valor

- **Substitui o bloco de notas** com captura mais rápida e catálogo com foto.
- **Uma foto vale mais que um nome**: quem vai ao mercado vê exatamente qual produto comprar.
- **Total em tempo real** e teto de gasto, sem app de finanças.
- **Funciona no mercado sem internet**, inclusive com o app fechando no meio da compra.

### 1.5 Métricas de sucesso (pessoais, não analytics)

- Registrar um item em **menos de 5 segundos**.
- Fazer uma compra completa **sem internet**, sem perder dado.
- O total do app bater com o cupom fiscal (ou a diferença ser explicada).
- O app ser usado em **3 compras consecutivas** sem volta ao bloco de notas.

### 1.6 Não-objetivos

- App de finanças pessoais (orçamento mensal, categorias de gasto, metas).
- Controle de estoque com quantidade em casa.
- Receitas, lista social, compartilhamento entre casas.
- Versão web, iOS ou publicação em loja.
- Notificações e automações.
- Código de barras (roadmap).

### 1.7 Anti-objetivos (o que faria o produto fracassar)

- Ser mais lento para anotar do que o bloco de notas.
- Perder dados por falha de sincronização.
- Exigir internet ou login no corredor do supermercado.
- Mostrar total errado.

---

## 2. Restrições e premissas

| Item | Valor |
|---|---|
| Plataforma | Android, `minSdk 26`, `targetSdk` estável atual |
| Aparelho de referência | Samsung Galaxy A54 (Android 13+) |
| Distribuição | APK assinado, instalação manual em até 3 aparelhos |
| Custo | R$ 0 (free tier do Supabase; sem loja) |
| Prazo | Sem prazo; entregas verticais |
| Idioma | pt-BR; moeda BRL; sem multi-moeda |
| Backend | Supabase (Postgres, Auth, Storage) |
| Contas | Uma conta = uma casa |

---

## 3. Abordagens consideradas

A escolha de tecnologia foi discutida em quatro opções. O registro é mantido porque a decisão foi do product owner, contra a recomendação técnica, e isso tem consequências no cronograma.

| Opção | Vantagens | Desvantagens | Esforço até o MVP |
|---|---|---|---|
| **Kotlin + Compose** (escolhida) | Stack oficial, controle total, melhor portfólio Android | Curva de aprendizado alta; 4–6 meses no ritmo do autor | 150–250 h |
| React Native + Expo (recomendada tecnicamente) | Reaproveita React/TS e o Supabase JS já conhecido | Dependência do ecossistema Expo/EAS | 60–100 h |
| Flutter | Ótimo SDK Supabase, UI consistente | Dart do zero, sem reaproveitamento | 80–130 h |
| Capacitor | Reaproveita 100% do stack web | Webview, câmera inferior, portfólio mais fraco | 40–70 h |

**Decisão:** Kotlin + Jetpack Compose, por escolha do product owner, priorizando aprendizado e portfólio Android sobre velocidade de entrega.

**Mitigação do risco:** construção em fatias verticais, com app utilizável desde a primeira etapa, e sincronização por último.

---

## 4. Escopo funcional

### 4.1 Contas e casas

- Login por e-mail e senha (Supabase Auth), sessão persistente até logout.
- Duas contas: uma por casa. A esposa usa a mesma conta do Victor.
- Contas salvas no aparelho, com troca em um toque (senha apenas no primeiro login).
- Modo visitante: dados apenas locais, sem sincronização, com aviso explícito de que desinstalar apaga tudo. Ao criar conta, oferecer migração dos dados locais.
- Sem bloqueio próprio do app; vale o bloqueio do celular.

### 4.2 Produtos e categorias

- Cadastro completo (nome, categoria, unidade, foto, observação, preço de referência derivado) e captura rápida.
- Categorias pré-carregadas, editáveis, com ordem definida pelo usuário.
- Produto nunca é excluído: apenas inativado, preservando o histórico.
- Marca e tamanho pertencem à compra, não ao produto.

### 4.3 Captura rápida

- Campo único na tela inicial: digitar, pressionar Enter e o item entra na próxima compra.
- Autocomplete sobre o catálogo, com aviso de nome parecido.
- Produto inexistente é criado com nome e categoria "Outros"; completar depois.

### 4.4 Próxima compra

- Uma lista ativa automática por conta.
- Recebe automaticamente todo produto marcado como "acabou".
- Durante uma compra em andamento não existe lista ativa: marcar "acabou" adiciona o item **diretamente à compra em andamento**, como pendente.
- O app só abre direto na tela de compra no aparelho que iniciou aquela compra. Nos demais, a tela inicial mostra um aviso "Compra em andamento", com acesso em um toque.
- Itens agrupados por categoria; ordem das categorias definida pelo usuário; itens por ordem de inserção.
- Item removido da lista não altera o estado do produto; ele aparece em "Fora da lista".

### 4.5 Compra em andamento

- Iniciada por "Iniciar compra", com teto opcional.
- Ações por item: comprado (abre teclado com último preço), não tinha, editar quantidade/marca/observação, remover.
- Item comprado vai para "No carrinho", com desfazer.
- Botão "+" sempre visível.
- Total e barra do teto sempre visíveis.
- Estado preservado se o app fechar, a tela apagar ou a internet cair.

### 4.6 Preços e cálculos

- Preço digitado item a item, com máscara de moeda e teclado numérico.
- Preço pré-preenchido com o último preço pago, rotulado como sugestão.
- Comparação discreta com o último preço ("última: R$ 22,90 ↑7%").
- Teto por compra: aviso em 80%, vermelho acima de 100%, nunca bloqueia.
- Total do cupom opcional no fim, com diferença exibida.

### 4.7 Finalização

- Tela de revisão obrigatória: comprados, não encontrados, sem preço.
- Comprados voltam o produto para "em estoque"; não comprados entram na próxima lista.
- Compra vai para o histórico com snapshot imutável de nomes e preços.
- Reabrir a última compra finalizada para corrigir erro.
- Apagar compra do histórico com confirmação dupla (soft delete).

### 4.8 Histórico e gráficos

- Lista de compras com data, total e quantidade de itens.
- Gasto do mês no topo do histórico.
- Detalhe da compra com itens e preços.
- Gráfico de linha do preço por produto; R$/kg para produtos pesados.
- Sem exportação no MVP.

### 4.9 Configurações

- Ordem das categorias; gerenciar categorias.
- Tema claro/escuro/sistema.
- Contas salvas e troca de conta.
- Sair.
- Sobre o app e versão.

---

## 5. Regras de negócio

### Produtos

| ID | Regra |
|---|---|
| RN01 | Produto = identidade fixa (nome, categoria, unidade, foto, observação, ativo) + estado da casa (`em_estoque` / `acabou`). |
| RN02 | Produto nunca é excluído. Inativar remove do catálogo ativo e o histórico permanece. |
| RN03 | Marca e tamanho são atributos da compra. O último valor usado é sugerido, nunca imposto. |
| RN04 | Um produto pode aparecer no máximo uma vez por lista. |
| RN05 | Unidade vem de lista fixa (un, kg, g, L, ml, pacote, caixa, dúzia, bandeja). Produto pesado é registrado como `1 un × preço da etiqueta`, com peso opcional em gramas. |
| RN06 | Categorias vêm pré-carregadas e podem ser criadas, renomeadas, reordenadas e desativadas. |
| RN06a | Uma categoria só pode ser desativada se não tiver produtos ativos; o app oferece mover os produtos para outra categoria. "Outros" não pode ser desativada. |
| RN07 | Ao digitar um nome, autocomplete sugere produtos existentes e avisa quando há nome muito parecido. Não bloqueia a criação. |
| RN08 | Marcar "acabou" cria um item na lista ativa. Se o produto já está na lista, não duplica. |
| RN08a | Se existir uma compra em andamento, "acabou" adiciona o item a ela, como pendente, em vez de criar lista ativa. |
| RN09 | Remover um item da lista não altera o estado do produto: ele continua "acabou" e aparece em "Fora da lista". |
| RN10 | Marcar "em estoque" com o produto na lista pede confirmação e remove o item. |
| RN11 | Produto inativo não aparece na captura nem na busca; permanece no histórico. Ao inativar, item da lista ativa é removido com aviso. |

### Listas e compras

| ID | Regra |
|---|---|
| RN12 | Existe no máximo uma lista `ativa` e uma `em_compra` por conta. |
| RN12a | A compra em andamento é compartilhada pela conta: os dois aparelhos podem abrir, editar e finalizar. Cada aparelho guarda localmente se foi ele que iniciou a compra, e só nesse caso abre direto na tela de compra. |
| RN13 | Nome da compra é automático: "Compra de DD/MM". |
| RN14 | Editar quantidade planejada, marca ou observação de um item vale apenas para aquela compra. |
| RN15 | Cancelar uma compra devolve os itens à lista ativa, descarta preços e não gera histórico. |
| RN16 | Finalizar exige passar pela tela de revisão. |
| RN17 | Ao finalizar: comprados → produto `em_estoque`; não encontrados e pendentes → próxima lista ativa; removidos não voltam. |
| RN18 | Só a última compra finalizada pode ser reaberta. Reabrir reverte o estado dos produtos e devolve a compra para `em_compra`. |
| RN19 | Apagar compra do histórico exige confirmação dupla e é soft delete. |
| RN20 | Teto é opcional e por compra. Aviso em 80%; acima de 100% fica vermelho. Nunca bloqueia. |
| RN21 | Total do cupom é opcional. A diferença é exibida e nunca bloqueia a finalização. |
| RN22 | "Não tinha" mantém o item fora do total e o devolve à próxima lista. |

### Dinheiro

| ID | Regra |
|---|---|
| RN23 | Dinheiro é armazenado em centavos inteiros; peso em gramas. Nunca ponto flutuante. |
| RN24 | O subtotal é a fonte de verdade. O preço unitário é derivado (`subtotal / quantidade`) quando necessário. |
| RN25 | Item sem preço gera total **parcial**: "R$ 187,40 + 1 item sem preço". Nunca soma zero silenciosamente. |
| RN26 | O campo de preço abre com o último preço pago, rotulado como "último preço", e exige confirmação. |
| RN26a | Em produto vendido por peso (kg/g/L/ml), o campo **não** é pré-preenchido com o total anterior: mostra "último: R$ X,XX/kg" como referência e o usuário digita o total da etiqueta. |
| RN27 | A comparação de preço usa o último preço do mesmo produto. Se o tamanho mudou, o tamanho é exibido junto. Para produtos com peso, a comparação é feita em R$/kg. |
| RN28 | R$/kg = `round(preco_centavos × 1000 / peso_gramas)`. |
| RN29 | Promoção ("leve 3, pague 2") é registrada como total pago; o unitário exibido é a média. |
| RN30 | Quantidade padrão é 1, mínimo 1. Preço zero é permitido; negativo, proibido. |

### Contas e sincronização

| ID | Regra |
|---|---|
| RN31 | Uma conta = uma casa. Todo dado pertence a um `user_id`. |
| RN32 | Sessão persiste até logout explícito. |
| RN33 | Contas ficam salvas no aparelho; troca em um toque, sem redigitar senha. |
| RN33a | O banco local é multi-tenant: toda consulta filtra pelo `user_id` ativo. O cache das contas já usadas é preservado, então a troca de conta funciona offline. |
| RN34 | Visitante não sincroniza. Ao criar conta, a migração dos dados locais é oferecida. |
| RN35 | Conflito na mesma linha é resolvido por última escrita que chega ao servidor (LWW), por linha. |
| RN36 | Exclusões são soft delete (tombstone) e sincronizam como qualquer alteração. |
| RN37 | O app é 100% funcional offline, exceto o primeiro login e a sincronização. |
| RN38 | Nenhuma notificação no MVP. |

---

## 6. Jornadas do usuário

### 6.1 Primeiro acesso

1. Instala o APK, abre o app.
2. Escolhe "Criar conta" ou "Usar sem conta".
3. Se criar conta: e-mail e senha; sessão salva.
4. Vê o app vazio com o botão "Cadastrar primeiro produto" e o campo "O que acabou?".
5. Cadastra de 5 a 10 produtos (nome, categoria, unidade; foto opcional).
6. Ajusta a ordem das categorias, se quiser.

### 6.2 Rotina da casa

1. Percebe que o arroz acabou.
2. Abre o app, digita "arr" no campo de captura, escolhe "Arroz 5 kg".
3. O item entra na próxima compra e o produto passa a "acabou".
4. Opcional: tira a foto da embalagem na tela do produto.

### 6.3 Compra no mercado

1. Abre o app, toca em "Iniciar compra", define o teto (opcional).
2. Vê os itens por categoria, com miniatura da foto.
3. No corredor: toca no item → "Comprado" → confirma o preço pré-preenchido → item vai para "No carrinho".
4. Não achou o extrato → "Não tinha".
5. Achou algo fora da lista → "+" → captura rápida.
6. O total e a barra do teto atualizam a cada item.
7. App fecha ou perde internet → ao reabrir, volta exatamente onde parou.

### 6.4 Finalização

1. Toca em "Finalizar".
2. Revisão: total, teto, itens comprados, não encontrados, itens sem preço.
3. Opcional: digita o total do cupom e vê a diferença.
4. Confirma.
5. Comprados voltam para "em estoque"; não comprados entram na próxima lista.
6. Compra aparece no histórico.

### 6.5 Consulta de preço

1. Abre o histórico.
2. Vê o gasto do mês.
3. Entra em um produto e vê o gráfico de preço ao longo das compras.

### 6.6 Segunda casa

1. No celular do Victor: menu de contas → "Adicionar conta" → login da conta da mãe.
2. Troca entre as duas em um toque.
3. Cada conta tem catálogo, lista e histórico próprios.

---

## 7. Mapa de telas e navegação

Navegação principal por **três abas inferiores**: Início, Produtos, Histórico.

| # | Tela | Conteúdo | Ações principais |
|---|---|---|---|
| 1 | **Login / Boas-vindas** | E-mail, senha, "usar sem conta" | Entrar, criar conta, visitante |
| 2 | **Início** | Próxima compra (contador), campo de captura, itens por categoria com foto, "Fora da lista" recolhida | Capturar, iniciar compra, abrir item |
| 3 | **Produtos** | Busca, filtros (acabou / em estoque / inativos), lista | Novo produto, editar |
| 4 | **Produto** | Foto, nome, categoria, unidade, observação, ativo, estado, último preço | Salvar, trocar foto, inativar |
| 5 | **Compra** | Teto, total, barra, pendentes por categoria, "No carrinho" no rodapé | Comprar, não tinha, editar, remover, adicionar |
| 6 | **Finalizar** | Resumo, diferença do cupom, itens sem preço | Finalizar, voltar |
| 7 | **Histórico** | Gasto do mês, compras finalizadas | Abrir compra, apagar |
| 8 | **Detalhe da compra** | Itens com preço e quantidade | Reabrir (última), apagar |
| 9 | **Produto — histórico de preço** | Gráfico de linha, tabela de compras | Voltar |
| 10 | **Configurações** | Categorias e ordem, tema, contas, sair, sobre | — |
| 11 | **Contas** | Contas salvas, adicionar, trocar, remover | Trocar conta |

**Regras de navegação**
- Abrir o app com compra em andamento leva direto à tela 5 **apenas no aparelho que iniciou a compra**; nos demais, a tela inicial mostra o aviso "Compra em andamento".
- A tela 5 não tem barra de abas; sair exige "voltar" explícito, para evitar toque acidental.
- Captura rápida disponível nas telas 2 e 5.

---

## 8. Requisitos funcionais e critérios de aceitação

| ID | Requisito | Critério de aceitação |
|---|---|---|
| RF01 | Cadastrar produto | Salva com nome, categoria e unidade; aparece na busca; persiste após reiniciar o app |
| RF02 | Editar produto | Alterações refletem na lista ativa sem duplicar item |
| RF03 | Inativar produto | Some da busca e da captura; histórico intacto |
| RF04 | Foto do produto | Tirar da câmera ou escolher da galeria; salva comprimida; aparece na lista |
| RF05 | Categorias padrão | Primeiro acesso já tem categorias utilizáveis |
| RF06 | Reordenar categorias | Nova ordem persiste e reflete na lista |
| RF07 | Captura rápida | Digitar + Enter cria item em < 1 s, com produto existente ou novo |
| RF08 | Autocomplete | Digitar 2 letras sugere produtos; aviso de nome parecido |
| RF09 | Marcar "acabou" | Item aparece na lista ativa em < 300 ms |
| RF10 | Remover item | Item sai da lista; produto continua "acabou"; aparece em "Fora da lista" |
| RF11 | Iniciar compra | Cria estado `em_compra`; teto opcional |
| RF12 | Marcar comprado | Abre teclado com último preço; confirmar move para "No carrinho" |
| RF13 | Não tinha | Item marcado, fora do total, volta na próxima lista |
| RF14 | Adicionar durante a compra | Botão "+" cria item já pendente |
| RF15 | Total em tempo real | Soma atualiza a cada preço confirmado |
| RF16 | Teto | Barra e cor mudam em 80% e 100%; não bloqueia |
| RF17 | Item sem preço | Total exibido como parcial e item listado na revisão |
| RF18 | Finalizar | Revisão obrigatória; estados aplicados só após confirmar |
| RF19 | Cupom | Diferença calculada e exibida; não bloqueia |
| RF20 | Reabrir última compra | Reverte estados e devolve a compra para `em_compra` |
| RF21 | Histórico | Compra finalizada listada com data, total e nº de itens |
| RF22 | Apagar compra | Confirmação dupla; compra some da lista e do total do mês |
| RF23 | Gráfico de preço | Linha por compra; R$/kg quando há peso |
| RF24 | Gasto do mês | Soma das compras finalizadas do mês corrente |
| RF25 | Trocar de conta | Um toque; dados da outra conta carregam sem reiniciar |
| RF26 | Visitante | Funciona offline; aviso de perda ao desinstalar; migração ao criar conta |
| RF27 | Offline | Todas as operações acima funcionam em modo avião, exceto login e sync |
| RF28 | Retomada | Fechar o app no meio da compra e reabrir restaura o estado |
| RF29 | Sincronização | Alteração em um aparelho aparece no outro após abrir/atualizar |
| RF30 | Tema | Claro, escuro e sistema, sem reiniciar |

---

## 9. Requisitos não funcionais

| ID | Requisito | Meta |
|---|---|---|
| RNF01 | Abertura do app | Lista visível em < 1 s (aparelho de referência) |
| RNF02 | Feedback de ação | < 100 ms para marcar item (local) |
| RNF03 | Tamanho do APK | < 25 MB |
| RNF04 | Offline | 100% das funções de compra sem rede |
| RNF05 | Resiliência | Nenhuma perda de dado em fechamento abrupto ou morte de processo |
| RNF06 | Acessibilidade | Alvos ≥ 48 dp; fonte ≥ 16 sp; contraste WCAG AA; conteúdo descritivo em ícones |
| RNF07 | Uso com uma mão | Ações principais no terço inferior da tela |
| RNF08 | Bateria | Sem polling; sync só em foreground e via WorkManager com restrição de rede |
| RNF09 | Privacidade | Sem analytics, sem rastreamento, sem terceiros além do Supabase |
| RNF10 | Idioma | pt-BR, formatação `R$ 1.234,56` e data `dd/MM/yyyy` |
| RNF11 | Compatibilidade | Android 8.0+ (API 26); testado em Galaxy A54 |
| RNF12 | Manutenção | Módulo único, pacotes por feature, regras de dinheiro cobertas por teste |

---

## 10. Modelo de dados

### 10.1 Entidades (Postgres / Supabase)

Todas as tabelas têm `id uuid` gerado no cliente (offline-first), `user_id uuid`,
`created_at timestamptz`, `updated_at timestamptz` (definido pelo servidor) e,
quando aplicável, `deleted_at timestamptz`.

**profiles**
| Coluna | Tipo | Observação |
|---|---|---|
| id | uuid PK | = `auth.users.id` |
| nome_casa | text | "Casa do Victor", "Casa da mãe" |
| criado_em | timestamptz | |

**categories**
| Coluna | Tipo | Observação |
|---|---|---|
| id | uuid PK | |
| user_id | uuid | FK profiles |
| nome | text | |
| ordem | int | |
| ativo | boolean | default true |
| deleted_at | timestamptz | |

**products**
| Coluna | Tipo | Observação |
|---|---|---|
| id | uuid PK | |
| user_id | uuid | FK profiles |
| nome | text | |
| categoria_id | uuid | FK categories |
| unidade | text | enum fixo |
| foto_path | text | caminho no Storage |
| observacao | text | curta |
| ativo | boolean | default true |
| status_casa | text | `em_estoque` \| `acabou` |
| deleted_at | timestamptz | sempre null (produto não é excluído) |

**lists**
| Coluna | Tipo | Observação |
|---|---|---|
| id | uuid PK | |
| user_id | uuid | FK profiles |
| nome | text | "Compra de 23/10" |
| status | text | `ativa` \| `em_compra` \| `finalizada` \| `cancelada` |
| teto_centavos | int | nullable |
| total_centavos | int | nullable; parcial se houver item sem preço |
| total_cupom_centavos | int | nullable |
| iniciada_em | timestamptz | |
| finalizada_em | timestamptz | |
| reaberta_em | timestamptz | nullable |
| deleted_at | timestamptz | soft delete de histórico |

**list_items**
| Coluna | Tipo | Observação |
|---|---|---|
| id | uuid PK | |
| user_id | uuid | desnormalizado para RLS simples |
| list_id | uuid | FK lists |
| product_id | uuid | FK products |
| quantidade_planejada | int | default 1 |
| status | text | `pendente` \| `no_carrinho` \| `nao_encontrado` \| `removido` |
| marca | text | atributo da compra |
| tamanho | text | atributo da compra |
| observacao | text | |
| ordem | int | posição na categoria |
| quantidade_comprada | int | nullable |
| preco_unitario_centavos | int | nullable; derivado quando promoção |
| subtotal_centavos | int | nullable; **fonte de verdade** |
| peso_gramas | int | nullable; só para R$/kg |
| nome_snapshot | text | congelado ao finalizar |
| categoria_snapshot | text | congelado ao finalizar |
| unidade_snapshot | text | congelado ao finalizar |
| deleted_at | timestamptz | |

**local: sync_outbox** (só no Room)
| Coluna | Tipo |
|---|---|
| id | text PK |
| entity | text (`products`, `lists`, `list_items`, `categories`) |
| entity_id | text |
| op | text (`upsert`, `delete`) |
| payload | text (JSON) |
| attempts | int |
| last_error | text |
| created_at | long |

**local: sync_state**
| Coluna | Tipo |
|---|---|
| entity | text PK |
| last_pull_at | long |

### 10.2 Integridade

| Regra | Implementação |
|---|---|
| Uma lista ativa por conta | índice único parcial `lists(user_id) WHERE status='ativa' AND deleted_at IS NULL` |
| Uma compra em andamento por conta | índice único parcial `lists(user_id) WHERE status='em_compra' AND deleted_at IS NULL` |
| Um produto por lista | índice único parcial `list_items(list_id, product_id) WHERE deleted_at IS NULL` |
| Valores não negativos | `CHECK (teto_centavos IS NULL OR teto_centavos >= 0)`, idem subtotal |
| Quantidade mínima | `CHECK (quantidade_planejada >= 1)` |
| Status válidos | `CHECK (status IN (...))` |

### 10.3 Índices

- `products(user_id, ativo, status_casa)`
- `products(user_id, nome)`
- `list_items(list_id, status, ordem)`
- `list_items(product_id)` — para o gráfico de preço
- `lists(user_id, status)`
- `lists(user_id, finalizada_em DESC)`

### 10.4 Segurança (RLS)

- RLS ativa em todas as tabelas.
- Política única por tabela: `user_id = auth.uid()` para `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- `profiles`: `id = auth.uid()`.
- Storage: bucket privado `fotos`; política de objeto exige que o primeiro segmento do caminho seja `auth.uid()`.
- A chave `anon` pode ficar no app; a `service_role` nunca.

### 10.5 Sincronização de dados

- Room é a fonte da verdade na UI.
- Toda escrita local grava a linha e enfileira na `sync_outbox`, na mesma transação.
- `updated_at` é definido por trigger no servidor; o cursor de pull usa esse valor.
- Pull: por entidade, `WHERE updated_at > cursor`, aplicado em transação; linhas locais "sujas" não são sobrescritas.
- Conflito: LWW por linha. A UI mostra "atualizado há X" e permite atualização manual.
- Exclusões: `deleted_at` (tombstone), nunca `DELETE`.

---

## 11. Arquitetura técnica

### 11.1 Stack

| Camada | Tecnologia |
|---|---|
| Linguagem | Kotlin 2.x |
| UI | Jetpack Compose + Material 3 |
| Navegação | Navigation Compose (rotas type-safe) |
| Estado | ViewModel + `StateFlow` + `collectAsStateWithLifecycle` |
| Banco local | Room + migrations |
| Preferências | DataStore |
| Câmera | `ActivityResultContracts.TakePicture` (câmera do sistema) |
| Galeria | Photo Picker (`PickVisualMedia`) |
| Imagens | Coil 3 + compressão antes de persistir |
| Backend | Supabase via `supabase-kt` (Auth, PostgREST, Storage) isolado por interface |
| Sync | WorkManager + outbox |
| Serialização | kotlinx.serialization |
| Datas | `java.time` (nativo no minSdk 26) |
| DI | Container manual (`AppContainer`) |
| Testes | JUnit, Turbine, Room em memória, Compose UI Test |
| Build | Gradle KTS + version catalog; APK assinado |

**Decisões deliberadas de simplificação**
- Sem Hilt no MVP: menos uma ferramenta e um gerador de código.
- Sem CameraX: a câmera do sistema resolve uma foto por produto com muito menos código.
- Módulo único, pacotes por feature. Multi-módulo fica para quando doer.
- `supabase-kt` (comunitário) atrás de interfaces `AuthRepository` e `SyncRepository`; fallback para REST/Ktor sem tocar na UI.

### 11.2 Camadas

```
UI (Compose)  →  ViewModel  →  Repository  →  Room (verdade local)
                                     ↘       →  RemoteDataSource (Supabase)
                                     ↘       →  sync_outbox → SyncWorker
```

- **UI**: composables sem regra de negócio.
- **ViewModel**: estado da tela, eventos, validações de formulário.
- **Repository**: regras de negócio, transações, enfileiramento de sync.
- **Room**: persistência e consultas reativas (`Flow`).
- **RemoteDataSource**: chamadas ao Supabase, sem regra de negócio.
- **SyncWorker**: processa a outbox e faz pull.

### 11.3 Estrutura de pacotes

```
com.victorh.poenalista
├── app/            Application, AppContainer, MainActivity, navegação
├── core/
│   ├── model/      entidades de domínio, enums
│   ├── database/   Room, DAOs, migrations, converters
│   ├── network/    Supabase, DTOs, AuthRepository, SyncRepository
│   ├── sync/       Outbox, SyncWorker, merge
│   ├── money/      Centavos, formatação, cálculos
│   ├── images/     compressão, armazenamento de foto
│   └── designsystem/ tema, componentes
└── feature/
    ├── auth/
    ├── home/
    ├── products/
    ├── list/
    ├── shopping/
    ├── history/
    └── settings/
```

### 11.4 Tratamento de erros

- Erros de rede nunca bloqueiam a UI: a operação é local e a outbox tenta depois.
- Falha de sync é silenciosa para o usuário, exceto por um indicador discreto ("não sincronizado").
- Erros de validação aparecem no campo, em texto.
- Erros inesperados: snackbar com ação "tentar de novo", e log local (sem telemetria).

---

## 12. Autenticação e modo visitante

### 12.1 Fluxos

| Fluxo | Comportamento |
|---|---|
| Criar conta | E-mail + senha; perfil criado com nome da casa; sessão salva |
| Login | E-mail + senha; pull inicial completo |
| Sessão | Persistida em armazenamento seguro; renovação automática do token |
| Trocar conta | Contas salvas; troca em um toque; os dados em cache da conta aparecem imediatamente e o pull acontece quando houver rede |
| Sair | Remove a sessão apenas daquela conta e apaga o cache local dela; as demais contas permanecem |
| Recuperar senha | Link por e-mail (Supabase Auth) |
| Visitante | Sem conta; dados locais; aviso de perda ao desinstalar |
| Migração visitante → conta | Ao criar conta, oferecer "enviar meus dados"; copia linhas locais para a conta |

### 12.2 Regras

- Uma conta por casa; a esposa compartilha a conta do Victor.
- A conta da mãe pertence a ela: recuperação de senha depende do e-mail dela. **Pendente:** confirmar se ela tem e-mail ativo; caso contrário, usar um e-mail do Victor.
- Sem modo visitante em dois aparelhos: visitante é local por definição.
- Sessão de visitante não faz pull nem push.

---

## 13. Funcionamento offline e sincronização

### 13.1 Princípios

1. **Local primeiro**: toda leitura e escrita acontece no Room; a rede é opcional.
2. **Outbox transacional**: dado e intenção de sincronizar são gravados juntos.
3. **Pull incremental**: cursor por entidade, baseado em `updated_at` do servidor.
4. **Conflito por linha**: LWW, documentado e visível.
5. **Fotos independentes**: a foto sobe depois; o produto não espera.

### 13.2 Gatilhos de sync

- Ao abrir o app.
- Ao voltar do background.
- Ao entrar na tela de compra.
- Antes de abrir a revisão de finalização.
- 2 segundos após uma alteração (debounce).
- Ao concluir uma compra.
- Manual: puxar para atualizar.
- WorkManager com restrição de rede para retentativas.

### 13.3 Cenários críticos

| Cenário | Comportamento |
|---|---|
| Modo avião no mercado | Tudo funciona; outbox cresce; sync ao voltar a rede |
| App morto no meio da compra | Estado reconstruído do Room; volta para a tela de compra |
| Duas pessoas marcam o mesmo item | Última escrita vence; refresh mostra o estado final |
| Duas pessoas editam itens diferentes | Sem conflito |
| Foto tirada offline | Salva local; upload pendente; selo discreto |
| Primeiro login | Pull completo (dataset pequeno, < 1 s) |
| Conta trocada | Dados da conta anterior saem da UI; cache local isolado por `user_id` |
| Supabase pausado | App local funciona; sync falha com aviso; reativar no painel |

### 13.4 Fila e retentativa

- FIFO por entidade.
- Backoff exponencial: 30 s, 2 min, 10 min, 1 h, máximo 6 h.
- Após 10 falhas, o item fica marcado como erro e aparece em Configurações → "pendências de sincronização".

### 13.5 Consistência entre os dois aparelhos

Não há tempo real. A defasagem aceitável é de **segundos a poucos minutos**, coberta por:

- pull ao abrir, ao voltar do background e ao entrar na tela de compra;
- indicador discreto "atualizado há X";
- ao abrir a revisão de finalização, o app faz um pull e compara o total do servidor com o local; se divergir, mostra "a compra foi alterada em outro aparelho" e exige confirmação antes de finalizar.

Se essa defasagem se mostrar incômoda no uso real, a evolução é Supabase Realtime apenas para a lista em andamento — fora do MVP.

---

## 14. Fotos

| Aspecto | Decisão |
|---|---|
| Quantidade | Uma foto por produto |
| Origem | Câmera do sistema ou galeria |
| Momento | No ato da compra ou depois, na tela do produto |
| Permissões | Nenhuma em runtime: `TakePicture` sem declarar `CAMERA`; Photo Picker no Android 13+ |
| Compressão | Maior lado 1080 px, JPEG qualidade 80, meta < 300 KB |
| Orientação | EXIF aplicado na compressão; metadados removidos |
| Local | `filesDir/fotos/{product_id}.jpg` |
| Remoto | bucket privado `fotos`, caminho `{user_id}/{product_id}.jpg` |
| Exibição | Coil com cache em disco; URL assinada de 1 h para o bucket privado |
| Troca | Sobrescreve o arquivo e invalida o cache |
| Sem foto | Placeholder com a inicial do produto e selo "sem foto" |
| Exclusão | Ao inativar o produto, a foto permanece (histórico) |

---

## 15. Segurança e privacidade

- RLS em todas as tabelas e no Storage.
- Chave `anon` no app é aceitável; `service_role` nunca sai do painel.
- Bucket privado com URL assinada.
- Sessão em `EncryptedSharedPreferences`/Keystore (via supabase-kt).
- Sem analytics, crash reporting ou SDK de terceiros.
- Dados pessoais coletados: e-mail (login) e conteúdo das listas. Nada é compartilhado.
- Exclusão de conta: feita pelo painel do Supabase no MVP (não há usuários públicos).
- Backup do banco: exportação manual pelo painel; sem rotina automática no free tier.
- Keystore do APK guardado em dois lugares; perder a keystore impede atualização sem desinstalar.

---

## 16. Interface, acessibilidade e identidade

### 16.1 Princípios

- **Uma mão**: ações principais no terço inferior; abas embaixo.
- **Poucos toques**: comprar = 2 toques + confirmar preço.
- **Feedback imediato**: item marcado anima e move para "No carrinho"; total atualiza.
- **Legibilidade no corredor**: fonte grande, contraste alto, foto reconhecível.
- **Sem modais desnecessários**: teclado de preço é bottom sheet, não tela nova.

### 16.2 Identidade

- Nome: **Põe na Lista**.
- Tom: direto, brasileiro, sem infantilizar.
- Cor primária: verde (lista/mercado), com neutros quentes.
- Ícone: sacola/lista com check.
- Tema claro e escuro, Material 3, dynamic color desativado para manter identidade.

### 16.3 Acessibilidade

- Alvos ≥ 48 dp.
- Fonte mínima 16 sp; respeitar escala do sistema.
- `contentDescription` em todos os ícones e fotos.
- Contraste AA.
- Não depender só de cor para estados (usar ícone + texto).
- Testar com TalkBack nos fluxos de compra e finalização.

---

## 17. Plano de testes

### 17.1 Unitários (JVM)

- Dinheiro: subtotal, total parcial, teto, arredondamento, promoção com média.
- R$/kg com pesos variados e divisão por zero.
- Variação percentual com preço anterior zero.
- Máquina de estados da lista e do item (todas as transições válidas e inválidas).
- Regras de finalização e de retomada.
- Merge de sincronização: local sujo vence; remoto vence quando local limpo.

### 17.2 Room

- DAOs com banco em memória.
- Índices únicos parciais (uma lista ativa, um produto por lista).
- Migrations: criar versão N-1, migrar, validar dados.

### 17.3 UI (Compose)

- Captura rápida cria item.
- Comprado → preço → total atualizado.
- Item sem preço mostra total parcial.
- Finalizar aplica estados.
- Tema escuro não quebra layout.

### 17.4 Matriz manual (aparelho real)

| Cenário | Esperado |
|---|---|
| Modo avião durante toda a compra | Nenhuma perda; sync depois |
| Matar o app no meio da compra | Retoma no item correto |
| Negar/alterar permissão de câmera | App não quebra; oferece galeria |
| Tirar foto e cancelar | Nada é salvo |
| Armazenamento quase cheio | Erro claro, sem corromper |
| Dois aparelhos marcando ao mesmo tempo | Converge; sem duplicata |
| Reinstalar o APK e logar | Dados voltam |
| Trocar de conta e voltar | Dados corretos por conta |
| Visitante → criar conta | Migração sem duplicar |
| Supabase pausado | App local funciona; aviso de sync |
| Fuso horário diferente | Datas corretas |
| Fonte do sistema em 200% | Layout utilizável |

### 17.5 Critérios de qualidade

- Nenhum crash nos fluxos principais.
- Nenhuma perda de dado reproduzível.
- Total sempre igual à soma dos subtotais exibidos.
- Testes de dinheiro e de estados com cobertura alta (regras críticas).

---

## 18. MVP priorizado

### Must (v1.0)

- Login, sessão, troca de conta, visitante.
- Catálogo com categorias padrão, foto e captura rápida.
- Lista ativa automática e "acabou".
- Compra com preço, quantidade, "não tinha", "+".
- Total, teto e barra.
- Finalização com revisão, cupom e reabertura da última.
- Histórico, gasto do mês e gráfico de preço.
- Offline completo e sincronização entre os dois celulares da casa.
- Tema claro/escuro.

### Should (v1.1)

- Exportação/backup em arquivo.
- Código de barras no próprio catálogo.
- Mesclagem de produtos duplicados.
- Listas extras nomeadas (Farmácia, Churrasco).

### Could (v1.2+)

- Estoque simplificado (quantidade em casa).
- Sugestão de reposição com base no histórico.
- Notificação de compra mensal.
- Foto do cupom fiscal.

### Won't (não faremos)

- Publicação na Play Store.
- Multiusuário com permissões por membro.
- App de finanças com orçamento e categorias de gasto.
- Versão web/iOS.
- Receitas ou recursos sociais.

---

## 19. Roadmap

| Versão | Conteúdo | Critério de saída |
|---|---|---|
| 0.1 | Validação técnica (Compose + Room + Supabase) | App no celular lendo e gravando no Supabase |
| 0.2 | Catálogo local + captura rápida | 10 produtos cadastrados e usáveis offline |
| 0.3 | Compra local completa | Uma compra do início ao histórico, sem internet |
| 0.4 | Sincronização e fotos | Dois aparelhos convergem |
| 1.0 | Polimento, acessibilidade, APK assinado | Checklist de qualidade + release no GitHub |
| 1.1 | Barcode, exportação, mesclagem | — |
| 1.2 | Estoque e sugestões | — |

---

## 20. Complexidade e custos

### 20.1 Tempo (ritmo de ~8 h/semana)

| Etapa | Estimativa |
|---|---|
| Aprender Kotlin/Compose (em paralelo) | 40–60 h |
| Validação técnica | 8–12 h |
| Catálogo e captura | 25–35 h |
| Compra e finalização | 30–40 h |
| Sincronização e fotos | 40–60 h |
| Polimento, testes, APK | 20–30 h |
| **Total** | **160–240 h (4–6 meses)** |

### 20.2 Infraestrutura

| Item | Custo |
|---|---|
| Supabase free | R$ 0 (500 MB DB, 1 GB Storage, pausa após ~7 dias inativo) |
| Keepalive (GitHub Actions) | R$ 0 |
| Android Studio / SDK | R$ 0 |
| GitHub | R$ 0 |
| Play Store | não se aplica |
| **Total** | **R$ 0/mês** |

**Riscos de custo:** estourar 1 GB de Storage é improvável (60 produtos × 300 KB ≈ 18 MB). O risco real é a pausa por inatividade, mitigada por ping a cada 2–3 dias e reativação manual no painel.

---

## 21. Plano de implementação

Cada etapa termina com **algo instalável e usável**, e só avança com o critério de conclusão atendido.

### Etapa 0 — Validação técnica (1 fim de semana)
- Projeto Compose, navegação mínima, Room com uma tabela, Supabase Auth e upload de uma foto.
- **Concluída quando:** o app roda no A54, grava um produto local, autentica e envia/baixa uma foto.

### Etapa 1 — Catálogo local
- Produtos, categorias padrão, foto local, captura rápida, tela de produto, busca.
- **Concluída quando:** cadastrar 10 produtos com foto e usar tudo em modo avião.

### Etapa 2 — Lista e compra local
- "Acabou", lista ativa, compra, preço, total, teto, não tinha, finalização, histórico, reabrir.
- **Concluída quando:** uma compra completa, do "acabou" ao histórico, sem internet, com total correto.

### Etapa 3 — Sincronização
- Auth real, outbox, pull, conflitos, fotos remotas, duas contas, visitante.
- **Concluída quando:** dois aparelhos convergem e o modo avião não perde nada.

### Etapa 4 — Polimento
- Tema, acessibilidade, estados vazios/erro, gráfico de preço, animações, textos.
- **Concluída quando:** checklist de qualidade e matriz manual aprovados.

### Etapa 5 — Entrega
- APK assinado, README com prints, release no GitHub, keystore guardada.
- **Concluída quando:** instalação limpa em um aparelho novo + login restaura os dados.

---

## 22. Critérios de entrega (APK)

- APK assinado em release, versão nomeada (`1.0.0`), instalável em Android 8+.
- Instalação limpa + login restaura todos os dados.
- Funciona offline em todos os fluxos de compra.
- Keystore e senha guardadas em dois locais.
- README com: descrição, prints, como instalar, limitações conhecidas e stack.
- Release no GitHub com o APK anexado.
- Nenhum segredo no repositório (`.env`, chaves, keystore).

**Cadência de atualização:** o APK é atualizado manualmente, quando o Victor tem acesso físico ao aparelho da mãe. Por isso, cada versão deve ser conservadora: migrations do Room testadas com dados reais, versão anterior sempre instalável e nenhuma atualização com prazo apertado. Um runbook de assinatura e instalação fica no README.

---

## 23. Decisões, pendências e recomendações

### 23.1 Decisões tomadas

- App pessoal/familiar, sem publicação; APK em até 3 aparelhos.
- Duas contas, uma por casa; conta compartilhada dentro da casa.
- Kotlin + Compose (escolha do product owner).
- Produto com identidade fixa; marca e tamanho na compra.
- Produto nunca excluído; inativação + snapshot.
- Categorias padrão editáveis.
- Captura rápida como substituto do bloco de notas.
- Uma lista ativa automática alimentada por "acabou".
- Preço digitado no corredor, com último preço pré-preenchido.
- Teto por compra, sem bloqueio.
- Sem campo de desconto; subtotal é a fonte de verdade.
- Cupom fiscal opcional para conferência.
- Foto opcional, uma por produto, no mercado ou depois.
- Histórico com gráfico de preço e gasto do mês.
- Reabrir a última compra; apagar com confirmação dupla.
- Offline-first com LWW por linha.
- Sem notificações, sem exportação no MVP.
- A compra em andamento é compartilhada pela conta; a abertura automática na tela de compra vale só para o aparelho que a iniciou.
- Troca de conta funciona offline, com cache por conta preservado.
- Categoria só é desativada se estiver vazia; "Outros" é permanente.
- Produto pesado compara preço em R$/kg, não em total da etiqueta.

### 23.2 Pendências

| # | Pendência | Impacto |
|---|---|---|
| P1 | A conta da mãe usa e-mail dela ou do Victor? | Recuperação de senha |
| P2 | Confirmar limites atuais do free tier do Supabase antes do lançamento | Custo/keepalive |
| P3 | Validar `supabase-kt` na Etapa 0 | Risco técnico principal |
| P4 | O celular da mãe tem internet com frequência? | Frequência de sync e risco de perda local |
| P5 | Com que frequência o APK é atualizado no aparelho dela? | Conservadorismo das migrations |

### 23.3 Recomendações do autor da especificação

- Fazer a Etapa 0 antes de qualquer tela: ela responde se a stack escolhida se sustenta.
- Manter o sync por último e atrás de interface.
- Não adicionar campo de desconto, estoque ou notificações antes da v1.0.
- Guardar a keystore no primeiro dia em que o APK for assinado.

---

## 24. Riscos

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Curva do Kotlin atrasar o projeto | Alta | Alto | Fatias verticais; app usável desde a etapa 1 |
| `supabase-kt` imaturo | Média | Alto | Interface + fallback REST/Ktor; validar na etapa 0 |
| Sync com conflito silencioso | Média | Alto | LWW documentado; indicador de sync; testes de merge |
| Pausa do Supabase free | Alta | Médio | Keepalive + reativação manual; dados locais intactos |
| Perda da keystore | Baixa | Alto | Dois backups desde o primeiro APK assinado |
| Escopo crescer | Média | Alto | MVP Must/Should/Could/Won't |
| Volume de fotos | Baixa | Baixo | Uma foto por produto, comprimida |
| Android Studio no Windows | Média | Baixo | Build via linha de comando quando possível |
