# Git Flow — Padrão da Empresa

> Fluxo de branches adotado em **todos os repositórios** da empresa.

Este documento descreve o modelo Git Flow utilizado para integração contínua, estabilização de versões e deploy em produção. O fluxo possui **duas branches permanentes** (<font color="#9933FF">develop</font> e <font color="#005BAC">main</font>) e **branches temporárias** para desenvolvimento, release, correções em homologação e hotfixes.

---

## Sumário

- [Visão geral](#visão-geral)
- [Diagrama](#diagrama)
- [Branches permanentes](#branches-permanentes)
  - [<font color="#9933FF">develop</font>](#develop)
  - [<font color="#005BAC">main</font>](#main)
- [Branches temporárias](#branches-temporárias)
  - [<font color="#66CC00">feature</font>](#feature)
  - [<font color="#00CCCC">release</font>](#release)
  - [<font color="#FF9933">bugfix</font>](#bugfix-release)
  - [<font color="#FF3333">hotfix</font>](#hotfix)
- [<font color="#999999">Tags</font>](#tags)
- [Resumo de inputs e outputs](#resumo-de-inputs-e-outputs)
- [Regras gerais](#regras-gerais)
- [Checklist por cenário](#checklist-por-cenário)

---

## Visão geral

| Entidade        | Tipo        | Origem      | Destino                            |
| --------------- | ----------- | ----------- | ---------------------------------- |
| <font color="#9933FF">develop</font>       | Permanente  | —           | <font color="#66CC00">feature/*</font>, <font color="#00CCCC">release/*</font>           |
| <font color="#005BAC">main</font>          | Permanente  | —           | <font color="#FF3333">hotfix/*</font>, <font color="#999999">tags</font> de produção       |
| <font color="#66CC00">feature/*</font>     | Temporária  | <font color="#9933FF">develop</font>   | <font color="#9933FF">develop</font>                          |
| <font color="#00CCCC">release/*</font>     | Temporária  | <font color="#9933FF">develop</font>   | <font color="#005BAC">main</font> + <font color="#9933FF">develop</font>                 |
| <font color="#FF9933">bugfix/*</font>      | Temporária  | <font color="#00CCCC">release/*</font> | <font color="#00CCCC">release/*</font>                        |
| <font color="#FF3333">hotfix/*</font>      | Temporária  | <font color="#005BAC">main</font>      | <font color="#005BAC">main</font> + <font color="#9933FF">develop</font>                 |
| <font color="#999999">Tag (`v*.*.*`)</font>  | Marcador    | merge em <font color="#005BAC">main</font> | sync automático em <font color="#9933FF">develop</font> |

---

## Diagrama

![Branching Flow — Todos os repositórios](./assets/git-flow-diagram.png)

---

## Branches permanentes

São as **únicas branches fixas** do repositório. Elas nunca são deletadas e representam os dois estados estáveis do código:

- **<font color="#9933FF">develop</font>** — integração contínua de features concluídas
- **<font color="#005BAC">main</font>** — código em produção

---

# <font color="#9933FF">Develop</font>

| Propriedade   | Valor                                              |
| ------------- | -------------------------------------------------- |
| **Tipo**      | Permanente                                         |
| **Propósito** | Branch de integração; todo trabalho concluído converge aqui antes de ir para produção |

#### Inputs

| Origem              | Quando ocorre                              | Como                                                                 |
| ------------------- | ------------------------------------------ | -------------------------------------------------------------------- |
| <font color="#999999">Tag</font> de produção     | Após deploy de uma nova versão em <font color="#005BAC">main</font>   | CI executa: `git checkout develop && git merge <tag_name>`           |
| <font color="#66CC00">Feature</font> concluída   | PR/MR aprovado e mergeado                  | `git checkout develop && git merge <feature_branch_name>`            |



**Sincronização com produção (tag → develop):**

```bash
git checkout develop
git pull origin develop
git merge v1.0.0          # nome da última tag em produção
git push origin develop
```

> Normalmente executado **automaticamente pela CI** quando uma tag é criada em <font color="#005BAC">main</font>.

#### Outputs

| Destino         | Quando ocorre                                                                 | Comandos |
| --------------- | ----------------------------------------------------------------------------- | -------- |
| <font color="#66CC00">feature/*</font>     | Início de nova funcionalidade, melhoria ou correção não urgente               | [Ver <font color="#66CC00">Feature</font>](#feature) |
| <font color="#00CCCC">release/*</font>     | Critério de corte de versão atingido                                          | [Ver <font color="#00CCCC">Release</font>](#release) |

**Critérios para abrir uma release (exemplos):**

- **Data de corte:** toda quinta-feira pela manhã uma versão é fechada e uma nova release é iniciada
- **Volume:** após N features concluídas e mergeadas (ex.: 3 features prontas)
- **Demanda de negócio:** versão planejada para campanha, evento ou entrega contratual

---

# <font color="#005BAC">Main</font>

| Propriedade   | Valor                                              |
| ------------- | -------------------------------------------------- |
| **Tipo**      | Permanente                                         |
| **Propósito** | Reflete exatamente o que está em produção; código mergeado aqui deve estar pronto para deploy imediato |

#### Inputs

| Origem              | Quando ocorre                    | Como                          |
| ------------------- | -------------------------------- | ----------------------------- |
| <font color="#00CCCC">Release</font> finalizada  | QA/homologação aprovam a release | Merge da <font color="#00CCCC">release/*</font> via PR/MR |
| <font color="#FF3333">Hotfix</font> urgente      | Correção crítica em produção     | Merge da <font color="#FF3333">hotfix/*</font> via PR/MR  |

#### Outputs

| Destino      | Quando ocorre                                      | Comandos |
| ------------ | -------------------------------------------------- | -------- |
| <font color="#999999">Tag (`v*.*.*`)</font> | Imediatamente após merge em <font color="#005BAC">main</font>               | [Ver <font color="#999999">Tags</font>](#tags) |
| <font color="#FF3333">hotfix/*</font>   | Bug crítico em produção que não pode esperar release | [Ver <font color="#FF3333">Hotfix</font>](#hotfix) |

**Criação de tag (exemplo):**

```bash
git checkout main
git pull origin main
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0
```

> A tag dispara o pipeline de deploy para produção e, em seguida, a sincronização automática com <font color="#9933FF">develop</font>.

---

## Branches temporárias

Branches temporárias existem por um **propósito específico** e um **período limitado**. Devem ser **deletadas após o merge** (local e remoto).

---

## <font color="#66CC00">Feature</font>

| Propriedade        | Valor                                                        |
| ------------------ | ------------------------------------------------------------ |
| **Tipo**           | Temporária                                                     |
| **Duração típica** | Dias a poucas semanas (vida do ticket/issue)                   |
| **Por que existe** | Isolar desenvolvimento de funcionalidade sem impactar <font color="#9933FF">develop</font> |

#### Ciclo de vida

```
develop → feature/* → develop → (delete feature/*)
```

| Momento  | Ação                                              |
| -------- | ------------------------------------------------- |
| **Nasce**  | Início de um card/ticket de desenvolvimento       |
| **Vive**   | Commits incrementais + PR para revisão de código  |
| **Morre**  | Após merge em <font color="#9933FF">develop</font> e delete da branch remota |

#### Convenção de nome

```
feature/<ticket-id>-<descricao-curta>
```

**Exemplos:**

- `feature/APP-1234-login-biometrico`
- `feature/APP-5678-carrinho-multi-filial`

#### Comandos

```bash
# Criar feature a partir de develop atualizada
git checkout develop
git pull
git checkout -b feature/APP-1234-login-biometrico

# Trabalhar e publicar
git add .
git commit -m "feat(login): adiciona autenticação biométrica"
git push -u origin feature/APP-1234-login-biometrico

# Após aprovação do PR — merge em develop (via plataforma)

# Limpeza
git push origin --delete feature/APP-1234-login-biometrico
```

#### Inputs / Outputs

| Direção  | Detalhe                                      |
| -------- | -------------------------------------------- |
| **Input**  | Parte sempre de <font color="#9933FF">develop</font> atualizada       |
| **Output** | Merge de volta em <font color="#9933FF">develop</font> via Pull Request |

#### Commits

Seguir **Conventional Commits** (documento complementar). Exemplos: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`.

---

## <font color="#00CCCC">Release</font>

| Propriedade        | Valor                                                                 |
| ------------------ | --------------------------------------------------------------------- |
| **Tipo**           | Temporária                                                              |
| **Duração típica** | Alguns dias (QA, homologação e ajustes finais)                        |
| **Por que existe** | Congelar features para estabilização e deploy, sem bloquear <font color="#9933FF">develop</font> |

#### Ciclo de vida

```
develop → release/* → main (+ tag) → develop → (delete release/*)
                ↑
            bugfix/*
```

| Momento  | Ação                                                          |
| -------- | ------------------------------------------------------------- |
| **Nasce**  | Critério de corte de versão atingido                          |
| **Vive**   | QA, homologação, bugfixes, bump de versão, tags beta          |
| **Morre**  | Após merge em <font color="#005BAC">main</font> (com tag) |

#### Convenção de nome

```
release/<major>.<minor>.<patch>
```

**Exemplo:** `release/1.0.0`

#### Comandos

```bash
# Criar release a partir de develop
git checkout develop
git pull
git checkout -b release/1.0.0
git push -u origin release/1.0.0

# Marcar versão beta (opcional, durante homologação)
git tag -a v1.0.0-beta.0 -m "Beta 0 da release 1.0.0"
git push origin v1.0.0-beta.0

# Após QA aprovado — merge em main
git checkout main
git pull
git merge release/1.0.0
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin main --tags

# Limpeza
git push origin --delete release/1.0.0
```

#### Inputs / Outputs

| Direção  | Detalhe                                                          |
| -------- | ---------------------------------------------------------------- |
| **Inputs**  | <font color="#9933FF">develop</font> (snapshot no corte) + merges de <font color="#FF9933">bugfix/*</font>          |
| **Outputs** | Merge em <font color="#005BAC">main</font> (gera tag) + merge de retorno em <font color="#9933FF">develop</font>    |

---


## <font color="#FF9933">bugfix (release)</font>


| Propriedade        | Valor                                                              |
| ------------------ | ------------------------------------------------------------------ |
| **Tipo**           | Temporária                                                           |
| **Duração típica** | Horas a 1–2 dias                                                     |
| **Por que existe** | Corrigir defeitos encontrados **durante QA/homologação** da release |

> **Atenção:** <font color="#FF9933">bugfix</font> **≠** <font color="#FF3333">hotfix</font>. <font color="#FF9933">Bugfix</font> parte da <font color="#00CCCC">release/*</font> (pré-produção). <font color="#FF3333">Hotfix</font> parte de <font color="#005BAC">main</font> (produção).

#### Ciclo de vida

```
release/* → bugfix/* → release/* → (delete bugfix/*)
                
```

| Momento  | Ação                                    |
| -------- | --------------------------------------- |
| **Nasce**  | Bug encontrado na fase de testes da release |
| **Vive**   | Correção + reteste                      |
| **Morre**  | Após merge de volta na <font color="#00CCCC">release/*</font>      |

#### Convenção de nome

```
bugfix/<ticket-id>-<descricao-curta>
```

**Exemplo:** `bugfix/APP-9012-corrige-crash-checkout`

#### Comandos

```bash
# Criar bugfix a partir da release ativa
git checkout release/1.0.0
git pull origin release/1.0.0
git checkout -b bugfix/APP-9012-corrige-crash-checkout

git add .
git commit -m "fix(checkout): corrige crash ao aplicar cupom"
git push -u origin bugfix/APP-9012-corrige-crash-checkout

# Merge de volta na release (via PR)
git checkout release/1.0.0
git merge bugfix/APP-9012-corrige-crash-checkout
git push origin release/1.0.0

# Nova tag beta (se aplicável)
git tag -a v1.0.0-beta.1 -m "Beta 1 — corrige crash checkout"
git push origin v1.0.0-beta.1

# Limpeza
git push origin --delete bugfix/APP-9012-corrige-crash-checkout
```

#### Inputs / Outputs

| Direção  | Detalhe                                                                 |
| -------- | ----------------------------------------------------------------------- |
| **Input**  | Parte sempre da <font color="#00CCCC">release/*</font> ativa                                     |
| **Output** | Merge de volta na mesma <font color="#00CCCC">release/*</font>                                     |

---

## <font color="#FF3333">Hotfix</font>

| Propriedade        | Valor                                                        |
| ------------------ | ------------------------------------------------------------ |
| **Tipo**           | Temporária                                                     |
| **Duração típica** | Horas (urgência máxima)                                        |
| **Por que existe** | Corrigir bug **crítico já em produção** sem esperar release  |

#### Ciclo de vida

```
main → hotfix/* → main (+ tag patch) → develop → (delete hotfix/*)
```

| Momento  | Ação                                      |
| -------- | ----------------------------------------- |
| **Nasce**  | Incidente crítico em produção             |
| **Vive**   | Correção mínima, testes focados, deploy   |
| **Morre**  | Após merge em <font color="#005BAC">main</font>          |

#### Convenção de nome

```
hotfix/<major>.<minor>.<patch>
```

ou

```
hotfix/<ticket-id>-<descricao-curta>
```

**Exemplos:** `hotfix/1.0.1` · `hotfix/APP-9999-corrige-pagamento`

#### Comandos

```bash
# Criar hotfix a partir de main (produção)
git checkout main
git pull origin main
git checkout -b hotfix/1.0.1

git add .
git commit -m "fix(payment): corrige falha no PIX"
git push -u origin hotfix/1.0.1

# Merge em main + nova tag
git checkout main
git merge hotfix/1.0.1
git tag -a v1.0.1 -m "Hotfix 1.0.1 — corrige falha PIX"
git push origin main --tags

# Limpeza
git push origin --delete hotfix/1.0.1
```

#### Inputs / Outputs

| Direção   | Detalhe                                                          |
| --------- | ---------------------------------------------------------------- |
| **Input**   | Parte sempre de <font color="#005BAC">main</font> (estado atual de produção)              |
| **Outputs** | Merge em <font color="#005BAC">main</font> (tag patch) |

> **Regra:** <font color="#FF3333">hotfix</font> corrige o **mínimo necessário**. Melhorias ou refactors pertencem a uma <font color="#66CC00">feature/*</font>.

---

## <font color="#999999">Tags</font>

| Propriedade   | Valor                                              |
| ------------- | -------------------------------------------------- |
| **Tipo**      | Marcador permanente (não é branch)                 |
| **Propósito** | Identificar pontos exatos do histórico deployados  |

### <font color="#999999">Tipos de tag</font>

| Tag          | Quando                         | Exemplo                          |
| ------------ | ------------------------------ | -------------------------------- |
| **Beta**     | Durante homologação da release | `v1.0.0-beta.0`, `v1.0.0-beta.1` |
| **Produção** | Após merge em <font color="#005BAC">main</font>           | `v1.0.0`, `v1.0.1`               |

### <font color="#999999">Fluxo</font>

```
release → merge em main → tag v1.0.0 → CI sync em develop
hotfix  → merge em main → tag v1.0.1 → CI sync em develop
```

A seta de retorno **<font color="#999999">tag</font> → <font color="#9933FF">develop</font>** no diagrama representa:

```bash
git checkout develop
git merge v1.0.0   # ou v1.0.1, etc.
```

Executado pela **CI automaticamente** após cada tag de produção.

---

## Resumo de inputs e outputs

# <font color="#9933FF">Develop</font>

| Inputs                         | Outputs      |
| ------------------------------ | ------------ |
| <font color="#999999">Tag</font> de produção (via CI)       | <font color="#66CC00">feature/*</font>  |
| Merge de <font color="#66CC00">feature</font> concluída     | <font color="#00CCCC">release/*</font>  |

# <font color="#005BAC">Main</font>

| Inputs                  | Outputs                    |
| ----------------------- | -------------------------- |
| Merge da <font color="#00CCCC">release</font>        | <font color="#999999">Tag</font> de produção (`v*.*.*`) |
| Merge do <font color="#FF3333">hotfix</font>         | <font color="#999999">Tag</font> patch (`v*.*.*`)       |
| —                       | <font color="#FF3333">hotfix/*</font>                 |

## <font color="#66CC00">Feature</font>

| Inputs    | Outputs           |
| --------- | ----------------- |
| <font color="#9933FF">develop</font> | Merge em <font color="#9933FF">develop</font> |

## <font color="#00CCCC">Release</font>

| Inputs      | Outputs                              |
| ----------- | ------------------------------------ |
| <font color="#9933FF">develop</font>   | Merge em <font color="#005BAC">main</font> + <font color="#999999">tag</font>                |
| Merge do <font color="#FF9933">bugfix/*</font>  |     <font color="#FF9933">bugfix/*</font>   |

## <font color="#FF9933">bugfix (release)</font>

| Inputs       | Outputs              |
| ------------ | -------------------- |
| <font color="#00CCCC">release/*</font>  | Merge em <font color="#00CCCC">release/*</font> |

## <font color="#FF3333">Hotfix</font>

| Inputs  | Outputs                              |
| ------- | ------------------------------------ |
| <font color="#005BAC">main</font>  | Merge em <font color="#005BAC">main</font> + <font color="#999999">tag</font>    |

---

## Regras gerais

1. **<font color="#9933FF">develop</font> e <font color="#005BAC">main</font> nunca são deletadas.**
2. **Branches temporárias devem ser deletadas** após merge (local e remoto).
3. **Toda branch temporária nasce de uma branch atualizada** (`git pull` antes de `checkout -b`).
4. **<font color="#999999">Tags</font> de produção só são criadas em <font color="#005BAC">main</font>**, nunca em <font color="#9933FF">develop</font> ou <font color="#66CC00">feature</font> branches.
5. **<font color="#FF9933">Bugfix</font> ≠ <font color="#FF3333">Hotfix</font>:** <font color="#FF9933">bugfix</font> parte da <font color="#00CCCC">release</font> (pré-produção); <font color="#FF3333">hotfix</font> parte de <font color="#005BAC">main</font> (produção).
6. **Commits seguem Conventional Commits** — documento complementar a definir.
7. **Merges preferencialmente via Pull Request/MR**, com revisão de código obrigatória (exceto <font color="#FF3333">hotfix</font> crítico, com processo acelerado).

---

## Checklist por cenário

| Cenário                  | Branch base  | Merge em              | Tag?              | Delete branch? |
| ------------------------ | ------------ | --------------------- | ----------------- | -------------- |
| Nova funcionalidade      | <font color="#9933FF">develop</font>    | <font color="#9933FF">develop</font>             | Não               | Sim            |
| Fechar versão            | <font color="#9933FF">develop</font>    | <font color="#005BAC">main</font> + <font color="#9933FF">develop</font>    | Sim (`v*.*.*`)    | Sim            |
| Bug na homologação       | <font color="#00CCCC">release/*</font>  | <font color="#00CCCC">release/*</font>           | Beta (opcional)   | Sim            |
| Bug crítico em produção  | <font color="#005BAC">main</font>       | <font color="#005BAC">main</font>    | Sim (patch)       | Sim            |

---

# Padrões de Merge Request

> Detalhes e boas práticas que os times devem se atentar na hora de criar a aceitar MRs

---

## Sumário
- [Configurações iniciais](#configuracoes-iniciais)
- [Pontos de Atenção](#pontos-de-atencao)
- [Descrição de MR](#descricao-de-mr)
  - [Template de descrição](#template-de-descricao)

---

## Configurações iniciais

No repositório do GitLab, na aba de navegação <font color="#0000EC">**Code > Merge requests**</font>, há um botão <font color="#0000EC">**"New merge request"**.</font> ​

Ela o navegará para a tela de criação de MR, onde você deve selecionar a **<font color="#9933FF">_Source branch_ (fonte que possui as alterações)</font>** e a **<font color="#00B377">_Target branch_ (alvo para receber as alterações)</font>.​**

> Fique atento para selecionar as branches corretas, conforme o **Git Flow** do projeto!​

---

## Pontos de Atenção

Para manter a visibilidade adequada à revisão do código e evitar a criação de bugs, <font color="#0000EC">**o propósito e o escopo das alterações**</font> devem ser claros para todos.​

---

## Descrição de MR

> Ganho: padronização e visibilidade ao reviewer​

Todas as MRs (de <font color="#66CC00">feature</font>, <font color="#FF9933">bugfix</font>, mas principalmente de <font color="#00CCCC">release</font>) devem estar descritas e ter seu propósito claro para o revisor (reviewer). 

Para isso, por padrão, todos os projetos possuem um <font color="#0000EC">**template de descrição**,</font> cujo responsável da MR deve preencher o <font color="#0000EC">**tipo e a descrição e linkar os cartões do Business Map**</font> relacionados àquela entrega.​ Caso a equipe entenda necessário, poderá adicionar informações extra à template de seus projetos.

Durante a revisão, o reviewer poderá utilizar o <font color="#0000EC">**checklist**</font> para registrar os pontos analisados e manter um histórico de sua avaliação.

---

### Template de descrição:

```bash
# Tipo da mudança

- [ ] Nova funcionalidade
- [ ] Refatoração
- [ ] Correção de bug

# Descrição

_Descrição da mudança que está sendo implementada_

# Cartões relacionados

_Adicione o título e link dos cartões relacionados às entregas do MR_

[COD-XXX]()

# Checklist

- [ ] Código segue os padrões de desenvolvimento do projeto
- [ ] Código condiz com a documentação
- [ ] Testes unitários estão implementados
```
---
# Conventional Commits

> Convenção e padrão de nomenclatura de commits

---

## Sumário
- [Visão Geral](#visao-geral-conventional-commits)
- [Estrutura](#estrutura)
- [Tipos de commit](#tipos-de-commit)
  - [Exemplos](#exemplos)

---

## Visão Geral Conventional Commits

<font color="#00B377">**Commit semântico** (ou _conventional commit_)</font> é uma padronização de nomenclaturas de commits no projeto. Para isso, utiliza-se regras simples e claras.

> <font color="#04B377">Apesar de adicionar uma pequena carga adicional de trabalho, isso reduz o tempo gasto de compreensão das alterações posteriormente.</font>

Cada commit deve ter um <font color="#009CFF">**escopo claro**</font> e seguir o <font color="#009CFF">**princípio da responsabilidade única**</font>: ter apenas um único propósito e conter somente os arquivos alterados ligadas a esse propósito. 

>Isso facilita o versionamento semântico, a leitura do histórico e a observabilidade das mudanças no repositório!

Sempre que possível, cada commit deve deixar o código em <font color="#FF3333">**estado executável** </font> (sem erros de compilação nem falhas em tempo de execução). 

>Isso facilita a revisão, reversão e identificação de falhas!

---

## Estrutura

A estrutura de um commit semântico é clara e de fácil identificação, pois utiliza um formato definido na sua sintaxe:

```
<tipo>[escopo opcional]: <descrição>
```

A primeira e principal descrição de um commit semântico refere-se ao seu <font color="#9933FF">**tipo**</font>, que possui a finalidade de <font color="#9933FF">comunicar a intenção daquela alteração.</font>

---

## Tipos

Os principais tipos utilizados nos projetos da companhia são:

| Tipo    | Descrição                                                                                                                                                                  |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <font color="#00CF66">`feat`  | Adições de novas funcionalidades ou de quaisquer outras novas implantações ao código.                                                                                      |
| <font color="#FF9933">`fix`   | Tratamentos e correções de bugs.                                                                                                                                           |
| <font color="#9933FF">`docs`  | Inclusão ou alteração de documentação.                                                                                                                                     |
| <font color="#005BAC">`refac` | Mudanças que não alteram a funcionalidade final da tarefa impactada.                                                                                                       |
| <font color="#00BCFF">`test`  | Adição ou correção de testes no processo de testes automatizados.                                                                                                          |
| `chore` | Atualização de tarefas que não ocasionam alterações no código de produção, mas mudanças de ferramentas, configurações e bibliotecas que geralmente não entram em produção. |

---

## Exemplos

```
feat(BC Account): nova rota para o login
```
```
fix(684887): erro ao salvar pedido
```
```
test: testes do fluxo de buscar dados de usuário
```
```
chore: upgrade de versão do pacote
```

---

# Versionamento Semântico

> Upgrades de versão lógicos padronizados

Aqui você irá encontrar a explicação sobre Versionamento Semântico e quando utilizar cada upgrade.

---

## Sumário
- [Major](#major)
- [Minor](#minor)
- [Patch](#patch)
- [Resumo](#resumo)

---

## <font color="#FF0F4F">MAJOR</font>

> _1.0.0 -> 2.0.0_

Um upgrade de versão <font color="#FF0F4F">**major**</font> deve ser feito quando há uma <font color="#FF0F4F">**quebra de contrato </font> ou <font color="#FF0F4F">alteração estrutural intensa** </font>que gere <font color="#FF0F4F">**incompatibilidade**.</font>

Se constitui uma quebra de contrato: 
- **alteração de atributos obrigatórios da chamada;**
- **alteração do objeto de resposta;**
- **alteração de nomenclatura do contrato;**
- **alteração de endereço do contrato.**

Exemplos:

```
v2.4.5 -> v3.0.0: Remoção de rotas de cupom
```
```
v4.0.1 -> v5.0.0: Atualização do Java 13 para Java 21
```

---

## <font color="#009CFF">MINOR</font>

> _1.0.0 -> 1.1.0_

Um upgrade de versão <font color="#009CFF">**minor**</font> deve ser feito quando há uma <font color="#009CFF">**adição de funcionalidade ou evolução**</font>, como novos contratos, chamadas a serviços, telas e componentes.

<font color="#009CFF">**Refatorações que influenciam a experiência do usuário**</font> podem ser consideradas minors, desde que não quebrem contratos já estabelecidos.

Exemplos:

```
v2.2.5 -> v2.3.0: Adição de fluxo de confirmação de conta
```
```
v1.15.24 -> v1.16.0: Movimentação da tela de Saúde para a Bottom Bar do app
```

---

## <font color="#9933FF">PATCH</font>

> _1.0.0 -> 1.0.1_

Um upgrade de versão <font color="#9933FF">**patch**</font> deve ser feito quando há uma <font color="#9933FF">**correção de funcionalidade já existente**</font>, como bugfix, hotfix ou modificação mínimas de tokens estéticos.

Exemplos:

```
v5.1.9 -> v5.1.10: Correção no envio de logs ao DataDog
```
```
v1.10.2 -> v1.10.3: Remoção de ícone duplicado no card de produto
```

---

## Resumo

| Versão  | Upgrade            |Cenário                  |
| ------- | ------------------ | ------------------------|
| <font color="#FF0F4F">Major</font>  | 1.0.0 > 2.0.0    | Quebra de contrato ou incompatibilidade|
|<font color="#009CFF">Minor</font>| 1.0.0 > 1.1.0 | Nova funcionalidade ou evolução|
| <font color="#9933FF">Patch</font> | 1.0.0 > 1.0.1 | Correção de funcionalidade já existente

---

# Changelog

Changelog é uma documentação que conta a **linha do tempo** de um projeto, enumerando as mudanças significativas para cada versão.

---

## Princípios fundamentais

- Changelogs são para <font color="#00B377">**humanos, não máquinas**</font>. Mantenha a linguagem <font color="#00B377">**simples e compreensível!**</font>
- Deve haver uma entrada para <font color="#00B377">**cada nova versão**</font> da aplicação registrando as <font color="#00B377">**principais alterações e sua data de lançamento.** </font>
- A ordem das versões é decrescente <font color="#00B377">**(a mais recente acima)**.</font>
- Siga o <font color="#00B377">**Versionamento Semântico**</font> (major, minor, patch).

---

## Implementação

Para implementar um changelogs na sua aplicação RD, basta criar um arquivo nomeado <font color="#00B377">**CHANGELOG.md**</font> no repositório do projeto e seguir o modelo de exemplo:

```bash
## v2.0.0 - 25/05/2026
Sintexe: Essa versão remove a rota depreciada, faz algumas melhorias visuais e compatibiliza o código com novas tecnologias.

### Adicionado
- feat: adicionado token de estilização

### Corrigido
- fix: correção de imagem duplicada
- test: correção de testes de login falhos

### Removido
- feat: rota depreciada de login removida

### Alterado
- chore: upgrade de versão do Mongo para 1.2.3

---

## v1.3.0 - 25/05/2026
Sintexe: Essa versão adiciona uma rota nova de login e seus testes.

### Adicionado
- feat: nova rota para login
- feat: testes unitários de login

### Corrigido
- fix: correção de envio de logs ao DataDog

### Alterado
- chore: aviso de depreciação para antiga rota de login


```
 
