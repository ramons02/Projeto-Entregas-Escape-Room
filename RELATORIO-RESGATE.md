# Relatório de Resgate
- Equipe: Ramon
- Data: 06/10/2026
- Branch de trabalho: `resgate/equipe-ramon` (criada a partir de `release-dev`, `799c5ec`)

## Diagnóstico
Estado inicial: 7 branches locais, branch ativa `release-dev`, tags `v1.0.0-funcional` (`356ec6f`, versão boa) e `commit-perigoso` (`6572d8a`).

| # | Problema | Commit causador (autor) | Evidência |
|---|---|---|---|
| 1 | Não compilava: `EntregaService` removeu o `endereco` do construtor de `Mercadoria` e chamou `repository.gravar` (inexistente; o método é `salvar`) | `a70ee84` (Henrique Nunes) | `mvn clean package` + `git diff v1.0.0-funcional -- .../EntregaService.java` |
| 2 | Arquivo excluído: `util/Validador.java`, que o `EntregaService` usa (não era "sem uso") | `64f88f6` (Igor Reis) | `git log --diff-filter=D --summary`; erro `package br.edu.entregas.util does not exist` |
| 3 | Login com erro lógico: `\|\|` no lugar de `&&` e `senha == SENHA` (compara referência). `admin` entrava com qualquer senha | `9a6d3b0` (Felipe Rocha) | `git diff v1.0.0-funcional -- .../LoginService.java` |
| 4 | README reduzido a "Pergunte ao desenvolvedor como executar" | `0cd80f6` (Gustavo Melo) | `git log --all -- README.md` |
| 5 | Credenciais versionadas em `config/application.properties` (`db.user`, `db.password`, `api.token`) | `6572d8a` (Felipe Rocha) | `git log --all -- config/application.properties`, `git show commit-perigoso` |

Observações:
- O commit `9a6d3b0` do login é ancestral da `release-dev`; a branch `teste-login-descartavel` apenas aponta para ele.
- `hotfix-login` (`a6f356d`) contém só `NOTA-HOTFIX.txt`, nenhuma correção de código, apesar do nome.
- `docs-readme` (`7fe8faa`, Carla Souza) guardava o README bom; `feature-cadastro` (`2624e5c`) só acrescenta uma linha de validação e não foi integrada.

## Comandos Git utilizados
| Comando | Finalidade |
|---|---|
| `git status`, `git branch -a` | Ver o estado do repositório e as branches |
| `git switch -c resgate/equipe-ramon` | Criar a branch de resgate sem trabalhar na `main` |
| `git log --oneline --all --graph --decorate` | Mapear o histórico, branches e tags |
| `git log --stat --all` | Ver os arquivos alterados em cada commit |
| `mvn clean package` | Reproduzir o erro de compilação e validar as correções |
| `git log -- <arquivo>` | Descobrir quais commits tocaram `EntregaService` e `LoginService` |
| `git diff v1.0.0-funcional -- <arquivo>` | Comparar o arquivo atual com a versão funcional |
| `git show v1.0.0-funcional:<arquivo> > <arquivo>` | Restaurar `EntregaService` e `LoginService` da versão funcional |
| `git log --diff-filter=D --summary` | Localizar o arquivo excluído, o commit e o autor |
| `git checkout 64f88f6^ -- src/main/java/br/edu/entregas/util/Validador.java` | Recuperar o arquivo do commit anterior à exclusão |
| `git log --all -- README.md`, `git show docs-readme:README.md` | Achar e ler o README bom |
| `git checkout docs-readme -- README.md` | Recuperar o README sem reescrevê-lo |
| `git log --all -- config/application.properties`, `git show commit-perigoso` | Auditar o commit com credenciais |
| `git rm config/application.properties` | Remover o arquivo sensível da versão atual |
| `git check-ignore -v config/application.properties` | Confirmar que o `.gitignore` bloqueia o arquivo |
| `git switch main` + `git merge --no-ff resgate/equipe-ramon` | Integrar o resgate preservando o histórico |

## Commits relevantes
**Investigados (causadores)**
- `a70ee84` Henrique Nunes: quebrou `EntregaService`
- `64f88f6` Igor Reis: excluiu `Validador.java`
- `9a6d3b0` Felipe Rocha: quebrou o login
- `0cd80f6` Gustavo Melo: simplificou o README
- `6572d8a` Felipe Rocha: versionou credenciais (tag `commit-perigoso`)

**Correções na branch de resgate**
- `c69b405` restaura `EntregaService` (endereço e `salvar`)
- `52d350c` recupera `Validador.java`
- `5e9565e` corrige a autenticação em `LoginService`
- `21a75e3` restaura o README de `docs-readme`
- `4a8cca2` remove `config/application.properties` e atualiza o `.gitignore`
- `f520e9f` ajusta documentação do desafio

**Integração:** merge `853bcbe` na `main` (`--no-ff`).

**Fonte das versões boas:** `v1.0.0-funcional` (`356ec6f`, Bruno Costa) e `docs-readme` (`7fe8faa`, Carla Souza).

## Validação final
- **Compilação:** `mvn clean package` com exit 0, gerando `target/projeto-entregas-1.0.0.jar`; repetido na `main` após o merge.
- **Login:** `java -cp target/classes br.edu.entregas.Main`
  - `admin` / `12345678`: acesso autorizado
  - `admin` / `errada`: acesso negado
  - `outro` / `12345678`: acesso negado
  - `outro` / `errada`: acesso negado (teste direto em `LoginService`)
- **Cadastro:** a mercadoria é listada com o endereço de entrega: `#1 | Notebook | ... | AGUARDANDO ENVIO | Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000`.
- **README:** restaurado com requisitos (Java 17+, Maven 3.8+), compilação, execução e credenciais de demonstração.
- **Segurança:** `config/application.properties` removido da versão atual (`git ls-files config` retorna 0) e ignorado no `.gitignore`.

### Risco de segurança
Remover o arquivo da versão atual **não elimina o segredo do histórico**. O commit `6572d8a` continua acessível pelo hash, pela tag `commit-perigoso` e pela branch `release-dev`; qualquer clone antigo recupera os valores com `git show 6572d8a`. A senha do banco e o token da API devem ser considerados **comprometidos e rotacionados**. Reescrever o histórico (`git filter-repo`/BFG) exige force-push e invalida os clones existentes, e foi evitado para preservar as evidências do resgate.

### Aprendizados
- Histórico, tags e `git diff` contra uma versão conhecida localizam a origem de cada falha, em vez de só corrigir o sintoma.
- Mensagens de commit podem enganar ("sem uso", "fix", "hotfix"); é preciso conferir o diff.
- Recuperar pelo Git (`checkout`, `show`) mantém a rastreabilidade; reverter com commits próprios preserva as evidências.
- Segredo versionado é incidente de segurança, não apenas limpeza de arquivo.
