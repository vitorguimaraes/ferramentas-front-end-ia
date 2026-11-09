# Stack de desenvolvimento frontend com IA: skills, ferramentas e exemplos agnósticos

> Relatório de estratégia. Escopo: aplicações web em geral, sem depender de organização ou stack específica.
> Convenção: itens marcados com **[não verificado]** vêm de leitura parcial (descrições e listagens), não do conteúdo completo das fontes.

---

## 1. Resumo executivo

1. **Design system antes de `frontend-design`.** Se as decisões de cor, fonte, ícone e movimento já existem, a IA deve consumi-las como tokens, não reinventá-las a cada briefing. `frontend-design` fica reservada para exploração sem marca.
2. **Regra de divisão:** *se dá para escrever como teste, vira ferramenta; se exige julgamento, vira skill.* A skill decide o quê e por quê; a ferramenta prova que foi feito.
3. **Cinco camadas independentes:** tokens, decisão de interação, acessibilidade, adaptador de stack e verificação. Só o adaptador conhece a tecnologia.
4. **Fluxo de trabalho (planejar, implementar, revisar) não precisa ser construído.** O repositório `mattpocock/skills` cobre essa camada. Falta o que é específico de interface.
5. **Playwright entra como base da verificação**, com axe-core. `a11y-audit` e AccessLint entram só como complemento opcional.
6. **Exemplos em HTML + CSS vanilla** servem como especificação neutra e como *fixtures* de teste das ferramentas, não como código para copiar.

---

## 2. Arquitetura em camadas

```
┌──────────────────────────────────────────────────────────────┐
│ 6. FLUXO              planejar → implementar → verificar → revisar │  (terceiros: mattpocock/skills)
├──────────────────────────────────────────────────────────────┤
│ 5. VERIFICAÇÃO        ferramentas Python + Playwright + axe-core   │  (construir)
├──────────────────────────────────────────────────────────────┤
│ 4. ADAPTADOR DE STACK  uma skill curta por tecnologia              │  (construir sob demanda)
├──────────────────────────────────────────────────────────────┤
│ 3. ACESSIBILIDADE      teclado, foco, ARIA, aria-live, WCAG AA     │  (adaptar de terceiros)
├──────────────────────────────────────────────────────────────┤
│ 2. INTERAÇÃO           gatilho, regras, feedback, loops, microcopy │  (melhorar a existente)
├──────────────────────────────────────────────────────────────┤
│ 1. TOKENS / DESIGN SYSTEM  cor, tipografia, espaço, movimento      │  (construir primeiro)
└──────────────────────────────────────────────────────────────┘
```

Princípios de composição:

- **Dependência só para baixo.** Uma camada pode citar as inferiores, nunca as superiores.
- **Uma fonte de verdade por assunto.** Valores de duração e curva ficam nos tokens; a skill de microinterações diz "use o token de duração curta", nunca "150 ms".
- **Núcleo agnóstico, adaptador específico.** Nada das camadas 1 a 3 cita React, LiveView ou Tailwind.

---

## 3. Design system primeiro ou `frontend-design`?

| Situação | Base recomendada | Motivo |
|---|---|---|
| Produto com decisões visuais já tomadas | **Design system (tokens + regras)** | Evita que a IA gaste esforço em escolhas estéticas que você já fez e que divergem dos tokens |
| Protótipo ou página sem marca | `frontend-design` | Ela pede direção estética forte, fontes e paleta "deliberadas" e risco visual |
| Produto com marca, mas sem ideias de composição | Design system + princípios de composição copiados da `frontend-design` como referência | Mantém os tokens e aproveita o repertório de layout |

**Regra de precedência** (escrever na skill do design system): *tokens e fontes do sistema vencem qualquer escolha estética de outra skill.*

Observação: "mais performático" aqui significa **mais consistente e com menos retrabalho**. Não houve medição de tokens consumidos nem de tempo de geração.

---

## 4. Skills: o que criar, melhorar ou adotar

### 4.1 Visão geral

| # | Skill | Ação | Camada | Agnóstica? |
|---|---|---|---|---|
| S1 | `design-system` | **Criar** | 1 | Sim |
| S2 | `microinteractions` | **Melhorar** (já existe) | 2 | Sim |
| S3 | `interaction-accessibility` | **Adaptar** de terceiros + estender | 3 | Sim |
| S4 | `stack-adapter-<stack>` | **Criar** sob demanda (uma por stack) | 4 | Não, por definição |
| S5 | `ui-verification` | **Criar** (como usar as ferramentas e interpretar relatórios) | 5 | Sim |
| S6 | Fluxo (grill, spec, implement, review) | **Adotar** `mattpocock/skills` | 6 | Sim |
| S7 | `frontend-design` | **Adotar** só para exploração | 1 (opcional) | Sim |
| S8 | `a11y-audit` / AccessLint | **Opcional**, como checagem complementar | 5 | Parcial |

### 4.2 S1 — `design-system` (criar)

**Função:** dizer à IA *quais* decisões visuais são fixas e como consumi-las.

Conteúdo:
- `tokens/tokens.json` em formato compatível com o Design Tokens Community Group (relatório de Community Group, **não** padrão W3C): cor, tipografia, espaçamento, raio, sombra, **duração e curva de movimento**, ícones permitidos.
- Regras de uso: "nunca valor literal", escala de espaçamento, hierarquia de botões, estados obrigatórios.
- Regra de precedência sobre skills estéticas.
- Tokens de movimento nomeados (exemplo de escala: rápido, normal, entrada, saída), com valores ancorados em uma referência oficial (Fluent 2, Material 3 ou outra) e registrados como decisão.

Não deve conter: padrões de interação (S2) nem de acessibilidade (S3). Apenas exigir que os tokens passem nos limites do WCAG (contraste, alvo de toque).

Gatilhos: "tema", "tokens", "cores", "tipografia", "componente", "consistência visual", "design system".
Exclusões: foco e teclado; decisão de feedback.

### 4.3 S2 — `microinteractions` (melhorar)

Estado atual: `SKILL.md` com 65 linhas e seis arquivos em `references/` (gatilhos, regras, feedback, loops e modos, prática, checklist).

Melhorias:

| Melhoria | Por quê |
|---|---|
| `description` com **frase de exclusão** ("não usar para foco, teclado, ARIA nem definição de tokens") | Hoje dispara em quase todo pedido com "botão", "formulário" ou "animação" |
| Trocar durações e easings por **referência a tokens** | Evita divergir do design system |
| Adicionar o **contrato `data-state`** (idle, loading, success, error) | Torna a skill portável entre stacks |
| Regra de persistência de long loops: **o que lembrar, por quanto tempo, como limpar, privacidade** | O teste real mostrou 4 memórias criadas sem critério |
| Modo **Perguntar antes de animar** para pedidos vagos | Evita "animar tudo" |
| Linha de encaminhamento: "para foco e teclado, use `interaction-accessibility`" | Evita sobreposição |
| Rodada de avaliação cega com e sem a skill; podar o que não mudar o resultado | O ganho observado foi principalmente de consistência |

Aviso: a skill é um resumo reescrito de um livro (Dan Saffer, 2013). Manter só frases curtas com atribuição.

### 4.4 S3 — `interaction-accessibility` (adaptar de terceiros)

Base candidata: a skill `accessibility` do repositório `danielteles/awesome-agent-skills` (licença CC-BY-4.0, exige atribuição ao adaptar). Pontos fortes observados nas descrições: três modos (Generate, Review, Audit), WCAG 2.2 AA, WAI-ARIA e APG como fontes, formato de saída de um achado por linha, regras autossuficientes no `SKILL.md`.

O que adicionar:

1. **Fichas por componente** em `references/patterns/` (diálogo, menu, combobox/paleta de comandos, disclosure, toggle, tabs, toast/alert), cada uma com: padrão APG, teclado esperado, para onde vai o foco, atributos ARIA, critérios WCAG associados e como testar.
2. **Foco após re-render** e **restauração de foco** ao fechar overlays: foram os itens de maior ganho prático no teste real e nenhum pacote resolve sozinho.
3. **Mensagens de status** (`aria-live`, critério 4.1.3) e anúncio de mudança de rota.
4. **Preferência por elementos nativos** (por exemplo `<dialog>`) antes de ARIA e de focus trap manual.
5. **Delegação ao adaptador** para o mecanismo de cada stack.

O que **não** copiar: a skill `wcag-2.2` do Raze-Systems proíbe oferecer componentes prontos e tem licença AGPL-3.0. Usar só como consulta, sem instalar junto (as duas têm gatilhos parecidos e regras opostas sobre gerar código).

Lacunas a confirmar abrindo `references/` do danielteles **[não verificado]**: foco após re-render, focus trap, tratamento de Esc.

### 4.5 S4 — adaptadores de stack (criar sob demanda)

Uma skill pequena por tecnologia, que responde só: *como alternar o `data-state`, como gerenciar foco e como tratar movimento reduzido nesta stack.*

| Adaptador | Conteúdo típico |
|---|---|
| `stack-adapter-vanilla` | `data-state`, `<dialog>`, `inert`, `matchMedia('(prefers-reduced-motion)')` |
| `stack-adapter-react` | estado → `data-state`, refs para foco, `useId`, bibliotecas headless |
| `stack-adapter-phoenix-liveview` | `Phoenix.LiveView.JS` (transições, foco), `phx-click-loading`, hooks, `motion-reduce:` do Tailwind |
| `stack-adapter-vue` / `-svelte` | equivalente por framework |

Pacotes de UI **não** eliminam a necessidade:
- **Radix / shadcn / React Aria / Headless UI:** trazem papéis, teclado e foco de diálogos e menus; não resolvem nomes acessíveis, mensagens dinâmicas, contraste nem customizações que quebram o gatilho.
- **daisyUI e Tailwind:** são CSS. Não adicionam ARIA, focus trap nem tratamento de Esc. O preflight do Tailwind remove estilos de foco padrão de alguns elementos; é preciso reaplicar com `focus-visible:`.

### 4.6 S5 — `ui-verification` (criar)

Skill curta que ensina o agente a **usar** as ferramentas da seção 5:
- quando rodar cada uma (antes de declarar "pronto");
- como ler o relatório e priorizar (bloqueante, importante, sugestão);
- **checklist manual** do que a automação não pega: ordem de foco, qualidade do texto alternativo, microcopy, leitor de tela nos fluxos críticos;
- regra de saída: não declarar concluído com violação bloqueante aberta.

Referência de limite: ferramentas automáticas detectam só uma fração dos problemas (a skill do danielteles estima 30 a 40%).

### 4.7 S6 — fluxo (adotar `mattpocock/skills`)

Repositório MIT de skills pequenas e combináveis, contra frameworks que "assumem o processo". Pelos nomes: `grill-me`, `grill-with-docs`, `to-spec`, `to-tickets`, `implement`, `code-review`, `tdd`, `diagnosing-bugs`, `handoff`, `write-a-skill`, `writing-great-skills`.

Como encaixar:
- O `/setup-matt-pocock-skills` grava um bloco "Agent skills" no `CLAUDE.md` ou `AGENTS.md`. É o lugar para declarar o **critério de pronto**: "antes de concluir, rode `uiguard` e anexe o relatório".
- Instalar por um único caminho (plugin **ou** `skills.sh`), nunca os dois.
- Instalar o conjunto e depois podar; algumas skills dependem de outras.

Cuidados: contagens inconsistentes entre fontes (27, 38 e 56 skills); parte tem foco em TypeScript; muitas skills disparando pesam no contexto, então preferir as de uso explícito. **[não verificado]**: conteúdo dos `SKILL.md`.

### 4.8 S7 e S8 — opcionais

- **`frontend-design`:** só para páginas novas e sem marca. Em projeto com design system, conflita com fontes e paleta fixas e com "menos é mais".
- **`a11y-audit`:** pipeline Scan, Fix, Verify para WCAG 2.2 A e AA, com `a11y_scanner.py` e `contrast_checker.py`. Lista frameworks (React, Next, Vue, Angular, Svelte, HTML); pode não reconhecer templates como HEEx. **[não verificado]**: qualidade dos scripts.
- **AccessLint:** separa varredura, inspeção manual, correção e diff, e classifica evidência (verificado, a confirmar, exige pessoa). Exige Node 18+ e Chrome DevTools MCP. Fonte de terceiros.

Antes de instalar qualquer skill de terceiros: ler o `SKILL.md` e todos os scripts. Skills instruem o agente e podem incluir comandos de rede ou instruções indesejadas. Número de instalações não mede qualidade.

---

## 5. Ferramentas de código (Python) a construir

### 5.1 Contrato comum

Todas as ferramentas devem falar o mesmo formato, para que o agente leia um único relatório:

```json
{"rule":"motion.layout-property","severity":"error","file":"src/app.css","line":42,
 "message":"transition em 'width' (propriedade de layout)","wcag":null,
 "fix":"animar transform/opacity","evidence":"static"}
```

- **severity:** `error` (bloqueia), `warn`, `info`.
- **evidence:** `static`, `browser`, `needs-human`.
- **Exit code:** 0 sem erros; 1 com erros; 2 falha da própria ferramenta.
- **Saída:** JSON (para o agente) e texto resumido (para pessoas).
- **Config:** um arquivo na raiz do projeto apontando para `tokens.json`, globs de arquivos e limites.

Nome sugerido para o conjunto: `uiguard` (provisório).

### 5.2 Catálogo

| # | Ferramenta | O que verifica | Tipo | Prioridade |
|---|---|---|---|---|
| T1 | `tokens-lint` | Cor, fonte, ícone, espaçamento e duração **literais** fora dos tokens (CSS, HTML, templates, JSX, HEEx) | Estático | Alta |
| T2 | `motion-lint` | Animação de `width`, `height`, `top`, `left`; duração acima do limite; easing fora dos tokens; ausência de `prefers-reduced-motion`; animação infinita sem pausa | Estático | Alta |
| T3 | `contrast-check` | Pares fg/bg dos tokens: 4,5:1 texto normal e 3:1 texto grande e componentes (WCAG 1.4.3 e 1.4.11); estados hover, foco e desabilitado | Estático | Alta |
| T4 | `a11y-static` | `outline: none` sem substituto; `img` sem `alt`; botão e link sem nome; campo sem rótulo; `lang` ausente; alvo de toque menor que 24×24 CSS px (2.5.8) quando calculável | Estático | Média |
| T5 | `a11y-browser` | Playwright + **axe-core** na página renderizada; relatório com critério WCAG | Navegador | Alta |
| T6 | `keyboard-walk` | Percorre a página com Tab: ordem, armadilhas de foco, foco visível, itens sem alcance por teclado | Navegador | Alta |
| T7 | `overlay-test` | Diálogo e menu: abre, foco entra, Tab contido, Esc fecha, foco volta ao gatilho | Navegador | Alta |
| T8 | `rerender-focus` | Dispara a ação que re-renderiza e confere se o foco permanece ou é restaurado | Navegador | Média |
| T9 | `state-contract` | Para cada componente com `data-state`: transita entre estados, confere `aria-busy`, `aria-live`, texto e ícone redundantes (não só cor) | Navegador | Média |
| T10 | `reduced-motion-check` | Emula `prefers-reduced-motion: reduce` e mede animações ativas | Navegador | Média |
| T11 | `visual-regression` | Capturas em 320, 768 e 1280 px, claro e escuro, estados-chave; diff por pixel com tolerância | Navegador | Média |
| T12 | `reflow-check` | Sem rolagem horizontal em 320 px e com zoom de 200% (WCAG 1.4.10) | Navegador | Baixa |
| T13 | `tokens-build` | Gera CSS variables, configuração do utilitário CSS e outros formatos a partir de `tokens.json` | Gerador | Alta |
| T14 | `skill-lint` | Valida frontmatter, tamanho, links e referências órfãs das skills (já existe um validador no skill-creator) | Estático | Baixa |
| T15 | `report` | Consolida todas as saídas em um relatório único e priorizado, com diff entre execuções | Agregador | Alta |

### 5.3 Princípios de implementação

- **Não reescrever o motor de regras de acessibilidade.** Usar axe-core (cobre WCAG 2.0 a 2.2 nos níveis A, AA e AAA) e escrever só as regras próprias.
- **Parsing real, não regex.** Para CSS, usar um parser; para HTML, um parser tolerante. Templates de framework (JSX, HEEx, Vue) exigem *extractors* por linguagem; começar por HTML/CSS e adicionar um por stack.
- **Determinismo.** Mesma entrada, mesmo relatório. Nada de chamadas a modelo dentro das ferramentas.
- **Falso positivo é custo.** Toda regra precisa de forma de suprimir com justificativa (`/* uiguard-ignore regra: motivo */`) e a supressão aparece no relatório.
- **Cada regra tem um teste** com fixture *bad* (deve acusar) e *good* (não pode acusar); ver seção 7.

### 5.4 Ordem de construção

1. T13 `tokens-build` e T1 `tokens-lint`: sem tokens consumidos de forma verificável, o resto perde força.
2. T5 `a11y-browser` + T15 `report`: maior retorno por esforço.
3. T2 `motion-lint`, T3 `contrast-check`, T6 `keyboard-walk`, T7 `overlay-test`.
4. T4, T8, T9, T10.
5. T11 e T12 quando houver componentes estáveis para comparar.

---

## 6. Quando mesclar skill + ferramenta

A pergunta-guia: **a regra é objetiva e verificável por máquina?** Sim → ferramenta. Não → skill. Em muitos casos, os dois.

### 6.1 Casos em que a mescla vale a pena

| Caso | Por que mesclar | Como dividir |
|---|---|---|
| **Geração de artefatos derivados** (tokens → CSS, configuração do utilitário CSS, temas) | Cópia manual diverge; o modelo "arredonda" valores | Script gera; skill diz *quando* regenerar e proíbe editar o gerado |
| **Cálculo** (contraste, escala tipográfica, razão de espaçamento, tamanho de alvo) | Modelos erram aritmética | Script calcula; skill interpreta e decide |
| **Checagem exaustiva e repetitiva** (todas as transições, todos os `outline`) | O modelo esquece itens em arquivos grandes | Script varre; skill prioriza e corrige |
| **Comportamento só observável em execução** (foco, Tab, Esc, re-render) | Não dá para provar lendo código | Playwright prova; skill descreve o comportamento esperado por padrão |
| **Procedimento com ordem rígida** (empacotar, exportar, publicar) | O modelo pula passos | Script executa; skill explica quando e por quê |
| **Regra que a skill repete e o modelo ainda viola** | Instrução textual não basta | Promover a regra a ferramenta e deixar na skill só a justificativa |
| **Critério de pronto** | Evita "terminei" sem prova | Skill exige relatório limpo; ferramenta produz o relatório |

Padrão recomendado de fluxo: **skill decide → modelo implementa → ferramenta verifica → modelo corrige → pessoa revisa o que a máquina não vê.**

### 6.2 Casos em que **não** mesclar

- Decisão de gosto ou de contexto (tom do microcopy, quanta personalidade, hierarquia visual): fica na skill.
- Regra usada uma vez ou que muda toda semana: o custo de manter o script supera o ganho.
- Verificação que exigiria interpretar linguagem natural dentro da ferramenta: vira revisão humana ou do modelo, listada no checklist manual.
- Quando a ferramenta só repete o que o linter do projeto já faz.

### 6.3 Como declarar a ligação na skill

Dentro do `SKILL.md`, uma seção curta:

```markdown
## Verificação
Antes de concluir, rode: `uiguard --only motion,tokens`.
Regras cobertas pela ferramenta: motion.layout-property, tokens.literal-color.
Não coberto (revisar manualmente): quando animar, microcopy, ordem de foco.
```

Isso evita duplicar a regra em texto e em código: a skill aponta a regra da ferramenta e só explica o motivo.

---

## 7. Exemplos HTML + CSS vanilla para uma stack agnóstica

### 7.1 Para que servem

| Uso | Descrição |
|---|---|
| **Especificação visual neutra** | A IA lê o contrato (estados, tempos, atributos) e o traduz para a stack; não copia o código |
| **Fixtures das ferramentas** | Cada exemplo *good* deve passar; cada *bad* deve acusar a regra esperada. É como as ferramentas se testam |
| **Material de revisão humana** | Quem avalia vê o comportamento sem depender de framework |
| **Âncora contra deriva** | Mantém o comportamento esperado estável mesmo trocando de stack |

### 7.2 Formato de cada exemplo

```
examples/
  copy-button/
    SPEC.md          # estados, gatilho, tempos (por token), acessibilidade, microcopy
    good.html        # implementação de referência, só com tokens
    bad.html         # violações intencionais, uma por regra
    expect.json      # regras que o bad.html deve acusar
```

`SPEC.md` mínimo: objetivo, gatilho, estados (`idle → loading → success → idle`), tempos por token, canais de feedback, regras de acessibilidade (nome acessível, `aria-live`, foco, movimento reduzido), critérios WCAG associados e critério de aceite testável.

### 7.3 Contrato portável

Estado no atributo, estilo no CSS, comportamento no que a stack oferecer:

```html
<button class="btn" data-state="idle" aria-live="polite">Copiar link</button>
```

```css
.btn { transition: background-color var(--dur-fast) var(--ease-out); }
.btn[data-state="success"] { background: var(--color-success); }
@media (prefers-reduced-motion: reduce) {
  .btn { transition-duration: 0.01ms; }
}
```

Qualquer stack só precisa trocar o valor de `data-state` (JS, hook de servidor, estado de framework). O CSS e os tokens não mudam. Esse é o ponto de contato com os adaptadores (S4).

### 7.4 Cuidados

- Os exemplos de demonstração que usam CSS puro como "motor" (checkbox hack, `:focus` para animar, `:has`) **não** são código canônico: servem para ver o comportamento sem JavaScript. O exemplo canônico deve usar `data-state` e, quando necessário, JavaScript mínimo.
- Um exemplo não prova que funciona na sua stack. O adaptador e o teste do componente real fecham essa lacuna.
- Manter poucos exemplos, só dos padrões usados de fato (12 a 15). Cada exemplo é mais um artefato a manter.
- Não colocar o HTML inteiro na janela de contexto: indexar por ID e ler só o bloco necessário.

### 7.5 Quando vale incluir exemplos em cada skill

| Skill | Exemplos vanilla? | Observação |
|---|---|---|
| `design-system` | Sim, uma página de amostra de tokens e estados de componente | Serve também para regressão visual |
| `microinteractions` | Sim, por padrão (copiar, toggle, loading de botão, erro de formulário, toast, painel) | Cada um com `good` e `bad` |
| `interaction-accessibility` | Sim, **essencial** (diálogo, menu, combobox, disclosure) | Preferir elementos nativos nos exemplos |
| `stack-adapter-*` | Mínimo: só o trecho que liga `data-state` ao mecanismo da stack | Exemplo completo vira ruído |
| `ui-verification` | Não; usa os fixtures das ferramentas | |

---

## 8. Estrutura de repositório sugerida

```
ai-frontend-stack/
├─ AGENTS.md                      # critério de pronto e skills ativas
├─ skills/
│  ├─ design-system/
│  │  ├─ SKILL.md
│  │  ├─ tokens/tokens.json
│  │  └─ references/
│  ├─ microinteractions/
│  │  ├─ SKILL.md
│  │  ├─ references/
│  │  └─ examples/<padrão>/{SPEC.md,good.html,bad.html,expect.json}
│  ├─ interaction-accessibility/
│  │  ├─ SKILL.md
│  │  ├─ references/patterns/
│  │  └─ examples/<componente>/...
│  ├─ ui-verification/SKILL.md
│  └─ stack-adapter-<stack>/SKILL.md
├─ tools/uiguard/
│  ├─ uiguard/                    # pacote Python
│  ├─ extractors/                 # um por linguagem de template
│  ├─ tests/                      # roda as ferramentas nos fixtures good/bad
│  └─ pyproject.toml
└─ evals/
   ├─ prompts/                    # prompts de avaliação com e sem skill
   ├─ rubric.md
   └─ triggers.md                 # frases que devem e que não devem acionar cada skill
```

---

## 9. O que ainda falta (lacunas)

Itens **não pesquisados**; são hipóteses de lacuna, não conclusões.

| Lacuna | Tipo provável |
|---|---|
| Testes de interface e e2e (como escrever, não só rodar) | Skill |
| Desempenho percebido e real (fontes, imagens, carregamento) | Ferramenta (orçamento de desempenho) + skill curta |
| Layout responsivo e reflow | Ferramenta (T12) + regras nos tokens |
| Formulários e estado (validação, estados de rede, otimismo) | Skill |
| Segurança de front-end (XSS, CSP, dependências) | Skill + ferramenta existente |
| Conteúdo e microcopy por idioma | Skill |
| Modo escuro, alto contraste, `forced-colors` | Tokens + ferramenta |
| Regressão visual entre versões | Ferramenta (T11) |
| Persistência no cliente (chaves, expiração, privacidade) | Convenção do projeto + skill |

---

## 10. Plano de execução

| Fase | Entrega | Critério de saída |
|---|---|---|
| **0. Linha de base** | 10 a 15 prompts reais rodados **sem** skills; lista das falhas recorrentes | Falhas priorizadas por frequência |
| **1. Tokens** | `tokens.json` + `tokens-build` + `tokens-lint` + skill `design-system` | IA gera CSS só com tokens; linter zerado |
| **2. Verificação mínima** | `a11y-browser` + `report` + `overlay-test` + `keyboard-walk` | Relatório único em um componente real |
| **3. Acessibilidade** | Skill adaptada com fichas e exemplos *good/bad* | Fixtures *bad* acusados; *good* limpos |
| **4. Interação** | Revisão da skill de microinterações (seção 4.3) | Melhora mensurável na rubrica |
| **5. Adaptador** | Um `stack-adapter` para a stack principal | Prompts da fase 0 saem corretos nela |
| **6. Fluxo** | `AGENTS.md` com critério de pronto + skills de fluxo escolhidas | Nenhuma tarefa "concluída" sem relatório limpo |
| **7. Poda** | Remover o que não mudou nenhum resultado | Menos contexto, mesmo resultado |

---

## 11. Como medir

- **Mesmo modelo, com e sem a stack.** Comparar com outro modelo mistura duas variáveis.
- **Rubrica binária** por item (0 ou 1), com parte checada automaticamente pelas ferramentas.
- **Teste cego:** quem pontua não sabe qual versão usou a skill.
- **Repetição:** 2 a 3 execuções por prompt e por versão.
- **Metas:** pelo menos +2 itens por prompt e nenhum prompt pior com a stack.
- **Gatilhos:** 10 frases por skill, 5 que devem acionar e 5 que não; ajustar a `description` até acertar quase todas.
- **Prompts-modelo:** botão copiar; formulário de login com erro; fluxo de exclusão; painel lateral de filtros; upload com falha de rede; pedido vago ("deixe mais moderno, com animações").
- **Teste de auditoria:** colar um componente existente e pedir só a auditoria; conferir o formato estruturado e as violações priorizadas.

---

## 12. Riscos

| Risco | Mitigação |
|---|---|
| Skills de terceiros com instruções ou scripts indesejados | Ler `SKILL.md` e scripts antes de instalar; preferir fork com revisão |
| Duas skills de acessibilidade ativas se contradizendo | Escolher uma; escrever regra de precedência |
| Excesso de skills consumindo contexto | `SKILL.md` curto; detalhe em `references/`; poda periódica |
| Ferramenta com muitos falsos positivos | Supressão justificada; fixtures *good*; começar com poucas regras |
| Falsa sensação de conformidade | Relatório sempre lista o que continua manual; ferramentas cobrem só parte |
| Licenças (AGPL, CC-BY) | Registrar origem e licença; CC-BY exige atribuição; validar AGPL com jurídico |
| Popularidade confundida com qualidade | Avaliar com a rubrica própria |
| Dois sistemas definindo o mesmo valor | Tokens como fonte única; skills só referenciam |

---

## 13. Referências

Fontes consultadas em buscas e leituras parciais. Confirmar o conteúdo atual antes de adotar.

- WCAG 2 (repositório): https://github.com/w3c/wcag
- WCAG 2, rascunhos do editor: https://w3c.github.io/wcag/
- ARIA Authoring Practices Guide, padrões: https://www.w3.org/WAI/ARIA/apg/patterns/
- Skill `accessibility` (danielteles, CC-BY-4.0): https://github.com/danielteles/awesome-agent-skills/blob/main/skills/accessibility/SKILL.md
- Skill `wcag-2.2` (Raze-Systems, AGPL-3.0): https://github.com/Raze-Systems/wcag-ai-skill
- Skills de fluxo (Matt Pocock, MIT): https://github.com/mattpocock/skills
- Catálogo de skills e contagem de instalações: https://www.skills.sh/
- Páginas do `mattpocock/skills` no catálogo: https://www.skills.sh/mattpocock/skills

Também usados (sem URL registrada aqui; localizar pelo nome): técnica C39 e critério 2.3.3 do WCAG; documentação de movimento do Fluent 2; "Easing and duration" do Material Design 3; MDN `prefers-reduced-motion`; especificação de tokens do Design Tokens Community Group; repositório do axe-core; skills `a11y-audit` e AccessLint.

---

## 14. Decisões em aberto

1. **Stack principal** do primeiro adaptador (define o escopo de S3 e S4).
2. **Referência oficial de movimento** a adotar como base dos tokens.
3. **Qual skill de acessibilidade de terceiros** adaptar (a recomendação atual é a do danielteles, após abrir `references/`).
4. **Nível alvo:** WCAG 2.2 AA como padrão; AAA só onde houver motivo explícito (por exemplo, o critério 2.3.3 é AAA, mas respeitar `prefers-reduced-motion` é barato).
5. **Onde guardar** skills e ferramentas (repositório próprio, monorepo ou pacote instalável).
