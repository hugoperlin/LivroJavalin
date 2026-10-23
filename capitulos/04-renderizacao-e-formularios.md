# Capítulo 4: Renderização de Html e Formulários

Conforme descrito anteriormente, um sistema web é divido em duas partes: **front-end** que é responsável pela interface gráfica de interação do usuário com o sistema e geralmente é descrita utilizando páginas `html`; **back-end** que é o "programa" propriamente dito que será executado.

Toda a interface gráfica do sistema será definida através de arquivos `html`. Em um projeto utilizando Javalin, esses arquivos devem estar localizados dentro da pasta de recursos do projeto (`src/main/resources/public`). Assim um projeto terá a seguinte estrutura.

```text
src/
└── main/
    ├── java/
    │   ├── modelo/
    |   |   ├── entidades/ 
    │   │   |   └── Usuario.java
    |   |   └── Gerenciador.java 
    │   ├── controle/
    │   │   └── CadastroController.java
    │   └── App.java
    └── resources/
        └── public/
            ├── index.html
            └── cadastro.html
```

Os arquivos `html` servem de modelos (_templates_) para que servidor Java gere as páginas HTML dinamicamente antes de enviá-las ao navegador do usuário. Essa abordagem é conhecida como **Server-Side Rendering (SSR)**.

Para realizar essa tarefa no Javalin sem poluir o código Java com tags HTML, utilizamos um **Template Engine** (Motor de Modelos). O escolhido para a nossa disciplina é o **Thymeleaf**.


---

## 🏗️ O que é o Thymeleaf?

O Thymeleaf é um motor de templates HTML dinâmico para Java. Sua principal vantagem é a filosofia de **Natural Templates**: os arquivos de visualização são arquivos HTML5 válidos que podem ser abertos e visualizados diretamente em qualquer navegador, mesmo sem o servidor Java estar rodando.

O Thymeleaf utiliza atributos especiais com o prefixo `th:` nas próprias tags HTML para injetar dados e executar lógicas de exibição vindas do Java.

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│  Objetos Java   │ ────► │ Engine Thymeleaf│ ────► │  Página HTML    │
│ (Model/Context) │       │ (Processa th:*) │       │  (Gerada/SSR)   │
└─────────────────┘       └─────────────────┘       └─────────────────┘
```

---

## Configurando as Dependências (Maven)

Para utilizar o Thymeleaf com o Javalin 6+, precisamos adicionar o módulo de renderização e o próprio motor do Thymeleaf ao nosso arquivo `pom.xml`:

```xml
<!-- Módulo de Renderização do Javalin -->
<dependency>
    <groupId>io.javalin</groupId>
    <artifactId>javalin-rendering</artifactId>
    <version>6.7.0</version>
</dependency>

<!-- Engine do Thymeleaf -->
    <dependency>
      <groupId>org.thymeleaf</groupId>
      <artifactId>thymeleaf</artifactId>
      <version>3.1.5.RELEASE</version>
    </dependency>
```

A classe JavalinUtils já configura o servidor para utilizar o Thymeleaf como motor de renderização.

---

## Renderizando um arquivo

Para que seja possível renderizar um arquivo `html` como forma de resposta a uma requisição, iremos invocar o método `render` do objeto `Context`.

No exemplo abaixo, quando o método `get` da classe `Controler` for invocado, o arquivo `index.html` será carregado e processado pelo Javalin, enviando o seu conteúdo como forma de resposta para a requisição.

```java

public class Controler{

    public void get(Context ctx){
        ctx.render("index.html");
    }

}

```

---

## Formulários Html e Envio de Dados

Quando desenvolvemos um sistema web, é comum enviarmos dados do cliente (navegador) para serem processados pelo servidor. Uma das formas mais comuns de interação é através do preenchimento de campos de um formulário `html`. 

No exemplo abaixo é definido um formulário com dois campos de texto, um para o nome e outro para o e-mail. Note que para ambos as propriedades `name` e `id` foram definidas, indicando uma identificação única dentro do formulário. Essa informação é importante, pois ela servirá para recuperar os dados no servidor.

```html
<!-- cadastro.html -->
<html>
<body>
    <form action="/cadastro" method="post">
        <label for="nome">Nome:</label>
        <input type="text" name="nome" id="nome"/><br/>
        <label for="email">E-mail</label>
        <input type="text" name="email" id="email"/><br/>
        <button type="submit" class="btn-primary">Salvar</button>
    </form>
</body>
</html>
```

No servidor, definiremos um método para acessar e manipular os dados enviados pelo corpo do formulário. Isso será feito através do método `formParam` do objeto `Context`. Esse método recebe como parâmetro uma string que representa o nome do elemento html.

```java

public class CadastroControler{

    public void get(Context ctx){
        ctx.render("cadastro.html");
    }

    public void post(Context ctx){
        String nome = ctx.formParam("nome");
        String email = ctx.formParam("email");

        //... Enviar os dados para uma classe responsável pelo cadastro

        ctx.render("resultado.html");
    }
}
```

Na classe App, deve existir uma rota que corresponda a propriedade `action` do formulário com o verbo `post`. 

```java

public class App{

    public static void main(String[] args){

        var app = JavalinUtils.makeApp(8080);
        CadastroControler controler = new CadastroControler();
        //...
        app.get("/cadastro",controler::get);
        app.post("/cadastro",controler::post)
    }

}
```
---

## 5. Exemplo Prático Completo

Vejamos uma aplicação completa que demonstra os verbos e o uso de parâmetros:


```java
//arquivo Usuario.java
package br.edu.tads;

public class Usuario{
    private String nome;
    private String email;

    public Usuario(String nome, String email){
        this.nome = nome;
        this.email = email;
    }

    //...gets, sets e toString
}

//arquivo Gerenciador.java

package br.edu.tads;

import java.util.ArrayList;

public class Gerenciador{

    private ArrayList<Usuario> usuarios;

    public Gerenciador(){
        usuarios = new ArrayList<>();
    }

    public String adicionar(String nome, String email){

        Usuario usuario = new Usuario(nome,email);

        this.usuarios.add(usuario);

        return "Cadastrado";
    }

}


//arquivo CadastroController.java
package br.edu.tads;

import io.javalin.http.Context;

public class CadastroController{

    private Gerenciador gerenciador;

    public CadastroController(Gerenciador gerenciador){
        this.gerenciador = gerenciador;
    }

    public void get(Context ctx){
        ctx.render("cadastro.html");
    }

    public void post(Context ctx){
        String nome = ctx.formParam("nome");
        String email = ctx.formParam("email");

        String resultado = gerenciador.adicionar(nome,email);

        ctx.render("resultado.html");
    }
}


//arquivo App.java

package br.edu.tads;

import io.javalin.Javalin;

public class App{

    public static void main(String[] args){

        var app = JavalinUtils.makeApp(8080);
        Gerenciador gerenciador = new Gerenciador();
        CadastroControler controler = new CadastroControler(gerenciador);
        //...
        app.get("/cadastro",controler::get); 
        app.post("/cadastro",controler::post);
    }

}
```

## 6. Template HTML com Thymeleaf

Conforme apresentado, o Thymeleaf permite a renderização de arquivos `html` através da invocação de métodos pelo Javalin. Isso é muito importante, pois permite a separação entre as camadas de lógica (resumindo a grosso modo o código Java) e a camada de apresentação (os arquivos `html`).

Até o momento, os arquivos são renderizados de forma estática, ou seja, após criados, não conseguimos modificar o seu conteúdo. Para aplicações reais, essa limitação é inviável: precisamos exibir relatórios, listas de alunos cadastrados e mensagens personalizadas geradas em tempo de execução. Assim, é necessário algum meio de inserir conteúdo dinâmico nos arquivos `html`. E é essa a principal funcionalidade do Thymeleaf.

Através de propriedades especiais, é possível controlar o conteúdo que será renderizado no código `html`. Ou seja, dados dinâmicos, gerados durante a execução da aplicação são incluídos na camada de apresentação, mantendo ainda a separação.

Uma das maiores vantagens do Thymeleaf é o conceito de Natural Templates. Os arquivos .html utilizam atributos próprios (com o prefixo th:) inseridos diretamente nas tags HTML padrão.

Isso significa que:

1. O arquivo .html pode ser aberto diretamente no navegador durante o desenvolvimento de telas, sem a necessidade do servidor Java estar rodando.

2. O texto padrão contido dentro da tag serve como um placeholder (rascunho) visual. Quando o servidor processa a página, o Thymeleaf substitui o texto estático pelo valor real da variável Java.

Um template, ou modelo, é um arquivo `html` que contém toda a estrutura de apresentação de uma página da aplicação, incluíndo também os pontos onde serão inseridos os dados dinâmicos. 

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Template</title>
</head>
<body>
    <h1 th:text="${conteudo}">Este texto será substituído</h1>
</body>
</html>
```

No exemplo anterior, note que a tag `h1`, possui uma propriedade adicional chamada `th:text`, cujo valor é `"${conteudo}`, que representa uma variável. No momento da renderização, se uma variável com o nome conteúdo existir, o seu valor substituirá o texto a ser mostrado.

O Thymeleaf possui um conjunto extenso de atributos, que permitem o controle de diversas partes de um template. Abaixo são apresentados os principais. Para mais detalhes do funcionamento de cada atributo, sugere-se a consulta à [documentação](https://www.thymeleaf.org/doc/tutorials/3.1/usingthymeleaf.html).

###  Principais Atributos do Thymeleaf Utilizados

| Atributo | Função no Template | Exemplo de Uso |
| :--- | :--- | :--- |
| `th:text="${var}"` | Substitui o conteúdo da tag pelo valor da variável Java | `<h1 th:text="${aluno.nome}">João</h1>` |
| `th:each="item : ${lista}"` | Itera sobre uma coleção de objetos Java (`List`, `Set`) | `<tr th:each="aluno : ${listaAlunos}">` |
| `th:if="${condicao}"` | Renderiza a tag HTML apenas se a condição for verdadeira | `<p th:if="${aluno.aprovado}">Aprovado</p>` |
| `th:unless="${condicao}"` | Renderiza a tag HTML apenas se a condição for falsa | `<p th:unless="${aluno.aprovado}">Reprovado</p>` |
| `th:action="@{/rota}"` | Define a URL de destino de um formulário com mapeamento seguro | `<form th:action="@{/alunos/salvar}" method="post">` |
| `th:value="${var}"` | Preenche o atributo `value` de um campo de formulário (`<input>`) | `<input type="text" name="nome" th:value="${aluno.nome}">` |


### Tipos de Expressões (`${...}`, `@{...}`, etc.)

* **Variáveis Globais do Modelo:** `${variável}`
  ```html
  <span th:text="${aluno.nome}"></span>
  ```
* **Seleção em Objetos (Acesso a Membros):** `*{propriedade}` (usado em conjunto com `th:object`)
  ```html
  <div th:object="${aluno}">
      <p th:text="*{nome}"></p>
      <p th:text="*{email}"></p>
  </div>
  ```
* **URLs e Links do Sistema:** `@{/caminho}`
  ```html
  <!-- Rota Estática -->
  <a th:href="@{/alunos/novo}">Novo</a>
  <!-- Rota Dinâmica com Parâmetro de Path -->
  <a th:href="@{/alunos/{id}(id=${aluno.id})}">Editar</a>
  <!-- Rota com Query Parameter (/alunos?status=ativo) -->
  <a th:href="@{/alunos(status='ativo')}">Filtrar</a>
  ```
* **Textos/Literal Substitutivos:** `|Texto ${variável}|`
  ```html
  <span th:text="|Bem-vindo, ${usuario.nome}!|"></span>
  ```

---

### Estruturas de Controle e Lógica

#### Condicionais (`th:if`, `th:unless`)
```html
<!-- Exibe se a condição for VERDADEIRA -->
<div th:if="${aluno.ativo}" class="badge-success">Ativo</div>

<!-- Exibe se a condição for FALSA (unless = a menos que) -->
<div th:unless="${aluno.ativo}" class="badge-danger">Inativo</div>

<!-- Operador Ternário Inline -->
<span th:class="${aluno.ativo} ? 'texto-verde' : 'texto-vermelho'">Status</span>
```

#### Iteração / Laços de Repetição (`th:each`)
```html
<tr th:each="aluno, iter : ${alunos}">
    <td th:text="${iter.index}">0</td>            <!-- Índice base 0 -->
    <td th:text="${iter.count}">1</td>            <!-- Contador base 1 -->
    <td th:text="${aluno.nome}">Nome</td>
    <td th:text="${iter.even} ? 'Par' : 'Ímpar'">Paridade</td>
</tr>
```
> **Objeto de Estado da Iteração (`iter`):** `.index`, `.count`, `.size`, `.current`, `.even`, `.odd`, `.first`, `.last`.

---

#### Mapeamento de Formulários HTML

```html
<form action="/alunos" method="POST" class="form-grid">
    <div class="form-group">
        <label for="nome">Nome:</label>
        <!-- O atributo 'name' DEVE bater com a leitura no Javalin -->
        <input type="text" id="nome" name="nome" th:value="${aluno?.nome}" required>
    </div>

    <div class="form-group">
        <label for="status">Situação:</label>
        <select id="status" name="status">
            <option value="ATIVO" th:selected="${aluno?.status == 'ATIVO'}">Ativo</option>
            <option value="INATIVO" th:selected="${aluno?.status == 'INATIVO'}">Inativo</option>
        </select>
    </div>

    <button type="submit" class="btn-primary">Salvar</button>
</form>
```


## 7. Injeção de dados do Java no HTML

Após a definição do template, a inserção dos dados dinâmicos é feita através no Javalin no momento da renderização do arquivo. Para tanto, deve-se passar um dicionário de dados, contendo as variáveis utilizadas no template e seus respectivos valores.

Um dicionário é uma estrutura de dados, que permite mapear chaves e valores. Geralmente as chaves são definidas por Strings e os valores podem ser qualquer tipo de dado, tanto primitivos quanto objetos. Em Java, um dicionário é representado por objetos do tipo [`Map`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html). 

```java

    //instancia um dicionário
    Map<String,Object> dados = new HashMap<>();

    Usuario usuario = new Usuario();

    ArrayList<Usuario> usuarios = new ArrayList<>();

    //insere uma chave com seu valor
    dados.put("conteudo","Um texto para ser mostrado");
    dados.put("usuario",usuario);
    dados.put("lista",usuarios);
```

No momento da renderização, deve ser informado o dicionário com os dados. 

```java

    //...
     Map<String,Object> dados = new HashMap<>();
    //...

    ctx.render("template.html", dados);
```

É importante ressaltar que as chaves do dicionário devem coincidir com variáveis utilizadas no corpo do template. 

## 8. Layouts e Reutilização de Fragmentos (`th:fragment`)

Em sistemas web, é boa prática evitar a duplicação de código HTML comum (como menus de navegação, cabeçalhos, rodapés e importações de CSS/JS). No Thymeleaf, você resolve isso usando Layouts Base e Fragmentos.

Essa abordagem funciona de forma semelhante ao padrão Template Method da Orientação a Objetos: definimos uma estrutura ancestral fixa (esqueleto da página) e permitimos que as páginas filhas "sobrescrevam" apenas o bloco de conteúdo principal.

### O Layout Base (base.html)

O arquivo base.html define o esqueleto da aplicação.

No exemplo abaixo, destacam-se duas configurações essenciais:

1. th:fragment="layout (conteudo)" na tag <html>: Declara que o documento inteiro funciona como um componente reutilizável chamado layout, o qual aceita um parâmetro nomeado conteudo (um bloco HTML dinâmico).

2. th:replace="${conteudo}" na tag <main>: Indica o ponto exato onde o elemento <main> padrão será substituído pelo conteúdo injetado pela página específica.


```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="pt-BR" th:fragment="layout (conteudo)">
<head>
    <meta charset="UTF-8">
    <title>Sistema Escolar</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>

    <!-- Cabeçalho Fixo do Sistema -->
    <header class="app-header">
        <h1>Painel Administrativo</h1>
        <nav class="app-nav">
            <ul>
                <li><a href="/">Início</a></li>
                <li><a href="/alunos">Alunos</a></li>
            </ul>
        </nav>
    </header>

    <!-- Ponto de Injeção: Este container será substituído pelo conteúdo da página específica -->
    <main class="container" th:replace="${conteudo}">
        <p>Conteúdo padrão (exibido apenas se nada for injetado).</p>
    </main>

    <!-- Rodapé Fixo do Sistema -->
    <footer class="app-footer">
        <p>&copy; 2026 - Sistema Escolar Javalin</p>
    </footer>

</body>
</html>
```

## A Página Específica (index.html)

A página index.html define apenas o conteúdo que é exclusivo dela. Ela "decora" seu conteúdo reutilizando o esqueleto do base.html.

Sintaxe explicada:

 th:replace="~{base :: layout (~{::main})}": 
- base :: layout: Chama o fragmento layout declarado no arquivo base.html.

~{::main}: Captura a tag <main> desta página local e a passa como o argumento conteudo para o layout base.

```html

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" 
      lang="pt-BR" 
      th:replace="~{base :: layout (~{::main})}">
<head>
    <title>Página Inicial</title>
</head>
<body>

    <!-- Apenas a tag <main> precisa ser definida nesta página -->
    <main>
        <section class="card">
            <h2>Bem-vindo ao Sistema!</h2>
            <p>Este conteúdo específico foi injetado automaticamente dentro do layout base.</p>
        </section>
    </main>

</body>
</html>
```