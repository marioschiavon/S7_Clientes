# Patch note — 02/11/2026

**Cliente:** Juliano Carneiro — Groomer Genius
**Apps:** `groomer-genius-b2b` e `groomer-genius-testes` (alterações idênticas nos dois)
**Página principal:** `tutores_pets`
**Status:** testado no preview dos dois apps · deploy para Live feito pelo Mário

---

## 1. Experiência pós-cadastro do tutor

**Problema relatado pelo cliente:** depois de salvar um tutor, o veterinário ficava perdido. O formulário continuava na tela e o card do tutor ficava escondido lá embaixo, longe do botão "Adicionar Pet".

**O que mudou**
- O formulário "Novo Tutor" fecha depois de salvar.
- No lugar dele aparece o card **"✓ Tutor cadastrado! Agora adicione o pet:"** (`Group Tutor Novo`), com nome, telefone, email, o lápis de editar e o botão **Adicionar Pet**.
- A tela rola automaticamente até o card.
- Ao clicar em "Cadastrar Tutor" de novo, o card some e só o formulário aparece.
- A lista continua ordenada pelo mais recente, então o tutor novo aparece no topo.

**Técnico**
- Novo custom state na página: `tutor_novo` (Tutor).
- Os dois workflows de salvar foram ajustados: `Button Salvar1` (sem clínica) e `Button Continuar` do Popup atenção (com clínica). Agora gravam `tutor_novo = Result of step 1`, mostram `Group Tutor Novo` e rolam até ele.
- `Button Cadastrar Tutor`: passou a esconder `Group Tutor Novo`.

---

## 2. Excluir, arquivar e restaurar tutor

**Regra de negócio**
- Tutor **sem pets e sem pedidos** pode ser **excluído** de verdade. Os endereços dele também são apagados. Também pode ser arquivado.
- Tutor **com pet ou pedido** só pode ser **arquivado**. Ele sai das listas, mas o histórico de pets e pedidos fica preservado.
- Tutor arquivado pode ser **restaurado**.
- Só o veterinário que cadastrou o tutor (`Id_vet = Current User`) vê a opção de remover.

**Interface**
- Ícone de lixeira vermelha ao lado do lápis nos cards das duas listas ("Meus Pacientes" e "Clientes Clínica").
- Popup **"Remover tutor"** mostra o nome do tutor, a quantidade de pets e de pedidos e um texto explicando a regra.
  - **Excluir:** só aparece com 0 pets e 0 pedidos.
  - **Arquivar:** vira **Restaurar** quando o tutor já está arquivado.
- Checkbox **"Ver arquivados"** na Lista de Tutores: mostra só os arquivados da aba selecionada, para restaurar.
- Toasts de confirmação em cada ação.

**Banco de dados — tipo Tutor (campos novos)**
| Campo | Tipo |
|---|---|
| `arquivado` | yes/no |
| `data_arquivamento` | date |
| `arquivado_por` | User |

Os tutores já cadastrados ficam com o campo vazio e continuam aparecendo normalmente, porque o filtro usa `arquivado ≠ yes`. Não foi preciso migrar dados.

**Workflows (tutores_pets)**
- Lixeira (nos dois cards) → Show + Display data no `Popup Remover Tutor`.
- Excluir → Delete list of enderecos do tutor → Delete Tutor → fecha popup → toast. Só executa quando *Search Pets:count + Search Pedidos:count = 0*. A regra vale no workflow, não só na interface.
- Arquivar → `arquivado = yes`, `data_arquivamento = Current date/time`, `arquivado_por = Current User` → fecha popup → toast.
- Restaurar → `arquivado = no` → fecha popup → toast.

**Filtro de arquivados (`arquivado ≠ yes`)**
- `tutores_pets`: RepeatingGroup Tutor meu, RepeatingGroup Tutor clinica e as duas SearchBox de tutor. O checkbox "Ver arquivados" troca a fonte de dados para `arquivado = yes`.
- `criar_pedido`: SearchBox de tutor.
- `clinicas_vet`: SearchBox de tutor.

**Mantido sem filtro (intencional)**
- `clientes`: listagem geral (admin) continua mostrando todos.
- `recompra`: a busca de tutor existente continua encontrando arquivados, para não duplicar cadastro.

---

## 3. Testes realizados (preview, b2b e testes)
- Cadastro com e sem clínica → card de sucesso aparece, formulário fecha, tutor no topo da lista.
- Arquivar → tutor sai da lista e aparece em "Ver arquivados".
- Restaurar → tutor volta para a lista.
- Excluir tutor sem pets e sem pedidos → registro apagado.
- Tutor com 1 pet e 1 pedido → popup mostra só "Arquivar".
- Registros de teste ("Teste Claude") apagados nos dois apps.

## 4. Observações
- Ao cadastrar tutor **com clínica**, a lista abre na aba "Clientes Clínica". É comportamento que já existia, mantido. O card de sucesso aparece do mesmo jeito.
- O `groomer-genius-testes` não tem algumas alterações **anteriores** a hoje que existem na b2b: validação de CPF no cadastro, um grupo extra no Popup Tutor e um workflow de Page Loaded. Não foram replicadas nesta entrega.
- Comportamentos que já existiam e não foram alterados: a mensagem "CPF inválido" continua visível ao reabrir o formulário vazio, e os campos com máscara embaralham os dígitos quando o texto é digitado muito rápido.
