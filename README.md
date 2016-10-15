# archsim
A Javascript processor implementation that can simulate any load-store architecture

## How to Use
To use the processor, you must first create an ISA.

### Creating your ISA
Your ISA must be a function which returns a list of your supported instructions. The function takes as input the processor's register file, the memory unit, and the program counter:

```
var isa = function (Register, Memory, ProgramCounter) {
  // ...
};
```

Again, you have to return a list of supported instructions. The format for instructions is:

### Instruction Format
```
{
  cmd: "[name of instruction]",
  desc: "[description of instruction]",
  syntax: [
    {
      src1: Register | Memory | Number
      src2: Register | Memory | Number
      dest: Register | Memory
    },
    ...
  ],
  eval: function (src1, src2) {
    // operate on src1 and src2 and return the result
  }
}
```

The `syntax` field is an array of supported syntax structures for the instruction. You can define multiple syntax structures for the same instruction; the decoder will move forward with whichever one is satisfied first.

The `src1`, `src2`, and `dest` fields should contain the type of the operand supported for this structure. For instance, the following defines a syntax structure for an instruction that can take a register descriptor as its first operand, a number as its second operand, and will store the result of the `eval` function into a memory cell:

```
syntax: [
  {
    src1: Register,
    src2: Number,
    dest: Memory
  }
]
```

If `dest` is not supplied, the processor will store the result (if it exists) into `src1`.

The `eval` function doesn't need to return a result; nothing will be stored if this is the case. This is useful for functions that mutate some internal state but don't require results to be stored anywhere. More on that below.

#### Internal State
Since your ISA is a function, you can use local variables to mimic processor internal components like condition bits, overflow registers, etc. You could even simulate an accumulator or stack architecture in the same way.

```
var isa = function (Register, Memory, ProgramCounter) {
  var overflow = 0;
  var condition = false;

  var accum = 0;

  return [
    // ...
  ];
}
```

#### Branching
Since the ProgramCounter is supplied as an argument to the ISA function, you are free to manipulate its value. Here's an example of an instruction format for an unconditional branch:

```
{
  cmd: "bra",
  desc: "Unconditional branch by a relative instruction offset",
  syntax: [
    {
      src1: Number
    }
  ],
  eval: function (src1) {
    ProgramCounter.set(ProgramCounter.get() + src1);
  }
}
```

### Instantiating the Processor
```
var processor = new Processor(isa, regCount, regSize, memCellCount, memCellSize);
```

To instantiate the processor, you call the function exposed by the `archsim` module and supply the ISA generator (see above) and the following arguments:

- `regCount`: The number of registers in your processor
- `regSize`: The size (in bits) of your registers (values are truncated to fit!)
- `memCellCount`: The number of memory cells in the memory unit
- `memCellSize`: How many bits each memory cell stores (8 should be common)

### Writing code
Operands are fetched in the same order they are defined in the syntax, so if you define `src1`, then `src2`, and then `dest`, the instruction can be used like this:

```
instr   {src1}, {src2}, {dest}
```

#### Register and Memory descriptors
To access registers, use `r[i]`, where `i` is the number of the register you wish to access. If `i` is out of range according to `regCount`, an error is thrown.

To access memory cells, use `m[i]`, where `m` is the "address" of the memory cell you wish to access. If `i` is out of range according to `memCellCount`, an error is thrown.

### Executing Code
To execute a program, just use `load()` followed by `exec()`. These operations can be chained:

```
processor.load(asm).exec();
```
