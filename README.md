# JVM Interpretador: Software Basico (CIC0104), UnB 2026/1

Implementacao de uma Java Virtual Machine em C++11 capaz de **ler, exibir e executar** arquivos `.class` do formato binario da JVM sem precisar de JRE instalada.

---

## Sumario

- [Participantes](#participantes)
- [Tecnologias](#tecnologias)
- [Escopo do Projeto](#escopo-do-projeto)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Requisitos](#requisitos)
- [Como Executar](#como-executar)
- [Detalhamento Tecnico](#detalhamento-tecnico)

---

## Participantes

| Nome | Matricula |
|------|-----------|
| Breno Back dos Santos Miranda da Silva | 190063980 |
| Danilo Silveira da Silva | 222014142 |
| Gustavo Vieira de Araujo | 211068440 |
| Julia Paulo Amorim | 241039270 |
| Leticia Goncalves Bomfim | 241002411 |
| Mariana Soares Oliveira | 231013663 |

---

## Tecnologias

| Tecnologia | Uso |
|-----------|-----|
| C++11 | Implementacao da JVM |
| GCC / g++ | Compilacao (Linux) |
| MinGW-w64 | Compilacao (Windows) |
| Make | Build system (Linux e Windows) |
| Doxygen | Geracao de documentacao |

---

## Escopo do Projeto

| Funcionalidade | Status |
|----------------|--------|
| Leitor de `.class` (parser BIG-ENDIAN completo) | Implementado |
| Exibidor estruturado com mnemônicos (modo `-d`) | Implementado |
| Constant Pool com resolucao de todos os 11 tipos | Implementado |
| Disassembly com `tableswitch` / `lookupswitch` | Implementado |
| Interpretador com dispatch table O(1) | Implementado |
| Pilha de frames (operand stack + variaveis locais + PC) | Implementado |
| Heap unificado para objetos e arrays | Implementado |
| Area de metodos com carregamento automatico de classes | Implementado |
| Heranca e polimorfismo (`find_method_ex`) | Implementado |
| Excecoes (`athrow`, `exception_table`, propagacao) | Implementado |
| Arrays com bounds check automatico | Implementado |
| Garbage Collector | Nao implementado |
| Biblioteca padrao Java (`java.lang`, `java.io`) | Nao implementado |
| `invokedynamic` / `BootstrapMethods` | Nao implementado |
| Multithreading | Nao implementado |

---

## Estrutura do Projeto

| Diretorio / Arquivo | Descricao |
|---|---|
| `Makefile` | build Windows (mingw32-make) |
| `Makefile.linux` | build Linux (`make -f Makefile.linux`) |
| `make.bat` | atalho Windows |
| `test.bat` / `test.sh` | testes (Windows / Linux) |
| `txt-all.bat` / `txt-all.sh` | gera `.txt` de todos os `.class` (Windows / Linux) |
| `Doxyfile` | config Doxygen |
| `docs/` | documentacao gerada pelo Doxygen |
| `include/types.h` | tipos primitivos (u1, u2, u4) |
| `include/errors.h` | enum `JvmError` |
| `include/class_file.h` | structs do formato `.class` |
| `include/class_reader.h` | leitura e liberacao de `ClassFile` |
| `include/constant_pool.h` | resolucao do Constant Pool |
| `include/attributes.h` | parse de Code, LineNumberTable, Exceptions |
| `include/opcodes.h` | enum `Opcode` + tabela `mnemonic[256]` |
| `include/displayer.h` | exibicao formatada |
| `include/frame.h` | `Frame` (operand stack + local vars + PC) |
| `include/jvm_stack.h` | pilha de frames |
| `include/object.h` | `JObject` (campos de instancia) |
| `include/array.h` | `JArray` (arraylength + elements) |
| `include/method_area.h` | `MethodArea` + `ClassEntry` |
| `include/interpreter.h` | JVM, dispatch table, heap, interpret loop |
| `src/main.cpp` | ponto de entrada (`-d` exibidor \| padrao interpretador) |
| `src/class_reader.cpp`, `constant_pool.cpp`, `attributes.cpp`, `opcodes.cpp`, `displayer.cpp`, `frame.cpp`, `jvm_stack.cpp`, `object.cpp`, `array.cpp`, `method_area.cpp`, `interpreter.cpp` | implementacao correspondente a cada header |
| `src/opcodes/` | handlers por categoria de opcode: `arithmetic.cpp`, `load_store.cpp`, `stack_ops.cpp`, `control.cpp`, `invoke.cpp`, `field_ops.cpp`, `object_ops.cpp`, `array_ops.cpp`, `convert.cpp`, `exceptions.cpp` |
| `tests/class/` | arquivos `.class` de teste |
| `tests/java/` | arquivos `.java` de teste |
| `exemplos/` | arquivos `.class` dos exemplos do professor |

---

## Requisitos

- `g++` com suporte a C++11 (GCC >= 4.8)
- `make` (Linux) ou `mingw32-make` (Windows, via [MinGW-w64](https://www.mingw-w64.org/))

---

## Como Executar

### Compilar (Linux)

```bash
make -f Makefile.linux
```

O executavel e gerado em `build/jvm`.

### Compilar (Windows)

```bat
make.bat
```

O executavel e gerado em `build\jvm.exe`.

### Modo interpretador: executa o `main` da classe

```bash
./build/jvm tests/class/HelloWorld.class        # Linux
build\jvm.exe tests\class\HelloWorld.class      # Windows
```

### Modo exibidor: exibe a estrutura do `.class`

```bash
./build/jvm -d tests/class/HelloWorld.class                      # Linux
build\jvm.exe -d tests\class\HelloWorld.class                    # Windows
```

Salvar saida em arquivo com `-o`:

```bash
./build/jvm -d -o output/HelloWorld.txt tests/class/HelloWorld.class   # Linux
```

Gerar um `.txt` por classe e `output/all.txt` com tudo:

```bash
make -f Makefile.linux output-all   # Linux
txt-all.bat                         # Windows
```

### Rodar todos os testes

```bash
make -f Makefile.linux test   # Linux
test.bat                      # Windows
```

---

## Detalhamento Tecnico

### Leitor/Exibidor (modo `-d`)

- Parser BIG-ENDIAN completo do formato `.class` (Java SE 8)
- Constant Pool com resolucao recursiva de todos os 11 tipos (`Utf8`, `Class`, `NameAndType`, `Fieldref`, `Methodref`, `String`, `Integer`, `Float`, `Long`, `Double`, `InterfaceMethodref`)
- Disassembly com mnemonicos, operandos resolvidos, `tableswitch` / `lookupswitch` formatados

### Interpretador (modo padrao)

- **Dispatch table O(1)**: `OpcodeHandler dispatch[256]` indexada pelo byte do opcode
- **Pilha de frames**: `JvmStack` com `Frame` (operand stack + variaveis locais + PC)
- **Heap unificado**: `vector<HeapEntry>` para objetos e arrays; referencias como `int32_t`
- **Area de metodos**: `unordered_map<string, ClassEntry*>` com carregamento automatico
- **Objetos**: `JObject` com campos de instancia cobrindo toda a hierarquia de heranca
- **Arrays**: `JArray` com campo `arraylength` explicito; bounds check automatico
- **Heranca e polimorfismo**: `find_method_ex` retorna a classe declarante do metodo
- **Excecoes**: `athrow`, `exception_table`, propagacao entre frames

Opcodes implementados por categoria em `src/opcodes/`:

| Arquivo | Categoria |
|---------|-----------|
| `arithmetic.cpp` | iadd, isub, imul, idiv, irem, ineg, fadd, dadd, ladd... |
| `load_store.cpp` | iload, istore, aload, astore, ldc, bipush, sipush... |
| `stack_ops.cpp` | pop, pop2, dup, dup2, swap |
| `control.cpp` | goto, if_icmp*, if_acmp*, tableswitch, lookupswitch, return* |
| `invoke.cpp` | invokevirtual, invokespecial, invokestatic, invokeinterface |
| `field_ops.cpp` | getfield, putfield, getstatic, putstatic |
| `object_ops.cpp` | new, instanceof, checkcast, arraylength |
| `array_ops.cpp` | newarray, anewarray, iaload, iastore, baload... |
| `convert.cpp` | i2l, i2f, i2d, l2i, f2i, d2i, i2b, i2c, i2s |
| `exceptions.cpp` | athrow |

---

> Documentacao gerada com auxilio de IA.
