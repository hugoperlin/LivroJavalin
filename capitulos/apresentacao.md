



# Programação Orientada a Objetos com Javalin
## Da Lógica em Console à Arquitetura Web Moderna

Bem-vindo ao livro/apostila de **Programação Orientada a Objetos (POO)** aplicada com o microframework **Javalin**. 

Este material foi planejado e escrito sob medida para estudantes dos cursos superiores de tecnologia, em especial **Tecnologia em Análise e Desenvolvimento de Sistemas (TADS)**, que estão concluindo a etapa de Lógica de Programação e iniciando a fascinante jornada da modelagem orientada a objetos.

---

### 🎯 A Filosofia Pedagógica: Por Que Aposentar o `Scanner`?

Tradicionalmente, os primeiros cursos de POO em Java utilizam aplicações de linha de comando (*CLI*). Os estudantes passam semanas escrevendo laços com `Scanner` para capturar dados do teclado e `System.out.println` para imprimir menus no terminal do sistema operacional:

```
=== MENU PRINCIPAL ===
1 - Cadastrar Aluno
2 - Listar Alunos
Opção: _
```

Embora esse modelo tenha valor histórico, ele apresenta três problemas graves para o aluno contemporâneo:
1. **Frustração visual e desmotivação:** O console parece desconectado do mercado e das aplicações reais (web, mobile e APIs).
2. **Poluição do código:** O código de controle de fluxo de menus de terminal (loops infinitos, limpeza de buffer com `scanner.nextLine()`) consome mais atenção do estudante do que o próprio conceito de **Classe, Atributo, Método e Encapsulamento**.
3. **Abstração difícil de visualizar:** É difícil entender a utilidade de "encapsular dados em um Objeto" quando o objeto só existe para ser fatiado em dezenas de `println` logo em seguida.

#### A Proposta Deste Material
Nossa proposta inverte essa dinâmica: **utilizaremos a Web como interface utilitária.**

```
   TRADICIONAL (Console)                  MODERNO (Javalin + Web)
+-------------------------+             +-------------------------+
|      Teclado / CLI      |             |   Navegador             |
|   (Scanner / println)   |             |   (HTTP Requests / HTML)|
+------------+------------+             +------------+------------+
             |                                       |
             v                                       v
+-------------------------+             +-------------------------+
|      Código em Java     |             |    Servidor Javalin     |
|   (Objetos em Memória)  |             |  (Rotas, Objetos & BD)  |
+-------------------------+             +-------------------------+
```

Em vez de ler strings pelo teclado, sua aplicação receberá respostas em html. O **Javalin** entra aqui não como um fim em si mesmo, mas como a ponte mais enxuta e elegante possível entre a rede mundial de computadores e as suas classes Java.

---

### 🧠 Pré-requisitos
Para aproveitar ao máximo este livro, você precisa apenas de:
- Conhecimento prévio de **Lógica de Programação** (variáveis, tipos primitivos, estruturas condicionais `if/else`, estruturas de repetição `while/for`).
- JDK (Java Development Kit) versão **17** ou superior instalado em sua máquina.
- Um editor de código ou IDE moderna: **VS Code**, **IntelliJ IDEA Community** ou **Eclipse**.

---

