# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: _9_ syscalls
- ex1b_write: _8_ syscalls

**2. Por que há diferença entre printf() e write()?**

```
O write() escreve diretamente no file descriptor, já o printf() utiliza buffers antes de chamar o write(). Dessa forma, dependendo do tamanho do buffer ou, em casos mais comuns, das quebras de linha, pode-se gerar mais write() do que printf().
```

**3. Qual implementação você acha que é mais eficiente? Por quê?**

```
A função write() é mais eficiente, pois envia dados diretamente ao file descriptor, assim evita possiveis overheads no buffer.
```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: _3_
- Bytes lidos: _127_

### Comando strace:
```bash
strace -e open,read,close ./ex2_leitura
```

### Análise

**1. Por que o file descriptor não foi 0, 1 ou 2?**

```
Os file descriptors 0, 1 e 2 são reservados e nesse caso retornou o 3 que era o próximo fd livre.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
O read() retornou 0, indicando que chegou ao fim do arquivo (nomeclatura conhecida - EOF (end of file)).
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: _24_ (esperado: 25)
- Caracteres: _1299_
- Chamadas read(): _21_
- Tempo: _0.000083_ segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |  82             | 0.000166  |
| 64          |  21             | 0.000083  |
| 256         |  06             | 0.000118  |
| 1024        |  02             | 0.000119  |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Como discutido anteriormente, o tamanho do buffer afeta o número de sysyscalls, pois o tamanho do buffer corresponde à quantidade de dados trazida através da syscall. Ou seja, quando menor o buffer, menos dados são trazidos e consequentemente mais chamadas serão necessárias.
```

**2. Como você detecta o fim do arquivo?**

```
O read() retorna 0 para indicar EOF.
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: _1364_
- Operações: _6_
- Tempo: _0.000958_ segundos
- Throughput: _1390.48_ KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para confirmar que os bytes lidos foram gravados.
```

**2. Que flags são essenciais no open() do destino?**

```
As seguintes flags:  O_WRONLY (escrita)| O_CREAT (criar)| O_TRUNC (apagar)
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
Um processo em modo usuário utiliza das syscalls para comunicar o kernel, que por sua ver irá acessar os recursos privilegiados isolados de outros processos
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os file descriptors permitem a comunicação entre o sistema operacional e arquivos/outros recursos de I/O através das syscalls. Assim, concluimos sua importância na manipulação de arquivos e outros dispositivos em comunicação.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Assim como discutido anteriormente, buffers maiores diminuem a quantidade de syscalls necessárias e, consequentemente, o overhead do kernel, aumentando desempenho.
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** _./ex4_copia_

**Por que você acha que foi mais rápido?**

```
Nesse caso, o ./ex4_copia foi mais rápido, pois o arquivo é pequeno e o overhead desse códgio foi menor em comparação com a cópia feita pelo cp. Porém, isso muda com arquivos maiores.
```

---

## Entrega

Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em `traces/`
- [X] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
