# Entendimento sobre o CDD

CDD - Cognitive-Driven Development (Desenvolvimento Orientado a Cognição) é uma metodologia para desenvolver código-fonte em que o desenvolvedor está preocupado em ter um controle de carga cognitiva em seu código.

Realizando um cálculo de complexidade intríseca do código-fonte, é determinado um limite de complexidade para cada classe. Se uma classe ultrapassar o limite estabelecido, é um alerta para o desenvolvedor distribuir a complexidade de seu código, dividindo em blocos de códigos mais compreensíveis.

O princípio é que com menor carga coginitia, mais fácil é para outros desenvolvedores compreenderem o código e para realização de manutenção e correção de falhas.

Para o cálculo de complexidade contamos:

- 1 ponto para cada ramificação (if/else/case/switch);
- 1 ponto para tratamento deexeções (try/catch/finally);
- 1 ponto para acoplamento com tipos criados dentro do sistema;
- 1 ponto para funções passadas como argumentos.

Códigos a tecnologias tranversais, como uso de frameworks e bibliotacas, não contam pontos. É considerado um conhecimento prévio que o desenvolvedor já deve possuir.

Exemplo de contagem:

```java
public class CompraRequest {

    //Dados do comprador
    @Email
    private String email;
    private String nome;
    @CpfCnpj //1 - Anotação criada no sistema
    private String documento;
    private String endereco;
    private String cidade;
    @ExistId(dominioClasse = Pais.class, nomeCampo = "id") //1 - Anotação criada no sistema
    private Long idPais;
    @ExistId(dominioClasse = Estado.class, nomeCampo = "id") //Anotação criada no sistema, mas já foi contabilizada
    private Long idEstado;

    //Dados da compra
    @Valid
    private PedidoCompraRequest pedido; //1 - Classe PedidoCompraRequest criada no sistema
    
    //1 - Classe Compra criada no sistema
    public Compra toModel(EntityManager manager){

        //1 - Classe Pais criada no sistema
        Pais pais = manager.find(Pais.class, this.idPais);
        //1 - Classe Estado criada no sistema
        //2 - Operador ternário - ramificação (conta 1 para o operador '?' e mais 1 para o operador ':')
        Estado estado = this.idEstado == null ? null : manager.find(Estado.class, this.idEstado);

        //1 - Classe ItemCompra criada no sistema
        List<ItemCompra> itens = this.pedido.getItens()
                .stream()
                .map(itemRequest -> itemRequest.toModel(manager))  //1 - Função como parâmetro 
                .collect(Collectors.toList());

        //Classe Compra já foi contabilizado
        Compra compra = new Compra(this.email, this.nome, this.sobrenome, this.documento,
                this.endereco, this.complemento, this.cidade, pais, estado,
                this.telefone, this.cep, itens, this.pedido.getTotal());
        compra.setCupom(this.pedido.getCodigoCupom(), manager);

        return compra;
    }
}
```

No código acima, a classe *CompraRequest* contém 10 pontos de carga intrínseca (PCI).

A recomendação de limites de PCI para as classes são:
- Para aplicações e apis web, entre 5 a 9 PCI;
- Para classes com atributos de dependências, geralmente as classes conhecidas como Controlles e Services, 7 PCI;
- Para classes com atributos de dados, geralmente são as Entities, value objects e classes de entrada e saída de dados, 9 PCI.


A classe *CompraRequest* pode ser considerada uma classe com atributos de dados e portanto limite de 9 PCI, a classe está portanto acima do limite.

Abaixo, removemos o operador ternário de forma a diminuir os PCI e ficar com a contagem dentro do limite de 9:

```java
public class CompraRequest {

    //Dados do comprador
    @Email
    private String email;
    private String nome;
    @CpfCnpj //1 - Anotação criada no sistema
    private String documento;
    private String endereco;
    private String cidade;
    @ExistId(dominioClasse = Pais.class, nomeCampo = "id") //1 - Anotação criada no sistema
    private Long idPais;
    @ExistId(dominioClasse = Estado.class, nomeCampo = "id") //Anotação criada no sistema, mas já foi contabilizada
    private Long idEstado;

    //Dados da compra
    @Valid
    private PedidoCompraRequest pedido; //1 - Classe PedidoCompraRequest criada no sistema
    
    //1 - Classe Compra criada no sistema
    public Compra toModel(EntityManager manager){

        //1 - Classe Pais criada no sistema
        Pais pais = manager.find(Pais.class, this.idPais);
        //1 - Classe Estado criada no sistema
        Estado estado = null;
        //1 - Ramificação
        if ( this.idEstado != null ) {
            estado = manager.find(Estado.class, this.idEstado);
        }

        //1 - Classe ItemCompra criada no sistema
        List<ItemCompra> itens = this.pedido.getItens()
                .stream()
                .map(itemRequest -> itemRequest.toModel(manager))  //1 - Função como parâmetro 
                .collect(Collectors.toList());

        //Classe Compra já foi contabilizado
        Compra compra = new Compra(this.email, this.nome, this.sobrenome, this.documento,
                this.endereco, this.complemento, this.cidade, pais, estado,
                this.telefone, this.cep, itens, this.pedido.getTotal());
        compra.setCupom(this.pedido.getCodigoCupom(), manager);

        return compra;
    }
}
```
