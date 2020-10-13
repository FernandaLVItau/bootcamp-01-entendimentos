# Anotações de códigos - JAVA

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
* Inicia a transação, finaliza a transação, trata erros, faz callback.
*Quando usa EntityManage precisa utilizar esta anotação, mas se usar Repository, o repository já tem os tratamentos de transação e não precisa da anotação*

## @PersistenceContext
Uma instancia **EntityManager** está associado a um contexto de persistencia
Lida com um conjunto de entidades que contêm dados a serem persistidos em, por exemplo, um banco de dados. O Contexto está ciente dos diferentes estados que uma entidade pode ter em relação ao contexto e armazernamento (ex. BD).

## @RestController
Marca  qye a classe é um controlador (controller) o qual cada método retorna um objeto de domínio em vez de uma visualização (view).
Esta anotação é um 'atalho' do uso das anotações:
- **@Controller**: Indica um tipo específico de compomente, o qual permite que a classe seja detectada automaticamente;
- **@ResponseBody**: Indica que o(s) método(s) retorna(m) um valor associado ao corpo de uma resposta da web.

## Serialização e desserialização
Serialização: Converter de outro formato, ex. texto JSON, para objeto;
Desserialização: Converter de objeto para outro formato necessário, ex. texto JSON;

O Spring precisa que as entidades contenham os setters e getters e/ou construtores para realizar as conversões automaticamente, por exemplo, para o controller retornar um objeto, que é uma entidade, e exibir na api o objeto em formato JSON, a entidade precisa ter os métodos getters.

## @InitBinder
Funciona como um pré-processador para cada requisição feita para o controller.

## WebDataBinder
Um tipo específico para requisição web de DataBinder, que permite definir propriedades para serem pré-processados na requisição.
Exemplo de uso para adicionar validação em uma requisicção:

```java
@InitiBinder
public void init(WebDataBinder binder) {
  binder.addValidators(new compraValidador());
}
```

## Anotação Bean Validation
Cria uma anotação para validações, como por exemplo o **@NotBlanck**.
Como criar:

1. Definir a anotação com uma **@interface** (note que utiliza uma @ e não apenas interface);
  1.1. É necessário os seguintes atributos:
    1.1.1. **message** com a mensagem padrão do erro;
    1.1.2. **groups** que permite especificar grupos de validação. Por *default* é um *array* vazio do tipo *Class<?>*;
    1.1.3. **playload** que pode ser utilizado por clientes da **API Jakarta Bean Validation** para personalizar **playload** de uma restrição. Ela não é usada pela própria API;
1.2. É utilizado as seguintes anotações:
  1.2.1. **@Target** é informado o alvo da anotação, por exemplo, **Field** (campo);
  1.2.2. **@Retention(RUNTIME)** especifica que a nova anotação estará disponível em tempo de execução;
  1.2.3. **@Documented** informa que a nova anotação estará contida no JavaDoc;
  1.2.4. **@Constraint(validatedBy = [..])** indica qual o validador será utilizado para validar os elementos anotados.

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
```

2. Definir o validador;
  2.1. A classe deve implementar a interface **ConstraintValidator** com dois parâmetros, a anotação criada e o tipo que o validador lida, ex. String;
  2.2. Deve implementar os dois métodos da interface:
  2.2.1. **inicialize** que fornece acesso aos valores de atributo da restrição, permitindo armazena-los em um campo do validador;
  2.2.2. **isValid** contém a lógica do validador.

```java
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

## Assert
Utilizado normalmente em testes, mas pode ser utilizado para validar status ao invés de utilizar *if*.
Se a condição falhar, é executado uma exceção com a mensagem definida.
Exemplo:
```java
Assert.state(list.size() <=1, "Foi encontrado mais de um registro");
```
No exemplo acima, é considerado um erro, que executa uma exceção, se na lista for tiver mais de um registro.

# Diferença entre @NotNull, @NotEmpty e @NotBlanck

Anotação | Dá erro na validação se
 | for nulo | estiver vazio | se estiver vazio depois de aparado
------------ | -------------
**@NotNull** | x | | 
**@NotEmpty** | x | x |
**@NotBlanck** | x | x | x

Nulo: Quando o objeto = null.
Vazio: Quando tamanho/comprimento do objeto = 0, i.e.: String "", Array [], Map {}, Char ''.
Aparado: Rempove espaços da String, i.e.: de "   " fica "".
