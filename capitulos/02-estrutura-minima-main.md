# Capítulo 02: A Estrutura Mínima — O Método Main e a Porta 8080

## 1. Configurando o Projeto com Maven

Para que o Java reconheça o Javalin e suas ferramentas auxiliares, precisamos declarar essas dependências em um gerenciador de projetos. No ecossistema Java, a ferramenta padrão da indústria é o **Apache Maven**, configurado por meio do arquivo `pom.xml`.

Crie um projeto Maven simples e certifique-se de que o seu arquivo `pom.xml` contenha a seguinte estrutura:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>br.edu.ifpr.pgua.tads</groupId>
    <artifactId>oo-javalin</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- Servidor Web Javalin -->
        <dependency>
            <groupId>io.javalin</groupId>
            <artifactId>javalin</artifactId>
            <version>7.2.3</version>
        </dependency>

        <!-- Biblioteca de Logs (evita avisos no terminal) -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.12</version>
        </dependency>

        <!-- NOVAS DEPENDÊNCIAS VÃO AQUI! -->

    </dependencies>
</project>
```

As dependências de um projeto podem variar muito. Sempre que for necessária incluir novas dependências no projeto, o arquivo `pom.xml` deve ser alterado. Se necessitar, inclua o bloco de definição da dependência nesse espaço. 

---

## 2. O Conceito de Porta de Rede e Localhost

Para deixar as coisas claras, daqui para frente, nossos projetos serão o desenvolvimento de aplicações web que rodarão em um servidor utilizando código Java.

Antes de colocarmos o servidor no ar, precisamos compreender dois conceitos fundamentais de redes de computadores:

### O Que É `localhost` (127.0.0.1)?
Quando seu programa roda no terminal, ele interage diretamente com o sistema operacional local. Quando criamos uma aplicação Web, o servidor precisa ser alcançável por um endereço de rede.

O termo **`localhost`** (cujo endereço IP numérico padrão é `127.0.0.1`) é um endereço especial que significa: *"este próprio computador em que estou trabalhando"*. É o espelho da sua máquina.

### O Que É uma Porta de Rede (Ex: 8080)?

Imagine um grande condomínio de apartamentos:

```
        Rua da Internet, nº 127.0.0.1 (IP / Endereço do Prédio)
       +-------------------------------------------------------+
       | [Apt 80]     [Apt 443]    [Apt 3306]    [Apt 8080]    |
       | Web Padrão   Web Seguro     MySQL        Nosso Javalin|
       +-------------------------------------------------------+
```

* O **Endereço IP** (`127.0.0.1`) identifica o **prédio** (o seu computador físico).
* A **Porta** (`8080`) identifica o **apartamento específico** onde o seu programa Java está escutando.

Se um computador possui dezenas de programas conectados à internet simultaneamente (navegador, Spotify, Discord, banco de dados), o sistema operacional sabe para qual programa entregar cada pacote de dados através do número da porta.

Por convenção, a porta `8080` é amplamente utilizada no desenvolvimento para testes de servidores web alternativos à porta padrão `80`.

---

## 3. O Código do Primeiro Servidor: `App.java`

Crie o arquivo `src/main/java/br/edu/ifpr/pgua/tads/App.java` com o código abaixo:

```java
package br.edu.ifpr.pgua.tads;

import io.javalin.Javalin;

public class App {

    public static void main(String[] args) {
        
        var app = Javalin.create(config -> {
        config.routes.get("/", ctx -> ctx.result("Hello World"));
        }).start(7070);
    
    }
}
```

---

## 4. O Ciclo de Vida: Por Que o Programa Não Fecha?

Nos seus programas anteriores em console, quando a última linha do método `main` era executada, a JVM (Java Virtual Machine) finalizava imediatamente a execução e você voltava para o prompt do terminal:

```
[Console Antigo]
main() inicia -> lê variáveis -> faz contas -> imprime -> main() termina -> PROGRAMA FECHOU
```

No Javalin, algo totalmente diferente acontece:

```
[Servidor Web com Javalin]
main() inicia 
    |
    v
Javalin.create().start(8080)
    |
    v
[Abre uma Thread em segundo plano escutando a porta 8080]
    |
    +---> O programa PERMANECE VIVO em loop de escuta!
          Aguardando conexões de navegadores e clientes.
```

O método `start(8080)` cria um processo que mantém a JVM em execução contínua. Enquanto você não interromper o processo (por exemplo, pressionando `Ctrl + C` no terminal ou clicando no botão "Stop" da sua IDE), o seu servidor continuará vivo, pronto para responder requisições a qualquer momento.

---

## 5. Testando no Navegador

1. Execute a classe `App` na sua IDE (ou com o comando `mvn compile exec:java -Dexec.mainClass="br.edu.tads.App"`).
2. Observe as mensagens informativas no terminal (logs do Javalin e Jetty informando que a porta 8080 está aberta).
3. Abra seu navegador de internet favorito (Chrome, Firefox, Edge).
4. Na barra de endereços, digite:
   ```
   http://localhost:8080
   ```
5. Você verá na tela:
   ```
   Parabéns! Seu servidor Javalin está funcionando com sucesso!
   ```
6. Agora, altere a URL no navegador para:
   ```
   http://localhost:8080/boas-vindas
   ```
7. A resposta mudará instantaneamente!

Você acaba de criar e testar seu primeiro servidor web utilizando apenas Java puro e um microframework enxuto. No próximo capítulo, vamos entender detalhadamente o que são **rotas**, **endpoints** e os **verbos HTTP**.
