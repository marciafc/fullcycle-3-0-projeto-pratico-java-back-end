## O conceito de pirâmide de testes

  - Unit (+ rápido, não depende de framework, junit runner)
  
  - Integration -> testa se duas ou mais unidades que se interagem estão funcionando como esperado. O framework é encontrado na camada de infraestrutura. Sobe o framework (pelo menos uma parte dele - persistência, rest, chamadas http... para testar em conjunto - e isso demora mais do que o unitário que nao precisa subir nada junto)
    - Se uma chamada http está indo com os parâmetros corretos, verbo correto - poderia estar usando um [WireMock](https://wiremock.org/) para isso
  
  - E2E (+ lento, menor quantidade) - sobe o sistema inteiro
    - Cucumber
	- Desde o controller, todo o sistema
	
## Criando o projeto base com Gradle

  - $ git clone https://github.com/devfullcycle/FC3-admin-catalogo-de-videos-java
  - $ git checkout modulos/01/entidade-categorias
  
## Configurando os submódulos do projeto base

  - domain
  - application -> casos de uso
  - infrastructure -> injeção de dependências (liga casos de uso aos entrypoints - entradas)
  
  - IntelliJ: New \ Module
    - "settings.gradle" aparece um include com novo módulo criado
	- Remover pasta "src" da raiz
	
  - Module 'application' e 'infrastructure' importar o 'domain' como dependência
    - implementation(project(":domain"))
	
  - Module 'infrastructure' importar o 'application' como dependência
    - implementation(project(":application"))
	
  - O Main está no module 'infrastructure'
  
## Criando a entidade category com TDD

  - [domain] com.fullcycle.admin.catalogo.domain.category.Category.java
  
    - Usar 'Instant' -> marco no tempo, não leva em conta o Timezone, é sempre UTC 
	
      - LocalDateTime -> específico dia, específica hora 
	  
    - **Category.newCategory(final String aName, ...)** -> Factory method -> instanciar uma categoria
	
	  - final nos parâmetros para serem imutáveis	  
	  
	  - **private Category(final String aName,...)** -> construtor private e parâmetros imutáveis
  
    - CategoryTest
	
	  - givenAValidParams_whenCallNewCategory_thenInstantiateACategory
	    - Category.newCategory(expectedName, expectedDescription, expectedIsActive)
	
## Implementando as classes do DDD

  - [domain] com.fullcycle.admin.catalogo.domain
  
    - Classes abstract -> nao podem ser interfaces, não é para implementar:
  
      - ```ValueObject``` 

      - ```Identifier extends ValueObject```
        - Objeto de valor é **definido pelos seus atributos**
		
      - ```Entity<ID extends Identifier>```
	    - A entidade é **definido pelo seu identificador**
	    - Sobrescrever o equals e o hashCode baseado no id
	    - No contrutor não permite instanciar com id indevido 
		  - ```Objects.requireNonNull(id, "'id' should not be null")```
	  
      - ```AggregateRoot<ID extends Identifier> extends Entity<ID>```
	    - É o agregado raiz

## Tornando a classe category um AggregateRoot

  - [domain] com.fullcycle.admin.catalogo.domain.category
  
    - ```Category extends AggregateRoot<CategoryID>``` -> é um agregado
	
    - ```CategoryID extends Identifier``` -> objeto de valor, representa algo
	
      - private **CategoryID**(final String value) -> construtor privado
	  
        - Contém uma contraint: Objects.requireNonNull(value)
		
      - public static **CategoryID** from(final String anId) -> Factory method
	  
      - public static **CategoryID** unique() -> Factory method
        - id único
	 
      - getValue() -> não terá setValue()
	  
	  - equals e hashCode
	 
## Introdução as validações de negócio 	 

  - [domain] com.fullcycle.admin.catalogo.domain.category.**CategoryValidator**
  
    - CategoryValidator extends Validator	
	
	- CategoryValidator -> validador externo
	
  - [domain] com.fullcycle.admin.catalogo.domain.validation.**Validator** 
    - Validator -> classe pai dos validadores
    - private final ValidationHandler handler;
  
  - CategoryTest
  
    - givenAnInvalidNullName_whenCallNewCategoryAndValidate_thenShouldReceiveError
	- DomainException
  
## Implementando o category validator	 

  - [domain] Entity.java
    - public abstract void validate(ValidationHandler handler);
	  - Notification -> acumula os erros
	
  - [domain] pacote com.fullcycle.admin.catalogo.domain.validation
    - ValidationHandler.java -> interface fluente
	- record Error.java

  - [domain] Category.java - override do método validate
    - Chamar o validador da categoria
	
```  
	@Override
    public void validate(final ValidationHandler handler) {
        new CategoryValidator(this, handler).validate();
    }  
```

  - [domain] com.fullcycle.admin.catalogo.domain.validation.handler.**ThrowsValidationHandler**.java
  
    - Não é acumulativo
	- Lançar uma exceção cada vez que receber um tipo de erro
	  - throw DomainException nos append
```	
   @Override
    public ValidationHandler append(final Error anError) {
        throw DomainException.with(anError);
    }

    @Override
    public ValidationHandler append(final ValidationHandler anHandler) {
        throw DomainException.with(anHandler.getErrors());
    }	
```

  - [domain] CategoryValidator.java -> checkNameConstraints()
    - implementar as validações
	
  - [domain] CategoryTest -> usando o ThrowsValidationHandler

```
Assertions.assertThrows(DomainException.class, () -> 
                  actualCategory.validate(new ThrowsValidationHandler()));  
```

## Refinando nossas exceptions

  - RuntimeException tem essa assinatura:
  
```  
    protected RuntimeException(String message, Throwable cause,
                               boolean enableSuppression,
                               boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }    
```
	
  - **boolean enableSuppression** e **boolean writableStackTrace**: argumentos que impactam a performance da aplicação, não coloca stacktrace dentro da exception - em uma aplicação de alta performance é muito importante
  
  - Criar exception:
    - **NoStacktraceException** extends RuntimeException  
	
```	
  	// chamando o construtor super() que recebe os argumentos enableSuppression e writableStackTrace
	public NoStacktraceException(final String message, final Throwable cause) {
        super(message, cause, true, false);
    }	
	
	// construtor que vai chamar o outro construtor
 	public NoStacktraceException(final String message) {
        this(message, null);
    }	
```

## Implementando demais validações com TDD

  - [domain] CategoryTest
  
   - givenAnInvalidEmptyName_whenCallNewCategoryAndValidate_thenShouldReceiveError
```
final var actualException =
                Assertions.assertThrows(DomainException.class, () -> actualCategory.validate(new ThrowsValidationHandler()));	
```
   - givenAnInvalidNameLengthLessThan3_whenCallNewCategoryAndValidate_thenShouldReceiveError

```
 final var actualException =
                Assertions.assertThrows(DomainException.class, () -> actualCategory.validate(new ThrowsValidationHandler()));	
```
   - givenAnInvalidNameLengthMoreThan255_whenCallNewCategoryAndValidate_thenShouldReceiveError
```
final var actualException =
                Assertions.assertThrows(DomainException.class, () -> actualCategory.validate(new ThrowsValidationHandler()));	
```
   - givenAValidEmptyDescription_whenCallNewCategoryAndValidate_thenShouldReceiveOK	
```	
Assertions.assertDoesNotThrow(() -> actualCategory.validate(new ThrowsValidationHandler()));	
```
   - givenAValidFalseIsActive_whenCallNewCategoryAndValidate_thenShouldReceiveOK
```
 Assertions.assertDoesNotThrow(() -> actualCategory.validate(new ThrowsValidationHandler()));
```

## Comportamento de ativar e inativar uma category

  - [domain] CategoryTest
  
  - **DESATIVAR** givenAValidActiveCategory_whenCallDeactivate_thenReturnCategoryInactivated
	
	  - isAfter: será uma data futura
	
	    - Assertions.assertTrue(actualCategory.getUpdatedAt().**isAfter**(updatedAt))

    - Como o deactivate() altera "updatedAt", salvar em uma variável antes

```
// ...
final var createdAt = aCategory.getCreatedAt();
final var updatedAt = aCategory.getUpdatedAt();

// ...
final var actualCategory = aCategory.deactivate();
	
// ...
Assertions.assertEquals(createdAt, actualCategory.getCreatedAt());
Assertions.assertTrue(actualCategory.getUpdatedAt().isAfter(updatedAt));
Assertions.assertNotNull(actualCategory.getDeletedAt());
```
  - **ATIVAR** givenAValidInactiveCategory_whenCallActivate_thenReturnCategoryActivated
  
    - aCategory.activate()
 
## Implementando update de uma category

- [domain] CategoryTest
  
  - **ATUALIZAR** -> aCategory.update(expectedName, expectedDescription, expectedIsActive);
  
    - givenAValidCategory_whenCallUpdate_thenReturnCategoryUpdated
	
	- givenAValidCategory_whenCallUpdateToInactive_thenReturnCategoryUpdated
	
	- givenAValidCategory_whenCallUpdateWithInvalidParams_thenReturnCategoryUpdated
	
	  - Após o update() nao vai chamar/validar o validate() para não dar exceção -> name == null
```
final var actualCategory = aCategory.update(expectedName, expectedDescription, expectedIsActive);

Assertions.assertEquals(aCategory.getId(), actualCategory.getId());
Assertions.assertEquals(expectedName, actualCategory.getName());
```

## Criando a interface do Gateway de category

  - [domain] com.fullcycle.admin.catalogo.domain.category.**CategoryGateway**.java
  
    - public interface CategoryGateway

  - record -> dto imutável para passar informação de um lado para o outro
  
  - Paginaçao -> pacote **com.fullcycle.admin.catalogo.domain.pagination**
  
    - Pagination
	
	- SearchQuery
	
## Revisão do módulo

  - TDD
  
  - DDD
  
  - Clean Architecture