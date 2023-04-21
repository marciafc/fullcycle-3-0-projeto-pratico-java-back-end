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

