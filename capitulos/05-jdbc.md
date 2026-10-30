# Capítulo 05: Persistência com JDBC e SQLite

## 1. Por Que SQLite e JDBC?

Até o momento, todos os dados manipulados pelo sistema existiam unicamente na memória volátil do computador. Se a energia acabasse ou o servidor fosse reiniciado, tudo desaparecia.

Para criar um sistema corporativo real, precisamos de **Persistência de Dados**: gravar as informações em um dispositivo de armazenamento secundário (disco rígido / SSD). Para isso, utilizaremos um SGDB, que permite gravar e ler dados de forma eficiente.

### Por Que Escolhemos o SQLite?
Bancos de dados tradicionais (como MySQL, PostgreSQL e Oracle) exigem a instalação de um servidor de banco de dados à parte, configuração de portas de rede, criação de usuários, permissões e senhas.

O **SQLite** é diferente: ele é uma biblioteca C embutida cujo banco de dados é **um único arquivo comum no disco** (ex: `aplicacao.db`). É leve, transacional, padrão de mercado em dispositivos móveis (Android e iOS) e ideal para estudantes aprenderem SQL e JDBC sem complicações de infraestrutura.


### O Que É JDBC?

**JDBC** (*Java Database Connectivity*) é a API padrão do Java para comunicação com bancos de dados relacionais através de comandos SQL. Ela é uma especificação que define como será realizada a troca de mensagens entre o sistema Java e o SGDB. Importante ressaltar que cada desenvolvedor do SGDB desenvolve a sua própria implementação do JDBC, que é inserida como uma dependência do projeto.


### Configurando a Dependência do SQLite no `pom.xml`

Para que seja possível que um sistema em Java se comunique com um banco de dados SQLite (ou qualquer outro SGBD), é necessário que exista um driver de comunicação. O driver é responsável por implementar todo o processo de troca de mensagens entre o Java e o SGDB. Para ter acesso a um driver, você deve inserir a dependência no arquivo `pom.xml`.

Para o SQLite, esta deve ser a dependência inserida no seu `pom.xml`:

```xml
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.45.2.0</version>
</dependency>
```
---

Caso você pretenda conectar o seu sistema em um banco de dados MySQL/MariaDB, você pode utilizar esta dependência:

```xml

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.33</version>
</dependency> 

```


## 2. Principais Classes do JDBC

O JDBC utiliza quatro interfaces principais do pacote `java.sql` para conectar o código Java a um banco de dados relacional.

┌──────────────┐       ┌────────────┐             ┌───────────────────┐             ┌───────────┐
│ DriverManager            │ ─►│ Connection            │ ───► │ PreparedStatement                 │ ───► │            ResultSet │
└──────────────┘       └────────────┘              └───────────────────┘             └───────────┘
 Carrega o driver     Abre a sessão       Prepara o comando SQL       Lê os dados


---

### `DriverManager`

A classe `DriverManager` gerencia a lista de drivers de banco de dados disponíveis na aplicação.

* **Função principal:** Estabelece a conexão física entre o código Java e o servidor de banco de dados.
* **Ação principal:** Executa o método `getConnection(url, usuario, senha)`.
* **Regra de uso:** Use o `DriverManager` para obter um objeto do tipo `Connection`.

---

### `Connection`

A interface `Connection` representa uma sessão aberta com o banco de dados.

* **Função principal:** Mantém o canal de comunicação ativo durante o envio de instruções.
* **Ações principais:**
  * Cria objetos para executar instruções SQL (`prepareStatement()`).
  * Controla transações no banco de dados (`commit()` e `rollback()`).
  * Fecha a sessão ativa com o banco de dados (`close()`).
* **Regra de uso:** Feche sempre a `Connection` após o término da operação para liberar memória.


Aqui está um exemplo de como criar uma conexão com o banco de dados SQLite.

```java

public class ConexaoSQLite{

    public static void main(String[] args){

        try(Connection con=DriverManager.getConnection("jdbc:sqlite:app.db")){ 
            System.out.println("Conexão realizada!"); 
        }catch(SQLException e){
            System.out.println("Erro no BD!->"+e.getMessage());
        }
    }
}

```
Algumas observações a respeito do código de exemplo anterior.

As operações de comunicação com SGDB através do JDBC devem necessariamente prever possíveis erros e como proceder caso eles ocorram. Para isso, é necessário o uso do que chama-se de tratamento de exceções. Isso é feito empregando os blocos `try...catch`. A ideia central é que operações que possam gerar erros sejam inseridos dentro do bloco `try`. Caso o erro aconteça, as operações dentro do bloco `catch` são executadas como forma de recuperação do  erro. 

As operações do JDBC podem gerar exceções do tipo `SQLException`. Assim é obrigatório o uso do bloco `try` associado com um bloco `catch(SQLException e)`. Você pode pensar o bloco `catch` como uma função/método que será invocado quando uma exceção do tipo definido como parâmetro ocorrer. 

Note que na sintaxa do bloco `try` temos a definição de uma instrução dentro dos parênteses, que é instanciação de um objeto do tipo `Connection`. Este tipo de bloco é chamado `try-with-resources`. A grande vantagem de uso deste tipo de bloco é que ao final da execução a conexão é automaticamente fechada, mesmo em caso de erro.

Outro ponto importante de ser comentado é que o método `getConnection` recebe como parâmetro uma string que especifica a url de conexção com o banco de dados. No caso `jdbc:sqlite:app.db`, significa que esta conexão irá utilizar o padrão JDBC, para conectar a um banco sqlite que estará armazenado no arquivo `app.db`. 

Os parâmetro de conexão com um banco de dados MySQL são um pouco diferentes. A url geralmente tem o formato `jdbc:mysql://IP_DO_SERVIDOR/NOME_DO_BANCO`. Também devem ser passados como parâmetros o usuário e senha de acesso ao banco.

---

### `PreparedStatement`

A interface `PreparedStatement` representa uma instrução SQL pré-compilada.

* **Função principal:** Envia comandos SQL ao banco de dados com parâmetros dinâmicos.
* **Ações principais:**
  * Substitui os caracteres de interrogação (`?`) por valores reais (`setString()`, `setInt()`, `setDouble()`).
  * Executa consultas de leitura (`executeQuery()`).
  * Executa comandos de escrita (`executeUpdate()`).
* **Regra de uso:** Use sempre o `PreparedStatement` em vez de `Statement` simples para evitar ataques de injeção de SQL (*SQL Injection*).

Após o estabelecimento de uma conexão é possível o envio de comandos ao SGBD utilizando a linguagem SQL. Para isso, utiliza-se um objeto do tipo `PreparedStatement`. As instruções preparadas, podem conter parâmetros dentro do comando SQL. Isso aumenta a segurança, evitando possíveis ataques de injeção de SQL.

Para que o comando de fato seja executado no SGDB, deve-se invocar o comando `executeUpdate()` para operações de escrita (INSERT, UPDATE, DELETE). O método retorna o número de linhas afetadas pela operação SQL.


```java

public class ConexaoSQLite{

    public static void main(String[] args){

        try(Connection con=DriverManager.getConnection("jdbc:sqlite:app.db")){ 
            
            String sql = "INSERT INTO USUARIO(nome,cpf,telefone,email) VALUES (?,?,?,?)";

            PreparedStatement pstm = con.prepareStatement(sql);
            
            pstm.setString(1,"Zé");
            pstm.setString(2,"123.456.789-00");
            pstm.setString(3,"041-9999-0011");
            pstm.setString(4,"ze@teste.com");

            int ret = pstm.executeUpdate();

            if(ret == 1){
                System.out.println("Usuário inserido com sucesso");
            }

        }catch(SQLException e){
            System.out.println("Erro no BD!->"+e.getMessage());
        }
    }
}

```

Note que no comando SQL de INSERT, não são passados os valores e sim `?`. Cada interrogação representa um parâmetro que será informado no momento da execução. Para informar um parâmetro utiliza-se os métodos `setString`, se o parâmetro for texto, `setInt`, se o parâmetro for inteiro, ou `setDouble`, se o parâmetro for decimal. A ordem dos parâmetros é contada da esquerda para a direita iniciando em 1. A posição do parâmetro deve ser informada no método de ajuste.

---

### `ResultSet`

A interface `ResultSet` representa uma tabela de dados retornada por uma consulta SQL do tipo `SELECT`.

* **Função principal:** Guarda as linhas de dados retornadas pelo banco de dados.
* **Ações principais:**
  * Move o cursor para a próxima linha da tabela (`next()`).
  * Lê o valor de uma coluna da linha atual (`getString()`, `getInt()`, `getDouble()`).
* **Regra de uso:** Percorra o `ResultSet` dentro de um laço `while (rs.next())` para processar cada linha de dados.

---

Caso o comando SQL faça a leitura de dados através do comando SELECT, deve-se utilizar o comando `executeQuery` do objeto `PreparedStatement`. Este comando retornára um objeto do tipo `ResultSet`, que representa o resultado da seleção. 

```java

public class ConexaoSQLite{

    public static void main(String[] args){

        try(Connection con=DriverManager.getConnection("jdbc:sqlite:app.db")){ 
            
            String sql = "SELECT * FROM usuarios";

            PreparedStatement pstm = con.prepareStatement(sql);

            ResultSet rs = pstm.executeQuery();

            while(rs.next()){
                int id = rs.getInt("id");
                String nome = rs.getString("nome");
                String cpf = rs.getString("cpf");
                String telefone = rs.getString("telefone");
                String email = rs.getString("email");

                System.out.println("Id:+"id+" Nome:"+nome+" cpf:"+cpf+" telefone:"+telefone+" email:"+email);
            }

        }catch(SQLException e){
            System.out.println("Erro no BD!->"+e.getMessage());
        }
    }
}

```
