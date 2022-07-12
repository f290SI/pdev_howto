
## Tutorial API - PARTE I

### Requisitos

Para realização deste projeto iremos utilizar as dependencias. Como de praxe, crie o projeto `pdev` com as dependencias listadas abaixo.

#### Dependencias

 Dependencia | Descrição
 -- | --
 Web | Servidor de aplicação Web Apache TomCat.
 H2 | Servidor de Banco de dados em memória.
 JPA | Módulo para modelagem Objeto-Relacional de entidade de persistência em Java.
 MySQL | Conector para acesso à base de dados do SGBD MySQL.
 Security | Módulo de segurança para autenticação e gerenciamento de acesso.

### Application Properties

Iremos testar a aplicação em 3 ambientes distintos; ambiente de teste utilizando H2; ambiente de desenvolviemento utilizando banco de dados MySQL e ambiente de produção com hospedagem no [Heroku](https://www.heroku.com/).

> Criem os arquivos de configuração descritos abaixo para cada um dos ambientes citados.

#### Properties [resources/application.properties]

```properties
spring.profiles.active=test
```

#### Test Properties [resources/application-test.properties]
 ```properties
server.port=9000
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.jpa.defer-datasource-initialization=true
```
#### Dev Properties [resources/application-dev.properties]
```properties
server.port=9001
spring.datasource.url=jdbc:mysql://localhost:3306/pdev
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=root
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.hibernate.ddl-auto=update
```

#### Prod Properties [resources/application-prod.properties]
```properties
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=false

```

 ## Spring Security

 O Spring Security será responsável pela segurança do projeto, garantindo que os recursos da API sejam disponibilizados somente aos usuários autenticados com o devido nível de acesso concedido.

 ### Teste inicial Spring Security

 Após a criação do projeto, crie um end-point para testes e verifique se o end-point está bloqueado, caso esteja prosseguiremos para as configurações do Spring Security.

 1. Crie a classe `api/v1/resources/TestResource.java`;
 2. Adicione o trecho abaixo para criar um recurso de testes

```java
package br.com.fatecararas.pdevhowto.api.v1.resources;

import java.util.Map;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1")
public class TestResource {

    @GetMapping("/test")
    public ResponseEntity<Map<String, String>> test() {
        return ResponseEntity.ok().body(Map.of(
            "name", "PDev", 
            "version", "0.1.1", 
            "description", "Aplicação de compartilhamento de dicas para desenvolvedores da Fatec Araras."));
    }

}
``` 

3. Faça a requisição no Web Browser `http://localhost:8080/api/v1/test`. Um tela para autenticação deverá aparecer no navegador; caso não apareça para a execução do projeto e o abra novamente. O usuário padrão para o Spring security é `user`, a senha é gerada sempre que a aplicação for inciada.
4. Localize a senha e a copie para realizar a autenticação para a requisição realizada no Web Browser. Para a localizar, verifique o console pela string `Using generated security password`, a senha tem o pattern UUID como o exemplo abaixo.

```shell
Using generated security password: 2aff3b86-95df-4d47-a133-317081e02127
```

### Autenticação InMemory

Para compreendermos as principais configurações do Spring Security, iremos implementar a segurança urilizando validação em memória e depois por banco de dados.

#### Classes de configuração

Iremos criar um arquivo de configuração para as autenticações da API. As anotaçõesdo Spring Boot facilitam o precesso de configuração do projeto permitindo que os desenvolvedores possam ser mais produtivos e dispensem menos tempo em configurações de projeto.

1. Crie o arquivo com a classe `/api/v1/config/SecurityConfig.java`.
2. Inclua as anotações `@Configuration` e `@EnableWebSecurity`.
3. Extenda a classe `WebSecurityConfigurerAdapter `. A classe deve estar como o trecho abaixo.

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    // TODO: restante do código...
}
```

4. Faça a sobrescrita do método `configure` que contenha na assinatura a classe `HttpSecurity`.

```java
        // Recursos que não precisarão de autenticação
        final String[] PUBLIC_MATCHERS = { "/h2-console/**", "/api/v1/test" };

        http
                .authorizeRequests()
                .antMatchers(PUBLIC_MATCHERS).permitAll() // Liberar recursos listados
                .anyRequest().authenticated() // Inferir autenticação nos demais recursos
                .and()
                    .httpBasic() // Utilizar autenticação básica ao invés de página padrão do Spring
                .and()
                    .csrf() 
                    .disable();  // Desabilitar Cross Site Request Forgery             

        http.headers().frameOptions().disable(); // Necessário para sistema de frames utilizados pelo H2
```

#### Autenticação InMemory 

O Spring Security possui formas distintas para facilitar a autenticação; neste exemplo iremos utilizar a autenticação com configuração de usuários e roles em memória, esta é a forma mais simples de configuração.

Resumidamente, as requisições são capturadas e são filtradas conforme as configurações da classed `SecurityConfig.java`; porém é necessário validar os usuários e suas permissões, para tal, precisamos de um `Service` que obtenha as credenciais suas respectivas permissões.

Nesta parte do tutorial iremos implementar a autenticação em memória. O service `UserDetailsService` é responsável pela obtenção das credenciais. Realize os passos abaixo:

1. Faça a sobrescrita do método `userDetailsService` e tranforme-o em um `Bean` para que fique globalmente disponível na aplicação.

```java
    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        UserDetails user = User.withDefaultPasswordEncoder()
                .username("usuario")
                .password("123456")
                .roles("USER")
                .build();

        return new InMemoryUserDetailsManager(user);
    }
```

2. Remova a url do end-point de testes do `PUBLIC_MATCHERS`.
3. Reinicie e aplicação e teste o end-point com as credenciais ``InMemory.

# PARABÉNS! Terminamos a primeira parte

Na próxima parte iremos implementar a autenticação em banco de dados.