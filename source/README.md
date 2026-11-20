# API do Regime Geral

Código-fonte da API do Regime Geral da Calculadora de Tributos da Reforma Tributária sobre o Consumo:
cálculo dos tributos do regime geral (CBS, IBS e Imposto Seletivo). É o mesmo fonte que gerou o `api-regime-geral.jar` da versão offline.

## Requisitos

- JDK 25 (`java` no `PATH` ou `JAVA_HOME`);
- acesso ao Maven Central: o Maven Wrapper (`mvnw`) baixa o Maven, os plugins e as dependências
  na primeira execução. Não é preciso instalar o Maven nem configurar outro repositório.

## Compilar e testar

Rode a partir deste diretório:

```bash
./mvnw verify          # Linux e macOS
mvnw.cmd verify        # Windows
```

O comando gera o banco de dados, compila, roda os testes e gera `target/api-regime-geral.jar`.

Para só gerar o jar, sem os testes: `./mvnw package -DskipTests`.

## Banco de dados

O banco SQLite da calculadora não acompanha o fonte: ele é criado pelos scripts de `sql/`, com o
Flyway, no primeiro build.

- `sql/criacao/` tem o script de criação e os que rodam antes e depois da migração;
  `sql/manutencao/` tem as alterações, aplicadas em ordem numérica;
- quando `calculadora/db/calculadora-pro.db` não existe, o build cria o banco em `target/banco/` e, se
  a migração termina sem erro, copia o arquivo para `calculadora/db/`. Com o arquivo no lugar, os
  builds seguintes não geram o banco de novo;
- para gerar só o banco: `./mvnw initialize` (`mvnw.cmd initialize` no Windows);
- para gerar o banco de novo, apague `calculadora/db/calculadora-pro.db` e rode o build. O banco é
  sempre criado do zero: o Flyway não é aplicado sobre um banco já gerado.

A API e os testes abrem o banco somente para leitura.

## Executar

```bash
java -XX:-UseBASE64Intrinsics -jar target/api-regime-geral.jar --spring.profiles.active=offline
```

Rode a partir deste diretório: a API abre o banco em `./calculadora/db/`.
A API fica em `http://localhost:8080/api` e a documentação em
`http://localhost:8080/api/swagger-ui.html`.

A opção `-XX:-UseBASE64Intrinsics` é um contorno temporário de segurança para a CVE-2026-114938
(corrupção de memória no decodificador Base64 do OpenJDK, em processadores x86-64 com AVX-512
VBMI). Use-a em toda execução da API enquanto o JDK instalado não trouxer a correção; com um JDK
corrigido, ela pode ser retirada.
