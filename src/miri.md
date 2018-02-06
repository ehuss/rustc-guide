# Miri

Miri (**MIR** **I**nterpreter) is a virtual machine for executing MIR without
compiling to machine code. It is usually invoked via `tcx.const_eval`.

Its core datastructures can be found in `src/librustc/mir/interpret`. This is
mainly the error enum and the `Value` and `PrimVal` types. A `Value` can be
either `ByVal` (a single `PrimVal`), `ByValPair` (two `PrimVal`s, usually fat
pointers or two element tuples) or `ByRef`, which is used for anything else and
refers to a virtual allocation. These allocations can be accessed via the
methods on `tcx.interpret_interner`.

If you are expecting a numeric result, you can use `unwrap_u64` (panics on
anything that can't be representad as a `u64`) or `to_raw_bits` which results
in an `Option<u128>` yielding the `ByVal` if possible.

## Miri allocations

A miri allocation is either a byte sequence of the memory or an `Instance` in
the case of function pointers. Byte sequences can additionally contain
relocations that mark a group of bytes as a pointer to another allocation. The
actual bytes at the relocation refer to the offset inside the other allocation.

## Miri interpretation

Although the main entry point to constant evaluation is the `tcx.const_eval`
query, there are additional functions in `src/librustc_mir/interpret/const_eval`
that allow accessing the fields of a `Value` (`ByRef` or otherwise). You should
never have to access an `Allocation` directly except for translating it to the
compilation target (atm just LLVM).

### Details

Miri starts by allocating a virtual stack frame for the current constant that
is being evaluated. There's essentially no difference between a constant and a
function with no arguments, except that constants do not allow local (named)
variables at the time of writing this guide.

A stack frame is defined by the `Frame` type in
`src/librustc_mir/interpret/eval_context.rs` and contains all the local
variables memory (`None` at the start of evaluation).

Miri now calls the `step` method (in `src/librustc_mir/interpret/step.rs`) until
it either returns an error or has no further statements to execute. Each
statement will now initialize or modify the locals or the virtual memory
referred to by a local. This might require evaluating other constants or
statics, which just recursively invokes `tcx.const_eval`.
