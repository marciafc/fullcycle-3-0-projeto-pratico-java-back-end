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

      - ```Identifier extends ValueObject**
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
	
  - [domain] com.fullcycle.admin.catalogo.domain.validation.**Validator** 	
  
  - CategoryTest
  
    - givenAnInvalidNullName_whenCallNewCategoryAndValidate_thenShouldReceiveError
	- DomainException
  
## Implementando o category validator	 
	 
	
	
  
