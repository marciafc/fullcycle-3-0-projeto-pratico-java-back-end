## A camada de aplicação na arquitetura limpa

  - git clone https://github.com/devfullcycle/FC3-admin-catalogo-de-videos-java.git
  - git checkout modulos/02/usecases-categoria
  
  - Module 'application'
    - Use Cases - orquestrar o comportamento das entidades
	
## Criando a estrutura base de Usecase

  - **Use Cases** na arquitetura limpa implementam o **pattern Command** 
  
  - **Pattern command**: a semântica do que ele faz, é o nome da classe (único método **execute**)
  
    - Exemplo: CreateCategoryUseCase.java
  
  - UseCase.java -> ```public abstract class UseCase<IN, OUT>```
    - Na maioria das vezes recebe algo e retorna algo -> IN, OUT
	
  - UnitUseCase.java -> ```public abstract class UnitUseCase<IN>```	
    - Recebe algo, mas não devolve nada
	
	
  - NullaryUseCase.java -> ```public abstract class NullaryUseCase<OUT>```
    - Não recebe nada como argumento, somente tem uma saída
	
## Importando o mockito para mockar comportamento

  - [Mockito](https://site.mockito.org/)
  
  - Mockito integrado com Jupiter 
    - Incluir dependência do Mockito no 'build.gradle' do module 'application'
      - testImplementation 'org.mockito:mockito-junit-jupiter:4.5.1'
	- Jupiter (JUnit 5) -> Jupiter codinome do JUnit 5
	
  - Só incluir a annotation ```@ExtendWith(MockitoExtension.class)``` para utilizar
  
```  
@ExtendWith(MockitoExtension.class)
public class UseCaseTest {
  // ...
}
```

## Testes unitários do caso de uso de criação de categoria

  - CreateCategoryUseCaseTest.java
```
    // 1. Teste do caminho feliz
	givenAValidCommand_whenCallsCreateCategory_shouldReturnCategoryId	
   
```  
  - Chamando o método verdadeiro com Mockito -> **thenCallRealMethod()**
    - ```when(objeto.nomeMetodo(any())).thenCallRealMethod()```
		
  - Diferença de usar **thenReturn x thenAnswer**
    - [Mockito thenReturn vs thenAnswer](https://stacktraceguru.com/unittest/thenreturn-vs-thenanswer)
	
  - **verify** -> verifica que o 'categoryGateway' foi chamado UMA vez no método 'create'
  
  - **argThat** -> comparando o parâmetro de entrada com o que obteve de resposta
  
  - **returnsFirstArg** -> retornar o primeiro parâmetro informado
    - que é a nova categoria criada
  
```  

when(categoryGateway.create(any()))
                .thenAnswer(returnsFirstArg());

final var actualOutput = useCase.execute(aCommand).get();

// Fazendo as asserções do retorno do método execute()
Assertions.assertNotNull(actualOutput);
Assertions.assertNotNull(actualOutput.id());				
	
// Fazendo asserção	que o categoryGateway.create() foi chamado UMA vez e que
// os parâmetros de entrada (expectedName, expectedDescription e expectedIsActive)
// são iguais ao da resposta (aCategory -> lambda) além de outros testes de 'aCategory'
Mockito.verify(categoryGateway, times(1)).create(
		argThat(aCategory ->
                           Objects.equals(expectedName, aCategory.getName())
                        && Objects.equals(expectedDescription, aCategory.getDescription())
                        && Objects.equals(expectedIsActive, aCategory.isActive())
                        && Objects.nonNull(aCategory.getId())
                        && Objects.nonNull(aCategory.getCreatedAt())
                        && Objects.nonNull(aCategory.getUpdatedAt())
                        && Objects.isNull(aCategory.getDeletedAt())  
			   )
);						
```

## Implementando o caso de uso de criação de categoria 
  
  - continuação da aula anterior
  
  - [application] com.fullcycle.admin.catalogo.application.category.create
  
    - entrada -> record CreateCategoryCommand.java
	- saída -> record CreateCategoryOutput.java
	- use case -> **abstract class** CreateCategoryUseCase
	  - ainda não é a implementação
	  - não trabalhe para uma implementação, trabalhe para uma abstração
	    - nao acople o código na implementação e sim na abstração
    - use case -> DefaultCreateCategoryUseCase extends CreateCategoryUseCase
      - aqui é a implementação!
	  - injeção de dependência via construtor
	    - evitar dependência cíclica, por exemplo
	
## Cobrindo com demais cenários de teste o caso de uso de criação de categoria
  
  - CreateCategoryUseCaseTest.java
```
    // 2. Teste passando uma propriedade inválida (name)
	givenAInvalidName_whenCallsCreateCategory_thenShouldReturnDomainException	
	
```  
  - Mock não usar no método
  
    - Não: Mockito.mock(CategoryGateway.class) OU new DefaultCreateCategoryUseCase
	
	- Sim: como abaixo
	
```
// O Mockito vai instanciar -> mais sintax sugar os testes
@ExtendWith(MockitoExtension.class)
public class CreateCategoryUseCaseTest {

    @InjectMocks
    private DefaultCreateCategoryUseCase useCase;

    @Mock
    private CategoryGateway categoryGateway;

}
```

  - Verifica se não chamou **nenhuma vez**, passar any() -> não terá verificação
    - Mockito.verify(categoryGateway, times(0)).create(any());

  - CreateCategoryUseCaseTest.java
```
    // 3. Teste criando uma categoria inativa
	givenAValidCommandWithInactiveCategory_whenCallsCreateCategory_shouldReturnInactiveCategoryId	
	
```  

  - CreateCategoryUseCaseTest.java
```
    // 4. Teste simulando um erro generico vindo do gateway
	givenAValidCommand_whenGatewayThrowsRandomException_shouldReturnAException	
	
```
