# Projeto prático - Java (Back-end)

  - Codeflix

  - Diagrama C4
  
  - Java 17
  
    - sdkman - [https://sdkman.io/usage](https://sdkman.io/usage)
	
	  - /home/nome-usuario/.sdkman
	  - sdk version
	  - $ sdk list java
	  - $ sdk install java 17.0.2-open
	  - $ java -version
        - openjdk version "17.0.2" 2022-01-18
      - Trocar version		
        - $ sdk use java 11.0.12-open
	    - $ sdk current java
		- $ java -version
		
    - IntelliJ
	
	- VSCode - [Instalar os plugins](https://github.com/codeedu/guia-rapido-java-vscode#configurando-vscode) + [Instalar plugin "Lombok Annotations Support for VS Code"](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-lombok)
	
	- Docker - [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
	
  - Modelagem estratégica com DDD	
  
    - Padrões táticos
	
      - Value Objects -> medir, quantificar e descrever algo no domínio, não tem ID, somente atributos que descrevem o objeto, são imutáveis
	  
      - Aggregates e entities
	    - Aggregates são as entidades raiz do sistema, possuem repositório
		- Entities possui um id, id é o que o identifica
		
      - Domain services -> serviços voltados para o domínio (regra de negócio)
	    - não será utilizado
		- no lugar será utilizado casos de uso -> clean arch
		
      - Factories -> criar agregados e entidades que são complexas
	  
  - Clean Architecture	  
  
  - Screaming Architecture
  
  - Components
  
    - Entities - pacote de domínio
	- Gateways - pacote de domínio
	- Use Cases - pacote de application
	- Presenters - pacote de infraestrutura
	
  - Código-fonte
  
    - [https://github.com/devfullcycle/FC3-admin-catalogo-de-videos-java](https://github.com/devfullcycle/FC3-admin-catalogo-de-videos-java)
	
	- [https://github.com/marciafc/FC3-admin-catalogo-de-videos-java](https://github.com/marciafc/FC3-admin-catalogo-de-videos-java)
	  
## Entidade categorias

  - [Notas](entidade-categorias/README.MD)
