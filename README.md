# Testes para código gerenciado
## Tecnologia e Ferramentas Utilizadas
- .NET Framework: Uma plataforma de desenvolvimento de software desenvolvida pela Microsoft que oferece um ambiente para criar, implantar e executar aplicativos e serviços.
- Rider: Uma IDE multiplataforma desenvolvida pela JetBrains para desenvolvimento em .NET, entre outras linguagens e frameworks, foi utilizada devido a limitação do Visual Studio em meu sistema operacional Linux.
- NUnit: Uma estrutura de teste unitário para a plataforma .NET que fornece uma maneira simples e limpa de escrever e executar testes de unidade em código .NET.

## Relatório
Inicialmente foi criado um projeto de tipo Console Application com o código inicial do artigo, que se refere a classe `BankAccount`. Em seguida, seguindo as instruções foi criado o projeto de teste de unidade, que precisou ser criado em NUnit devido a incompatibilidade da IDE com o MSTest, e a classe de testes foi nomeada como `BankTests`. O código base para os testes precisou ser alterado, já que no NUnit Framework as declarações são um pouco diferentes, de forma a qual o `[TestClass]` e o `[TestMethod]` foram respectivamente substituídos por `[TestFixture]` e `[Test]`. 

<img src="https://github.com/vict0rcarvalh0/Unit-Test/blob/main/assets/image1.png">

### 1. Criação do primeiro método de teste
O primeiro método de teste foi criado com o propósito de verificar que um valor válido, ou seja, um que seja menor que o saldo e maior que zero, retira a quantidade correta da conta de acordo com o método `Debit` da classe `BankAccount`.
```csharp
[Test]
public void Debit_WithValidAmount_UpdatesBalance()
{
    // Arrange
    double beginningBalance = 11.99;
    double debitAmount = 4.55;
    double expected = 7.44;
    BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

    // Act
    account.Debit(debitAmount);

    // Assert
    double actual = account.Balance;
    Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
}
```

Nesse método, o AssertAreEqual é utilizado para verificar se o saldo final é o esperado.

Resultado:
<img src="https://github.com/vict0rcarvalh0/Unit-Test/blob/main/assets/image2.png">

Propositalmente o tutorial deixa um trecho de código errado no método `Debit` da classe `BankAccount`, que foi do valor do saque que foi adicionado ao saldo ao invés de ser subtraído, já que o propósito é debitar da conta. Diante dos parâmetros passados de 11,99 de saldo inicial e 4,55 de débito, deveria resultar em 7,44. No entanto, o resultado foi de 16,54. Por isso, o teste falha(Falso negativo)

### 2. Correção do primeiro método de teste
O método `Debit` foi corrigido por meio da alteração de `m_balance += amount;` para `m_balance -= amount;`.

Resultado:
<img src="https://github.com/vict0rcarvalh0/Unit-Test/blob/main/assets/image3.png">
O teste está conforme o esperado(Verdadeiro positivo), ou seja, quando um valor é debitado de uma conta, ele é subtraído e bate com o valor esperado.

### 3. Criação de novos métodos de teste
Nessa etapa, foi criado um novo método de teste com propósito de verificar o comportamento correto quando o valor do débito é menor que 0. Onde é declarado um saldo inicial positivo e uma quantidade de débito negativa.
```csharp
[Test]
public void Debit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange()
{
    // Arrange
    double beginningBalance = 11.99;
    double debitAmount = -100.00;
    BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

    // Act and assert
    Assert.Throws<System.ArgumentOutOfRangeException>(() => account.Debit(debitAmount));
}
```
OBS: O método `ThrowsException` foi substituído por `Throws` devido a versão do .NET utilizada em conjunto ao Framework de testes NUnit.

O método Throws declara que a exceção correta foi gerada, o que faz com que se o teste falhar, uma Exception estoure e mostre uma exceção um pouco mais específica, visando encontrar o motivo da falha de maneira mais fácil.

O outro método de teste criado segue uma lógica parecida, que consiste em testar quando o valor de débito é maior que o saldo, que deveria indicar que o saldo é insuficient.Onde o corpo é quase igual ao do último método, mas com uma quantidade de débito maior que a do montante inicial.
```csharp
[Test]
public void Debit_WhenAmountIsMoreThanBalance_ShouldThrowArgumentOutOfRange()
{
    // Arrange
    double beginningBalance = 11.99;
    double debitAmount = 100.00;
    BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

    // Act and assert
    Assert.Throws<System.ArgumentOutOfRangeException>(() => account.Debit(debitAmount));
}
```

Resultados:
<img src="https://github.com/vict0rcarvalh0/Unit-Test/blob/main/assets/image4.png">
Ambos os testes passam como o esperado, já que as exceções, mesmo sendo exceções, satisfazem a definição do teste. Porém, indica falso positivo, já que o resultado é inválido, mas satisfaz o comportamento esperado.

### Refatorando os métodos de teste
Alterando o código do método `Debit`, podemos implementar o ApplicationException para que os métodos `Debit_WhenAmountIsMoreThanBalance_ShouldThrowArgumentOutOfRange` e `Debit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange` falhem devidamente(Verdadeiro negativo), ao invés de passarem no teste(satisfazem a condição esperada). Mas o artigo não foca nisso, mas sim na melhoria dos métodos de teste já criados, por meio da seguinte modificação na classe `BankAccount`, que define mensagens de erro mais específicas:
`csharp
public const string DebitAmountExceedsBalanceMessage = "Debit amount exceeds balance";
public const string DebitAmountLessThanZeroMessage = "Debit amount is less than zero";
`
```csharp
if (amount > m_balance)
{
    throw new System.ArgumentOutOfRangeException("amount", amount, DebitAmountExceedsBalanceMessage);
}

if (amount < 0)
{
    throw new System.ArgumentOutOfRangeException("amount", amount, DebitAmountLessThanZeroMessage);
}
```

Além disso, foi feito um encapsulamento da chamada a `Debit` em um bloco try/catch, de forma que capture uma exceção específica que é esperada e verifique a mensagem associada. Outra adição é de um return fora do try/catch com uma declaração `Assert.Fail`, já que o bloco catch captura a exceção, mas o método continua sendo executado, com essa implementação se a exceção não for a esperada pelo teste, ele falha.
```csharp
[Test]
public void Debit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange()
{
    // Arrange
    double beginningBalance = 11.99;
    double debitAmount = -100.00;
    BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);
    
    // Act
    try

    {
        account.Debit(debitAmount);
    }
    catch (System.ArgumentOutOfRangeException e)
    {
        // Assert
        StringAssert.Contains(e.Message, BankAccount.DebitAmountLessThanZeroMessage);
        return;
    }
    Assert.Fail("The expected exception was not thrown.");
}
```
```csharp
[Test]
public void Debit_WhenAmountIsMoreThanBalance_ShouldThrowArgumentOutOfRange()
{
    // Arrange
    double beginningBalance = 11.99;
    double debitAmount = 100.00;
    BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

    // Act
    try

    {
        account.Debit(debitAmount);
    }
    catch (System.ArgumentOutOfRangeException e)
    {
        // Assert
        StringAssert.Contains(e.Message, BankAccount.DebitAmountExceedsBalanceMessage);
        return;
    }
    Assert.Fail("The expected exception was not thrown.");
}
```

## Aprendizados
### Declarações Arrange, Act e Assert
A compreensão e aplicação das declarações Arrange, Act e Assert são fundamentais no contexto de testes de unidade. A declaração Arrange é utilizada para configurar o ambiente de teste, preparando o estado inicial necessário para executar o teste. A declaração Act representa a ação que está sendo testada, ou seja, é o momento em que o comportamento ou método em análise é invocado. Por fim, a declaração Assert é responsável por verificar se o comportamento esperado foi alcançado após a execução da ação, por meio de asserções que comparam o resultado obtido com o resultado esperado.

### Criação de Projeto de Teste em C#
A criação de um projeto de teste em C# envolve a configuração de um ambiente separado para testar o código de produção. Neste projeto de teste, são escritos os casos de teste que verificam o comportamento das unidades de código, como classes e métodos. Além disso, é importante organizar e estruturar os testes de forma adequada para garantir a eficácia e a manutenibilidade do conjunto de testes.

### Uso do Framework de Testes NUnit
O uso do framework de testes NUnit proporcionou uma maneira simples e eficaz de escrever e executar testes de unidade em código .NET. A sintaxe limpa e intuitiva do NUnit facilita a criação de casos de teste e asserções, além de oferecer recursos avançados para configurar o ambiente de teste e gerenciar a execução dos testes. A familiaridade com as funcionalidades do NUnit é essencial para garantir a qualidade dos testes e a confiabilidade do processo de desenvolvimento de software. Além disso, foi muito interessante utilizar uma ferramenta diferente do proposto no artigo, pois precisei adaptar poucas coisas e a lógica no fim das contas, foi idêntica para ambos os Frameworks.

### Tratamento de Exceções
Entendimento do tratamento de exceções em testes de unidade, incluindo a captura e verificação de exceções lançadas pelo código em teste para garantir que o comportamento esperado seja alcançado, aumentando a robustez e a confiabilidade dos testes.

### Importância dos Testes Negativos
Reconhecimento da importância dos testes negativos, que verificam condições de falha com entradas inválidas, onde os cenários de erro são esperados. Esses testes são fundamentais para garantir que o software se comporte corretamente mesmo diante de condições inesperadas ou inválidas, melhorando a qualidade e a confiabilidade do código.
