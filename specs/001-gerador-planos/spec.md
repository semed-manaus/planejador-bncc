# Feature Specification: Gerador de Planos BNCC

**Feature Directory**: `specs/001-gerador-planos`

**Created**: 2026-10-01

**Status**: Draft

**Input**: User description: "Desenvolver o Planejador BNCC para uso de professores..."

## Clarifications
### Session 2026-10-01
- Q: Como o sistema deve lidar com modificações não salvas no editor Markdown caso a sessão do professor expire em background? → A: Interceptar erro de autenticação (401), manter os dados na tela e pedir reautenticação.
- Q: Como o sistema deve se comportar se a busca no catálogo BNCC falhar ou não retornar resultados para a pesquisa? → A: Bloquear o fluxo com erro (exige habilidade válida do catálogo para prosseguir).
- Q: Como o sistema deve tratar retornos da IA que contenham Markdown malformado? → A: Renderizar o que for possível e deixar o texto cru no editor para o professor corrigir manualmente.

## User Scenarios & Testing *(mandatory)*

<!--
  Priorities (P1, P2, P3) based on user journey.
-->

### User Story 1 - Autenticação e Sessão (Priority: P1)

Como professor, quero fazer login usando uma conta de demonstração pré-cadastrada, para que eu possa acessar meu ambiente seguro e privado.

**Why this priority**: Sem autenticação, não é possível garantir isolamento de dados entre os professores (um requisito primário de privacidade).

**Independent Test**: Pode ser testado tentando fazer login com credenciais válidas e inválidas, garantindo a criação de sessão (cookie/token) e efetuando logout com sucesso.

**Acceptance Scenarios**:

1. **Given** que o usuário está na tela de login, **When** insere as credenciais da demonstração 1, **Then** é redirecionado para a tela "Meus planos" com a sessão iniciada.
2. **Given** que o usuário autenticado clica em "Sair", **When** o comando é processado, **Then** a sessão é encerrada e ele retorna para a tela de login.

---

### User Story 2 - Consulta ao Catálogo BNCC e Formulário Base (Priority: P1)

Como professor autenticado, quero consultar as habilidades da BNCC por nível, ano, eixo, código ou texto, para que eu possa selecionar os objetivos da minha aula e iniciar a geração do plano.

**Why this priority**: É o gatilho principal para a criação do plano. O catálogo permite direcionar o modelo de IA.

**Independent Test**: Testado buscando por um código (ex: EF05CI02), selecionando-o, preenchendo "instrução pedagógica", "duração" (>0) e marcando "recursos digitais", permitindo o submissão do formulário.

**Acceptance Scenarios**:

1. **Given** o formulário de novo plano, **When** o professor digita "EF05CI02", **Then** a habilidade aparece nas opções e pode ser selecionada.
2. **Given** que a duração inserida é 0 ou vazia, **When** o usuário tenta submeter, **Then** um alerta visual de erro ("Informe uma duração maior que zero") bloqueia o envio.

---

### User Story 3 - Geração por IA, Falhas e Atomicidade (Priority: P1)

Como professor, quero confirmar a geração e visualizar um estado de carregamento enquanto a IA trabalha, para que eu saiba que meu rascunho está sendo processado. Se algo falhar, o estado anterior deve ser preservado.

**Why this priority**: É a essência do produto. A UX precisa ser robusta em caso de falha externa e evitar dados parciais sujos no banco.

**Independent Test**: Interceptando e simulando falha na resposta da IA para garantir que nada é salvo, e simulando sucesso para garantir que o Loading desaparece e a tela do rascunho surge.

**Acceptance Scenarios**:

1. **Given** o formulário preenchido corretamente, **When** submetido, **Then** a interface entra em modo de carregamento (progress bar) indicando processamento.
2. **Given** uma falha no serviço de IA, **When** o retorno de erro acontece, **Then** a aplicação exibe um alerta de erro, preserva os dados inseridos e não cria nenhum plano no banco.

---

### User Story 4 - Edição, Pré-Visualização e Salvamento (Priority: P2)

Como professor, quero visualizar o retorno da IA em Markdown, alternar entre edição e pré-visualização, e salvar manualmente as alterações feitas.

**Why this priority**: Garantir que a IA seja apenas um rascunho ("Human in the loop"); a validação do docente é obrigatória.

**Independent Test**: Modificar o Markdown gerado e alternar para pré-visualização; os dados renderizados devem refletir a mudança. O salvamento deve disparar um "toast" de sucesso.

**Acceptance Scenarios**:

1. **Given** um plano gerado pela IA com status "RASCUNHO", **When** o professor altera o Markdown, **Then** a aba de Pré-visualização reflete a mudança renderizada.
2. **Given** o usuário na tela de edição, **When** clica em "Salvar", **Then** os dados são persistidos e um alerta visual verde (Toast) confirma o sucesso.
3. **Given** modificações não salvas, **When** o usuário tenta sair da página, **Then** um Modal intercepta a navegação perguntando "Sair sem salvar?".

---

### User Story 5 - Listagem e Isolamento de Planos (Priority: P2)

Como professor autenticado, quero ver a lista ("Meus planos") apenas com os meus próprios rascunhos, para que eu tenha certeza de que minhas informações estão isoladas e acessíveis apenas por mim.

**Why this priority**: Cumprir a regra de isolamento descrita na Constituição (Tenant isolation).

**Independent Test**: Autenticar como "Professor 1", criar um plano, fazer logout, autenticar como "Professor 2", verificar que o plano do Professor 1 não está listado nem pode ser acessado via URL direta.

**Acceptance Scenarios**:

1. **Given** o Professor 1 que criou um plano com id X, **When** loga no painel, **Then** vê o plano X listado.
2. **Given** o Professor 2 acessando o sistema, **When** visita a lista de planos ou tenta acessar via rota o ID X, **Then** não vê o plano na listagem e recebe acesso negado.

---

### Edge Cases

- **Catálogo Indisponível/Vazio**: Caso o serviço do catálogo falhe, a seleção será bloqueada e o sistema não permitirá o envio do formulário, exibindo mensagem de erro (é mandatório ter uma habilidade válida para gerar o plano).
- **Tratamento de Sessão Expirada**: Se o token de sessão expirar enquanto o professor digita o Markdown, qualquer tentativa de salvamento retornará erro 401. A interface interceptará esse erro, preservará os dados não salvos na tela e solicitará a reautenticação (ex: modal ou redirecionamento de tela de login que retorna ao mesmo estado de edição).
- **Markdown Malformado pela IA**: O sistema de frontend renderizará as tags o mais fielmente possível, deixando falhas de sintaxe expostas no editor de texto cru para correção manual pelo professor. O backend não bloqueará nem sanitarizará erros de markup da IA.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O sistema DEVE fornecer login e logout, mantendo uma sessão segura.
- **FR-002**: O sistema DEVE inicializar o banco de dados via "seed" com pelo menos duas contas de demonstração pré-configuradas e uma massa de dados mínima do catálogo BNCC.
- **FR-003**: O sistema NÃO DEVE fornecer registro (sign-up) público ou painel administrativo.
- **FR-004**: O sistema DEVE restringir o acesso a qualquer plano, de modo que apenas o autor (professor) possa ler, editar ou listar aquele plano.
- **FR-005**: O sistema DEVE validar o formulário de geração, impedindo submissão caso a duração seja inválida ou campos obrigatórios faltem.
- **FR-006**: O sistema DEVE suportar a exibição de componentes de interface do Figma (Toasts de sucesso, Loading bars, Badges de Status "Rascunho" e "Auxílio por IA").
- **FR-007**: A integração com a IA DEVE ocorrer via backend (sem expor a chave de API ao cliente).
- **FR-008**: O sistema DEVE ser transacional no salvamento: planos não podem ser persistidos de forma incompleta ("suja") em caso de timeout ou falha na integração externa.
- **FR-009**: O editor de Markdown DEVE possuir duas abas funcionais (Edição / Pré-visualização).
- **FR-010**: O sistema NÃO DEVE incluir finalização, versionamento histórico do plano, geração de PDF ou publicação pública externa.

### Key Entities

- **User (Professor)**: Conta pré-existente (demonstração), gerencia sua própria sessão.
- **BNCC Skill (Habilidade)**: Item estático do catálogo da BNCC que contém código (ex: EF05CI02), descrição e metadados de filtro.
- **Lesson Plan (Plano de Aula)**: Documento que relaciona o Professor, as Habilidades selecionadas, metadados (duração, recursos digitais) e o conteúdo em Markdown (com indicação se sofreu auxílio de IA e se está no estado "Rascunho").

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: O isolamento de dados entre o Professor 1 e Professor 2 não permite brechas de acesso em nenhuma rota.
- **SC-002**: A simulação de falha na IA não insere nenhum registro correspondente na tabela de Planos (Atomicidade comprovada).
- **SC-003**: O fluxo completo (Login > Busca BNCC > Loading > Editor Markdown > Logout) pode ser executado e validado em menos de 1 minuto em condições ideais.
- **SC-004**: Todas as respostas visuais, validações e badging aderem à semântica mapeada no Figma.

## Assumptions

- O catálogo mínimo da BNCC incluído no seed conterá dados suficientes de múltiplas séries/eixos para testar o sistema de buscas (ex. pelo menos 10-20 habilidades do Ensino Fundamental).
- A IA a ser integrada será de processamento síncrono ou com mecanismo de "long-polling/streaming" configurado de tal maneira que a resposta caiba dentro da janela tolerada de requisição web pelo cliente.
- Como "Não há versão de histórico de plano", cada vez que o professor salva as alterações, o documento de Markdown sofre uma sobreposição (update direto).
