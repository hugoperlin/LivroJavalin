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