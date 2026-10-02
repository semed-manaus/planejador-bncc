<!--
Sync Impact Report:
- Version change: Unversioned -> 1.0.0
- Added 8 Core Principles based on user specifications.
- Removed unused template sections 2 and 3.
- Populated Governance details and Dates.
-->
# Planejador BNCC Constitution

## Core Principles

### I. Especificação Antes do Código (Spec-First)
Comportamentos e critérios de aceitação DEVEM ser especificados antes da escrita de código. A geração de código sem especificação prévia e aprovação é estritamente proibida.

### II. Separação de Arquitetura e Segurança
DEVE-SE manter uma separação clara entre frontend, API e integrações externas. Segredos, chaves de API e credenciais DEVEM residir exclusivamente no backend.

### III. Autenticação e Autorização Estrita
DEVE-SE autenticar os professores. A aplicação DEVE garantir que cada usuário tenha acesso autorizado unicamente aos seus próprios planos de aula, prevenindo vazamento de dados.

### IV. Rascunhos de IA (Human-in-the-Loop)
A saída de qualquer modelo de Inteligência Artificial DEVE ser tratada como um rascunho editável, estando sempre sujeita à revisão, modificação e aprovação docente final.

### V. Validação e Atomicidade
Entradas de usuários e respostas de integrações externas DEVEM ser validadas. Em caso de falha durante a geração ou processamento, a operação não deve resultar em planos parciais ou corrompidos.

### VI. Persistência e Reprodutibilidade
Os dados DEVEM ser persistidos utilizando "migrations" estruturadas e rotinas de "seed" reprodutíveis para garantir consistência entre ambientes.

### VII. Consistência de UI/UX e Acessibilidade
O desenvolvimento DEVE usar componentes e tokens coerentes com o Design System do Figma. A aplicação DEVE garantir acessibilidade e ser totalmente responsiva (desktop, tablet e celular).

### VIII. Testes, Documentação e Artefatos
Comportamentos críticos DEVEM ser testados e a execução DEVE ser documentada. Artefatos de projeto devem ser versionados; arquivos `.env` ou credenciais NUNCA devem ser versionados.

## Governance

- Esta Constituição atua como a lei primária do projeto Planejador BNCC.
- Todas as revisões de código (PRs) e agentes automatizados DEVEM verificar e garantir o cumprimento estrito destes princípios.
- Alterações nestes princípios exigem atualização da versão do documento e uma justificativa clara.
- Quebras nos princípios de segurança (como o versionamento de segredos) bloqueiam imediatamente qualquer aprovação ou integração de código.

**Version**: 1.0.0 | **Ratified**: 2026-10-01 | **Last Amended**: 2026-10-01
