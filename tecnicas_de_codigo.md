# Anotações de códigos - JAVA

## Índice
1. [EntityManager](#entitymanager)
1. [@Transactional](#transactional)
1. [@PersistenceContext](#persistencecontext)
1. [@RestController](#restcontroller)
1. [Serialização e desserialização](#serialização-e-desserialização)
1. [@Autowired](#autowired)
1. [@InitBinder](#initbinder)
1. [WebDataBinder](#webdatabinder)
1. [Adicionar validação no InitBinder](#adicionar-validação-no-initbinder)
1. [Anotação Bean Validation](#anotação-bean-validation)
1. [Assert](#assert)
1. [Diferença entre @NotNull, @NotEmpty e @NotBlanck](#diferença-entre-notnull-notempty-e-notblanck)
1. [BigDecimal](#bigdecimal)
1. [.stream()](#stream)
1. [Lambda e método por referência](#lambda-e-método-por-referência)


## EntityManager

Classe responsável por gerenciar o ciclo de vida das entidades.

A classe é capaz de:

- Localizar entidades;
- Executar JPQL ou SQL nativo;
- Persistir entidades;
- Atualizar entidades;
- Remover entidades.


## @Transactional

Anotação para realiza transação.

* Inicia a transação; finaliza a transação; trata erros; faz callback.

*Quando usa EntityManage precisa utilizar esta anotação, mas se usar Repository, o repository já tem os tratamentos de transação e não precisa da anotação.*


## @PersistenceContext

Uma instancia **EntityManager** está associado a um contexto de persistencia

Lida com um conjunto de entidades que contêm dados a serem persistidos em, por exemplo, um banco de dados. O Contexto está ciente dos diferentes estados que uma entidade pode ter em relação ao contexto e armazernamento (ex. BD).


## @RestController

Marca  que a classe é um controlador (controller) o qual cada método retorna um objeto de domínio em vez de uma visualização (view).

Esta anotação é um 'atalho' do uso das anotações:

- **@Controller**: Indica um tipo específico de compomente, o qual permite que a classe seja detectada automaticamente;
- **@ResponseBody**: Indica que o(s) método(s) retorna(m) um valor associado ao corpo de uma resposta da web.


## Serialização e desserialização

Serialização: Converter de outro formato, ex. texto JSON, para objeto;

Desserialização: Converter de objeto para outro formato necessário, ex. texto JSON;

O Spring precisa que as entidades contenham os setters e getters e/ou construtores para realizar as conversões automaticamente, por exemplo, para o controller retornar um objeto, que é uma entidade, e exibir na api o objeto em formato JSON, a entidade precisa ter os métodos getters.


## @Autowired

Marca um construtor, campo, método setter ou método de configuração para ser autocarregado pelos recursos de injeção de dependência do Spring.


## @InitBinder

Funciona como um pré-processador para cada requisição feita para o controller.


## WebDataBinder

Um tipo específico para requisição web de DataBinder, que permite definir propriedades para serem pré-processados na requisição.

Por exemplo, adicinar para realizar algumas validações antes de processar a requisição.


## Adicionar validação no InitBinder

```java
@Autowired
Validador1 validador1;

@InitiBinder
public void init(WebDataBinder binder) {
  binder.addValidators(validado1, new validador2());
}
```

Ambas as formas podem ser passadas como parâmetro para `addValidators`. Se a classe que realiza a validação tiver propriedades que usam Sprint para inicializar, como por exemplo **@Autowired**, deve realizar da forma do `validador1`.

As classes de validação devem implementar **Validator** do *org.springframework.validation.Validator*.


## Anotação Bean Validation

Cria uma anotação para validações, como por exemplo o **@NotBlanck**.

Como criar:

1. Definir a anotação com uma **@interface** (note que utiliza uma @ e não apenas interface);
   1. É necessário os seguintes atributos:
      1. **message** com a mensagem padrão do erro;
      1. **groups** que permite especificar grupos de validação. Por *default* é um *array* vazio do tipo *Class<?>*;
      1. **playload** que pode ser utilizado por clientes da **API Jakarta Bean Validation** para personalizar **playload** de uma restrição. Ela não é usada pela própria API;
   1. É utilizado as seguintes anotações:
      1. **@Target** é informado o alvo da anotação, por exemplo, **Field** (campo);
      1. **@Retention(RUNTIME)** especifica que a nova anotação estará disponível em tempo de execução;
      1. **@Documented** informa que a nova anotação estará contida no JavaDoc;
      1. **@Constraint(validatedBy = {..})** indica qual o validador será utilizado para validar os elementos anotados.
1. Definir o validador;
   1. A classe deve implementar a interface **ConstraintValidator** com dois parâmetros, a anotação criada e o tipo que o validador lida, ex. String;
   1. Deve implementar os dois métodos da interface:
      1. **inicialize** que fornece acesso aos valores de atributo da restrição, permitindo armazena-los em um campo do validador;
      1. **isValid** contém a lógica do validador.

```java
/*
 * Anotação para validar se o id de um registro existe na base de dados.
*/
@Target(FIELD)
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {ExistIdValidator.class})
public @interface ExistId {

    String message() default "não cadastrado";
    Class<?>[] groups() default { };
    Class<? extends Payload>[] payload() default { };

    String nomeCampo();         // Será passado na anotação o atribudo nomeCampo com o nome do campo que será validado 
    Class<?> dominioClasse();   // Será passado na anotação o atributo dominioClasse com o domínio da Classe da entidade 
}

/*
 * Validação da anotação ExistId.
*/
public class ExistIdValidator implements ConstraintValidator<ExistId, Long> {
    private String nomeCampo;
    Class<?> dominioClasse;

    @PersistenceContext
    private EntityManager manager;

    @Override
    public void initialize(ExistId constraintAnnotation) {
        nomeCampo = constraintAnnotation.nomeCampo();
        dominioClasse = constraintAnnotation.dominioClasse();
    }

    @Override
    public boolean isValid(Long value, ConstraintValidatorContext constraintValidatorContext) {
        if (value == null) {
            return true;
        }
        
        Query query = manager.createQuery("select 1 from "+dominioClasse.getName()+" where "+nomeCampo+"=:value");
        query.setParameter("value", value);
        List<?> list = query.getResultList();

        return !list.isEmpty();
    }
}
```

Exemplo de uso da anotação criada acima:
```java
@ExistId(dominioClasse = Pais.class, nomeCampo = "id")
private Long idPais;
```


## Assert
Utilizado normalmente em testes, mas pode ser utilizado para validar status ao invés de utilizar *if*.

Se a condição falhar, é executado uma exceção com a mensagem definida.

Exemplo:
```java
Assert.state(list.size() <=1, "Foi encontrado mais de um registro");
```
No exemplo acima, é considerado um erro que executa uma exceção, se na lista tiver mais de um registro.


## Diferença entre @NotNull, @NotEmpty e @NotBlanck

Anotação | Se for nulo | Se estiver vazio | Se estiver vazio depois de aparado
-------- | ------------|------------------|-----------------------------------
**@NotNull** | x | | 
**@NotEmpty** | x | x |
**@NotBlanck** | x | x | x

Nulo: Quando o objeto = null.

Vazio: Quando tamanho/comprimento do objeto = 0, i.e.: String "", Array [], Map {}, Char ''.

Aparado: Rempove espaços da String, i.e.: de " teste " fica "teste".


## BigDecimal

As casas decimais são mais precisos que o **double**, portanto mais recomendado para trabalhar com valores monetários.

A desvantagem é no uso para operações.


## .stream()

Converte **List** para uma interface do tipo **Stream** e dela podemos realizar operações **Filter**, **Map** e **Reduce**.

Esta interface permite chamar um método depois do outro, ou seja, de forma encadeara.

No exemplo abaixo, filtra dados de uma lista, mapea os dados do objeto para números inteiros e por fim realiza a somatória destes valores mapeados:

```java
lista.stream().filter(...).mapToInt(...).sum()
```


## Lambda e método por referência

Considere o exemplo em que há uma lista de objeto `Livro1` que será convertida para uma lista de objetos `Livro2`. A classe `Livro2` tem um construtor que recebe um objeto `Livro1` como parâmetro.

-> Código 1 - Utlizando laço *for*
```java
for(Livro l : listaLivro) {
  lista2Livro[] = new Livro2(l);
}
```

-> Código 2 - Utilizando Lambda
```java
lista2Livro = listaLivro.stream().map(l -> { return new Livro2(l); });
```

-> Cósigo 3 - Utilizando método por referência
```java
lista2Livro = listaLivro.stream().map(Livro2::new);
```

Lambda é uma função anônima.

Uma expressão lambda pode ser substituda por uma referência, o argumento é obtido por inferência.
