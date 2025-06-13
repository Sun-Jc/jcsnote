# ZKP Programmability Analogy (Draft)

by JCS, Jun 10, 2025

In this note, I will show some analogies between ZKP programmability and the modern computer.
It contains many historical perspectives.
Caveat: I could make unrigorous statement and miss a lot of references, as I am not an expert of computer history.
I am happy to constantly improve and update this note.

This note is my retrospective of the history of ZKP programmability.
I tried to figure out what ZKVM really brings to the table.
It will not explain what is ZKP or how to build a ZKVM.
It will not discuss any properties of ZKP protocols such as post-quantum security.

## Evolution: Sacrifice Efficiency for Programmability

Programmability measures the ability that a machine (in hardware or software) can be adapted according to the requirements.
From the developers' perspective, the difficulty level of expressing the requirements represents the programmability.

The trend of programmability is from hardware-coupling to more abstract and decoupling software engineering.

The program efficiency evolves from high to low, but as the performance of the machine is increasing, the overall speed of the program still remains acceptable and probably faster.

It becomes much easier to build large scale applications with more abstraction.
This is what the trend brings, at the cost of efficiency which becomes trivial though.


## Dawn

During 1980s, the theory on verifiable computing (proofs, arguments and zero-knowledge) was born.
It shows the possibility of ZKP.
Note: A good review of the history can be found in pazk.

In 1930s-1940s, the Church and Turing opened the door of computability, and Shockley, Brattain and Bardeen brought transistor to the world.

This is the dawn era.

## App-ZK | Hardwires

Since the end of 1980s, ZKP protocols for specific applications were proposed.
They are App-ZK, App-Specific-ZK.
There were no programmability in ZKP protocols.

The early computers are also hardwired, but soon evolved, as electronic computers are born to hold stored-programs.

This era is too early and short, let's move on.

## ZK-App | Softwares

It was in the late 1990s that ZKP protocols first became programmable.
In late 2000s, this programmability appeared to be more practical.
Arithmetic circuits are ZKP protocols' hardware.
It consists of ADD gate and MUL gate and every signal is an element of a finite field.
ZKP starts to have "frontends" which is gradually decoupled from the proving backend.

In late 1940s, the computers became programmable (EDSAC).
The computer hardware became programmable.

But the underpinning hardware of ZKP and electronic computers are so different that it brings major challenges in the programmability of ZKP.
One is arithmetic circuits, the other is electronic circuits of Boolean logic.

Besides, ZKP is about checking the correctness of the computation, not the computation itself.
So, to write a ZK program, developers need to write two parts:
- The witness computation program, which acts as normal program and provides hints of assignment of variables;
- The circuit, i.e. the constraint system;

So, the ZK program consists of the following components:
- Prover
  - Witness Generation (App-Specific)
  - Constraint System (App-Specific)
  - Prover (App-Independent)
- Verifier (App-Independent)

There is no mapping between ZKP programs and computer programs, and thus people cannot migrate easily.

### ZK-Lib | Asembly

People call programming in ZKP writing circuits, whereas in practice they do not write arithmetic circuits directly.
Instead, ZKP programs are described as Constraint Systems.
There are wires, aka signals, aka variables defined in the system.
You can add constraints to the system, which involves the variables and enforce them to satisfy some relations.

The basic type of constraints is polynomial.
In early 2010s, the most classic Rank-1 Constraint System (R1CS) was proposed.
(To learn why it is called Rank-1, see [the original thesis](https://dspace.mit.edu/bitstream/handle/1721.1/87953/880419628-MIT.pdf?sequence=2&isAllowed=y).)
In R1CS, each constraint is an equation.
The left hand side is the product of two linear combinations of the variables.
The right hand side is one linear combination of the variables.
Equivalently, it is an assert-zero constraint for some degree-2 polynomial.
In some other systems, the degree can be higher.
Refer to CCS for more details.

Modern ZKP protocols also introduces lookup constraints.
Lookup is a constraint that the value of a variable must be inside some pre-defined set (table).
Indexed lookup further imposes the index equivalence in the table.
Refer to Lasso for more details.

Besides, since ZKP is about checking, not computing, developers should also define how should the assignment of some variable be computed, outside the constraint system.
These variables are checked in the constriant system, given that checking is easier than computing directly.
This makes ZK App programming more complicated: unavoidable dual programming.

There are libaries for writing zkp circuits, including Gnark, Halo2, Bellman/Bellperson/Bellpepper, Libsnark, Arkworks, SnarkJS.
They are deeply coupled with the proving system.
The benefit is that the developer can combine the witness computation part and the circuit more easily.
It is efficient given that the developer can express the constraints in a finer grain way.
For example, equivalent checking can be achieved by either a more sparse circuit and a more compact circuit, by choosing different strategies to create the linear combination.

I would like to call them the "Assembly Language" of ZKP programs.

### ZK-Progamming Language | High-Level Language

#### ZK-DSL, ZK-HDL

Since late 2018s, people tried to wrap the circuit librabry into a DSL.
They have simpler semantics, with more gadgets and libraries.
Examples include Circom (High-Descrption Language), ZoKrates, Zinc (stopped), Chiquito (stopped) and Noir.

They provide more abstractions, and thus more programmability.
For example, when the developer writes `let y = 1/x`, under the hood, it is translated into the following circuit-witness-program:
1) Create a variable `y` in the circuit.
2) Compute the inverse of `x` **outside** the circuit, as the circuit does not support division. Then assign this value to variable `y`.
3) Add a constraint that `y * x = 1` in the circuit.

The DSL compiler hides the complexity from the developer.

However, it could be less efficient, as it is less flexible.
Additionally, for complicated witness computation, the developer still needs to write the program outside the circuit, which maybe even more difficult to acomplish with zkDSL.

The zkDSL developers still need to be aware of some ZKP knowledge, such as the data type and the finite field.
There is only finite field type and Boolean type.
No integer type, unsigned or signed.
No string type.

Overall, zk programming is still painful.

I would like to call them the "High-Level Language" of ZKP programs.
It resembles the HLLs such as ALGOL, BCPL, COBOL, C, etc.
(These HLLs have been invented shortly after the electronic computers were born, and they have quickly evolved since 1950s.)
But zkDSLs are less powerful and are still very unpopular.

#### ZK-Compiler

Some projects such as zk-LLVM are trying to target ZKP for the compilers of commonly used HLLs such as C, C++, Rust, etc.
In theory, I think it is feasible to compile the HLLs into an equivalent zk program.
The artifact consists of a witness computations program, a zkp circuit for checking the witness, and a template prover.
I guess that similar to ZKVM, there should also be some memory consisitency checking mechanisms, please see ZK-LLVM's design for more details.
For some programs such as GoLang, there could be some light runtime in the design of the zkp circuits.

Compared to zkDSL, this approach is friendly to much more developers and hides completely the zkp programming stuff including witness generation dual programming and finite field.

This approach is more efficient than zkVM, and zkLLVM might explain more in their introduction.

ZK-Compilers are ideal, as they completely decouple the ZKP from the application program.
However, I guess, it requires a lot more effort than zkVM.
Unfortunatelly, there is no community strong enough to support.

> [!NOTE]
> It is reminiscent of the Dalvik in Android.
> The adoption of Dalvik allowed the vast Java community to develop Android apps, which is key to the mass adoption of Android.
> If Android compiled C/C++ programs into native binaries, the program will run faster but the compilers development might be a very big problem: porting compilers to various targets is not easy.
> If Android app only accepts a new programming language, the adoption will be a big problem.

## ZK-VM | Abstract-VMs

In early 2010s, the idea of running a RAM machine with ZKP was born.
And since the beginning of 2020s, it went practical as part of the Ethereum layer 2 scaling solutions.

ZKVM fully decouples the application program from ZKP.
Developers are totally agnostic of ZKP: no witness generation, no constraint system, no finite field.
Developer can enjoy the complete list of data types and operations: uint, int, string; division, multiplication, etc.
(Note: for RV32I(M), there is still no support for floating point.)

By ZKVM, the ZKP circuit just checks some execution trace of the VM.
The trace is generated by a virtual machine emulator.
The circuit's job is to check if the trace is indeed generated by an emulator which aligns with the VM ISA spec.
The circuit should reject (unsat) the trace if no spec-compliant emulator can generate it.

The generated ZK Program of ZKVM consists of the following components:
- Prover
  - Extended Emulator / Witness Generation (App-Independent)
    - It just treat application program as some input data.
  - Constraint System (App-Independent)
  - Prover (App-Independent)
- Verifier (App-Independent)

With ZKVM, it is easy to maintain and upgrade the application, as the zkp circuit is for stable VM, the app upgrade does not change the circuit.

ZKVM not only enhances the programmability, but also the security.
If an applicaiton is written in ZKDSL/ZKLib, the responsibility of bug-free program is on the app developer.
With ZKVM, the responsibility transfers to the ZKVM developer.
Once the ZKVM is formally verified to realize the consistency of "emulator-circuit", the security of the application is also guaranteed.
See [Verified-ZKEVM](https://verified-zkevm.org/) for more details.

It resembles the abstract-VMs such as Java/JVM and CSharp/.Net/CLR, which were born in 1990s.
Although the execution effciency is much lower than the "native" programs, these VMs unlocked the possiblity of building large scale softwares with collaboration by a large group of developers.
(TODO: contribution of Java or OOP?)

For ZKP, this approach faciltates the vast majority of developers to build ZK applications.
It is a must for the mass adoption of ZKP.
Also, it is likely a long term solution, just like Java and WASM.

Overall, despite the fact that ZKVM is less efficient than zkDSL/zk-compiler, it empowers the vast majority of developers to build zk applications and the efficiency loss can be compensated by the fast enhancement of the ZK protocols.

### Specialized ZKVM, e.g. zkEVM

The development of practical ZKVM is promoted by Ethereum layer 2 scaling.
zkEVMs emerged in early 2020s.
Some ZK-friendly ISAs are also proposed for building ZKVMs, such as Cario and OlaVM.
Compared to ZKVMs with generic ISA (e.g. RISC-0), these ZKVMs are more specialized and the applications are limited, but the efficiency is higher.

### Generic ZKVM and VM-Over-ZKVM

Specialized ZKVM was firstly built to solve specific problems, mostly verifying Ethereum transactions.
Back then, the ZKP protocols alone were not efficient enough to prove more complicated programs with low overhead.
During 2022-2023, ZKVMs with generic ISAs such as RISC-0 emerged.
The targets of the applications are more general, including WebAssembly, RISC-V, MIPS.
Potentially, there could be more types of applications and zkVMs can go beyond Web3.

However, so far, the most popular application is still Ethereum (see [eth proofs](https://x.com/eth_proofs)), and an important benchmarks for the ZKVMs is the speed of verifying Ethereum transactions.
Generally, the architecture is VM-Over-ZKVM.
For Ethereum, it is EVM-Over-(RISC-V-)ZKVM.
This approach allows minimal change for EVM upgrade, which improves the security.
Even though it introduces a lot of overhead, the efficiency loss can be compensated by the fast enhancement of the ZK protocols.
So, the *VM-Over-ZKVM* architecture is gaining more popularity.

There are plenty of related projects.
You can find a list of ZKEVM/ZKVMs at [mainnet EVM snarkification](https://docs.google.com/presentation/d/1IzUsFZcq3Eg9zQfeLG2b9N99nJHJsbJ-TVVAReJ-9SA/edit?slide=id.g36178b4cd5a_0_2#slide=id.g36178b4cd5a_0_2).

### Folding | Paged Memory

ZKP is considered to be "scalable" if a program of arbitrary size can be executed.
ZKP used to be monolithic, and the program can only be treated as a whole, which blocked the proving of large programs.
Then continuation and recursive ZKP were introduced.
People started to look for solutions to IVC (Incremental Verifiable Computation) and PCD (Proof-Carrying Data).
The idea of folding scheme, which is the extremely efficient primitive for IVC, was proposed.
A Cambrian explosion of folding-based ZKP protocols was witnessed since the publication of Nova.

This is analogous to the paged memory management in the history modern computers.

With IVC, ZKP went from fast to both fast and scalable (Kothapalli).

This is a key technology for ZKVM: programs can be chunked into arbitrary number of pieces and the VM execution repeats chunk by chunk.

### Challenges

Is ZKVM already the silver bullet to achieve the ZKP programmability?

Currently, the normal program cannot be migrated to ZKVM seamlessly.
- One is that some paradigm shift is still required.
For example, a traditional ID verification program just do the parsing and verification and output the result.
To let it be a ZK program, the final output should also include some public property and nonce, which needs some wrapping.
(I expecte that this is a easy problem to solve.)
- Another problem is bare-metal programming for RISC-V ZKVM.
- The full ISA is still not supported by ZKVMs yet, which is also a blocking issue for app development.

Get back to a global viewpoint, could the speed of ZKVM kill the benefit of its enhancement on the programmability?

#### Bare Metal (RISC-V) ZKVM

Someone would expect that a program can be fully compiled into some binary executable, but in fact it would not.

For RISC-V VM, for example, when there is a `malloc` in a C program, it would be translated into a system call opcode which call the OS API for memory allocation at runtime.
The process such as finding a free memory block is not part of the executable, but instead, it is taken care by the OS.

For developers without expertise of embedded system, they will face the challenge of [bare-metal programming](https://en.wikipedia.org/wiki/Bare_machine).
Many important primitives will be missing, such as the memory allocation and IO.
Some works must be introduced to replace stdlib, such as the `no-std` attribute in Rust.

If ZKVM only supports bare-metal programming, the programmability is still limited.
So, for RISC-V ZKVM, although the developer can use Rust/C/C++, bare-metal programming is still required.
Hopefully in the future, this problem may be mitigated.
For ZKVM of other abstract VMs such as WASM and EVM, the bare metal programming is not required.

#### Overhead

The speed of the ZKVM is couple of magnitudes slower.
The (RISC-V) ZKVM emulates the hardware without OS, which already adds a lot of overhead.
Moreover, it runs slow.
The overall end-to-end speed could be a big problem, especially for VM-on-ZKVM solutions.
We need end-to-end benchmarking.


## VM Friendly ZKP "Hardware"

To mitigate the root difference between ZKP and electronic computers, some try to change the "hardware" of ZKP.
That is to replace the Arithmetic Circuits with other constraint systems such as Boolean Circuits.
Boolean circuits are more similar to electronic computers.
It is easier to express bitwise operations and maybe easier for formal verification.
But the size of Boolean circuits is very large, which is very inefficient.
There are already some ZKP protocols optimized for Boolean circuits.
If arithmetic circuits are still adopted, there are also some places in the ZKP proving protocols that can be optimized.


