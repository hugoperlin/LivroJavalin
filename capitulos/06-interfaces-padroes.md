# Interfaces em Java

Uma **interface** em Java é um contrato de programação. Ela define quais métodos uma classe deve possuir, mas não define como esses métodos funcionam.

---

## 1. O Que É Uma Interface?

Uma interface funciona como uma tomada elétrica padrão na parede. A tomada define o formato dos pinos (o contrato). Qualquer aparelho (geladeira, televisão, carregador) que seguir esse formato funciona na tomada, independentemente de como o aparelho foi fabricado por dentro.

Na Orientação a Objetos, a interface garante que diferentes classes possuam os mesmos métodos, permitindo o uso do **Polimorfismo** e o baixo acoplamento entre as camadas do sistema.

Importante salientar que, assim como uma classe, ao definir uma interface, um novo tipo de dados é criado. Assim, é possível definir variáveis usando esse novo tipo. A característica importante, é que todos os objetos daquele tipo, terão, obrigatóriamente, os métodos definidos na interface.

---

## 2. Regras Principais das Interfaces

Ao criar e usar uma interface em Java, siga estas regras:

1. **Declaração:** Use a palavra-chave `interface` em vez de `class`.
2. **Métodos sem Corpo:** Por padrão, os métodos declarados na interface não possuem implementação (terminam com ponto e vírgula `;`).
3. **Sem Instanciação:** Você não pode criar um objeto diretamente de uma interface usando o operador `new` (exemplo: `new Notificador()` gera erro de compilação).
4. **Implementação Obrigatória:** Uma classe que assina o contrato usa a palavra-chave `implements` e é obrigada a escrever o código de todos os métodos da interface.
5. **Múltiplas Interfaces:** Diferente da herança de classes (onde uma classe só pode estender uma superclasse), uma classe em Java pode implementar várias interfaces ao mesmo tempo.

---

## 3. Exemplo de Código Java

O código a seguir, cria a interface `Notificador`, que define a assinatura de um método chamado `enviarMensagem`, que recebe dois parâmetros do tipo string. Note que é possível inferir que o método deverá enviar uma mensagem contida no texto para o destinatário. Porém, não fica especificado como isso deverá ser realizado.

```java
public interface Notificador {
    // Método abstrato: define apenas a assinatura (nome, parâmetros e retorno)
    public void enviarMensagem(String destinatario, String texto);
}
```

As operações para realizar a funcionalidade dos métodos, são definidas nas classes concretas, que implementam uma ou mais interfaces.

```java
public class NotificadorEmail implements Notificador{

    //...

    @Override
    public void enviarMensagem(String destinatario, String texto){
        //utilizar uma biblioteca para enviar email...
    }
}

public class NotificadorSms implements Notificador{

    //...

    @Override
    public void enviarMensagem(String destinatario, String texto){
        //utilizar uma biblioteca para enviar SMS...
    }
}

public class Main {
    public static void main(String[] args) {
        // Declaramos a variável com o tipo da INTERFACE, mas instanciamos a CLASSE concreta
        Notificador servico = new NotificadorEmail();
        servico.enviarMensagem("aluno@ifpr.edu.br", "Sua nota foi lançada!");

        // Podemos trocar a implementação facilmente sem alterar o restante do código
        servico = new NotificadorSms();
        servico.enviarMensagem("(41) 99999-0000", "Sua nota foi lançada!");
    }
}

```
Note que foram definidas duas classes que implementam a interface `Notificador`, logo é garantido dizer que todos os objetos das classes possuirão uma implementação do método `enviarMensagem`. Assim, é possível alternar entre um objeto e outro sem maiores problemas, característica de um código baseado no polimorfismo. Veja que na implementação do `main`, a variável `servico` em um primeiro momento armazena um objeto do tipo `NotificadorEmail` e logo depois passa a armazenar um objeto do tipo `NotificadorSms`. Isso só é possível porque a `servico` é do tipo `Notificador`.

# Padrões DAO e Repository: Organizando o Acesso aos Dados

Quando desenvolvemos aplicações web com Java e Javalin, é fundamental separar a lógica de negócios (e as rotas HTTP) do código que guarda ou recupera informações. Os padrões **DAO (Data Access Object)** e **Repository** existem exatamente para criar essa fronteira limpa.

Embora ambos isolem a persistência de dados, eles possuem propósitos e níveis de abstração diferentes na arquitetura de software.

---

## 1. O Padrão DAO (Data Access Object)

O **DAO** é um padrão focado na **infraestrutura de persistência**. Ele atua como um tradutor direto entre os objetos Java e as tabelas de um banco de dados relacional.

* **Foco:** Operações CRUD básicas (`INSERT`, `SELECT`, `UPDATE`, `DELETE`) mapeadas para uma tabela específica.
* **Proximidade com o Banco:** O DAO conhece detalhes técnicos como comandos SQL, `PreparedStatement` e `ResultSet` do JDBC.
* **Nomenclatura típica:** `inserir()`, `atualizar()`, `excluir()`, `buscarPorId()`.

```text
[ Camada de Negócio / Controller ]
                │
                ▼
      ┌───────────────────┐
      │     AlunoDao      │ ◄── Interface (Contrato)
      └───────────────────┘
                ▲
                │
      ┌───────────────────┐
      │  AlunoDaoSQLite   │ ◄── Executa SQL direto no banco
      └───────────────────┘
```

### Exemplo Prático de DAO

```java
import java.util.List;

public interface AlunoDao {
    void inserir(Aluno aluno);
    void atualizar(Aluno aluno);
    void excluir(String matricula);
    Aluno buscarPorMatricula(String matricula);
    List<Aluno> listarTodos();
}
```

---

## 2. O Padrão Repository

O **Repository** é um padrão que simula uma **coleção de objetos em memória** (como um `ArrayList` ou `Set`), escondendo completamente a existência de um banco de dados.

* **Foco:** Regras de negócio e agregados. O código que usa um Repository sente como se estivesse manipulando uma lista na memória RAM.
* **Abstração:** Um Repository pode usar um ou mais DAOs por baixo dos panos para montar um objeto complexo.
* **Nomenclatura típica:** `adicionar()`, `remover()`, `obterAprovados()`, `buscarPorTurma()`.

```text
[ Camada de Negócio / Controller ]
                │
                ▼
     ┌─────────────────────┐
     │   AlunoRepository   │ ◄── Simula uma coleção de objetos em memória
     └─────────────────────┘
                │
                ▼ (Pode delegar para um DAO ou lista em memória)
     ┌─────────────────────┐
     │   Banco / Memória   │
     └─────────────────────┘
```

### Exemplo Prático de Repository

```java
import java.util.List;
import java.util.Optional;

public interface AlunoRepository {
    void adicionar(Aluno aluno);
    void remover(Aluno aluno);
    Optional<Aluno> buscarPorMatricula(String matricula);
    List<Aluno> buscarAprovados(); // Consulta orientada à regra de negócio
}
```

---

## 3. Comparação Direta: DAO vs. Repository

| Critério | Padrão DAO | Padrão Repository |
| :--- | :--- | :--- |
| **Visão / Abstração** | Orientado ao **Banco de Dados** (tabelas e linhas). | Orientado ao **Domínio** (coleção de objetos em memória). |
| **Nível de Granularidade** | Geralmente existe um DAO para cada tabela do banco. | Existe um Repository para cada Entidade principal (Agregado). |
| **Métodos Comuns** | `insert()`, `update()`, `delete()`, `selectById()` | `add()`, `remove()`, `findByCriteria()` |
| **Quando usar?** | Sistemas CRUD diretos com JDBC puro e SQL explícito. | Sistemas com regras de negócio ricas ou troca fácil entre memória e banco. |

# Classes Utilitárias

## Arquivos .env

Quando desenvolvemos projetos que utilizam serviços externos, como um banco de dados por exemplo, é comum necessitar de credenciais de acesso (usuário e senha). Esse tipo de dado nunca deve ser escrito diretamente no código-fonte, pois representa um grande problema de segurança. 

Uma maneira segura de armazenar este tipo de dado é utilizando um arquivo de ambiente, também conhecido como `.env`. Geralmente este arquivo é criado na raiz do projeto e definido durante a configuração do sistema. 

Um arquivo de variáveis de ambiente possui o formato NOME_DA_VARIÁVEL=VALOR,sendo que cada variável é definida em uma linha.

```env
DB_URL="jdbc:sqlite:app.db"
DB_NAME="restaurante"
DB_USER="root"
DB_PASSWORD="1234"
```

Para que seja possível carregar um arquivo de variáveis de ambiente, a classe `Env` foi criada. Ela utiliza a biblioteca Dotenv. Desta forma, é necessário incluir a seguinte dependência no arquivo `pom.xml` do seu projeto.

```xml
    <dependency>
      <groupId>io.github.cdimascio</groupId>
      <artifactId>dotenv-java</artifactId>
      <version>2.2.4</version>
    </dependency>
```

```java 

import io.github.cdimascio.dotenv.Dotenv;

public class Env {
    
    private static Dotenv dotenv;
    private Env(){}
    public static String get(String key){
        if(dotenv == null){
            dotenv = Dotenv.load();
        }
        return dotenv.get(key);
    }
 
}
```

Para carregar uma variável, a senha do banco por exemplo, utilize a sintaxe `String senha=Env.get("DB_PASSWORD")` em qualquer lugar do seu código.

## Fábrica de Conexões

A classe abaixo encapsula a complexidade de estabelecimento de uma conexão com um banco de dados. Ao mesmo tempo, cria uma fila de conexões abertas, evitando que muitas conexões estejam abertas ao mesmo tempo, degradando assim a eficiência do banco de dados. Ela depende da classe `Env`.

```java

public class FabricaConexoes {

    private static int MAX_CONNECTIONS = 5;
    private final String URL_DB;
    private final String DB_NAME;
    private final String USER;
    private final String PASSWORD;
    private final String CON_STRING;

    private Connection[] conexoes;

    private static FabricaConexoes instance;

    private FabricaConexoes() {
        conexoes = new Connection[MAX_CONNECTIONS];
        // carregar dados de conexão do .env
        URL_DB = Env.get("DB_URL");
        DB_NAME = Env.get("DB_NAME");

        if(!URL_DB.contains("sqlite")){
            CON_STRING = URL_DB+"/"+DB_NAME;
        }else{
            CON_STRING = URL_DB;
        }

        USER = Env.get("DB_USER");
        PASSWORD = Env.get("DB_PASSWORD");
    }

    public static FabricaConexoes getInstance() {
        if (instance != null) {
            return instance;
        }
        instance = new FabricaConexoes();
        return instance;
    }

    public Connection getConnection() throws SQLException {

        for (int i = 0; i < MAX_CONNECTIONS; i++) {
            if (instance.conexoes[i] == null || instance.conexoes[i].isClosed()) {
                instance.conexoes[i] = DriverManager.getConnection(CON_STRING, USER, PASSWORD);
                return instance.conexoes[i];
            }
        }
        throw new SQLException("Máximo de conexões");
    }
}
```

## Resultado

Um problema que frequentemente encontramos no desenvolvimento de sistemas é como melhorar a comunicação entre classes. Em muitos casos, o retorno de somente um tipo de dados em um método impõe uma grande limitação.

Veja o seguinte exemplo. O método `public ArrayList<Usuario> listar()`, pode deve retornar uma lista de objetos do tipo `Usuario`. Este é o comportamento esperado desse método. Porém, e se no momento da execução do método, ocorrer um erro no banco de dados, o que deve ser retornado? Uma lista vazia? O valor `null`? Uma exceção?

Buscando uma solução para este problema, foi criada a classe `Resultado`, que permite qualificar o retorno de métodos. Para utilizar a classe em seu projeto inclua os trechos de código abaixo em seu arquivo `pom.xml`.

```xml
  <!-- Esse trecho deve ser inserido antes da seção <dependencies> -->
  <repositories>
    <repository>
      <id>jitpack.io</id>
      <url>https://jitpack.io</url>
    </repository>
  </repositories>
```

```xml
    <!-- Esse trecho deve ser inserido na seção <dependencies> -->
    <dependency>
      <groupId>com.github.hugoperlin</groupId>
      <artifactId>results</artifactId>
      <version>1.0.1</version>
    </dependency>

```

Um resultado pode ser do tipo sucesso ou do tipo erro. Note que se o método é executado sem erro, o retorno é criado invocando o método `sucesso` da classe `Resultado`, passando como parâmetro uma mensagem e o objeto de retorno. Caso a execução gere uma exceção, o método `erro` é invocado, passando como parâmetro a mensagem de erro. Desta forma, a classe que utilizar o método listar, poderá saber o tipo de resultado que está recebendo.

```java

public class UsuarioDaoImpl implements UsuarioDao{

    //...

    public Resultado<ArrayList<Usuario>> listar(){
        ArrayList<Usuario> lista = new ArrayList<>();

        try{
            //...
            return Resultado.sucesso("Usuário carregados",lista);
        }catch(SQLException e){
            return Resultado.erro(e.getMessage());
        }

    }
}

```


```java

public class Exemplo{

    public static void main(String[] args){

        UsuarioDao dao = new UsuarioDaoImpl();


        Resultado<ArrayList<Usuario>> resultado = dao.listar();

        if(resultado.isSucesso()){
            ArrayList<Usuario> lista = resultado.comoSucesso().getObj();
            //...
        }else{
            String mensagemErro = resultado.comoErro().getMsg();
        }
    }

}

```



