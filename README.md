# Guia Didático: Apache Maven

Este guia explica os principais conceitos do Apache Maven, desde o básico até tópicos mais avançados como multi-módulos e plugins.

---

## 1. O que é Apache Maven e para o que serve

O **Apache Maven** é uma ferramenta de **automação e gerenciamento de build** para projetos, especialmente projetos Java. Ele resolve três grandes problemas do dia a dia de um desenvolvedor:

- **Gerenciamento de dependências**: baixa e organiza as bibliotecas (jars) que seu projeto precisa, sem que você precise baixá-las manualmente.
- **Padronização da estrutura do projeto**: todo projeto Maven segue uma convenção de pastas (`src/main/java`, `src/test/java`, etc.), o que facilita a vida de qualquer desenvolvedor que entre no projeto.
- **Automação do ciclo de build**: compilar, testar, empacotar (jar/war) e publicar o projeto seguem sempre os mesmos passos, de forma reprodutível em qualquer máquina.

O lema do Maven é **"convenção sobre configuração"**: se você seguir a estrutura padrão, precisa configurar muito pouco.

---

## 2. O que é o POM

**POM** significa **Project Object Model** (Modelo de Objeto do Projeto). É o arquivo `pom.xml`, localizado na raiz do projeto, que descreve:

- Identidade do projeto (`groupId`, `artifactId`, `version`)
- Dependências que o projeto usa
- Plugins e configurações de build
- Informações do projeto (nome, licença, desenvolvedores, etc.)

Exemplo mínimo de um `pom.xml`:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.exemplo</groupId>
  <artifactId>meu-projeto</artifactId>
  <version>1.0.0</version>
</project>
```

- **groupId**: identifica a organização/empresa (ex: `com.exemplo`).
- **artifactId**: nome do projeto/módulo (ex: `meu-projeto`).
- **version**: versão do artefato (ex: `1.0.0`, `1.0.0-SNAPSHOT`).

Essas três informações juntas formam as **coordenadas do artefato**, usadas para identificá-lo de forma única em qualquer repositório Maven.

---

## 3. O Super POM

Todo `pom.xml` que você escreve **herda implicitamente** de um POM padrão chamado **Super POM**. Ele é embutido no próprio Maven e define valores-padrão como:

- Diretório de código-fonte: `src/main/java`
- Diretório de testes: `src/test/java`
- Diretório de saída: `target`
- Repositório remoto padrão: Maven Central

Você pode ver o Super POM efetivo (o seu POM já mesclado com o Super POM) rodando:

```bash
mvn help:effective-pom
```

Isso é útil para entender exatamente quais configurações estão realmente ativas no seu build, mesmo as que você nunca declarou explicitamente.

---

## 4. Repositório remoto (Maven Central) e repositório local

O Maven não armazena as dependências dentro do seu projeto. Em vez disso, ele usa um sistema de **repositórios**:

### Repositório remoto: Maven Central
É o repositório público oficial, onde ficam hospedadas a maioria das bibliotecas open-source Java (Spring, JUnit, Gson, etc.). Quando você declara uma dependência, o Maven a busca lá (ou em outros repositórios remotos configurados) caso não a tenha localmente.

### Repositório local
É uma pasta no **seu computador**, geralmente em:

```
~/.m2/repository
```

Funciona como um **cache**: na primeira vez que uma dependência é baixada do repositório remoto, ela é armazenada no repositório local. Nas próximas builds (no mesmo ou em outros projetos), o Maven reutiliza essa cópia local em vez de baixar tudo de novo.

Fluxo resumido:

```
Seu projeto --precisa de dependência--> Repositório Local (~/.m2)
                                              |
                                    não encontrou? busca em
                                              v
                                    Repositório Remoto (Maven Central)
```

---

## 5. Como adicionar dependências

Dependências são adicionadas dentro da tag `<dependencies>` no `pom.xml`. Exemplo, adicionando o JUnit para testes:

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

Passos práticos:

1. Descubra as coordenadas (`groupId`, `artifactId`, `version`) da biblioteca — geralmente pesquisando no site [Maven Central Search](https://search.maven.org).
2. Adicione o bloco `<dependency>` dentro de `<dependencies>` no seu `pom.xml`.
3. Rode `mvn compile` ou deixe sua IDE sincronizar — o Maven baixa a dependência automaticamente.

---

## 6. Publicar localmente (`mvn install`) e reutilizar em outro projeto

Às vezes você quer usar um projeto (ex: uma biblioteca própria) dentro de **outro projeto**, sem publicá-lo em um repositório remoto. Para isso existe o comando:

```bash
mvn install
```

Esse comando:

1. Compila o projeto.
2. Roda os testes.
3. Empacota o artefato (ex: `.jar`).
4. **Copia esse artefato para o repositório local** (`~/.m2/repository`), com base nas coordenadas (`groupId:artifactId:version`) do `pom.xml`.

Depois disso, **qualquer outro projeto na sua máquina** pode declarar essa biblioteca como dependência normalmente:

```xml
<dependency>
  <groupId>com.exemplo</groupId>
  <artifactId>minha-lib</artifactId>
  <version>1.0.0</version>
</dependency>
```

O Maven vai encontrá-la no repositório local, sem precisar buscar na internet. Isso é muito usado durante o desenvolvimento de bibliotecas internas, antes de publicá-las em um repositório remoto real (como Nexus, Artifactory ou Maven Central).

---

## 7. Tipos de dependência, transitividade e escopos

### Dependências transitivas

Quando você adiciona uma dependência, o Maven também baixa **as dependências dessa dependência** — chamadas de **dependências transitivas**. Por exemplo, se sua biblioteca A depende de B, e você depende de A, o Maven traz B automaticamente para o seu projeto.

Você pode visualizar toda a árvore de dependências com:

```bash
mvn dependency:tree
```

### Escopos (scope)

O escopo define **quando e onde** uma dependência está disponível durante o build. Os principais são:

| Escopo | Disponível em | Vai para o pacote final? | Uso típico |
|---|---|---|---|
| `compile` (padrão) | compilação, testes, execução | Sim | Bibliotecas usadas em produção |
| `provided` | compilação, testes | Não | Servlet API, Lombok |
| `runtime` | testes, execução | Sim | Drivers JDBC |
| `test` | apenas testes | Não | JUnit, Mockito |
| `system` | igual a `provided`, mas exige caminho manual do jar | Não | Casos raros/legados |
| `import` | usado apenas em `dependencyManagement` com `pom` | — | BOMs (Bill of Materials) |

---

## 8. Dicas sobre escopos, dependências opcionais e exclusion

### Dependências opcionais (`optional`)

Marcar uma dependência como opcional indica que ela é usada internamente pela sua biblioteca, mas **não deve ser propagada** para quem depende de você (não é transitiva):

```xml
<dependency>
  <groupId>com.exemplo</groupId>
  <artifactId>lib-extra</artifactId>
  <version>1.0.0</version>
  <optional>true</optional>
</dependency>
```

Use isso quando sua biblioteca oferece um recurso extra que exige uma dependência pesada, mas que só será usado por quem realmente precisar dela.

### Exclusion (excluindo dependências transitivas)

Às vezes uma dependência transitiva causa conflito (ex: duas versões diferentes da mesma lib) ou simplesmente não é necessária. Nesse caso, use `<exclusions>`:

```xml
<dependency>
  <groupId>com.exemplo</groupId>
  <artifactId>lib-x</artifactId>
  <version>2.0.0</version>
  <exclusions>
    <exclusion>
      <groupId>com.exemplo</groupId>
      <artifactId>lib-conflitante</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

### Boas práticas gerais

- Prefira `test` e `provided` sempre que a dependência não precisa ir para produção — isso reduz o tamanho do artefato final.
- Use `mvn dependency:tree` para identificar conflitos de versão antes que virem bugs em produção.
- Centralize versões usando `<dependencyManagement>` no POM pai, especialmente em projetos multi-módulo.

---

## 9. Maven Build Lifecycle (Ciclo de Vida do Build)

O Maven organiza o build em **ciclos de vida (lifecycles)**. Um ciclo de vida é uma sequência de **fases (phases)**, e cada fase, por baixo dos panos, executa uma ou mais **goals** de plugins. Entender essa hierarquia é essencial:

```
Lifecycle (ciclo de vida)
   └── Phase (fase) — ex: compile, test, package
          └── Goal (objetivo) — ex: compiler:compile, surefire:test
```

O Maven possui **3 ciclos de vida independentes entre si**: `clean`, `default` e `site`. Rodar um não executa o outro automaticamente (por isso é comum ver comandos como `mvn clean install`, combinando dois ciclos na mesma chamada).

### 9.1 Ciclo `clean`

Responsável por limpar artefatos gerados em builds anteriores. Fases:

```
pre-clean → clean → post-clean
```

- **clean**: remove o diretório `target` (arquivos compilados, jars gerados, relatórios, etc.).

Comando típico:

```bash
mvn clean
```

### 9.2 Ciclo `default`

É o ciclo principal, responsável por **compilar, testar, empacotar e publicar** o projeto. Suas fases mais usadas, em ordem:

```
validate → compile → test → package → verify → install → deploy
```

- **validate**: valida se o projeto está correto e todas as informações necessárias estão disponíveis.
- **compile**: compila o código-fonte.
- **test**: executa os testes unitários.
- **package**: empacota o código compilado no formato definido (jar, war, etc.).
- **verify**: executa verificações adicionais de qualidade/integração.
- **install**: instala o pacote no repositório local (`~/.m2`).
- **deploy**: publica o pacote em um repositório remoto, para outros times/projetos usarem.

**Ponto-chave:** ao rodar uma fase, o Maven executa automaticamente **todas as fases anteriores** dela, na ordem definida. Ou seja, `mvn install` também compila, testa e empacota antes de instalar. Da mesma forma, `mvn package` roda validate, compile e test antes de empacotar.

### 9.3 Ciclo `site`

Responsável por gerar **documentação e relatórios** do projeto (ex: relatório de cobertura de testes, Javadoc, etc.). Fases:

```
pre-site → site → post-site → site-deploy
```

- **site**: gera o site/documentação do projeto (normalmente em `target/site`).
- **site-deploy**: publica essa documentação em um servidor remoto.

### 9.4 Phases vs. Goals

- Uma **phase** (fase) representa uma **etapa do processo de build** (ex: `compile`, `test`). Ela não faz nada sozinha — é apenas um "marco" no ciclo de vida.
- Uma **goal** (objetivo) é a **ação concreta** executada por um plugin (ex: `compiler:compile`, `surefire:test`). É a goal que realmente compila, testa ou empacota.

Cada phase tem uma ou mais goals **vinculadas (bound)** a ela por padrão. Por exemplo, na fase `compile` do ciclo `default`, a goal `compiler:compile` (do `maven-compiler-plugin`) é executada automaticamente.

Você pode:

- Executar uma **phase**, o que dispara todas as goals vinculadas a ela e a todas as fases anteriores:

```bash
mvn test
```

- Executar uma **goal específica** de um plugin diretamente, sem passar pelo ciclo de vida inteiro:

```bash
mvn compiler:compile
```

- Combinar múltiplas phases/goals na mesma chamada, executadas em sequência:

```bash
mvn clean install
```

---

## 10. Projetos multi-módulo

Um projeto **multi-módulo** é um projeto Maven "guarda-chuva" que agrupa vários sub-projetos (módulos), cada um com seu próprio `pom.xml`, mas gerenciados a partir de um **POM pai**.

Estrutura típica:

```
meu-sistema/               (POM pai — packaging: pom)
├── pom.xml
├── modulo-core/
│   └── pom.xml
├── modulo-api/
│   └── pom.xml
└── modulo-web/
    └── pom.xml
```

POM pai (`meu-sistema/pom.xml`):

```xml
<project>
  <groupId>com.exemplo</groupId>
  <artifactId>meu-sistema</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>

  <modules>
    <module>modulo-core</module>
    <module>modulo-api</module>
    <module>modulo-web</module>
  </modules>
</project>
```

Cada módulo filho referencia o pai:

```xml
<parent>
  <groupId>com.exemplo</groupId>
  <artifactId>meu-sistema</artifactId>
  <version>1.0.0</version>
</parent>

<artifactId>modulo-core</artifactId>
```

**Vantagens**:

- Um único `mvn install` na raiz constrói todos os módulos na ordem correta de dependência entre eles.
- Versões e dependências comuns podem ser centralizadas no POM pai (via `<dependencyManagement>`), evitando duplicação.
- Facilita organizar sistemas grandes em partes menores e independentes (ex: `core`, `api`, `web`), mas que continuam versionadas e construídas juntas.

---

## 11. Plugins

Plugins são o **motor real** do Maven: praticamente tudo que o Maven faz (compilar, testar, empacotar) é executado por um plugin por trás dos panos. As fases do ciclo de vida apenas "chamam" plugins configurados para executar naquele momento.

Exemplo: o plugin do compilador, configurando a versão do Java:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.13.0</version>
      <configuration>
        <source>17</source>
        <target>17</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```

Alguns plugins comuns:

- **maven-compiler-plugin**: define a versão do Java usada na compilação.
- **maven-surefire-plugin**: executa os testes unitários na fase `test`.
- **maven-jar-plugin** / **maven-war-plugin**: define como o artefato final é empacotado.
- **maven-shade-plugin** / **maven-assembly-plugin**: gera um "fat jar" (jar com todas as dependências embutidas).
- **spring-boot-maven-plugin**: empacota aplicações Spring Boot como jar executável.

Você também pode executar um plugin diretamente, sem passar por uma fase do ciclo de vida, usando o formato `groupId:artifactId:goal`, por exemplo:

```bash
mvn org.apache.maven.plugins:maven-dependency-plugin:3.6.1:tree
```

Ou, de forma abreviada, quando o plugin é conhecido pelo Maven:

```bash
mvn dependency:tree
```

---

## Resumo rápido

| Conceito | O que é |
|---|---|
| Maven | Ferramenta de build e gerenciamento de dependências |
| POM (`pom.xml`) | Arquivo que descreve o projeto |
| Super POM | POM padrão do qual todo POM herda |
| Repositório local | Cache de dependências na sua máquina (`~/.m2`) |
| Repositório remoto | Fonte pública de bibliotecas (ex: Maven Central) |
| `mvn install` | Compila, testa, empacota e instala no repositório local |
| Escopo | Define onde/quando uma dependência está disponível |
| Lifecycle | Sequência de fases: validate → compile → test → package → verify → install → deploy |
| Multi-módulo | Vários projetos Maven organizados sob um POM pai |
| Plugin | Executa as ações reais por trás das fases do build |