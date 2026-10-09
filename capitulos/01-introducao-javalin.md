  # Capítulo 01: Adeus Scanner — Introdução ao Javalin

## 1. O Ponto de Partida: O Mundo do Console

Se você chegou até este livor, é muito provável que seus primeiros programas em Java tenham sido parecidos com o código abaixo:

```java
import java.util.Scanner;

public class SistemaConsole {
    public static void main(String[] args) {
        Scanner leitor = new Scanner(System.in);

        System.out.println("=== CADASTRO SIMPLES ===");
        System.out.print("Digite o nome: ");
        String nome = leitor.nextLine();

        System.out.print("Digite a idade: ");
        int idade = leitor.nextInt();

        System.out.println("\nDados recebidos com sucesso!");
        System.out.println("Nome: " + nome + " | Idade: " + idade);

        leitor.close();
    }
}
```

Esse modelo é excelente para aprender **lógica de programação**: você aprendeu o que são variáveis, como tomar decisões com `if/else` e como repetir blocos com `while` e `for`.

A interação com o usuário é realizada através do console de comandos, tanto para a entrada de dados usando o teclado, quanto para a sída de dados através de texto. 

No entanto, quando começamos a estudar **Orientação a Objetos (POO)**, o console possui algumas limitações pedagógicas importantes:
1. **O teclado trava o fluxo de execução:** A aplicação para e fica esperando indefinidamente até o usuário apertar *Enter*.
2. **Entrada e saída limitadas:** As aplicações ficam limitadas ao teclado e ao texto para entrada e saída de dados. Até é possível utilizar algumas bibliotecas para criar interfaces mais intuitivas, porém geralmente fogem ao escopo do estudo.
3. **Isolamento total:** Esse programa só roda na sua máquina e só aceita comandos de quem estiver sentado fisicamente na frente do seu teclado.
4. **Interface incomum:** Apesar de muito poderoso, o console de comando não é utilizado usualmente pela maioria dos usuários de sistemas computacionais, sendo preferível o emprego de interfaces gráficas.

---

## 2. A Mudança de Paradigma: Arquitetura Cliente-Servidor

No desenvolvimento de software profissional, quase nenhum sistema de gestão opera através do terminal do sistema operacional. Em vez disso, adotamos a **Arquitetura Cliente-Servidor**:

```
+-------------------------------------------------------------+
|                     CLIENTE (Frontend)                      |
|      (Navegador Web, Aplicativo Mobile ou Postman)          |
+-------------------------------------------------------------+
                              |
                              | 1. REQUISIÇÃO (HTTP Request)
                              |    "Servidor, me dê a lista de alunos"
                              v
+-------------------------------------------------------------+
|                     SERVIDOR (Backend)                      |
|       (Aplicação Java com Javalin escutando a rede)         |
+-------------------------------------------------------------+
                              |
                              | 2. RESPOSTA (HTTP Response)
                              |    "Aqui estão os alunos: [ { ... } ]"
                              v
+-------------------------------------------------------------+
|                     CLIENTE (Frontend)                      |
|                (Exibe os dados na tela)                     |
+-------------------------------------------------------------+
```

### A Analogia do Restaurante

Imagine um restaurante tradicional:
* **Você (O Usuário):** Senta-se à mesa e olha o cardápio.
* **O Garçom (O Navegador - Frontend):** Você chama o garçom e faz o pedido: *"Por favor, traga-me um prato X"*. O garçom anota seu pedido e o leva até a cozinha.
* **A Cozinha (O Servidor Java - Backend):** O cozinheiro recebe a comanda, pega os ingredientes na despensa (Banco de Dados), monta o prato (executa as regras da Orientação a Objetos) e entrega a bandeja pronta ao garçom.
* **A Entrega (O Navegador - Frontend):** O garçom retorna à sua mesa com a comida pronta.

No nosso curso:
- O **Garçom** será um navegador web - Google Chrome, Firefox, Opera, etc.
- A **Cozinha** serão as nossas classes Java rodando no **Javalin**.
- O **Garçom** e a **Cozinha** conhecem uma forma específica de comunicação, chamado protocolo **HTTP**.

---

## 3. Por Que NÃO Começamos com Spring Boot?

O **Spring Boot** é atualmente o framework Java para Web mais utilizado no mercado corporativo. Sendo assim, por que não o utilizamos desde a primeira aula?

A resposta é puramente **pedagógica**: *o Spring Boot possui "mágica" demais para quem está aprendendo os fundamentos.*

Observe como é uma classe simples no Spring Boot:

```java
// CÓDIGO APENAS ILUSTRATIVO - NÃO FAREMOS ASSIM
@RestController
@RequestMapping("/alunos")
public class AlunoController {

    @Autowired
    private AlunoService service;

    @GetMapping
    public List<Aluno> listar() {
        return service.buscarTodos();
    }
}
```

Para um aluno iniciante, surgem perguntas legítimas e paralisantes:
- Quem instancia a classe `AlunoController`? Onde está o operador `new`?
- O que significa `@Autowired`? Como a variável `service` ganha valor sem ninguém chamar o construtor?
- O que é uma anotação (`@`) e como ela altera o comportamento do código em tempo de execução?

Quando a ferramenta esconde o funcionamento básico da linguagem, o estudante deixa de aprender Orientação a Objetos e passa a memorizar anotações decoradas.

---

## 4. Por Que o Javalin é Perfeito para o Ensino de POO?

O **Javalin** ([javalin.io](https://javalin.io)) é um **microframework** minimalista para Java e Kotlin construído sobre o servidor web Jetty.

Principais vantagens pedagógicas:
1. **Sem anotações mágicas:** Não há `@RestController`, `@Autowired` ou proxies ocultos. O código é Java padrão e transparente.
2. **Método `main` explícito:** O servidor começa dentro do bom e velho `public static void main(String[] args)`. Você sabe exatamente onde o programa começa e onde termina.
3. **Instanciação transparente:** Você cria seus próprios objetos usando `new`, passa referências via construtor e manipula seus métodos diretamente.
4. **Leve e veloz:** Inicializa em poucos milissegundos e consome quase nenhuma memória.
5. **POO em Primeiro Plano:** O Javalin cuida apenas de escutar a rede e entregar os dados para você. Todo o restante é modelagem orientada a objetos pura (Classes, Objetos, Encapsulamento, Coleções, Polimorfismo).

Observe como um servidor completo é criado no Javalin:

```java
import io.javalin.Javalin;

public class App {
    public static void main(String[] args) {
        // Inicializa o servidor web na porta 8080
        Javalin app = Javalin.create().start(8080);

        // Define uma rota acessível via navegador
        app.get("/", ctx -> ctx.result("Olá, Mundo Web com Javalin!"));
    }
}
```

Note a clareza: você chamou um método estático `create()`, encadeou o método `start(8080)` e registrou uma rota `get`. Não há arquivos XML de configuração, nem reflexão oculta. Tudo o que você aprendeu sobre chamadas de métodos e objetos se aplica diretamente.

---

## 5. Resumo do Capítulo

- O console (`Scanner` e `System.out.println`) é útil para lógica básica, mas é síncrono, limitado e não reflete sistemas modernos.
- A Web adota a arquitetura **Cliente-Servidor**, onde requisições solicitam ações e respostas retornam dados estruturados.
- O Spring Boot esconde a Orientação a Objetos sob dezenas de anotações complexas e injeções mágicas.
- O **Javalin** é a escolha ideal para o aprendizado porque mantém o controle total nas mãos do desenvolvedor, tornando os conceitos de Classes, Objetos e Métodos palpáveis e visíveis.
