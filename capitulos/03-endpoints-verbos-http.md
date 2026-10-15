# Capítulo 03: Endpoints e Verbos HTTP na Prática

## 1. O Que É uma Rota e um Endpoint?

No mundo do desenvolvimento web e das APIs (Interfaces de Programação de Aplicações), chamamos de **Rota** ou **Endpoint** o endereço específico que identifica um recurso que o servidor disponibiliza.

Pense no endereço de um site como o índice de um livro:

```
http://localhost:8080/usuarios
\___________________/ \_______/
       Origem           Caminho (Endpoint / Recurso)
```

Quando o cliente faz uma requisição para `/usuarios`, o servidor sabe que deve acionar o bloco de código responsável por usuarios, e não por funcionários ou produtos.

---

## 2. A Semântica dos Verbos HTTP

No protocolo HTTP, não basta informar **qual** recurso você deseja acessar; você também deve informar **o que** deseja fazer com ele. Essa intenção é expressa através dos **Verbos HTTP** (também chamados de Métodos HTTP).

Os quatro verbos fundamentais utilizados na criação de APIs são:

| Verbo HTTP | Ação Esperada | Equivalente no Banco de Dados (CRUD) | Exemplo de Intenção |
| :--- | :--- | :--- | :--- |
| **GET** | Consultar / Obter dados | **R**ead (Ler / Selecionar) | "Traga a lista de alunos" |
| **POST** | Criar um novo recurso | **C**reate (Inserir) | "Cadastre este novo aluno" |
| **PUT** | Atualizar um recurso existente | **U**pdate (Atualizar) | "Altere os dados do aluno 10" |
| **DELETE** | Remover um recurso | **D**elete (Excluir) | "Exclua o aluno 10 do sistema" |

### Como o Javalin Organiza os Verbos?

No Javalin, o mapeamento é intuitivo e direto. O próprio nome do método no objeto `app` corresponde ao verbo HTTP:

```
Requisição Chegando
        |
        v
+------------------+
| Verbo + Caminho  |
+--------+---------+
         |
         |---> GET    /usuarios      ===> app.get("/usuarios", ctx -> ...)
         |---> POST   /usuarios      ===> app.post("/usuarios", ctx -> ...)
         |---> PUT    /usuarios/{id} ===> app.put("/usuarios/{id}", ctx -> ...)
         +---> DELETE /usuarios/{id} ===> app.delete("/usuarios/{id}", ctx -> ...)
```

Perceba que você pode ter o mesmo caminho (`/usuarios`) associado a comportamentos totalmente diferentes dependendo do verbo utilizado.

---

## 3. Rotas que Invocam Código

De uma maneira simplificada ao definirmos um endpoint de uma aplicação, associamos três informações: 

1. O verbo HTTP que será utilizado;
2. O caminho/rota que será informado no navegador;
3. A funcionalidade que será executada no servidor e gera uma resposta para aquela requisição.

Analisando o Javalin, pode-se notar que os métodos que representam os verbos, esperam dois parâmetros:

1. Uma string que representa a rota;
2. Um método que recebe como parâmetro um objeto do tipo [Context](https://javadoc.io/doc/io.javalin/javalin/6.7.0/io/javalin/http/Context.html).

Existem algumas possibilidades de como definir o método que será executado quando uma rota for acessada. Pode-se utilizar um método anônimo: `ctx -> {...}`. Neste caso, toda a funcionalidade do método fica escrita juntamente com a definição da rota. Para pequenas operações, isso não é um problema. 

Quando os métodos são mais complexos, o código tende a ficar mais difícil de ler e compreender. Além do mais, é importante lembrarmos que estamos trabalhando com orientação à objetos, onde um dos pilares é o **Encapsulamento**. Assim, o recomendado é criar uma classe que defina os métodos que serão executados. Chamaremos essa classe de **Controler** e ela será responsável pela interface entre as requisições web (provenientes do navegador) e as classes internas do sistema. É recomendável que sejam definidos diferentes controladores, para cada uma das funcionalidades providas pelo sistema.

```java

public class Controler{

    public void get(Context ctx){
        ctx.result("O método get do controler foi executado!");
    }

}

```

Para _ligar_ uma rota com um método da classe controladora, utilizaremos a seguinte sintaxe `objeto::metodo`. Essa é uma forma de referenciar um método de um determinado objeto.

```java

public class App{

    public static void main(String[] args){

        var app = JavalinUtils.makeApp(8080);
        Controler controler = new Controler();
        //...
        app.get("/boas-vindas",controler::get);
    }

}
```

