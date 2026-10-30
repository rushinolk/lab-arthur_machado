# Laboratório 01: Decomposição de Monolitos e Cloud-Native

**Disciplina:** Engenharia de Backend
**Professor:** Ronaldo Amaral
**Aluno:** Arthur Machado Gomes
**Matrícula:** 202021200210

> **Diretriz de Execução:** Este laboratório deve ser desenvolvido e submetido **individualmente**. Realize seus commits com as respostas diretamente neste arquivo no repositório da disciplina.

---

## Cenário Base: MonolitoShop
O e-commerce **MonolitoShop** roda em um único servidor e compartilha um banco de dados relacional gigante. Com a proximidade de picos de acesso, a equipe de engenharia identificou falhas graves de lentidão e travamentos. Você foi escalado para planejar a modernização arquitetural deste sistema legado.

### Parte 1: Bounded Contexts e Microsserviços (Eixo Y - Decomposição)
Atualmente, a classe `Usuario` no código fonte do monolito possui 45 atributos, misturando dados de login, endereço de entrega, número do cartão de crédito e histórico de navegação no catálogo. O objetivo do Eixo Y é extrair fronteiras de negócio (*Bounded Contexts*) para projetar nossos futuros microsserviços.

**Tarefa 1.1 - Mapeamento:** Separe o sistema em 3 Bounded Contexts principais (ex: Identidade, Pagamentos, Logística) que atuarão como microsserviços independentes. Distribua quais informações da classe "Usuário" legado devem pertencer a cada novo microsserviço.

> **Sua Resposta 1.1:**
> - Microsserviço 1 (Identidade): id, nome, e-mail, CPF, telefone, login, senha (hash), tokens de sessão e permissões.
> - Microsserviço 2 (Pagamentos):dados do cartão de crédito, forma de pagamento preferida, histórico de transações e dados de cobrança.
> - Microsserviço 3 (Logística): endereços de entrega, CEP, complemento, preferências de entrega e rastreio.

**Tarefa 1.2 - Diagrama de Arquitetura:** Crie um diagrama demonstrando a quebra do monolito nesses 3 novos microsserviços. O diagrama deve ilustrar as fronteiras dos contextos e seus respectivos bancos de dados isolados. (Utilize a sintaxe Mermaid diretamente neste Markdown ou insira o link de uma imagem gerada em ferramentas como draw.io/Lucidchart).

> **Exemplo de como estruturar seu diagrama Mermaid (Substitua pelos seus dados e expanda conforme necessário):**
> 
> ```mermaid
> graph TD
>     Client[Cliente / App] --> API[API Gateway]
>     
>     API --> MS1[Microsserviço 1]
>     API --> MS2[Microsserviço 2]
>     API --> MS3[Microsserviço 3]
> 
>     MS1 --> DB1[(Banco de Dados)]
>     MS2 --> DB2[(Banco de Dados)]
>     MS3 --> DB3[(Banco de Dados)]
> ```

> **Sua Resposta 1.2 (Insira seu diagrama abaixo):**
> ```mermaid
> graph TD
>    Client[Cliente / App] --> API[API Gateway]
>
>    subgraph BC1[Bounded Context: Identidade]
>        MS1[Serviço de Identidade]
>        DB1[(Banco Identidade)]
>        MS1 --> DB1
>    end
>
>    subgraph BC2[Bounded Context: Pagamentos]
>        MS2[Serviço de Pagamentos]
>        DB2[(Banco Pagamentos)]
>        MS2 --> DB2
>    end
>
>    subgraph BC3[Bounded Context: Logística]
>        MS3[Serviço de Logística]
>        DB3[(Banco Logística)]
>        MS3 --> DB3
>    end
>
>    API --> MS1
>    API --> MS2
>    API --> MS3
>
>    MS2 -. consulta id do usuário .-> MS1
>    MS3 -. consulta id do usuário .-> MS1
> ```

### Parte 2: O Cubo AKF e a Escalabilidade (Eixo X - Clonagem)
Durante os testes de carga, o servidor web atingiu 100% de CPU. A equipe decidiu colocar um *Load Balancer* e subir 3 instâncias idênticas do servidor. No entanto, ao fazer isso, os clientes começaram a reclamar que os itens desapareciam misteriosamente do carrinho de compras a cada clique.

**Tarefa:** Explique por que os itens somem do carrinho ao clonar o servidor (estado) e qual é a solução Cloud-Native (Stateless) para resolver este problema.

> **Sua Resposta:**
> Os itens somem porque o carrinho fica salvo na memória de um servidor só. Com o Load Balancer cada clique pode cair em um servidor diferente, e esse servidor não tem o carrinho que foi salvo no outro  or isso parece que o carrinho esvaziou.
> A solução é deixar os servidores stateless, ou seja, sem guardar dados do usuário dentro deles. O carrinho passa a ficar em um lugar externo que todos os servidores acessam, como um banco de dados.

### Parte 3: Particionamento de Dados (Eixo Z)
O serviço de Pedidos foi extraído com sucesso do monolito, mas seu banco de dados começou a apresentar lentidão devido ao volume massivo de transações originadas de todo o Brasil.

**Tarefa:** Proponha uma estratégia de *Sharding* (particionamento) para o banco de dados de Pedidos. Qual seria a sua chave de partição (Shard Key) e por quê?

> **Sua Resposta:**
> Eu usaria o estado (UF) do cliente como chave de partição pois os pedidos são enviados para todos os estados do Brasil.
> Como alguns estados podem ter um volume maior de pedidos que outros, uma segunda partição por id_cliente pode ajudar a evitar sobrecarga